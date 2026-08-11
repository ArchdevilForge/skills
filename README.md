# ArchdevilForge Skills

18 AI agent skills, managed in `~/.agents/skills/`, shared across agents via symlinks.

## Skills

**Mindset**
- `hustler-mind` — 14 master traders (Justin Sun, GCR, Arthur Hayes, Soros, Livermore) in one: scenario router + risk redlines

**Reversing**
- `js-reverse` — JS signature reversing
- `apk-reverse` — APK unpack / decompile / Frida
- `protocol-re` — protocol RE: capture, dissect, parse
- `reverse-core` — binary RE methodology (Go/Rust/Ghidra)
- `chain-trace` — multi-chain token forensics (public APIs)

**Dev**
- `prompt-loop` — prompt & agent loop engineering
- `find-skill` — skill discovery
- `mcp-builder` — MCP server guide

**Docs**
- `docx` `pdf` `xlsx` `diagram` — documents & diagrams

**Design**
- `design-taste` — anti-slop frontend design
- `ui-ux` — 50 styles / 21 palettes / 50 font pairs

**Content & Thinking**
- `content` — copywriting + de-AI rewrite
- `thinking` — 8 thinking frameworks
- `uv` — Python uv

## Install

```bash
cp -r <skill> ~/.agents/skills/
# or symlink
ln -s $(pwd)/<skill> ~/.agents/skills/<skill>
```

## Notes

- Sanitized: no keys, paths, or private data
- KISS: skeleton SKILL.md + on-demand `references/`
