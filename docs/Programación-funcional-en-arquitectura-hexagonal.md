### Programación funcional en arquitectura hexagonal

En el contexto de **JavaScript en frontend**, el uso de clases no es tan común como en otros lenguajes orientados a objetos. Por eso, en este enfoque hemos preferido evitarlas y adoptar un estilo más **funcional y pragmático**.

#### Ventajas de este enfoque

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

#### Resumen

Este enfoque aprovecha la **naturaleza funcional de JavaScript** para simplificar la arquitectura:

- ✅ **Validaciones**: funciones puras, reutilizables y testeables
- ✅ **Repositorios**: funciones tipadas, inyectables sin boilerplate
- ✅ **Casos de uso**: orquestan funciones en lugar de depender de objetos instanciados