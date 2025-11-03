# ¿Diferencia entre View y Component?

## 🧱 Arquitectura de la Interfaz

### 🖥️ **View (Page)**

La **View** —también llamada *Page*— se encarga de **orquestar toda la pantalla**.
Sus responsabilidades principales son:

1️⃣ **Composición:** agrupa y organiza los distintos componentes visuales que conforman la página.
2️⃣ **Gestión de dependencias:** resuelve y conecta los **casos de uso**, **repositorios**, **servicios de navegación** u otros recursos externos necesarios.
3️⃣ **Manejo de efectos secundarios:** controla los **side-effects** externos (peticiones, listeners, suscripciones, etc.).
4️⃣ **Sin reglas de dominio:** no contiene lógica de negocio; su función es puramente estructural y de coordinación.

---

### 🧩 **Component**

El **Component** es una **pieza reutilizable de la UI** con su propia **lógica de presentación**.
Sus funciones principales son:

1️⃣ **Estado y eventos:** gestiona su estado interno, las interacciones del usuario y los eventos de interfaz.
2️⃣ **Validaciones de UI:** controla aspectos visuales y validaciones relacionadas con la experiencia del usuario.
3️⃣ **Sin lógica de negocio:** no implementa reglas de dominio; se comunica con los **casos de uso** mediante **props** o **hooks**, delegando la lógica compleja a otros niveles.


Ejemplo mínimo (React) con dependencias mínimas

## Componente de formulario (solo React)

```tsx
import React, { useState } from 'react';
import { isCourseTitleValid, isCourseDurationValid } from '../modules/courses/domain';

type CreateInput = { title: string; duration: number };

interface Props {
  onCreate: (data: CreateInput) => Promise<void>
};

export function CreateCourseForm({ onCreate }: Props) {
  const [title, setTitle] = useState('');
  const [duration, setDuration] = useState(''); // como texto para el input

  const isTitleValid = isCourseTitleValid(title);
  const isDurationValid = isCourseDurationValid(Number(duration));
  const canSubmit = isTitleValid && isDurationValid;

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!canSubmit) return;
    await onCreate({ title, duration: Number(duration) });
    setTitle('');
    setDuration('');
  };

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Title
        <input
          aria-label="title"
          value={title}
          onChange={(e) => setTitle(e.target.value)}
        />
      </label>

      <label>
        Duration
        <input
          aria-label="duration"
          value={duration}
          onChange={(e) => setDuration(e.target.value)}
        />
      </label>

      {!isTitleValid && <p role="alert">Title must be 5–100 chars</p>}
      {!isDurationValid && <p role="alert">Duration must be > 0</p>}

      <button type="submit" disabled={!canSubmit}>
        Create
      </button>
    </form>
  );
}
```

## View que orquesta dependencias (mínimo imprescindible)

```tsx
import React from 'react';
import { CreateCourseForm } from './CreateCourseForm';
import { createCourseFactory } from '../modules/courses/use-cases/createCourse';
import { createHttpCourseRepository } from '../modules/courses/infra/httpCourseRepository';

// La View orquesta y resuelve dependencias mediante imports,
// dejando el componente puro y fácil de testear.
const repo = createHttpCourseRepository(); // implementa CourseRepository (fetch/axios...)
const createCourse = createCourseFactory(repo); // devuelve (input) => Promise<void>

export function CreateCourseView() {
  return <CreateCourseForm onCreate={createCourse} />;
}
```
