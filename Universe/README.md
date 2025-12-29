# Universe

**Position:** 1 of 6  
**Imports:** Nothing (root of hierarchy)  
**Exports:** Dependencies, tooling, dev environment

The meta-project layer. Everything needed to build, lint, format, test, and develop.

---

## Structure

```
Universe/
├── spaces/
│   ├── packages/       # Package declarations
│   ├── linters/        # Linter declarations
│   ├── formatters/     # Formatter declarations
│   ├── checks/         # Check/test declarations
│   ├── shells/         # Shell environment declarations
│   └── env/            # Runtime env var declarations (mirrors Scope)
└── bindings/
    ├── packages/       # Concrete packages (nixpkgs, pypi, npm)
    ├── linters/        # Concrete linter configs
    ├── formatters/     # Concrete formatter configs
    ├── checks/         # Concrete checks
    ├── shells/         # Concrete shell configs
    └── env/            # Dev overrides for partial testing
```

---

## Canonical Subdirs

| Subdir | spaces/ | bindings/ |
|--------|---------|-----------|
| packages/ | Package type declarations | Concrete packages (vendor-specific) |
| linters/ | Linter interface | Concrete linter (eslint, ruff, etc.) |
| formatters/ | Formatter interface | Concrete formatter (prettier, black) |
| checks/ | Check/test interface | Concrete checks |
| shells/ | Shell interface | Concrete shell + aliases |
| env/ | Env var schema (mirrors Scope) | Dev overrides |

---

## env/ Special Case

`Universe/spaces/env/` mirrors `Scope/spaces/` structure. This allows:
- Dev-time overrides without full module wiring
- Partial testing of individual components
- Quick iteration before committing to Scope defaults

---

## Examples

**packages/runtime/index.nix (space):**
```nix
# Package type for runtime dependencies
{ }
```

**packages/runtime/index.nix (binding):**
```nix
# Concrete runtime packages
{ pkgs }:
with pkgs; [ python311 nodejs_20 ]
```

**shells/index.nix (binding):**
```nix
# Dev shell with aliases
{ pkgs }:
pkgs.mkShell {
  packages = [ ... ];
  shellHook = ''
    alias backup="nix run .#backup"
  '';
}
```

---

## Invariants

- No business logic
- No configuration values (those go in Scope)
- Tool-agnostic naming in spaces/
- Vendor names only in bindings/
