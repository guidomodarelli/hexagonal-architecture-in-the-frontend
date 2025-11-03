# 🧩 **Servicios de Aplicación y Servicios de Dominio**

---

## 🚀 **Servicios de Aplicación**

Los **servicios de aplicación** son los **puntos de entrada** de nuestra aplicación.
Como se observa en el esquema general, los **controladores** (ya sean de línea de comandos o HTTP) **invocan directamente a estos servicios**.

### 🔧 Funciones principales

* Representan **de forma atómica** un caso de uso del sistema.
* Si se producen modificaciones en el estado de la aplicación:

  * Actúan como **barrera transaccional** frente al sistema de persistencia.
  * **Publican los eventos de dominio** correspondientes.
* **Coordinan** las llamadas entre los distintos componentes del sistema para ejecutar un caso de uso específico.

📌 En resumen, pueden denominarse indistintamente **servicios de aplicación** o **casos de uso**.

---

## 🧠 **Servicios de Dominio**

Los **servicios de dominio** agrupan la **lógica de negocio reutilizable** entre múltiples servicios de aplicación.

### 💡 Ejemplo ilustrativo

Supongamos que nuestra aplicación tiene **dos casos de uso** diferentes.
En ambos necesitamos la lógica que:

* Accede al repositorio de videos.
* Busca un video por su identificador.
* Lanza una excepción de dominio **`VideoNotFound`** si el video no existe.
* Devuelve el video si lo encuentra.

👉 Es importante destacar que **la excepción no la lanza el repositorio**, sino el servicio de dominio.

Para **evitar duplicar** esta lógica, la extraemos en un **servicio de dominio compartido**, invocable desde ambos casos de uso.

### ⚠️ Responsabilidades

Los **servicios de dominio**:

* **Nunca** deben publicar eventos de dominio.
* **Nunca** deben gestionar transacciones.

Esa responsabilidad recae exclusivamente en el **Application Service**, que:

* Garantiza la **atomicidad** del caso de uso.
* Se encarga de **publicar los eventos** correspondientes.

---

## ⚖️ **Diferencias entre usar Servicios de Dominio y de Infraestructura desde la Aplicación**

### 🏗️ **Servicio de Aplicación → Servicio de Infraestructura**

Al interactuar con un servicio de infraestructura desde un servicio de aplicación:

* Aplicamos el **principio de inversión de dependencias**.
* El servicio de aplicación **recibe por constructor** el colaborador de infraestructura, pero **a través de una interfaz definida en el dominio**.
* Esto permite:

  * Evitar acoplamientos con implementaciones de terceros.
  * **Simular colaboradores** durante las pruebas (por ejemplo, inyectar un repositorio en memoria en lugar de uno en MySQL para mayor velocidad).

---

### 🧩 **Servicio de Aplicación → Servicio de Dominio**

Cuando un servicio de aplicación invoca a uno de dominio, la situación cambia:

* Los servicios de dominio contienen **solo lógica de negocio**, por lo tanto **no es necesario desacoplarlos** mediante interfaces.
* Como **no involucran operaciones de entrada/salida**, **no tiene sentido** usar implementaciones diferentes durante las pruebas.
* En los tests, **es conveniente pasar por el servicio de dominio** para **cubrirlo indirectamente** al probar un caso de uso.

Por ello:

* ❌ No se requiere una interfaz para los servicios de dominio.
* ✅ Agregar una capa adicional de indirección solo aportaría **complejidad innecesaria**.

### 🧱 Sobre la instanciación

Existe un punto debatible:

* ¿Debe la **instanciación del servicio de dominio** hacerse dentro del servicio de aplicación?
* ¿O conviene **inyectarlo ya instanciado**, aun sin interfaz?

👉 Ambas opciones son válidas; la elección depende del **criterio del desarrollador**, considerando las ventajas y desventajas de cada enfoque.

---

## 🔍 **Repaso final**

1️⃣ **Caso de uso:**
Si tenemos un caso de uso llamado **`AddProductToCart`** que utiliza un servicio de dominio **`ProductToCartAdder`**,
¿quién publica el evento de dominio **`ProductAddedToCart`**?
✅ **`AddProductToCart`**, es decir, el **servicio de aplicación**.

2️⃣ **Interfaz en servicios de dominio:**
¿Es necesario definir una interfaz para el servicio de dominio?
✅ **No.** Solo se modifican cuando cambia la lógica de negocio y deben probarse **indirectamente** mediante los casos de uso.

3️⃣ **Encapsulación de lógica de negocio:**
¿Siempre encapsulamos la lógica en servicios de dominio?
✅ **No.** Solo cuando:

* Necesitamos **reutilizarla en varios casos de uso**, o
* Queremos **simplificar** y hacer más legible el servicio de aplicación en escenarios complejos.
