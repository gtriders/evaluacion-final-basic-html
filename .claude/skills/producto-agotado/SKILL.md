---
name: producto-agotado
description: Cuando el usuario diga que se acabó un ingrediente o producto (por ejemplo "se acabó la reineta"), marca como agotados todos los productos que lo contienen en Uber Eats (pedidos), adriasushi.cl y adriasangucheria.cl, y avisa qué se desactivó. También sirve para reactivar productos cuando vuelve el stock.
---

# Producto agotado

Cuando el usuario avisa que se acabó un ingrediente o producto, esta skill:

1. Identifica qué productos dependen de ese ingrediente.
2. Los desactiva en cada canal: Uber Eats (pedidos), adriasushi.cl y, si corresponde, adriasangucheria.cl.
3. Guarda el estado para poder reactivarlos después.
4. Responde con un resumen claro de qué quedó desactivado y qué requiere acción manual.

## Entradas

- **Ingrediente o producto agotado** (obligatorio): por ejemplo "reineta", "salmón", "palta".
- **Acción**: `agotar` (por defecto) o `reactivar`.

Si el usuario no dice la acción, asume `agotar`.

## Paso 1: Mapa de ingredientes

Antes de tocar nada, busca el ingrediente en el mapa de productos del proyecto. Si no existe, créalo en `.claude/skills/producto-agotado/ingredientes.json` con este formato:

```json
{
  "reineta": {
    "uber_eats": ["Sushi roll salmón reineta", "Nigiri reineta"],
    "adriasushi.cl": ["Roll Reineta Furai"],
    "adriasangucheria.cl": []
  }
}
```

- Un ingrediente puede aparecer en varios productos y en varios canales.
- Si un producto usa el ingrediente solo como acompañamiento o en una variante, inclúyelo igual y márcalo en el resumen.
- Si el mapa no tiene el ingrediente, **pregunta al usuario qué productos lo usan** antes de desactivar nada. No adivines.

## Paso 2: Buscar en Uber Eats

Usa `mcp__Uber_Eats__search` para confirmar que el producto existe en el local y obtener su nombre exacto. Para hacerlo necesitas la dirección de entrega; si no la tienes, pídela al usuario.

Esta herramienta **solo busca**: no permite desactivar productos. Por eso el resultado de Uber Eats se reporta como "pendiente de desactivar manualmente" hasta que exista una herramienta de edición de menú.

## Paso 3: Desactivar en los sitios web

Para adriasushi.cl y adriasangucheria.cl, la skill no tiene acceso de escritura al sitio desde este repo. Entrega al usuario la lista exacta de productos a ocultar o marcar como "agotado" en cada sitio.

Si el sitio vive en este repositorio (archivos HTML), busca los productos con `Grep` por nombre, márcalos como agotados, y publícalos en la rama designada.

## Paso 4: Registrar el estado

Actualiza `.claude/skills/producto-agotado/estado.json` para que quede claro qué está agotado y desde cuándo:

```json
{
  "reineta": {
    "agotado_desde": "2026-10-16",
    "uber_eats": "pendiente_manual",
    "adriasushi.cl": "pendiente_manual",
    "adriasangucheria.cl": "no_aplica"
  }
}
```

## Paso 5: Respuesta

Responde en español, en pocas líneas:

- Ingrediente marcado y fecha.
- Productos afectados por canal.
- Qué quedó hecho y qué falta hacer manualmente.
- Un recordatorio: "¿Quieres que lo reactive cuando vuelva el stock?"

Ejemplo:

> Reineta marcada como agotada (16-10-2026).
> - Uber Eats: 2 productos. Falta desactivarlos en el portal de comercio, la herramienta actual solo busca.
> - adriasushi.cl: 1 producto. Falta ocultarlo en el sitio.
> - adriasangucheria.cl: sin productos con reineta.

## Reglas

- Nunca reportes un producto como desactivado si no lo confirmaste en el canal correspondiente. Di "pendiente" cuando no puedas confirmarlo.
- No borres productos del mapa; solo cambia su estado.
- Para reactivar, sigue los mismos pasos con la acción `reactivar` y actualiza `estado.json`.
