# FSC Documentation

Complete specification for Filesystem Calculus v1.0.

```
docs/
├── Universe/README.md      # Dependencies and tooling
├── Scope/README.md         # Configuration space
├── Domain/README.md        # Pure capabilities
├── Mutability/README.md    # Effectful capabilities
├── Phase/README.md         # Composition
├── Artifacts/README.md     # Outputs
└── checklists/README.md    # Decision trees
```

---

## Core Philosophy

FSC is a **type-theoretic filesystem organization** based on categorical duality:

| Concept | spaces/ | bindings/ |
|---------|---------|-----------|
| Role | Interface/Schema | Implementation |
| Analogy | Free monad | Forgetful functor |
| Nature | Continuous/Abstract | Discrete/Concrete |
| Operation | Apply | Eval |
| Naming | Tool-agnostic | Vendor-specific |

**The 6 layers form a strict import hierarchy:**
```
Universe → Scope → Domain → Mutability → Phase → Artifacts
```
Each layer may only import from layers above it.

---

## Tooling Constraints

**Terminal-native, code-first, maximum control.**

### Preferences (Strict)

1. **TUI over GUI** - Terminal interfaces preferred; no full GUI applications
2. **Code-based configuration** - All config as code, no point-and-click
3. **Programmatic bindings** - Tools must have CLI/API/library bindings
4. **Lightweight rendering** - Simple renderers OK (pygame, terminal graphics), heavy GUI frameworks not
5. **Composable** - Tools must pipe/compose with other tools
6. **Inspectable** - State must be readable/debuggable

### Anti-patterns

- ❌ Full GUI applications (Audacity, Blender GUI, IDEs)
- ❌ Config via menus/dialogs
- ❌ Opaque binary state
- ❌ Tools without CLI/scripting interface

### Acceptable

- ✅ TUI applications (htop, lazygit, yazi)
- ✅ CLI tools with rich output
- ✅ Libraries with REPL/scripting (SuperCollider, pygame)
- ✅ Headless/server mode of GUI apps
- ✅ Terminal-based rendering (sixel, kitty graphics)

---

## Language Agnosticism

**FSC is language-agnostic.** Files can be in any language (`.nix`, `.org`, `.hs`, `.py`, `.v`, etc.).

**Nix composition happens only at packaging/aggregation boundaries:**
- `index.nix` files aggregate and compose
- Non-Nix files (`index.org`, `index.hs`, etc.) contain domain logic in their native language
- Nix wiring occurs at Universe (packaging) and Phase (composition) layers

**Pattern:**
```
Domain/
├── spaces/
│   └── user/
│       └── index.hs      # Haskell type definition (native language)
└── bindings/
    └── user/
        └── index.hs      # Haskell implementation (native language)

Phase/
└── bindings/
    └── index.nix         # Nix wiring that composes the Haskell modules
```

**Rationale:**
- Each language has its own type system and idioms
- Forcing everything through Nix loses expressiveness
- Nix's role is packaging and composition, not domain logic
- This keeps FSC truly universal across tech stacks

---

## Invariants

### 1. File Naming
- Only `index.*` or `flake.nix` allowed
- Extension determines language (`.nix`, `.org`, `.hs`, `.py`, `.v`, etc.)
- No other filenames (no `default.nix`, no `config.nix`, etc.)
- **Exception**: Artifacts/bindings/ may contain runtime-generated files with other names (e.g., date-based logs)

### 2. Index Role
An `index.*` file is either:
- **Aggregator**: Imports and merges subdirectories
- **Leaf node**: Contains atomic, single-concern configuration

### 3. Leaf Node Atomicity
**When a leaf `index.*` contains 2+ distinct vendors/tools, split into subdirs.**

❌ Bad:
```
plugins/index.nix  # telescope, gitsigns, cmp, orgmode...
```

✅ Good:
```
plugins/
├── index.nix           # Aggregator
├── navigation/index.nix    # telescope, oil, harpoon
├── git/index.nix           # gitsigns, fugitive
├── completion/index.nix    # cmp, luasnip
└── org/index.nix           # orgmode, headlines
```

### 4. Spaces/Bindings Duality
- Every `spaces/X/` should have corresponding `bindings/X/`
- `spaces/` = abstract interface (tool-agnostic names)
- `bindings/` = concrete implementation (vendor-specific)

### 5. No Vendor Names in Spaces
```
✅ Domain/spaces/editor/       # tool-agnostic
❌ Domain/spaces/vim/          # vendor-specific (belongs in bindings/)
```

### 6. Import Hierarchy
| Layer | May Import |
|-------|------------|
| Universe | Nothing |
| Scope | Universe |
| Domain | Universe, Scope |
| Mutability | Universe, Scope, Domain |
| Phase | Universe, Scope, Domain, Mutability |
| Artifacts | All |

---

## Layer Summary

| # | Layer | Purpose | spaces/ | bindings/ |
|---|-------|---------|---------|-----------|
| 1 | Universe | Dependencies, tooling | Package types | Concrete packages |
| 2 | Scope | Configuration schema | Option definitions | Default values |
| 3 | Domain | Pure capabilities | Type definitions | Implementations |
| 4 | Mutability | Effectful capabilities | Effect signatures | Effect handlers |
| 5 | Phase | Composition/wiring | Pipeline structure | Concrete wiring |
| 6 | Artifacts | Outputs | Output schemas | Concrete outputs |

---

## Aggregator Pattern

Parent `index.nix` merges all subdirs:
```nix
import ./navigation/index.nix
  // import ./git/index.nix
  // import ./completion/index.nix
  // import ./org/index.nix
```

---

## Module Structure

Every FSC module has:
```
module/
├── flake.nix           # Entry point, references Universe
├── .envrc              # "use flake" for direnv
├── Universe/           # Layer 1
│   ├── index.nix
│   ├── spaces/
│   └── bindings/
├── Scope/              # Layer 2
├── Domain/             # Layer 3
├── Mutability/         # Layer 4
├── Phase/              # Layer 5
└── Artifacts/          # Layer 6
```

---

## Linting

Run the FSC linter:
```bash
./Universe/bindings/checks/fsc-lint/index.sh .
```

Checks:
1. File naming (only index.*, flake.nix)
2. Canonical directories present
3. Spaces/bindings 1-1 mapping
4. Leaf node atomicity
5. No vendor names in spaces/
6. Import hierarchy violations
7. Flake structure
8. Direnv configuration
