# 📝 Estrategia de Refactor (*Legacy* a Hexagonal)

El refactor debe ser **incremental y seguro**, garantizado mediante *tests* de aceptación que validen el comportamiento en cada paso.

## 🎯 Fases del Refactor

### 1️⃣ Añadir *Tests* de Aceptación (E2E)
Cubrir flujos críticos (crear, listar, borrar) y asegurar un entorno determinista (mocks, *seed* de datos). Estos *tests* serán tu red de seguridad durante todo el proceso.

### 2️⃣ Identificar Responsabilidades (*Seams*)
Analizar el código actual para ubicar dónde está cada responsabilidad:
- **UI**: Componentes que renderizan y capturan eventos
- **Orquestación**: Código que coordina múltiples operaciones
- **Casos de uso**: Lógica de negocio (aunque esté mezclada con UI)
- **Validaciones**: Reglas que verifican datos
- **Persistencia**: Llamadas a `localStorage`, APIs, etc.

Documenta cómo se relacionan entre sí (qué llama a qué) para identificar puntos de separación (*seams*) donde puedes insertar abstracciones.

### 3️⃣ Migración Gradual a TypeScript
Comenzar por tipos esenciales: Entidades, DTOs e *interfaces* de repositorio. Esto proporciona seguridad de tipos desde el inicio.

### 4️⃣ Definir Interfaz de Repositorio
Extraer interacciones con la fuente de datos (*localStorage*, API) a una *interface* en *domain*. Implementar el adaptador concreto en *infrastructure*.

### 5️⃣ Implementar Casos de Uso
Crear funciones puras en *application* que reciban la interfaz del repositorio vía inyección de dependencias. Mantén la lógica agnóstica de la infraestructura.

### 6️⃣ Mover Reglas de Negocio al Dominio
Crear *value-objects* y validadores. **Principio clave:** el dominio lanza errores; los casos de uso los capturan y manejan.

### 7️⃣ Modularizar `main.ts`
Reducir a solo la **composición de dependencias** (*Composition Root*): 
- Instanciar repositorios
- Construir casos de uso

### 8️⃣ Escribir *Tests* Unitarios
Con la lógica aislada:
- *Tests* unitarios para *domain* y *application* (usando repositorios *mock*)
- *Tests* de integración para adaptadores (*infrastructure*)

### 9️⃣ Refactorizar UI
Extraer lógica de presentación a *custom hooks* o servicios que deleguen en casos de uso. La UI debe ser lo más **delgada** posible.

### 🔟 Revisión y Optimización Continua
Revisar código refactorizado, optimizar según necesidad y **validar que los *tests* de aceptación sigan en verde**.

### 1️⃣1️⃣ Documentar Decisiones (ADRs)
Registrar decisiones de arquitectura (*Architecture Decision Records*) para mantener contexto y razonamiento.

### 1️⃣2️⃣ Iterar
Repetir el proceso para otras áreas del sistema hasta completar la migración.

---

## ✅ Beneficios de este Enfoque

- ✨ **Migración segura y controlada**
- 🛡️ Minimiza riesgos al negocio
- 📊 Permite medir progreso incremental
- 🔄 Facilita *rollback* si es necesario
- 👥 Equipo puede trabajar en paralelo en diferentes capas
