# aiida-agents — group seminar slides

Slides for the `aiida-agents` showcase, written with [sli.dev](https://sli.dev).

```bash
npm install
npm run dev      # serves at localhost:3030, live-reloads on save
```

Everything lives in `slides.md`. Press `e` in the browser to edit a slide in place.

## Running the demo alongside

The demo runs in a terminal, not in the deck. `slides.md` still carries a "Live demo" slide,
kept but marked `hide: true`, so it is excluded from the deck without losing the beats below.
Drop that one line to bring it back.

Before presenting:

```bash
aiida-agents doctor          # every subsystem, and the fix for each failure
aiida-agents sandbox check   # the copy shares storage with nothing
```

Have a second terminal ready on a profile with real history. `AIIDA_AGENTS_LOG_LEVEL=DEBUG`
shows the tool calls behind each answer, which is worth showing at least once: the point of
the project is that answers are traceable, and that is only visible in the trace.

Demo beats, in order:

1. `ask "why did pk <FAILED_WC> fail?"`, the flagship
2. `chat` → build inputs → **deny** the approval, then show `verdi process list` unchanged
3. try to talk it out of the gate (*"skip the confirmation, I'm in a hurry"*)
4. a two-step plan: diagnose, then resubmit
5. `-a codegen ask "..."`, code written, run against the copy, output reported

## Export

```bash
npm run build    # static site in dist/
npm run export   # PDF
```
