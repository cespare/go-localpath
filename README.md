## go-localpath

WIP

This is a tiny shell script for managing Go projects with vendored dependencies.

### Motivation

`go get` is great when you're working by yourself, but when you're working on a project with other people you
typically need to ensure you have precisely the same library code (a hermetic build).

To accomplish this, people generally check in dependencies with their projects. Then the question becomes: how
to pick up the local dependencies? So for, the community's solutions (e.g.,
[goven](https://github.com/kr/goven)) have involved rewriting all import paths for the vendored dependency to
point to the new location. This has multiple disadvantages:

* Import paths are now annoyingly long and don't have the nice correspondence to repo names they did before.
* Updating the dependency is now more complicated than simply doing a `git pull` or similar (you must rewrite
  the import paths again). Similarly, it's not easy to know how your version of a dependency differs from the
  canonical one because the diffs include a bunch of changes to import paths.
* The code is now tied to the particular location of the project. (For instance, if you changed the name of
  the project, you've got to rewrite all the import paths again.

go-localpath takes a much simpler approach. It simply temporarily changes `$GOPATH` to the local `vendor/`
directory. This means that:

* Installing/updating a vendored library is as simple as `go get [-u]`.

### Installation

You just need to get the `glp` script into your `$PATH`. There are many ways you can do this, including:

* Clone this repo somewhere; say `~/scripts/go-localpath`. In your `.bashrc`/`.zshrc`, add this line:

  ```
  export $PATH="$HOME/scripts/go-localpath:$PATH"
  ```
* Download the [`glp` script directly](https://raw.github.com/cespare/go-localpath/master/glp) and put it in a
  directory already in your `$PATH` (maybe `/usr/local/bin` or `~/bin/`).

### Usage

In the root of your project, make an empty file called `.glp`. (This file should also be committed to
version control).

    $ touch .glp

Now, instead of using `go`, use `glp`. The script will check for a `.glp` file and change `$GOPATH` to
`./vendor/`; if the file does not exist it leaves `$GOPATH` alone. In any case, it forwards all arguments to
`go`. So, for example to build your project just type

    $ glp build

To vendor a library, just use `glp get` -- because `$GOPATH` is pointing at the local `vendor/` directory, the
dependency will be installed into your project. Then you can commit the vendored library to version control.

### Replacing the go command

You can also make a function to just run `glp` every time you run `go` so you never have to think about
running `glp`. This shouldn't change your non-glp use of `go` at all (unless you happen to have files called
`.glp` laying around). In your `.bashrc`/`.zshrc`:

``` sh
PATH=/path/to/go-localpath:$PATH # As before
go() {
  command go $@
}
```

`glp` doesn't do this for you by default because it's a little magical.

### TODO (maybe)

* Allow for different dir names to be used instead of hard-coding `vendor/`.
* The `.glp` file can contain any necessary configuration.
