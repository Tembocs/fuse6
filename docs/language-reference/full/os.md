# full.os

> Public module: `full.os`
>
> Status: SPECIFIED — W22.

`full.os` covers operating-system identity, platform-level handles, and process
termination/status conventions.

## Example

```fuse
import full.os as os;

let family = os.family();
if family == "windows" {
    os.exit(1);
}
```

## Notes

Platform-specific behavior must be explicit rather than hidden behind vague
cross-platform wording.