---
name: khouwledge-judge
description: Use when you need a quick, cheap judgement about an image of a Houdini result — a viewport or render frame captured through houmcp (the Houdini MCP) — such as "did the sim blow up", "which of these frames is worse", "how damaged is the cloth". Covers the five ask_* tools of the khouwledge MCP server, which to pick, how to read their probabilities, and which failures the local vision model is measured to miss. Not for looking up Houdini facts (use khouwledge-query).
---

# khouwledge-judge — asking the local vision model about an image

Each `ask_*` tool is **one judgement** by a local vision model running on the Spark cluster. Treat it as **evidence, not a verdict.** It is cheaper than looking at the image yourself, and it pays off when you ask the same question across many frames.

## Picking a tool (one call = one question)

| You want | Tool | You get |
|---|---|---|
| yes / no | `ask_noul(instructions, images, yes_means?, no_means?)` | p(yes) and conf |
| one of several | `ask_choice(instructions, options, images)` | the chosen option, a probability per option, conf |
| a degree on an ordered scale | `ask_score(instructions, levels, images)` | a continuous score (expected level index) and a probability per level |
| every option that applies | `ask_select(instructions, options, images, threshold?)` | p(yes) per option; they do not sum to 1 |
| A or B | `ask_compare(instructions, images_a, images_b)` | the winner and conf |

- Three questions about the same image means **three calls.** The images sit at the front of the prompt, so the server's prefix cache encodes them once (about 0.3 s per call after the first).
- At most 20 options or levels. In `ask_choice` the order is what the model sees (the first option is A).
- Give `ask_score` levels **low to high.** Keep the threshold in your own reasoning.
- Say what yes and no mean in `yes_means` / `no_means`, one clause each; it keeps the judgement steady.

## Sending images
- **base64 strings only.** File paths are not accepted (the server is remote). A `data:image/png;base64,` prefix is fine; otherwise set `mime` (default `image/jpeg`).
- **Send small images.** A 480×270 frame is about 122 tokens, and doubling the resolution made judgements **worse.** At most 4 images per call, 16 MiB each.
- EXR is refused. Have houmcp write JPG or PNG.
- Put in `context` what the shot is supposed to show, frame numbers and units.

## Reading the answer
- Probabilities come only with `mode="logprobs"` (the default). `mode="json"` returns the value alone.
- `conf` is how concentrated the probability is on one option (0 = uniform, 1 = certain). Read p near 0.5, or a low conf, as **"don't know".**
- Even at temperature 0, about 8% of identical requests flip. Do not decide a borderline case on one call.
- `ask_compare` has an order bias. On a close call use `swap_and_average=true` (twice the cost).
- For hard cases use `observe_first=true`: the model writes down what it sees before judging. It costs one extra request and adds an `observation:` line — read it to find out **what the model failed to see.**

## What to trust (measured)
- **Failures that change the silhouette are caught**: an object leaving the frame (0.94), a ball falling through the ground (0.91).
- **Failures that change only shape or texture are mostly missed**: torn or exploded cloth (0.44 at best, 0.14 with default settings). The problem is perception, not judgement — the model described exploded cloth as "mostly flat".
- So **"no problem" is weak evidence.** If you suspect a mesh or texture failure, measure it through houmcp (bounding box, point count, max velocity) or look at the image yourself.
- These tools do **not describe** images. Writing prose is your job.

## When it fails
- A connection or backend error usually means the vision model (the vLLM planner) on the Spark is down — it is taken down during re-indexing. Tell the user and look at the image yourself instead.
- If a judgement is plainly wrong, record it with `report_error(kind="result", …)`. The table above only improves as those cases pile up.
