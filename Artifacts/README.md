# Artifacts

**Position:** 6 of 6  
**Imports:** Universe, Scope, Domain, Mutability, Phase  
**Exports:** Outputs (schemas and concrete outputs)

The output layer. Everything the project produces.

---

## Structure

```
Artifacts/
├── spaces/           # Output schemas/contracts
│   ├── logs/
│   ├── metrics/
│   ├── reports/
│   ├── builds/
│   └── ...
└── bindings/         # Concrete outputs
    ├── logs/
    ├── metrics/
    ├── reports/
    ├── builds/
    └── ...
```

---

## Spaces = Output Schemas

Define what outputs look like (contracts):

```nix
# Artifacts/spaces/logs/index.nix
{
  # Log output schema
  # format: JSON
  # fields: timestamp, level, message, context
}
```

```python
# Artifacts/spaces/metrics/index.py
from pydantic import BaseModel

class Metric(BaseModel):
    name: str
    value: float
    timestamp: str
    labels: dict[str, str]
```

---

## Bindings = Concrete Outputs

Where outputs actually go:

```nix
# Artifacts/bindings/logs/index.nix
{
  path = "/var/log/app/";
  rotation = "daily";
}
```

```nix
# Artifacts/bindings/builds/index.nix
{ pkgs }:
pkgs.stdenv.mkDerivation {
  name = "my-app";
  src = ./.;
  # ...
}
```

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

- Every output has a schema in spaces/
- No surprise artifacts
- 1-1 spaces ↔ bindings mapping
- Subdir name = artifact type name
- Maps to flake outputs where applicable
