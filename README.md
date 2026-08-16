# X4-0Z-Pr0pI4_Llave

Raíz pública de especificación para una arquitectura soberana de identidad, prueba, permiso, revocación y recuperación.

> **Estado:** fundacional y experimental. No contiene todavía una implementación criptográfica lista para producción.

## Distinción nuclear

La llave no es el secreto almacenado. Es la arquitectura que permite demostrar, autorizar, revocar y reconstruir sin exponer aquello que protege.

Este repositorio separa explícitamente:

- **identidad:** continuidad reconocible del sujeto;
- **credencial:** medio limitado para demostrar una propiedad;
- **llave:** capacidad criptográfica o lógica;
- **prueba:** evidencia verificable sin autoridad ilimitada;
- **permiso:** autorización contextual, temporal y revocable.

## Límites de seguridad

Nunca incluir:

- claves privadas, frases semilla, tokens o contraseñas;
- credenciales reales o archivos `.env`;
- datos biométricos operativos;
- datos personales identificables;
- material criptográfico de producción;
- volcados de bases de datos o registros privados.

Los ejemplos futuros deberán usar identidades, secretos y datos exclusivamente ficticios.

## Organización

- `docs/00-fundamento/`: propósito e invariantes.
- `docs/10-especificacion/`: entidades, estados y operadores.
- `docs/20-modelo-de-amenazas/`: activos, adversarios y mitigaciones.
- `docs/30-decisiones/`: decisiones arquitectónicas.
- `schemas/`: futuros contratos serializables.

No se introducirán `src/` ni dependencias hasta definir una cápsula ejecutable mínima y sus criterios de validación.

## Licencia

GNU Affero General Public License v3.0. Antes de incorporar código debe verificarse que su copyleft, incluido el uso mediante servicios de red, coincide con el modelo de distribución previsto.

## Contribución

La rama `main` representa el estado revisado. Los cambios deberían proponerse mediante ramas y pull requests, manteniendo trazabilidad y revisión antes de integrar.
