# Scope

**Position:** 2 of 6  
**Imports:** Universe  
**Exports:** Configuration schema and defaults

The configuration space. Like Nix module options — defines what can be configured and provides defaults.

---

## Structure

```
Scope/
├── spaces/           # Options schema (what can be configured)
│   ├── database/
│   ├── server/
│   ├── auth/
│   └── ...
└── bindings/         # Defaults (concrete default values)
    ├── database/
    ├── server/
    ├── auth/
    └── ...
```

---

## Spaces = Options Schema

Each `spaces/` subdir defines configurable options:

```nix
# Scope/spaces/database/index.nix
{ lib, ... }:
{
  options.database = {
    host = lib.mkOption { type = lib.types.str; };
    port = lib.mkOption { type = lib.types.port; };
    pool_size = lib.mkOption { type = lib.types.int; };
  };
}
```

---

## Bindings = Defaults

Each `bindings/` subdir provides default values:

```nix
# Scope/bindings/database/index.nix
{
  database = {
    host = "localhost";
    port = 5432;
    pool_size = 10;
  };
}
```

---

## 1-1 Mapping

Every `spaces/x/` must have a corresponding `bindings/x/`:

```
Scope/spaces/database/   ↔   Scope/bindings/database/
Scope/spaces/server/     ↔   Scope/bindings/server/
Scope/spaces/auth/       ↔   Scope/bindings/auth/
```

---

## Secrets Pattern

Secrets are handled via file paths, not values:

```nix
# Scope/spaces/auth/index.nix
{
  options.auth = {
    secret_key_file = lib.mkOption { type = lib.types.path; };
  };
}

# Scope/bindings/auth/index.nix
{
  auth = {
    secret_key_file = "/run/secrets/auth_key";
  };
}
```

The actual secret is never in the repo — only the path to it.

---

## Runtime Access

Other languages access Scope via env vars:

```python
# Domain or Mutability code
import os
db_host = os.getenv("DATABASE_HOST")
db_port = int(os.getenv("DATABASE_PORT"))
```

The bridge: `Universe/spaces/env/` generates these env vars from Scope.

---

## Invariants

- No magic literals in downstream code
- All configurable values defined here
- 1-1 spaces ↔ bindings mapping
- Secrets via file paths, not inline values
- **Paths are configuration** - file paths, URLs, etc. belong in Scope, not hardcoded in Mutability
