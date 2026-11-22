---
trigger: always_on
---

# Regla General de Orquestación, Subagentes y Enrutamiento Multi-Modelo

Esta regla define cómo elegir el **modo de ejecución** (directo, subagente, workflow o enjambre) y **enrutar dinámicamente entre modelos y proveedores** según el riesgo, la complejidad y las suscripciones o cuotas disponibles en el entorno actual. Cuando se lance un **Swarm**, **Workflow** o **Subagente**, esta regla actúa como preferencia explícita del usuario para asignar perfiles y modelos en lugar de heredar siempre el mismo modelo del coordinador.

---

## 1. Descubrimiento dinámico de suscripciones, modelos y pools de cuota

Antes de abrir subagentes/workers o elegir un respaldo (*fallback*), inspeccionar qué modelos, proveedores y cuotas reales están activos en el entorno actual (por ejemplo, consultando la lista viva de modelos/cuotas del orquestador, el selector del IDE o las claves configuradas).

### Cómo identificar pools reales
- Agrupar modelos por **límite compartido y ventana de reinicio** (dos modelos de la misma familia o del mismo proveedor con idéntico porcentaje y hora de reinicio suelen pertenecer al mismo pool).
- **Regla de oro:** nunca hacer fallback hacia un modelo que comparta el mismo pool que se acaba de agotar.

### Adaptación según el tipo de suscripción disponible
1. **Entorno multi-pool / multi-proveedor:**
   - Clasificar dinámicamente los pools disponibles en tres roles:
     - **Pool de ventana corta / recarga rápida:** absorbe el trabajo diario (UI, exploración, refactors, ejecución rutinaria). Preferir esfuerzo `Medio` o `Ligero` para estirar la ventana.
     - **Pool independiente secundario:** segundo pilar para backend estándar, tareas medianas y **fallback principal** cuando el pool de ventana corta baje de **35%** o el pool premium esté protegido.
     - **Pool premium / cuota semanal escasa:** proteger con **reserva obligatoria del 35%** (reservar ese último 35% solo para bugs críticos, seguridad, migraciones de BD o solicitud explícita del usuario). Evitar usar modelos auxiliares que consuman del mismo pool premium.
   - **Regla de Pool Cruzado con el Coordinador:** el Coordinador activo ya consume su propio pool en cada turno al arrastrar todo el historial de la conversación. Si el Coordinador está en un pool con menor cuota (p. ej. < **60%** o con ≥ **20 puntos** menos que el pool secundario), asignar los workers al **pool independiente con mayor cuota** (incluida UI/frontend) para no drenar el pool del Coordinador por partida doble.
   - *Equilibrio activo:* si el pool de ventana corta baja de 35% y el secundario tiene > 50%, mover los siguientes workers al pool secundario hasta la recarga.
2. **Entorno con un solo proveedor (ej. solo Anthropic, solo OpenAI/Codex o solo Gemini):**
   - Balancear por **niveles de capacidad/costo** dentro de la misma familia: `Ligero/Rápido` → `Medio` → `Alto/Razonamiento`.
   - Reservar el nivel `Alto/Máximo` exclusivamente para planificación arquitectónica, seguridad, migraciones y bugs complejos; ejecutar el trabajo mecánico y estándar en `Medio` o `Ligero`.
3. **Suscripción de pago por uso (API directa sin cuota fija):**
   - Aplicar estrictamente **"Planifica alto, ejecuta medio, explora ligero"** para optimizar costo y latencia, limitando el paralelismo innecesario y el uso de revisores duplicados.
4. **Sin visibilidad de cuota restante en el entorno:**
   - Operar en **modo conservador**: arrancar workers en `Medio`, recolectar contexto en `Ligero`, reservar el nivel máximo para alto riesgo y reaccionar ante el primer `429` o límite cambiando de proveedor/pool o reduciendo paralelismo.

---

## 2. Evaluar 4 ejes antes de elegir modo (Umbral de Swarm, Cero Doble Lectura y Silencio)

1. **Riesgo / reversibilidad** *(prioridad sobre el tamaño)*:
   - Cambios en autenticación, pagos, seguridad, producción o migraciones de BD **nunca** se delegan a modelos ligeros ni degradan silenciosamente.
2. **Modo por defecto según el agente activo y peso del contexto:**
   - **Con agente orquestador (ciclo Plan → Ejecución):**
     - *Directo con el ejecutor:* solo cuando el cambio toca **1 sola capa** **y** **≤ 4 archivos** a editar (siempre que la conversación tenga **< 100 pasos**).
     - *Swarm obligatorio (2–3 workers, máx. 4):* cuando se toquen **≥ 2 capas o frentes independientes** (p. ej. Backend + Frontend), en lotes de **≥ 5 puntos/archivos**, o cuando la conversación principal ya supere **~100 pasos** (un worker nuevo arranca con 0 pasos de historial).
   - **Con modelo directo (sin orquestador):**
     - Usar **Modo Directo** por defecto; no abrir enjambres salvo que el usuario lo pida explícitamente.
   - **Prohibido duplicar lecturas entre Coordinador y Worker:**
     - Cuando una tarea va a Swarm, el Coordinador solo hace una lectura ligera de enrutamiento (identificar archivos afectados y contratos entre capas).
     - Si el Coordinador ya leyó un archivo, debe pegar las firmas, tipos o fragmentos exactos en el `brief` e instruir al worker a **no re-leer** archivos cuyo contrato ya esté incluido en el `brief`.
   - **Regla «Coordinador en silencio durante Swarm» (Cero Polling):**
     - Tras lanzar los workers (`add`), el Coordinador **termina su turno inmediatamente** y espera la notificación reactiva del board.
     - **Prohibido** hacer polling en bucle sobre el estado del enjambre, **prohibido** crear timers para vigilar workers, y **prohibido** quedarse editando archivos o prototipos/mapas en paralelo en la conversación principal (toda edición, incluido el mapa o scripts auxiliares, se delega a un worker).
     - **Confiar en la verificación del worker:** si el worker reporta `[done]` con tests y build limpios, cerrarlo (`kill`) de inmediato sin re-leer los archivos que editó.
   - **Paralelo vs. secuencial dentro de un equipo:**
     - *En paralelo (2–3 workers)* solo si son **100% independientes** (sin tocar los mismos archivos ni depender del contrato del otro) y pueden correr en **pools distintos** con cuota ≥ **40%**.
     - *Secuencial (uno por uno)* si existe dependencia (ej. backend → frontend) o cuota baja (< **40%**): cerrar (`kill`) el worker que termina antes de lanzar el siguiente.
   - **Sin revisor redundante en tareas normales:** reservar verificación cruzada solo para cambios críticos de alto riesgo o cierre final si el usuario lo solicita.
   - **Operación repetitiva sobre N ítems independientes** (auditorías masivas, búsquedas extensas, cambios mecánicos por lote) → **subagente ligero de investigación** o **workflow programado**.
3. **Tamaño de contexto:**
   - Archivos muy extensos, síntesis multi-archivo o tareas visuales/UI → priorizar un modelo `Medio` de **ventana de contexto amplia** y recarga rápida (respetando la regla de Pool Cruzado con el Coordinador).
4. **Complejidad:**
   - Aplicar **"Planifica alto, ejecuta medio"**. Todo worker de implementación arranca **obligatoriamente en esfuerzo `Medio`**.

---

## 3. Matriz de selección por tipo de tarea

> **Regla estricta de arranque:** todos los workers de implementación (`backend`, `frontend`, `seed`, `mapa`) arrancan en **`Medio`**. El perfil **`Alto`** se reserva únicamente para planificación, arquitectura/seguridad o escalado en la Sección 4 tras **2 fallos reales** en `Medio`.

| Tipo de tarea | Modo recomendado | Perfil de modelo y esfuerzo |
|---|---|---|
| Consultas rápidas, docs, scripts simples, boilerplate | Directo (o Workflow si son muchos archivos) | **Ligero / Rápido** (en pool secundario si el principal < 25%) |
| Exploración amplia de codebase | Subagente de lectura (`Ligero`) + síntesis (`Medio`) | Recolección en **Ligero**; síntesis arquitectónica en **Medio** |
| Desarrollo estándar (CRUD, APIs, refactors diarios) | Ejecutor directo (1 capa, ≤ 4 archivos) o Worker en Swarm | **Medio** en pool secundario o de recarga rápida (preservar pool premium) |
| Lógica de negocio crítica / contable / tests críticos | Ejecutor directo o Worker dedicado | **Medio** en pool de mayor razonamiento (≥ 35% si es escaso); nunca **Ligero** |
| Frontend / UI, tareas visuales, prototipos o contexto grande | Ejecutor directo (UI pequeña) o Worker en Swarm | **Medio** con contexto amplio (aplicando Pool Cruzado con el Coordinador; subir a **Alto** solo tras 2 fallos) |
| Arquitectura compleja o bugs difíciles | Plan previo → Ejecución enfocada | Plan en **Alto** (o mejor modelo disponible); ejecución en **Medio**; máximo esfuerzo solo tras reproducir con un test que falle |
| Incidente en producción | Directo en solo lectura (nunca Swarm) | Triage en **Medio**; escalar a **Alto** si no aparece causa raíz |
| Seguridad / auditoría | Workflow por módulo + validación cruzada | **Alto** con doble verificación en otra familia; modelo **Ligero** nunca da el veredicto final |
| Migraciones (BD / framework) | Plan + rollback (`Alto`) → lote (`Medio/Ligero`) → verificador | Planificación y verificación en **Alto**; ejecución mecánica por lotes |
| Infra / CI / IaC | Directo o Plan con revisión | **Alto** con *dry-run* obligatorio y confirmación del usuario |
| Rendimiento | Medir/perfilar antes de editar | Optimizar con perfil **Alto** |

---

## 4. Resiliencia y adaptación ante fallos ("ajustándose a lo que pase")

- **Ante `429` o límite de cuota / capacidad:**
  - **No subir el esfuerzo** ni reintentar a ciegas en el mismo pool.
  - Conmutar el worker o la siguiente etapa hacia el **siguiente pool independiente disponible** manteniendo el nivel `Medio` (conservando su conversación si la herramienta lo permite, sin matar ni perder el contexto).
- **Ante caída del proveedor, timeout o modelo no disponible/retirado:**
  - Seleccionar automáticamente el perfil equivalente del pool sano disponible.
  - Si **todos** los pools adecuados están agotados o caídos, detener tareas nuevas, mostrar qué cuota/reinicio queda en cada pool y preguntar al usuario si prefiere esperar o cambiar de estrategia.
- **Ante errores técnicos persistentes, tests que fallan o bucles:**
  - Escalar esfuerzo (`Ligero → Medio → Alto → Máximo`) únicamente después de **2 intentos fallidos** con el nivel actual.
- **Freno obligatorio en tareas críticas:**
  - En autenticación, pagos, seguridad, producción o migraciones de BD, **pausar y avisar al usuario** antes de degradar a un modelo `Ligero` por falta de cuota.

---

## 5. Comportamiento en Modo Orquestado, alertas de selector e higiene de contexto

- **Con Orquestador activo / Swarm / Workflows / Subagentes:**
  1. Consultar modelos y cuotas vivas antes de lanzar workers.
  2. Aplicar el balanceo y fallbacks de esta regla automáticamente sin pedir confirmación previa por cada worker.
  3. Informar brevemente al usuario el equipo asignado, el modelo/pool elegido y la cuota restante (si el entorno la expone).
- **Alerta temprana de cambio de selector (en Modo Orquestado y en Modo Directo):**
  - Siempre que el pool del modelo u orquestador activo en el selector baje de **35%** (o llegue a **0%**) habiendo otro pool disponible con **≥ 50%**, incluir al final de la respuesta una alerta de **1 línea** recomendando cambiar al orquestador o modelo de ese pool.
- **Higiene de conversaciones largas:**
  - Cuando una conversación supere **~150 pasos**, sugerir en **1 línea** compactar el contexto (`/compact`) o abrir una sesión nueva para no re-enviar cientos de pasos en cada turno del coordinador.
