---
name: cs2-re-analysis
description: Analyze Counter-Strike 2 / Source 2 native binaries with either IDA Pro MCP or Ghidra MCP. Use for crash analysis, static annotation, high-confidence renames/comments/types, behavior summaries, recovered structure, uncertainty tracking, CS2 schema dump assistance, and hl2sdk-cs2/hl2sdk/cstrike15_src reference-source assistance.
---

# CS2 RE Analysis

Use this skill for CS2 / Source 2 reverse engineering with either `IDA Pro MCP` or `Ghidra MCP`.

Analysis modes:

- `Crash analysis`: recover a specific crash path and identify the most likely root cause.
- `Static annotation`: improve symbols, comments, and types without requiring a crash context.

Treat the active backend, crash context, and MCP tool results as the source of truth. Use schema dumps and Source reference code only as auxiliary evidence.

## Intake

Determine backend first. If only one backend MCP is available, use it. If both IDA Pro MCP and Ghidra MCP are available, or the backend is ambiguous, ask:

```text
Which reverse-engineering backend should I use?

1. IDA Pro MCP
2. Ghidra MCP
```

Use `Crash analysis` when the user provides a crash address, exception context, call stack, minidump notes, module offset, or asks for a crash root cause.

Use `Static annotation` when the user asks to clean up, annotate, type, rename, map, or understand code without a concrete crash.

If the mode is ambiguous, ask:

```text
Which mode should I use?

1. Crash analysis: analyze a specific crash path and root cause.
2. Static annotation: improve names, comments, and types for a selected scope.
```

Then ask:

```text
Do you have local auxiliary CS2 / Source-related reference data that I can use during analysis?

Schema dump example:
D:\GithubProjects\GameTracking-CS2\DumpSource2\schemas

Reference source examples:
D:\GithubProjects\hl2sdk-cs2
D:\GithubProjects\hl2sdk
D:\GithubProjects\cstrike15_src

Report preference:
- full report.md
- concise report.md
- no report, backend annotations and renames only

If yes, please provide the available path(s).
If no auxiliary data is available, I will continue with backend-only analysis.
```

Defaults if the user explicitly says to proceed with defaults:

- Backend: use the only available backend; ask if both are available.
- Mode: infer from the request; use `Static annotation` if no crash context exists.
- Auxiliary data: none.
- Report: concise `report.md`.

## Evidence Priority

Use this priority order:

1. Active backend evidence: decompiler output, disassembly/listing, xrefs, strings, imports, exports, symbols, comments, types, data references, and crash context.
2. Matching CS2 / Source 2 schema dump data.
3. Matching `hl2sdk-cs2`, `hl2sdk`, `cstrike15_src`, or similar reference source.
4. General Source 2 / CS2 domain knowledge.

Use lower-priority evidence to guide investigation, not to override binary evidence. If schema or source conflicts with the binary, prefer the binary and record the conflict when relevant.

## Backend Adapter

Use backend-neutral reasoning, then apply changes through the active backend.

### IDA Pro MCP

- Use Hex-Rays pseudocode as the primary decompiler view.
- Use IDA disassembly when pseudocode is ambiguous or hides important behavior.
- Apply function names, local names, argument names, labels, regular comments, repeatable comments, types, structs, and enums through IDA MCP tools.
- Use IDA xrefs, strings, imports, exports, data references, function metadata, and type information to support conclusions.

### Ghidra MCP

- Use the Ghidra decompiler as the primary decompiler view.
- Use the Ghidra listing/disassembly when decompiler output is ambiguous or hides important behavior.
- Apply function names, symbols, labels, parameter names, local names, plate comments, pre-comments, EOL comments, repeatable comments, data types, structures, unions, and enums through Ghidra MCP tools.
- Use Ghidra xrefs, strings, imports, exports, data references, decompiler results, Data Type Manager concepts, and symbol tables to support conclusions.
- Use P-code, P-code graph data flow, or emulation only when it directly clarifies recovered behavior.
- Respect any Ghidra MCP naming or convention enforcement returned by the tools.

## Auxiliary Data

Use schema dumps to identify class names, field names, field offsets, inheritance, enums, flags, handles, resource types, and networked fields. Apply schema-backed names only when offsets and usage match the analyzed binary.

Use reference source to recognize naming conventions, interface names, virtual method families, service access patterns, function signatures, handles, entity concepts, resource patterns, strings, vectors, and CUtl-style containers. Apply source-derived names only when supported by binary evidence such as strings, xrefs, imports, vtable slot usage, schema data, field access patterns, or call signatures.

Inspect only the relevant schema/source files needed for the current scope.

## Shared Rules

- Inspect decompiler output first when available.
- Inspect disassembly/listing when decompiler output is ambiguous or hides important behavior.
- Add backend comments for every high-confidence finding.
- Rename all materially useful high-confidence symbols within the selected scope.
- Prioritize renames that clarify crash paths, object identity, schema-backed fields, handles, ownership, subsystem boundaries, API/interface roles, and future xrefs.
- Skip low-value temporaries or symbols whose renamed form does not improve analysis.
- Change variable, argument, return, pointer, array, struct, enum, and function-pointer types when existing types are misleading.
- Prioritize schema classes, entity pointers, handles, vtables, CUtl-style containers, arrays, resource handles, panel pointers, callbacks, jobs, and interfaces.
- Never convert number bases manually. Use the `int_convert` MCP tool for every address, RVA, VA, offset, decimal, hexadecimal, binary, or other base conversion.
- Do not claim a root cause unless supported by backend evidence, crash context, matching schema data, matching source data, or deterministic scripts that model recovered logic.
- Do not patch binaries, write hooks, or present a fix as proven unless the root cause is evidence-backed.
- Do not run broad brute force, random search, or speculative scripts.

## Rename And Comment Policy

Rename materially useful symbols when one or more are true:

- Schema class or field names match observed offsets and object usage.
- Reference source names match binary behavior, vtable usage, signatures, strings, or schema-backed types.
- Strings identify a Source 2 subsystem, class, interface, resource, event, command, or error path.
- Imports, interface calls, or API calls reveal behavior.
- Xrefs show a function is consistently used for one purpose.
- Vtable, RTTI, constructor, destructor, or schema metadata identifies an object type.
- Field offsets are repeatedly accessed in a way that matches schema data or stable usage.
- Crash context or static data flow shows a parameter or local variable directly participates in important behavior.

Do not rename just to increase rename count. A rename should make navigation, xrefs, type recovery, crash-path reasoning, or future review materially better.

Write backend comments as short, reviewable evidence statements:

- Prefer one sentence per comment line.
- Keep each line focused on one evidence-backed fact.
- Split multiple facts at the same address into separate comment lines.
- Comment pointer provenance, schema/field matches, null checks, stale-handle checks, bounds checks, virtual calls, lifetimes, refcounts, release paths, jobs, callbacks, thread-sensitive regions, safe/unsafe branches, and subsystem boundaries.

## Crash Analysis Mode

Workflow:

1. Collect crash context: module, image base, crash address, exception code, crashing instruction, registers, thread, call stack, and logs.
2. Use `int_convert` for any required VA/RVA/offset/base conversion.
3. Locate the crashing function and basic block in the active backend.
4. Inspect decompiler output near the crash.
5. Inspect disassembly/listing when needed and comment the faulting instruction.
6. Identify the Source 2 subsystem and object type using strings, xrefs, interfaces, schema data, vtables, field offsets, and call context.
7. Trace data flow backward from the crashing pointer, handle, index, field, resource, or object.
8. Trace callers and callees to recover the crash path.
9. Recover relevant types and apply schema/source-backed names only when supported.
10. Inspect boundary conditions: null checks, handle serial checks, bounds checks, resource state checks, thread ownership checks, and vtable validation.
11. Apply materially useful high-confidence renames, comments, and type corrections throughout the crash path.
12. Use small deterministic scripts only to replay recovered address math, table indexing, bit flags, schema offset matching, or decoded constants.
13. If requested, write `report.md`.

Crash report content:

- Full report: backend used, crash context, auxiliary data paths, renamed symbols, type changes, step-by-step crash path, most likely root cause, confidence, evidence versus hypotheses, helper scripts or pseudocode, unresolved questions, and validation steps.
- Concise report: backend used, crash context, most likely root cause, confidence, key evidence, important renames/comments/types, auxiliary data used, and remaining blocker if unresolved.

## Static Annotation Mode

If the user does not provide a scope, ask:

```text
What static annotation scope should I work on?

Examples:
- current function
- function at <address>
- all xrefs around <string/global/vtable>
- subsystem such as weapons, player pawn, schema system, resources, panorama, prediction, networking
- address range or function list
```

Workflow:

1. Confirm static annotation scope.
2. Survey strings, imports, xrefs, exports, vtables, RTTI/schema references, and nearby functions in scope.
3. Inspect decompiler output of each high-value function.
4. Inspect disassembly/listing where decompiler output is unclear.
5. Match field offsets and class usage against schema data when available.
6. Match interface and helper behavior against reference source only when binary behavior supports it.
7. Rename materially useful high-confidence functions, globals, vtables, locals, arguments, labels, fields, structs, and constants.
8. Apply type corrections that clarify call signatures, object pointers, arrays, handles, containers, callbacks, and return values.
9. Add concise one-sentence-per-line comments explaining evidence-backed facts.
10. Prefer comments at key decision points, object loads, field accesses, virtual calls, resource/handle resolution, constructors/destructors, and ownership boundaries.
11. Track uncertain findings as hypotheses only when useful.
12. If requested, write `report.md`.

Static annotation reports must not present a `root cause` section unless the user switches to `Crash analysis` or a concrete crash path is recovered.

Static annotation report content:

- Full report: backend used, selected scope, auxiliary data paths, renamed symbols, type changes, schema/source matches, important comments, behavior summary, recovered structure, uncertainty, and suggested next annotation targets.
- Concise report: backend used, selected scope, most important renames/types, key schema/source matches, behavior summary, recovered structure highlights, and uncertainty or remaining blockers.
- No report: rely on backend comments, names, and type changes as the deliverable.

## Final Response

Include backend, mode, schema/source usage, report preference, and either root cause confidence for Crash analysis or behavior summary / recovered structure / uncertainty for Static annotation.

