# Error Patterns — AIDLC

Catálogo de errores recurrentes, sus causas raíz y cómo resolverlos.

## Formato de Entrada

```markdown
### [ID] Nombre del patrón

**Contexto:** Dónde y cuándo aparece este error.
**Causa raíz:** Por qué ocurre, no solo qué ocurre.
**Solución aplicada:** Pasos exactos para resolverlo.
**Señales tempranas:** Cómo detectarlo antes de que explote.
**Veces registrado:** N
**¿Skill creado?:** Sí / No — [link si existe]
```

---

## Patrones Registrados

---

### EP-001 — WebFetch falla con repos GitHub

**Contexto:** El agente intenta leer archivos de un repo GitHub usando `WebFetch`
con URLs del tipo `https://github.com/...` o `https://raw.githubusercontent.com/...`.

**Síntoma:** Error 404 o respuesta vacía, incluso en repos públicos.

**Causa raíz:** `WebFetch` no maneja la autenticación OAuth ni los redirects
de GitHub correctamente. GitHub redirige según el estado de autenticación
del cliente, y `WebFetch` no tiene sesión de GitHub.

**Solución aplicada:** Usar `gh` CLI con decodificación base64:

```bash
# Leer un archivo de cualquier repo (público o privado con acceso)
gh api repos/{owner}/{repo}/contents/{path} --jq '.content' | base64 -d

# Ejemplo real — AIDLC FOUNDATION.md:
gh api repos/felobp-arch/aidlc/contents/FOUNDATION.md --jq '.content' | base64 -d

# Listar archivos de un directorio:
gh api repos/{owner}/{repo}/contents/{dir} --jq '.[].name'
```

**Señales tempranas:**
- La URL contiene `github.com` o `raw.githubusercontent.com`
- El agente ya intentó `WebFetch` y obtuvo 404 o HTML de login

**Regla mnemónica:** `github.com` → `gh api`. Siempre. Sin excepción.

**Veces registrado:** 1 (sesión 2026-04-08, proyecto Security Focus OS)  
**¿Skill creado?:** No — la solución es un one-liner, no requiere skill

---

### EP-002 — Zod v4: `.trim()` después de `.min()` no rechaza whitespace-only

**Contexto:** Schema Zod para validar strings de entrada de usuario. Se espera
que un string de solo espacios (`'   '`) sea rechazado.

**Síntoma:** `schema.safeParse({ field: '   ' })` retorna `success: true`.
El valor en `result.data.field` es `''` (string vacío tras el trim).

**Causa raíz:** En Zod (v3 y v4), el orden de los métodos importa.
`z.string().min(1).trim()` evalúa `min(1)` sobre el valor **antes** del trim.
`'   '` tiene 3 caracteres → pasa `min(1)` → se transforma a `''`.
El resultado final es un string vacío que pasó la validación de longitud.

**Solución aplicada:** Siempre poner `.trim()` **antes** de `.min()`:

```typescript
// ❌ Incorrecto — acepta strings solo-whitespace
z.string().min(1).max(200).trim()

// ✅ Correcto — rechaza strings solo-whitespace
z.string().trim().min(1).max(200)
```

**Regla mnemónica:** "Transforma primero, valida después."

**Señales tempranas:**
- El schema tiene `.trim()` al final de la cadena
- Tests de validación no cubren el caso `'   '` (whitespace-only)

**Veces registrado:** 1 (sesión 2026-04-08, proyecto Security Focus OS)  
**¿Skill creado?:** No — corrección de una línea en el schema

---

### EP-003 — Boundaries de fechas: midnight-to-midnight ≠ días calendario

**Contexto:** Tests unitarios con lógica de proporción de tiempo transcurrido
(elapsed_ratio) entre dos fechas ISO 8601.

**Síntoma:** Un test de boundary falla porque el valor calculado en el test
no coincide con el valor calculado en la función. El threshold real es
ligeramente distinto al esperado.

**Causa raíz:**
`new Date('2026-12-31').getTime() - new Date('2026-01-01').getTime()`
= **364 días** (midnight a midnight), no 365.

El año 2026 tiene 365 días, pero la diferencia entre el primer instante de
Jan 1 y el primer instante de Dec 31 es solo 364 días. El día 31 de diciembre
no está "contenido" en la diferencia — solo su inicio.

**Solución aplicada:**
- Para tests de comportamiento general: usar valores claramente por encima/debajo del threshold (margen de ±5%)
- Para tests de boundary exacto: calcular el ratio programáticamente en el test:

```typescript
const cycleMs = new Date('2026-12-31').getTime() - new Date('2026-01-01').getTime()
const elapsedMs = new Date('2026-07-01').getTime() - new Date('2026-01-01').getTime()
const ratio = elapsedMs / cycleMs  // 181/364 = 0.4973...
const expected = ratio * targetValue
const greenThreshold = expected * 0.70  // calcular, no asumir
```

**Señales tempranas:**
- Test de boundary que falla por 0.1–0.5 en un valor numérico
- El test asume un número redondo de días entre dos fechas de año completo

**Veces registrado:** 1 (sesión 2026-04-08, proyecto Security Focus OS)  
**¿Skill creado?:** No

---

> Última actualización: 2026-04-08  
> Total de patrones: 3 (EP-001, EP-002, EP-003)  
> Skills generados: 0
