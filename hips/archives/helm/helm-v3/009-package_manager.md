# Appendix B: What Is a Package Manager?

If Helm is "the package manager for Kubernetes", we need to have a common
explanation of what we mean by "package manager".

Helm's notion of package management is informed by _operating system package
managers_ such as:
 - [APT](https://www.debian.org/doc/manuals/debian-reference/ch02.en.html)
 - [RPM](http://rpm.org/documentation.html)
 - [Homebrew](https://docs.brew.sh/)
 - [APK](https://wiki.alpinelinux.org/wiki/Alpine_Linux_package_management)
 - [Portage](https://wiki.gentoo.org/wiki/Portage)
 - [Snap](https://docs.snapcraft.io/core/)

It is important to not overzealously compare Helm to programming language
library package managers, as such systems have a fundamentally different goal.

Package managers such as Apt, Homebrew, and RPM provide the following services:

- They define a package format
- They define a package repository architecture
- They define a security model (signing, verification, etc)
- They specify a programming langugage (or languages) to be used for management
  logic (e.g. shell scripts, Ruby)
- They manage the installation, upgrade, and deletion of a package
- They provide dependency resolution features and version management
- They expose configuration options that the user may override
- They start services (e.g. when installing a database, the database is
  started) and often provide tooling for administrators to manage those
  services (e.g. systemd/upstart/sysV init scripts)
- They provide tools to search, list, and discover packages
- They implement (or make it easy to implement) environment-specific (os,
  processor, distro) optimizations
- They provide an opinion as to how to best construct and maintain packages
- They surface documentation (typically via output during installation)

Package managers tend to involve all of these because it is important in
package management to keep external dependencies to a minimum.

Consequently, if Helm is to fit into such a paradigm, it is natural that Helm
will provide at least most, if not all of these.

Helm is not a configuration management tool (such as Chef or Puppet). Helm does
provide certain features of such systems, though. Helm's templating and
configuration model is a common feature of configuration managers (and also of
package managers).
