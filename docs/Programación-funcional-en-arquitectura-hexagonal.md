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

## ¿Tienen sentido los ValueObjects?

En programación orientada a objetos, un **ValueObject** encapsula un valor y concentra la lógica asociada a él. Así evitamos que esa lógica termine dispersa en la entidad principal. Por ejemplo, en lugar de que la clase `Course` tenga propiedades primitivas como `string` o `number`, cada propiedad se representa mediante su propio ValueObject: `CourseTitle`, `ImageUrl`, `CourseId`, etc.

La ventaja es que, si necesitamos agregar validaciones (ejemplo: longitud mínima o máxima del título), no lo haríamos en la clase `Course`, sino en el ValueObject correspondiente.

---

### ¿Y en el frontend?

En el frontend podemos aplicar el mismo patrón, pero de forma más ligera y funcional. En lugar de definir clases, podemos encapsular cada valor en un archivo independiente —lo que podríamos llamar un **ValueFile**— que exporta:

1. **El tipo semántico** (alias sobre un primitivo).
2. **Las reglas de validación**.
3. **Las funciones auxiliares** (errores, normalizaciones, etc.).

Ejemplo en **TypeScript** (`CourseTitle.ts`):

```ts
// Tipo semántico
export type CourseTitle = string;

// Constantes de validación
export const COURSE_TITLE_MIN_LENGTH = 5;
export const COURSE_TITLE_MAX_LENGTH = 100;

// Validaciones
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

De esta forma, en la interfaz `Course` ya no trabajamos con `string`, sino con `CourseTitle`:

```ts
export interface Course {
  id: CourseId;
  title: CourseTitle;
  imageUrl: ImageUrl;
}
```

---

### Beneficios de este enfoque

* **Semántica fuerte:** el código expresa mejor el dominio (`CourseTitle` vs `string`).
* **Consistencia:** las reglas viven junto al valor que afectan.
* **Evolutivo:** si al principio un valor no tiene lógica extra, basta con un alias de tipo. Si más adelante necesita validaciones, lo ampliamos en el mismo archivo, sin ensuciar la entidad principal.
* **Funcional:** no dependemos de clases ni instancias, pero seguimos respetando la filosofía de los ValueObjects.

---

👉 En resumen: **sí tiene sentido aplicar ValueObjects en el frontend**, pero con un enfoque práctico: tipos alias + funciones puras en archivos separados. Es más liviano que en backend, pero mantiene la semántica y disciplina del dominio.
