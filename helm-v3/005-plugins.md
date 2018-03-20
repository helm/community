# Plugins

The plugin model will undergo a few changes in order to make it possible to
support a broader range of features along a broader range of platforms.

## Lua Plugins

The existing Helm plugin mechanism has enabled numerous extensions to Helm
(e.g., [here](https://docs.helm.sh/related/#helm-plugins) and
[here](https://github.com/search?q=topic%3Ahelm-plugin&type=Repositories)).
While this has been useful there have been two shortcomings:

1. Many plugines are written as shell scripts assuming a POSIX environment and
   familiar linux-style userland. Helm, and Kubernetes as a whole, supports
   windows where these plugins don't work natively.
2. Other plugins are written in a compiled languages (e.g., Go) or a general
   scripting language (e.g., Python) that have system dependencies in order to
   work. These can have version complications as well (e.g., a system Python of
   3 and a plugin Python need of 2.7).

Along with Helm supporting Lua for scripting chart extensions, it will support
plugins in Lua. These plugins will only have a dependency on Helm to run and
can be ported cross platform.
> [name=Matt Butcher] The main danger here is that we have to make it so that
> charts cannot have implicit reliance on Lua plugins. I think that is fairly
> straightforward, though. We just have to make sure that it's not possible to
> execute plugin code from within a chart.


### Lua Plugin Sandboxing

Like charts, each plugin will be executed in an isolated Lua sandbox. Unlike
chart scripts, plugins will be granted access to all core libraries, and will
not require security prompts to accept.
