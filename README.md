# CS2 Static Analysis Skill

A reverse-engineering skill for analyzing Counter-Strike 2 / Source 2 native binaries in disassembler.

This skill is designed for disassembler-assisted analysis of CS2 modules, and related native binaries.

It supports two main workflows:

* **Crash analysis**: recover a specific crash path and identify the most likely root cause.
* **Static annotation**: improve a database with evidence-backed function names, variable names, comments, and type information.

## Important Requirement

This skill requires **[IDA Pro MCP](https://github.com/mrexodia/ida-pro-mcp)** or **[Ghidra MCP](https://github.com/bethington/ghidra-mcp)**.

## For Best Results

Provide these paths in your first prompt when available:

* [`cstrike15_src`](https://github.com/perilouswithadollarsign/cstrike15_src): Source 1 / CS:GO reference source for naming and behavior hints.
* [`hl2sdk-cs2`](https://github.com/Wend4r/sourcesdk): Source 2 / CS2 SDK headers, especially `public/tier0` and `public/tier1` base types.
* [`DumpSource2`](https://github.com/SteamTracking/GameTracking-CS2/tree/0804fd4cea27889a168cf1e81b1c2f3503ec18a1/DumpSource2/schemas): GameTracking schema dumps for class names, fields, offsets, enums, and networked fields.

The skill treats the disassembler database as the source of truth. Reference sources are used only when binary evidence matches.

## Example Prompt

```text
Use the CS2 RE analysis skill.

Backend: IDA Pro / Ghidra

Reference paths:
- cstrike15_src: D:\GithubProjects\cstrike15_src
- hl2sdk-cs2: D:\GithubProjects\hl2sdk-cs2
- schemas: D:\GithubProjects\GameTracking-CS2\DumpSource2\schemas

Your contexts here
```

## License

GPL-3.0
