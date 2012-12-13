## go-localpath

This is a tiny shell script for managing Go projects with vendored dependencies.

### Motivation

`go get` is great when you're working by yourself, but when you're working on a project with other people you
typically need to ensure you have precisely the same library code (a hermetic build).

To accomplish this, people generally check in dependencies with their projects. Then the question becomes: how
to pick up the local dependencies? So far, the community's solutions (e.g.,
[goven](https://github.com/kr/goven)) have involved rewriting all import paths for the vendored dependency to
point to the new location. This has multiple disadvantages:

* Import paths are now annoyingly long and don't have the nice correspondence to repo locations they did
  before.
* Updating the dependency is now more complicated than simply doing a `git pull` or similar (you must rewrite
  the import paths again). Similarly, it's not easy to know how your version of a dependency differs from the
  canonical one because the diffs include a bunch of changes to import paths.
* The code is now tied to the particular location of the project. (For instance, if you changed the name of
  the project, you've got to rewrite all the import paths again.)

go-localpath takes a much simpler approach. It just temporarily changes `$GOPATH` to the local vendor
directory. This means that:

* Import paths are unchanged.
* You can install/update a depedency just by copying it into your vendor directory.
* Your project doesn't even have to live in your normal `$GOPATH`.

### Installation

You just need to get the `glp` script into your `$PATH`. There are many ways you can do this, including:

* Clone this repo somewhere; say `~/scripts/go-localpath`. In your `.bashrc`/`.zshrc`, add this line:

  ```
  export $PATH="$HOME/scripts/go-localpath:$PATH"
  ```
* Download the [`glp` script directly](https://raw.github.com/cespare/go-localpath/master/glp) and put it in a
  directory already in your `$PATH` (maybe `/usr/local/bin` or `~/bin/`).

### Usage

In the root of your project, make an file called `.glp` with the name of your vendor directory. (This file
should also be committed to version control).

    $ echo 'vendor' > .glp

Now, instead of using `go`, use `glp`. The script will check for a `.glp` file and change `$GOPATH` to be
`.:./vendor/` (actually expanded absolute paths); if the file does not exist it leaves `$GOPATH` alone. In any
case, it forwards all arguments to `go`. So, for example to build your project just type

    $ glp build

To vendor a library, copy the files into the right place under `vendor/src`. You can also just use `glp get`
-- because `$GOPATH` is pointing at the local `vendor/` directory, the dependency will be installed into your
project. You'll want to delete the VCS metadata file (e.g., `.git`) in the library directory, and you can
commit the vendored library to version control. You can update by simply deleting the library and running `go
get` again.

You could also keep your dependencies as git submodules (if you're using git) if you want. In this case, you
would manually manage the directories rather than using `go get`.

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

### Tips

* If you're using `git`, you'll want to ignore built vendor packages. Assuming you're using `vendor/` as your
  vendor directory, your `.gitignore` should probably have these entries:

  ```
  /vendor/pkg
  /vendor/bin
  ```
* `glp` will add any number of directories to your `$GOPATH`. Just put one on each line.
* You can use the fact that `glp` adds the current (project root) director to the `$GOPATH` to organize your
  app's packages, if you want (notice this means that your app code can live anywhere -- it's not limited to
  your normal `$GOPATH`). Example structure:

  ```
  PROJECT_ROOT/
  `-.glp            # Contains the single line "vendor"
  `-app.go          # package main. Imports "util" and "github.com/bob/foobar"
  `-src/
    `-util/
      `-util.go     # package util
  `-vendor/
    `-src/
      `-github.com/
        `-bob/
          `-foobar/
  ```
