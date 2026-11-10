# Java Collections Framework (JCF) Refresher: Fundamentals, Idioms & Python-to-Java Rosetta Stone

```
========================================================================================================================
                                     JAVA COLLECTIONS FRAMEWORK (JCF) ARCHITECTURE
========================================================================================================================

                                          ┌──────────────────────┐
                                          │  Iterable<E> (Root)  │
                                          └──────────┬───────────┘
                                                     │
                                          ┌──────────▼───────────┐
                                          │    Collection<E>     │
                                          └──────────┬───────────┘
                         ┌───────────────────────────┼───────────────────────────┐
                         ▼                           ▼                           ▼
              ┌─────────────────────┐     ┌─────────────────────┐     ┌─────────────────────┐
              │       List<E>       │     │       Set<E>        │     │      Queue<E>       │
              │  Ordered, Indexable │     │ Unique, No Duplicates│    │ FIFO Processing     │
              └──────────┬──────────┘     └──────────┬──────────┘     └──────────┬──────────┘
                         │                           │                           │
          ┌──────────────┴──────────────┐            │                           ▼
          ▼                             ▼            │                ┌─────────────────────┐
   ┌─────────────┐               ┌────────────┐      │                │      Deque<E>       │
   │  ArrayList  │               │ LinkedList │      │                │ Double-Ended Queue  │
   │  (Dynamic   │               │  (Doubly-  │      │                └──────────┬──────────┘
   │   Array)    │               │   Linked)  │      │                           │
   └─────────────┘               └────────────┘      │                           ▼
                                                     │                ┌─────────────────────┐
                         ┌───────────────────────────┴───────────┐    │     ArrayDeque      │
                         ▼                                       ▼    │ (Fast Stack/Queue)  │
                  ┌─────────────┐                         ┌───────────┴─┐└─────────────────────┘
                  │   HashSet   │                         │   TreeSet   │
                  │   (O(1)     │                         │ (Red-Black  │
                  │  Hash-Table)│                         │ O(log n))   │
                  └──────┬──────┘                         └─────────────┘
                         ▼
                  ┌─────────────┐
                  │LinkedHashSet│
                  │ (Insertion) │
                  └─────────────┘

  ┌───────────────────────────────── MAP HIERARCHY (SEPARATE ROOT) ─────────────────────────────────┐
  │                                                                                                  │
  │                                       ┌──────────────────────┐                                   │
  │                                       │      Map<K, V>       │                                   │
  │                                       └──────────┬───────────┘                                   │
  │                         ┌────────────────────────┼────────────────────────┐                      │
  │                         ▼                        ▼                        ▼                      │
  │                  ┌─────────────┐          ┌─────────────┐          ┌─────────────┐               │
  │                  │   HashMap   │          │LinkedHashMap│          │   TreeMap   │               │
  │                  │  (Buckets + │          │ (LRU Cache /│          │ (Red-Black  │               │
  │                  │  Tree-Bins) │          │  Insertion) │          │ Key Sorted) │               │
  │                  └─────────────┘          └─────────────┘          └─────────────┘               │
  │                                                                                                  │
  │  Thread-Safe Enterprise Variant:                                                                │
  │  • ConcurrentHashMap (Lock-free reads via volatile + bin-head synchronized writes + CounterCell)  │
  └──────────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 1. Python-to-Java Collections Rosetta Stone Cheat Sheet

Coming back to Java from Python? Use this quick-reference lookup to immediately map Python data structures and idioms to their modern Java equivalents:

| Data Structure / Operation | Python 3 | Java 17 / 21 LTS | Time Complexity |
| :--- | :--- | :--- | :--- |
| **Dynamic Array** | `nums = []` / `nums.append(5)` | `List<Integer> nums = new ArrayList<>();`<br>`nums.add(5);` | Append: $O(1)$ amortized<br>Lookup: $O(1)$ |
| **Lookup by Index** | `val = nums[2]` | `int val = nums.get(2);` | $O(1)$ |
| **Length / Size** | `len(nums)` | `nums.size()` | $O(1)$ |
| **Membership Check** | `x in nums` / `x in my_set` | `nums.contains(x)` / `set.contains(x)` | List: $O(n)$, Set: $O(1)$ |
| **Hash Set** | `s = {1, 2, 3}` / `s.add(4)` | `Set<Integer> s = new HashSet<>();`<br>`s.add(4);` | $O(1)$ average |
| **Hash Map / Dict** | `d = {"a": 1}` / `d["b"] = 2` | `Map<String, Integer> d = new HashMap<>();`<br>`d.put("b", 2);` | $O(1)$ average |
| **Dict Get with Default** | `d.get("key", 0)` | `d.getOrDefault("key", 0)` | $O(1)$ |
| **Default Dict (Group By)** | `d = defaultdict(list)`<br>`d[k].append(v)` | `d.computeIfAbsent(k, key -> new ArrayList<>()).add(v);` | $O(1)$ |
| **Immutable Tuple** | `pt = (10, 20)` | `record Point(int x, int y) {}`<br>`Point pt = new Point(10, 20);` | Value-based, stack-friendly |
| **Min-Heap (Priority Queue)**| `import heapq`<br>`heapq.heappush(h, val)` | `PriorityQueue<Integer> pq = new PriorityQueue<>();`<br>`pq.offer(val);` | Push/Pop: $O(\log n)$ |
| **Max-Heap** | `heapq.heappush(h, -val)` | `PriorityQueue<Integer> pq = new PriorityQueue<>(Comparator.reverseOrder());` | Push/Pop: $O(\log n)$ |
| **Double-Ended Queue (Deque)**| `from collections import deque`<br>`q.appendleft(1); q.pop()` | `Deque<Integer> q = new ArrayDeque<>();`<br>`q.addFirst(1); q.pollLast();` | $O(1)$ |
| **LIFO Stack** | `stack = []`<br>`stack.append(x); stack.pop()`| `Deque<Integer> stack = new ArrayDeque<>();`<br>`stack.push(x); stack.pop();` | $O(1)$ |
| **List Comprehension** | `[x * 2 for x in nums if x > 0]` | `nums.stream().filter(x -> x > 0).map(x -> x * 2).toList();` | $O(n)$ |
| **Sorting In-Place** | `nums.sort(reverse=True)` | `nums.sort(Comparator.reverseOrder());` | $O(n \log n)$ (TimSort) |
| **Sort by Custom Key** | `users.sort(key=lambda u: u.age)`| `users.sort(Comparator.comparingInt(User::age));` | $O(n \log n)$ |
| **Dictionary Iteration** | `for k, v in d.items():` | `d.forEach((k, v) -> System.out.println(k + "=" + v));` | $O(n)$ |

---

## 2. The Core Collection Interfaces & Implementations

### 2.1 The `List<E>` Family: Ordered, Index-Accessible Collections

```java
// 1. Instantiating Lists:
List<String> mutableList = new ArrayList<>();        // Standard resizable array (capacity: 10, grows 1.5x)
List<String> fixedList   = Arrays.asList("A", "B");   // Fixed-size wrapper over array (CANNOT add/remove!)
List<String> modernList  = List.of("A", "B", "C");    // Immutable (Java 9+), rejects nulls!

// 2. Fundamental Operations:
mutableList.add("Apple");
mutableList.add(0, "Banana");                         // Inserts at index 0 (shifts elements right: O(n))
String item = mutableList.get(0);                      // Random access: O(1)
mutableList.set(0, "Blueberry");                      // Replaces element at index 0: O(1)
boolean removed = mutableList.remove("Apple");         // Linear scan + shift: O(n)
int size = mutableList.size();                         // O(1)
boolean hasApple = mutableList.contains("Apple");      // Linear scan: O(n)

// 3. Sorting Lists (Modern Idiom):
mutableList.sort(Comparator.naturalOrder());          // Ascending
mutableList.sort(Comparator.reverseOrder());          // Descending
// Multi-field sorting (e.g. sort by age descending, then by name ascending):
record User(String name, int age) {}
List<User> users = new ArrayList<>(List.of(new User("Alice", 30), new User("Bob", 25)));
users.sort(Comparator.comparingInt(User::age).reversed().thenComparing(User::name));
```

#### `ArrayList` vs. `LinkedList` vs. `CopyOnWriteArrayList`:
- **`ArrayList`**: Default choice in 99.9% of production code. Contiguous memory array. CPU L1/L2 cache prefetching makes it exponentially faster than `LinkedList` even for head insertions up to small array sizes!
- **`LinkedList`**: Doubly-linked nodes (`Node { Node prev; E item; Node next; }`). Every node incurs 24 bytes of memory overhead. Cache-unfriendly pointer chasing. **Never use in modern high-performance Java.**
- **`CopyOnWriteArrayList`**: Thread-safe variant. Mutating operations (`add`, `set`) create a fresh copy of the underlying array. Read operations (`get`, iterator) run lock-free with zero synchronization. Ideal for read-heavy observer subscriber registries.

---

### 2.2 The `Set<E>` Family: Unique, Duplicate-Free Collections

```java
// 1. Instantiation:
Set<String> hashSet   = new HashSet<>();              // Unordered, fastest: O(1) average
Set<String> linkedSet = new LinkedHashSet<>();        // Predictable: preserves insertion order!
Set<String> treeSet   = new TreeSet<>();              // Sorted: Red-Black tree O(log n)
Set<String> immutSet  = Set.of("X", "Y", "Z");         // Immutable set (Java 9+), throws on duplicates!

// 2. Fundamental Operations:
hashSet.add("EUR");
hashSet.add("USD");
boolean addedDuplicate = hashSet.add("EUR");          // Returns FALSE! "EUR" already exists.
boolean exists = hashSet.contains("USD");              // O(1) average lookup

// 3. TreeSet Navigable Power (Range queries, floor, ceiling):
TreeSet<Integer> scores = new TreeSet<>(List.of(10, 25, 50, 75, 90));
int lowest  = scores.first();                         // 10
int highest = scores.last();                          // 90
Integer floorVal   = scores.floor(55);                // 50 (largest element <= 55)
Integer ceilingVal = scores.ceiling(55);              // 75 (smallest element >= 55)
NavigableSet<Integer> range = scores.subSet(25, true, 75, true); // [25, 50, 75]
```

> [!IMPORTANT]
> **The `equals()` & `hashCode()` Contract**:
> Both `HashSet` and `HashMap` rely on `hashCode()` to find the bucket index and `equals()` to resolve collision equality. If you store a custom class in a Set without overriding both `equals()` and `hashCode()`, duplicate objects will be added, breaking Set semantics!

---

### 2.3 The `Map<K, V>` Family: Key-Value Associations

```java
// 1. Instantiation:
Map<String, Integer> map = new HashMap<>();           // Default, unordered: O(1) average
Map<String, Integer> linked = new LinkedHashMap<>(16, 0.75f, true); // Access-order LRU cache!
Map<String, Integer> sorted = new TreeMap<>();        // Sorted by keys: O(log n)
Map<String, Integer> immut  = Map.of("A", 1, "B", 2); // Immutable up to 10 entries
Map<String, Integer> bigImmut = Map.ofEntries(        // Immutable for arbitrary entries
    Map.entry("USD", 84),
    Map.entry("EUR", 92)
);

// 2. Essential Operations & Modern Idioms:
map.put("Apple", 100);
int qty = map.getOrDefault("Banana", 0);              // Returns 0 if key missing

// computeIfAbsent (The Java counterpart to Python's defaultdict):
Map<String, List<String>> userRoles = new HashMap<>();
userRoles.computeIfAbsent("admin", role -> new ArrayList<>()).add("READ_WRITE");

// merge (Count frequency / accumulator):
Map<String, Integer> wordCounts = new HashMap<>();
wordCounts.merge("apple", 1, Integer::sum);           // If absent: sets 1. If present: adds 1.

// 3. Iterating Maps:
// Approach A: Functional forEach (Cleanest!)
map.forEach((key, val) -> System.out.println(key + " -> " + val));

// Approach B: entrySet loop (Fastest for traditional loops)
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}
```

---

### 2.4 `Queue<E>` & `Deque<E>`: FIFO Queues, LIFO Stacks & Priority Heaps

In modern Java, **never use the legacy `java.util.Stack` class** (it extends `Vector` and synchronizes every single method, introducing massive thread lock contention). Always use **`ArrayDeque`** for both stacks and queues!

```java
// 1. LIFO Stack (ArrayDeque is 5x faster than java.util.Stack):
Deque<String> stack = new ArrayDeque<>();
stack.push("Page1");
stack.push("Page2");
String top = stack.peek();                            // "Page2" (reads top without removing)
String popped = stack.pop();                          // "Page2" (removes top)

// 2. FIFO Queue:
Queue<String> queue = new ArrayDeque<>();
queue.offer("Request1");                              // Enqueue at tail
queue.offer("Request2");
String head = queue.peek();                           // "Request1"
String processed = queue.poll();                      // "Request1" (dequeues head)

// 3. PriorityQueue (Binary Min-Heap by default):
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
minHeap.offer(40);
minHeap.offer(10);
minHeap.offer(30);
int minVal = minHeap.poll();                          // 10 (smallest element polled first: O(log n))

// PriorityQueue as Max-Heap (Reverse Comparator):
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
maxHeap.offer(40);
maxHeap.offer(10);
maxHeap.offer(30);
int maxVal = maxHeap.poll();                          // 40 (largest element polled first)
```

---

## 3. Top 3 Production Traps & How to Avoid Them

### Trap 1: The `ConcurrentModificationException` Iteration Trap
```java
// ❌ WRONG: Modifying a collection inside a standard foreach loop:
List<String> tokens = new ArrayList<>(List.of("A", "DROP", "B"));
for (String token : tokens) {
    if ("DROP".equals(token)) {
        tokens.remove(token); // THROWS ConcurrentModificationException!
    }
}

// ✔ FIX 1: Modern Java removeIf (Fastest & cleanest):
tokens.removeIf("DROP"::equals);

// ✔ FIX 2: Explicit Iterator.remove() (If maintaining index / complex state):
Iterator<String> it = tokens.iterator();
while (it.hasNext()) {
    if ("DROP".equals(it.next())) {
        it.remove(); // Safe! Keeps internal modCount and expectedModCount in sync.
    }
}
```

---

### Trap 2: `Arrays.asList()` vs. `List.of()` vs. `new ArrayList<>()`
```java
// 1. Arrays.asList():
List<String> list1 = Arrays.asList("A", "B");
list1.set(0, "Z"); // Allowed! Mutates backing array.
list1.add("C");    // THROWS UnsupportedOperationException! Fixed size!

// 2. List.of() (Java 9+):
List<String> list2 = List.of("A", "B");
list2.set(0, "Z"); // THROWS UnsupportedOperationException! Strictly immutable!
list2.add("C");    // THROWS UnsupportedOperationException!

// 3. new ArrayList<>(List.of(...)) (Fully mutable):
List<String> list3 = new ArrayList<>(List.of("A", "B"));
list3.add("C");    // Works! Fully resizable and mutable.
```

---

### Trap 3: Java 21 Sequenced Collections (`getFirst`, `getLast`, `reversed`)
Before Java 21, getting the first or last element across `List`, `Deque`, and `LinkedHashSet` required inconsistent, clumsy calls:
- `list.get(0)` vs `list.get(list.size() - 1)`
- `deque.getFirst()` vs `deque.getLast()`
- `linkedSet.iterator().next()` vs (no way to get last without full traversal!)

**Java 21 JEP 431 introduced Sequenced Collections**, providing a uniform API:
```java
List<String> list = new ArrayList<>(List.of("alpha", "beta", "gamma"));
String first = list.getFirst();                       // "alpha"
String last  = list.getLast();                        // "gamma"
List<String> reversedView = list.reversed();          // Non-copy reverse view!
```

---

## 4. Runnable Verification Suite: Full Collections Refresher POC

The following standalone, self-contained Java program demonstrates all foundational collection patterns, Python-to-Java idioms, sorting techniques, safe removals, and Java 21 Sequenced Collections:

```java
package com.mastery.collections;

import java.util.*;

/**
 * Java Collections Framework (JCF) Comprehensive Refresher POC
 * 
 * Demonstrates:
 * 1. ArrayList vs ArrayDeque (Stack & Queue)
 * 2. TreeSet range queries & NavigableSet
 * 3. HashMap computeIfAbsent and merge (Python defaultdict / Counter equivalent)
 * 4. Multi-field sorting with Comparator
 * 5. Safe element removal (removeIf vs ConcurrentModificationException)
 * 6. Java 21 Sequenced Collections (getFirst, getLast, reversed)
 */
public class CollectionsRefresherMasteryPOC {

    record Transaction(String id, String currency, int amount) {}

    public static void main(String[] args) {
        System.out.println("=================================================================");
        System.out.println("  JAVA COLLECTIONS FRAMEWORK: PRACTICAL REFRESHER SUITE");
        System.out.println("=================================================================\n");

        demonstrateListAndSorting();
        demonstrateStackAndQueue();
        demonstrateMapIdioms();
        demonstrateNavigableTreeSet();
        demonstrateSafeRemoval();
        demonstrateJava21SequencedCollections();
    }

    private static void demonstrateListAndSorting() {
        System.out.println("[1] List Sorting & Comparator Chaining:");
        List<Transaction> txs = new ArrayList<>(List.of(
                new Transaction("TX-1", "USD", 500),
                new Transaction("TX-2", "EUR", 1200),
                new Transaction("TX-3", "USD", 1200),
                new Transaction("TX-4", "GBP", 300)
        ));

        // Sort by amount descending, then by currency ascending
        txs.sort(Comparator.comparingInt(Transaction::amount)
                .reversed()
                .thenComparing(Transaction::currency));

        for (Transaction tx : txs) {
            System.out.printf("    • %-6s | %3s | Amount: %,5d\n", tx.id(), tx.currency(), tx.amount());
        }
        System.out.println();
    }

    private static void demonstrateStackAndQueue() {
        System.out.println("[2] ArrayDeque: Fast Stack (LIFO) & Queue (FIFO):");

        // LIFO Stack (Replaces obsolete java.util.Stack)
        Deque<String> breadcrumbs = new ArrayDeque<>();
        breadcrumbs.push("Home");
        breadcrumbs.push("Checkout");
        breadcrumbs.push("Payment");
        System.out.println("    Stack Top: " + breadcrumbs.peek());
        System.out.println("    Popped: " + breadcrumbs.pop());
        System.out.println("    New Top: " + breadcrumbs.peek());

        // Min-Heap PriorityQueue
        PriorityQueue<Integer> minHeap = new PriorityQueue<>();
        minHeap.addAll(List.of(80, 20, 50, 10));
        System.out.print("    Min-Heap Polling Order: ");
        while (!minHeap.isEmpty()) {
            System.out.print(minHeap.poll() + " ");
        }
        System.out.println("\n");
    }

    private static void demonstrateMapIdioms() {
        System.out.println("[3] Map Modern Idioms (computeIfAbsent & merge):");

        // Idiom 1: Grouping by key (Python's defaultdict(list))
        Map<String, List<String>> cityUsers = new HashMap<>();
        cityUsers.computeIfAbsent("Pune", k -> new ArrayList<>()).add("Ankit");
        cityUsers.computeIfAbsent("Pune", k -> new ArrayList<>()).add("Rohan");
        cityUsers.computeIfAbsent("Bangalore", k -> new ArrayList<>()).add("Priya");
        System.out.println("    Grouped Users: " + cityUsers);

        // Idiom 2: Frequency counter (Python's collections.Counter)
        Map<String, Integer> wordFrequencies = new HashMap<>();
        List<String> words = List.of("kafka", "redis", "kafka", "jvm", "kafka", "jvm");
        for (String w : words) {
            wordFrequencies.merge(w, 1, Integer::sum);
        }
        System.out.println("    Frequencies: " + wordFrequencies + "\n");
    }

    private static void demonstrateNavigableTreeSet() {
        System.out.println("[4] TreeSet Navigable Range & Boundary Lookups:");
        TreeSet<Integer> latencyBuckets = new TreeSet<>(List.of(10, 25, 50, 100, 250, 500, 1000));

        int observedLatency = 78;
        Integer lowerBucket = latencyBuckets.floor(observedLatency);    // Largest <= 78
        Integer higherBucket = latencyBuckets.ceiling(observedLatency); // Smallest >= 78

        System.out.printf("    Observed: %d ms -> Floor Bucket: %d ms | Ceiling Bucket: %d ms\n",
                observedLatency, lowerBucket, higherBucket);
        System.out.println("    Sub-range (25ms to 250ms): " + latencyBuckets.subSet(25, true, 250, true) + "\n");
    }

    private static void demonstrateSafeRemoval() {
        System.out.println("[5] Safe Removal (removeIf avoiding ConcurrentModificationException):");
        List<String> services = new ArrayList<>(List.of("auth-svc", "legacy-svc", "payment-svc", "legacy-v1"));

        // Safely removes any element starting with "legacy"
        services.removeIf(name -> name.startsWith("legacy"));
        System.out.println("    Active Fleet after removeIf: " + services + "\n");
    }

    private static void demonstrateJava21SequencedCollections() {
        System.out.println("[6] Java 21 Sequenced Collections:");
        List<String> pipeline = new ArrayList<>(List.of("Ingest", "Transform", "Validate", "Persist"));

        System.out.println("    First Stage: " + pipeline.getFirst());
        System.out.println("    Last Stage : " + pipeline.getLast());
        System.out.println("    Reversed View: " + pipeline.reversed());
        System.out.println();
    }
}
```

---

## 5. Staff / Principal Engineer Interview Guide: Collections Strategy

### Scenario 1: Sizing `ArrayList` & `HashMap` to Prevent Microservice Latency Spikes
> **Interviewer**: *"In our high-throughput payment ingestion gateway, we create an `ArrayList` and a `HashMap` for every batch of 10,000 inbound records. We observe periodic CPU spikes and GC stalls during batch deserialization. How do you optimize this?"*

- **Principal Engineer Response**:
  1. **Root Cause**: *"Default constructor sizing. An `ArrayList` begins with an internal capacity of 10. Growing from 10 to 10,000 requires **18 array reallocation cycles** (`10 -> 15 -> 22 ... -> 10,000`). Each resize allocates a new underlying array and executes a memory copy via `System.arraycopy()`, generating massive young-generation garbage and CPU overhead."*
  2. **HashMap Resize Pitfall**: *"A default `new HashMap<>()` starts with capacity 16 and a 0.75 load factor (threshold = 12). Sizing to 10,000 triggers **10 rehashing and bucket doubling cycles**, where every single node's hash is re-masked and re-linked!"*
  3. **The Fix**:
     ```java
     // Pre-size ArrayList with exact capacity:
     List<Record> list = new ArrayList<>(10_000);

     // Pre-size HashMap: expectedSize / loadFactor + 1:
     // 10,000 / 0.75 + 1 ≈ 13,334 -> rounds to power of two: 16,384
     Map<String, Record> map = new HashMap<>(16_384);
     ```
     *"This guarantees **zero intermediate resize cycles**, cuts batch allocation latency by $>60\%$, and eliminates transient GC churn."*
