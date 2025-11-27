# Java Parallel Streams - Exercise 2 Reference Guide

## Assignment Overview

**Course:** Computación Paralela y Distribuida  
**Institution:** Universidad Nacional de Colombia

### Tasks to Implement

You must transform three imperative methods into their parallel stream equivalents within `StudentAnalytics.java`:

1. `averageAgeOfEnrolledStudentsParallelStream()`
2. `mostCommonFirstNameOfInactiveStudentsParallelStream()`
3. `countNumberOfFailedStudentsOlderThan20ParallelStream()`

**Constraints:**
- Cannot modify public or protected method signatures
- Cannot delete existing methods
- Can add helper methods as needed

---

## Key Concepts

### Creating Parallel Streams

Any sequential stream can be transformed into a parallel stream:

```java
// From a Collection
List<Student> students = getStudents();
Stream<Student> parallelStream = students.parallelStream();

// Or convert existing stream to parallel
Stream<Student> parallelStream = students.stream().parallel();
```

### The Three Pillars: Filter, Map, Reduce

#### 1. Filter - Selecting Elements

```java
// Keep only students matching a condition
students.parallelStream()
    .filter(student -> student.getAge() > 20)
    .filter(student -> student.isEnrolled())
```

#### 2. Map - Transforming Elements

```java
// Extract specific attributes
students.parallelStream()
    .map(Student::getAge)           // Get ages
    .map(Student::getFirstName)     // Get names
```

#### 3. Reduce - Computing Results

```java
// Average (terminal operation)
double avgAge = students.parallelStream()
    .mapToInt(Student::getAge)
    .average()
    .orElse(0.0);

// Count (terminal operation)
long count = students.parallelStream()
    .filter(condition)
    .count();
```

---

## Patterns for Your Implementation

### Pattern 1: Computing Average

**Imperative Pattern:**
```java
double sum = 0;
int count = 0;
for (Student s : students) {
    if (s.isEnrolled()) {
        sum += s.getAge();
        count++;
    }
}
return count > 0 ? sum / count : 0.0;
```

**Parallel Stream Pattern:**
```java
return students.parallelStream()
    .filter(Student::isEnrolled)
    .mapToInt(Student::getAge)
    .average()
    .orElse(0.0);
```

### Pattern 2: Finding Most Common Element

**Imperative Pattern:**
```java
Map<String, Integer> frequency = new HashMap<>();
for (Student s : students) {
    if (!s.isActive()) {
        String name = s.getFirstName();
        frequency.put(name, frequency.getOrDefault(name, 0) + 1);
    }
}
// Find max frequency entry
```

**Parallel Stream Pattern (Basic):**
```java
return students.parallelStream()
    .filter(student -> !student.isActive())
    .map(Student::getFirstName)
    .collect(Collectors.groupingBy(
        Function.identity(),
        Collectors.counting()
    ))
    .entrySet().stream()
    .max(Map.Entry.comparingByValue())
    .map(Map.Entry::getKey)
    .orElse(null);
```

**Optimized Pattern (Recommended):**
```java
// Avoid unnecessary mapping at the end by extracting directly
Map.Entry<String, Long> maxEntry = students.parallelStream()
    .filter(student -> !student.isActive())
    .map(Student::getFirstName)  // This map IS necessary - transforms Student to String
    .collect(Collectors.groupingBy(
        Function.identity(),
        Collectors.counting()
    ))
    .entrySet().stream()
    .max(Map.Entry.comparingByValue())
    .orElse(null);

return maxEntry != null ? maxEntry.getKey() : null;
```

**When mapping is necessary vs. avoidable:**
- `.map(Student::getFirstName)` - **NECESSARY**: Must transform Student objects into Strings for grouping
- `.map(Map.Entry::getKey)` - **AVOIDABLE**: Can extract key directly from Optional result
- Saves one stream operation and improves readability

### Pattern 3: Counting with Conditions

**Imperative Pattern:**
```java
int count = 0;
for (Student s : students) {
    if (s.isFailed() && s.getAge() > 20) {
        count++;
    }
}
return count;
```

**Parallel Stream Pattern (Basic):**
```java
return students.parallelStream()
    .filter(Student::isFailed)
    .filter(student -> student.getAge() > 20)
    .count();
```

**Optimized Pattern (Recommended):**
```java
// Combine multiple filters into one for better performance
return students.parallelStream()
    .filter(student -> student.isFailed() && student.getAge() > 20)
    .count();
```

**Why this optimization works:**
- Reduces stream pipeline overhead (single filter vs multiple)
- Java's `&&` short-circuits: stops evaluating when first condition fails
- Better CPU cache utilization with fewer intermediate operations
- No mapping needed - we're only counting, not transforming elements

---

## When to Avoid Mapping Patterns

### General Rule: Only Map When You Need to Transform

**Mapping adds overhead.** Only use `.map()` when you need to transform elements into a different type or extract specific fields for downstream operations.

### ❌ Avoid Mapping When:

#### 1. Counting Elements (No Transformation Needed)
```java
// ❌ BAD: Unnecessary mapping
long count = students.parallelStream()
    .filter(s -> s.getAge() > 20)
    .map(Student::getAge)  // Unnecessary! We're just counting
    .count();

// ✅ GOOD: Direct counting
long count = students.parallelStream()
    .filter(s -> s.getAge() > 20)
    .count();
```

#### 2. Extracting from Optional Results
```java
// ❌ BAD: Extra map operation
String name = students.parallelStream()
    .filter(s -> s.getGrade() > 90)
    .findFirst()
    .map(Student::getName)
    .orElse("None");

// ✅ GOOD: Extract after getting Optional
Optional<Student> topStudent = students.parallelStream()
    .filter(s -> s.getGrade() > 90)
    .findFirst();
String name = topStudent.isPresent() ? topStudent.get().getName() : "None";
```

#### 3. Multiple Filters Can Be Combined
```java
// ❌ BAD: Multiple filter operations
count = students.parallelStream()
    .filter(s -> !s.checkIsCurrent())
    .filter(s -> s.getAge() > 20)
    .filter(s -> s.getGrade() < 65)
    .count();

// ✅ GOOD: Single combined filter
count = students.parallelStream()
    .filter(s -> !s.checkIsCurrent() && s.getAge() > 20 && s.getGrade() < 65)
    .count();
```

### ✅ Mapping IS Necessary When:

#### 1. Computing Numeric Statistics
```java
// NECESSARY: Need to extract numeric values for average
double avgAge = students.parallelStream()
    .filter(Student::checkIsCurrent)
    .mapToDouble(Student::getAge)  // Required for numeric operations
    .average()
    .orElse(0.0);
```

#### 2. Grouping or Collecting by Attribute
```java
// NECESSARY: Need to extract names for grouping
Map<String, Long> nameFrequency = students.parallelStream()
    .filter(s -> !s.checkIsCurrent())
    .map(Student::getFirstName)  // Required to group by name
    .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));
```

#### 3. Transforming to Different Type
```java
// NECESSARY: Converting Student objects to String list
List<String> names = students.parallelStream()
    .map(Student::getFullName)  // Required for type transformation
    .collect(Collectors.toList());
```

### Performance Impact

| Pattern | Operations | Overhead | When to Use |
|---------|-----------|----------|-------------|
| No mapping | Filter → Count | Minimal | Counting with conditions |
| Single map | Filter → Map → Collect | Moderate | Extracting/grouping attributes |
| Multiple maps | Filter → Map → Map → Result | High | Avoid when possible |

**Key Insight:** Each intermediate operation (filter, map) adds overhead. Minimize operations while maintaining clarity.

---

## Common Operations Reference

### Filtering Operations

```java
// Single condition
.filter(Student::isEnrolled)

// Multiple conditions (chained)
.filter(student -> student.getAge() > 20)
.filter(Student::isFailed)

// Complex condition
.filter(student -> student.isActive() && student.getGrade() >= 3.0)
```

### Mapping Operations

```java
// Method reference
.map(Student::getFirstName)
.map(Student::getAge)

// Lambda expression
.map(student -> student.getFullName())

// Numeric mapping for calculations
.mapToInt(Student::getAge)
.mapToDouble(Student::getGrade)
```

### Terminal Operations

```java
// Counting
.count()

// Average (requires numeric stream)
.mapToInt(Student::getAge).average().orElse(0.0)

// Collecting to collection
.collect(Collectors.toList())

// Grouping
.collect(Collectors.groupingBy(Student::getMajor))

// Frequency count
.collect(Collectors.groupingBy(Function.identity(), Collectors.counting()))
```

---

## Working with Optional

Many stream operations return `Optional<T>`:

```java
// Safe way to handle optional results
OptionalDouble avgOptional = students.parallelStream()
    .mapToInt(Student::getAge)
    .average();

double avg = avgOptional.orElse(0.0);  // Provide default value

// Or for Objects
Optional<String> nameOptional = students.parallelStream()
    .findFirst()
    .map(Student::getFirstName);

String name = nameOptional.orElse("Unknown");
```

---

## Quick Reference Card

| Operation | Purpose | Example |
|-----------|---------|---------|
| `.parallelStream()` | Create parallel stream | `list.parallelStream()` |
| `.filter(predicate)` | Keep matching elements | `.filter(s -> s.getAge() > 18)` |
| `.map(function)` | Transform elements | `.map(Student::getName)` |
| `.mapToInt(function)` | Convert to int stream | `.mapToInt(Student::getAge)` |
| `.count()` | Count elements | `.filter(...).count()` |
| `.average()` | Calculate average | `.mapToInt(...).average()` |
| `.collect()` | Gather to collection | `.collect(Collectors.toList())` |
| `.orElse(value)` | Handle Optional | `.average().orElse(0.0)` |

---

## Important Notes

1. **Parallel Streams Benefits:** Automatically splits work across multiple threads for better performance on large datasets

2. **Method References:** Use `Student::getAge` instead of `s -> s.getAge()` when possible (cleaner code)

3. **Chaining:** All filter, map operations can be chained before the terminal operation

4. **Terminal Operations:** These complete the stream pipeline: `count()`, `average()`, `collect()`, `findFirst()`, etc.

---

## References

- [Java 8 Stream API Documentation](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html)
- [Collectors Documentation](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html)
- [Optional Documentation](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html)

---

## Optimizations Applied in This Exercise

### Summary of Implementation Choices

#### Method 1: `averageAgeOfEnrolledStudentsParallelStream`
```java
return Arrays.stream(studentArray)
    .parallel()
    .filter(Student::checkIsCurrent)
    .mapToDouble(Student::getAge)  // NECESSARY: Required for numeric operations
    .average()
    .orElse(0.0);
```
- **mapToDouble IS required** - Can't compute average without extracting numeric values
- Uses specialized `DoubleStream` for better performance than generic `Stream<Double>`

#### Method 2: `mostCommonFirstNameOfInactiveStudentsParallelStream`
```java
Map.Entry<String, Long> maxEntry = Arrays.stream(studentArray)
    .parallel()
    .filter(student -> !student.checkIsCurrent())
    .map(Student::getFirstName)  // NECESSARY: Must extract names for grouping
    .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()))
    .entrySet().stream()
    .max(Map.Entry.comparingByValue())
    .orElse(null);

return maxEntry != null ? maxEntry.getKey() : null;
```
- **First map IS required** - Must transform Student → String for grouping
- **Final map AVOIDED** - Extracts key directly from entry instead of chaining `.map(Map.Entry::getKey)`
- Saves one stream operation

#### Method 3: `countNumberOfFailedStudentsOlderThan20ParallelStream`
```java
return (int) Arrays.stream(studentArray)
    .parallel()
    .filter(student -> !student.checkIsCurrent() && student.getAge() > 20 && student.getGrade() < 65)
    .count();
```
- **No mapping needed** - Only counting, no transformation required
- **Single combined filter** - More efficient than three separate filters
- Leverages short-circuit evaluation for best performance

### Why These Optimizations Matter

1. **Reduced Memory Allocation**: Fewer intermediate objects created
2. **Better Cache Performance**: Less data movement between operations
3. **Lower Synchronization Overhead**: Fewer pipeline stages in parallel execution
4. **Short-Circuit Evaluation**: Combined conditions stop early when false

### Performance Results

With optimizations, you should see:
- Method 1 & 3: **1.2x+ speedup** minimum
- Method 2: **0.5 × CPU cores speedup** (more complex due to grouping overhead)

---

## Testing Your Implementation

Run the provided test cases to verify your parallel stream implementations produce the same results as the imperative versions:

```bash
# Run all tests
cd ejercicio_2
mvn test

# Run specific test
mvn test -Dtest=StudentAnalyticsTest#testAverageAgeOfEnrolledStudents
```

The tests will show:
- ✓ Correctness: Results match imperative implementations
- ✓ Performance: Speedup meets minimum requirements

Make sure your parallel implementations are deterministic and thread-safe!