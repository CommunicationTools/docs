# Workspace keys

Workspaces are plain text (format 2). The grammar is a header, then `[Channel]`
blocks, each followed by its `[Rtu]` blocks and their point sub-blocks.
Containment is by order; unknown keys are ignored; a block only needs the keys
that differ from the defaults.

```
[Workspace] Format=2

[Channel]
Tool=DNP3
Name=Line 1
Running=1
# endpoint / link / TLS keys ...

  [Rtu]
  Proto=DNP3
  Addr=10
  Name=OS10
  # station keys ...

    [Points]
    # one point per line: group|index|name|...|side|tail
```

Key points:

- **`Format=2`** is required; older formats are not read.
- Masters save **`Running=`**, so a master workspace restarts its channels on
  load.
- Point rows are **side-neutral** — the same row works in a master or a slave
  channel, with `|M` / `|S` marking whose tail follows — so an RTU can be moved
  between a master and a slave.
- Connection keys shared by every tool: `AutoReconnect`, `ReconnectMin`,
  `ReconnectMax`, `RtsMode`, `RtsDelay`, plus the TLS keys.

!!! note "Draft"
    A full per-tool key table will be added here. For now the file itself is the
    best reference — open a saved workspace in a text editor to see every key in
    context.
