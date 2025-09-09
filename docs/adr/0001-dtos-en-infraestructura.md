# ADR 0001: Ubicación de DTOs, Puertos y Adaptadores en Frontend Hexagonal

- Estado: Aceptado
- Fecha: 2025-09-09

## Contexto

En discusiones recientes surgieron dudas sobre:

- Dónde ubicar los DTOs (¿dominio, aplicación, infraestructura o `shared`?).
- En qué capa viven los puertos (interfaces) y quién debe implementarlos.
- Si la infraestructura puede importar entidades del dominio.
- Cómo evitar que `aplicacion` dependa de `infraestructura` al usar DTOs.

## Decisión

1. Los DTOs que representan contratos externos (HTTP/SDK/storage) viven en `infraestructura/api/dto`.
2. La capa de aplicación define sus propios inputs/comandos (p. ej. `CrearUsuarioInput`) y no importa DTOs de infraestructura.
3. Los puertos (interfaces) viven en `aplicacion/puertos`.
4. Las implementaciones concretas de puertos (adaptadores), incluidos repositorios, viven en `infraestructura/`.
5. La infraestructura puede importar `aplicacion` (para conocer el puerto) y `dominio` (para construir entidades/VO). `dominio` y `aplicacion` no importan `infraestructura`.
6. Separar funciones de acceso externo (`infraestructura/api/*`) del repositorio es recomendado, pero opcional si se prioriza simplicidad.

## Consecuencias

- Existirá una capa de mapeo en `infraestructura/api/dto/mapper.ts` para convertir DTO ↔ dominio.
- Los casos de uso serán estables y testables sin conocer detalles de red ni formatos externos.
- Se sugiere añadir reglas de lint para reforzar la dirección de dependencias (ver `docs/hex-frontend/Reglas-de-Dependencias.md`).
- `shared` se usará con moderación; evitar un `User` genérico que propague opcionales/invariantes débiles.

## Referencias

- Guía: `docs/hex-frontend/DTOs-Puertos-Adaptadores.md`
- Ejemplo: `docs/hex-frontend/Ejemplo-CreateUser.md`
- Reglas de dependencias: `docs/hex-frontend/Reglas-de-Dependencias.md`

