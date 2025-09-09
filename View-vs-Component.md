# ¿Diferencia entre View y Component?

- View (Page): orquesta la pantalla. Compone componentes, resuelve dependencias (casos de uso, repos, navegación) y maneja side-effects externos. Sin reglas de dominio.
- Component: pieza de UI reutilizable con lógica de presentación (estado, eventos, validaciones de UI). No implementa reglas de negocio; llama casos de uso vía props o hooks.

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
