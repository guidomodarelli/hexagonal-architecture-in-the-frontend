# FAQ

## 💡 Confusión sobre la Regla de Dependencia

**❓ Planteamiento del problema**

Si cada capa debe comunicarse únicamente con la inmediatamente siguiente, surge la duda:
¿Por qué, en la implementación de un repositorio, instanciamos agregados del dominio?
¿Está permitido omitir capas intermedias en estos casos?

La **regla de dependencia** se aplica principalmente al **flujo de peticiones desde el exterior hacia el interior**.
Su objetivo es **favorecer la tolerancia al cambio** y **proteger el dominio** de cualquier contaminación proveniente de la capa de infraestructura.

Consideramos que **no es especialmente problemático** que las implementaciones de repositorios —incluyendo aquellas usadas en pruebas— **conozcan e instancien agregados del dominio**.

Esto se debe a que:

1️⃣ Lo que realmente persistimos son **los agregados del dominio**.

2️⃣ Por tanto, **instanciarlos resulta inevitable** en este contexto.

**🚨 Lo verdaderamente crítico**

Lo que sí representaría una **violación grave** de la arquitectura sería el caso inverso:
que el **dominio dependiera de la capa de aplicación o de la infraestructura**.

Un ejemplo claro de este problema se da con los **Value Objects de identificadores únicos en PHP**:

* PHP no ofrece soporte nativo para **UUIDs**.
* Esto nos obliga a crear un **Value Object** acoplado a una **librería externa** encargada de las validaciones.

Incluso en este escenario, la **estrategia correcta** consiste en **encapsular dicha dependencia en una única clase**, evitando así que el resto del dominio se vea afectado o contaminado.

## ¿Cómo obtengo todos los errores al validar mi Value Objects?

Al trasladar la **validación de las restricciones de negocio** a los **Value Objects**, surge una cuestión importante:
👉 ¿Cómo podemos recuperar **todos los errores** de una solicitud, por ejemplo, al enviar un formulario?

Es esencial entender que **existen dos tipos de validación**, y cada uno cumple un rol distinto:

* **Value Objects:**
  Se encargan de **impedir la creación o persistencia** de objetos con valores inválidos que no cumplan las **reglas del negocio**.

* **Validación orientada a la experiencia del usuario (UX):**
  Busca **mostrar todos los errores detectados al mismo tiempo**, evitando que el usuario deba reenviar el formulario varias veces por errores individuales.

Para atender ambas necesidades —la integridad del dominio y la usabilidad— se recomienda implementar **dos niveles de validación**:

1️⃣ **A nivel de controlador:**

* Su objetivo es **recopilar y devolver todos los errores** encontrados en la solicitud.
* Mejora la **experiencia de usuario**, permitiendo que todos los errores se muestren de una sola vez.

2️⃣ **A nivel de Value Objects:**

* Garantiza las **restricciones de integridad del dominio**.
* Evita que se instancien o persistan objetos que no cumplan las **reglas de negocio** definidas.

La validación en los Value Objects y la validación a nivel de controlador **no son excluyentes**, sino **complementarias**:

* Una protege la **consistencia del dominio**.
* La otra mejora la **interacción del usuario** con el sistema.

## Inyección de Servicios de Dominio en los Casos de Uso

Durante el desarrollo, hemos defendido la práctica de **instanciar los servicios de dominio directamente dentro de los casos de uso**. Sin embargo, existen diferentes enfoques para abordar este tema, cada uno con sus propias **ventajas y desventajas**.

¿Qué ocurre si un **servicio de dominio**, instanciado en múltiples casos de uso, **comienza a requerir más dependencias**?

* ¿Deberíamos modificar **cada caso de uso** donde fue instanciado?
* ¿No sería más práctico **inyectarlo mediante el constructor**?

Efectivamente, una opción más flexible consiste en **inyectar el servicio de dominio a través del constructor** del caso de uso.
De esta manera:

* El **contenedor de dependencias** o un **factory method** se convierte en el **único punto de modificación**, en caso de que el servicio de dominio necesite más dependencias en el futuro.

**⚠️ Aspectos Clave a Tener en Cuenta**

1️⃣ **Señal de alerta al agregar dependencias:**
Si un servicio de dominio empieza a requerir muchas dependencias, esto puede indicar que **está asumiendo demasiadas responsabilidades**, violando el **Principio de Responsabilidad Única (SRP)** dentro de los principios **SOLID**.

2️⃣ **Evitar la inyección en los servicios de dominio:**
Es preferible **no inyectar dependencias** directamente en los servicios de dominio, ya que estos deberían mantenerse **estables y poco propensos a cambios**.

**🧪 Consideraciones para las Pruebas Unitarias**

* No deberíamos **mockear los servicios de dominio**, dado que **contienen la lógica esencial del dominio**, precisamente la que deseamos validar en los tests unitarios.
* Una alternativa viable es **instanciar el servicio de dominio del mismo modo que el caso de uso**, asegurando así una **verificación real y coherente** del comportamiento del sistema.

## Si olvidamos guardar en la base de datos cuando se realiza un `POST /video/video-id`, ¿qué prueba debería fallar?

Los tests unitarios.

**Análisis General**

1️⃣ **Test de integración**

* Este test **no fallaría**, ya que la **integración con la base de datos seguiría siendo correcta**.
* El problema no radica en la conexión ni en la interacción con la base de datos, sino en que **no se está utilizando realmente**.
* Por lo tanto, este caso puede **descartarse** como origen del fallo.

2️⃣ **Test unitario**

* Si el test está **bien diseñado** y se especifica que debe invocarse el método `save()` del colaborador `VideoRepository`, entonces **fallaría de manera evidente**.
* Este fallo permitiría **detectar claramente el problema**, ya que el método esperado no estaría siendo ejecutado.

3️⃣ **Test de aceptación**

* En este caso, el resultado podría **generar dudas** dependiendo de **qué tipo de verificaciones** se realicen.
* Si el test solo valida que la respuesta HTTP sea **`201 Created`**, **no fallaría**, porque el endpoint seguiría respondiendo correctamente.
* Sin embargo, si la prueba también **comprueba que el registro existe en la base de datos** tras la ejecución, entonces **sí fallaría**, revelando la ausencia del guardado efectivo.
