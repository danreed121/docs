---
title: "Restriction Flags"
description: "Write Restriction Flags disallow specified kinds of content changes during an operation. By default, no changes are disallowed. If a write operation..."
---
Write Restriction Flags disallow specified kinds of content changes during an operation. By default, no changes are disallowed. If a write operation attempts a content change that a flag disallows, an error is thrown and the entire operation (including cascaded operations) is rolled back.

## Format

The `restrictionFlags` form parameter takes a string of letters. Each letter corresponds to a flag. Flags can be combined freely.

```
restrictionFlags=aD
```

The above example disallows adding a root entity at the top level and deleting any entity at the top level.

## Flag Reference

There are 4 basic change types (Add, Update, Delete, Restore) applied at 3 different scopes:

### Top-Level Root Entity

Restricts the root entity of the top-level operation only.

| Flag | Change Type |
|------|-------------|
| `a` | Add |
| `u` | Update |
| `d` | Delete |
| `r` | Restore |

### Top-Level Any Entity

Restricts any entity (root or nested) within the top-level operation.

| Flag | Change Type |
|------|-------------|
| `A` | Add |
| `U` | Update |
| `D` | Delete |
| `R` | Restore |

### Entire Change Set

Restricts all Content Entities in any operation across the entire cascade.

| Flag | Change Type |
|------|-------------|
| `Ạ` | Add |
| `Ụ` | Update |
| `Ḍ` | Delete |
| `Ṛ` | Restore |

<Info>
The "entire change set" flags use uppercase letters with a dot below (Unicode combining dot below).
</Info>

## Scope Hierarchy

The three scopes form a hierarchy from narrowest to broadest:

1. **Top-level root entity** (`a`, `u`, `d`, `r`) — Only the root entity of the direct operation
2. **Top-level any entity** (`A`, `U`, `D`, `R`) — Any entity in the direct operation, including nested
3. **Entire change set** (`Ạ`, `Ụ`, `Ḍ`, `Ṛ`) — All entities across all cascaded operations

## Examples

```
# Disallow adding or deleting at the top level (root entity only)
restrictionFlags=ad

# Disallow all updates anywhere in the cascade
restrictionFlags=Ụ

# Read-only: disallow all changes everywhere
restrictionFlags=ẠỤḌṚ

# Allow only updates to the root entity (disallow everything else)
restrictionFlags=adAUDRẠḌṚ
```
