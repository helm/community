# Package Manager UX

This document is for usability research on the UX and interaction interface for other package managers.

(This re-creates some of the work done during the Helm 1 and Helm 2 planning cycles.)

## A Goal of Helm: Follow Existing Patterns

To reduce the cognitive overhead of Helm, one of Helm's UX goals is to follow the patterns set down by other package managers.

Matching the UX of `kubectl` is of secondary importance.

In practice, this plays out as follows:

- When considering Helm's package managemet operations, Helm _should_ follow the patterns and practices of other package management tools.
- When working with Kubernetes-specific concepts (such as namespaces, pods, tunnels, etc.), Helm _should_ follow the patterns and practices of `kubectl` or other popular Kubernetes tools.
- In more general circumstances, Helm _should_ follow the _de facto_ patterns found in contemporary Linux/UNIX tooling.

Helm _should not_ follow the Plan9/Go patterns that are not broadly implemented in UNIX/Linux (e.g. `-long`). That Helm is implemented in Go does not _ipso facto_ mean that Helm must follow the opinions of a very small group of developers over the opinions of the vastly larger UNIX 
community.

## Package Managers and Their Characteristics

The following table represents a summary of command line tooling for popular package managers. Package managers considered came in one of two types: _OS_ types install application/tool packages onto operating systems. _Lang_ types install language-specific packages (lirbaries, tools) into appropriate environments.

| Tool     | Type | Cmd | Install | Upgrade        | Delete       | Create | Repo Update | Search           | About Pkg      |
| ----     | ---- | ------ | ------- | -------        | ------       | ------ | ----------- | ------           | ---------      |
| apt-get  | OS   | y      | install | upgrade        | remove       | -      | update      | apt-cache search | apt-cache show |
| yum      | OS   | y      | install | update/upgrade | remove/erase | -      | -           | search           | info           |
| brew     | OS   | y      | install | upgrade        | uninstall    | create | update      | search           | info           |
| pacman   | OS   | n      | --sync  | --upgrade      | --remove     | -      | -           | --query          | --query        |
| emerge   | OS   | n      | NONE    | NONE           | --unmerge    | -      | --sync      | --search         | --search       |
| choco    | OS   | y      | install | upgrade        | uninstall    | new    | ~update~    | search           | info           |
| npm      | Lang | y      | install | update/upgrade | uninstall/rm | init/create | -      | search           | view/info/show |
| pod      | Lang | y      | install | update         | -            | spec create | -      | search           | search         |
| pip      | Lang | y      | install | install -U     | uninstall    | -      | -           | search           | show           |
| composer | Lang | y      | install | update         | remove       | init   | -           | search           | show           |
| kubectl  | -    | y      | create  | apply          | delete       | create | -           | -                | -              |
| helm 2   | CN   | y      | install | upgrade        | delete       | create | update      | search           | inspect        |

Note that `kubectl` is included for reference, though it is _not_ a package manager, and is not weighed into the results below.

Other package managers looked at, but not deemed popular enough to be considered influencers: Mix (erlang), Cargo (Rust), Dep (Go), Glide (Go), GoFish (OS), CPAN (perl), Yarn (JavaScript), APK (Alpine Linux). Other than CPAN, we saw no major divergence from the table above (though we saw a few cases where `add` was used instead of `install`).

### Apt

- Apt uses multiple tools. `apt-get` and `apt-cache` are the most frequently used.
- Apt does not layer subcomments. There are only direct subcommands and flags
- Create new packages with other tools

### Yum

- Yum layers subcommands, but without a consistent pattern:
    - `yum clean headers` (NOUN VERB NOUN)
    - `yum version groupinfo` (NOUN NOUN NOUN)
    - `yum groups install` (NOUN NOUN VERB)
- Yum syncs with remote repositories based on local configuration
- Create new packages with RPM commands

### Brew

No notes

### Pacman

Arch Linux uses `pacman`, which does not support subcommands.

- `pacman` updates from remote repos as needed.
- Search is multi-modal, being able to describe packages as well as listing them

### Emerge (Portage)

- Create new packages with `ebuild`
- `emerge PKG` handles install and upgrade. In other words, installation is the default action

### choco (Chocolatey)

- The `update` command is deprecated, and future version will sync differently

### NPM

- Fun fact: `npm isntall` is an alias for `npm install`, as is `npm add` and `npm i`.
- Update, upgrade, and up are aliases
- Uninstall, remove, rm, and un are aliases
- view, info, and show are aliases

### pod (cocoapods)

- Supports subcommands, usually as NOUN NOUN VERB (`pod spec create`), but occasionally NOUN NOUN NOUN (`pod trunk info`)
- I cannot find docs on how to remove a pod

### pip

- A few commands have subcommands in the form NOUN NOUN VERB (`pip config get`)

### Composer

Composer is the PHP package manager.

- `composer` has only one level of commands, but uses flags for subcommands (`composer config --list`).
- The exception to the above, though, is the `compoers global` command, which is NOUN NOUN VERB or NOUN NOUN NOUN.

### kubectl

`kubectl` is not a package manager, but has subcommands that are similar to package managers

- `create` combines generation and installation
- Upgrading can be done several different ways, each with a separate verb: `apply`, `edit`, `patch`, `convert`, and `replace`
- There is no standard pattern to `kubectl`'s subcommands:
    - `kubectl get` (NOUN VERB)
    - `kubectl logs` (NOUN NOUN)
    - `kubectl config view` (NOUN NOUN VERB)
    - `kubectl config current-context` (NOUN NOUN NOUN)
- NOTE: It is claimed that `kubectl` uses NOUN VERB NOUN (`kubectl get pod foo`), but this is a misunderstanding: the kind argument is not a command component. It's an argument that depends on what kinds are installed.

## Analysis of Results

Commonalities between package managers:

- All of the tools except for `emerge` made install/upgrade/delete top level commands. (emerge is simpler; install is the default if no command is specified).
- Many of these tools also had subcommands as well (e.g. `pod`, `composer`, `yum`), but did not choose to fold all commands into a particular pattern.
    - For subcommands, the `NOUN NOUN VERB` pattern was the most common, but definitely not the only option.
    - The only discernable pattern used for seperating top level and multi-level commands is: "popular operations are top level"
    - Not all the commands on top-level dealt with the same concepts (e.g. one command may operate on packages, while another operates on files).
- Aliases are supported in a few tools
- `upgrade` is general preferred over `update` for describing the process of installing a newer version of an existing package
- For package managers that support synchronizing a local cache with a remote auhtority, `update` is the preferred term, in spite of its ambiguity.
- For the package managers that support scaffolding out new packages, `create` is the most widely used term.
- `show` and `info` are about equal in representation for showing package information. `inspect`, which Helm uses, is not represented in any other package manager.
- For deleting a package, `remove` is the most common term, followed closely by `uninstall`.
- None of the examined systems had more than three layers of commands: `CMD SUBCMD SUBCMD`.

## Recommendations for Helm 3

The following commands should be changed (with the old name aliased to the new name):

- `helm delete` should be renamed `helm remove`
- `helm inspect` should be renamed `helm show`

While there is little to draw from, it might be prudent to rename `helm fetch` to `helm download`.

After analyzing other package managers, it does not appear that grouping commands into "logical subgroups" is an appropriate action. No other tools that we could find do this. Subgroup usage is _ad hoc_. The convention Helm 2 used (subgroups for less popular commands) _appears_ to be the convention used industry-wide.

No other changes are necessary for Helm to fit in with the common idioms found in package managers.

_Note:_ The Helm 2 `reset` command is no longer present in Helm 3. Earlier discussions about renaming it are no longer relevant.