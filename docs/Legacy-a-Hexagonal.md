# 📝 Estrategia de Refactor (*Legacy* a Hexagonal)

El refactor debe ser incremental y con garantías mediante *tests* de aceptación.

1.  **1️⃣ Añadir *Tests* de Aceptación (E2E):** Cubrir flujos críticos (crear, listar, borrar) y asegurar un entorno determinista (mocks, *seed*).
2.  **2️⃣ Identificar Responsabilidades (*Seams*):** Mapear UI, orquestación, casos de uso, validaciones y persistencia en el código existente.
3.  **3️⃣ Migración Gradual a TypeScript:** Migrar primero tipos esenciales (Entidades, DTOs, *interfaces* de repositorio) para ganar seguridad.
4.  **4️⃣ Definir Interfaz de Repositorio:** Extraer interacciones con la fuente de datos (*localStorage*, API) a una *interface* en *domain* y proveer la implementación concreta en *infrastructure*.
5.  **5️⃣ Implementar Casos de Uso:** Crear funciones puras en *application* que reciban la interfaz del repositorio (inyección de dependencias).
6.  **6️⃣ Mover Reglas de Negocio al Dominio:** Crear *value-objects* y validadores. El dominio lanza errores; los casos de uso los capturan.
7.  **7️⃣ Modularizar `main.ts`:** Reducir a solo la **composición de dependencias** (*Composition Root*): crear repositorios, construir casos de uso y conectar *handlers* de UI.
8.  **8️⃣ Escribir *Tests* Unitarios:** Una vez aislada la lógica, escribir *tests* unitarios para *domain* y *application* (usando repositorios *mock*) y *tests* de integración para los adaptadores (*infrastructure*).
9.  **9️⃣ Refactorizar UI:** Extraer lógica de presentación a *custom hooks* que llamen a los casos de uso, manteniendo la UI lo más delgada posible.
10. **🔟 Revisión y Optimización Continua:** Revisar el código refactorizado, optimizar según sea necesario y asegurar que los *tests* de aceptación sigan pasando.
11. **1️⃣1️⃣ Documentar Decisiones:** A medida que se realicen cambios, documentar las decisiones de diseño y las razones detrás de ellas para futuras referencias.
12. **1️⃣2️⃣ Iterar:** Repetir el proceso para otras partes del sistema hasta completar la migración a una arquitectura hexagonal.

Este enfoque permite una migración segura y controlada hacia una arquitectura hexagonal, minimizando riesgos y asegurando la continuidad del negocio.
