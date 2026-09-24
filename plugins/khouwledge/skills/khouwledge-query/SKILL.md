---
name: khouwledge-query
description: Use whenever a question touches Houdini (nodes, parameters, VEX/HScript/OpenCL functions, HOM Python, simulation guides) and the khouwledge MCP server is connected. Says when to call search vs get_*, how to read the ranked results, what the store does not know (verified parameter names, notes), and why an empty answer is not a failure. For judging images use khouwledge-judge instead.
---

# khouwledge-query — looking things up in the Houdini docs index

## When
For any Houdini question, **call `search` before guessing.** Never trust memory for VEX signatures, node parameters or HOM members; confirm them every time with `get_function` / `get_node` / `get_python_class`.

## Order
1. `search(query)` finds candidates. Identifiers (`pcfind`, `hou.OpNodeType.deprecated`, `tube.radius`) come first, marked `(exact)`.
2. Confirm with the tool for that kind (one tool = one facet):
   - `get_node(id, folder?)` — parameter list. Over 40 parameters it returns counts per folder; call again with `folder`. The root folder is `folder="(root)"`.
   - `get_function(name, language?)` — signatures. `language` is `vex` / `hscript` / `python` / `opencl`; pass it when the same name exists in more than one language.
   - `get_python_class(name, member?, group?)` — pass `member` when you want one member.
   - `get_doc_page(id, section?, part?)` — one section of prose. Node, function and class ids work too.
   - `get_units(target)` knowledge units, `get_notes(target)` notes, `list_domains(context)` the domain table, `status()`.
   - `report_error(kind, asked, actual, …)` — see "When it is wrong" below.
3. **The last line of every answer counts the other facets** (`sections: … · params: 30`). If you need more, ask for that facet by name with another tool or argument. Do not ask for a bigger answer.

## Reading results
- Ranking is deliberately tilted **core nodes > guide pages > Labs / third party**. For a conceptual question the right page may sit at rank 3–7. Read all ten lines, not just the first.
- `[crowds/excluded]` and `[…/low]` mean a low-priority domain, not a wrong answer.
- Some entities come **from the installed Houdini, not from the docs** (VEX header functions, Python modules outside HOM, OpenCL kernels). They may have a signature and no prose. When the origin is a **package** (KineFX, APEX, …) it is not part of the base install and may be missing on the user's machine — check the package is there before relying on it.

## What it does not know (do not bluff)
- **`#name` in `get_node` is a doc anchor slug. Do not put it in a script.** It matches the real parameter name on some nodes (`#itermethod` on `sop/block_end`) and not on more of them (`sop/tube`: `#radius` is really `rad`, `#center` is `t`, `#u-order` is `orderu`). A hyphenated name (`#u-order`) is certainly not a real name. Some parameters exist only in the docs (`#u-wrap` on tube). **Only a name in parentheses, `(rad)`, is verified**, and very few are so far (see the legend line `names: … N verified`). For real names, types and defaults, ask the Houdini MCP (houmcp) for `parmTemplates()`.
- `get_units` holds only a handful of units and `get_notes` always returns 0. **An empty answer means nothing is recorded yet, not an error.**
- This server never runs Houdini. Cooking and scene edits belong to houmcp.

## When it is wrong — do not just move on
If an answer from khouwledge turns out to be wrong (especially after trying it in Houdini), **record it with `report_error`.** `kind` is `knowledge` (a fact is wrong), `search` (irrelevant results or nothing found) or `result` (followed the answer and it did not work). Only `asked` and `actual` are required, but always include the `entity_ids` that gave the wrong answer. Reports land as unreviewed drafts on the server and a person promotes them to knowledge units. The index itself is not changed.
