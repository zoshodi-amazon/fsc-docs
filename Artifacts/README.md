# Artifacts

**Position:** 6 of 6  
**Imports:** Universe, Scope, Domain, Mutability, Phase  
**Exports:** Outputs (schemas and concrete outputs)

The output layer. Everything the project produces.

---

## Structure

```
Artifacts/
├── spaces/           # Output schemas/templates (native language)
│   ├── logs/
│   ├── metrics/
│   ├── reports/
│   ├── builds/
│   └── ...
└── bindings/         # Concrete outputs (runtime artifacts)
    ├── logs/
    ├── metrics/
    ├── reports/
    ├── builds/
    └── ...
```

---

## Language Agnosticism

**Artifacts are language-agnostic.** Schemas/templates live in their native language:

```
Artifacts/
├── spaces/
│   ├── daily/
│   │   └── index.org     # Org-mode template (schema in native language)
│   ├── metrics/
│   │   └── index.py      # Python schema (pydantic, etc.)
│   └── builds/
│       └── index.nix     # Nix derivation schema
└── bindings/
    ├── daily/
    │   └── *.org         # Runtime artifacts (date-based files)
    ├── metrics/
    │   └── index.py      # Concrete metric outputs
    └── builds/
        └── index.nix     # Concrete derivations
```

**Nix composition happens at Phase/ or Universe/ layers**, not in Artifacts.

---

## Spaces = Output Schemas (Native Language)

Define what outputs look like in their native language:

```org
# Artifacts/spaces/daily/index.org
#+TITLE: Daily Log Template
* Daily Log: %<%Y-%m-%d>
** Goals
** Time Blocks
** Notes
```

```python
# Artifacts/spaces/metrics/index.py
from pydantic import BaseModel

class Metric(BaseModel):
    name: str
    value: float
    timestamp: str
```

```nix
# Artifacts/spaces/builds/index.nix
{
  # Build output schema
}
```

---

## Bindings = Concrete Outputs

Where outputs actually live. **Runtime artifacts may have non-index filenames:**

```
Artifacts/bindings/daily/
├── 2025-01-01.org    # Runtime artifact (date-based)
├── 2025-01-02.org
└── ...
```

```nix
# Artifacts/bindings/builds/index.nix
{ pkgs }:
pkgs.stdenv.mkDerivation {
  name = "my-app";
  src = ./.;
}
```

**Note:** The `index.*` naming rule has an exception for Artifacts/bindings/ - runtime-generated files (logs, daily notes, etc.) may use descriptive names.

---

## Flake Outputs Pattern

Artifacts map to flake outputs:

```
Artifacts/
├── spaces/
│   ├── packages/        # Package output schema
│   ├── apps/            # App output schema
│   ├── devShells/       # DevShell output schema
│   └── checks/          # Check output schema
└── bindings/
    ├── packages/        # Concrete packages
    ├── apps/            # Concrete apps
    ├── devShells/       # Concrete devShells
    └── checks/          # Concrete checks
```

---

## No Surprise Outputs

Every runtime artifact must have a schema:

- ✅ `Artifacts/spaces/logs/` exists → logs are expected
- ❌ Random log file appears → violation (no schema)

**If it's not in Artifacts/spaces/, it shouldn't exist at runtime.**

---

## Invariants

- Every output has a schema in spaces/ (native language)
- No surprise artifacts (if not in spaces/, shouldn't exist)
- 1-1 spaces ↔ bindings mapping
- Subdir name = artifact type name
- Maps to flake outputs where applicable
- **Exception**: bindings/ may contain runtime-generated files with non-index names
- Nix composition happens at Phase/ or Universe/, not here
