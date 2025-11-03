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

