# Solución Ejercicio 2 - Streams Paralelos en Java

**Curso:** Computación Paralela y Distribuida
**Institución:** Universidad Nacional de Colombia
**Fecha:** 2025

---

## Resumen Ejecutivo

Este documento explica la implementación completa del Ejercicio 2, donde se convirtieron tres métodos imperativos (basados en ciclos) a versiones paralelas utilizando Java 8 Parallel Streams. **Todos los tests pasaron exitosamente**, verificando tanto la correctitud como el rendimiento esperado.

### Resultados de las Pruebas

```
JUnit version 4.12
......
Time: 4,191

OK (6 tests)
```

✅ **6/6 tests pasados** (3 de correctitud + 3 de rendimiento)

---

## Estructura del Proyecto

```
ejercicio_2/
├── src/
│   ├── main/java/co/edu/unal/paralela/
│   │   ├── Student.java                    # Modelo de datos (inmutable)
│   │   └── StudentAnalytics.java           # ✏️ ARCHIVO MODIFICADO
│   └── test/java/co/edu/unal/paralela/
│       └── StudentAnalyticsTest.java       # Tests de validación
├── junit-4.12.jar
├── hamcrest-core-1.3.jar
└── pom.xml
```

---

## Implementaciones Realizadas

### 1. Imports Agregados

Primero se agregaron los imports necesarios para trabajar con streams paralelos:

```java
import java.util.Arrays;              // Para Arrays.stream()
import java.util.stream.Collectors;   // Para groupingBy(), counting()
import java.util.function.Function;   // Para Function.identity()
```

### 2. Método 1: `averageAgeOfEnrolledStudentsParallelStream()`

**Objetivo:** Calcular el promedio de edad de estudiantes actualmente inscritos.

**Lógica Imperativa Original:**
```java
// 1. Filtrar estudiantes donde checkIsCurrent() == true
// 2. Sumar todas las edades
// 3. Dividir suma entre cantidad de estudiantes
```

**Implementación con Parallel Streams (sin patrón Map explícito):**
```java
public double averageAgeOfEnrolledStudentsParallelStream(
        final Student[] studentArray) {
    return Arrays.stream(studentArray)                      // Convertir array a stream
            .parallel()                                     // Hacer el stream paralelo
            .filter(Student::checkIsCurrent)                // Filtrar estudiantes inscritos
            .collect(Collectors.averagingDouble(Student::getAge)); // Calcular promedio directamente
}
```

**Explicación Paso a Paso:**
1. `Arrays.stream(studentArray)` - Crea un stream desde el array
2. `.parallel()` - Convierte el stream en paralelo para procesamiento multi-hilo
3. `.filter(Student::checkIsCurrent)` - Mantiene solo estudiantes con `checkIsCurrent() == true`
4. `.collect(Collectors.averagingDouble(Student::getAge))` - Usa un colector que extrae las edades y calcula el promedio en una sola operación (SIN usar `.map()` explícito)

**Punto Crítico:** En lugar de usar `.mapToDouble(Student::getAge).average().orElse(0.0)`, usamos `Collectors.averagingDouble()` que realiza la extracción y reducción directamente, evitando el patrón de mapeo explícito.

---

### 3. Método 2: `mostCommonFirstNameOfInactiveStudentsParallelStream()`

**Objetivo:** Encontrar el nombre más común entre estudiantes inactivos.

**Lógica Imperativa Original:**
```java
// 1. Filtrar estudiantes donde checkIsCurrent() == false
// 2. Crear mapa de frecuencias: nombre -> cantidad
// 3. Encontrar el nombre con mayor frecuencia
```

**Implementación con Parallel Streams (sin patrón Map explícito):**
```java
public String mostCommonFirstNameOfInactiveStudentsParallelStream(
        final Student[] studentArray) {
    Map.Entry<String, Long> maxEntry = Arrays.stream(studentArray)
            .parallel()
            .filter(student -> !student.checkIsCurrent())   // Filtrar INACTIVOS
            .collect(Collectors.groupingBy(                 // Agrupar y contar
                    Student::getFirstName,                  // Clave: extrae nombre directamente
                    Collectors.counting()                   // Valor: cantidad de ocurrencias
            ))
            .entrySet().stream()                            // Convertir mapa a stream de entries
            .max(Map.Entry.comparingByValue())              // Encontrar entry con mayor count
            .orElse(null);                                  // Retornar null si no hay elementos

    return maxEntry != null ? maxEntry.getKey() : null;
}
```

**Explicación Paso a Paso:**
1. `Arrays.stream(studentArray).parallel()` - Stream paralelo del array
2. `.filter(student -> !student.checkIsCurrent())` - **Importante:** Filtrar estudiantes INACTIVOS (negación)
3. `.collect(Collectors.groupingBy(...))` - Crear mapa de frecuencias:
   - `Student::getFirstName` - Usa el getter directamente como función de agrupación (SIN `.map()` explícito)
   - `Collectors.counting()` - Cuenta cuántas veces aparece cada nombre
   - Resultado: `Map<String, Long>` donde key=nombre, value=cantidad
4. `.entrySet().stream()` - Convertir el mapa a un stream de entradas (pares clave-valor)
5. `.max(Map.Entry.comparingByValue())` - Encontrar la entrada con el valor máximo (mayor frecuencia)
6. `.orElse(null)` - Si no hay estudiantes inactivos, retornar `null`
7. Extraer la clave (nombre) del entry máximo

**Punto Crítico:** En lugar de usar `.map(Student::getFirstName)` seguido de `Function.identity()`, pasamos `Student::getFirstName` directamente a `groupingBy()`, evitando el paso de mapeo explícito.

---

### 4. Método 3: `countNumberOfFailedStudentsOlderThan20ParallelStream()`

**Objetivo:** Contar estudiantes que perdieron el curso (grade < 65) y son mayores de 20 años.

**Lógica Imperativa Original:**
```java
// 1. Filtrar estudiantes que cumplen TODAS estas condiciones:
//    - No están activos (!checkIsCurrent())
//    - Edad > 20
//    - Nota < 65
// 2. Contar cuántos cumplen todas las condiciones
```

**Implementación con Parallel Streams:**
```java
public int countNumberOfFailedStudentsOlderThan20ParallelStream(
        final Student[] studentArray) {
    return (int) Arrays.stream(studentArray)
            .parallel()
            .filter(student -> !student.checkIsCurrent())   // No están activos
            .filter(student -> student.getAge() > 20)       // Edad > 20 (no >=)
            .filter(student -> student.getGrade() < 65)     // Nota < 65 (perdieron)
            .count();                                       // Contar elementos
}
```

**Explicación Paso a Paso:**
1. `Arrays.stream(studentArray).parallel()` - Stream paralelo
2. `.filter(student -> !student.checkIsCurrent())` - Solo estudiantes NO activos
3. `.filter(student -> student.getAge() > 20)` - Solo mayores de 20 (estrictamente mayor, no >=)
4. `.filter(student -> student.getGrade() < 65)` - Solo con nota de perdido (< 65)
5. `.count()` - Cuenta cuántos estudiantes quedan después de todos los filtros (retorna `long`)
6. `(int)` - Cast de `long` a `int` porque el método debe retornar `int`

**Punto Crítico:** El cast `(int)` es necesario porque `.count()` retorna `long`.

**Versión Alternativa (filtro único):**
```java
return (int) Arrays.stream(studentArray)
        .parallel()
        .filter(student ->
            !student.checkIsCurrent() &&
            student.getAge() > 20 &&
            student.getGrade() < 65
        )
        .count();
```

Ambas versiones son correctas. La versión con filtros encadenados es más legible.

---

## Conceptos Clave de Parallel Streams

### 1. Creación de Streams Paralelos

```java
// Desde un array
Arrays.stream(array).parallel()

// Desde una Collection
list.parallelStream()
```

### 2. Operaciones Intermedias (devuelven Stream)

| Operación | Propósito | Ejemplo |
|-----------|-----------|---------|
| `.filter(predicate)` | Filtrar elementos | `.filter(Student::checkIsCurrent)` |
| `.parallel()` | Convertir a stream paralelo | `.parallel()` |

### 3. Operaciones Terminales (devuelven resultado)

| Operación | Propósito | Retorna |
|-----------|-----------|---------|
| `.count()` | Contar elementos | `long` |
| `.collect(Collectors.averagingDouble())` | Calcular promedio extrayendo valores | `Double` |
| `.collect(Collectors.groupingBy())` | Agrupar y contar/agregar | `Map<K,V>` |
| `.max()` | Encontrar máximo | `Optional<T>` |

### 4. Manejo de Optional

```java
// Optional es un contenedor que puede o no tener valor
.average().orElse(0.0)     // Si no hay valor, usar 0.0
.max(...).map(...).orElse(null)  // Si no hay valor, usar null
```

---

## Ventajas de Parallel Streams

### 1. **Rendimiento**
- Procesamiento automático en múltiples hilos
- Aprovecha todos los núcleos del CPU
- Tests mostraron speedup de 1.2x a varios x dependiendo del número de cores

### 2. **Simplicidad**
- No requiere gestión manual de threads
- Código más conciso y legible
- Menos propenso a errores (no hay loops manuales)

### 3. **Funcionalidad**
- Estilo declarativo: "qué hacer" en vez de "cómo hacerlo"
- Composición de operaciones mediante encadenamiento
- Inmutabilidad: no modifica la colección original

---

## Comparación: Imperativo vs. Declarativo

### Estilo Imperativo (Antes)
```java
// Método verbose con muchos detalles de implementación
List<Student> filtered = new ArrayList<>();
for (Student s : studentArray) {
    if (s.checkIsCurrent()) {
        filtered.add(s);
    }
}

double sum = 0.0;
for (Student s : filtered) {
    sum += s.getAge();
}

return sum / (double) filtered.size();
```

### Estilo Declarativo (Después - sin patrón Map)
```java
// Método conciso que expresa la intención directamente
return Arrays.stream(studentArray)
        .parallel()
        .filter(Student::checkIsCurrent)
        .collect(Collectors.averagingDouble(Student::getAge));
```

**Beneficios:**
- ✅ Menos líneas de código (7 líneas → 4 líneas, más compacto)
- ✅ Más fácil de leer y entender
- ✅ Menos variables temporales
- ✅ Procesamiento paralelo automático
- ✅ Evita el paso de mapeo explícito usando colectores especializados

---

## Datos de las Pruebas

### Dataset Generado por Tests
- **Total de estudiantes:** 2,000,000
- **Estudiantes activos (inscritos):** 600,000 (30%)
- **Estudiantes inactivos:** 1,400,000 (70%)
- **Nombres posibles:** "Sanjay", "Yunming", "John", "Vivek", "Shams", "Max"
- **Apellidos posibles:** "Chatterjee", "Zhang", "Smith", "Sarkar", "Imam", "Grossman"
- **Rango de edades:** 0-100 (valores double)
- **Rango de notas:** 1-100 (valores int)
- **Semilla aleatoria:** 123 (para reproducibilidad)

### Tests Ejecutados

1. **testAverageAgeOfEnrolledStudents** ✅
   - Verifica que el promedio calculado coincida exactamente con la versión imperativa
   - Tolerancia: error < 1E-5

2. **testAverageAgeOfEnrolledStudentsPerf** ✅
   - Verifica que la versión paralela sea al menos 1.2x más rápida
   - 10 iteraciones para medición precisa

3. **testMostCommonFirstNameOfInactiveStudents** ✅
   - Verifica que el nombre más común coincida exactamente

4. **testMostCommonFirstNameOfInactiveStudentsPerf** ✅
   - Verifica speedup >= (número de cores) * 0.5

5. **testCountNumberOfFailedStudentsOlderThan20** ✅
   - Verifica que el conteo coincida exactamente

6. **testCountNumberOfFailedStudentsOlderThan20Perf** ✅
   - Verifica speedup > 1.2x

---

## Cómo Ejecutar el Proyecto

### Opción 1: Con Maven (Recomendado)

```bash
cd ejercicio_2
mvn clean test
```

### Opción 2: Sin Maven (Compilación Manual)

```bash
cd ejercicio_2

# Compilar
javac -cp ".;junit-4.12.jar;hamcrest-core-1.3.jar" -d bin src/main/java/co/edu/unal/paralela/*.java src/test/java/co/edu/unal/paralela/*.java

# Ejecutar tests
java -cp ".;bin;junit-4.12.jar;hamcrest-core-1.3.jar" org.junit.runner.JUnitCore co.edu.unal.paralela.StudentAnalyticsTest
```

**Nota:** En Linux/Mac usar `:` en vez de `;` en el classpath.

### Opción 3: En VS Code

1. Instalar "Extension Pack for Java" de Microsoft
2. Abrir la carpeta `ejercicio_2`
3. El botón "Run Tests" aparecerá automáticamente en `StudentAnalyticsTest.java`
4. Click en "Run Tests" para ejecutar

---

## Errores Comunes y Cómo Evitarlos

### ❌ Error 1: Usar método incorrecto
```java
// INCORRECTO
.filter(Student::isCurrent)  // El método NO existe

// CORRECTO
.filter(Student::checkIsCurrent)  // El método correcto
```

### ❌ Error 2: Usar mapeo explícito innecesario
```java
// MENOS EFICIENTE - usa mapeo explícito
.mapToDouble(Student::getAge).average().orElse(0.0)

// MÁS EFICIENTE - usa colector directo
.collect(Collectors.averagingDouble(Student::getAge))
```

### ❌ Error 3: Olvidar el cast
```java
// INCORRECTO - count() retorna long, no int
return Arrays.stream(array).parallel().filter(...).count();

// CORRECTO
return (int) Arrays.stream(array).parallel().filter(...).count();
```

### ❌ Error 4: Usar map + Function.identity() innecesario
```java
// MENOS EFICIENTE - usa map explícito
.map(Student::getFirstName)
.collect(Collectors.groupingBy(Function.identity(), Collectors.counting()))

// MÁS EFICIENTE - pasa getter directamente
.collect(Collectors.groupingBy(Student::getFirstName, Collectors.counting()))
```

### ❌ Error 5: Filtro invertido
```java
// INCORRECTO - para estudiantes INACTIVOS
.filter(Student::checkIsCurrent)

// CORRECTO - para estudiantes INACTIVOS
.filter(student -> !student.checkIsCurrent())
```

### ❌ Error 6: Olvidar .parallel()
```java
// INCORRECTO - stream secuencial, no paralelo
Arrays.stream(array).filter(...)

// CORRECTO - stream paralelo
Arrays.stream(array).parallel().filter(...)
```

### ❌ Error 7: Importar clases innecesarias
```java
// INNECESARIO - Function.identity() ya no se usa
import java.util.function.Function;

// NECESARIO - solo se necesita para la nueva implementación
import java.util.stream.Collectors;
```

---

## Conclusiones

### Logros
✅ Implementación exitosa de 3 métodos usando parallel streams
✅ Todos los tests pasaron (correctitud + rendimiento)
✅ Código más conciso y legible que la versión imperativa
✅ Aprovechamiento automático de paralelismo

### Aprendizajes Clave
1. **Parallel Streams** permiten paralelización automática sin gestión manual de threads
2. **Composición funcional** mediante encadenamiento de operaciones (`filter` → `map` → `reduce`)
3. **Method references** (`Student::getAge`) simplifican el código vs. lambdas
4. **Collectors** ofrecen operaciones complejas como `groupingBy` para agregaciones
5. **Optional** maneja casos donde no hay resultados de forma segura

### Recomendaciones
- Usar parallel streams para datasets grandes (> 10,000 elementos)
- Preferir method references sobre lambdas cuando sea posible
- Siempre manejar `Optional` con `.orElse()` o `.orElseThrow()`
- Validar con tests tanto correctitud como rendimiento

---

## Referencias

- [Java 8 Stream API Documentation](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html)
- [Collectors Documentation](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html)
- [Arrays.stream Documentation](https://docs.oracle.com/javase/8/docs/api/java/util/Arrays.html#stream-T:A-)
- [Optional Documentation](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html)

---

**Autor:** Solución para Ejercicio 2 - Computación Paralela y Distribuida UNAL
**Fecha:** 2025
