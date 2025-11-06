# Programación funcional en arquitectura hexagonal

En el contexto de **JavaScript en frontend**, el uso de clases no es tan común como en otros lenguajes orientados a objetos. Por eso, en este enfoque hemos preferido evitarlas y adoptar un estilo más **funcional y pragmático**.

## Ventajas de este enfoque

**1. Sin instanciación de objetos**

No existe un `new Course()`. El objeto que recibimos como parámetro ya cumple con la interfaz de curso, así que la responsabilidad se limita a **validar sus propiedades** en lugar de construir instancias.

**2. Validaciones como funciones puras**

Al no haber constructores, no podemos ejecutar las validaciones en la creación de la instancia. En su lugar, definimos **funciones independientes y reutilizables** —por ejemplo `courseID.ts`, `courseTitle.ts`, etc.— que validan cada aspecto del curso y lanzan una excepción si algo es incorrecto.

```typescript
// Ejemplo conceptual
ensureCourseIsValid(course) // Agrupa todas las validaciones
```

**Beneficios:**
- Funciones **puras y testeables** de forma aislada
- **Reutilizables en la lógica de UI**, manteniendo consistencia entre capas
- Separación clara de responsabilidades

**3. Repositorios como funciones**

En vez de definir una interfaz de repositorio y luego instanciarla con algo como `createLocalStorageRepository()`, el tipado y la definición del repositorio pueden hacerse **directamente como funciones exportadas**.

```typescript
// En lugar de: repository.save(course)
// Usamos: saveCourse(course)
```

Así, un caso de uso recibe directamente la función que necesita (`saveCourse`, `findCourseById`, etc.), sin necesidad de encapsularlas en un objeto.

---

## Resumen

Este enfoque aprovecha la **naturaleza funcional de JavaScript** para simplificar la arquitectura:

- ✅ **Validaciones**: funciones puras, reutilizables y testeables
- ✅ **Repositorios**: funciones tipadas, inyectables sin boilerplate
- ✅ **Casos de uso**: orquestan funciones en lugar de depender de objetos instanciados

## ¿Tienen sentido los Value Objects?

En programación orientada a objetos, un **Value Object** encapsula un valor y concentra su lógica asociada, evitando que esta se disperse en la entidad principal. Por ejemplo, en lugar de que `Course` tenga propiedades primitivas (`string`, `number`), cada una se representa mediante su propio Value Object: `CourseTitle`, `ImageUrl`, `CourseId`, etc.

**Ventaja clave:** las validaciones (longitud, formato, rangos) viven en el Value Object correspondiente, no en la entidad.

---

### Aplicación en frontend: Value Files

En frontend podemos adoptar el mismo patrón de forma **funcional y ligera**. En lugar de clases, usamos **archivos independientes** (Value Files) que exportan:

1. **Tipo semántico** (alias sobre primitivos)
2. **Reglas de validación**
3. **Funciones auxiliares** (errores, normalizaciones)

**Ejemplo** (`CourseTitle.ts`):

```typescript
// Tipo semántico
export type CourseTitle = string;

// Constantes de validación
export const COURSE_TITLE_MIN_LENGTH = 5;
export const COURSE_TITLE_MAX_LENGTH = 100;

// Validación
export function isCourseTitleValid(title: string): boolean {
  return (
    title.length >= COURSE_TITLE_MIN_LENGTH &&
    title.length <= COURSE_TITLE_MAX_LENGTH
  );
}

// Error asociado
export function CourseTitleNotValidError(title: string): Error {
  return new Error(`Title "${title}" is not valid.`);
}
```

La interfaz `Course` usa tipos semánticos en lugar de primitivos:

```typescript
export interface Course {
  id: CourseId;
  title: CourseTitle;
  imageUrl: ImageUrl;
}
```

---

### Beneficios

- **Semántica rica:** `CourseTitle` expresa mejor el dominio que `string`
- **Consistencia:** las reglas viven junto al valor que gobiernan
- **Evolutivo:** empieza con un alias simple, añade validaciones cuando sea necesario
- **Testeable:** funciones puras, fáciles de probar en aislamiento
- **Sin overhead:** no requiere instancias ni constructores

---

**Conclusión:** Los Value Objects tienen pleno sentido en frontend mediante un enfoque pragmático: **tipos alias + funciones puras** en archivos separados. Mantiene la semántica y disciplina del dominio sin la complejidad de clases.
