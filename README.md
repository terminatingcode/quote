This package collects pithy sayings.

It's part of a demonstration of

[A Tour of Versioned Go (vgo)](https://research.swtch.com/vgo-tour)

## Package versioning with Go:
See Russ Cox's presentation on [Go with Versions](https://www.youtube.com/watch?v=F8nrpe0XWRg&list=PLq2Nv-Sh8EbbIjQgDzapOFeVfv5bGOoPE&index=3&t=0s)
### Import Compatibility Rule:

_“If an old package and a new package have the **same import path**, the new package must be backwards-compatible with the old package.”_

### Semantic Import Versioning:
put the version in the import path
> Ex: import “my/thing/v2/sub/pkg”

#### Benefits:
- removes diamond dependencies
- no ambiguity as to which library behaviour to expect due to version

#### Objections:
- major versions are ugly in path
- updating import paths is annoying
  - BUT: easy to write a tool to handle handle that PLUS upcoming `go fix`
- builds should not allow both v1 and v2 (Dep disallows this)
- Too much overhead for experimenting designing packages
  - BUT v0 is a compatibility-free zone which doesn’t enforce restrictions 
- and v0 will be omitted from import paths same as v1

### Repeatability
_“The result of a build of a given version of a package should not change over time”_

Example project requirements:
```
A 1.20 needs B ≥ 1.3, C ≥ 1.8
B 1.3 needs D ≥ 1.3
C 1.8 needs D ≥ 1.4
```
vgo will not care if D 1.5 higher are released. It will **always choose D 1.4** given these constraints

I.e. [**Minimum version selection**](https://research.swtch.com/vgo-mvs)

- Objection: using the latest version is a feature! Tools should update things automatically
  - BUT why do we have lock files? Because repeatability is more important

## Go mod Cheatsheet:
```
$ go mod help
Go mod provides access to operations on modules.

Note that support for modules is built into all the go commands,
not just 'go mod'. For example, day-to-day adding, removing, upgrading,
and downgrading of dependencies should be done using 'go get'.
See 'go help modules' for an overview of module functionality.

Usage:

	go mod <command> [arguments]

The commands are:

	download    download modules to local cache
	edit        edit go.mod from tools or scripts
	graph       print module requirement graph
	init        initialize new module in current directory
	tidy        add missing and remove unused modules
	vendor      make vendored copy of dependencies
	verify      verify dependencies have expected content
	why         explain why packages or modules are needed
```

## Useful Links:
- Go 1.11 [Release Notes](https://golang.org/doc/go1.11)
- Go Package Registry: [Athens](https://github.com/gomods/athens)
- Original [vgo proposal](https://research.swtch.com/vgo)
