![afero logo-sm](https://cloud.githubusercontent.com/assets/173412/11490338/d50e16dc-97a5-11e5-8b12-019a300d0fcb.png)

A FileSystem Abstraction System for Go

[![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/spf13/afero/ci.yaml?style=flat-square)](https://github.com/spf13/afero/actions/workflows/ci.yaml)
[![go.dev reference](https://img.shields.io/badge/go.dev-reference-007d9c?logo=go&logoColor=white&style=flat-square)](https://pkg.go.dev/mod/github.com/spf13/afero)
![GitHub go.mod Go version](https://img.shields.io/github/go-mod/go-version/spf13/afero?style=flat-square&color=61CFDD)
[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/spf13/afero/badge?style=flat-square)](https://deps.dev/go/github.com%252Fspf13%252Fafero)

# Overview

Afero is a filesystem framework providing a simple, uniform and universal API
interacting with any filesystem, as an abstraction layer providing interfaces,
types and methods. Afero has an exceptionally clean interface and simple design
without needless constructors or initialization methods.

Afero is also a library providing a base set of interoperable backend
filesystems that make it easy to work with, while retaining all the power
and benefit of the os and ioutil packages.

Afero provides significant improvements over using the os package alone, most
notably the ability to create mock and testing filesystems without relying on the disk.

It is suitable for use in any situation where you would consider using the OS
package as it provides an additional abstraction that makes it easy to use a
memory backed file system during testing. It also adds support for the http
filesystem for full interoperability.


## Afero Features

* A single consistent API for accessing a variety of filesystems
* Interoperation between a variety of file system types
* A set of interfaces to encourage and enforce interoperability between backends
* An atomic cross platform memory backed file system
* Support for compositional (union) file systems by combining multiple file systems acting as one
* Specialized backends which modify existing filesystems (Read Only, Regexp filtered)
* A set of utility functions ported from io, ioutil & hugo to be afero aware
* Wrapper for go 1.16 filesystem abstraction `io/fs.FS`

# Using Afero

Afero is easy to use and easier to adopt.

A few different ways you could use Afero:

* Use the interfaces alone to define your own file system.
* Wrapper for the OS packages.
* Define different filesystems for different parts of your application.
* Use Afero for mock filesystems while testing

## Step 1: Install Afero

First use go get to install the latest version of the library.

    $ go get github.com/spf13/afero

Next include Afero in your application.
```go
import "github.com/spf13/afero"
```

## Step 2: Declare a backend

First define a package variable and set it to a pointer to a filesystem.
```go
var AppFs = afero.NewMemMapFs()

or

var AppFs = afero.NewOsFs()
```
It is important to note that if you repeat the composite literal you
will be using a completely new and isolated filesystem. In the case of
OsFs it will still use the same underlying filesystem but will reduce
the ability to drop in other filesystems as desired.

## Step 3: Use it like you would the OS package

Throughout your application use any function and method like you normally
would.

So if my application before had:
```go
os.Open("/tmp/foo")
```
We would replace it with:
```go
AppFs.Open("/tmp/foo")
```

`AppFs` being the variable we defined above.


## List of all available functions

File System Methods Available:
```go
Chmod(name string, mode os.FileMode) : error
Chown(name string, uid, gid int) : error
Chtimes(name string, atime time.Time, mtime time.Time) : error
Create(name string) : File, error
Mkdir(name string, perm os.FileMode) : error
MkdirAll(path string, perm os.FileMode) : error
Name() : string
Open(name string) : File, error
OpenFile(name string, flag int, perm os.FileMode) : File, error
Remove(name string) : error
RemoveAll(path string) : error
Rename(oldname, newname string) : error
Stat(name string) : os.FileInfo, error
```
File Interfaces and Methods Available:
```go
io.Closer
io.Reader
io.ReaderAt
io.Seeker
io.Writer
io.WriterAt

Name() : string
Readdir(count int) : []os.FileInfo, error
Readdirnames(n int) : []string, error
Stat() : os.FileInfo, error
Sync() : error
Truncate(size int64) : error
WriteString(s string) : ret int, err error
```
In some applications it may make sense to define a new package that
simply exports the file system variable for easy access from anywhere.

## Using Afero's utility functions

Afero provides a set of functions to make it easier to use the underlying file systems.
These functions have been primarily ported from io & ioutil with some developed for Hugo.

The afero utilities support all afero compatible backends.

The list of utilities includes:

```go
DirExists(path string) (bool, error)
Exists(path string) (bool, error)
FileContainsBytes(filename string, subslice []byte) (bool, error)
GetTempDir(subPath string) string
IsDir(path string) (bool, error)
IsEmpty(path string) (bool, error)
ReadDir(dirname string) ([]os.FileInfo, error)
ReadFile(filename string) ([]byte, error)
SafeWriteReader(path string, r io.Reader) (err error)
TempDir(dir, prefix string) (name string, err error)
TempFile(dir, prefix string) (f File, err error)
Walk(root string, walkFn filepath.WalkFunc) error
WriteFile(filename string, data []byte, perm os.FileMode) error
WriteReader(path string, r io.Reader) (err error)
```
For a complete list see [Afero's GoDoc](https://godoc.org/github.com/spf13/afero)

They are available under two different approaches to use. You can either call
them directly where the first parameter of each function will be the file
system, or you can declare a new `Afero`, a custom type used to bind these
functions as methods to a given filesystem.

### Calling utilities directly

```go
fs := new(afero.MemMapFs)
f, err := afero.TempFile(fs,"", "ioutil-test")

```

### Calling via Afero

```go
fs := afero.NewMemMapFs()
afs := &afero.Afero{Fs: fs}
f, err := afs.TempFile("", "ioutil-test")
```

## Using Afero for Testing

There is a large benefit to using a mock filesystem for testing. It has a
completely blank state every time it is initialized and can be easily
reproducible regardless of OS. You could create files to your heart’s content
and the file access would be fast while also saving you from all the annoying
issues with deleting temporary files, Windows file locking, etc. The MemMapFs
backend is perfect for testing.

* Much faster than performing I/O operations on disk
* Avoid security issues and permissions
* Far more control. 'rm -rf /' with confidence
* Test setup is far more easier to do
* No test cleanup needed

One way to accomplish this is to define a variable as mentioned above.
In your application this will be set to afero.NewOsFs() during testing you
can set it to afero.NewMemMapFs().

It wouldn't be uncommon to have each test initialize a blank slate memory
backend. To do this I would define my `appFS = afero.NewOsFs()` somewhere
appropriate in my application code. This approach ensures that Tests are order
independent, with no test relying on the state left by an earlier test.

Then in my tests I would initialize a new MemMapFs for each test:
```go
func TestExist(t *testing.T) {
	appFS := afero.NewMemMapFs()
	// create test files and directories
	appFS.MkdirAll("src/a", 0755)
	afero.WriteFile(appFS, "src/a/b", []byte("file b"), 0644)
	afero.WriteFile(appFS, "src/c", []byte("file c"), 0644)
	name := "src/c"
	_, err := appFS.Stat(name)
	if os.IsNotExist(err) {
		t.Errorf("file \"%s\" does not exist.\n", name)
	}
}
```

# Available Backends

## Operating System Native

### OsFs

The first is simply a wrapper around the native OS calls. This makes it
very easy to use as all of the calls are the same as the existing OS
calls. It also makes it trivial to have your code use the OS during
operation and a mock filesystem during testing or as needed.

```go
appfs := afero.NewOsFs()
appfs.MkdirAll("src/a", 0755)
```

## Memory Backed Storage

### MemMapFs

Afero also provides a fully atomic memory backed filesystem perfect for use in
mocking and to speed up unnecessary disk io when persistence isn’t
necessary. It is fully concurrent and will work within go routines
safely.

```go
mm := afero.NewMemMapFs()
mm.MkdirAll("src/a", 0755)
```

#### InMemoryFile

As part of MemMapFs, Afero also provides an atomic, fully concurrent memory
backed file implementation. This can be used in other memory backed file
systems with ease. Plans are to add a radix tree memory stored file
system using InMemoryFile.

## Network Interfaces

### SftpFs

Afero has experimental support for secure file transfer protocol (sftp). Which can
be used to perform file operations over a encrypted channel.

### GCSFs

Afero has experimental support for Google Cloud Storage (GCS). You can either set the
`GOOGLE_APPLICATION_CREDENTIALS_JSON` env variable to your JSON credentials or use `opts` in
`NewGcsFS` to configure access to your GCS bucket.

Some known limitations of the existing implementation:
* No Chmod support - The GCS ACL could probably be mapped to *nix style permissions but that would add another level of complexity and is ignored in this version.
* No Chtimes support - Could be simulated with attributes (gcs a/m-times are set implicitly) but that's is left for another version.
* Not thread safe - Also assumes all file operations are done through the same instance of the GcsFs. File operations between different GcsFs instances are not guaranteed to be consistent.


## Filtering Backends

### BasePathFs

The BasePathFs restricts all operations to a given path within an Fs.
The given file name to the operations on this Fs will be prepended with
the base path before calling the source Fs.

```go
bp := afero.NewBasePathFs(afero.NewOsFs(), "/base/path")
```

### ReadOnlyFs

A thin wrapper around the source Fs providing a read only view.

```go
fs := afero.NewReadOnlyFs(afero.NewOsFs())
_, err := fs.Create("/file.txt")
// err = syscall.EPERM
```

# RegexpFs

A filtered view on file names, any file NOT matching
the passed regexp will be treated as non-existing.
Files not matching the regexp provided will not be created.
Directories are not filtered.

```go
fs := afero.NewRegexpFs(afero.NewMemMapFs(), regexp.MustCompile(`\.txt$`))
_, err := fs.Create("/file.html")
// err = syscall.ENOENT
```

### HttpFs

Afero provides an http compatible backend which can wrap any of the existing
backends.

The Http package requires a slightly specific version of Open which
returns an http.File type.

Afero provides an httpFs file system which satisfies this requirement.
Any Afero FileSystem can be used as an httpFs.

```go
httpFs := afero.NewHttpFs(<ExistingFS>)
fileserver := http.FileServer(httpFs.Dir(<PATH>))
http.Handle("/", fileserver)
```

## Composite Backends

Afero provides the ability have two filesystems (or more) act as a single
file system.

### CacheOnReadFs

The CacheOnReadFs will lazily make copies of any accessed files from the base
layer into the overlay. Subsequent reads will be pulled from the overlay
directly permitting the request is within the cache duration of when it was
created in the overlay.

If the base filesystem is writeable, any changes to files will be
done first to the base, then to the overlay layer. Write calls to open file
handles like `Write()` or `Truncate()` to the overlay first.

To writing files to the overlay only, you can use the overlay Fs directly (not
via the union Fs).

Cache files in the layer for the given time.Duration, a cache duration of 0
means "forever" meaning the file will not be re-requested from the base ever.

A read-only base will make the overlay also read-only but still copy files
from the base to the overlay when they're not present (or outdated) in the
caching layer.

```go
base := afero.NewOsFs()
layer := afero.NewMemMapFs()
ufs := afero.NewCacheOnReadFs(base, layer, 100 * time.Second)
```

### CopyOnWriteFs()

The CopyOnWriteFs is a read only base file system with a potentially
writeable layer on top.

Read operations will first look in the overlay and if not found there, will
serve the file from the base.

Changes to the file system will only be made in the overlay.

Any attempt to modify a file found only in the base will copy the file to the
overlay layer before modification (including opening a file with a writable
handle).

Removing and Renaming files present only in the base layer is not currently
permitted. If a file is present in the base layer and the overlay, only the
overlay will be removed/renamed.

```go
	base := afero.NewOsFs()
	roBase := afero.NewReadOnlyFs(base)
	ufs := afero.NewCopyOnWriteFs(roBase, afero.NewMemMapFs())

	fh, _ = ufs.Create("/home/test/file2.txt")
	fh.WriteString("This is a test")
	fh.Close()
```

In this example all write operations will only occur in memory (MemMapFs)
leaving the base filesystem (OsFs) untouched.


## Desired/possible backends

The following is a short list of possible backends we hope someone will
implement:

* SSH
* S3

## Third-party library

- Alibaba Cloud OSS: [messikiller/afero-oss](https://github.com/messikiller/afero-oss)

# About the project

## What's in the name

Afero comes from the latin roots Ad-Facere.

**"Ad"** is a prefix meaning "to".

**"Facere"** is a form of the root "faciō" making "make or do".

The literal meaning of afero is "to make" or "to do" which seems very fitting
for a library that allows one to make files and directories and do things with them.

The English word that shares the same roots as Afero is "affair". Affair shares
the same concept but as a noun it means "something that is made or done" or "an
object of a particular type".

It's also nice that unlike some of my other libraries (hugo, cobra, viper) it
Googles very well.

## Release Notes

See the [Releases Page](https://github.com/spf13/afero/releases).

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

## Releasing

As of version 1.14.0, Afero moved implementations with third-party libraries to
their own submodules.

Releasing a new version now requires a few steps:

```
VERSION=X.Y.Z
git tag -a v$VERSION -m "Release $VERSION"
git push origin v$VERSION

cd gcsfs
go get github.com/spf13/afero@v$VERSION
go mod tidy
git commit -am "Update afero to v$VERSION"
git tag -a gcsfs/v$VERSION -m "Release gcsfs $VERSION"
git push origin gcsfs/v$VERSION
cd ..

cd sftpfs
go get github.com/spf13/afero@v$VERSION
go mod tidy
git commit -am "Update afero to v$VERSION"
git tag -a sftpfs/v$VERSION -m "Release sftpfs $VERSION"
git push origin sftpfs/v$VERSION
cd ..

git push
```

TODO: move these instructions to a Makefile or something

## Contributors

Names in no particular order:

* [spf13](https://github.com/spf13)
* [jaqx0r](https://github.com/jaqx0r)
* [mbertschler](https://github.com/mbertschler)
* [xor-gate](https://github.com/xor-gate)

## License

Afero is released under the Apache 2.0 license. See
[LICENSE.txt](https://github.com/spf13/afero/blob/master/LICENSE.txt)
