# Mental Models — Product Requirements Document

**Status:** Draft (revised after design interview)
**Last updated:** 2026-06-25
**Owner:** Vignesh (sole user)
**Framing:** This is a **productivity intervention**, not a hobby build. Success is "I did more of my real tasks," not "I enjoyed building it." That framing is binding — see Kill Switch.

---

## Problem Statement

Ideas and tasks pile up faster than they get done. The bottleneck isn't capture — it's *starting*: after a full workday, executive dysfunction means even small tasks stall because the cost to begin is high, the list is overwhelming, and choosing what to do next is itself a tax. Existing to-do apps optimize for storing tasks, not for lowering the cost of beginning one. This app is a personal system that captures ideas/tasks in seconds, organizes them through a few reusable mental lenses (for memory and prioritization), and — the keystone — tells you *one thing you can actually do right now* given your current time and energy, or gives you permission to rest.

## Build vs. Buy (decided: build)

Most of the raw feature list is commodity. [Amazing Marvin](https://amazingmarvin.com/) is purpose-built for executive dysfunction (90+ toggleable strategies, a Procrastination Wizard, capacity feedback, task rollover); TickTick/Todoist cover capture, priority, and subtasks. Buying covers ~80% of the original list on day one.

Decision: **build anyway.** Reasons, in order: (1) unwillingness to pay a subscription; (2) a custom Lenses + LLM layer tuned to how *I* think and to my actual life-areas; (3) the build is useful .NET/PWA/LLM practice. Reasons (2) and (3) only justify the build *if it also functions as a real intervention* — hence the kill switch.

**Biggest risk, named:** building the app is more rewarding than using it, so the build can become the procrastination. Every feature must defend itself against "does this make capturing or starting faster?" If not, it's cut from v1.

## Goals

- Capture an idea or task in seconds, with only a title required.
- Turn a vague idea into concrete, startable subtasks with LLM help.
- Lower the cost of starting: always surface *one* doable next action matched to current time + energy — or explicit permission to stop.
- Organize work through a few stable mental lenses so the backlog is memorable and prioritizable, not a flat dumping ground.
- Survive real use — still opened daily three weeks in (see Success Metrics and Kill Switch).

## Non-goals

- Not multi-user, collaborative, or shared. One person, one device.
- Not a calendar, not a recurring-task engine, not deadline-enforcement (optional due dates only — see Features).
- Not cross-device. **Phone-only (Galaxy S24 Ultra), Android, portrait, v1.**
- Not a quantified-self tracker. Activity/energy logging is light, opt-in, and never a chore.
- **Not gamified.** No XP, levels, or streaks in v1. (Rationale in Constraints.)
- Not a savings / impulse-spending tool. That is a separate product, parked on the someday shelf.
- Not a rebuild of commodity to-do features beyond what the Lenses + LLM layer needs.

## User Persona

**Vignesh — the only user.** Software engineer, comfortable with code, runs LLMs via API and locally. Captures lots of ideas/tasks; consistently stalls on *initiation*, especially evenings after work — and has said outright that weekday-evening *doing* mostly doesn't happen. Success = starts and finishes things he'd otherwise leave rotting, and keeps using the app past the novelty period.

## The Mental-Models Scheme — "Lenses"

Every item is tagged along a few *orthogonal* lenses. No single axis carries all the weight; the combination makes items memorable (chunked) and prioritizable. **Capture requires only a title** — the LLM proposes lens values as a single tap-to-accept suggestion, and you correct only what's wrong. You never hand-fill multiple fields.

**The lenses:**

1. **Area** *(single-select; the memory lens, shown at capture)* — a stable life-area: Day Job, Export Biz, Finance, Home, Health, Learning, Cat, Fun. Areas are mental "rooms"; you remember an item by which room it lives in.

2. **Energy/Context** *(shown at capture)* — Brain-on / Brain-off / Social / Out-and-about. The state you need to be in. Evenings skew Brain-off; this lets Right Now stop suggesting Brain-on work when you have nothing left.

3. **Effort** *(Light / Medium / Heavy; LLM-inferred, surfaced in Right Now)* — how hard the work is once started.

4. **Duration** *(Quick <15m / Session 15–60m / Project multi-session; LLM-inferred)* — anything tagged Project must be broken into subtasks before it's actionable.

5. **Priority** *(derived, overridable)* — score from importance (you set) and time-sensitivity (from a due date if one exists; see Deadlines). LLM-suggested, always overridable.

**The activation problem — resolved by deletion, not modeling.** An earlier draft had an "Activation cost" axis (how hard to start). It was cut: activation is *contextual* (coffee is a "haul" on waking, a "flick" at 2pm), so storing it on the task is wrong half the time. Replaced by one rule that does the real work: **every actionable item must carry a defined, tiny first physical step** ("put the pod in the machine"), generated by the LLM at capture so it costs you nothing. The single biggest lever against executive dysfunction is a first move that's already written down and trivially small.

**Idea vs. Task — one structure.** Under the hood there is one entity: an **Item** with an optional parent and a "parked/someday" flag. A **Task** is a leaf (actionable, has a first step); an **Idea** is a parent/container. UI still talks about "ideas" and "tasks." An idea is never "done" by itself — it's *live* while it has open tasks and is *archived* when finished. **Only tasks with a defined first step appear in Right Now.** A **someday shelf** (built day one) holds parked ideas — never in Right Now, never nagging, reviewed only when you choose — so capture-everything doesn't rot into a guilt pile.

## Features & Requirements

**Capture — P0**
- Add an idea (heading + brief) or task in one screen; **only a title required.** *(P0)*
- LLM proposes Area, Energy, Effort, Duration, Priority, and a tiny first step as one tap-to-accept suggestion. *(P0)*
- Edit / complete / archive items; nothing is hard-deleted silently. *(P0)*
- Someday shelf for parked ideas. *(P0)*

**LLM subtask suggestion — P0**
- From an idea's heading + brief, the LLM proposes 3–7 subtasks, each with a concrete first step and suggested lens values; accept/edit/reject per subtask. *(P0)*
- Re-run / refine suggestions. *(P1)*

**Right Now (keystone) — P0**
- Input current minutes + energy; **deterministic local logic** (no network, no LLM call) filters tasks whose Duration fits and Energy matches, ranks by Priority, shows top 1–3 with each one's tiny first step. *(P0)*
- "Not now" cycles to the next candidate, no penalty. *(P0)*
- **Never returns empty.** When nothing fits: if you have a sliver of capacity, surface a pre-seeded tiny "anytime" micro-action; if you're depleted, show a **"resting is the right call"** screen with what you already did today. Permission to stop is a feature, not a cop-out. *(P0)*

**Bedtime ritual (the retention loop) — P0**
- One reliable daily trigger (provisional default: **bedtime** — revisited after the kill-test). At bedtime the app does **not** push you to do tasks (bad for sleep, low capacity); it runs **reflect + capture + queue**: a quick "what you did today," a capture dump, and **"tomorrow's one thing"** — pre-decide and pre-shrink the single first action for your higher-capacity future self. *(P0)*
- Actual *doing* happens on-demand in real doing-windows (weekend, free pockets) by opening Right Now manually. *(P0 behaviour, no new feature needed.)*

**Deadlines — P0 (thin)**
- Optional due-date field; most tasks won't have one. *(P0)*
- Priority's time-sensitivity derives from the due date when present; otherwise importance alone. *(P0)*
- A separate, always-visible **"due soon" strip.** *(P0)*
- Deadlines do **not** override energy-matching in Right Now — surfacing a Brain-on task because it's due doesn't make you able to do it when fried. *(P0)*
- No calendar sync, no recurrence, no push notifications in v1 — except optionally a single reminder for items with a real due date. *(P1)*

**Energy check-ins + routine analysis — P1**
- ~4 anchored check-in points (wake / midday / post-work / pre-sleep), **not** every 30 minutes. Each logs felt-energy plus 2–3 inputs (hours slept, coffees, one-word mood). *(P1)*
- **Voice-primary** check-in with a one-tap fallback: you talk, the LLM extracts the structured inputs + free context. Voice requires network (acceptable; not the offline-critical path). *(P1)*
- A **skip is treated as missing data, not low energy** — the next check-in asks you to label the gap ("heads-down, or running on empty?"). *(P1)*
- First weeks: the app only *describes* ("here's your energy by time of day"), makes **no predictions.** *(P1)*

**Predictive capacity + motivation — P2 (experimental, gated)**
- Only after logged data visibly shows a repeatable pattern does the LLM predict capacity and suggest how to motivate yourself. If no pattern emerges, this is never built. *(P2)*

**Platform & backup — P0**
- Installable PWA on the S24 Ultra; **capture and Right Now work offline**; only LLM features need network. *(P0)*
- **Automatic** backup to Google Drive (`appDataFolder` scope), debounced (on app close + periodic), last-write-wins single file. Drive's native file revision history provides rollback. One-time Google OAuth setup. No manual export step. *(P0)*

## Success Metrics

- **Capture latency:** < 5 seconds from app open to a saved item.
- **Sustained use (the real test):** opened and used ≥ 5 days/week for 3 consecutive weeks.
- **Initiation:** ≥ 1 previously-stalled task started per day via Right Now (in real doing-windows).
- **LLM usefulness:** ≥ 50% of suggested subtasks accepted or lightly edited.
- **Backlog health:** median age of open items stays flat or falls.

## Kill Switch (binding commitment)

If the app fails the sustained-use test (≥ 5 days/week for 3 weeks), the response is to **stop or revert** to Marvin/TickTick — **not** to add features. "It's not sticking, so I'll build streaks/gamification/a better Right Now" is the procrastination spiral in disguise and is explicitly forbidden. This commitment exists because the build is more fun than the use, and that pull has to be fenced off in advance.

## Constraints & Assumptions

- **Platform:** PWA, single codebase, phone-only. Recommended stack: lightweight JS framework (Svelte or React) + Vite PWA + local-first storage (IndexedDB via Dexie). Alternative to reuse .NET skills: Blazor WebAssembly PWA (heavier, weaker PWA ecosystem).
- **LLM access:** cloud API via a tiny serverless proxy that holds the key (Cloudflare Worker / Vercel function, free tier) — the only backend. Subtask generation uses a **cheap, fast model** (Haiku / mini class).
- **LLM cost & security:** **plain API key + hard ~$5/month provider spend cap** for the MVP; proxy protected by a shared-secret header + rate limit. The spend cap is the real safety net — it bounds the blast radius even if access control fails. The Pro plan's new Agent SDK credit pool ($20/mo, as of 2026-06-15) is a **later $0-incremental swap**, deferred because subscription-OAuth token refresh adds maintenance not worth taking on in v1.
- **Data:** lives on the phone (IndexedDB); auto-backed-up to Drive. No accounts, no server DB. Phone-only means last-write-wins backup is safe (single writer); proper sync is deferred until a second device is a real need.
- **Offline-first:** capture, edit, Right Now, and the bedtime ritual work with no network. LLM features degrade gracefully offline.
- **No gamification in v1:** XP/levels/streaks are a weak, usually temporary retention lever for executive dysfunction, and streaks risk shame-driven abandonment on the first break. Accomplishment comes from the honest "what you did today" reflection and real progress, not synthetic points. Gamification is at most a P2 experiment *after* the kill-test passes — never a crutch to rescue retention.
- **Privacy assumption:** sending tasks and (P1) voice check-ins to a cloud LLM is acceptable. If that changes, the design flips to local Ollama and the architecture changes.
- **Maintenance bias:** solo dev, scarce evening time, executive dysfunction. Fewer features, ruthless capture speed, no infra that needs babysitting.
- **Cost:** LLM ~$1–2/month at solo volume (capped at $5); Worker + PWA hosting free tier. Near-zero running cost.

## Timeline & Phases

**Phase 0 — MVP (the only phase that must ship; keep it brutally small so the build finishes before the novelty does).**
Capture (title-only) with LLM-suggested lenses + tiny first step; LLM subtask suggestion; Right Now (deterministic, offline, never-empty); someday shelf; optional due dates + due-soon strip; the bedtime reflect/capture/"tomorrow's one thing" ritual; automatic Drive backup; installable offline PWA.

**Phase 1 — Learning loop (only after MVP survives the 3-week test).**
Voice-primary energy check-ins (4 anchored points, labeled skips, describe-only); refine/re-run subtasks; optional due-date reminders; share-sheet quick capture.

**Phase 2 — Experimental (gated on evidence).**
Predictive capacity + motivation suggestions, only if the check-in data shows a real pattern. Gamification experiment, only if the kill-test has already passed.

## Open Questions

1. **Retention trigger:** bedtime is the provisional default (the moment stated with certainty). Confirm or change after the first weeks of real use — the kill-test data should inform this.
2. **Stack:** Svelte/React PWA (recommended) vs. Blazor WASM to reuse .NET skills — decide before build starts.
3. **Model choice:** which cheap model (Claude Haiku vs. an equivalent) for subtask generation.
4. **Privacy:** confirmed OK sending tasks + voice check-ins to a cloud LLM? If not, design for local Ollama up front.
