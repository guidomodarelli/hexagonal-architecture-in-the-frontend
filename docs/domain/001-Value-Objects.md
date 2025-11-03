# 🧩 **Value Objects**

Los **Value Objects** son clases que se **identifican por el valor que representan**, no por un identificador único.

---

## 🎯 **Concepto principal**

Mientras que las **entidades de dominio** (por ejemplo, un `Video`) poseen un **identificador único**, los **Value Objects** modelan conceptos cuyo valor **define completamente su identidad**.

➡️ Si el valor cambia, **ya no representan el mismo concepto**.
Por ello, se dice que **se identifican por el valor que contienen**.

El objetivo de los Value Objects es **garantizar la validez** de los valores utilizados en nuestro dominio, **evitando instancias o persistencias inválidas** que no cumplan con las reglas del negocio.

---

## 💻 **Ejemplo: `VideoURL`**

```ts
class VideoURL {
  private readonly value: string;

  constructor(value: string) {
    this.guardValidURL(value);
    this.value = value;
  }

  private guardValidURL(url: string): void {
    try {
      new URL(url);
    } catch (error) {
      throw new Error(`The url <${url}> is not well formatted`);
    }
  }

  getValue(): string {
    return this.value;
  }
}
```

---

## 🧠 **¿Qué beneficios aporta modelar conceptos de dominio con Value Objects?**

1️⃣ **Semántica de dominio:**
Contar con tipos propios que representen conceptos específicos (como `VideoURL`, `UserRange`, `Rating`, etc.) hace que el **código sea más legible** y que **exprese mejor los conceptos del dominio**.

2️⃣ **Cohesión:**
Al tener una clase que modela, por ejemplo, las URLs de los videos, toda la **lógica relacionada queda autocontenida** en ella.
Esto logra que la **lógica esté más cerca de los datos que necesita**, lo cual aporta dos beneficios adicionales:

* **Evitar comprobaciones redundantes:**
  Desde el momento en que recibimos un objeto del tipo `VideoURL`, podemos **omitir verificaciones repetitivas** en otras partes del código.
  La validación ocurre en la **instanciación** del objeto, garantizando que siempre sea una URL válida. Así, no será necesario comprobar si es nula o incorrecta más adelante.

* **“Imán” de lógica:**
  Al principio, la creación de un Value Object puede parecer innecesaria.
  Sin embargo, conforme el sistema crece, se vuelven **puntos naturales donde centralizar pequeñas porciones de lógica** que, de otro modo, terminarían dispersas en servicios o modelos sobredimensionados.

---

## ⚙️ **Resumen visual**

| Beneficio                   | Descripción                                                   |
| --------------------------- | ------------------------------------------------------------- |
| 🧾 Semántica de dominio     | Representa claramente los conceptos del negocio en el código. |
| 🧱 Cohesión                 | La lógica y los datos viven juntos en un mismo lugar.         |
| 🚫 Menos redundancia        | Se validan los datos una sola vez al instanciar.              |
| 🧲 Centralización de lógica | Los Value Objects sirven como puntos naturales de extensión.  |

---
