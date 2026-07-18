# GymBro

PWA mobile-first para seguir un plan de entrenamiento de 26 semanas en el gimnasio: sesión del día, registro serie a serie (peso, reps, RIR), timer de descanso, lógica de progresión, peso objetivo manual, notas de sesión y export/import de progreso.

Todo funciona **client-side**: el plan y el historial viven en `localStorage` del dispositivo. Sin backend, sin cuentas.

## Publicar en GitHub Pages

1. Crea un repositorio (público) y sube estos archivos a la raíz:
   `index.html · manifest.json · sw.js · icon-192.png · icon-512.png`
2. En el repo: **Settings → Pages → Source: Deploy from a branch → Branch: main / (root) → Save**
3. En 1-2 minutos tendrás la app en `https://TU_USUARIO.github.io/NOMBRE_REPO/`

## Instalar como app en el móvil

- **Android (Chrome):** abre la URL → menú ⋮ → *Añadir a pantalla de inicio* (o el aviso de instalación).
- **iOS (Safari):** abre la URL → botón compartir → *Añadir a pantalla de inicio*.

Tras la primera visita, el service worker cachea todo y la app **funciona sin conexión**.

## Migrar datos desde la versión local (file://)

`localStorage` es por origen: los datos guardados abriendo el archivo local no aparecen en la URL de Pages.

1. En la versión antigua: menú ⋮ → **Exportar todo el historial**
2. En la nueva URL: carga el plan JSON y luego menú ⋮ → **Importar progreso**

## Flujo de revisión con IA

1. Al acabar de entrenar, escribe tus sensaciones en **Notas de la sesión**.
2. Menú ⋮ → **Exportar última sesión** → genera `ultima_sesion_YYYY-MM-DD.json` con ejercicios, series (peso/reps/RIR) y tus notas.
3. Pásale ese archivo a tu asistente para valorar la sesión y planificar la siguiente.
4. Con la recomendación, fija el peso con **✎ Fijar peso objetivo** en cada ejercicio: la app lo pre-rellenará en la serie 1 de la próxima sesión.

## Actualizar el plan

Botón 🗑️ → borra solo el plan (el historial se conserva) → sube el nuevo `plan_entrenamiento.json`.

# Formato del JSON de plan de entrenamiento

Este documento describe la estructura que debe cumplir un fichero JSON para que la aplicación lo cargue correctamente. Si al importar aparece el error `El JSON no tiene la estructura esperada (faltan "weeks" o "plan_start")`, es porque uno de los campos obligatorios de nivel raíz está ausente o mal escrito.

## Campos obligatorios (nivel raíz)

Estos campos son **imprescindibles**. Sin ellos, la aplicación rechazará el fichero.

| Campo | Tipo | Descripción |
|---|---|---|
| `plan_start` | string | Fecha de inicio del plan en formato ISO `YYYY-MM-DD` (ej. `"2026-07-20"`). |
| `weeks` | array | Array con **una entrada por semana** del plan. Cada semana debe seguir la estructura descrita más abajo. |

## Campos recomendados (nivel raíz)

Aunque la aplicación no falla sin ellos, se recomienda incluirlos para que la vista muestre información completa.

| Campo | Tipo | Descripción |
|---|---|---|
| `goal` | string | Objetivo general del plan (ej. `"Recomposición corporal · 10km"`). |
| `version` | string | Versión del plan (ej. `"v1.0"`) — útil para changelog. |
| `athlete` | object | Datos del atleta: edad, peso, altura, medicación, suplementación, etc. |
| `plan_end` | string | Fecha de fin del plan en formato ISO. |
| `plan_weeks` | integer | Número total de semanas del plan (debe coincidir con `weeks.length`). |
| `sessions_per_week` | object | Objeto con `mandatory` (int), `optional` (int) y `breakdown` (string). |
| `week_structure` | object | Descripción de cada día de la semana (`monday`, `tuesday`, etc.). |
| `nutrition` | object | Kcal por tipo de día, proteína/día, déficit semanal aproximado. |
| `deload_rule` | string | Explicación de en qué semanas se hace deload. |

## Estructura de cada semana (`weeks[i]`)

Cada elemento del array `weeks` debe contener:

### Campos obligatorios en cada semana

| Campo | Tipo | Descripción |
|---|---|---|
| `week_num` | integer | Número de semana (1, 2, 3…). |
| `days` | array | Array de **exactamente 7 días** (día 0 = lunes, día 6 = domingo). |

### Campos recomendados en cada semana

| Campo | Tipo | Descripción |
|---|---|---|
| `phase` | string | Nombre de la fase o mesociclo (ej. `"Mesociclo 2 — Intensificación"`). |
| `is_deload` | boolean | `true` si esa semana es de descarga, `false` si no. |
| `tss_target` | integer | Carga acumulada semanal orientativa. |
| `sessions_mandatory` | integer | Nº de sesiones obligatorias esa semana. |
| `nutrition` | object | Kcal por día de la semana + macros. |
| `objective` | string | Objetivo específico de esa semana. |

## Estructura de cada día (`weeks[i].days[j]`)

Cada día del array debe tener al menos:

| Campo | Tipo | Descripción |
|---|---|---|
| `day` | integer | 0 = lunes, 1 = martes, 2 = miércoles, 3 = jueves, 4 = viernes, 5 = sábado, 6 = domingo. |
| `type` | string | Tipo de sesión: `"strength"`, `"mobility"`, `"easy_run"`, `"rest"`, `"rest_active"`. |
| `label` | string | Título corto del día. |
| `description` | string | Descripción de la sesión. |
| `duration` | string | Duración estimada (ej. `"55 min"`). |

### Campos adicionales según el tipo de día

- **Si `type` es `"strength"` o `"easy_run"`** → debe incluir `phases` (array de fases del entreno).
- **Si `type` es `"mobility"`** → puede incluir `activities` (array de bloques con sus propias `phases`).
- **Si `type` es `"rest"`** → puede incluir `optional_run` u `optional_cardio` con la misma estructura que una sesión.

### Estructura de cada fase (`phases[k]`)

Cada objeto dentro de `phases` debe tener:

| Campo | Tipo | Descripción |
|---|---|---|
| `n` | string | Número de la fase (`"1"`, `"2"`, `"3"`…). |
| `desc` | string | Descripción del ejercicio o bloque (incluir series, reps, RIR, descansos entre corchetes). |
| `time` | string | Tiempo estimado de esa fase (ej. `"8 min"`, `"14 min"`). |

## Ejemplo mínimo válido

Este es el JSON más pequeño que la aplicación aceptará sin error:

```json
{
  "plan_start": "2026-07-20",
  "weeks": [
    {
      "week_num": 1,
      "days": [
        {
          "day": 0,
          "type": "strength",
          "label": "Full Body A",
          "description": "Glúteo + femoral + hombro",
          "duration": "55 min",
          "phases": [
            {
              "n": "1",
              "desc": "Calentamiento + activación glúteo",
              "time": "8 min"
            },
            {
              "n": "2",
              "desc": "Hip thrust — 3×12 [RIR 3] [descanso 90s]",
              "time": "12 min"
            }
          ]
        },
        { "day": 1, "type": "rest", "label": "Descanso", "description": "Descanso completo", "duration": "0" },
        { "day": 2, "type": "rest", "label": "Descanso", "description": "Descanso completo", "duration": "0" },
        { "day": 3, "type": "rest", "label": "Descanso", "description": "Descanso completo", "duration": "0" },
        { "day": 4, "type": "rest", "label": "Descanso", "description": "Descanso completo", "duration": "0" },
        { "day": 5, "type": "rest", "label": "Descanso", "description": "Descanso completo", "duration": "0" },
        { "day": 6, "type": "rest", "label": "Descanso", "description": "Descanso completo", "duration": "0" }
      ]
    }
  ]
}
```

## Errores comunes

- **`El JSON no tiene la estructura esperada (faltan "weeks" o "plan_start")`** → falta uno de los dos campos raíz obligatorios, o están mal escritos (revisa mayúsculas/minúsculas).
- **`plan_start` con formato incorrecto** → debe ser `YYYY-MM-DD`, no `DD/MM/YYYY` ni otros formatos.
- **`weeks` vacío o no array** → debe ser un array `[]`, aunque tenga una sola semana.
- **Menos de 7 días por semana** → cada semana debe tener 7 objetos en `days` (día 0 al 6), aunque algunos sean de descanso.
- **`phases` como objeto en vez de array** → siempre debe ser un array `[]`, aunque tenga una sola fase.

## Valores permitidos para `type` (en days)

| Valor | Significado |
|---|---|
| `strength` | Sesión de fuerza. Requiere `phases`. |
| `mobility` | Sesión de movilidad. Puede tener `activities` con `phases` dentro. |
| `easy_run` | Rodaje suave / carrera. Requiere `phases`. |
| `rest` | Descanso completo. `phases` opcional. Puede incluir `optional_run`. |
| `rest_active` | Descanso activo (caminata, movilidad ligera). |

## Notas adicionales

- Los campos `chart_type`, `nutrition_kcal`, `notes` a nivel día son opcionales pero recomendados para vista rica.
- Se pueden añadir campos personalizados a nivel raíz sin romper la carga (la app ignora los que no reconoce).
- Para planes con protocolos especiales (rehabilitación, HSR, protocolo tendón, etc.), añade objetos independientes a nivel raíz como `pull_up_progression_protocol`, `triceps_tendon_protocol`, etc.
