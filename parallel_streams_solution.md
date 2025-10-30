# Java Parallel Streams Exercise 2 - Complete Solution Guide

## Project Context

**Course:** Computación Paralela y Distribuida  
**Institution:** Universidad Nacional de Colombia  
**File to Modify:** `ejercicio_2/src/main/java/co/edu/unal/paralela/StudentAnalytics.java`

## Constraints

- ❌ Cannot modify public or protected method signatures
- ❌ Cannot delete existing methods
- ✅ Can add private helper methods if needed
- ✅ Must use parallel streams (`.parallelStream()`)
- ✅ Cannot use loops (`for`, `while`) in the parallel implementations

---

## Student Class API Reference

The `Student` class (located in `Student.java`) provides these exact methods:

```java
public String getFirstName()     // Returns student's first name (String)
public String getLastName()      // Returns student's last name (String)
public double getAge()           // Returns student's age (double, not int!)
public int getGrade()            // Returns student's grade (int, 0-100 scale)
public boolean checkIsCurrent()  // Returns true if student is currently enrolled
```

**Critical Implementation Notes:**
- ⚠️ The method is `checkIsCurrent()` NOT `isCurrent()` - using wrong method name will cause compilation errors
- ⚠️ Age is `double` type - must use `mapToDouble()` not `mapToInt()`
- A student is "enrolled/current" when `checkIsCurrent()` returns `true`
- A student is "inactive" when `checkIsCurrent()` returns `false`
- A student "failed" when **ALL THREE** conditions are met:
  - `checkIsCurrent()` returns `false` (not currently enrolled)
  - `getAge() > 20`
  - `getGrade() < 65`

---

## Task 1: averageAgeOfEnrolledStudentsParallelStream

### Imperative Logic to Convert

```java
// 1. Filter: Keep only currently enrolled students (checkIsCurrent() == true)
// 2. Map: Extract their ages
// 3. Reduce: Calculate average age
```

### Expected Behavior

- **Input:** Array of Student objects (`Student[] studentArray`)
- **Output:** `double` - Average age of enrolled students
- **Filter condition:** `student.checkIsCurrent()` must return `true`
- **Edge case:** Returns `0.0` when no enrolled students exist

### Exact Method Signature (DO NOT MODIFY)

```java
public double averageAgeOfEnrolledStudentsParallelStream(
        final Student[] studentArray)
```

**Parameters:**
- `studentArray` - Array of Student objects (can be very large: 2,000,000 students in tests)

### Parallel Stream Pattern

```java
public double averageAgeOfEnrolledStudentsParallelStream(
        final Student[] studentArray) {
    return Arrays.stream(studentArray)
        .parallel()
        .filter(Student::checkIsCurrent)
        .mapToDouble(Student::getAge)
        .average()
        .orElse(0.0);
}
```

### Key Points

1. Use `Arrays.stream(studentArray).parallel()` to create parallel stream
2. Filter enrolled students using `Student::checkIsCurrent`
3. Use `mapToDouble()` instead of `mapToInt()` since age is `double`
4. Use `.average()` terminal operation
5. Handle empty result with `.orElse(0.0)`

---

## Task 2: mostCommonFirstNameOfInactiveStudentsParallelStream

### Imperative Logic to Convert

```java
// 1. Filter: Keep only inactive students (checkIsCurrent() == false)
// 2. Map: Extract their first names
// 3. Group: Count frequency of each name
// 4. Reduce: Find name with maximum frequency
```

### Expected Behavior

- **Input:** Array of Student objects (`Student[] studentArray`)
- **Output:** `String` - Most common first name among inactive students
- **Filter condition:** `student.checkIsCurrent()` must return `false`
- **Edge case:** Returns `null` if no inactive students exist
- **Tie-breaking:** If multiple names have same max frequency, any one is acceptable

### Exact Method Signature (DO NOT MODIFY)

```java
public String mostCommonFirstNameOfInactiveStudentsParallelStream(
        final Student[] studentArray)
```

**Test Scenario:**
- With 1,400,000 inactive students from test data
- Possible names: "Sanjay", "Yunming", "John", "Vivek", "Shams", "Max"
- Must match imperative version's result exactly

### Parallel Stream Pattern

```java
public String mostCommonFirstNameOfInactiveStudentsParallelStream(
        final Student[] studentArray) {
    return Arrays.stream(studentArray)
        .parallel()
        .filter(student -> !student.checkIsCurrent())
        .map(Student::getFirstName)
        .collect(Collectors.groupingBy(
            Function.identity(),
            Collectors.counting()
        ))
        .entrySet().stream()
        .max(Map.Entry.comparingByValue())
        .map(Map.Entry::getKey)
        .orElse(null);
}
```

### Key Points

1. Filter **inactive** students: `student -> !student.checkIsCurrent()`
2. Extract first names using `Student::getFirstName`
3. Use `Collectors.groupingBy()` to create frequency map
4. `Function.identity()` keeps the name as the key
5. `Collectors.counting()` counts occurrences
6. Convert map entries to stream and find max by value
7. Extract the key (name) from the max entry
8. Return `null` if no inactive students exist

### Required Imports

```java
import java.util.stream.Collectors;
import java.util.function.Function;
import java.util.Arrays;
```

---

## Task 3: countNumberOfFailedStudentsOlderThan20ParallelStream

### Imperative Logic to Convert

```java
// 1. Filter: Keep only students who meet ALL conditions:
//    - NOT currently enrolled (checkIsCurrent() == false)
//    - Age > 20
//    - Grade < 65
// 2. Reduce: Count matching students
```

### Expected Behavior

- **Input:** Array of Student objects (`Student[] studentArray`)
- **Output:** `int` - Count of failed students older than 20
- **Definition of "failed student":**
  - ✅ `checkIsCurrent()` returns `false` (not currently enrolled)
  - ✅ `getAge() > 20` (strictly greater than, not >=)
  - ✅ `getGrade() < 65` (failing grade)
  - All three conditions must be true simultaneously (AND logic)

### Exact Method Signature (DO NOT MODIFY)

```java
public int countNumberOfFailedStudentsOlderThan20ParallelStream(
        final Student[] studentArray)
```

**Test Scenario:**
- Random student data with grades 1-100
- Ages between 0-100
- Must count only students meeting ALL three criteria
- Result must match imperative version exactly

### Parallel Stream Pattern

```java
public int countNumberOfFailedStudentsOlderThan20ParallelStream(
        final Student[] studentArray) {
    return (int) Arrays.stream(studentArray)
        .parallel()
        .filter(student -> !student.checkIsCurrent())
        .filter(student -> student.getAge() > 20)
        .filter(student -> student.getGrade() < 65)
        .count();
}
```

### Alternative (Single Filter)

```java
public int countNumberOfFailedStudentsOlderThan20ParallelStream(
        final Student[] studentArray) {
    return (int) Arrays.stream(studentArray)
        .parallel()
        .filter(student -> 
            !student.checkIsCurrent() && 
            student.getAge() > 20 && 
            student.getGrade() < 65
        )
        .count();
}
```

### Key Points

1. Use chained filters or single compound filter
2. `.count()` returns `long`, cast to `int` for return type
3. All three conditions must be true (AND logic)
4. Failed = not current + grade < 65

---

## Common Imports Needed

Add these to the top of `StudentAnalytics.java` (if not already present):

```java
package co.edu.unal.paralela;

import java.util.List;
import java.util.ArrayList;
import java.util.Map;
import java.util.HashMap;
import java.util.stream.Stream;
import java.util.Arrays;              // ADD THIS - for Arrays.stream()
import java.util.stream.Collectors;   // ADD THIS - for groupingBy(), counting()
import java.util.function.Function;   // ADD THIS - for Function.identity()
```

**Note:** The file already has some imports. Only add the missing ones (last three lines).

---

## Testing Your Solution

### Running Tests (Windows/VSCode Terminal)

```bash
# Navigate to project directory
cd ejercicio_2

# Run tests with Maven
mvn clean test

# Or run with checkstyle validation
mvn clean checkstyle:check test
```

### Expected Test Results

The tests verify:
1. **Correctness**: Parallel implementation produces same results as imperative
2. **Performance**: Parallel version runs at least 1.2x faster (tasks 1 & 3) or ncores * 0.5 faster (task 2)

### Test Data

- **Total students:** 2,000,000
- **Current/enrolled students:** 600,000 (30%)
- **Inactive students:** 1,400,000 (70%)
- **First names used:** {"Sanjay", "Yunming", "John", "Vivek", "Shams", "Max"}
- **Last names used:** {"Chatterjee", "Zhang", "Smith", "Sarkar", "Imam", "Grossman"}
- **Random seed:** 123 (deterministic test data)
- **Age range:** 0-100 (double values)
- **Grade range:** 1-100 (int values)

### Test Methods (from StudentAnalyticsTest.java)

**Task 1 Tests:**
```java
testAverageAgeOfEnrolledStudents()      // Correctness test (1 iteration)
testAverageAgeOfEnrolledStudentsPerf()  // Performance test (10 iterations)
// Performance requirement: speedup > 1.2x
```

**Task 2 Tests:**
```java
testMostCommonFirstNameOfInactiveStudents()      // Correctness test (1 iteration)
testMostCommonFirstNameOfInactiveStudentsPerf()  // Performance test (10 iterations)
// Performance requirement: speedup >= ncores * 0.5
```

**Task 3 Tests:**
```java
testCountNumberOfFailedStudentsOlderThan20()      // Correctness test (1 iteration)
testCountNumberOfFailedStudentsOlderThan20Perf()  // Performance test (10 iterations)
// Performance requirement: speedup > 1.2x
```

### What Tests Verify

1. **Correctness:** Parallel implementation produces **exact same result** as imperative version
   - Uses `assertEquals` for String/int comparisons
   - Uses `assertTrue` with error threshold (< 1E-5) for double comparisons

2. **Performance:** Parallel version achieves required speedup
   - Speedup = (Sequential Time) / (Parallel Time)
   - Sequential time: Time for REPEATS iterations of imperative method
   - Parallel time: Time for REPEATS iterations of parallel stream method

3. **Edge Cases:**
   - Empty arrays (no students)
   - No students matching filter criteria
   - All students matching filter criteria

---

## Quick Reference: Stream Operations

| Operation | Purpose | Example |
|-----------|---------|---------|
| `Arrays.stream(array).parallel()` | Create parallel stream from array | Required for all tasks |
| `.filter(predicate)` | Keep elements matching condition | `.filter(Student::checkIsCurrent)` |
| `.map(function)` | Transform elements | `.map(Student::getFirstName)` |
| `.mapToDouble(function)` | Transform to double stream | `.mapToDouble(Student::getAge)` |
| `.count()` | Count elements (returns long) | `.count()` |
| `.average()` | Calculate average (returns OptionalDouble) | `.average().orElse(0.0)` |
| `.collect(Collectors.groupingBy())` | Group and count | Frequency map creation |
| `.orElse(defaultValue)` | Handle Optional empty case | `.orElse(null)` or `.orElse(0.0)` |

---

## Implementation Checklist

### Task 1: averageAgeOfEnrolledStudentsParallelStream
- [ ] Create parallel stream from `studentArray`
- [ ] Filter by `checkIsCurrent()` returning true
- [ ] Map to double ages
- [ ] Calculate average
- [ ] Handle empty case with `orElse(0.0)`
- [ ] Return type is `double`

### Task 2: mostCommonFirstNameOfInactiveStudentsParallelStream
- [ ] Create parallel stream from `studentArray`
- [ ] Filter by `checkIsCurrent()` returning false
- [ ] Map to first names
- [ ] Group by name and count
- [ ] Find max entry by count
- [ ] Extract name from max entry
- [ ] Handle empty case with `orElse(null)`
- [ ] Return type is `String`

### Task 3: countNumberOfFailedStudentsOlderThan20ParallelStream
- [ ] Create parallel stream from `studentArray`
- [ ] Filter by `!checkIsCurrent()`
- [ ] Filter by `age > 20`
- [ ] Filter by `grade < 65`
- [ ] Count matching students
- [ ] Cast long to int
- [ ] Return type is `int`

---

## Code Style Requirements

Based on checkstyle.xml configuration:

1. Use 4-space indentation
2. No star imports
3. English comments only
4. Method names in camelCase
5. No tabs, only spaces
6. Proper whitespace around operators
7. Line length considerations

---

## Final Implementation Template

```java
package co.edu.unal.paralela;

import java.util.List;
import java.util.ArrayList;
import java.util.Map;
import java.util.HashMap;
import java.util.stream.Stream;
import java.util.Arrays;
import java.util.stream.Collectors;
import java.util.function.Function;

/**
 * A wrapper class for various analytics methods.
 */
public final class StudentAnalytics {
    
    // ========== EXISTING IMPERATIVE METHODS (DO NOT MODIFY) ==========
    
    /**
     * Sequentially computes the average age of all currently enrolled students
     * using loops.
     */
    public double averageAgeOfEnrolledStudentsImperative(
            final Student[] studentArray) {
        // ... existing implementation ...
    }
    
    /**
     * Sequentially computes the most common first name of all students who are
     * not currently enrolled using loops.
     */
    public String mostCommonFirstNameOfInactiveStudentsImperative(
            final Student[] studentArray) {
        // ... existing implementation ...
    }
    
    /**
     * Sequentially computes the number of students who have failed the course
     * who are older than 20 years old.
     */
    public int countNumberOfFailedStudentsOlderThan20Imperative(
            final Student[] studentArray) {
        // ... existing implementation ...
    }
    
    // ========== PARALLEL STREAM IMPLEMENTATIONS (TO COMPLETE) ==========
    
    /**
     * Computes the average age of all currently enrolled students using
     * parallel streams. Must reflect the functionality of
     * averageAgeOfEnrolledStudentsImperative. This method must NOT use loops.
     *
     * @param studentArray Student data for this class
     * @return Average age of enrolled students
     */
    public double averageAgeOfEnrolledStudentsParallelStream(
            final Student[] studentArray) {
        // TODO: Implement using parallel stream
        // 1. Create parallel stream from studentArray
        // 2. Filter students where checkIsCurrent() is true
        // 3. Map to their ages (use mapToDouble since age is double)
        // 4. Calculate average
        // 5. Return 0.0 if no students match
        throw new UnsupportedOperationException();
    }
    
    /**
     * Computes the most common first name of all students who are not
     * currently enrolled using parallel streams. Must reflect the functionality
     * of mostCommonFirstNameOfInactiveStudentsImperative. This method must NOT
     * use loops.
     *
     * @param studentArray Student data for this class
     * @return Most common first name of inactive students
     */
    public String mostCommonFirstNameOfInactiveStudentsParallelStream(
            final Student[] studentArray) {
        // TODO: Implement using parallel stream
        // 1. Create parallel stream from studentArray
        // 2. Filter students where checkIsCurrent() is false
        // 3. Map to their first names
        // 4. Group by name and count frequencies
        // 5. Find the name with maximum count
        // 6. Return null if no inactive students exist
        throw new UnsupportedOperationException();
    }
    
    /**
     * Computes the number of students who have failed the course and are older
     * than 20 years old using parallel streams. A failing grade is anything
     * below 65. A student has failed the course if they have a failing grade
     * and are not currently enrolled. Must reflect the functionality of
     * countNumberOfFailedStudentsOlderThan20Imperative. This method must NOT
     * use loops.
     *
     * @param studentArray Student data for this class
     * @return Number of failed grades from students older than 20 years old
     */
    public int countNumberOfFailedStudentsOlderThan20ParallelStream(
            final Student[] studentArray) {
        // TODO: Implement using parallel stream
        // 1. Create parallel stream from studentArray
        // 2. Filter students where checkIsCurrent() is false
        // 3. Filter students where age > 20
        // 4. Filter students where grade < 65
        // 5. Count matching students
        // 6. Cast long to int for return
        throw new UnsupportedOperationException();
    }
}
```

**Implementation Steps:**
1. Replace each `throw new UnsupportedOperationException();` line with actual implementation
2. Use the patterns provided in sections above
3. Do NOT modify method signatures or existing imperative methods
4. Ensure all code is in English (comments and variable names)

---

## Common Pitfalls to Avoid

1. ❌ **Wrong method name:** Don't use `student.isCurrent()` - the method is `checkIsCurrent()`
   ```java
   // WRONG
   .filter(Student::isCurrent)
   
   // CORRECT
   .filter(Student::checkIsCurrent)
   ```

2. ❌ **Wrong stream conversion:** Don't use `mapToInt()` for age - age is `double` type
   ```java
   // WRONG - will cause compilation error
   .mapToInt(Student::getAge)
   
   // CORRECT
   .mapToDouble(Student::getAge)
   ```

3. ❌ **Forgot to cast count():** The `.count()` method returns `long`, must cast to `int`
   ```java
   // WRONG - type mismatch error
   return Arrays.stream(studentArray).parallel().filter(...).count();
   
   // CORRECT
   return (int) Arrays.stream(studentArray).parallel().filter(...).count();
   ```

4. ❌ **Wrong filter logic:** In tasks 2 and 3, filter for NOT current (`!checkIsCurrent()`)
   ```java
   // WRONG - for task 2 (inactive students)
   .filter(Student::checkIsCurrent)
   
   // CORRECT - for task 2 (inactive students)
   .filter(student -> !student.checkIsCurrent())
   ```

5. ❌ **Missing .parallel():** Don't forget to make the stream parallel
   ```java
   // WRONG - will work but won't be parallel
   Arrays.stream(studentArray).filter(...)
   
   // CORRECT - creates parallel stream
   Arrays.stream(studentArray).parallel().filter(...)
   ```

6. ❌ **Using loops:** Don't use `for`, `while`, or `forEach` loops in parallel implementations
   ```java
   // WRONG - defeats the purpose of streams
   for (Student s : studentArray) { ... }
   
   // CORRECT - use stream operations
   Arrays.stream(studentArray).parallel()...
   ```

7. ❌ **Wrong comparison operators:** Age must be strictly greater than 20 (not >=)
   ```java
   // WRONG
   .filter(student -> student.getAge() >= 20)
   
   // CORRECT  
   .filter(student -> student.getAge() > 20)
   ```

8. ❌ **Missing imports:** Forgetting required imports will cause compilation errors
   ```java
   // Must have these imports:
   import java.util.Arrays;
   import java.util.stream.Collectors;
   import java.util.function.Function;
   ```

9. ❌ **Incorrect Optional handling:** Must use `orElse()` for empty results
   ```java
   // WRONG - might throw NoSuchElementException
   .average().getAsDouble()
   
   // CORRECT
   .average().orElse(0.0)
   ```

10. ❌ **Modifying method signatures:** Cannot change public method declarations
    ```java
    // WRONG - changed signature
    public double averageAgeOfEnrolledStudentsParallelStream(List<Student> students)
    
    // CORRECT - must keep exact signature
    public double averageAgeOfEnrolledStudentsParallelStream(final Student[] studentArray)
    ```

---

## Success Criteria

Your implementation is complete when:

1. ✅ All three methods throw no `UnsupportedOperationException`
2. ✅ All correctness tests pass
3. ✅ All performance tests pass (speedup requirements met)
4. ✅ Checkstyle validation passes
5. ✅ No loops used in parallel implementations
6. ✅ Code compiles without errors
7. ✅ All methods use `.parallel()` streams

---

## References

- [Java 8 Stream API](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html)
- [Collectors Documentation](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html)
- [Arrays.stream Documentation](https://docs.oracle.com/javase/8/docs/api/java/util/Arrays.html#stream-T:A-)
