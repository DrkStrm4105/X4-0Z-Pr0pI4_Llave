# Modelo de amenazas inicial

## Activos

- continuidad de identidad;
- capacidad soberana de decisión;
- secretos criptográficos;
- permisos y su historial;
- evidencia de autenticidad;
- mecanismos de recuperación.

## Adversarios considerados

- atacante remoto;
- colaborador con privilegios excesivos;
- interfaz o adaptador defectuoso;
- dependencia comprometida;
- pérdida o robo de dispositivo;
- correlación de metadatos;
- error humano.

## Amenazas prioritarias

| Amenaza | Consecuencia | Control inicial |
|---|---|---|
| Secreto publicado | Compromiso irreversible | Prohibición, detección y rotación |
| Confundir identidad con credencial | Exposición total | Separación de entidades |
| Reutilizar permiso fuera de contexto | Escalada de privilegios | Alcance y vigencia explícitos |
| Convertir ausencia en autorización | Consentimiento fabricado | Estado ternario |
| Punto único de recuperación | Pérdida o captura total | Fragmentación y umbral |
| Traductor con pérdida silenciosa | Corrupción semántica | Pruebas de ida y vuelta |
| Criptografía no auditada | Falsa seguridad | Etiqueta experimental y revisión externa |

## Límites de confianza

La interfaz puede presentar una decisión, pero no crearla. El almacenamiento puede conservar evidencia, pero no ampliar permisos. Un adaptador puede traducir formatos, pero debe declarar toda pérdida semántica.
