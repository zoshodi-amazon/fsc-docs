# Domain

**Position:** 3 of 6  
**Imports:** Universe, Scope  
**Exports:** Pure capabilities (types and implementations)

The pure capability layer. Type definitions and their implementations with no side effects.

---

## Structure

```
Domain/
├── spaces/           # Type definitions
│   ├── user/
│   ├── order/
│   ├── product/
│   └── ...
└── bindings/         # Implementations
    ├── user/
    ├── order/
    ├── product/
    └── ...
```

---

## Spaces = Type Definitions

Language-appropriate type definitions:

**Nix:**
```nix
# Domain/spaces/user/index.nix
{
  # User type schema
}
```

**Python (pydantic):**
```python
# Domain/spaces/user/index.py
from pydantic import BaseModel

class User(BaseModel):
    id: str
    email: str
    name: str
```

**TypeScript:**
```typescript
// Domain/spaces/user/index.ts
interface User {
  id: string;
  email: string;
  name: string;
}
```

---

## Bindings = Implementations

Concrete implementations of the types:

```python
# Domain/bindings/user/index.py
from Domain.spaces.user import User

def create_user(email: str, name: str) -> User:
    return User(id=generate_id(), email=email, name=name)

def validate_user(user: User) -> bool:
    return "@" in user.email
```

---

## Pure Scripts

Scripts with pure logic live here:

```
Domain/
├── spaces/
│   └── backup/
│       └── index.nix    # Backup type/interface
└── bindings/
    └── backup/
        └── index.sh     # Pure backup logic (no side effects)
```

The script itself is pure — it takes input and produces output. Side effects (actually writing files) happen in Mutability.

---

## 1-1 Mapping

```
Domain/spaces/user/    ↔   Domain/bindings/user/
Domain/spaces/order/   ↔   Domain/bindings/order/
Domain/spaces/backup/  ↔   Domain/bindings/backup/
```

---

## Invariants

- No side effects (no IO, no network, no filesystem writes)
- No magic literals (config comes from Scope via env vars)
- Pure functions only
- Types in spaces/, implementations in bindings/
- Subdir name = type name
