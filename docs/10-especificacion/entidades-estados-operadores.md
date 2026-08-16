# Entidades, estados y operadores

## Entidades iniciales

- **Sujeto:** fuente soberana de decisiones.
- **Identidad:** continuidad verificable del sujeto.
- **Credencial:** evidencia limitada y presentable.
- **Llave:** capacidad para efectuar una operación autorizada.
- **Prueba:** evidencia verificable de una afirmación.
- **Permiso:** relación contextual, temporal y revocable.
- **Evento:** transición identificable y ordenable.

## Estado mínimo

```text
State = {
  subject_id,
  identity_ref,
  credential_ref,
  permission_state,
  valid_from,
  valid_until,
  event_id,
  previous_event_id
}
```

Las referencias no deben contener secretos.

## Estado ternario de permiso

- `-1`: denegado o revocado.
- `0`: indeterminado, pendiente, expirado o fuera de contexto.
- `+1`: autorizado dentro de alcance y vigencia explícitos.

La ausencia de decisión permanece en `0`; nunca se traduce automáticamente a `+1`.

## Operadores iniciales

- `Emitir: Identidad × Alcance → Credencial`
- `Probar: Credencial × Afirmación → Prueba`
- `Verificar: Prueba × Contexto → Resultado`
- `Autorizar: Sujeto × Solicitud → Permiso`
- `Restringir: Permiso × Condición → Permiso'`
- `Revocar: Permiso(+1) → Permiso(-1)`
- `Expirar: Permiso(+1) × Tiempo → Permiso(0)`
- `Recuperar: FragmentosVálidos → CapacidadRestaurada`

Cada operador deberá definir dominio, codominio, precondiciones, efectos, fallos y reversibilidad antes de implementarse.
