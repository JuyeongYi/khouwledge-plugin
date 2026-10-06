---
name: khouwledge-query
description: Use whenever a question touches Houdini (nodes, parameters, VEX/HScript/OpenCL functions, HOM Python, simulation guides) and the khouwledge MCP server is connected. Says when to call search vs the query_* tools, how to read the ranked results, what the store does not know (verified parameter names, notes), and why an empty answer is not a failure. For judging images use khouwledge-judge instead.
---

# khouwledge-query — looking things up in the Houdini docs index

## When
For any Houdini question, **call `search` before guessing.** Never trust memory for VEX signatures, node parameters or HOM members; confirm them every time with `query_node` or the `query_*` tool for the language.

## Order
1. `search(query)` finds candidates. Identifiers (`pcfind`, `hou.OpNodeType.deprecated`, `tube.radius`) come first, marked `(exact)`.
2. Confirm with the tool for that kind (one tool = one facet):
   - `query_node(id, folder?)` — parameter list. Over 40 parameters it returns counts per folder; call again with `folder`. The root folder is `folder="(root)"`.
   - One lookup per language, so languages never mix: `query_vex(name)`, `query_hscript(name)`, `query_opencl(name)`, `query_python(name, member?, group?)`, `query_apexscript(name, member?, group?)`. **If you do not know the language, use `search`.** A bare member name works (`query_python("createNode")`); if several classes share it you get `Class.member` lines and the footer says how many were hidden. `query_vex` also shows the `#include` header. APEX Script verbs carry their version in the name (`apex.sop.skeletonblend.v3_0`).
   - `query_doc_page(id, section?, part?, full?)` — one section of prose. Node, function and class ids work too.
   - `query_units(target, kind?)` knowledge units, `query_notes(target)` notes, `list_domains(context)` the domain table, `status()`.
   - `report_error(kind, asked, actual, …)` — see "When it is wrong" below.
3. **The last line of every answer counts the other facets** (`sections: … · params: 30`). If you need more, ask for that facet by name with another tool or argument. Do not ask for a bigger answer.

## Reading results
- Ranking is deliberately tilted **core nodes > guide pages > Labs / third party**. For a conceptual question the right page may sit at rank 3–7. Read all ten lines, not just the first.
- `[crowds/excluded]` and `[…/low]` mean a low-priority domain, not a wrong answer.
- Some entities come **from the installed Houdini, not from the docs** (VEX header functions, Python modules outside HOM, OpenCL kernels). They may have a signature and no prose. When the origin is a **package** (KineFX, APEX, …) it is not part of the base install and may be missing on the user's machine — check the package is there before relying on it.

## What it does not know (do not bluff)
- **`#name` in `query_node` is a doc anchor slug. Do not put it in a script.** It matches the real parameter name on some nodes (`#itermethod` on `sop/block_end`) and not on more of them (`sop/tube`: `#radius` is really `rad`, `#center` is `t`, `#u-order` is `orderu`). A hyphenated name (`#u-order`) is certainly not a real name. Some parameters exist only in the docs (`#u-wrap` on tube). **Only a name in parentheses, `(rad)`, is verified**, and very few are so far (see the legend line `names: … N verified`). For real names, types and defaults, ask the Houdini MCP (houmcp) for `parmTemplates()`.
- `query_units` holds only a handful of units and `query_notes` always returns 0. **An empty answer means nothing is recorded yet, not an error.**
- This server never runs Houdini. Cooking and scene edits belong to houmcp.

## When it is wrong — do not just move on
If an answer from khouwledge turns out to be wrong (especially after trying it in Houdini), **record it with `report_error`.** `kind` is `knowledge` (a fact is wrong), `search` (irrelevant results or nothing found) or `result` (followed the answer and it did not work). Only `asked` and `actual` are required, but always include the `entity_ids` that gave the wrong answer. Reports land as unreviewed drafts on the server and a person promotes them to knowledge units. The index itself is not changed.
