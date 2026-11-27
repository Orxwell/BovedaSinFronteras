# Plata sin fronteras: plan de implementación de la demo

> **Para agentes:** SUB-SKILL REQUERIDA: usar superpowers:subagent-driven-development (recomendada) o superpowers:executing-plans para implementar este plan tarea por tarea. Los pasos usan casillas (`- [ ]`) para el seguimiento.

**Meta:** una demo en vivo, en un solo laptop, que corre de punta a punta la historia de Andrés y Rosa. Incluye el pago por reclamar, la primera vez, la velocidad, la estafa frenada, el reporte y el asesor, con un vigilante de reglas probado y juicios de Jev simulados.

**Arquitectura:**
- Una sola app Next.js (App Router) en TypeScript. Corre como un proceso local con estado en memoria (`globalThis`).
- Toda la lógica vive en `src/domain/`: TypeScript puro, sin imports de Next ni de React, probado con el ejecutor de pruebas de Node.
- Una sola ruta de API (`/api/demo`) recibe acciones y devuelve el estado completo.
- La interfaz (`src/app/` y `src/ui/`) es "tonta": dibuja el estado y envía las acciones que traen los botones de cada mensaje.

**Stack:** Node ≥ 24 (esta máquina tiene v26.3.0, que corre `.ts` sin compilar), Next.js, React y TypeScript. Las pruebas usan `node --test`, sin dependencias nuevas.

**Spec:** [docs/superpowers/specs/2026-11-27-plata-sin-fronteras-demo-design.md](../specs/2026-11-27-plata-sin-fronteras-demo-design.md). Quien ejecuta lee el spec y el plan.

**Rama:** `feat/plata-sin-fronteras-demo`, creada desde `docs/demo-design`. Nunca se commitea a `main`.

## Restricciones globales

**Dependencias.** Solo estas, aprobadas con el spec:
- runtime: `next`, `react`, `react-dom`;
- desarrollo: `typescript`, `@types/node`, `@types/react`, `@types/react-dom`.

Nada de ESLint, Tailwind, frameworks de pruebas, kits de UI, Three.js ni el SDK de TypeSafe sin otra aprobación.

**Idioma.**
- El código, los identificadores, los comentarios y los commits van en inglés (Conventional Commits).
- Todo lo que lee la persona usuaria va en español.
- Los documentos del proyecto van en español.

**Datos.** Solo datos ficticios. Llaves con formato `@nombre.algo` o `tel:nombre.demo`. Ningún número de celular, documento ni nombre real.

**Tiempos simulados.** Se usan exactamente estos valores (los pasos van en este orden; ver la desviación 1 más abajo):

| Paso | Tiempo |
|---|---|
| `fx` (tasa fijada) | 40 ms |
| `watchdog` (vigilante) | 120 ms |
| `aml` (antilavado) | 80 ms |
| `debit` (débito) | 300 ms |
| `credit` (crédito) | 200 ms |
| **Total** | **740 ms** |

Un pago por reclamar se detiene antes del crédito, a los **540 ms**.

**Tasa ficticia:** `FX_USD_COP = 4000`.

**Umbrales de política para la respuesta a la pausa:**
- suplantación `>= 0.8` → `hold`;
- tipo de estafa distinto de `none` con probabilidad `>= 0.7` → `hold`.

**Rótulo de Jev**, exacto en cada juicio mostrado:
`Respuesta simulada de Jev (ilustrativa). En producción: una llamada a la API de System One.`

**Reglas que no se rompen:**
- No hay texto generado: toda frase visible es fija y escrita por una persona.
- Una retención (`hold`) nunca mueve plata.
- La retención preventiva tras un reporte solo se pone en cola para revisión humana.

**Verificación:** `npm run verify`, que corre `typecheck`, `test` y `build`. Es el mismo comando en CI.

**TypeScript que Node puede correr sin compilar:**
- los imports relativos llevan la extensión `.ts`;
- los tipos se importan con `import type`;
- no hay `enum`, `namespace` ni propiedades de parámetro (`erasableSyntaxOnly`).

**Límite de capas:** `src/domain/**` no importa `next`, `react` ni nada de `src/app` o `src/ui`.

### Desviaciones del spec decididas en este plan

1. **Orden de los pasos del grafo.** El spec dibuja `Débito → Tasa → Vigilante → AML → Crédito`. Aquí el débito pasa después del vigilante y del control antilavado: `fx → watchdog → aml → debit → credit`. Con el orden del spec, una pausa o una retención ocurriría con la plata ya debitada, y eso contradice "una retención nunca mueve plata". Los totales del spec no cambian: 740 ms, y 540 ms para el pago por reclamar.
2. **Sin libro contable aparte.** La lista de pagos (`payments`) es el registro. La cronología del reporte sale de ahí.
3. **Pagos con monto fijo.** No hay pantalla para escribir montos o llaves. Los pagos salen de botones del chat o de los botones del presentador, con montos fijos del guion.

## Foco de revisión

Estas son las cinco fallas más probables que el spec implica pero no detalla. Cada una tiene su prueba en la tarea indicada.

1. **Doble clic o acción repetida.** Lo esperado es que se aplique una sola vez: no hay pagos ni créditos duplicados. Prueba en las tareas 2 y 5.
2. **Montos inválidos:** 0, negativos, con decimales en COP o mayores al saldo. Lo esperado es que se rechacen con un mensaje, sin crear pago y sin tocar saldos. Prueba en la tarea 5.
3. **Código de reclamo equivocado, o reclamar dos veces.** Lo esperado es que no se acredite, o que se acredite una sola vez. Prueba en la tarea 6.
4. **Montos escritos o dichos en español** en el reporte: `$150.000`, `150 mil`, `150000`, y en palabras ("ciento cincuenta mil"). Lo esperado es que se reconozcan los tres primeros; con el último, el caso se arma con el último pago. Prueba en la tarea 10.
5. **Texto libre que no coincide con nada, o voz no disponible.** Lo esperado es decidir solo con las reglas, mostrar de nuevo los botones y dejar siempre visible la opción de escribir. Pruebas en las tareas 4 y 5; revisión manual en la tarea 8.

## Mapa de archivos

```
.gitignore                        .superpowers/, node_modules/, .next/, *.tsbuildinfo
package.json                      scripts dev/build/start/test/typecheck/verify; engines node >=24
tsconfig.json
next-env.d.ts                     generado por el primer build y commiteado
.github/workflows/verify.yml      CI: npm ci && npm run verify
src/domain/money.ts               Money, conversión y formato
src/domain/types.ts               todos los tipos compartidos (estado, acciones, chat, System One)
src/domain/seed.ts                seedState(): estado inicial ficticio
src/domain/chat.ts                nextId(), pushMessage()
src/domain/actions.ts             dispatch(state, action): punto de entrada único; momentos del guion
src/domain/watchdog.ts            evaluate(), WATCHDOG_CONFIG
src/domain/copy.ts                REASON_COPY, JEV_LABEL y otras frases fijas
src/domain/jev.ts                 getJudgments() (punto de cambio a Jev en vivo), constructores de pedidos, normalize()
src/domain/jev-fixtures.ts        respuestas escritas a mano con forma de System One
src/domain/policy.ts              decideAfterPause(), describeDecision()
src/domain/payments.ts            PAYMENT_STEPS, envío, confirmar/cancelar, respuestas a la pausa, contacto
src/domain/claims.ts              pago por reclamar: abrir, verificar código, dónde guardar
src/domain/report.ts              extracción de candidatos, armado y envío del caso, texto de denuncia
src/domain/advisor.ts             perfil de riesgo, plan de ahorro
src/domain/*.test.ts              pruebas junto a cada módulo
src/app/layout.tsx, globals.css
src/app/page.tsx                  escenario (client component)
src/app/api/demo/route.ts         GET estado / POST acción
src/ui/useDemoState.ts            fetch del estado y envío de acciones
src/ui/Phone.tsx, Chat.tsx, Cards.tsx, VoiceInput.tsx
src/ui/PaymentGraph.tsx, WatchdogView.tsx, PresenterControls.tsx
docs/demo-guion.md                guion y checklist del ensayo
README.md                         reescrito en español
```

## Reparto y orden

- **Tarea 1:** la hace I1 primero. I2 espera, o prepara estilos.
- **Ingeniero 1, dominio y API:** tareas 2 → 3 → 4 → 5 → 6 → 10 → 11.
- **Ingeniero 2, interfaz:** tareas 7 → 8 → 9. Empieza apenas está la tarea 2, porque los tipos y la API ya existen.
- **La tarea 11 (asesor)** la toma quien se libere primero.
- **La tarea 12 la hacen los dos** el lunes 2026-11-30.
- **Prioridades del spec:**
  - P0: tareas 1–9;
  - P1: tareas 10 y 12;
  - P2: tarea 11.
- Las tareas 7–9 tocan solo `src/app/page.tsx`, `src/app/globals.css` y `src/ui/`. Las demás tocan `src/domain/` y la ruta de API. Así se evitan conflictos.

---

### Tarea 1: Scaffold y verificación (I1)

**Archivos:**
- Crear: `.gitignore`, `package.json`, `tsconfig.json`, `.github/workflows/verify.yml`, `src/domain/money.ts`, `src/domain/money.test.ts`, `src/app/layout.tsx`, `src/app/page.tsx` (provisoria) y `next-env.d.ts` (lo genera el build).

**Interfaces:**
- Produce:
  - `type Currency = 'USD' | 'COP'`
  - `type Money = { currency: Currency; minor: number }`, donde `minor` son centavos en USD y pesos en COP
  - `FX_USD_COP = 4000`
  - `usdToCop(m: Money): Money`
  - `copToUsd(m: Money): Money`: redondea hacia abajo al centavo
  - `formatMoney(m: Money): string`
  - `isValidMinor(currency: Currency, minor: number): boolean`: entero y > 0

- [ ] **Paso 1: Crear la rama y el `.gitignore`**

```bash
git switch -c feat/plata-sin-fronteras-demo docs/demo-design
```

`.gitignore` con estas líneas: `.superpowers/`, `node_modules/`, `.next/`, `*.tsbuildinfo`.

- [ ] **Paso 2: Crear `package.json` e instalar solo lo aprobado**

`package.json` lleva `"private": true`, `"type": "module"`, `"engines": { "node": ">=24" }` y estos scripts:

| Script | Comando |
|---|---|
| `dev` | `next dev` |
| `build` | `next build` |
| `start` | `next start` |
| `test` | `node --test "src/**/*.test.ts"` |
| `typecheck` | `tsc --noEmit` |
| `verify` | `npm run typecheck && npm test && npm run build` |

```bash
npm install next react react-dom
npm install --save-dev typescript @types/node @types/react @types/react-dom
```

- [ ] **Paso 3: Crear `tsconfig.json`**

Opciones de `compilerOptions`:

| Opción | Valor |
|---|---|
| `target` | `ES2022` |
| `lib` | `["dom", "dom.iterable", "esnext"]` |
| `module` | `esnext` |
| `moduleResolution` | `bundler` |
| `jsx` | `preserve` |
| `incremental` | `true` |
| `plugins` | `[{ "name": "next" }]` |

Opciones booleanas en `true`: `strict`, `noEmit`, `allowImportingTsExtensions`, `verbatimModuleSyntax`, `erasableSyntaxOnly`, `isolatedModules`, `skipLibCheck` y `resolveJsonModule`.

`include`: `["next-env.d.ts", "src/**/*.ts", "src/**/*.tsx", ".next/types/**/*.ts"]`.

Si `next build` ajusta el archivo, se commitea lo que deje.

- [ ] **Paso 4: Escribir la prueba que falla, `src/domain/money.test.ts`**

```ts
import { test } from 'node:test';
import assert from 'node:assert/strict';
import { formatMoney, usdToCop, copToUsd, isValidMinor } from './money.ts';

test('formats COP and USD in es-CO style', () => {
  assert.equal(formatMoney({ currency: 'COP', minor: 800000 }), '$800.000');
  assert.equal(formatMoney({ currency: 'USD', minor: 20000 }), 'US$200');
  assert.equal(formatMoney({ currency: 'USD', minor: 2050 }), 'US$20,50');
});
test('converts at the synthetic rate', () => {
  assert.deepEqual(usdToCop({ currency: 'USD', minor: 20000 }), { currency: 'COP', minor: 800000 });
  assert.deepEqual(copToUsd({ currency: 'COP', minor: 150000 }), { currency: 'USD', minor: 3750 });
  assert.deepEqual(copToUsd({ currency: 'COP', minor: 1 }), { currency: 'USD', minor: 0 });
});
test('only positive integers are valid amounts', () => {
  assert.equal(isValidMinor('COP', 150000), true);
  for (const bad of [0, -1, 1.5, Number.NaN]) assert.equal(isValidMinor('COP', bad), false);
});
```

- [ ] **Paso 5: Correr `npm test`.** Esperado: FALLA porque no existe `./money.ts`.

- [ ] **Paso 6: Implementar `src/domain/money.ts`** con las firmas de arriba. El formato usa `Intl.NumberFormat('es-CO')`, con 2 decimales en USD solo si los centavos no son 0.

- [ ] **Paso 7: Página provisoria que prueba el import con `.ts`**

- `src/app/layout.tsx` es el layout mínimo, con `<html lang="es">`.
- `src/app/page.tsx` muestra `formatMoney({ currency: 'COP', minor: 800000 })` importado desde `'../domain/money.ts'`.

- [ ] **Paso 8: Correr `npm run verify`.** Esperado: typecheck sin errores, 3 pruebas en PASS y build OK. Si el build rechaza los imports con `.ts`, hay que resolverlo aquí antes de seguir, porque todas las tareas dependen de eso.

- [ ] **Paso 9: CI en `.github/workflows/verify.yml`**

- Se dispara en `push` y en `pull_request`.
- Corre en `ubuntu-latest` con `actions/checkout@v4` y `actions/setup-node@v4` (`node-version: 26`, `cache: npm`).
- Comandos: `npm ci` y luego `npm run verify`.

- [ ] **Paso 10: Commit**

```bash
git add .gitignore package.json package-lock.json tsconfig.json next-env.d.ts .github/workflows/verify.yml src/
git commit -m "chore: scaffold Next.js app with verify pipeline"
```

---

### Tarea 2: Tipos, estado inicial, despacho y API (I1)

**Archivos:**
- Crear: `src/domain/types.ts`, `src/domain/seed.ts`, `src/domain/chat.ts`, `src/domain/actions.ts`, `src/app/api/demo/route.ts`
- Prueba: `src/domain/actions.test.ts`

**Interfaces:**
- Consume: `Money` y `Currency` de la tarea 1.
- Produce: los tipos de abajo, completos. Las tareas 3–11 los usan sin cambiarlos. La interfaz (tareas 7–9) programa contra ellos desde ya.

```ts
// src/domain/types.ts
import type { Money, Currency } from './money.ts';

export type UserId = string;
export type UiLevel = 'simple' | 'compact';
export type WatchdogLevel = 'allow' | 'notice' | 'pause' | 'hold';
export type ReasonId = 'R1' | 'R2' | 'R3' | 'R4' | 'R5' | 'R6' | 'R7' | 'T1' | 'T2';
export type ScamType = 'impersonation_official' | 'family_emergency' | 'investment_promise' | 'fake_purchase' | 'romance' | 'none';
export type PaymentStepId = 'fx' | 'watchdog' | 'aml' | 'debit' | 'credit';
export type BeatId = 'b1-first-send' | 'b3-second-send' | 'b4-scam-attempt' | 'b5-fake-purchase';
export type RiskProfile = 'conservative' | 'moderate' | 'bold';
export type KeyReportStatus = 'none' | 'reported' | 'confirmed';

export type User = {
  id: UserId; name: string; country: 'CL' | 'CO'; timeZone: string; homeCurrency: Currency;
  registered: boolean; level: UiLevel | null; trustedContactId: UserId | null;
  balances: Record<Currency, number>; savingsCop: number;
  device: { isNew: boolean; firstSeenAt: number }; credentialResetAt: number | null;
  riskScores: number[]; savingsGoal: { goalCop: number; months: number } | null;
};
export type Key = { key: string; ownerId: UserId | null; holderMasked: string; registeredAt: number | null; reportStatus: KeyReportStatus };
export type PaymentIntent = { fromUserId: UserId; toKey: string; amount: Money };
export type WatchdogReason = { id: ReasonId; weight: number };
export type WatchdogResult = { level: WatchdogLevel; score: number; reasons: WatchdogReason[] };
export type TraceStep = { step: PaymentStepId; ms: number; status: 'done' | 'waiting' | 'stopped' };

// System One shape (docs.typesafe.ai/api.md), mirrored for fixtures
export type NoulQuestion = { type: 'noul'; instructions: string; criteria?: { true: string; false: string } };
export type ChoiceQuestion = { type: 'choice'; instructions: string; criteria: Record<string, string | null> };
export type ScoreQuestion = { type: 'score'; instructions: string; criteria: string[] };
export type SystemOneRequest = { model: string; state: unknown; questions: Record<string, NoulQuestion | ChoiceQuestion | ScoreQuestion> };
export type NoulAnswer = { type: 'noul'; noul: number };
export type ChoiceAnswer = { type: 'choice'; choice: string; confidence: number; probabilities: Record<string, number> };
export type ScoreAnswer = { type: 'score'; score: number; confidence: number; legend: Record<string, string>; probabilities: Record<string, number> };
export type SystemOneResponse = { model: string; answers: Record<string, NoulAnswer | ChoiceAnswer | ScoreAnswer>; usage: { input_tokens: number; output_tokens: number } };

export type PauseJudgments = { impersonation: number; scamType: ScamType; scamTypeP: number; pressure: number };
export type PauseDecision = 'hold' | 'confirm' | 'buttons';
export type PauseRecord = { text: string; request: SystemOneRequest; response: SystemOneResponse | null; judgments: PauseJudgments | null; decision: PauseDecision };

export type Payment = {
  id: string; fromUserId: UserId; toKey: string; amount: Money; credited: Money | null;
  status: 'completed' | 'pending-claim' | 'awaiting-confirm' | 'paused' | 'held' | 'cancelled';
  watchdog: WatchdogResult; trace: TraceStep[]; createdAt: number; pause: PauseRecord | null;
};
export type PendingClaim = { id: string; paymentId: string; toKey: string; amount: Money; otp: string; claimed: boolean };
export type ScamCase = {
  id: string; userId: UserId; paymentId: string | null; amount: Money | null; toKey: string | null;
  scamType: ScamType; scamTypeP: number; transcript: string; status: 'draft' | 'submitted'; complaintText: string | null;
};
export type ComplianceHoldRequest = { caseId: string; key: string; status: 'pending-review' };

export type Card =
  | { kind: 'jars'; userId: UserId }
  | { kind: 'whatsapp-link'; claimId: string }
  | { kind: 'sms-code'; code: string }
  | { kind: 'case'; caseId: string }
  | { kind: 'complaint'; caseId: string }
  | { kind: 'savings-plan'; userId: UserId };
export type TextTarget =
  | { type: 'pause-text'; paymentId: string }
  | { type: 'verify-claim'; claimId: string }
  | { type: 'report-text' }
  | { type: 'advisor-answer' };
export type ChatButton = { label: string; action: ActionBody };
export type ChatMessage = {
  id: string; from: 'belo' | 'user' | 'external'; text: string; card: Card | null;
  buttons: ChatButton[]; expectsText: TextTarget | null; answered: boolean; jevLabel: boolean;
};

export type ActionBody =
  | { type: 'reset' }
  | { type: 'beat'; beat: BeatId }
  | { type: 'set-level'; userId: UserId; level: UiLevel }
  | { type: 'send'; fromUserId: UserId; toKey: string; amount: Money }
  | { type: 'confirm-payment'; paymentId: string }
  | { type: 'cancel-payment'; paymentId: string }
  | { type: 'pause-answer'; paymentId: string; answer: 'yes' | 'no' }
  | { type: 'pause-text'; paymentId: string; text: string }
  | { type: 'call-contact'; userId: UserId }
  | { type: 'open-claim'; claimId: string }
  | { type: 'verify-claim'; claimId: string; otp: string }
  | { type: 'keep-in'; userId: UserId; currency: Currency }
  | { type: 'show-jars'; userId: UserId }
  | { type: 'report-start'; userId: UserId }
  | { type: 'report-text'; userId: UserId; text: string }
  | { type: 'report-set-type'; caseId: string; scamType: ScamType }
  | { type: 'report-submit'; caseId: string }
  | { type: 'advisor-start'; userId: UserId }
  | { type: 'advisor-goal'; userId: UserId; goalCop: number; months: number }
  | { type: 'advisor-answer'; userId: UserId; text: string }
  | { type: 'advisor-confirm'; userId: UserId };
export type Action = ActionBody & { id: string; messageId?: string };

export type DemoState = {
  seq: number; now: number;
  users: Record<UserId, User>; keys: Record<string, Key>;
  payments: Payment[]; claims: PendingClaim[]; cases: ScamCase[]; complianceQueue: ComplianceHoldRequest[];
  chats: Record<UserId, ChatMessage[]>; appliedActionIds: string[];
};
// ACTION_TYPES: readonly array literal with every ActionBody['type'] above, used by the route to validate requests
export const ACTION_TYPES = ['reset', 'beat', /* …every type… */ 'advisor-confirm'] as const satisfies readonly ActionBody['type'][];
```

- `chat.ts`:
  - `nextId(state: DemoState, prefix: string): string` devuelve `${prefix}-${++state.seq}`. Es determinista.
  - `pushMessage(state: DemoState, userId: UserId, msg: Partial<ChatMessage> & { text: string }): ChatMessage` completa los valores por defecto: `from: 'belo'`, `card: null`, `buttons: []`, `expectsText: null`, `answered: false`, `jevLabel: false`.
- `seed.ts`: `seedState(): DemoState`, con estos datos exactos:
  - **Reloj:** `now = Date.parse('2026-11-28T14:50:00Z')`, que son las 09:50 en Bogotá.
  - **Usuarios:**
    - `andres`: `CL`, `America/Santiago`, `homeCurrency 'USD'`, registrado, `level 'compact'`, `balances { USD: 124000, COP: 0 }`, dispositivo no nuevo.
    - `rosa`: `CO`, `America/Bogota`, `homeCurrency 'COP'`, **no registrada**, `level null`, `trustedContactId 'andres'`, saldos en 0.
    - `mule1`: nombre `'J. P.'`, registrado, `CO`.
    - `shop`: nombre `'Ofertas T.'`, registrado, `CO`.
  - **Llaves:**

    | Llave | Dueño | Nombre enmascarado | Registrada hace |
    |---|---|---|---|
    | `tel:rosa.demo` | `null` | `'R. ***'` | sin registrar (`null`) |
    | `@seguridad.belo` | `mule1` | `'J. P***'` | 2 días |
    | `@ofertas.tienda` | `shop` | `'O. T***'` | 40 días |
    | `@andres.demo` | `andres` | `'A. ***'` | 400 días |

  - **Historial de Andrés:** 4 pagos `completed` a terceros de US$150, 650, 320 y 410 (`minor` 15000, 65000, 32000, 41000).
  - **Chat de Andrés:** un mensaje: `"Mamá (celular ficticio) aún no tiene Belo. Le llegará un link para recibir."` con el botón `"Enviar US$200"`, que dispara `{ type: 'send', fromUserId: 'andres', toKey: 'tel:rosa.demo', amount: { currency: 'USD', minor: 20000 } }`.
- `actions.ts`: `dispatch(state: DemoState, action: Action): DemoState`.
  1. Si `action.id` ya está en `appliedActionIds`, devuelve `state` sin cambios.
  2. Si trae `messageId`, marca ese mensaje como `answered = true` en cualquier chat.
  3. Registra el id.
  4. Despacha `reset`, que devuelve `seedState()`, y `set-level`.
  5. Los tipos de `ActionBody` que todavía no tienen implementación lanzan `Error('not implemented: <type>')`.
  6. Un tipo que no está en `ActionBody` lanza `Error('unknown action: <type>')`.
- `route.ts`:
  - `export const dynamic = 'force-dynamic'`.
  - Estado en `globalThis.__demoState ??= seedState()`.
  - `GET` devuelve el estado.
  - `POST` valida que el body sea un objeto con `id: string` y con `type` dentro de `ACTION_TYPES`. Si no, responde `400`. Si pasa, despacha y devuelve el estado.
  - Si `dispatch` lanza, responde `400 { error }`.

- [ ] **Paso 1: Prueba que falla, `src/domain/actions.test.ts`**

```ts
test('seed is synthetic and Rosa starts unregistered', () => {
  const s = seedState();
  assert.equal(s.users.rosa.registered, false);
  assert.equal(s.keys['tel:rosa.demo'].ownerId, null);
  for (const k of Object.keys(s.keys)) assert.match(k, /^(@|tel:)[a-z0-9.]+$/);
});
test('reset returns the seed', () => {
  const s = dispatch(seedState(), { id: 'a1', type: 'set-level', userId: 'andres', level: 'simple' });
  assert.deepEqual(dispatch(s, { id: 'a2', type: 'reset' }), seedState());
});
test('the same action id is applied once', () => {
  let s = dispatch(seedState(), { id: 'x', type: 'set-level', userId: 'andres', level: 'simple' });
  s = dispatch(s, { id: 'x', type: 'set-level', userId: 'andres', level: 'compact' });
  assert.equal(s.users.andres.level, 'simple');
});
test('messageId marks the message answered', () => {
  const s = seedState(); const msgId = s.chats.andres[0].id;
  dispatch(s, { id: 'y', type: 'set-level', userId: 'andres', level: 'simple', messageId: msgId });
  assert.equal(s.chats.andres[0].answered, true);
});
test('unknown action types throw', () => {
  assert.throws(() => dispatch(seedState(), { id: 'z', type: 'nope' } as unknown as Action), /unknown action/);
});
```

- [ ] **Paso 2: Correr `npm test`.** Esperado: FALLA porque faltan los módulos.
- [ ] **Paso 3: Implementar `types.ts`, `chat.ts`, `seed.ts` y `actions.ts`** con las firmas y los valores de arriba.
- [ ] **Paso 4: Correr `npm test`.** Esperado: PASS.
- [ ] **Paso 5: Implementar `route.ts`** y probarlo a mano. Con `npm run dev` corriendo:
  - `curl -s localhost:3000/api/demo` devuelve JSON con `users.rosa`.
  - `curl -s -XPOST localhost:3000/api/demo -d '{"id":"q","type":"nope"}'` devuelve 400.
- [ ] **Paso 6: Correr `npm run verify`.** Esperado: todo en PASS.
- [ ] **Paso 7: Commit:** `feat(domain): add demo state, dispatch and API route`

---

### Tarea 3: Vigilante (I1)

**Archivos:**
- Crear: `src/domain/watchdog.ts`, `src/domain/copy.ts`
- Prueba: `src/domain/watchdog.test.ts`

**Interfaces:**
- Consume: `PaymentIntent`, `WatchdogResult`, `ReasonId` y `KeyReportStatus` de la tarea 2.
- Produce:

```ts
export type WatchdogContext = {
  now: number; hourLocal: number; balanceMinor: number; isFirstPaymentToKey: boolean;
  historyMinor: number[]; recentOutgoingAt: number[];
  keyRegisteredAt: number | null; keyReportStatus: KeyReportStatus;
  device: { isNew: boolean; firstSeenAt: number }; credentialResetAt: number | null;
};
export const WATCHDOG_CONFIG: {
  weights: { R1: 20; R2: 20; R3: 10; R4: 10; R5: 15; R6_reported: 25; R6_confirmed: 60; R7: 10; T1: 50; T2: 50 };
  thresholds: { notice: 25; pause: 50; hold: 80 };
  r2BalanceRatio: 0.8; r3MinHistory: 3; r3MedianMultiplier: 3; r4Start: 22; r4End: 6;
  r5Count: 3; r5WindowMs: 600_000; r7MaxAgeMs: 604_800_000; t1WindowMs: 86_400_000; t2WindowMs: 3_600_000;
};
export function evaluate(payment: PaymentIntent, ctx: WatchdogContext): WatchdogResult;
// copy.ts
export const REASON_COPY: Record<ReasonId, { simple: string; compact: string }>;
export const JEV_LABEL = 'Respuesta simulada de Jev (ilustrativa). En producción: una llamada a la API de System One.';
```

**Reglas** (los valores salen de `WATCHDOG_CONFIG`). `amount` es `payment.amount.minor`:

| Regla | Se activa cuando |
|---|---|
| R1 | `isFirstPaymentToKey` |
| R2 | `amount >= 0.8 × balanceMinor` |
| R3 | hay menos de 3 pagos en el historial, **o** `amount > 3 × median(historyMinor)`; la mediana de una lista par promedia los dos del medio |
| R4 | `hourLocal >= 22 \|\| hourLocal < 6` |
| R5 | hay 3 o más `recentOutgoingAt` en los últimos 10 min |
| R6 | `keyReportStatus` es `reported` (25) o `confirmed` (60) |
| R7 | `keyRegisteredAt !== null` y la llave tiene menos de 7 días |
| T1 | `device.isNew` y menos de 24 h desde `firstSeenAt` |
| T2 | `credentialResetAt` hace menos de 1 h |

**Nivel según el puntaje:** `>= 80` es `hold`; `>= 50` es `pause`; `>= 25` es `notice`; lo demás es `allow`. Las razones salen en el orden R1 … T2.

Para la regla R4, conviene marcar en el código: `// ponytail: one default window for every user; per-user usual hours once history exists`.

- [ ] **Paso 1: Prueba que falla.** Contexto base y casos:

```ts
const NOW = Date.parse('2026-11-28T15:00:00Z'); const DAY = 86_400_000;
const base: WatchdogContext = { now: NOW, hourLocal: 11, balanceMinor: 1_000_000, isFirstPaymentToKey: false,
  historyMinor: [500_000, 600_000, 700_000], recentOutgoingAt: [], keyRegisteredAt: NOW - 400 * DAY,
  keyReportStatus: 'none', device: { isNew: false, firstSeenAt: NOW - 400 * DAY }, credentialResetAt: null };
const pay = (minor: number, currency: 'COP' | 'USD' = 'COP') => ({ fromUserId: 'u', toKey: '@x.demo', amount: { currency, minor } });
const ids = (r: WatchdogResult) => r.reasons.map((x) => x.id);

test('quiet payment is allow', () => assert.deepEqual(evaluate(pay(100_000), base), { level: 'allow', score: 0, reasons: [] }));
test('each rule fires alone with its weight', () => {
  const cases: [ReasonId, Partial<WatchdogContext>, number, number][] = [
    ['R1', { isFirstPaymentToKey: true }, 100_000, 20],
    ['R2', {}, 800_000, 20],
    ['R3', { historyMinor: [] }, 100_000, 10],
    ['R4', { hourLocal: 23 }, 100_000, 10],
    ['R5', { recentOutgoingAt: [NOW - 60_000, NOW - 120_000, NOW - 180_000] }, 100_000, 15],
    ['R6', { keyReportStatus: 'reported' }, 100_000, 25],
    ['R7', { keyRegisteredAt: NOW - 2 * DAY }, 100_000, 10],
    ['T1', { device: { isNew: true, firstSeenAt: NOW - 3_600_000 } }, 100_000, 50],
    ['T2', { credentialResetAt: NOW - 600_000 }, 100_000, 50],
  ];
  for (const [id, over, amount, weight] of cases) {
    const r = evaluate(pay(amount), { ...base, ...over });
    assert.deepEqual(ids(r), [id], id); assert.equal(r.score, weight, id);
  }
});
test('R4 window edges', () => {
  for (const [h, fires] of [[21, false], [22, true], [5, true], [6, false]] as const)
    assert.equal(ids(evaluate(pay(100_000), { ...base, hourLocal: h })).includes('R4'), fires, String(h));
});
test('R3 is strictly above 3x median', () => {
  const c = { ...base, historyMinor: [100_000, 100_000, 100_000], balanceMinor: 10_000_000 };
  assert.equal(ids(evaluate(pay(300_000), c)).includes('R3'), false);
  assert.equal(ids(evaluate(pay(300_001), c)).includes('R3'), true);
});
test('beat 1: first send to mother is allow', () => {
  const r = evaluate(pay(20_000, 'USD'), { ...base, balanceMinor: 124_000, isFirstPaymentToKey: true,
    historyMinor: [15_000, 65_000, 32_000, 41_000], hourLocal: 10, keyRegisteredAt: null });
  assert.equal(r.level, 'allow'); assert.equal(r.score, 20);
});
test('beat 4: scam attempt lands in pause with score 70', () => {
  const r = evaluate(pay(880_000), { ...base, balanceMinor: 880_000, isFirstPaymentToKey: true, historyMinor: [],
    hourLocal: 22, keyRegisteredAt: NOW - 2 * DAY });
  assert.equal(r.level, 'pause'); assert.equal(r.score, 70); assert.deepEqual(ids(r), ['R1', 'R2', 'R3', 'R4', 'R7']);
});
test('beat 5: fake purchase lands in notice', () => {
  const r = evaluate(pay(150_000), { ...base, balanceMinor: 880_000, isFirstPaymentToKey: true, historyMinor: [],
    keyRegisteredAt: NOW - 40 * DAY });
  assert.equal(r.level, 'notice'); assert.equal(r.score, 30);
});
test('confirmed key on a first payment is hold', () => {
  assert.equal(evaluate(pay(100_000), { ...base, isFirstPaymentToKey: true, keyReportStatus: 'confirmed' }).level, 'hold');
});
test('every reason has simple and compact copy', () => {
  for (const id of ['R1', 'R2', 'R3', 'R4', 'R5', 'R6', 'R7', 'T1', 'T2'] as const) {
    assert.ok(REASON_COPY[id].simple.length > 0 && REASON_COPY[id].compact.length > 0, id);
  }
});
```

- [ ] **Paso 2: Correr `npm test`.** Esperado: FALLA.
- [ ] **Paso 3: Implementar `evaluate` y `WATCHDOG_CONFIG`** en `watchdog.ts`. El módulo no importa nada fuera de `types.ts`.
- [ ] **Paso 4: Escribir `copy.ts`** con `REASON_COPY` y `JEV_LABEL`. Textos `simple` iniciales (se pueden pulir):

| Regla | Texto |
|---|---|
| R1 | "Es la primera vez que le mandas plata a esta persona." |
| R2 | "Es casi todo lo que tienes." |
| R3 | "Es más de lo que sueles mandar." |
| R4 | "Es tarde en la noche." |
| R5 | "Hiciste varios pagos seguidos." |
| R6 | "Otras personas reportaron esta cuenta." |
| R7 | "Esta cuenta es muy nueva." |
| T1 | "Estás usando un celular nuevo." |
| T2 | "Cambiaste tu clave hace muy poco." |

  Los textos `compact` van en una línea técnica corta, por ejemplo R2: "≥80% del saldo".
- [ ] **Paso 5: Correr `npm test`.** Esperado: PASS.
- [ ] **Paso 6: Commit:** `feat(watchdog): add deterministic rules engine`

---

### Tarea 4: Jev simulado y política de la pausa (I1)

**Archivos:**
- Crear: `src/domain/jev.ts`, `src/domain/jev-fixtures.ts`, `src/domain/policy.ts`
- Prueba: `src/domain/jev.test.ts`, `src/domain/policy.test.ts`

**Interfaces:**
- Consume: los tipos de System One, `PauseJudgments`, `PauseDecision` y `ScamType` de la tarea 2.
- Produce:

```ts
// jev.ts
export type JudgmentKind = 'pause' | 'report' | 'advisor';
export function normalize(text: string): string;               // lowercase, strip accents, collapse spaces
export function getJudgments(kind: JudgmentKind, req: SystemOneRequest): SystemOneResponse | null; // THE live-swap point
export function buildPauseRequest(text: string, ctx: { amount: Money; toKeyMasked: string }): SystemOneRequest;
export function buildReportRequest(transcript: string): SystemOneRequest;
export function buildAdvisorRequest(text: string): SystemOneRequest;
export function toPauseJudgments(res: SystemOneResponse): PauseJudgments;
export function judgePauseAnswer(text: string, ctx: { amount: Money; toKeyMasked: string }):
  { request: SystemOneRequest; response: SystemOneResponse | null; judgments: PauseJudgments | null };
// jev-fixtures.ts
export type Fixture = { id: string; kind: JudgmentKind; keywords: string[][]; response: SystemOneResponse };
export const FIXTURES: Fixture[];
// policy.ts
export type PauseInput = { kind: 'button'; answer: 'yes' | 'no' } | { kind: 'text'; judgments: PauseJudgments | null };
export function decideAfterPause(input: PauseInput): PauseDecision;
export function describeDecision(j: PauseJudgments | null, d: PauseDecision): string;
```

**Pedidos.** Todos usan `model: 'jev-latest'` y `state: { text, ...ctx }`. Ids de las preguntas:

| Pedido | Preguntas |
|---|---|
| Pausa | `impersonation` (noul); `scam_type` (choice con las 6 opciones de `ScamType`); `pressure` (score con 4 niveles: ninguna, baja, media, alta) |
| Reporte | `scam_type` (choice) |
| Asesor | `loss_tolerance` (score con 5 niveles, de "sacaría todo de inmediato" a "pondría más plata") |

Las instrucciones se escriben en español, igual que las preguntas del spec §6.6.

**Coincidencia.** `getJudgments` recorre `FIXTURES` del `kind` pedido, en orden. Una fixture coincide si **cada** grupo de `keywords` tiene al menos una palabra contenida en `normalize(state.text)`. Si coincide, devuelve su `response`; si ninguna coincide, devuelve `null`. Es la única función que lee fixtures. El código lleva este comentario: `// live mode: replace this body with a server-side POST to https://api.typesafe.ai/v1/systemone`.

**Fixtures** (todas con `model: 'jev-1.14.0'` y `usage: { input_tokens: 0, output_tokens: 0 }`; probabilidades que suman 1 ± 0,01):

| id | kind | keywords | respuesta |
|---|---|---|---|
| `impersonation-call` | pause | `[['llam','telefono','celular'], ['belo','banco','asesor','funcionario','entidad']]` | impersonation 0.94; scam_type `impersonation_official` 0.88; pressure 2.6 |
| `family-emergency` | pause | `[['hijo','hija','nieto','familiar','sobrino'], ['accidente','urgente','preso','hospital','carcel']]` | impersonation 0.15; `family_emergency` 0.82; pressure 2.8 |
| `own-decision` | pause | `[['regalo','cumpleanos','arriendo','mercado','deuda','prestamo']]` | impersonation 0.03; `none` 0.90; pressure 0.2 |
| `fake-purchase` | report | `[['compr','pedido','tienda','oferta','producto'], ['nunca','llego','pague','envio']]` | scam_type `fake_purchase` 0.86 |
| `risk-low` | advisor | `[['sacaria','retiraria','miedo','nervios','asust']]` | loss_tolerance 0.6 |
| `risk-mid` | advisor | `[['esperaria','esperar','tranquila','paciencia']]` | loss_tolerance 2.1 |
| `risk-high` | advisor | `[['pondria','compraria','aprovech','invertiria']]` | loss_tolerance 3.4 |

La `confidence` de cada choice se calcula como `(pmax - 1/n) / (1 - 1/n)`, siguiendo la documentación.

**Política:**
- `decideAfterPause`:
  - botón `yes` → `hold`;
  - botón `no` → `confirm`;
  - texto con `judgments` en `null` → `buttons`;
  - `impersonation >= 0.8` → `hold`;
  - `scamType !== 'none' && scamTypeP >= 0.7` → `hold`;
  - cualquier otro caso → `confirm`.
- `describeDecision` arma la frase fija de la vista del vigilante, con números en formato es-CO. Ejemplos:
  - `"Suplantación 0,94 ≥ 0,8 → retención"`
  - `"Sin coincidencia en el guion → solo reglas, botones"`

- [ ] **Paso 1: Pruebas que fallan**

```ts
// jev.test.ts
test('normalize strips accents and case', () => assert.equal(normalize('  Me LLAMÓ  uno de Belo '), 'me llamo uno de belo'));
test('impersonation phrase from the script', () => {
  const r = judgePauseAnswer('me llamó uno de Belo diciendo que mi cuenta estaba en peligro', ctx);
  assert.deepEqual(Object.keys(r.request.questions), ['impersonation', 'scam_type', 'pressure']);
  assert.equal(r.judgments?.impersonation, 0.94);
  assert.equal(r.judgments?.scamType, 'impersonation_official');
  assert.equal(r.judgments?.scamTypeP, 0.88);
});
test('tolerates other wording of the same scam', () => {
  assert.equal(judgePauseAnswer('Un asesor del banco me llamaron por teléfono', ctx).judgments?.scamType, 'impersonation_official');
});
test('unmatched text returns null judgments', () => {
  const r = judgePauseAnswer('hola cómo estás', ctx);
  assert.equal(r.response, null); assert.equal(r.judgments, null);
});
test('every fixture has System One shape', () => {
  for (const f of FIXTURES) for (const a of Object.values(f.response.answers)) {
    if (a.type === 'noul') assert.ok(a.noul >= 0 && a.noul <= 1, f.id);
    else { const sum = Object.values(a.probabilities).reduce((x, y) => x + y, 0); assert.ok(Math.abs(sum - 1) <= 0.01, f.id); }
  }
});
// policy.test.ts
test('buttons', () => { assert.equal(decideAfterPause({ kind: 'button', answer: 'yes' }), 'hold'); assert.equal(decideAfterPause({ kind: 'button', answer: 'no' }), 'confirm'); });
test('text thresholds', () => {
  const j = (impersonation: number, scamType: ScamType, scamTypeP: number) => ({ kind: 'text' as const, judgments: { impersonation, scamType, scamTypeP, pressure: 1 } });
  assert.equal(decideAfterPause(j(0.8, 'none', 0.9)), 'hold');
  assert.equal(decideAfterPause(j(0.79, 'none', 0.9)), 'confirm');
  assert.equal(decideAfterPause(j(0.1, 'family_emergency', 0.7)), 'hold');
  assert.equal(decideAfterPause(j(0.1, 'family_emergency', 0.69)), 'confirm');
  assert.equal(decideAfterPause({ kind: 'text', judgments: null }), 'buttons');
});
```

- [ ] **Paso 2: Correr `npm test`.** Esperado: FALLA.
- [ ] **Paso 3: Implementar `jev-fixtures.ts`, `jev.ts` y `policy.ts`** con las firmas y los valores de arriba.
- [ ] **Paso 4: Correr `npm test`.** Esperado: PASS.
- [ ] **Paso 5: Commit:** `feat(jev): add scripted System One judgments and pause policy`

---

### Tarea 5: Pagos, aviso, pausa y retención (I1)

**Archivos:**
- Crear: `src/domain/payments.ts`
- Modificar: `src/domain/actions.ts` (despachar `send`, `confirm-payment`, `cancel-payment`, `pause-answer`, `pause-text` y `call-contact`)
- Prueba: `src/domain/payments.test.ts`

**Interfaces:**
- Consume: `evaluate`, `REASON_COPY`, `judgePauseAnswer`, `decideAfterPause`, `pushMessage` y `nextId`.
- Produce:

```ts
export const PAYMENT_STEPS: readonly { id: PaymentStepId; ms: number }[]; // fx 40, watchdog 120, aml 80, debit 300, credit 200
export function hourIn(timeZone: string, at: number): number;       // Intl.DateTimeFormat hourCycle 'h23'
export function buildWatchdogContext(state: DemoState, intent: PaymentIntent): WatchdogContext;
export function sendPayment(state: DemoState, intent: PaymentIntent): Payment | null; // null = rejected
export function confirmPayment(state: DemoState, paymentId: string): void;
export function cancelPayment(state: DemoState, paymentId: string): void;
export function answerPause(state: DemoState, paymentId: string, answer: 'yes' | 'no'): void;
export function answerPauseText(state: DemoState, paymentId: string, text: string): void;
export function callContact(state: DemoState, userId: UserId): void;
```

**Cómo se arma el contexto del vigilante** (`buildWatchdogContext`):

| Campo | Valor |
|---|---|
| `historyMinor` | pagos `completed` de quien envía, en la misma moneda |
| `recentOutgoingAt` | `createdAt` de esos pagos |
| `isFirstPaymentToKey` | no hay ningún pago `completed` ni `pending-claim` de quien envía a esa llave |
| `hourLocal` | `hourIn(user.timeZone, state.now)` |
| `keyRegisteredAt` y `keyReportStatus` | salen de la llave |

**Qué hace `sendPayment`:**

1. **Validaciones.** Si `isValidMinor` es falso, el saldo no alcanza o la llave no existe, agrega un mensaje de Belo a quien envía y devuelve `null`. En ese caso no crea pago ni toca saldos.
2. **Cálculo del vigilante.** Calcula `watchdog` y crea el pago. El crédito va en la `homeCurrency` de quien recibe y se convierte con `usdToCop` o `copToUsd` si hace falta.
3. **Según el nivel:**

| Nivel | Qué pasa |
|---|---|
| `allow` | Se ejecuta: corren los 5 pasos `done`, se debita a quien envía y se acredita a quien es dueño de la llave, con estado `completed`. Si la llave no tiene dueño, ver la tarea 6. Cada parte recibe un mensaje corto. |
| `notice` | Estado `awaiting-confirm`. Traza: `fx done`, `watchdog waiting`. Mensaje de una línea con la razón principal y los botones "Continuar" (`confirm-payment`) y "Cancelar" (`cancel-payment`). |
| `pause` | Estado `paused`. Traza: `fx done`, `watchdog stopped`. Mensaje: `Un momento, {nombre}. Vas a mandar {monto} a "{holderMasked}".` + hasta 2 razones en el nivel de la persona + `¿Alguien te está pidiendo este pago por teléfono ahora mismo?` Botones "Sí" y "No"; "Llamar a {contacto}" si hay contacto de confianza. `expectsText: { type: 'pause-text', paymentId }`. |
| `hold` | Estado `held`. Traza: `watchdog stopped`. Mensaje con los botones "Llamar a {contacto}" y "Cancelar pago". Aviso al contacto de confianza: `Pausamos un pago de {nombre} de {monto} que parece una estafa. Llámala.` |

4. **Ejecución diferida.** `confirmPayment` ejecuta un pago en `awaiting-confirm` o `paused`: completa los pasos que faltan, en `done`. `cancelPayment` lo deja en `cancelled` sin mover plata. Ninguno de los dos actúa sobre pagos `held` ni sobre pagos ya terminados.
5. **Respuestas a la pausa:**
   - `answerPause` y `answerPauseText` usan `decideAfterPause`.
   - `hold` hace lo mismo que el nivel `hold`.
   - `confirm` agrega el mensaje "¿Seguro que quieres enviarlo?" con los botones Confirmar y Cancelar.
   - `buttons` agrega "No te entendí bien. ¿Alguien te está pidiendo este pago por teléfono?" con los botones Sí y No, y deja el pago en `paused`.
   - `answerPauseText` guarda `payment.pause = { text, request, response, judgments, decision }`. El mensaje con el juicio lleva `jevLabel: true`.
6. **Llamada al contacto.** `callContact` agrega "Llamando a {contacto}… (simulado)" en el chat de quien llama y "{nombre} te está llamando" en el del contacto.

- [ ] **Paso 1: Pruebas que fallan.**
  - **Ayudante de prueba `registeredRosa()`:** parte de `seedState()`, deja a Rosa registrada y dueña de `tel:rosa.demo` (`registeredAt = now - 5 min`), con `balances.COP = 880_000`, y pone el reloj en 22:40 de Bogotá: `Date.parse('2026-11-29T03:40:00Z')`.
  - **`allowed payment completes`:** Andrés envía US$20 a `tel:rosa.demo` con el reloj a las 10:00 de Bogotá.
    - Estado `completed`; `andres.balances.USD` baja 2000; `rosa.balances.COP` sube 80_000.
    - Pasos de la traza: `['fx','watchdog','aml','debit','credit']`, todos en `done`, y la suma de `ms` es 740.
  - **`pause moves no money`:** Rosa envía 880_000 COP a `@seguridad.belo`.
    - Estado `paused`, `watchdog.score` 70 y saldos sin cambios.
    - Último paso de la traza: `{ step: 'watchdog', status: 'stopped' }`.
    - El último mensaje de Rosa tiene botones con las etiquetas `Sí` y `No`, y `expectsText.type === 'pause-text'`.
  - **`pause yes → held, no money moves, contact notified`:** el último mensaje de Andrés contiene `Rosa`.
  - **`pause text impersonation → held with judgments`:** `payment.pause.decision === 'hold'` y `payment.pause.judgments.impersonation === 0.94`.
  - **`pause text unmatched → still paused, buttons again`:** `decision === 'buttons'`, estado `paused` y el último mensaje vuelve a tener `Sí` y `No`.
  - **`notice then confirm completes; cancel moves nothing`.**
  - **`held payment cannot be confirmed`:** `confirmPayment` sobre `held` no cambia saldos ni estado.
  - **`invalid amounts are rejected`** (foco 2): para `0`, `-1`, `1.5` y `880_001`, `sendPayment` devuelve `null`, `payments.length` no cambia, los saldos no cambian y se agrega 1 mensaje a Rosa.
  - **`unknown key is rejected`.**
  - **`same send action id twice creates one payment`** (foco 1): vía `dispatch` con el mismo `id`; `payments.length` sube 1 y el saldo baja una sola vez.
- [ ] **Paso 2: Correr `npm test`.** Esperado: FALLA.
- [ ] **Paso 3: Implementar `payments.ts`** con las firmas de arriba, y despachar las 6 acciones desde `actions.ts`.
- [ ] **Paso 4: Correr `npm test`.** Esperado: PASS.
- [ ] **Paso 5: Commit:** `feat(payments): add send flow with watchdog levels`

---

### Tarea 6: Pago por reclamar, dónde guardar y momentos del guion (I1)

**Archivos:**
- Crear: `src/domain/claims.ts`
- Modificar: `src/domain/payments.ts` (la rama "llave sin dueño" de `allow`) y `src/domain/actions.ts` (`open-claim`, `verify-claim`, `keep-in`, `show-jars` y `beat`; además, `set-level` sigue con el código si hay un reclamo abierto)
- Prueba: `src/domain/claims.test.ts`

**Interfaces:**
- Consume: lo que produce la tarea 5.
- Produce:

```ts
export const CLAIM_OTP = '4821';
export function createClaim(state: DemoState, payment: Payment): PendingClaim; // called by sendPayment when key has no owner
export function openClaim(state: DemoState, claimId: string): void;
export function verifyClaim(state: DemoState, claimId: string, otp: string): void;
export function keepIn(state: DemoState, userId: UserId, currency: Currency): void;
export function showJars(state: DemoState, userId: UserId): void;
export function runBeat(state: DemoState, beat: BeatId): void; // in actions.ts
```

**El flujo:**

1. **Pago a una llave sin dueño.** El pago queda en `pending-claim`.
   - Traza: `fx`, `watchdog`, `aml` y `debit` en `done` (540 ms), más `credit` en `waiting`.
   - Se debita a quien envía y se crea la reclamación con `otp: CLAIM_OTP`.
   - Quien envía recibe "Le avisamos a Mamá. La plata la espera hasta que la reclame."
   - El chat de quien recibe agrega un mensaje `from: 'external'`: `"WhatsApp · Andrés te envió plata con Belo. Recíbela aquí:"`, con la card `whatsapp-link` y el botón "Abrir link" (`open-claim`).
2. **`openClaim`.** Belo saluda con "¡Hola, Rosa! Andrés te mandó plata con Belo." y pregunta "¿Has usado antes una app de banco?".
   - Botón "Sí" → `set-level compact`. Botón "No" → `set-level simple`.
   - Después de `set-level`, si hay un reclamo abierto, sigue con dos mensajes:
     - "Para recibirla, confirma que este celular es tuyo. Te mandamos un código."
     - un mensaje `external` con la card `sms-code` (`"Belo: tu código es 4821. No lo compartas con nadie."`), el botón "Usar el código 4821" y `expectsText: { type: 'verify-claim', claimId }`.
3. **`verifyClaim`.**
   - **Código equivocado:** el mensaje "Ese código no coincide. Revisa el SMS." y nada más cambia.
   - **Código correcto:** marca el reclamo como `claimed`, registra a Rosa, le asigna la llave con `registeredAt = now`, le acredita, completa el paso `credit` en `done` y deja el pago en `completed`.
     - Rosa recibe "¡Listo! Recibiste {monto} de Andrés." con la card `jars` y la pregunta "¿Dónde los quieres guardar?", con los botones "En pesos" y "En dólares".
     - Andrés recibe "Rosa recibió tu plata." con el botón "Enviar US$20".
   - **Reclamo ya cobrado:** no hace nada.
4. **`keepIn('USD')`.** Convierte todo el saldo en COP a USD con `copToUsd`. `keepIn('COP')` solo confirma con un mensaje. Las dos opciones agregan una explicación fija y corta de qué es un dólar digital, con la card `jars`.
5. **`runBeat`.** Fija el reloj y luego actúa:

| Beat | Reloj | Acción |
|---|---|---|
| `b1-first-send` | `2026-11-28T15:00:00Z` | `send` de Andrés por US$200 a `tel:rosa.demo` |
| `b3-second-send` | +5 min | `send` de Andrés por US$20 a `tel:rosa.demo` |
| `b4-scam-attempt` | `2026-11-29T03:40:00Z` (22:40 en Bogotá) | `send` de Rosa a `@seguridad.belo` por **todo su saldo en COP** |
| `b5-fake-purchase` | `2026-11-29T16:00:00Z` | `send` de Rosa por 150_000 COP a `@ofertas.tienda` |

- [ ] **Paso 1: Pruebas que fallan**
  - **`send to unregistered phone creates a claim`:** el estado es `pending-claim`; la suma de `ms` en `done` es 540; el último paso es `{ step: 'credit', status: 'waiting' }`; `claims.length` es 1; el último mensaje de Rosa trae la card `whatsapp-link`; Andrés baja 20000 en USD.
  - **`open-claim asks the level question`:** el último mensaje de Rosa tiene botones con las etiquetas `['Sí', 'No']`.
  - **`wrong otp does not credit`** (foco 3): Rosa sigue en COP 0, `claimed` es `false` y el pago sigue en `pending-claim`.
  - **`right otp registers and credits`:** `rosa.registered` es `true`; el dueño de la llave es `rosa`; `rosa.balances.COP` es 800_000; el pago queda `completed` con `credit` en `done`.
  - **`claiming twice credits once`** (foco 3): llamar a `verifyClaim` dos veces con el código correcto deja COP 800_000.
  - **`keep-in USD converts all COP`:** con 800_000 COP, quedan USD 20000 y COP 0.
  - **`full story reaches the pause at beat 4`:** corre vía `dispatch` la secuencia `reset → beat b1 → open-claim → set-level simple → verify-claim 4821 → keep-in COP → beat b3 → beat b4`. El último pago queda `paused`, con `watchdog.score` 70 y las razones `['R1','R2','R3','R4','R7']`.
  - **`beat 5 after the story lands in notice`:** luego de lo anterior, el `beat b5` deja el último pago en `awaiting-confirm`.
- [ ] **Paso 2: Correr `npm test`.** Esperado: FALLA.
- [ ] **Paso 3: Implementar `claims.ts`, la rama sin dueño en `payments.ts` y `runBeat`** en `actions.ts`.
- [ ] **Paso 4: Correr `npm test`.** Esperado: PASS. Luego `npm run verify`, también PASS.
- [ ] **Paso 5: Commit:** `feat(claims): add claimable payments and demo beats`

---

### Tarea 7: Escenario, teléfonos y chat (I2, desde que esté la tarea 2)

**Archivos:**
- Crear: `src/ui/useDemoState.ts`, `src/ui/Phone.tsx`, `src/ui/Chat.tsx`, `src/ui/PresenterControls.tsx`, `src/app/globals.css`
- Modificar: `src/app/page.tsx` (reemplaza la provisoria) y `src/app/layout.tsx` (importa `globals.css`)

**Interfaces:**
- Consume: `DemoState`, `ChatMessage`, `ActionBody`, `UiLevel` y `BeatId` (tarea 2); `GET` y `POST` `/api/demo`.
- Produce:
  - `useDemoState(): { state: DemoState | null; send(body: ActionBody, messageId?: string): Promise<void>; busy: boolean }`. Cada envío usa `crypto.randomUUID()` como `id`.
  - `<Phone title: string; children />`
  - `<Chat userId: UserId; state: DemoState; onAction(body: ActionBody, messageId?: string) />`
  - `<PresenterControls onAction(body: ActionBody) />`

**Comportamiento:**

- **`page.tsx`** es un client component. Arma la grilla del spec §6.1: a la izquierda el teléfono de Andrés (nivel `compact`), en el centro un espacio reservado para el grafo (tarea 9) y a la derecha el de Rosa. El nivel de Rosa sale de `user.level`, y es `simple` mientras sea `null`.
- **`Chat`:**
  - dibuja los mensajes según `from` (Belo a la izquierda, la persona a la derecha y `external` con estilo de notificación);
  - muestra los botones solo si `!answered`, y cada botón llama `onAction(button.action, message.id)`;
  - si el último mensaje tiene `expectsText`, muestra un input de texto con "Enviar", que arma la acción según `TextTarget`:
    - `pause-text` → `{ type: 'pause-text', paymentId, text }`
    - `verify-claim` → `{ type: 'verify-claim', claimId, otp: text }`
    - `report-text` → `{ type: 'report-text', userId, text }`
    - `advisor-answer` → `{ type: 'advisor-answer', userId, text }`
  - si `message.jevLabel` es verdadero, muestra `JEV_LABEL` en letra chica debajo del mensaje;
  - cuando la persona está registrada, muestra fijas al pie las acciones rápidas "Mi plata" (`show-jars`), "Quiero ahorrar" (`advisor-start`) y "Me estafaron" (`report-start`);
  - hace scroll al último mensaje.
- **`PresenterControls`** muestra un botón por momento ("1 · Andrés envía US$200", "3 · Andrés envía US$20", "4 · Llamada del estafador", "5 · Compra falsa") y "Reiniciar".
- **Estilo:** cálido y tranquilo, sin colores de alarma. Botones grandes en el nivel `simple`.

- [ ] **Paso 1: Implementar los cinco archivos** con las firmas de arriba.
- [ ] **Paso 2: Revisión manual** con `npm run dev` en `http://localhost:3000`:
  - aparecen los dos teléfonos y el chat de Andrés muestra "Enviar US$200";
  - "Reiniciar" devuelve el estado inicial;
  - un botón ya usado desaparece.

  Con las tareas 5 y 6 integradas, además: "Enviar US$200" muestra en el teléfono de Rosa el mensaje de WhatsApp.
- [ ] **Paso 3: Correr `npm run verify`.** Esperado: PASS.
- [ ] **Paso 4: Commit:** `feat(ui): add demo stage with phones and chat`

---

### Tarea 8: Niveles, tarjetas y voz (I2)

**Archivos:**
- Crear: `src/ui/Cards.tsx`, `src/ui/VoiceInput.tsx`
- Modificar: `src/ui/Chat.tsx` y `src/app/globals.css`

**Interfaces:**
- Consume: `Card`, `DemoState`, `formatMoney` y `JEV_LABEL`.
- Produce:
  - `<CardView card: Card; state: DemoState; onAction(body: ActionBody) />`
  - `<VoiceInput onText(text: string) />`

**Comportamiento:**

- **Niveles:** `simple` usa letra ≥ 20 px, botones a todo el ancho y un mensaje destacado a la vez. `compact` usa letra de 14 px y botones en línea. Un interruptor discreto en cada teléfono despacha `set-level`.
- **Cards:**

| Card | Qué muestra |
|---|---|
| `jars` | Dos frascos (pesos y dólares) con altura proporcional al saldo y el monto con `formatMoney`; si hay `savingsCop`, un tercer frasco "Meta" |
| `whatsapp-link` | Tarjeta estilo WhatsApp con el link |
| `sms-code` | Burbuja de SMS |
| `case` | Tipo de estafa, monto, llave y cronología, con los botones "Enviar reporte" y "Cambiar tipo" |
| `complaint` | Texto de la denuncia con el botón "Copiar" (`navigator.clipboard.writeText`) |
| `savings-plan` | Plan del asesor (tarea 11) |

  Las cards `case` y `complaint` leen `state.cases`; la de `savings-plan` lee el usuario.
- **`VoiceInput`:**
  - usa `window.SpeechRecognition ?? window.webkitSpeechRecognition` con `lang = 'es-CO'` e `interimResults = false`; el botón 🎤 alterna grabar y detener, y al terminar llama `onText(transcript)`;
  - si la API no existe o el permiso se niega, no muestra el botón;
  - el input de texto siempre está (foco 5);
  - se monta junto al input de texto cuando hay `expectsText`.

- [ ] **Paso 1: Implementar** con las firmas de arriba.
- [ ] **Paso 2: Revisión manual:**
  - en Chrome, dictar "me llamó uno de Belo" en la pausa llena el texto y lo envía;
  - en Firefox, que no tiene la API, no aparece 🎤 y sí aparece el input (foco 5);
  - el interruptor de nivel cambia el tamaño en vivo;
  - los frascos reflejan los saldos después de `keep-in`.
- [ ] **Paso 3: Correr `npm run verify`.** Esperado: PASS.
- [ ] **Paso 4: Commit:** `feat(ui): add adaptive levels, cards and voice input`

---

### Tarea 9: Grafo del pago y vista del vigilante (I2)

**Archivos:**
- Crear: `src/ui/PaymentGraph.tsx`, `src/ui/WatchdogView.tsx`
- Modificar: `src/app/page.tsx` (columna central)

**Interfaces:**
- Consume: `Payment`, `PAYMENT_STEPS`, `REASON_COPY`, `JEV_LABEL` y `describeDecision`.
- Produce:
  - `<PaymentGraph payment: Payment | null />`: muestra el último pago de `state.payments`.
  - `<WatchdogView payment: Payment />`

**Comportamiento:**

- **`PaymentGraph`:**
  - SVG con 5 nodos en el orden de `PAYMENT_STEPS` y etiquetas en español: Tasa, Vigilante, Antilavado, Débito, Crédito. Debajo de cada nodo va su tiempo.
  - Un punto avanza paso a paso en cámara lenta: cada paso dura `ms × SLOW_MOTION` en tiempo real, con `const SLOW_MOTION = 4`.
  - Arriba, un contador muestra los ms simulados acumulados con el rótulo "(simulado)".
  - **Colores por estado:** `done` verde; `stopped` ámbar, y ahí se detiene el punto; `waiting` gris punteado.
  - En `pending-claim`, la etiqueta del nodo Crédito pasa a "Esperando a {nombre}".
  - Cuando cambia la traza del mismo pago, solo anima los pasos nuevos.
  - Abajo va un carril punteado lento, "Compensación entre países en dólares digitales (después, por lotes)", con un punto que lo recorre después de cada pago entre países `completed`.
- **`WatchdogView`** se abre bajo el grafo cuando el nivel del pago es `pause` o `hold`, o cuando hay `payment.pause`. Muestra:
  - las razones, con `REASON_COPY[id].compact` y su peso;
  - el puntaje y el nivel;
  - si hay `payment.pause`: las `instructions` de cada pregunta del pedido, las respuestas (`noul` en %, `choice` con su probabilidad y `score`), `JEV_LABEL` y la línea de `describeDecision`.

- [ ] **Paso 1: Implementar** con las firmas de arriba.
- [ ] **Paso 2: Revisión manual:**
  - momento 1: el grafo se detiene en "Esperando a Rosa" con el contador en 540 ms; tras el reclamo, completa Crédito;
  - momento 3: recorre todo hasta 740 ms;
  - momento 4: se frena en Vigilante en ámbar y se abre la vista con R1, R2, R3, R4 y R7 y el puntaje 70;
  - tras dictar la frase, la vista muestra el juicio, el rótulo y "Suplantación 0,94 ≥ 0,8 → retención".
- [ ] **Paso 3: Correr `npm run verify`.** Esperado: PASS.
- [ ] **Paso 4: Commit:** `feat(ui): add payment graph and watchdog view`

---

### Tarea 10: Reporte de estafa (I1, P1)

**Archivos:**
- Crear: `src/domain/report.ts`
- Modificar: `src/domain/actions.ts` (`report-start`, `report-text`, `report-set-type`, `report-submit`)
- Prueba: `src/domain/report.test.ts`

**Interfaces:**
- Consume: `getJudgments`, `buildReportRequest`, `pushMessage` y `formatMoney`.
- Produce:

```ts
export type Candidates = { amountsCop: number[]; keys: string[]; dates: string[] };
export function extractCandidates(text: string): Candidates;
export function buildCase(state: DemoState, userId: UserId, transcript: string): ScamCase;
export function complaintText(state: DemoState, c: ScamCase): string;
export function startReport(state: DemoState, userId: UserId): void;
export function reportText(state: DemoState, userId: UserId, text: string): void;
export function setCaseType(state: DemoState, caseId: string, scamType: ScamType): void;
export function submitReport(state: DemoState, caseId: string): void;
```

**Extracción** (sobre `normalize` sin quitar `$`, `.` ni `@`):

- **Montos:** `/\$?\s?(\d{1,3}(?:\.\d{3})+|\d+)(\s?mil)?/g`. El valor son los dígitos sin puntos, × 1000 si va seguido de "mil". Se descartan los menores a 1000.
- **Llaves:** `/@[a-z0-9._-]+/g`.
- **Fechas:** las palabras `hoy`, `ayer` y `antier`, y `\d{1,2}/\d{1,2}`.

Las funciones llevan este comentario: `// scripted stand-in for Jev's select-don't-generate step: code extracts, ledger match picks`.

**Armado del caso** (`buildCase`):

- **Pago del caso:** el último pago `completed` de la persona cuyo monto o llave coincide con un candidato; si ninguno coincide, el último pago `completed`; si no hay pagos, `null`.
- **Tipo de estafa:** `scamType` y `scamTypeP` salen de `getJudgments('report', buildReportRequest(transcript))`. Si devuelve `null`, quedan en `none` y 0.

**Flujo:**

1. `startReport` dice "Cuéntame qué pasó. Puedes hablar o escribir." y espera texto (`report-text`).
2. `reportText` arma el caso en `draft` y agrega un mensaje con la card `case` y `jevLabel: true`. Botones:
   - "Enviar reporte" → `report-submit`;
   - "Cambiar tipo" → un mensaje con un botón por cada `ScamType`, que despacha `report-set-type`.
3. **`submitReport`** pasa el caso a `submitted` y genera `complaintText`. Además:
   - si la llave tiene dueño, agrega `{ caseId, key, status: 'pending-review' }` a `complianceQueue` y muestra "En revisión por el equipo de Belo";
   - si la llave estaba en `none`, la pasa a `reported`;
   - Rosa recibe un mensaje con la card `complaint`;
   - el contacto de confianza recibe "Rosa reportó una estafa. Llámala.";
   - **no mueve saldos.**

**Texto de la denuncia** (`complaintText`), fijo, una línea por dato:
- `Borrador de denuncia (generado por Belo, datos simulados)`
- `Denunciante: {nombre}`
- `Hechos: {transcript}`
- `Tipo de estafa: {etiqueta}`
- `Monto: {monto}`
- `Llave de destino: {llave}`
- `Fecha del pago: {fecha es-CO}`

- [ ] **Paso 1: Pruebas que fallan**

```ts
test('extracts Spanish amounts, keys and dates', () => {
  assert.deepEqual(extractCandidates('Le pagué $150.000 a @ofertas.tienda ayer y nunca llegó el pedido'),
    { amountsCop: [150000], keys: ['@ofertas.tienda'], dates: ['ayer'] });
  assert.deepEqual(extractCandidates('pagué 150 mil').amountsCop, [150000]);
  assert.deepEqual(extractCandidates('pagué 150000 hace 2 días').amountsCop, [150000]);
  assert.deepEqual(extractCandidates('ciento cincuenta mil').amountsCop, []);
});
```

- Además:
  - **`case picks the matching payment`:** después de la historia de la tarea 6 más el momento 5 confirmado, el caso apunta al pago de 150_000 a `@ofertas.tienda`, con `scamType` `fake_purchase` 0.86.
  - **`words-only amount falls back to the last completed payment`** (foco 4).
  - **`submit queues review, marks key reported, moves no money`:** `complianceQueue.length === 1`; `keys['@ofertas.tienda'].reportStatus === 'reported'`; los saldos de todos quedan iguales antes y después; `complaintText` contiene `Ofertas` o la llave, y `150.000`.
  - **`submitting twice is idempotent`:** un segundo `submitReport` no duplica la entrada en la cola.
- [ ] **Paso 2: Correr `npm test`.** Esperado: FALLA.
- [ ] **Paso 3: Implementar `report.ts`** y despachar las 4 acciones.
- [ ] **Paso 4: Correr `npm test`.** Esperado: PASS.
- [ ] **Paso 5: Commit:** `feat(report): add scam report with compliance review queue`

---

### Tarea 11: Asesor (P2, quien se libere primero)

**Archivos:**
- Crear: `src/domain/advisor.ts`
- Modificar: `src/domain/actions.ts` (`advisor-start`, `advisor-goal`, `advisor-answer`, `advisor-confirm`)
- Prueba: `src/domain/advisor.test.ts`

**Interfaces:**
- Consume: `getJudgments` y `buildAdvisorRequest`.
- Produce:

```ts
export const RISK_QUESTIONS: string[]; // [0] = 'Si tus $100.000 bajaran a $90.000 por una semana, ¿qué harías?'
export const ADVISOR_DISCLAIMER = 'Esto es educación y simulación, no una recomendación de inversión personalizada.';
export const CREDIT_EDUCATION: string; // fixed Spanish copy: start a credit history with a small amount paid on time
export function profileFromScores(scores: number[]): RiskProfile | null;  // avg < 1.5 conservative, < 2.75 moderate, else bold; [] → null
export function savingsPlan(goalCop: number, months: number, profile: RiskProfile):
  { monthlyCop: number; dollarShare: number; monthlyPesosCop: number; monthlyDollarsUsd: Money; dollarsIfDown5Cop: number; dollarsIfUp5Cop: number };
// dollarShare: conservative 0, moderate 0.5, bold 0.8; monthlyCop = ceil(goal/months);
// dollarsIf*: value in COP of the dollar part of the whole goal if the synthetic rate moves -5% / +5% (illustrative, not a forecast)
export function startAdvisor(state: DemoState, userId: UserId): void;
export function setGoal(state: DemoState, userId: UserId, goalCop: number, months: number): void;
export function answerRisk(state: DemoState, userId: UserId, text: string): void;
export function confirmPlan(state: DemoState, userId: UserId): void;
```

**Flujo:**

1. `startAdvisor` pregunta "¿Para qué quieres ahorrar?" con el botón "$300.000 en 3 meses" (`advisor-goal`, 300000, 3).
2. `setGoal` guarda la meta y hace la pregunta de perfil siguiente (`RISK_QUESTIONS[riskScores.length % length]`) con `expectsText: { type: 'advisor-answer' }`.
3. `answerRisk` usa `getJudgments('advisor', …)`:
   - **Si hay respuesta:** agrega el `score` a `riskScores` y muestra el perfil, con `jevLabel: true`, la card `savings-plan`, `ADVISOR_DISCLAIMER` y los botones "Confirmar plan" (`advisor-confirm`) y "Otra pregunta" (vuelve al paso 2). Agrega un segundo mensaje fijo, `CREDIT_EDUCATION`, sobre cómo empezar un historial de crédito con un monto pequeño (spec §6.8).
   - **Si es `null`:** ofrece 3 botones fijos que despachan `advisor-answer` con un texto que sí coincide: "Los sacaría", "Esperaría" y "Pondría más".
4. `confirmPlan` es la única acción del asesor que mueve plata, y solo porque la persona la confirmó:
   - mueve `monthlyPesosCop` de `balances.COP` a `savingsCop`;
   - convierte la parte en dólares con `copToUsd`;
   - si el saldo no alcanza, avisa y no mueve nada;
   - después muestra la card `jars`.

- [ ] **Paso 1: Pruebas que fallan**
  - **`profile thresholds`:** `[1.4]` da `conservative`; `[1.5]` da `moderate`; `[2.75]` da `bold`; `[]` da `null`.
  - **`plan math`:** `savingsPlan(300000, 3, 'moderate')` da `monthlyCop` 100000, `monthlyPesosCop` 50000 y `monthlyDollarsUsd` `{ currency: 'USD', minor: 1250 }`; `dollarsIfDown5Cop` es 142500 y `dollarsIfUp5Cop` es 157500.
  - **`risk answer via scripted Jev`:** "Esperaría tranquila a que suba" agrega 2.1 a `riskScores`.
  - **`unmatched answer offers fixed options`:** el último mensaje tiene 3 botones.
  - **`confirm moves only the confirmed monthly amount`:** también se prueba que, sin saldo suficiente, los saldos no cambian.
- [ ] **Paso 2: Correr `npm test`.** Esperado: FALLA.
- [ ] **Paso 3: Implementar `advisor.ts`** y despachar las acciones.
- [ ] **Paso 4: Correr `npm test`.** Esperado: PASS.
- [ ] **Paso 5: Commit:** `feat(advisor): add progressive risk profile and savings plan`

---

### Tarea 12: README, guion y video de respaldo (los dos; lunes 2026-11-30)

**Archivos:**
- Modificar: `README.md`, reescrito en español
- Crear: `docs/demo-guion.md`

**Contenido:**

- **`README.md`** explica:
  - qué es y para quién;
  - la historia en 6 momentos;
  - cómo correrlo (`npm ci`, `npm run dev` y `npm run verify`);
  - la arquitectura en un párrafo;
  - qué es simulado: Jev, los tiempos, los rieles, el KYC y los datos;
  - "lo que no hace": ningún agente mueve plata y la retención tras un reporte va a revisión humana;
  - un enlace al spec y al planteamiento del reto.

  Elimina la promesa actual de "notificación a los organismos de control" por IA agéntica.
- **`docs/demo-guion.md`** tiene tres partes:
  - **Los momentos del guion:** qué botón toca el presentador, qué dice y la frase exacta para la voz.
  - **Checklist previo:**
    - Chrome, con permiso de micrófono;
    - `npm run build && npm start`, que es más estable que `dev`;
    - "Reiniciar" antes de empezar;
    - en el momento 2, elegir "En pesos", porque el momento 5 necesita pesos;
    - un plan B si falla la voz: escribir la frase.
  - **Cómo grabar el video de respaldo:** una grabación de pantalla de la página completa corriendo los momentos 1 a 6.

- [ ] **Paso 1: Escribir los dos documentos.**
- [ ] **Paso 2: Ensayo completo cronometrado.** Esperado: los momentos 1 a 4 entran en ≤ 3 minutos. Si no entran, se recorta según el spec §7.
- [ ] **Paso 3: Grabar el video.** El archivo queda fuera del repo; se sube donde lo pida el hackathon.
- [ ] **Paso 4: Correr `npm run verify`.** Esperado: PASS.
- [ ] **Paso 5: Commit:** `docs: rewrite README and add demo script`
