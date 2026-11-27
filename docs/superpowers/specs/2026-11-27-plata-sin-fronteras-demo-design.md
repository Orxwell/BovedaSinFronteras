# Plata sin fronteras: demo design

Date: 2026-11-27 · Status: draft for approval · Challenge statement: [docs/reto-belo-plata-sin-fronteras.md](../../reto-belo-plata-sin-fronteras.md)

## 1. Intent

We are building a hackathon entry for the Belo challenge "Plata sin fronteras". The goal is the highest possible score on the challenge rubric (section 07): Impact 30%, Innovation 20%, Technical feasibility 20%, Implementation feasibility 20%, Presentation 10%.

The entry is **one end-to-end user story**, not four separate mini-demos. It touches all four challenge ideas, and **fraud protection ("Seguridad sin fricción") is the piece built in depth**. The other three are thin but working.

Why one story: the challenge asks for a demo that "muestra el flujo completo desde el punto de vista de la persona". It also asks "¿funciona de punta a punta?" (Technical feasibility). One coherent flow beats four half-finished ones on both counts.

Why security is the deep piece: it is the most dramatic moment of the story. It works end to end with deterministic rules. Belo could adopt it soon, which helps Implementation feasibility. It gives the AI a clear, defensible role, which helps Innovation.

## 2. The story

All people and data are fictional.

- **Andrés**, 29, freelancer in Santiago (Chile). Experienced app user.
- **Rosa**, 68, his mother, in Bogotá (Colombia). She has never had a bank account. She uses WhatsApp.

Demo beats, in order:

1. **Cross-border payment.** Andrés sends US$200 to Rosa. The payment graph animates the happy path: Rosa receives COP 800.000 in under a second (simulated). Synthetic rate: 4.000 COP/USD.
2. **First time.** Rosa opens Belo for the first time. The app asks one question to pick her level, and she gets the simple level. She learns what a digital dollar is through a "jars" card, then chooses where to keep the money (pesos or dollars).
3. **Scam stopped.** A fake "Belo advisor" phones Rosa (the presenter narrates this part). He pushes her to move all her money to a "safe account". She tries to send her whole balance to a new key at 22:40. The watchdog pauses the payment and asks her one question. She answers by voice: "me llamó uno de Belo diciendo que mi cuenta estaba en peligro". Recorded Jev judgments flag impersonation, and the payment is escalated to a hold. Rosa sees "Llamar a Andrés", and Andrés gets a notification. On the graph, the packet stops at the Vigilante node, and the node expands to show how it decided.
4. **Scam not stopped, then reported.** In a second scenario, Rosa pays COP 150.000 for a fake online purchase. It is low risk, so it passes with a soft notice only. Later she taps "Me estafaron", tells what happened in a voice note, and the app builds the case. She reviews it and submits.
5. **Advisor.** Rosa says she wants to save COP 300.000 for December. The advisor asks one risk-profile question, shows her profile, and proposes a savings plan with a simulator. She confirms. The advisor never moves money by itself.

## 3. Goals, non-goals, anti-goals

**Goals**
- A demo that runs the full story above, reliably, on stage.
- A real, tested, framework-free rules engine for the watchdog that Belo could lift as-is.
- A visible explanation of *why* the payment is fast and *why* the watchdog decided what it did.
- A UI that a 68-year-old first-time user and a tech-savvy 29-year-old can both use.

**Non-goals**
- Integration with real payment rails, real KYC, or real FX.
- Account-takeover detection beyond 1–2 rules. The depth goes to scams where the user is tricked into paying ("authorized push payment" scams).
- Real credit underwriting or executing investments.
- A native mobile app.

**Anti-goals**: these make the entry fail even if it technically works.
- Any AI or agent that moves, freezes, or releases money on its own. The challenge excludes "agentes que operan solos con la plata de la persona".
- Real personal data anywhere: code, fixtures, logs, screenshots.
- Presenting recorded AI answers or simulated latencies as live or measured.
- Scare-heavy UI or dark patterns toward elderly users.

## 4. Constraints

- **Challenge rules:** working prototype; simulated data or public services only; deliverables are a pitch, a demo, and a **public repository**.
- **Colombian regulation.** Respect it; the demo never proposes evading controls. Verify the following before the pitch. This document does not assert specifics:
  - Investment advice is regulated by the Superintendencia Financiera (SFC). The advisor is framed as **education and simulation**, not personalized investment advice.
  - Cross-border transfers go through the exchange regime (authorized FX intermediaries). Belo's actual license status in Colombia is unverified.
  - AML controls (SARLAFT) belong in the payment path.
  - Habeas data (Ley 1581 de 2012) applies to the shared list of reported keys and to voice notes.
- **No autonomous money movement:** deterministic rules decide; AI only interprets language; the user or a human reviewer makes every final call.
- **Dependencies:** under the owner's rules, every new dependency needs its need, license, and maintenance justified, plus approval before it is installed.
- **Assumptions, unconfirmed by the team:** delivery around **2026-12-01**, inferred from the "Construye en Público" deadline; team size and skills unknown.

## 5. Approaches considered

| Approach | Trade-off | Decision |
|---|---|---|
| **A. One TypeScript app** (Next.js frontend + API routes, one deploy) | One language and one deploy. Slower if the team is Python-first. | **Chosen** by the team. |
| B. Web frontend + Python backend (FastAPI) | Comfortable for a Python team. Two projects and two deploys, with more glue between them. | Rejected. |
| C. Static site, no server | Fastest to build. Jev cannot be called live without exposing the key, so every AI answer is canned and "funciona de punta a punta" suffers. | Rejected. |

**Jev (TypeSafe System One) is simulated by default.** The team has never used Jev. Decision: recorded judgments that follow the **same request/response shape** as the real System One API. An optional live mode at the end swaps one function for the real HTTP call. See section 8.

## 6. Decided design

### 6.1 Demo stage

One page, built for a projector:

```
┌───────────┬────────────────────────────────────────────┬───────────┐
│  Andrés   │  Payment graph (SVG)                       │   Rosa    │
│  phone    │  Debit ─ FX lock ─ Watchdog ─ AML ─ Credit │   phone   │
│ (compact) │  ......... async settlement lane ......... │  (simple) │
│           │  [Watchdog view expands here when needed]  │           │
└───────────┴────────────────────────────────────────────┴───────────┘
                  presenter controls: next beat · reset
```

Both phones are live UI driven by the same backend state. Presenter controls advance the scripted beats and reset the synthetic data.

### 6.2 UI: chat-based, two levels

- **The whole app is a conversation with Belo.** Each operation is a message; each answer is a button. Voice input is available in place of tapping.
- **Two levels in the same chat:**
  - *Simple* (Rosa): one question per message, large text, a voice button, explanations built on everyday examples.
  - *Compact* (Andrés): denser messages and shortcuts.
- **Level choice:** first-run question "¿Has usado antes una app de banco?". The user can switch at any time. Deterministic, no AI.
- **Jars card:** shown when the user asks for their balance or when the app explains digital dollars. One jar for pesos, one for dollars; saving means moving money between jars.
- **Everything happens in the chat:** the watchdog pause, the advisor's profile question, and the scam report are all chat messages.

Watchdog pause message (simple level):

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

### 6.3 Cross-border payment mechanism (thin)

The proposed mechanism, for the pitch and the animation:

- **Pre-funded liquidity per country.** Belo holds balances in each country. The sender is debited, and the receiver is credited immediately from the destination-country pool.
- **Async net settlement.** Cross-country rebalancing happens later in digital dollars, in batches, off the user's critical path. The user waits only as long as a local payment takes.
- **Fewer rejections.** Before sending, the destination key is validated and the masked holder name is shown, as Bre-B does.
- **In-path controls:** FX quote lock, AML/sanctions screening, and the watchdog.

**Simulated target timings.** Use these exact numbers everywhere and label them "simulated":

| Step | Target |
|---|---|
| Debit (sender) | 300 ms |
| FX quote lock | 40 ms |
| Watchdog | 120 ms (budget < 200 ms) |
| AML / sanctions screening | 80 ms |
| Credit (receiver) | 200 ms |
| **Total the receiver waits** | **740 ms** |
| Async settlement | batched, minutes (shown as a slow background lane) |

### 6.4 Watchdog (deep)

**Shape.** A pure TypeScript module with no framework imports:
`evaluate(payment, context) → { level, score, reasons[] }`
Rule weights and level thresholds live in one config table. The module runs on the payment's hot path; Jev never does.

**Rules.** Authorized push payment scams, the focus:

| ID | Signal |
|---|---|
| R1 | First payment from this user to this key |
| R2 | Amount ≥ 80% of available balance |
| R3 | Amount well above the user's usual payments (or user has little history) |
| R4 | Unusual hour for this user (default for new users: 22:00–06:00) |
| R5 | Velocity: several outgoing payments in a short window |
| R6 | Destination key reported by other users. Two tiers: *reported, unverified* gets a moderate weight; *confirmed by compliance* gets a high weight |
| R7 | Destination key registered very recently |

Account-takeover rules, 1–2 only:

| ID | Signal |
|---|---|
| T1 | New device combined with a sensitive action shortly after |
| T2 | PIN/credential reset followed by a payout shortly after |

**Levels:**

| Level | User experience |
|---|---|
| `allow` | No friction. |
| `notice` | One-line soft notice in the chat; user continues with one tap. |
| `pause` | The question message shown above. The user can answer with buttons or with voice/text. |
| `hold` | The payment is held, not executed. The user can cancel or call their trusted contact or Belo. The trusted contact is notified. A hold never moves money. Releasing a hold is out of scope for the demo. |

**Escalation from `pause`:**
- If the user taps "Sí", the payment escalates to `hold`.
- If the user answers in their own words, the answer goes to Jev (section 6.5). Policy in code: impersonation ≥ 0.8, **or** a scam type other than `none` with probability ≥ 0.7, escalates to `hold`. Otherwise the user may proceed with one confirmation.
- If the text is unrecognized in recorded mode, or Jev errors or times out in live mode, the result falls back to rules only and the buttons.

**Explanations.** Every reason maps to a fixed, human-written sentence per UI level. No generative text anywhere.

**Trusted contact.** Rosa names Andrés. He is notified on `pause`→`hold` and on scam reports. He never approves, blocks, or moves anything.

### 6.5 Jev judgments

Jev (TypeSafe's System One model) returns typed judgments with probabilities, not text.

**On the pause answer.** One request, three questions over the same state (the user's words plus payment context):
- `noul` (yes/no probability): does the description indicate someone impersonating a bank, Belo, a government agency, or an authority?
- `choice`: scam type, one of `impersonation_official`, `family_emergency`, `investment_promise`, `fake_purchase`, `romance`, `none`.
- `score`: pressure/urgency described, on levels none / low / medium / high.

**On the scam report** (section 6.6):
- `choice`: scam type, same options.
- **Select, don't generate.** Code extracts candidate amounts, dates, and keys from the transcript. Jev selects which candidate is the amount and the key involved.

**On the advisor:**
- `score` loss tolerance from the user's free-text answer, on five ordered levels.

**Scripted mode (default).** A fixtures file holds, per scripted scenario, the request that would be sent and the response, in the real API's shape. The builder must take that shape from the current TypeSafe API docs.

- **Matching:** input is matched to a scenario by keyword sets, not exact phrases. Example: a mention of a call plus "Belo", "banco", or "asesor" maps to the impersonation scenario. This tolerates speech-recognition errors. Anything that matches no scenario triggers the rules-only fallback. *(Builder-check resolution, my call.)*
- **Backup input:** the presenter can always type the scripted phrase instead of speaking it.
- **Honest labels.** If the fixture values were captured from real Jev calls, the label reads *"Respuesta de Jev grabada. En producción: una llamada a la API de System One."* If they were written by hand, it reads *"Respuesta simulada de Jev (ilustrativa). En producción: una llamada a la API de System One."* Hand-written values must never be labeled "grabada". *(Builder-check resolution, my call.)*

**Watchdog view.** When the graph's Vigilante node expands, it shows:
- which rules fired and their weights,
- the score and level,
- the question sent to Jev,
- the typed answers with probabilities,
- the policy line that decided.

### 6.6 Scam report (victim path)

1. The user taps "Me estafaron" in the chat.
2. The user tells what happened in a voice note, or types it.
3. Code builds the case. The timeline and amount come from the synthetic ledger; the scam type and the selected fields come from Jev.
4. The user reviews and edits the case card, then submits.
5. Submitting triggers these, all simulated:
   - The case is filed with Belo.
   - If the destination is a Belo account, a **preventive hold request is queued for human compliance review**. The UI says "en revisión por el equipo de Belo". It is never triggered automatically by Jev or by the report alone.
   - The destination key becomes *reported, unverified* for rule R6.
   - A pre-filled complaint text for the Fiscalía is generated. The user sends it herself, by copy or download.
   - The trusted contact is notified.

### 6.7 Advisor (thin)

- One short risk-profile question per visit, answered in free text. Jev scores loss tolerance, and code averages the answers into a profile: conservative, moderate, or bold. The user can see and correct it.
- Goal-based: the user states a goal, and the advisor shows a simulator comparing saving in pesos against digital dollars, using **synthetic** rates with no forecasts. It adds an education card on building credit history with a small amount.
- Every action requires the user's confirmation. A visible disclaimer says this is education and simulation, not personalized investment advice.

### 6.8 Data

- Synthetic, in-memory state, seeded at start and resettable from the presenter controls: users, balances, ledger, keys, reported-key list, and Jev fixtures.
- Keys are obviously fake alias-style keys, such as `@rosa.demo`. No real phone numbers, IDs, or names.

### 6.9 Voice

- Browser speech recognition, which is reliable only in Chromium-based browsers and sends audio to the browser vendor.
- A typed fallback is always present.
- Spanish (Colombia) recognition quality is unverified; test it early.
- The demo machine must run Chrome.

## 7. Priorities

My call; the team can reorder.

| Priority | Scope |
|---|---|
| P0 | Demo stage; chat UI with both levels; beats 1–3 (payment graph happy path, first time with jars, scam stopped); watchdog module with tests; recorded Jev mode and watchdog view. |
| P1 | Beat 4: scam report. |
| P2 | Beat 5: advisor. |
| P3 | Live Jev mode, only if a TypeSafe key is available. Our search suggests OpenRouter also lists Jev; unverified. |

## 8. Live Jev mode (optional, P3)

- Only the function that obtains judgments changes: it calls the System One endpoint server-side instead of reading fixtures. The API key stays server-side.
- The TypeSafe JS SDK, or a plain `fetch`, is decided at that point. If it is the SDK, it needs dependency approval.
- Same fallback as before: on error or timeout, the result falls back to rules only.

## 9. How each rubric criterion is addressed

| Criterion | Weight | How |
|---|---|---|
| Impact | 30% | Real pains in one story: a costly, slow remittance; an unbanked first-time user; the scam that targets elderly users; reporting after a loss. |
| Innovation | 20% | Watchdog asks instead of blocking. Jev judgments are combined by explicit policy, so the AI has a clear role but is never the decider. Trusted contact. Shared reported-key signal with a two-tier review. |
| Technical feasibility | 20% | Full story runs end to end. Tested rules engine. Graceful fallback when the AI is unavailable. |
| Implementation feasibility | 20% | Watchdog is a framework-free module. Mechanism built on concepts Belo already uses (local pools, digital dollars). Regulatory framing is explicit. |
| Presentation | 10% | One persona, one stage page, a visible "why it's fast" and "why it paused". |

## 10. Left to the builder

- Exact rule weights and thresholds, tuned so the scripted scenarios land on their intended levels.
- File layout, component structure, styling and palette. Visual direction: minimalist chat, warm and calm, no scare colors.
- How the backend pushes state to both phones on the stage page.
- Test runner: prefer something with no new dependency, such as Node's built-in runner.
- Defining the project's verification command and running the same command in CI. The repo has none yet.
- Deployment target. A public URL is nice to have; the public repository is required.
- Exact wording of all user-facing Spanish copy.
