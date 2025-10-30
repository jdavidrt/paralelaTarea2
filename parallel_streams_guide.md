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

**Parallel Stream Pattern:**
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

**Parallel Stream Pattern:**
```java
return students.parallelStream()
    .filter(Student::isFailed)
    .filter(student -> student.getAge() > 20)
    .count();
```

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

## Testing Your Implementation

Run the provided test cases to verify your parallel stream implementations produce the same results as the imperative versions:

```bash
# In VSCode terminal (Windows)
javac StudentAnalytics.java
java StudentAnalytics
```

Make sure your parallel implementations are deterministic and thread-safe!