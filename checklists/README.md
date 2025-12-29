# Checklists

Decision trees for "where does X go?"

---

## Primary Decision Tree

```
Is it...
│
├─ a dependency/package?
│  └─→ Universe/spaces/packages/
│
├─ dev tooling (lint/format/check)?
│  └─→ Universe/spaces/{linters,formatters,checks}/
│
├─ a shell/alias?
│  └─→ Universe/spaces/shells/
│
├─ an env var definition?
│  └─→ Universe/spaces/env/
│
├─ a configurable option?
│  └─→ Scope/spaces/
│
├─ a default value?
│  └─→ Scope/bindings/
│
├─ a pure type/model?
│  └─→ Domain/spaces/
│
├─ a pure function/implementation?
│  └─→ Domain/bindings/
│
├─ an external interaction (IO, network, DB)?
│  └─→ Mutability/spaces/ (signature)
│  └─→ Mutability/bindings/ (handler)
│
├─ a keymap/shortcut?
│  └─→ Mutability/spaces/keymaps/
│
├─ a CLI interface?
│  └─→ Mutability/spaces/cli/
│
├─ composition/wiring of capabilities?
│  └─→ Phase/
│
└─ an output/artifact?
   └─→ Artifacts/spaces/ (schema)
   └─→ Artifacts/bindings/ (concrete)
```

---

## Script Placement

```
Script with pure logic (no side effects):
└─→ Domain/bindings/{script-name}/index.sh

Script's CLI interface:
└─→ Mutability/spaces/cli/{script-name}/index.nix

Script's shell alias:
└─→ Universe/bindings/shells/index.nix

Script's keymap trigger:
└─→ Mutability/bindings/keymaps/{script-name}/index.nix
```

---

## Effect Classification

```
Does it...
│
├─ read/write files?
│  └─→ Mutability/spaces/filesystem/
│
├─ make HTTP requests?
│  └─→ Mutability/spaces/http/
│
├─ query/mutate a database?
│  └─→ Mutability/spaces/database/
│
├─ respond to user input?
│  └─→ Mutability/spaces/keymaps/
│
├─ expose a CLI?
│  └─→ Mutability/spaces/cli/
│
├─ expose an API?
│  └─→ Mutability/spaces/api/
│
└─ emit logs/metrics?
   └─→ Mutability/spaces/telemetry/
```

---

## Naming Checklist

Before creating a directory, verify:

- [ ] Name is tool-agnostic (in spaces/)
- [ ] Name is the type/attrset name
- [ ] Corresponding binding exists (1-1 mapping)
- [ ] Parent directory is correct canonical dir
- [ ] No magic literals in the index file

---

## Verification Questions

Run `tree` and answer:

| Question | Answer Location |
|----------|-----------------|
| What does this project do? | Domain/spaces/ |
| What are the dependencies? | Universe/bindings/packages/ |
| What can be configured? | Scope/spaces/ |
| What are the defaults? | Scope/bindings/ |
| What effects exist? | Mutability/spaces/ |
| What's the pipeline? | Phase/spaces/ |
| What outputs are produced? | Artifacts/spaces/ |
| Is wiring complete? | 1-1 spaces/ ↔ bindings/ |

---

## Anti-Patterns

| Anti-Pattern | Correct Approach |
|--------------|------------------|
| Magic number in code | Define in Scope/spaces/, use via env var |
| Tool name in spaces/ | Tool-agnostic name; tool name in bindings/ |
| Effect in Domain/ | Move to Mutability/ |
| Type definition in bindings/ | Move to spaces/ |
| Subproject inside project | Separate flake, import via Universe |
| Nested spaces/bindings | Flat structure only |
