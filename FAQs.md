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

