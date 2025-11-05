### Programación funcional en arquitectura hexagonal

En el contexto de **JavaScript en frontend**, el uso de clases no es tan común como en otros lenguajes orientados a objetos. Por eso, en este enfoque hemos preferido evitarlas y adoptar un estilo más funcional.

Algunos puntos interesantes de este planteo:

1. **Sin instanciación de objetos**
   No existe un `new Course`. El objeto que recibimos como parámetro ya cumple con la interfaz de curso, así que la responsabilidad se limita a **validar sus propiedades**.

2. **Validaciones como funciones puras**
   Al no haber constructores, no podemos ejecutar las validaciones en la creación de la instancia. En su lugar, definimos funciones independientes —por ejemplo `courseID.ts`, `courseTitle.ts`, etc.— que validan cada aspecto del curso y lanzan una excepción si algo es incorrecto.
   De esta forma, podemos tener una función central `ensureCourseIsValid(course)` que agrupe estas validaciones.
   Además, estas funciones son **reutilizables en la lógica de UI**, manteniendo consistencia entre capas.

3. **Repositorios como funciones sueltas**
   En vez de definir una interfaz de repositorio y luego instanciarla con algo como `createLocalStorageRepository()`, el tipado y la definición del repositorio pueden hacerse directamente como funciones exportadas.
   Así, un caso de uso recibe directamente la función que necesita (`saveCourse`, `findCourseById`, etc.), sin necesidad de encapsularlas en un objeto.

---

👉 En resumen, este enfoque aprovecha la naturaleza funcional de JavaScript para simplificar la arquitectura:

* **Validaciones** como funciones puras, reutilizables y testeables.
* **Repositorios** como funciones tipadas, inyectables sin boilerplate.
* **Casos de uso** que orquestan funciones en lugar de depender de objetos instanciados.