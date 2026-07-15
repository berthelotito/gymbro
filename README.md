# Gym Tracker

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
