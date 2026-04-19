# full.chan

> Public module: `full.chan`
>
> Status: SPECIFIED — W22.

`full.chan` defines channels, sending, receiving, closing, and any blocking or
try-style message-passing operations.

## Example

```fuse
import full.chan.Chan;

let ch = Chan[I32].new();
spawn move fn() -> () { ch.send(10).unwrap(); };
let value = ch.recv()?;
```

## Notes

The guide should explicitly define close behavior and send/receive failure
semantics.