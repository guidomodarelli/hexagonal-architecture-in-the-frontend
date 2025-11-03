# 🧱 Arquitectura Hexagonal

La **Arquitectura Hexagonal** (también conocida como *Ports and Adapters*) busca desacoplar la lógica de negocio central del resto del sistema, permitiendo una mayor flexibilidad, mantenibilidad y capacidad de prueba.

---

## 🧩 Capas Principales

### **1️⃣ Dominio (Domain)**

Es el núcleo de la aplicación y contiene **la lógica de negocio central**.
Define las **reglas de negocio** y no depende de ninguna otra capa.

Elementos típicos del dominio:

* **Entidades**
* **Value Objects**
* **Servicios de dominio**
* **Tipos e interfaces**
* **Funciones de validación**

📌 *Ejemplo:* `Auth`, `Product`, `Course`, `AuthRepository`, `ProductRepository`, `CourseRepository`

---

### **2️⃣ Aplicación (Application)**

Actúa como **puente entre el dominio y el mundo exterior**.
Se encarga de los **casos de uso** y del **flujo transaccional** de la aplicación.
Aquí es donde se orquesta la comunicación entre las diferentes capas.

📌 *Ejemplo:* `AuthCommand`, `AuthCommandHandler`, `Authenticator`, `CourseCreator`, `CourseRenamer`

---

### **3️⃣ Infraestructura (Infrastructure)**

Contiene las **implementaciones concretas** de los detalles técnicos:

* Llamadas a **APIs**
* Acceso a **bases de datos**
* **Ficheros** e **I/O**
* Código **acoplado a librerías o vendors externos**

Aquí se implementan las **interfaces definidas en el dominio**, traduciéndolas a código funcional según la tecnología utilizada.

📌 *Ejemplo:* `MySQLCourseRepository`, `RedisAuthRepository`

---

## 🧭 Dependencias entre Capas

Las dependencias **siempre deben apuntar hacia el interior**:

> Las capas externas dependen de las internas, **nunca al revés**.

```
❌ Estructura Incorrecta:
⊢- application
  ⊢- AuthCommand
  ⊢- AuthCommandHandler
  ⊢- Authenticator
  ⊢- CourseCreator
  ⊢- CourseRenamer
⊢- domain
  ⊢- Auth
  ⊢- Product
  ⊢- Course
  ⊢- AuthRepository
  ⊢- ProductRepository
  ⊢- CourseRepository
⊢- infrastructure
  ⊢- MySQLCourseRepository
  ⊢- RedisAuthRepository
```

---

## 🏗️ Arquitectura Hexagonal + Vertical Slicing

El concepto de **Vertical Slicing** propone dividir el sistema en **funcionalidades verticales completas**, donde cada *slice* incluye todas las capas necesarias (dominio, aplicación e infraestructura) para entregar un valor funcional al usuario.

Cada módulo es **independiente**, lo que favorece la modularidad, la escalabilidad y el trabajo en paralelo entre equipos.

```
✅ Estructura Recomendada:
⊢- auth
  ⊢- application
    ...
  ⊢- domain
    ...
  ⊢- infrastructure
    ...
⊢- courses
  ⊢- application
    ...
  ⊢- domain
    ...
  ⊢- infrastructure
    ...
...
```

---

## ⚙️ Regla de Dependencia

La **regla de dependencia** establece que **cada capa solo debe conocer las clases de la capa inmediatamente inferior**.

**Orden jerárquico (de exterior a interior):**

> Infraestructura → Aplicación → Dominio

🔒 Este principio permite modificar las capas externas sin afectar las internas.
Por ello, los elementos **más variables o dependientes de terceros** se ubican en la **capa de Infraestructura**.

---

## 🔌 Puertos y Adaptadores

* **Puertos (Ports):**
  Son las **interfaces** definidas en la capa de dominio para desacoplar la lógica de negocio de la infraestructura.
  📍 *Ejemplo:* `UserRepository`

* **Adaptadores (Adapters):**
  Son las **implementaciones concretas** de los puertos, las cuales traducen los contratos definidos en el dominio hacia la lógica específica de un proveedor o tecnología.
  📍 *Ejemplo:* `MySQLUserRepository`

---

🧠 En resumen, la arquitectura hexagonal junto al enfoque de vertical slicing permite desarrollar sistemas **modulares, escalables y fácilmente mantenibles**, donde la lógica de negocio permanece protegida de los detalles técnicos externos.
