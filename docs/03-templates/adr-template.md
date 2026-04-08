# ADR — Architecture Decision Record Template

> Un ADR documenta una decisión de arquitectura significativa: qué se decidió,
> por qué, y cuáles son las consecuencias. Son artefactos permanentes.
>
> Guardar en: `docs/decisions/ADR-XXXX-titulo-descriptivo.md`

---

## ADR-[XXXX] — [Título descriptivo de la decisión]

**Fecha:** YYYY-MM-DD  
**Estado:** [Propuesta | Aceptada | Rechazada | Deprecada | Supersedida por ADR-YYYY]  
**Decisores:** [nombres o roles]  
**Contexto del proyecto:** [FHC / Nombre del proyecto]

---

## Contexto

[Describe la situación que hace necesaria esta decisión. Qué problema existe,
qué fuerzas están en tensión, qué restricciones son relevantes.

Escribe en tiempo presente: "El sistema necesita...", "Actualmente..."]

---

## Decisión

[Describe la decisión tomada en voz activa y tiempo presente.

"Usaremos X porque..."
"Adoptaremos el patrón Y para..."
"No implementaremos Z porque..."]

---

## Opciones Consideradas

### Opción A: [nombre] ← Elegida

**Descripción:** [qué es esta opción]

**Pros:**
- Pro 1
- Pro 2

**Contras:**
- Contra 1
- Contra 2

---

### Opción B: [nombre]

**Descripción:** [qué es esta opción]

**Pros:**
- Pro 1

**Contras:**
- Contra 1 — [razón por la que esta la descarta]

---

### Opción C: [nombre] (si aplica)

**Descripción:** [qué es esta opción]

**Pros:**
- Pro 1

**Contras:**
- Contra 1

---

## Consecuencias

### Positivas
- [Beneficio directo de esta decisión]
- [Otro beneficio]

### Negativas / Trade-offs
- [Costo o limitación que acepta esta decisión]
- [Deuda técnica que introduce, si la hay]

### Neutrales
- [Cosas que cambian pero no son ni buenas ni malas en sí mismas]

---

## Implicaciones de Implementación

[Qué hay que hacer como resultado de esta decisión.
Puede incluir: archivos a crear, patrones a seguir, convenciones a adoptar.]

```
Ejemplo de estructura o código que ilustra la decisión:

src/
├── services/   ← lógica aquí siempre
├── api/        ← solo controllers
└── db/         ← solo adapters
```

---

## Compliance y Seguridad

[Si aplica: implicaciones de esta decisión en términos de seguridad,
regulación (SFC Circular 007), o auditoría.]

---

## Referencias

- [Link a SRD relacionado]
- [Link a ADR que esta supersede, si aplica]
- [Artículo o documentación que fundamenta la decisión]
- [Ticket o issue relacionado]

---

## Historial de Cambios

| Fecha | Cambio | Autor |
|-------|--------|-------|
| YYYY-MM-DD | Creado | [nombre] |
| YYYY-MM-DD | Estado cambiado a Aceptada | [nombre] |
