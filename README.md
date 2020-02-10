# duplicacy-fuse

## Overview

This is my first effort at implementing a file system in userspace.

## Purpose

Allows snapshots and revisions on storage in a configured duplicacy repository to be mounted (read-only) as a file system.

## Building

Should build and run on both Windows and Linux.

On Windows, it can be built with, or without CGO support using the CGO_ENABLED env var.

## Requirements

Check out [cgofuse](https://github.com/billziss-gh/cgofuse) for full requirements, however basics are:

* Linux : gcc, libfuse-dev
* Windows (cgo) : [WinFsp](https://github.com/billziss-gh/winfsp), gcc
* Windows (!cgo)  : [WinFsp](https://github.com/billziss-gh/winfsp)

## CLI

### Building

```sh
go get -u gitlab.com/andrewheberle/duplicacy-fuse
```

Optional: To build the !cgo version on Windows, set the "CGO_ENABLED" environment variable to "0" before building.

### Usage

```sh
duplicacy-fuse <mount point> [options]
```

### Options

  `-o repositorty=<path>` : Path to repository. Defaults to current directory.
  
  `-o storage=<storage name>` : Remote storage to mount. Defaults to "default".

  `-o password=<storage password>`: Password for the specified storage.
  
  `-o snapshot=<id>` : Which snapshot id to mount. Unless `all` option is specified, defaults to the repository snapshot if for the chosen storage.
  
  `-o revision=<number>` : Which revision to mount. Cannot be specified when the `all` is specified.

  `-o cachedir=<dir>` : Where to do caching via the KV store (currently uses [bitcask](https://github.com/prologic/bitcask)) for revision contents. Defaults to `$HOME/.duplicacy-fuse` on Linux and `%USERPROFILE%\.duplicacy-fuse` on Windows.
  
  `-o all` : Display all snapshot ids for the specified storage.

## Library

```go
import (
  "github.com/billziss-gh/cgofuse/fuse"
  "gitlab.com/andrewheberle/duplicacy-fuse/dpfs"
)

func main() {
  duplicacyfs := dpfs.NewDuplicacyfs()
  host := fuse.NewFileSystemHost(duplicacyfs)
  mountpoint := "/a/path"
  options := []string{"-o", "repository=/path/to/repo"}
  host.Mount(mountpoint, options)
}
```
  
## Status

Basic functionality seems to all work fine, although this has only been tested on Windows.

Performance is quite slow when opening files.

Memory use is quite high also as the listing of all files in a snapshot is loaded into memory when initially browsing a revision. This is only used for Readdir operations, this will be fixed soon.

## TODO

* Make SFTP storage using key based authentication work
* Extend use of KV store for Readdir operations so memory use is reduced
* Tests - Implement tests for the various fuse methods (current ones for Getattr always fail)
* KV store is an Interface so potentially a better/faster option could be implemented.
* Make better :)
* Upstream?