# Plata sin fronteras: diseño de la demo

Fecha: 2026-11-27 · Estado: borrador para aprobación · Planteamiento del reto: [docs/reto-belo-plata-sin-fronteras.md](../../reto-belo-plata-sin-fronteras.md)

## 1. Intención

Construimos una propuesta para el reto de Belo "Plata sin fronteras". La meta es el puntaje más alto posible en la rúbrica del reto (sección 07):

| Criterio | Peso |
|---|---|
| Impacto | 30% |
| Innovación | 20% |
| Viabilidad técnica | 20% |
| Viabilidad de implementación | 20% |
| Presentación | 10% |

La propuesta es **una sola historia de punta a punta**, no cuatro mini-demos separadas. Toca las cuatro ideas del reto, y la **protección contra fraude ("Seguridad sin fricción") es la pieza que se construye a fondo**. Las otras tres son livianas pero funcionan.

**Por qué una sola historia:** el reto pide una demo que "muestra el flujo completo desde el punto de vista de la persona", y la rúbrica pregunta "¿funciona de punta a punta?" (viabilidad técnica). Un flujo coherente le gana a cuatro piezas a medias en ambos puntos.

**Por qué la seguridad va a fondo:**
- Es el momento más dramático de la historia.
- Funciona de punta a punta con reglas deterministas.
- Belo podría adoptarla pronto, lo que suma en viabilidad de implementación.
- Le da a la IA un papel claro y defendible, lo que suma en innovación.

## 2. La historia

Todas las personas y los datos son ficticios.

- **Andrés**, 29 años, freelancer en Santiago (Chile). Usa apps con soltura.
- **Rosa**, 68 años, su mamá, en Bogotá (Colombia). Nunca tuvo cuenta bancaria y todavía no usa Belo. Usa WhatsApp.

Momentos de la demo, en orden:

1. **Enviar a alguien sin cuenta.** Andrés le envía US$200 al número de celular de Rosa.
   - Antes de enviar, ve: "Rosa aún no tiene Belo. Le llegará un link para recibir."
   - El pago no se rechaza. El grafo recorre débito → tasa fijada → vigilante → controles antilavado y se detiene en un nodo **"Esperando a Rosa"**: la plata queda reservada para ella.
2. **Primera vez.** En el teléfono de Rosa aparece un mensaje estilo WhatsApp con el link para reclamar (simulado). Lo abre y empieza el chat de Belo:
   - la app hace una pregunta para elegir su nivel, y a Rosa le toca el nivel sencillo;
   - completa un registro simplificado (datos inventados y un código de un solo uso simulado a su número);
   - aprende qué es un dólar digital con una tarjeta de "frascos";
   - recibe COP 800.000 y elige dónde guardarlos (pesos o dólares).

   Tasa ficticia: 4.000 COP por dólar.
3. **Velocidad.** Andrés envía US$20 más "para el taxi". Como Rosa ya tiene cuenta, el grafo recorre el camino completo y ella recibe en **740 ms** (simulado).
4. **Estafa frenada.** Un falso "asesor de Belo" llama a Rosa; el presentador narra esta parte. Le insiste en pasar toda su plata a una "cuenta segura". A las 22:40, Rosa intenta enviar todo su saldo a una llave nueva.
   - El vigilante pausa el pago y le hace una pregunta.
   - Ella responde por voz: "me llamó uno de Belo diciendo que mi cuenta estaba en peligro".
   - Los juicios simulados de Jev detectan suplantación, y el pago pasa a retención.
   - Rosa ve "Llamar a Andrés", y a Andrés le llega un aviso.
   - En el grafo, el paquete se frena en el nodo Vigilante, y el nodo se abre para mostrar cómo decidió.
5. **Estafa no frenada y reportada.** En un segundo escenario, Rosa paga COP 150.000 por una compra falsa en internet. El riesgo es bajo, así que pasa solo con un aviso suave. Después toca "Me estafaron", cuenta lo que pasó en una nota de voz y la app arma el caso. Ella lo revisa y lo envía.
6. **Asesor.** Rosa dice que quiere ahorrar COP 300.000 para diciembre. El asesor le hace una pregunta de perfil de riesgo, le muestra su perfil y le propone un plan de ahorro con un simulador. Ella confirma. El asesor nunca mueve plata por su cuenta.

No sabemos cuánto dura el turno en vivo. Los momentos 1 a 4 deben caber en unos 3 minutos. Los momentos 5 y 6 se muestran en vivo si sobra tiempo; si no, van en el pitch y en el video de respaldo.

## 3. Metas, fuera de alcance y lo que no puede pasar

**Metas**
- Una demo que corra la historia completa, de forma confiable y en vivo.
- Un motor de reglas real y probado para el vigilante, sin dependencias del framework, que Belo pueda llevarse tal cual.
- Una explicación visible de *por qué* el pago es rápido y *por qué* el vigilante decidió lo que decidió.
- Una interfaz que puedan usar tanto una persona de 68 años que la usa por primera vez como un joven de 29 que domina la tecnología.

**Fuera de alcance**
- Integración con rieles de pago, KYC o cambio de divisas reales.
- Llamar a Jev en vivo: el equipo no tiene acceso a TypeSafe para este evento.
- Detección de robo de cuenta más allá de una o dos reglas. La profundidad va a las estafas en que engañan a la persona para que pague ("estafas de pago autorizado").
- Evaluación real de crédito o ejecución de inversiones.
- App móvil nativa, hosting para muchos usuarios o estado por visitante.

**Lo que no puede pasar**, porque hace fallar la propuesta aunque técnicamente funcione:
- Una IA o un agente que mueva, congele o libere plata por su cuenta. El reto excluye los "agentes que operan solos con la plata de la persona".
- Datos personales reales en cualquier parte: código, fixtures, logs, capturas o video.
- Presentar respuestas de IA simuladas, o latencias simuladas, como si fueran en vivo o medidas.
- Una interfaz alarmista o con patrones engañosos hacia personas mayores.

## 4. Restricciones

**Reglas del reto:** prototipo funcional; solo datos simulados o servicios públicos; se entregan un pitch, una demo y un **repositorio público**.

**Entrega:** martes **2026-12-01**. **Dos ingenieros.**

**Formato de presentación:**
- en vivo, guiada por un presentador, en un solo laptop, con un **video grabado de respaldo**;
- la demo corre como un solo proceso de servidor local de larga duración;
- nada depende del Wi-Fi del lugar, salvo, opcionalmente, el reconocimiento de voz del navegador.

**Regulación colombiana.** Hay que respetarla, y la demo nunca propone evadir controles. Antes del pitch hay que verificar lo siguiente; este documento no afirma detalles:
- La asesoría de inversión está regulada por la Superintendencia Financiera (SFC). Por eso el asesor se presenta como **educación y simulación**, no como asesoría de inversión personalizada.
- Las transferencias entre países pasan por el régimen cambiario (intermediarios del mercado cambiario autorizados). No está verificado qué licencia tiene Belo en Colombia.
- Los controles antilavado (SARLAFT) van dentro del camino del pago. El registro simplificado de la demo representa un producto real con KYC simplificado; hay que revisar qué tipos de cuenta de bajo monto o simplificada aplican.
- La ley de habeas data (Ley 1581 de 2012) aplica a la lista compartida de llaves reportadas, a las notas de voz y a los datos del registro.

**Nada de mover plata sin la persona:** las reglas deterministas deciden; la IA solo interpreta lenguaje; la persona usuaria o un revisor humano toma cada decisión final.

**Dependencias.** Según las reglas del dueño del proyecto, cada dependencia nueva necesita justificar su necesidad, licencia y mantenimiento, y requiere aprobación. Se proponen las siguientes, y se aprueban junto con este documento:
- **Runtime:** `next`, `react` y `react-dom`, todas con licencia MIT.
  - Next.js da frontend y API en un mismo proyecto; Vercel lo mantiene activamente.
  - React y React DOM los mantiene Meta.
- **Desarrollo:**
  - `typescript`: licencia Apache-2.0; lo mantiene Microsoft.
  - `@types/node`, `@types/react` y `@types/react-dom`: licencia MIT; vienen de DefinitelyTyped.

Las versiones no se fijan aquí; quien construye elige las estables actuales. Todo lo demás que el scaffold trae por defecto queda **excluido** salvo aprobación aparte: ESLint, Tailwind, frameworks de pruebas, kits de UI, etc. Ni Three.js ni el SDK de TypeSafe.

## 5. Enfoques considerados

| Enfoque | Ventajas y desventajas | Decisión |
|---|---|---|
| **A. Una sola app en TypeScript** (Next.js con frontend y rutas de API, un proceso) | Un lenguaje y un proceso. Sería más lento si el equipo fuera más fuerte en Python. | **Elegido** por el equipo. |
| B. Frontend web + backend en Python (FastAPI) | Cómodo para un equipo de Python, pero son dos proyectos y hay más código para conectarlos. | Descartado. |
| C. Sitio estático, sin servidor | Lo más rápido de construir, pero sin estado en el servidor. Sin servidor no hay libro contable real, ni reclamo por link entre dos teléfonos, ni vigilante en el servidor evaluando cada pago. Eso debilita "funciona de punta a punta", y el camino a Jev en vivo necesitaría un servidor igual. | Descartado. |

**Jev (System One de TypeSafe) va simulado.** El equipo nunca ha usado Jev y no tiene acceso a TypeSafe para este evento. Decisión: juicios escritos a mano y rotulados con claridad, que siguen la **misma forma de pedido y respuesta** que la API pública de System One. Así la demo muestra exactamente cómo se usaría Jev. Pasar a la llamada real después significa reemplazar una sola función (sección 8).

## 6. Diseño decidido

### 6.1 Escenario de la demo

Una sola página, pensada para proyector:

```
┌───────────┬────────────────────────────────────────────┬───────────┐
│  Andrés   │  Grafo del pago (SVG)                      │   Rosa    │
│  teléfono │  Débito ─ Tasa ─ Vigilante ─ AML ─ Crédito │  teléfono │
│ (compacto)│  ...... carril de compensación diferida .. │ (sencillo)│
│           │  [Aquí se abre la vista del vigilante]     │           │
└───────────┴────────────────────────────────────────────┴───────────┘
             controles del presentador: siguiente momento · reiniciar
```

- Los dos teléfonos son interfaz real en la misma página, manejados por un solo proceso de backend con estado en memoria.
- Los controles del presentador avanzan los momentos del guion y reinician los datos ficticios.
- El video de respaldo es una grabación de pantalla de esta página corriendo los momentos.

### 6.2 Interfaz: chat con dos niveles

- **Toda la app es una conversación con Belo.** Cada operación es un mensaje y cada respuesta, un botón. Se puede hablar en vez de tocar.
- **Dos niveles en el mismo chat:**
  - *Sencillo* (Rosa): una pregunta por mensaje, letra grande, botón de voz y explicaciones con ejemplos del día a día.
  - *Compacto* (Andrés): mensajes más densos y atajos.
- **Elección de nivel:** la primera vez se pregunta "¿Has usado antes una app de banco?", y se puede cambiar cuando se quiera. Es una regla fija, sin IA.
- **Tarjeta de frascos:** aparece cuando la persona pregunta por su saldo o cuando la app explica qué es un dólar digital. Hay un frasco de pesos y uno de dólares; ahorrar es pasar plata de un frasco al otro.
- **Todo pasa dentro del chat:** el registro, la pausa del vigilante, la pregunta de perfil del asesor y el reporte de estafa son mensajes del chat.

Mensaje de pausa del vigilante (nivel sencillo):

```
┌──────────────────────────────┐
│  Un momento, Rosa            │
│  Es la primera vez que le    │
│  mandas plata a "J. P***"    │
│  y es todo tu saldo.         │
│  ¿Alguien te está pidiendo   │
│  este pago por teléfono      │
│  ahora mismo?                │
│    [ Sí ]        [ No ]      │
│    [ Responder con voz ]     │
│    Llamar a Andrés           │
└──────────────────────────────┘
```

### 6.3 Mecanismo de pago entre países (liviano)

El mecanismo que proponemos, para el pitch y la animación:

- **Plata ya ubicada en cada país.** Belo mantiene saldos propios en cada país. A quien envía se le debita, y a quien recibe se le acredita al instante desde el saldo del país de destino.
- **Compensación neta diferida.** El cruce de plata entre países se hace después, en dólares digitales y por lotes, fuera del camino que espera la persona. Ella solo espera lo que tarda un pago local.
- **Menos rechazos:**
  - antes de enviar, se valida la llave de destino y se muestra el nombre enmascarado del titular, igual que en Bre-B;
  - un pago a un número de celular sin cuenta Belo **no se rechaza**: se convierte en un **pago por reclamar** (sección 6.4).
- **Controles en el camino:** tasa de cambio fijada, controles antilavado y de listas restrictivas, y el vigilante.

**Tiempos meta simulados.** Usar exactamente estos números en todas partes, rotulados como "simulado":

| Paso | Meta |
|---|---|
| Débito (quien envía) | 300 ms |
| Tasa de cambio fijada | 40 ms |
| Vigilante | 120 ms (presupuesto < 200 ms) |
| Antilavado / listas restrictivas | 80 ms |
| Crédito (quien recibe) | 200 ms |
| **Lo que espera quien recibe (con cuenta)** | **740 ms** |
| Compensación diferida | por lotes, minutos (se muestra como un carril lento de fondo) |

En un pago por reclamar, el grafo se detiene en "Esperando a Rosa" después del control antilavado (540 ms) y corre el crédito cuando se completa el reclamo.

### 6.4 Pago por reclamar (link)

- **Lado de quien envía.** Si el número de destino no tiene cuenta Belo, quien envía lo ve antes de confirmar. Al confirmar, se le debita y el monto queda reservado como **pago por reclamar**.
- **Lado de quien recibe.** Le llega un mensaje con un link, simulado como un aviso estilo WhatsApp en el teléfono de Rosa. El link abre el chat de Belo y nunca pide contraseñas ni datos de tarjeta.
- **El reclamo:**
  1. registro simplificado, con datos inventados;
  2. código de un solo uso simulado al número de destino, para probar que es de quien recibe;
  3. el crédito.
- **Vencimiento.** Si nadie reclama el pago, se devuelve a quien lo envió después de un plazo fijo, por una regla del producto y no por un agente. No se muestra en la demo; se menciona en el pitch.
- **Riesgo de phishing.** El pitch reconoce que los links falsos de reclamo son un riesgo de phishing. La educación antiestafa de la propia demo lo refuerza: Belo nunca te pide por teléfono que muevas tu plata ni que compartas códigos.

### 6.5 Vigilante (a fondo)

**Forma.** Un módulo puro de TypeScript, sin imports del framework:
`evaluate(payment, context) → { level, score, reasons[] }`
Los pesos de las reglas y los umbrales de cada nivel viven en una sola tabla de configuración. El módulo corre en el camino rápido del pago; Jev nunca.

**Reglas para estafas de pago autorizado, el foco:**

| ID | Señal |
|---|---|
| R1 | Primer pago de esta persona a esta llave |
| R2 | Monto ≥ 80% del saldo disponible |
| R3 | Monto muy por encima de los pagos habituales de la persona (o la persona tiene poco historial) |
| R4 | Hora inusual para esta persona (para usuarios nuevos: 22:00–06:00) |
| R5 | Velocidad: varios pagos salientes en poco tiempo |
| R6 | Llave de destino reportada por otros usuarios. Dos niveles: *reportada, sin verificar* pesa moderado; *confirmada por cumplimiento* pesa alto |
| R7 | Llave de destino registrada hace muy poco |

Reglas de robo de cuenta, solo una o dos:

| ID | Señal |
|---|---|
| T1 | Dispositivo nuevo seguido poco después de una acción sensible |
| T2 | Cambio de PIN o credencial seguido poco después de un envío de plata |

**Niveles:**

| Nivel | Lo que vive la persona |
|---|---|
| `allow` | Sin fricción. |
| `notice` | Un aviso suave de una línea en el chat; la persona sigue con un toque. |
| `pause` | El mensaje con la pregunta, como el de arriba. La persona puede responder con los botones o con voz o texto. |
| `hold` | El pago queda retenido, no se ejecuta. La persona puede cancelarlo o llamar a su contacto de confianza o a Belo, y el contacto de confianza recibe un aviso. Una retención nunca mueve plata. Liberar una retención queda fuera del alcance de la demo. |

**Cómo se escala desde `pause`:**
- Si la persona toca "Sí", el pago pasa a `hold`.
- Si responde con sus palabras, la respuesta pasa al paso de Jev (sección 6.6). La política, en código: suplantación ≥ 0,8, **o** un tipo de estafa distinto de `none` con probabilidad ≥ 0,7, pasa a `hold`. Si no, la persona puede seguir con una confirmación.
- Si el texto no coincide con ningún escenario, se decide solo con las reglas y quedan los botones.

**Explicaciones.** Cada motivo corresponde a una frase fija, escrita por una persona, para cada nivel de la interfaz. En ningún lugar hay texto generado.

**Contacto de confianza.** Rosa elige a Andrés. Él recibe un aviso cuando un pago pasa de `pause` a `hold` y cuando hay un reporte de estafa. Nunca aprueba, bloquea ni mueve nada.

### 6.6 Juicios de Jev (simulados)

Jev, el modelo System One de TypeSafe, devuelve juicios tipados con probabilidades, no texto.

**Sobre la respuesta a la pausa.** Un pedido con tres preguntas sobre el mismo estado (las palabras de la persona y el contexto del pago):
- `noul` (probabilidad de sí o no): ¿la descripción indica que alguien se hace pasar por un banco, por Belo, por una entidad del gobierno o por una autoridad?
- `choice` (tipo de estafa), una de: `impersonation_official`, `family_emergency`, `investment_promise`, `fake_purchase`, `romance`, `none`.
- `score` (presión o urgencia descrita), en los niveles ninguna / baja / media / alta.

**Sobre el reporte de estafa** (sección 6.7):
- `choice`: tipo de estafa, con las mismas opciones.
- **Elegir en vez de generar.** El código extrae de la transcripción los montos, fechas y llaves candidatos, y Jev elige cuál es el monto y cuál la llave del caso.

**Sobre el asesor:**
- `score`: tolerancia a perder plata según la respuesta libre de la persona, en cinco niveles ordenados.

**Modo guion, el único modo para este evento.** Un archivo de fixtures guarda, para cada escenario del guion, el pedido que se enviaría y una respuesta escrita a mano. Ambos siguen la forma de la documentación pública de la API de TypeSafe (docs.typesafe.ai), que quien construye debe leer.

- **Coincidencia por palabras clave.** La entrada se asigna a un escenario por conjuntos de palabras clave, no por frases exactas. Por ejemplo, una mención de una llamada junto con "Belo", "banco" o "asesor" corresponde al escenario de suplantación. Así se toleran errores del reconocimiento de voz. Lo que no coincide con ningún escenario cae en la decisión solo con reglas.
- **Entrada de respaldo.** El presentador siempre puede escribir la frase del guion en vez de decirla.
- **Rótulo honesto en cada juicio:** *"Respuesta simulada de Jev (ilustrativa). En producción: una llamada a la API de System One."* Nunca dice "grabada" ni "en vivo".

**Vista del vigilante.** Cuando se abre el nodo Vigilante del grafo, muestra:
- qué reglas se activaron y su peso,
- el puntaje y el nivel,
- la pregunta que se le enviaría a Jev,
- las respuestas tipadas con sus probabilidades, con el rótulo de arriba,
- la línea de política que decidió.

### 6.7 Reporte de estafa (camino de la víctima)

1. La persona toca "Me estafaron" en el chat.
2. Cuenta lo que pasó en una nota de voz, o lo escribe.
3. El código arma el caso. La cronología y el monto salen del libro contable ficticio; el tipo de estafa y los datos elegidos salen del paso simulado de Jev.
4. La persona revisa y corrige la tarjeta del caso, y la envía.
5. Al enviarla pasa lo siguiente, todo simulado:
   - Se registra el caso en Belo.
   - Si el destino es una cuenta Belo, se pone en cola una **solicitud de retención preventiva para revisión humana de cumplimiento**. La interfaz dice "en revisión por el equipo de Belo". Nunca la dispara Jev, ni el reporte por sí solo.
   - La llave de destino pasa a *reportada, sin verificar* para la regla R6.
   - Se genera el texto de una denuncia prellenada para la Fiscalía. La persona la envía ella misma, copiándola o descargándola.
   - El contacto de confianza recibe un aviso.

### 6.8 Asesor (liviano)

- **Perfil de a poco.** En cada visita hay una pregunta corta de perfil de riesgo, que la persona responde con sus palabras. El paso simulado de Jev puntúa la tolerancia a perder plata, y el código promedia las respuestas en un perfil: conservador, moderado o arriesgado. La persona puede verlo y corregirlo.
- **Metas.** La persona dice qué meta tiene, y el asesor muestra un simulador que compara ahorrar en pesos con ahorrar en dólares digitales, con tasas **ficticias** y sin pronósticos. Suma una tarjeta educativa sobre cómo empezar un historial de crédito con un monto pequeño.
- **Confirmación.** Toda acción requiere que la persona la confirme. Un aviso visible dice que esto es educación y simulación, no asesoría de inversión personalizada.

### 6.9 Datos

- Estado ficticio en memoria, cargado al arrancar y reiniciable desde los controles del presentador. Incluye usuarios, saldos, libro contable, llaves, pagos por reclamar, lista de llaves reportadas y fixtures de Jev.
- Las llaves y números de celular son claramente falsos, como `@rosa.demo`. Ningún número, documento ni nombre real.

### 6.10 Voz

- Se usa el reconocimiento de voz del navegador. Solo funciona bien en navegadores basados en Chromium, y envía el audio al fabricante del navegador.
- Siempre hay opción de escribir.
- No está verificado qué tan bien reconoce el español de Colombia; hay que probarlo temprano en el laptop de la demo.
- El laptop de la demo debe usar Chrome. El video de respaldo cubre una falla.

## 7. Prioridades y reparto del trabajo

Las prioridades son decisión mía; el equipo puede reordenarlas.

| Prioridad | Alcance |
|---|---|
| P0 | Escenario de la demo; chat con los dos niveles; momentos 1 a 4 (pago por reclamar, primera vez con frascos, velocidad, estafa frenada); módulo del vigilante con pruebas; modo guion de Jev y vista del vigilante. |
| P1 | Momento 5 (reporte de estafa). Reescribir el README para que el repo público coincida con este diseño: el actual promete IA agéntica que notifica a las autoridades, y eso contradice "lo que no puede pasar". Grabar el video de respaldo. |
| P2 | Momento 6 (asesor). |

Reparto sugerido para dos ingenieros en cuatro días; quien construye puede ajustarlo.

- **Ingeniero 1, backend y lógica:** módulo del vigilante y sus pruebas, estado ficticio y libro contable, pagos por reclamar, fixtures de Jev y coincidencia por palabras clave, reporte de estafa.
- **Ingeniero 2, interfaz y escenario:** página del escenario y controles del presentador, chat con los dos niveles, tarjeta de frascos, animación del grafo del pago y vista del vigilante.
- **Juntos, el último día (lunes 2026-11-30):** ensayo, video de respaldo y README.

## 8. A futuro: Jev en vivo

Queda fuera de alcance para este evento. Cuando haya acceso a TypeSafe:

- Solo cambia la función que obtiene los juicios: llama al endpoint de System One desde el servidor en vez de leer los fixtures. La clave de API se queda en el servidor.
- Los rótulos pasan de "simulada" a la fuente real.
- El respaldo es el mismo: ante un error o una demora, se decide solo con las reglas.

El pitch lo menciona como el siguiente paso.

## 9. Cómo se atiende cada criterio de la rúbrica

| Criterio | Peso | Cómo |
|---|---|---|
| Impacto | 30% | Dolores reales en una sola historia: enviar a alguien sin cuenta; una remesa cara y lenta; una persona sin banco que usa la app por primera vez; la estafa que apunta a personas mayores; reportar después de perder plata. |
| Innovación | 20% | El vigilante pregunta en vez de bloquear. El papel de la IA está diseñado y se muestra con juicios tipados **simulados**, rotulados como tales, que se combinan con una política explícita: la IA tiene un papel claro pero nunca decide, y la integración en vivo es una sola función. Pagos por reclamar en vez de rechazos. Contacto de confianza. Señal compartida de llaves reportadas, con revisión en dos niveles. |
| Viabilidad técnica | 20% | La historia completa corre de punta a punta en un servidor real con estado. Motor de reglas probado. Respaldo elegante cuando falla la comprensión de lenguaje. Los juicios de IA son simulados para este evento. |
| Viabilidad de implementación | 20% | El vigilante es un módulo sin dependencias del framework. El mecanismo se basa en conceptos que Belo ya usa (saldos locales, dólares digitales). La regulación está explícita. La integración con Jev es una sola función. |
| Presentación | 10% | Una persona protagonista, una sola página de escenario, un "por qué es rápido" y un "por qué se pausó" visibles, y un video de respaldo. |

## 10. Lo que decide quien construye

- Los pesos y umbrales exactos de las reglas, ajustados para que cada escenario del guion caiga en el nivel que le corresponde.
- La organización de archivos, la estructura de componentes, el estilo y la paleta. Dirección visual: chat minimalista, cálido y tranquilo, sin colores de alarma.
- Cómo llega el estado del backend a los dos teléfonos de la página del escenario.
- El ejecutor de pruebas: preferir uno sin dependencias nuevas, como el ejecutor que trae Node.
- Definir el comando de verificación del proyecto y correr ese mismo comando en CI. El repo todavía no tiene uno.
- El texto exacto de todo lo que lee la persona usuaria en español.
