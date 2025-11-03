# 🧩 Tests Unitarios en Frontend

Los **tests unitarios en frontend** buscan mantener las unidades de negocio (casos de uso) pequeñas y fácilmente testeables.
Sin embargo, para cubrir el flujo **real del usuario**, solemos iniciar las pruebas desde el **componente**, lo que permite validar la ruta completa **UI → Caso de uso** con un **bajo acoplamiento**.

---

### 🔄 Flujo de arquitectura y niveles de prueba

```
View ===>  Component ===>  Use Case  ===>  Repository
                                                ↑
                                                └- - - - - - - Implementación

          <-----------------Unit test-----------------><---Integration test--->
<---------------------------------e2e----------------------------------------->
```

---

## 🧠 Qué cubren los tests de un componente

* 🧱 **Render básico y estados de la UI.**
* ✅ **Validaciones de interfaz** (no de dominio).
* 🧭 **Interacciones:** comprobar que se invoque el caso de uso con los datos correctos.
* ⚠️ **Estados de envío o errores** propios de la capa de presentación.

---

## 🧪 Qué testear (resumen)

1️⃣ **Componente:**

* Habilita o deshabilita el botón según la validez del formulario.
* Muestra errores de UI correctamente.
* Llama a `onCreate` con datos válidos `{ title, duration }`.

2️⃣ **Vista (View):** *(opcional)*

* Verifica que las dependencias estén bien conectadas.
* Usa un doble del repositorio y espera correctamente a `save`.

---

## ⚙️ Ejemplo de test básico (Jest + Testing Library)

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { CreateCourseForm } from './CreateCourseForm';

test('envía datos válidos', async () => {
  const user = userEvent.setup();
  const onCreate = jest.fn().mockResolvedValue(undefined);

  render(<CreateCourseForm onCreate={onCreate} />);

  await user.type(screen.getByLabelText(/title/i), 'React 101');
  await user.type(screen.getByLabelText(/duration/i), '60');
  await user.click(screen.getByRole('button', { name: /create/i }));

  expect(onCreate).toHaveBeenCalledWith({ title: 'React 101', duration: 60 });
});
```

---

## 📝 Notas importantes

* ⚙️ **Las validaciones de dominio** deben mantenerse en la capa **Domain/Application**.
  En este ejemplo solo se incluyen **validaciones de UI** para simplificar.

* 🧩 El **caso de uso real** debe recibir **interfaces (repositorios)** como parámetros, **sin depender de implementaciones concretas**.

* 🔄 Si se prefiere, el caso de uso puede **envolverse en un hook** (`useCreateCourse`), que luego el componente consuma.
  Sin embargo, **pasar `onCreate` por props** suele ser más **explícito y fácil de testear**.
