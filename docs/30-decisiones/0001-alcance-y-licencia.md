# ADR-0001 — Alcance fundacional y licencia

**Estado:** propuesta inicial  
**Fecha:** 2026-08-16

## Contexto

El repositorio fue creado con un README nominal y licencia AGPL-3.0, sin código ni especificación. Su nombre puede sugerir almacenamiento de material criptográfico sensible.

## Decisión

Usar inicialmente el repositorio como raíz pública de especificación. No almacenar secretos ni datos reales. Separar fundamento, especificación, amenazas, decisiones y esquemas antes de incorporar una implementación.

Mantener provisionalmente AGPL-3.0 y revisar su adecuación antes del primer código ejecutable.

## Consecuencias

- Se reduce el riesgo de construir sobre términos ambiguos.
- La arquitectura podrá revisarse antes de fijar tecnología.
- Cualquier servicio futuro basado en código AGPL deberá atender sus obligaciones de disponibilidad del código fuente.
- Esta decisión no valida ninguna hipótesis criptográfica.
