---
title: "Property Parameters"
description: "Property parameters assign values to content properties during Load and Set Properties operations. They are ignored for other operations."
---
Property parameters assign values to content properties during [Load](/api/operations/load) and [Set Properties](/api/operations/set-properties) operations. They are ignored for other operations.

## Format

```
prop.[propertyName]
```

Where `[propertyName]` is the name of the property to set.

## Multi-Valued Properties

A property can have multiple values by including the same parameter name multiple times:

```
prop.author=Alice
prop.author=Bob
```

## Removing Properties

To remove a property value from the new Content Instance, use the `.remove` suffix:

```
prop.[propertyName].remove
```

## Examples

```
# Set a single property
prop.title=My Document

# Set multiple values for a property
prop.category=news
prop.category=featured

# Remove a property
prop.oldField.remove
```

## See Also

- [Custom Property Parameters](/api/parameters/custom-property) — for passing property sets to cascaded operations
