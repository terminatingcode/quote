This package collects pithy sayings.

It's part of a demonstration of

[A Tour of Versioned Go (vgo)](https://research.swtch.com/vgo-tour)

## TL;DR:
- You can build a Go project from outside the $GOPATH or even defining it
- Minimum Version Selection algorithm increases repeatability of consistent builds with the same compatible dependency version and transient dependency versions.
- dependencies are stored in the Module Cache (`$HOME/go/src/mod`) that can be reused in other projects.

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

### To convert an existing dep project:
Convert a `Gopkg.lock` into a `go.mod`:
```
go mod init
```
Download any missing dependencies:
```
go build ./...
```
Ensure dependencies reflect:
```
go mod tidy
```

Upgrade a depdency:
- run `go get -u` to use the latest minor or patch releases
- run `go get -u=patch` to use the latest patch releases

## Build Cache:
Everytime a Go project is compiled, it will store results in the $GOCACHE directory. The $GOCACHE default location is `$HOME/.cache/go-build` where, as Russ Cox explains: "if you run this exact compiler on these exact inputs. this is the output you’d get. If the answer is not in the cache, your build uses a little more CPU to run the compiler instead of reusing the output."

## Module Cache:
Source code dependencies are now downloaded and stored in the module cache which defaults to `$GOPATH/src/mod`, defaulting to `$HOME/go/src/mod`. In addition to making source code available for reuse in a centralised location, this reduces the risk that source code, even with the same version number, has in anyway changed by avoiding downloading dependencies unless necessary. Russ Cox:
"The `go.sum` file will detect if you get a different answer on re-download, but knowing you got the wrong bits doesn’t help you make progress on actually building your code. Also these paths end up in file-line information in binaries, so they show up in stack traces, and the like and feed into tools like text editors or debuggers that don’t necessarily know how to trigger the right cache refresh."

## Useful Links:
- Go 1.11 [Release Notes](https://golang.org/doc/go1.11)
- Go Package Registry: [Athens](https://github.com/gomods/athens)
- Original [vgo proposal](https://research.swtch.com/vgo)
- Go Modules [Wiki](https://github.com/golang/go/wiki/Modules)
- [Taking Go Modules](https://dave.cheney.net/2018/07/14/taking-go-modules-for-a-spin) for a Spin by Dave Cheney
