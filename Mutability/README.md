# Mutability

**Position:** 4 of 6  
**Imports:** Universe, Scope, Domain  
**Exports:** Effectful capabilities (effect signatures and handlers)

The effectful capability layer. Everything that interacts with the outside world.

---

## Structure

```
Mutability/
├── spaces/           # Effect signatures
│   ├── keymaps/
│   ├── cli/
│   ├── api/
│   ├── database/
│   ├── filesystem/
│   └── ...
└── bindings/         # Effect handlers
    ├── keymaps/
    ├── cli/
    ├── api/
    ├── database/
    ├── filesystem/
    └── ...
```

---

## Common Effect Types

| Effect | spaces/ | bindings/ |
|--------|---------|-----------|
| keymaps/ | Keymap interface | Concrete key bindings |
| cli/ | CLI interface | Concrete CLI implementation |
| api/ | API endpoint signatures | Concrete API handlers |
| database/ | DB operation signatures | Concrete DB queries |
| filesystem/ | FS operation signatures | Concrete FS operations |
| http/ | HTTP client interface | Concrete HTTP calls |

---

## Spaces = Effect Signatures

Define what effects exist without implementation:

```nix
# Mutability/spaces/keymaps/index.nix
{
  # Keymap effect signature
  # backup: <leader>b → run backup
}
```

```python
# Mutability/spaces/database/index.py
from abc import ABC, abstractmethod
from Domain.spaces.user import User

class UserRepository(ABC):
    @abstractmethod
    def save(self, user: User) -> None: ...
    
    @abstractmethod
    def find(self, id: str) -> User | None: ...
```

---

## Bindings = Effect Handlers

Concrete implementations that perform effects:

```nix
# Mutability/bindings/keymaps/backup/index.nix
{
  # <leader>b → nix run .#backup
}
```

```python
# Mutability/bindings/database/index.py
import psycopg2
from Mutability.spaces.database import UserRepository
from Domain.spaces.user import User

class PostgresUserRepository(UserRepository):
    def save(self, user: User) -> None:
        # Actual DB write
        ...
```

---

## Keymaps Pattern

Keymaps connect Domain logic to user input:

```
Mutability/
├── spaces/
│   └── keymaps/
│       └── index.nix           # All keymap declarations
└── bindings/
    └── keymaps/
        ├── backup/
        │   └── index.nix       # <leader>b → backup
        ├── deploy/
        │   └── index.nix       # <leader>d → deploy
        └── index.nix           # Aggregator
```

---

## CLI Pattern

CLI interfaces for scripts:

```
Mutability/
├── spaces/
│   └── cli/
│       └── backup/
│           └── index.nix       # CLI interface for backup
└── bindings/
    └── cli/
        └── backup/
            └── index.nix       # Concrete CLI (argparse, etc.)
```

---

## Invariants

- All external interactions declared here
- Effect signatures in spaces/
- Effect handlers in bindings/
- 1-1 spaces ↔ bindings mapping
- Subdir name = effect type name
