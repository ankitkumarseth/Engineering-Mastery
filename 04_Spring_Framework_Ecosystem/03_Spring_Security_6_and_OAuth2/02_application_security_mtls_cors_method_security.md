# Transport & Application Security: mTLS, CORS Mechanics, CSRF & Method Security

High-security enterprise architectures—particularly Tier-0 financial networks, banking rails, and defense systems—enforce defense-in-depth across the entire request lifecycle. In this masterclass, we explore **Mutual TLS (mTLS) with Spring Boot 3.1+ SSL bundles, the notorious CORS-vs-Security filter ordering trap, CSRF mechanics, and high-performance method-level security with Spring Expression Language (SpEL)**.

---

## 1. Mutual TLS (mTLS) & Spring Boot 3.1+ SSL Bundles

Standard TLS validates only the server's identity to the client. **Mutual TLS (mTLS)** establishes true zero-trust network encryption: both client and server cryptographically verify each other's identity using X.509 digital certificates during the TLS handshake.

```
 Client (Payment Switch)                               Spring Boot Server (Settlement Core)
         │                                                            │
         │ 1. ClientHello (Supported Ciphers)                         │
         ├───────────────────────────────────────────────────────────►│
         │ 2. ServerHello + Server Certificate + CertificateRequest   │
         │◄───────────────────────────────────────────────────────────┤
         │ 3. Client Certificate + ClientKeyExchange + CertificateVerify│
         ├───────────────────────────────────────────────────────────►│
         │                                                            │
         │ [ Server verifies Client Cert against Truststore CA ]      │
         │ [ Client verifies Server Cert against Client Truststore ]  │
         │                                                            │
         │ 4. Handshake Finished (Encrypted Symmetric Session Key)    │
         │◄──────────────────────────────────────────────────────────►│
```

### The Spring Boot 3.1 Paradigm Shift: SSL Bundles
Prior to Spring Boot 3.1, configuring SSL required fragmented properties (`server.ssl.*`) that could not be shared across Web servers, RestTemplates, and WebClients.

Spring Boot 3.1 introduces unified, reloadable **SSL Bundles**:

```yaml
# application.yml
spring:
  ssl:
    bundle:
      jks:
        payment-switch-mtls:
          # Server identity (Private Key + Public Cert)
          key:
            alias: "settlement-server"
          keystore:
            location: "classpath:keystores/server-keystore.p12"
            password: "${KEYSTORE_SECRET}"
            type: "PKCS12"

          # Trusted Client Certificates Authority (CA)
          truststore:
            location: "classpath:keystores/server-truststore.p12"
            password: "${TRUSTSTORE_SECRET}"
            type: "PKCS12"

# Bind SSL Bundle to Embedded Tomcat
server:
  port: 8443
  ssl:
    bundle: "payment-switch-mtls"
    client-auth: "need" # Enforces mandatory mTLS; drops connection if client cert missing!
```

### Extracting Client X.509 Certificate in Spring Security
When `client-auth: need` is enabled, Spring Security can extract the authenticated subject directly from the TLS session without requiring HTTP headers:

```java
@Configuration
@EnableWebSecurity
public class MtlsSecurityConfig {

    @Bean
    public SecurityFilterChain mtlsFilterChain(HttpSecurity http) throws Exception {
        http
            .securityMatcher("/api/v1/switch/**")
            .x509(x509 -> x509
                .subjectPrincipalRegex("CN=(.*?)(?:,|$)") // Extracts Common Name (e.g. "switch-node-01")
                .userDetailsService(new X509CertificateUserDetailsService())
            )
            .authorizeHttpRequests(auth -> auth
                .anyRequest().hasRole("TRUSTED_SWITCH")
            );
        return http.build();
    }
}
```

---

## 2. The CORS vs. Security Filter Ordering Trap

One of the most confusing errors encountered by engineering teams is the **CORS pre-flight failure under Spring Security**.

### The Anatomy of an HTTP CORS Pre-Flight Call
When a single-page app (React/Vue/Angular) running on `https://portal.domain.com` makes a `POST` or `PUT` call with custom headers (such as `Authorization`) to an API on `https://api.domain.com`:
1. The browser **first** issues an HTTP **`OPTIONS` pre-flight request**.
2. The browser **intentionally omits the `Authorization` header** from the `OPTIONS` request.
3. The server must respond with `200 OK` and specific CORS headers (`Access-Control-Allow-Origin`, `Access-Control-Allow-Headers`).
4. Only upon receiving the `200 OK` pre-flight will the browser transmit the real `POST` request with the `Authorization: Bearer <JWT>` token!

### The Trap: Configuring CORS via `WebMvcConfigurer`
```java
// ❌ THE PRODUCTION TRAP: Configured in MVC instead of Security!
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**").allowedOrigins("https://portal.domain.com");
    }
}
```

### Why It Fails
1. Spring Security filters execute at the Servlet layer **before** Spring MVC's `DispatcherServlet` is reached.
2. The browser sends the HTTP `OPTIONS` pre-flight request without an `Authorization` header.
3. The request hits Spring Security's `AuthorizationFilter`.
4. Spring Security detects that the request has no credentials, returns **`HTTP 401 Unauthorized` (or `403 Forbidden`)**, and terminates the call.
5. The browser never receives the CORS headers; the browser console logs a generic `CORS error: Preflight request was not successful`.
6. `WebMvcConfigurer` was never even executed!

### The Staff-Level Solution: Integrated Security CORS
You must configure CORS directly within Spring Security so that `CorsFilter` is placed **at the very beginning of the `SecurityFilterChain`**:

```java
@Bean
public SecurityFilterChain apiFilterChain(HttpSecurity http) throws Exception {
    http
        // 1. MUST integrate cors directly in HttpSecurity!
        .cors(cors -> cors.configurationSource(corsConfigurationSource()))
        .csrf(csrf -> csrf.disable())
        .authorizeHttpRequests(auth -> auth
            // Spring Security will automatically permit all OPTIONS calls if CORS is enabled!
            .anyRequest().authenticated()
        )
        .oauth2ResourceServer(oauth2 -> oauth2.jwt(Customizer.withDefaults()));

    return http.build();
}

@Bean
public CorsConfigurationSource corsConfigurationSource() {
    CorsConfiguration config = new CorsConfiguration();
    config.setAllowedOrigins(List.of("https://portal.domain.com", "https://admin.domain.com"));
    config.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE", "OPTIONS"));
    config.setAllowedHeaders(List.of("Authorization", "Content-Type", "X-Trace-Id"));
    config.setExposedHeaders(List.of("X-Trace-Id"));
    config.setAllowCredentials(true);
    config.setMaxAge(3600L); // Cache pre-flight response for 1 hour to reduce network overhead

    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/**", config);
    return source;
}
```

---

## 3. CSRF Protection: When to Enforce vs. When to Disable

Cross-Site Request Forgery (CSRF) occurs when an unauthorized website tricks an authenticated user's browser into executing an unwanted action on a trusted site.

### The Litmus Test: How are Credentials Stored?

```
                     How does the Client transmit credentials?
                                      │
               ┌──────────────────────┴──────────────────────┐
               ▼                                             ▼
       HTTP Cookies (Session/OAuth)                  Authorization: Bearer <JWT>
   (Automatically attached by browser)             (Manually attached by JavaScript)
               │                                             │
               ▼                                             ▼
    ⚠️ CSRF ATTACK POSSIBLE!                       🛡️ CSRF ATTACK IMPOSSIBLE!
    Enforce CSRF Protection                        Disable CSRF (csrf.disable())
```

### When to Disable CSRF:
- **Stateless REST APIs**: Client applications (mobile apps, microservices, SPA frontends) store tokens in memory/storage and pass `Authorization: Bearer <JWT>`. Disabling CSRF (`csrf.disable()`) is completely secure and standard industry practice.

### When to Enforce CSRF (Single-Page Apps with Session Cookies):
If your architecture uses cookie-based sessions with an SPA frontend:
- Use **`CookieCsrfTokenRepository.withHttpOnlyFalse()`**:
  This writes the CSRF token into a cookie named `XSRF-TOKEN` with `HttpOnly = false`, allowing frontend JavaScript to read the token and echo it back in the `X-XSRF-TOKEN` HTTP header on write operations (`POST`, `PUT`, `DELETE`).

---

## 4. Method-Level Security: `@PreAuthorize` & SpEL Evaluation

While URL-based authorization in `SecurityFilterChain` handles macro-level access (e.g. `/api/v1/admin/**`), microservices require fine-grained, data-aware authorization at the domain layer.

### The Spring Security 6 Standard: `@EnableMethodSecurity`
Spring Security 6 deprecates `@EnableGlobalMethodSecurity` in favor of `@EnableMethodSecurity`:

```java
@Configuration
@EnableMethodSecurity(
    prePostEnabled = true, // Enables @PreAuthorize, @PostAuthorize, @PreFilter, @PostFilter (Default: true)
    securedEnabled = false, // Disables legacy @Secured
    jsr250Enabled = false   // Disables legacy @RolesAllowed
)
public class MethodSecurityConfig { }
```

### The 4 Core Method Annotations

| Annotation | Execution Point | Primary Use Case |
| :--- | :--- | :--- |
| **`@PreAuthorize`** | **Before** method runs. | Validating roles, scopes, or whether user owns the requested entity ID. |
| **`@PostAuthorize`** | **After** method runs, before returning to caller. | Inspecting the returned domain object (`returnObject.ownerId == authentication.name`). |
| **`@PreFilter`** | **Before** method runs. | Filtering collection arguments in-place before processing. |
| **`@PostFilter`** | **After** method runs. | Filtering elements from a returned `Collection` based on user permissions. |

### Enterprise SpEL Patterns in Production

```java
@Service
public class SettlementService {

    // 1. Role and Scope Evaluation
    @PreAuthorize("hasRole('OPERATOR') and hasAuthority('SCOPE_settlement:execute')")
    public void executeSettlementBatch(String batchId) { ... }

    // 2. Dynamic Parameter Inspection (User can only access their own account)
    @PreAuthorize("hasRole('ADMIN') or #accountNumber == authentication.principal.claims['account_id']")
    public AccountBalance getBalance(String accountNumber) { ... }

    // 3. PostAuthorize: Inspecting Return Object
    @PostAuthorize("returnObject.assignee == authentication.name or hasRole('AUDITOR')")
    public AuditReport generateReport(String reportId) {
        return reportRepository.findById(reportId);
    }

    // 4. Custom Security Expression Handler (Custom Business Logic)
    @PreAuthorize("@accountSecurityChecker.canDebitsBeIssued(#request, authentication)")
    public TransactionResult processDebit(DebitRequest request) { ... }
}
```

> [!NOTE]
> **Performance Optimization**:
> Evaluating complex SpEL expressions via reflection introduces slight CPU overhead. High-throughput hot paths ($>50\text{k}$ TPS) should perform coarse-grained role checks at the API Gateway or URL filter chain, reserving `@PreAuthorize` for domain mutations or sensitive administrative actions.

---

## 5. Runnable Verification POC: Method Security & SpEL Evaluation

Save and execute this self-contained class to observe how SpEL expressions evaluate permissions dynamically:

```java
package com.mastery.spring.security;

import org.springframework.expression.Expression;
import org.springframework.expression.ExpressionParser;
import org.springframework.expression.spel.standard.SpelExpressionParser;
import org.springframework.expression.spel.support.StandardEvaluationContext;

import java.util.List;
import java.util.Map;

public class MethodSecurityAndSpELPOC {

    public record UserPrincipal(String username, String department, List<String> roles) {}
    public record SettlementRequest(String id, double amount, String targetDepartment) {}

    public static class SecurityEvaluator {
        private final ExpressionParser parser = new SpelExpressionParser();

        public boolean evaluate(String spelExpression, UserPrincipal principal, SettlementRequest request) {
            StandardEvaluationContext context = new StandardEvaluationContext();
            context.setVariable("principal", principal);
            context.setVariable("req", request);

            Expression expression = parser.parseExpression(spelExpression);
            Boolean result = expression.getValue(context, Boolean.class);
            return result != null && result;
        }
    }

    public static void main(String[] args) {
        SecurityEvaluator evaluator = new SecurityEvaluator();

        UserPrincipal staffLead = new UserPrincipal("ankit", "PAYMENTS", List.of("ROLE_SETTLEMENT_LEAD", "ROLE_USER"));
        UserPrincipal juniorDev = new UserPrincipal("intern", "MARKETING", List.of("ROLE_USER"));

        SettlementRequest smallPayment = new SettlementRequest("SETTLE-101", 50000.0, "PAYMENTS");
        SettlementRequest hugePayment = new SettlementRequest("SETTLE-999", 5000000.0, "PAYMENTS");

        // Staff Rule: Must belong to target department AND have LEAD role OR amount < 100k
        String securityRule = "#principal.department == #req.targetDepartment and " +
                              "(#principal.roles.contains('ROLE_SETTLEMENT_LEAD') or #req.amount < 100000.0)";

        System.out.println("=== EVALUATING METHOD SECURITY SPEL RULES ===");
        System.out.println("Rule: " + securityRule);

        boolean test1 = evaluator.evaluate(securityRule, staffLead, hugePayment);
        System.out.println("\nTest 1 [Staff Lead ($5M Payment)]: Permitted? " + test1);

        boolean test2 = evaluator.evaluate(securityRule, juniorDev, smallPayment);
        System.out.println("Test 2 [Junior Dev Wrong Dept ($50k Payment)]: Permitted? " + test2);

        UserPrincipal paymentsAnalyst = new UserPrincipal("analyst", "PAYMENTS", List.of("ROLE_USER"));
        boolean test3 = evaluator.evaluate(securityRule, paymentsAnalyst, smallPayment);
        System.out.println("Test 3 [Payments Analyst ($50k Payment)]: Permitted? " + test3);

        boolean test4 = evaluator.evaluate(securityRule, paymentsAnalyst, hugePayment);
        System.out.println("Test 4 [Payments Analyst ($5M Payment)]: Permitted? " + test4 + " (Correctly Denied!)");
    }
}
```

---

## 6. Staff / Principal Interview Strategy: Transport & Application Security

### Classic Traps & High-Leverage Responses

#### Trap 1: *"Why does our frontend receive a CORS error on `POST /api/orders` even though we configured `allowedOrigins("*")` in `WebMvcConfigurer`?"*
- **Principal Counter**: *"Because Spring MVC's `WebMvcConfigurer` executes inside `DispatcherServlet`, which sits **behind** Spring Security's filter chain. When the browser makes a CORS pre-flight `OPTIONS` call, it does not send the `Authorization` header. Spring Security's `AuthorizationFilter` intercepts the request before it ever reaches MVC, sees no authentication token, and immediately returns `401 Unauthorized` without CORS headers. To resolve this, remove the CORS configuration from `WebMvcConfigurer` and register a `CorsConfigurationSource` directly inside the `SecurityFilterChain` (`http.cors(...)`). This ensures the `CorsFilter` runs at the front of the filter pipeline, intercepting `OPTIONS` requests and returning `200 OK` with CORS headers before authentication is enforced."*

#### Trap 2: *"How do you design zero-trust microservice communication between two internal services in Kubernetes?"*
- **Principal Counter**: *"I implement a defense-in-depth model: **Mutual TLS (mTLS) at the transport layer, paired with OAuth2 Client Credentials JWTs at the application layer**. The mTLS handshake (configured via Spring Boot 3.1 SSL bundles or Istio/Envoy sidecars) verifies that the connection originated from an authorized workload certificate in the cluster and encrypts the wire. The application-layer JWT validates organizational identity, tenant boundary, and fine-grained scopes. This dual approach ensures that even if network encryption terminates at a mesh ingress or sidecar, the application layer cannot be spoofed."*

#### Trap 3: *"What is the architectural downside of relying excessively on `@PostAuthorize`?"*
- **Principal Counter**: *"Resource waste and side effects. `@PostAuthorize` executes **after** the underlying method has already executed. If a service method performs heavy database queries, allocates memory, or triggers an external write (e.g. initiating an SMS or modifying a record) and `@PostAuthorize` subsequently fails, the caller receives `403 Forbidden`, but the operation has already mutated state or burned server resources. `@PostAuthorize` should strictly be used for read-only operations where authorization depends on entity fields that only the database query can reveal; all mutations must be guarded with `@PreAuthorize`."*
