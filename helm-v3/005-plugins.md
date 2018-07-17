# Plugins

The plugin model will undergo a few changes in order to make it possible to
support a broader range of features along a broader range of platforms.

## Standard Plugins

The existing plugin model (based on `exec` to local binaries) will continue to
be supported.

The current YAML format for `plugin.yaml` will be extended as follows:

```
name: "last"
version: "0.1.0"
usage: "get the last release name"
description: "get the last release name"
command: "$HELM_BIN --host $TILLER_HOST list --short --max 1 --date -r"
# New part:
platformCommand:
  - os: linux
    arch: i386
    command: "$HELM_BIN list --short --max 1 --date -r"
  - os: windows
    arch: amd64
    command: "$HELM_BIN list --short --max 1 --date -r"
```

The `platformCommand` section defines OS/Architecture specific variations of a
command.

The following rules will apply to processing commands:

- If platformCommand is present, it will be searched first.
- If both OS and Arch match the current platform, search will stop and the 
  command will be executed
- If OS matches and there is no more specific match, the command will be executed
- If no OS/Arch match is found, the default `command` will be executed
- If no `command` is present and no matches are found in `platformCommand`, Helm
  will exit with an error.

## Lua Plugins

The existing Helm plugin mechanism has enabled numerous extensions to Helm
(e.g., [here](https://docs.helm.sh/related/#helm-plugins) and
[here](https://github.com/search?q=topic%3Ahelm-plugin&type=Repositories)).
While this has been useful there have been two shortcomings:

1. Many plugins are written as shell scripts assuming a POSIX environment and
   familiar linux-style userland. Helm, and Kubernetes as a whole, supports
   windows where these plugins don't work natively.
2. Other plugins are written in a compiled language (e.g., Go) or a general
   scripting language (e.g., Python) that have system dependencies in order to
   work. These can have version complications as well (e.g., a system Python of
   3 and a plugin Python need of 2.7).

Along with Helm supporting Lua for scripting chart extensions, it will support
plugins in Lua. These plugins will only have a dependency on Helm to run and
can be ported cross platform.

Lua plugins _are executed separately_ from chart-based Lua scripts. The pattern
used for executing Lua plugins mirrors the pattern for writing command-line
applications. A `main()` function will be executed with access to STDIN, STDOUT,
STDERR, and all command-line parameters and options.

### Lua Plugin Sandboxing

Like charts, each plugin will be executed in an isolated Lua sandbox. Unlike
chart scripts, plugins will be granted access to all core libraries, and will
not require security prompts to accept.
