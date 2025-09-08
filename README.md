# Arquitectura Hexagonal en el Frontend

Este proyecto demuestra cómo implementar la arquitectura hexagonal (también conocida como arquitectura de puertos y adaptadores) en aplicaciones frontend. Esta arquitectura promueve la separación de responsabilidades y mejora la mantenibilidad y escalabilidad del código.

## ¿Existe lógica de negocio en el frontend?

Efectivamente, las aplicaciones frontend contienen lógica de negocio responsable de procesar datos, validar entradas y coordinar la interacción entre componentes de la interfaz. El objetivo es organizar esta lógica manteniéndola separada de la presentación e infraestructura, lo cual se logra mediante la arquitectura hexagonal.

La lógica de negocio define y controla las reglas y procesos que rigen la operación de una aplicación, independientemente del framework utilizado. Incluye validaciones, cálculos y orquestación de llamadas a servicios externos, y debe ser fácilmente testeable y mantenible.

Por ejemplo, al crear una nueva issue en GitHub, el frontend valida que no se pueda crear sin título, aunque esta validación también ocurra en el backend. Esta lógica reside en el frontend.

También consideramos la estructura de datos específica para cada contexto. Los datos de usuario necesarios en la página principal difieren de los requeridos en el desplegable de asignación de issues, requiriendo modelos distintos aunque compartan algunos campos.

Podríamos crear una interfaz TypeScript con campos opcionales:

```typescript
interface User {
  id: UUID;
  username: string;
  avatarUrl: string;
  name?: string; // Opcional
  status?: 'active' | 'inactive'; // Opcional
}
```

Sin embargo, esto añadiría condicionales innecesarios y complicaría el mantenimiento. Preferimos interfaces específicas para cada caso de uso:

```typescript
interface User {
  id: UUID;
  username: string;
  avatarUrl: string;
}

interface Assignee {
  id: UUID;
  username: string;
  avatarUrl: string;
  name: string; // Campo requerido para el desplegable
  status: 'active' | 'inactive'; // Campo requerido para el desplegable
}
```

Estas decisiones de diseño confirman la existencia de lógica de negocio en el frontend y la importancia de mantenerla bien organizada.

Ambas interfaces comparten campos base, pero `Assignee` incluye campos adicionales necesarios para el contexto específico del desplegable de asignación. Esta separación permite que cada parte de la aplicación utilice la estructura de datos más apropiada, mejorando la claridad y mantenibilidad del código.

## Frameworks + Arquitectura Hexagonal

La arquitectura hexagonal es agnóstica al framework utilizado. Puede implementarse con React, Vue, Angular o cualquier otro framework frontend. La clave está en seguir los principios de separación de responsabilidades y mantener la lógica de negocio aislada de la infraestructura y presentación.

```
--> View: Componentes de UI (React, Vue, etc.)
--> Application: Casos de uso y lógica de negocio
--> Domain: Entidades y reglas de negocio
--> Infrastructure: Adaptadores para APIs, almacenamiento, etc.
```

```
+---------------------+
|      View           |
|  (React, Vue, etc.) |
+----------+----------+
           |
           v
+----------+----------+
|    Application      |
| (Use Cases, Logic)  |
+----------+----------+
           |
           v
+----------+----------+
|      Domain         |
| (Entities, Rules)   |
+----------+----------+
           |
           v
+----------+----------+
|   Infrastructure    |
| (APIs, Storage, etc.)|
+---------------------+
```

```
*
⊢ App.tsx
⊢ modules/
  ⊢ courses/
    ⊢ application/
      ⊢ use-cases/
        ⊢ CreateCourse.ts
        ⊢ DeleteCourse.ts
        ⊢ GetCourses.ts
        ⊢ UpdateCourse.ts
      ⊢ services/
        ⊢ CourseService.ts
    ⊢ domain/
      ⊢ entities/
        ⊢ Course.ts
      ⊢ repositories/
        ⊢ CourseRepository.ts
      ⊢ value-objects/
        ⊢ CourseId.ts
        ⊢ CourseName.ts
    ⊢ infrastructure/
      ⊢ rest/
        ⊢ api/
          ⊢ CourseApi.ts
        ⊢ repositories/
          ⊢ CourseRepositoryImpl.ts
      ⊢ graphql/
        ⊢ api/
          ⊢ CourseGraphQLApi.ts
        ⊢ repositories/
          ⊢ CourseGraphQLRepositoryImpl.ts
    ⊢ presentation/
      ⊢ components/
        ⊢ CourseList.tsx
        ⊢ CourseForm.tsx
      ⊢ pages/
        ⊢ CoursesPage.tsx
```

La arquitectura hexagonal busca separar la lógica de negocio de la lógica de interfaz de usuario (mostrar/ocultar elementos, manejo de inputs, etc.).

Así, si decidimos cambiar de framework en el futuro, la lógica de negocio permanecerá intacta y solo será necesario reescribir la capa de presentación.

Al organizar la aplicación en capas, surge la pregunta sobre dónde ubicar la lógica de interfaz de usuario que maneja el framework: ¿es dominio, aplicación o infraestructura?

Podríamos considerarla infraestructura, ya que el framework es una dependencia externa. Sin embargo, esta capa a menudo sirve como punto de entrada en los tests unitarios, tradicionalmente asociado con la capa de aplicación.

Además, las particularidades de los frameworks frecuentemente limitan la estructura de la aplicación, por ejemplo, requiriendo un archivo `main.ts` dentro de la carpeta `src`. Esto sugiere que la capa de presentación trasciende la simple infraestructura.