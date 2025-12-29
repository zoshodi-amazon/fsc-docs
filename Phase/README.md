# Phase

**Position:** 5 of 6  
**Imports:** Universe, Scope, Domain, Mutability  
**Exports:** Composed pipelines (traced monoidal category)

The composition layer. How capabilities wire together into pipelines.

---

## Structure

```
Phase/
├── spaces/           # Pipeline/DAG structure
│   ├── ingest/
│   ├── transform/
│   ├── validate/
│   ├── persist/
│   └── ...
└── bindings/         # Concrete wiring
    ├── ingest/
    ├── transform/
    ├── validate/
    ├── persist/
    └── ...
```

---

## Traced Monoidal Category

Phase encodes composition as a traced monoidal category:
- Objects = types from Domain
- Morphisms = transformations
- Trace = feedback loops / recursion

```
ingest → transform → validate → persist
  A         B           C          D
```

---

## Spaces = Pipeline Structure

Define the DAG without concrete implementations:

```nix
# Phase/spaces/index.nix
{
  # Pipeline: ingest → transform → validate → persist
  stages = [ "ingest" "transform" "validate" "persist" ];
}
```

```
Phase/spaces/
├── ingest/
│   └── index.nix      # Input stage type
├── transform/
│   └── index.nix      # Transform stage type
├── validate/
│   └── index.nix      # Validation stage type
└── persist/
    └── index.nix      # Output stage type
```

---

## Bindings = Concrete Wiring

Wire Domain types and Mutability effects together:

```nix
# Phase/bindings/ingest/index.nix
{ domain, mutability }:
{
  # Wire: api.receive → domain.user.parse
  input = mutability.api.receive;
  output = domain.user.parse;
}
```

```python
# Phase/bindings/index.py
from Domain.bindings.user import create_user, validate_user
from Mutability.bindings.database import PostgresUserRepository
from Mutability.bindings.api import receive_request

def pipeline(request):
    data = receive_request(request)           # Mutability
    user = create_user(data.email, data.name) # Domain
    if validate_user(user):                   # Domain
        repo = PostgresUserRepository()
        repo.save(user)                       # Mutability
    return user
```

---

## Phase Ordering

Directory order = execution order:

```
Phase/spaces/
├── 01-ingest/
├── 02-transform/
├── 03-validate/
└── 04-persist/
```

Or use explicit DAG in index file.

---

## Invariants

- No new types (use Domain)
- No new effects (use Mutability)
- Only composition/wiring
- 1-1 spaces ↔ bindings mapping
- Subdir name = phase/stage name
