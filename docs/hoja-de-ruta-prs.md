# Plata sin fronteras: hoja de ruta de PRs

Guía para repartir el trabajo, revisar avances y marcar metas cumplidas.

- **Entrega:** domingo **2026-11-29, 23:59** (hora local, UTC-5).
- **Fechas:** las de este documento reemplazan las del [spec](superpowers/specs/2026-11-27-plata-sin-fronteras-demo-design.md) (martes 1 de diciembre) y las del [plan](superpowers/plans/2026-11-27-plata-sin-fronteras-demo.md) (lunes 30).
- **PR-N = Tarea N del plan.** PR-01 a PR-12 son los identificadores de esta guía, no los números de GitHub.

## 1. Metas y responsables

Las metas son las cuatro de la sección 3 del spec, en su orden.

| Meta | Qué es | Responsable | PRs |
|---|---|---|---|
| 1 | Una demo que corre la historia completa, confiable y en vivo | Ing. 1 | 01, 02, 05, 06, 10, 11, 12 |
| 2 | Un motor de reglas real y probado para el vigilante, sin dependencias del framework | Ing. 1 | 03, 04 |
| 3 | Una explicación visible de por qué el pago es rápido y por qué el vigilante decidió así | Ing. 2 | 09 |
| 4 | Una interfaz para una persona de 68 años y otra de 29 | Ing. 2 | 07, 08 |

**Ing. 1** es el otro ingeniero y corresponde al I1 del plan (dominio y API). **Ing. 2** corresponde al I2 (interfaz).

Excepciones, según el plan: **PR-11** lo toma quien se libere primero y **PR-12** lo hacen los dos. La carga del Ing. 1 es bastante mayor, así que el Ing. 2 debería ofrecerse para PR-11.

## 2. Tablero

Cada PR marca **su propia casilla** (`[x]`) en la misma rama, antes del merge. Si dos marcas chocan al actualizar la rama, se conservan ambas. La casilla de una meta se marca en el último PR que la completa.

### Meta 1: historia completa en vivo
- [ ] PR-01 · Scaffold y verificación · P0
- [ ] PR-02 · Estado, despacho y API · P0
- [ ] PR-05 · Pagos, aviso, pausa y retención · P0
- [ ] PR-06 · Pago por reclamar y momentos del guion · P0
- [ ] PR-10 · Reporte de estafa · P1
- [ ] PR-11 · Asesor · P2
- [ ] PR-12 · README y guion · P1
- [ ] **Meta 1 cumplida:** los momentos 1 a 4 corren seguidos en la interfaz en 3 minutos o menos, PR-12 está mergeado, PR-10 está mergeado o cortado, y el video está grabado

### Meta 2: vigilante
- [ ] PR-03 · Motor de reglas · P0
- [ ] PR-04 · Jev simulado y política de la pausa · P0
- [ ] **Meta 2 cumplida:** las pruebas pasan, cada momento del guion cae en su nivel y el vigilante no importa nada del framework

### Meta 3: explicación visible
- [ ] PR-09 · Grafo del pago y vista del vigilante · P0
- [ ] **Meta 3 cumplida:** el momento 3 muestra 740 ms y el momento 4 abre la vista con las reglas y la decisión

### Meta 4: interfaz para ambos
- [ ] PR-07 · Escenario, teléfonos y chat · P0
- [ ] PR-08 · Niveles, tarjetas y voz · P0
- [ ] **Meta 4 cumplida:** los dos niveles se ven distintos, la voz funciona en Chrome y siempre se puede escribir

## 3. Calendario

| Hora límite | Ing. 1 | Ing. 2 | Punto de control |
|---|---|---|---|
| Vie 27, 23:00 | PR-01, PR-02 | Probar voz es-CO en el laptop de la demo; preparar estilos | La interfaz ya puede programar contra la API |
| Sáb 28, 13:00 | PR-03, PR-04 | PR-07 | Vigilante probado y escenario visible |
| Sáb 28, 19:00 | PR-05 | PR-08 | Un pago se envía desde la interfaz |
| Sáb 28, 23:59 | PR-06 | PR-09 | **Momentos 1 a 4 de punta a punta** |
| Dom 29, 13:00 | PR-10 | Ajustes de integración; borrador del pitch | Momento 5 funciona |
| Dom 29, 16:00 | PR-11 (si aplica) | PR-11 (si aplica) | **Congelar funciones** |
| Dom 29, 20:00 | PR-12 | PR-12 | Ensayo, video y pitch listos |
| Dom 29, 21:00 | Enviar | Enviar | Margen hasta las 23:59 |

**Reglas de corte:**
1. Si el sábado a las 23:59 los momentos 1 a 4 no corren, se cancela PR-11 y todo el esfuerzo va a P0.
2. Si el domingo a las 12:00 falta mergear algún P0, se cancela PR-11.
3. Si el domingo a las 14:00 PR-10 no está mergeado, se cancela y el momento 5 va solo en el pitch.
4. Al cancelar PR-10 o PR-11, hay que **ocultar** su acción rápida ("Me estafaron" o "Quiero ahorrar"), porque daría error.

## 4. Reglas para todos los PRs

**Flujo:**

```bash
git switch main && git pull --ff-only
git switch -c <rama del PR>
# ... trabajo ...
npm run verify
```

- Un PR por tarea, hacia `main`, con el título exacto de esta guía. Merge con **squash**.
- Para el merge: `npm run verify` en verde (local y CI), revisión del otro ingeniero y casilla marcada.
- Revisiones rápidas: avisar al abrir el PR y revisar en unos 30 minutos.
- `npm run verify` existe desde PR-01. Se usa `npm`, como fija el plan.
- Código, comentarios y commits en inglés; lo que lee la persona usuaria, en español.

**No negociable:**
- Ninguna IA mueve, congela ni libera plata. Una retención nunca mueve plata.
- La retención preventiva tras un reporte solo se pone en cola para revisión humana.
- Solo datos ficticios. El repo es público.
- Nada simulado se presenta como real o medido.
- No hay texto generado: toda frase visible es fija.
- Dependencias: solo `next`, `react`, `react-dom`, `typescript`, `@types/node`, `@types/react` y `@types/react-dom`. Cualquier otra requiere aprobación.
- `src/domain/**` no importa `next`, `react`, `src/app` ni `src/ui`.
- Los imports relativos llevan `.ts` y los tipos se importan con `import type`.

**Para no chocar:** el Ing. 1 trabaja en `src/domain/` y `src/app/api/`; el Ing. 2, en `src/ui/`, `src/app/page.tsx`, `src/app/layout.tsx` y `src/app/globals.css`.

## 5. PRs del Ing. 1 (metas 1 y 2)

Cómo leer cada ficha:
- **Debe cumplir:** obligatorio; es lo que se revisa.
- **Coordinar:** lo usa la interfaz. Se puede cambiar, pero avisando antes al Ing. 2.
- **Libertad:** decides tú. El código y las pruebas del plan son una **propuesta, no obligatoria**; úsalos si te ahorran tiempo.

Aplica además el "Foco de revisión" del plan: cada falla de esa lista necesita su prueba.

### PR-01 · `chore(repo): scaffold Next.js app with verify pipeline`
`chore/scaffold-verify` · Meta 1 · P0 · Sin dependencias · [Tarea 1](superpowers/plans/2026-11-27-plata-sin-fronteras-demo.md#tarea-1-scaffold-y-verificación-i1)

- **Debe cumplir:**
  - `npm run verify` corre typecheck, pruebas y build, y pasa también desde cero (`rm -rf .next`).
  - CI corre ese mismo comando en `push` y `pull_request`.
  - Dinero sin decimales flotantes, tasa ficticia de 4.000 COP por dólar y montos válidos solo si son enteros positivos.
- **Coordinar:** `Money`, `formatMoney`, `FX_USD_COP`.
- **Libertad:** estructura de la configuración y pruebas adicionales.
- **Ojo:** el plan dice partir de `docs/demo-design`; ahora se parte de `main`. `.gitignore` ya existe: **agregar** `node_modules/`, `.next/` y `*.tsbuildinfo` sin borrar líneas.

### PR-02 · `feat(domain): add demo state, dispatch and API route`
`feat/demo-state-api` · Meta 1 · P0 · Depende de 01 · [Tarea 2](superpowers/plans/2026-11-27-plata-sin-fronteras-demo.md#tarea-2-tipos-estado-inicial-despacho-y-api-i1)

- **Debe cumplir:**
  - Estado inicial ficticio con los usuarios, llaves e historial del plan; "reiniciar" vuelve a él.
  - La misma acción repetida (doble clic) se aplica una sola vez.
  - `GET /api/demo` devuelve el estado; `POST` valida la acción y responde 400 si es inválida.
- **Coordinar:** todo `types.ts` (estado, acciones, mensajes, cards) y la forma de la API.
- **Libertad:** organización interna del despacho y del estado en memoria.
- **Orientación:** **este PR desbloquea al Ing. 2**, así que va primero. Si adelantas la constante `PAYMENT_STEPS`, desbloqueas también el grafo.

### PR-03 · `feat(watchdog): add deterministic rules engine`
`feat/watchdog-rules` · Meta 2 · P0 · Depende de 02 · [Tarea 3](superpowers/plans/2026-11-27-plata-sin-fronteras-demo.md#tarea-3-vigilante-i1)

- **Debe cumplir:**
  - Función pura `evaluate(payment, context) → { level, score, reasons[] }` con las reglas R1 a R7, T1 y T2 del spec §6.5.
  - Pesos y umbrales en una sola tabla de configuración.
  - Momento 1: `allow`. Momento 4: `pause`, puntaje 70, razones R1, R2, R3, R4 y R7. Momento 5: `notice`. Primer pago a una llave confirmada: `hold`.
  - Cada razón tiene una frase fija para nivel sencillo y compacto.
  - `JEV_LABEL` exacto, como está en el plan.
- **Coordinar:** `REASON_COPY`, `JEV_LABEL` y cualquier cambio de pesos que altere el puntaje 70.
- **Libertad:** implementación de las reglas, redacción de las frases y ajuste de pesos mientras cada momento caiga en su nivel.

### PR-04 · `feat(jev): add scripted System One judgments and pause policy`
`feat/scripted-jev-policy` · Meta 2 · P0 · Depende de 02 · [Tarea 4](superpowers/plans/2026-11-27-plata-sin-fronteras-demo.md#tarea-4-jev-simulado-y-política-de-la-pausa-i1)

- **Debe cumplir:**
  - Pedidos y respuestas con la forma de la API pública de System One (leer docs.typesafe.ai), sin SDK ni llamadas de red.
  - Una sola función obtiene los juicios, para cambiarla por la llamada real después.
  - Coincidencia por palabras clave. "me llamó uno de Belo diciendo que mi cuenta estaba en peligro" da suplantación 0,94.
  - Texto sin coincidencia: decide solo con reglas y muestra de nuevo los botones.
  - Política: "Sí" → `hold`; "No" → confirmar; suplantación ≥ 0,8 → `hold`; estafa distinta de `none` con probabilidad ≥ 0,7 → `hold`.
- **Coordinar:** `describeDecision` (ejemplo: "Suplantación 0,94 ≥ 0,8 → retención").
- **Libertad:** escenarios adicionales, palabras clave y demás probabilidades.

### PR-05 · `feat(payments): add send flow with watchdog levels`
`feat/payment-flow` · Meta 1 · P0 · Depende de 03 y 04 · [Tarea 5](superpowers/plans/2026-11-27-plata-sin-fronteras-demo.md#tarea-5-pagos-aviso-pausa-y-retención-i1)

- **Debe cumplir:**
  - Los cuatro niveles se comportan como en el spec §6.5. Pausa y retención no mueven plata.
  - Orden de pasos `fx → watchdog → aml → debit → credit`; un pago completo suma 740 ms.
  - Montos inválidos, saldo insuficiente o llave desconocida: mensaje, sin pago y sin tocar saldos.
  - Una retención nunca se puede confirmar, pero **sí cancelar**. El plan se contradice aquí (su botón "Cancelar pago" no haría nada); manda el spec.
  - El contacto de confianza recibe aviso al retener.
- **Coordinar:** `PAYMENT_STEPS`, estados del pago y estados de cada paso de la traza.
- **Libertad:** funciones internas y redacción de los mensajes.

### PR-06 · `feat(claims): add claimable payments and demo beats`
`feat/claimable-payments` · Meta 1 · P0 · Depende de 05 · [Tarea 6](superpowers/plans/2026-11-27-plata-sin-fronteras-demo.md#tarea-6-pago-por-reclamar-dónde-guardar-y-momentos-del-guion-i1)

- **Debe cumplir:**
  - Enviar a un número sin cuenta no se rechaza: queda por reclamar y se detiene a los 540 ms.
  - Pregunta de nivel, código de un solo uso, registro, crédito y elección entre pesos y dólares.
  - Código equivocado no acredita; reclamar dos veces acredita una sola vez.
  - Los momentos 1, 3, 4 y 5 funcionan desde los controles del presentador, sin duplicar envíos.
  - Una prueba recorre la historia completa hasta la pausa del momento 4.
- **Coordinar:** nombres de los momentos y acciones nuevas en los botones.
- **Libertad:** estructura del flujo de reclamo y textos.

### PR-10 · `feat(report): add scam report with compliance review queue`
`feat/scam-report` · Meta 1 · P1 · Depende de 06 · [Tarea 10](superpowers/plans/2026-11-27-plata-sin-fronteras-demo.md#tarea-10-reporte-de-estafa-i1-p1)

- **Debe cumplir:**
  - Del relato se arma un caso que la persona revisa, corrige y envía.
  - Reconoce `$150.000`, `150 mil` y `150000`; con el monto en palabras usa el último pago.
  - Al enviar: cola de revisión humana, llave marcada como reportada, borrador de denuncia y aviso al contacto. **No mueve saldos.**
  - Enviar dos veces no duplica el caso en la cola.
- **Coordinar:** campos del caso que muestran las cards `case` y `complaint`.
- **Libertad:** forma de extraer los datos y texto de la denuncia.

### PR-11 · `feat(advisor): add progressive risk profile and savings plan`
`feat/savings-advisor` · Meta 1 · P2 · Depende de 05 y 08 · Quien se libere primero · [Tarea 11](superpowers/plans/2026-11-27-plata-sin-fronteras-demo.md#tarea-11-asesor-p2-quien-se-libere-primero)

- **Debe cumplir:**
  - Pregunta de perfil, perfil visible que se ajusta respondiendo otra pregunta, simulador con tasas ficticias y sin pronósticos.
  - Aviso visible: es educación y simulación, no asesoría de inversión personalizada.
  - Solo mueve plata cuando la persona confirma.
  - La card `savings-plan` se ve en el chat.
- **Coordinar:** datos de la card `savings-plan`.
- **Libertad:** preguntas, perfiles y presentación del simulador.

## 6. PRs del Ing. 2 (metas 3 y 4)

El detalle de pantallas y firmas está en el plan.

### PR-07 · `feat(ui): add demo stage with phones and chat`
`feat/demo-stage-chat` · Meta 4 · P0 · Depende de 02 · [Tarea 7](superpowers/plans/2026-11-27-plata-sin-fronteras-demo.md#tarea-7-escenario-teléfonos-y-chat-i2-desde-que-esté-la-tarea-2)

- Los dos teléfonos y el espacio del grafo en una sola página, legible en proyector.
- El chat dibuja mensajes, botones (desaparecen al usarse), entrada de texto y etiqueta de Jev.
- Controles del presentador: momentos 1, 3, 4, 5 y "Reiniciar".
- La prueba de punta a punta con envío se completa cuando estén PR-05 y PR-06.

### PR-08 · `feat(ui): add adaptive levels, cards and voice input`
`feat/ui-levels-cards-voice` · Meta 4 · P0 · Depende de 07 · [Tarea 8](superpowers/plans/2026-11-27-plata-sin-fronteras-demo.md#tarea-8-niveles-tarjetas-y-voz-i2)

- Nivel sencillo (letra ≥ 20 px, botones anchos) y compacto, cambiables en vivo.
- Cards de frascos, link, SMS, caso y denuncia. La de `savings-plan` puede completarse en PR-11.
- Voz `es-CO` en Chrome; sin soporte o sin permiso no aparece el micrófono. Escribir siempre es posible.
- Estilo cálido y tranquilo, sin colores de alarma.

### PR-09 · `feat(ui): add payment graph and watchdog view`
`feat/payment-graph-view` · Meta 3 · P0 · Depende de 05 y 07 · [Tarea 9](superpowers/plans/2026-11-27-plata-sin-fronteras-demo.md#tarea-9-grafo-del-pago-y-vista-del-vigilante-i2)

- Grafo con los cinco pasos, contador rotulado "(simulado)" y carril de compensación diferida.
- Momento 1 se detiene en "Esperando a Rosa" (540 ms); momento 3 llega a 740 ms.
- Momento 4 se frena en Vigilante y muestra reglas, puntaje, juicios con su etiqueta y la línea que decidió.

## 7. PR compartido

### PR-12 · `docs(readme): rewrite README and add demo script`
`docs/readme-demo-script` · Meta 1 · P1 · Depende de todos los que entren · Ambos · [Tarea 12](superpowers/plans/2026-11-27-plata-sin-fronteras-demo.md#tarea-12-readme-guion-y-video-de-respaldo-los-dos-lunes-2026-11-30)

- README en español: qué es, para quién, historia, cómo correrlo, arquitectura, qué es simulado y qué no hace.
- Describe solo lo construido y elimina la promesa de que una IA notifica a organismos de control.
- Parte del README actual de `main`: acordar qué se conserva de los commits `2980052` y `ffed98e`.
- `docs/demo-guion.md` con los momentos, el checklist previo y cómo grabar el video.
- Ensayo cronometrado: momentos 1 a 4 en 3 minutos o menos.

## 8. Entregables sin PR

- [ ] Voz probada en el laptop de la demo (viernes o sábado temprano)
- [ ] Verificar la regulación colombiana citada en el spec §4 antes del pitch
- [ ] Pitch: problema, solución, para quién, vencimiento del reclamo, riesgo de phishing y Jev en vivo como siguiente paso
- [ ] Video de respaldo grabado (queda fuera del repo) y subido donde pida el hackathon
- [ ] Confirmar que el repo sigue público
- [ ] Registrar el belotag en Equipos, si hay cuenta Belo
- [ ] Entrega enviada
- [ ] Opcional: publicaciones de "Construye en Público" (plazo del reto: martes 1 de diciembre)
