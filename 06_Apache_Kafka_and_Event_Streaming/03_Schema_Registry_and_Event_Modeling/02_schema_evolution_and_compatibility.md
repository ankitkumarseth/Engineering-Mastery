# Schema Evolution, Compatibility Modes & Contract Guardrails

## 1. The Challenge of Living Schemas in Event-Driven Systems

In microservice architectures, producers and consumers evolve and deploy independently on uncoordinated schedules. In an event-sourced or event-driven system, historical events are immutable facts persisted in Kafka partitions for days, months, or years.

If an upstream producer updates an Avro schema, downstream consumers running older or newer code must continue to deserialize events without throwing `AvroTypeException` or halting event processing.

```
+-------------------------------------------------------------------------------+
|                       SCHEMA EVOLUTION COMPATIBILITY MODES                    |
|                                                                               |
|  1. BACKWARD: Consumers (New Schema V2) can read Producers (Old Schema V1)   |
|     → Upgrade Order: Upgrade all Consumers FIRST, then upgrade Producers.     |
|                                                                               |
|  2. FORWARD:  Consumers (Old Schema V1) can read Producers (New Schema V2)   |
|     → Upgrade Order: Upgrade all Producers FIRST, then upgrade Consumers.     |
|                                                                               |
|  3. FULL:     Both Backward AND Forward Compatible across V1 <===> V2         |
|     → Upgrade Order: ANY ORDER! Deploy consumers and producers autonomously! |
+-------------------------------------------------------------------------------+
```

---

## 2. Definitive Schema Registry Compatibility Matrix

The table below outlines the mathematical rules governing each compatibility level in Confluent / Apicurio Schema Registry:

| Compatibility Mode | Can New Schema Read Old Data? | Can Old Schema Read New Data? | Permitted Schema Modifications | Deployment Upgrade Sequence |
| :--- | :--- | :--- | :--- | :--- |
| **`BACKWARD`** (Default) | **YES** | NO | • Delete existing fields<br>• Add **optional fields with default values** | **Consumers First**, then Producers |
| **`FORWARD`** | NO | **YES** | • Add new fields<br>• Delete optional fields | **Producers First**, then Consumers |
| **`FULL`** | **YES** | **YES** | • Add **only optional fields with default values**<br>• Delete **only optional fields with default values** | **Any Order** (Autonomous CD pipelines) |
| **`NONE`** | NO | NO | • Any modification permitted (No validation) | High risk of runtime production outages |
| **`TRANSITIVE`** Variants | **YES** (Across all historical versions) | **YES** (Across all historical versions) | • Same rules as above, but verified against **V1, V2, ... VN**, not just the immediately prior version | Mandatory for topics retaining events long-term |

---

## 3. Avro Schema Evolution: Practical Patterns & Anti-Patterns

### 3.1 The Golden Rule of Defaults
> [!IMPORTANT]
> **Every newly added field in an enterprise Avro schema MUST have a default value.**
> If an older consumer reads a record produced without this field, or if a newer consumer reads an older historical record, Avro substitutes the default value. If no default is declared, deserialization fails immediately.

```json
// Example: Adding an optional loyalty tier field safely
{
  "name": "loyaltyTier",
  "type": ["null", "string"],
  "default": null,
  "doc": "Optional loyalty tier level. Defaults to null for historical records."
}
```

### 3.2 Renaming Fields Safely via `aliases`
Renaming a field in a schema is mechanically identical to **deleting the old field and adding a new field**. Under standard rules, this breaks backwards compatibility.
- **The Avro Solution**: Use the `aliases` property:

```json
{
  "name": "accountNumber",
  "type": "string",
  "aliases": ["customerAccountNumber", "legacyAcctId"],
  "doc": "Current field name is accountNumber; aliases map legacy serialized fields seamlessly."
}
```
When a consumer with this schema encounters binary data written with `customerAccountNumber`, Avro maps the old name to `accountNumber` automatically during deserialization.

### 3.3 The Enum Symbol Evolution Trap
Enums in Avro represent a classic production trap. If an enum is defined as:
```json
{
  "type": "enum",
  "name": "CardStatus",
  "symbols": ["ACTIVE", "SUSPENDED", "CLOSED"]
}
```
If a producer adds a new symbol `"BLOCKED_FRAUD"`:
- An existing consumer running the old schema fails with `AvroTypeException: No matching symbol` when encountering `"BLOCKED_FRAUD"`.
- **The Solution (Avro 1.9+)**: Always declare a `default` fallback symbol:
```json
{
  "type": "enum",
  "name": "CardStatus",
  "symbols": ["ACTIVE", "SUSPENDED", "CLOSED", "UNKNOWN_STATUS"],
  "default": "UNKNOWN_STATUS"
}
```
If an older consumer encounters an unrecognized future enum symbol, Avro safely defaults to `UNKNOWN_STATUS` rather than crashing.

---

## 4. Automated CI/CD Contract Guardrails

Manual PR reviews cannot catch subtle transitive compatibility violations. Enterprise pipelines enforce schema validation during build time:

```
[ Developer PR ] (Modifies /src/main/avro/CardEvent.avsc)
       |
       v
[ Gradle/Maven Schema Registry Plugin ]
       |
       v  (Sends candidate schema to Schema Registry test endpoint)
[ POST /compatibility/subjects/card-events-value/versions/latest ]
       |
       +--- HTTP 200: {"is_compatible": true}  ====> [ Build Passes & Merges ]
       |
       \--- HTTP 200: {"is_compatible": false} ====> [ BUILD FAILED! PR BLOCKED ]
```

### 4.1 Maven Plugin Integration (`pom.xml`)
```xml
<plugin>
    <groupId>io.confluent</groupId>
    <artifactId>kafka-schema-registry-maven-plugin</artifactId>
    <version>7.5.0</version>
    <configuration>
        <schemaRegistryUrls>
            <param>https://schema-registry.enterprise.com</param>
        </schemaRegistryUrls>
        <subjects>
            <card-events-value>src/main/avro/CardEvent.avsc</card-events-value>
        </subjects>
    </configuration>
    <executions>
        <execution>
            <id>test-compatibility</id>
            <phase>test</phase>
            <goals>
                <goal>test-compatibility</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```
If a developer deletes a required field or introduces a non-defaulted field, the `mvn test` build fails instantly on the developer's laptop before code is pushed to Git.

---

## 5. Runnable Verification Suite

The following self-contained Java verification suite implements a **Schema Compatibility Evaluation Engine**, evaluating base vs. candidate schemas across `BACKWARD`, `FORWARD`, and `FULL` compatibility rules:

```java
package com.engineering.mastery.kafka;

import java.util.*;

/**
 * SchemaCompatibilityValidatorPOC
 *
 * Demonstrates:
 * 1. Avro schema compatibility validation: BACKWARD, FORWARD, and FULL.
 * 2. Detecting non-defaulted field additions and field deletions.
 * 3. Simulating Avro aliases for backwards-compatible field renaming.
 */
public class SchemaCompatibilityValidatorPOC {

    // =========================================================================
    // 1. Schema Metamodel
    // =========================================================================
    public record AvroField(
            String name,
            String type,
            boolean hasDefault,
            Object defaultValue,
            List<String> aliases
    ) {}

    public record AvroRecordSchema(
            String name,
            Map<String, AvroField> fields
    ) {}

    public enum CompatibilityMode { BACKWARD, FORWARD, FULL }

    // =========================================================================
    // 2. Compatibility Engine
    // =========================================================================
    public record CompatibilityResult(boolean isCompatible, List<String> errorMessages) {}

    public static class SchemaCompatibilityEvaluator {

        public static CompatibilityResult checkCompatibility(
                AvroRecordSchema previous,
                AvroRecordSchema candidate,
                CompatibilityMode mode
        ) {
            List<String> errors = new ArrayList<>();

            // A. Check BACKWARD compatibility:
            // A new schema can read old data IF:
            // 1. Every field in candidate that is NOT in previous MUST have a default value!
            // 2. Fields deleted from previous are safe (new schema simply ignores them).
            if (mode == CompatibilityMode.BACKWARD || mode == CompatibilityMode.FULL) {
                for (AvroField candField : candidate.fields().values()) {
                    // Check if field existed in previous (or matches an alias)
                    boolean existed = previous.fields().containsKey(candField.name());
                    if (!existed) {
                        for (String alias : candField.aliases()) {
                            if (previous.fields().containsKey(alias)) {
                                existed = true;
                                break;
                            }
                        }
                    }

                    if (!existed && !candField.hasDefault()) {
                        errors.add(String.format(
                                "[BACKWARD VIOLATION] Added field '%s' has NO default value! New consumers will fail reading historical data.",
                                candField.name()));
                    }
                }
            }

            // B. Check FORWARD compatibility:
            // An old schema can read new data IF:
            // 1. Every field in previous that is NOT in candidate MUST have had a default value!
            // 2. New fields added to candidate are safe (old schema simply ignores them).
            if (mode == CompatibilityMode.FORWARD || mode == CompatibilityMode.FULL) {
                for (AvroField prevField : previous.fields().values()) {
                    boolean existsInCandidate = candidate.fields().containsKey(prevField.name());
                    if (!existsInCandidate) {
                        // Check if candidate renamed via alias
                        for (AvroField candField : candidate.fields().values()) {
                            if (candField.aliases().contains(prevField.name())) {
                                existsInCandidate = true;
                                break;
                            }
                        }
                    }

                    if (!existsInCandidate && !prevField.hasDefault()) {
                        errors.add(String.format(
                                "[FORWARD VIOLATION] Deleted field '%s' had NO default value in old schema! Old consumers will fail reading new data.",
                                prevField.name()));
                    }
                }
            }

            return new CompatibilityResult(errors.isEmpty(), errors);
        }
    }

    // =========================================================================
    // 3. Verification Runner
    // =========================================================================
    public static void main(String[] args) {
        System.out.println("================================================================================");
        System.out.println("     AVRO SCHEMA COMPATIBILITY VALIDATION TEST SUITE                            ");
        System.out.println("================================================================================\n");

        // 1. Base Schema V1
        Map<String, AvroField> v1Fields = new HashMap<>();
        v1Fields.put("cardId", new AvroField("cardId", "string", false, null, List.of()));
        v1Fields.put("amount", new AvroField("amount", "double", false, null, List.of()));
        v1Fields.put("currency", new AvroField("currency", "string", true, "USD", List.of()));
        AvroRecordSchema schemaV1 = new AvroRecordSchema("CardPaymentEvent", v1Fields);

        System.out.println("--- [1. Base Schema V1 Definition] ---");
        System.out.printf("Fields: %s%n%n", schemaV1.fields().keySet());

        // 2. Candidate Schema V2A: Illegal Modification (Adding mandatory field without default)
        System.out.println("--- [2. Testing Candidate V2A: Non-Defaulted Field Addition] ---");
        Map<String, AvroField> v2AFields = new HashMap<>(v1Fields);
        v2AFields.put("fraudScore", new AvroField("fraudScore", "int", false, null, List.of())); // NO DEFAULT!
        AvroRecordSchema schemaV2A = new AvroRecordSchema("CardPaymentEvent", v2AFields);

        CompatibilityResult resV2A = SchemaCompatibilityEvaluator.checkCompatibility(
                schemaV1, schemaV2A, CompatibilityMode.BACKWARD);
        System.out.printf("Compatibility (BACKWARD): %b%n", resV2A.isCompatible());
        for (String err : resV2A.errorMessages()) {
            System.out.println("  ❌ " + err);
        }

        // 3. Candidate Schema V2B: Legal Modification (Adding optional field with default)
        System.out.println("\n--- [3. Testing Candidate V2B: Adding Optional Field with Default] ---");
        Map<String, AvroField> v2BFields = new HashMap<>(v1Fields);
        v2BFields.put("fraudScore", new AvroField("fraudScore", "int", true, 0, List.of())); // WITH DEFAULT!
        AvroRecordSchema schemaV2B = new AvroRecordSchema("CardPaymentEvent", v2BFields);

        CompatibilityResult resV2B = SchemaCompatibilityEvaluator.checkCompatibility(
                schemaV1, schemaV2B, CompatibilityMode.FULL);
        System.out.printf("Compatibility (FULL):     %b%n", resV2B.isCompatible());
        System.out.println("  ✓ SUCCESS: Safe for autonomous deployment of producers and consumers in any order!");

        // 4. Candidate Schema V2C: Field Renaming via Avro Aliases
        System.out.println("\n--- [4. Testing Candidate V2C: Safe Field Renaming via Aliases] ---");
        Map<String, AvroField> v2CFields = new HashMap<>(v1Fields);
        v2CFields.remove("cardId");
        // Renamed cardId -> primaryCardNumber, aliased to "cardId"
        v2CFields.put("primaryCardNumber", new AvroField(
                "primaryCardNumber", "string", false, null, List.of("cardId")));
        AvroRecordSchema schemaV2C = new AvroRecordSchema("CardPaymentEvent", v2CFields);

        CompatibilityResult resV2C = SchemaCompatibilityEvaluator.checkCompatibility(
                schemaV1, schemaV2C, CompatibilityMode.FULL);
        System.out.printf("Compatibility (FULL with Aliases): %b%n", resV2C.isCompatible());
        System.out.println("  ✓ SUCCESS: Aliased field 'cardId' -> 'primaryCardNumber' preserves full compatibility!");

        System.out.println("\n================================================================================");
        System.out.println("                      VERIFICATION COMPLETED SUCCESSFULLY                       ");
        System.out.println("================================================================================");
    }
}
```

---

## 6. Staff / Principal Interview Strategy

### 6.1 Breaking Changes: When to Version the Topic
When an interviewer asks: *"We have a business requirement that fundamentally changes an event contract in a way that violates `FULL` compatibility (e.g., splitting a monolithic `PaymentEvent` into two distinct domain events). How do you execute this without downtime?"*

#### The Principal Architecture Playbook:
1. **Never Force a Breaking Schema into an Existing Topic**:
   - Do NOT change the Schema Registry mode to `NONE` to bypass validation. This leads to silent consumer deserialization crashes.
2. **The New Topic Version Pattern (`orders.v2`)**:
   - Create a brand-new Kafka topic: `orders.v2`.
   - Register the new schema under `orders.v2-value`.
3. **The Dual-Publish Transition Pipeline**:
   - **Phase 1**: Upstream producers are updated to dual-publish to both `orders.v1` (legacy Avro) and `orders.v2` (new Avro).
   - **Phase 2**: Downstream consumer groups migrate one by one to consume `orders.v2`.
   - **Phase 3**: Once telemetry verifies zero traffic on `orders.v1`, producer dual-publishing is disabled and `orders.v1` is deprecated and decommissioned.

### 6.2 Transitive Compatibility in Long-Lived Topics
When asked: *"Why is `BACKWARD_TRANSITIVE` preferred over simple `BACKWARD` in event-sourced systems?"*

#### The Staff Engineering Answer:
- Explain: *"Simple `BACKWARD` compatibility only verifies the candidate schema against version $N-1$."*
- *"In an event-sourced architecture (like Axon on Kafka), topics retain events forever. If Schema V1 had field $A$, V2 deleted field $A$ (legal in backward), and V3 re-introduced field $A$ with a different data type (e.g. from string to long):"*
  - V3 is backward-compatible with V2.
  - But when a consumer attempts to replay the topic from genesis (Offset 0), it encounters events written in V1 format and crashes!
- Propose the enterprise standard:
  - Always enforce **`BACKWARD_TRANSITIVE`** or **`FULL_TRANSITIVE`** on event-sourced topics to guarantee compatibility across all historical versions since cluster inception.
