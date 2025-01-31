# testing

Testing gccrs in the wild

## Requirements

### ftf >= 0.3

`cargo install --git https://github.com/cohenarthur/ftf`

### submodules

Either clone the repository using the `--recursive` flag, or initialize the submodules afterwards
using the following command:

`git submodule update --init`

## Generate the parsing test-suite

The test-suite adaptor is located in `testsuite-adaptor`. It is a simple program in charge of generating a sensible test-suite for gccrs from rustc's test-suite. For now, the only generation available is the generation of a parsing test-suite: The application launches `rustc` with the `-Z parse-only` flag and keeps track of the exit code, in order to make sure that gccrs with the `-fsyntax-only` flag has the same behavior.

Running the adaptor is time consuming: It takes roughly 5 minutes to generate the parsing test-suite on my machine.

Currently, the program simply launches the `rustc` installed on your system, which is an issue.

It also absolutely hammers your computer by launching $(nproc) instances of rustc to create the test-suite baseline.

You can run the application either in debug or release mode: As it is extremely IO-intensive, it does not benefit a lot from the extra optimizations (for now!).

You can generate a testsuite using the following arguments:

### --gccrs,-g

The gccrs executable you want to test. The command will be launched similarly to how an executable is launched from the shell, so you can either specify a relative path to an executable, an absolute path, or simply the name of an executable present in your path.

One way to do this is to copy a freshly built `rust1` from your `gccrs` local copy, and pass `--gccrs './rust1'` as an argument. If you've copied your whole build directory, the argument would look something like `--gccrs './build/gcc/rust1`.

If you have `gccrs` installed on your system, you can also simply pass `--gccrs gccrs`. Be careful in that running the testsuite with a full compiler driver will obviously be much longer.

### --rustc,-r

`rustc` executable to use and test. Similar rules apply.

### --rust_path

Path to the cloned rustc repository to extract test cases from.

### --gccrs_path

Path to the cloned gccrs repository to extract test cases from.

### --output-dir,-o

Directory to create and in which to store the adapted test cases. The directory will be created by the application.

### --yaml,-y

Path of the `ftf` test-suite file to create.

### --passes,-p

List of passes to run and generate a test suite from. The currently available passes are

|Pass|Description|
|---|---|
|gccrs-parsing|Tests `gccrs`'s parser. This allows testing `gccrs` against `rustc` in parsing-mode (`-Z parse-only` and `-fsyntax-only`)|
|rustc-dejagnu|Launch `rustc` against our dejagnu testsuite. This allows validating `gccrs`'s testsuite, making sure that tests are proper rust code|

You can give multiple values to this option as it takes a vector of passes. Generating multiple test-suites at once would look like so `--passes gccrs-parsing rustc-dejagnu`.

A single YAML file is generated, even for multiple passes.

## Running the test-suite

If everything went smoothly, you should simply be able to run `ftf` on the generated YAML file:

`ftf -f <generated_yaml>`

## Typical first invocation

```sh
> # the --manifest-path argument is to run the adaptor from the root of this repository
> cargo run --manifest-path testsuite-adaptor/Cargo.toml -- \
	--gccrs './rust1' --rustc rustc \
	--gccrs-path gccrs/ --rust_path rust/ \
	--output-dir sources/ --yaml testsuite.yml \
	--passes gccrs-parsing rustc-dejagnu
> ftf -f testsuite.yml -j$(nproc)
```

Running `ftf` on a single thread (default behavior if you do not pass a `-j/--jobs` argument) is not recommended as running through the whole parsing test-suite will easily take tens of minutes.

## Re-running the test-suite with an updated compiler

If you have already generated a test-suite and would simply like to run an updated version of `gccrs`, you can reuse the same YAML file.

```sh
> cp <your-gccrs-build-dir>/gcc/rust1 ./rust1
> ftf -f testsuite.yml -j$(nproc)
```
