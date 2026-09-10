---
title: "Transform Parameters"
description: "Transform parameters are passed to the XSLT transform as named parameters when a pubType is specified."
---
Transform parameters are passed to the XSLT transform as named parameters when a `pubType` is specified.

## Format

```
pram.[name]
```

Where `[name]` is the parameter name passed to the transform.

## Limitations

- Transform parameters **cannot be multi-valued** due to XSLT limitations
- Each `pram.[name]` can only have a single value

## Naming Convention

<Warning>
The `[name]` used for transform parameters you create **should start with an underscore** (e.g., `pram._myParam`).

Names without an underscore prefix:
- Will **not** be available for use through the View API
- Could collide with a system transform parameter of the same name (even if one does not currently exist, one could be added in a future version)
</Warning>

## Examples

```
# Custom transform parameter (recommended underscore prefix)
pram._outputFormat=pdf
pram._includeHistory=true

# System transform parameter (no underscore, reserved)
pram.someSystemParam=value
```
