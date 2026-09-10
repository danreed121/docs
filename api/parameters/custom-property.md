---
title: "Custom Property Parameters"
description: "Custom property parameters pass sets of property values intended for use by later operations in the cascade tree. They are not used for the LCM write..."
---
Custom property parameters pass sets of property values intended for use by later operations in the cascade tree. They are **not** used for the LCM write operation of the current call.

## Format

```
prop[customName].[propertyName]
```

Where:
- `[customName]` is a name identifying the property set
- `[propertyName]` is the name of the property within that set

## Behavior

- These values are accessible via extension functions within transforms
- They can be passed to extension function write operations as the properties for that operation
- They follow the same multi-value and `.remove` conventions as standard [Property Parameters](/api/parameters/property)

## Removing Values

To mark values for removal within a custom property set:

```
prop[customName].[propertyName].remove
```

## Examples

```
# Pass properties for a cascaded "metadata" operation
prop[metadata].author=Alice
prop[metadata].reviewer=Bob

# Pass properties for a cascaded "publish" operation
prop[publish].channel=web
prop[publish].priority=high
```

## See Also

- [Property Parameters](/api/parameters/property) — for properties applied to the current operation
