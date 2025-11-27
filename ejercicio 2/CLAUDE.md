# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Java 8 parallel programming assignment (Ejercicio 2) for Universidad Nacional de Colombia's Computación Paralela y Distribuida course. The goal is to convert imperative loop-based student analytics methods into parallel stream implementations.

## Project Structure

```
ejercicio_2/
├── pom.xml                              # Maven build configuration
├── src/
│   ├── main/
│   │   ├── java/co/edu/unal/paralela/
│   │   │   ├── Student.java             # Student data model (immutable)
│   │   │   ├── StudentAnalytics.java    # Main implementation file (3 methods to complete)
│   │   │   └── package-info.java
│   │   └── resources/
│   │       └── checkstyle.xml           # Code style rules
│   └── test/
│       └── java/co/edu/unal/paralela/
│           └── StudentAnalyticsTest.java # Correctness and performance tests
├── pom.xml
└── *.jar files                          # Dependencies (JUnit, pcdp-core, hamcrest)
```

## Build and Test Commands

### Build the project
```bash
cd ejercicio_2
mvn clean compile
```

### Run tests
```bash
mvn test
```

### Run checkstyle validation
```bash
mvn validate
```

### Run specific test
```bash
mvn test -Dtest=StudentAnalyticsTest#testAverageAgeOfEnrolledStudents
```

## Implementation Requirements

### Tasks to Complete in StudentAnalytics.java

Three methods need parallel stream implementations:

1. **`averageAgeOfEnrolledStudentsParallelStream()`** (line 46-49)
   - Compute average age of enrolled students (where `checkIsCurrent() == true`)
   - Must match behavior of `averageAgeOfEnrolledStudentsImperative()`

2. **`mostCommonFirstNameOfInactiveStudentsParallelStream()`** (line 99-102)
   - Find most common first name among inactive students (`checkIsCurrent() == false`)
   - Must match behavior of `mostCommonFirstNameOfInactiveStudentsImperative()`

3. **`countNumberOfFailedStudentsOlderThan20ParallelStream()`** (line 134-137)
   - Count students who: are not current, are older than 20, and have grade < 65
   - Must match behavior of `countNumberOfFailedStudentsOlderThan20Imperative()`

### Constraints

- Cannot modify public/protected method signatures
- Cannot delete existing methods
- NO loops allowed in parallel stream implementations
- Can add private helper methods if needed
- Must use Java 8 parallel streams (`parallelStream()` or `.parallel()`)

## Key Domain Model

### Student Class
- Immutable data class with fields: `firstName`, `lastName`, `age`, `grade`, `isCurrent`
- Key methods:
  - `getFirstName()`, `getLastName()`, `getAge()`, `getGrade()`
  - `checkIsCurrent()` - returns `true` if student is enrolled, `false` if inactive
- Failing grade: < 65

## Testing Strategy

Tests verify both **correctness** and **performance**:

- Correctness tests compare parallel implementations against imperative versions
- Performance tests expect at least 1.2x speedup (some expect 0.5 * nCores speedup)
- Test dataset: 2 million students (600k current, 1.4M inactive)

## Development Workflow

1. Read the imperative implementation to understand the logic
2. Identify the filter/map/reduce operations needed
3. Convert to parallel stream pipeline
4. Run correctness test first: `mvn test -Dtest=StudentAnalyticsTest#test<MethodName>`
5. Run performance test: `mvn test -Dtest=StudentAnalyticsTest#test<MethodName>Perf`
6. All tests must pass before submission

## Common Parallel Stream Patterns

Reference [parallel_streams_guide.md](parallel_streams_guide.md) for detailed patterns, including:

- Creating parallel streams from arrays: `Arrays.stream(array).parallel()`
- Filter-map-reduce pipelines
- Collectors for grouping and counting
- Handling `Optional` results with `.orElse()`
