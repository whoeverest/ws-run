# Workspace script runner

Run npm scripts in a yarn workspace.

### Usage:

```
wsrun [options] <command> [<arg1> <arg2> ...]

Mode (choose one):
  --parallel  Fully parallel mode (default)                                               [boolean]
  --stages    Run in stages: start with packages that have no deps                        [boolean]
  --serial    Same as "stages" but with no parallelism at the stage level                 [boolean]

Package Options:
  --recursive, -r  Execute the same script on all of its dependencies, too                [boolean]

Misc Options:
  --fast-exit        If at least one script exits with code > 0, abort                    [boolean]
  --collect-logs     Collect per-package output and print it at the end of each script    [boolean]
  --no-prefix        Don't prefix output                                                  [boolean]
  --bin              The program to pass the command to                  [string] [default: "yarn"]
  --done-criteria    Consider a process "done" when an output line matches the specified RegExp
  --exclude          Skip running the command for that package                             [string]
  --exclude-missing  Skip packages which lack the specified command in the scripts section of their
                     package.json                                                         [boolean]
  --report           Show an execution report once the command has finished in each package
                                                                                          [boolean]

Other Options:
  --help          Show help                                                               [boolean]
  --version       Show version number                                                     [boolean]
  --package, -p   Run only for these packages                                               [array]
  --if            Run main command only if this condition command runs successfully
  --ifDependency  Run main command only if packages dependencies passed the condition (not
                  available in parallel mode)                                             [boolean]
```

### Examples:

`yarn wsrun watch` will run `yarn watch` on every individual package, in parallel.

`yarn wsrun --stages build` will build all packages, in stages, starting from those that don't depend on other packages.

`yarn wsrun -p planc -r watch` will watch planc and all of its dependencies.

`yarn wsrun -p planc --exclude planc -r watch` will watch all of planc's dependencies but not planc

`yarn wsrun -p h4zip -r --stages build` will build all the deps. in order, then build h4zip

`yarn wsrun -p planc --stages --done-criteria='Compilation complete' -r watch` will watch planc deps, in order, continuing when command outputs "Compilation complete"

`yarn wsrun --exclude-missing test` will run the test script only on packages that have it

To specify multiple packages, use `-p` several times:

`yarn wsrun -p h4zip -p planc test` - run tests for both h4zip and planc

If you want to pass additional arguments to the command you can do that by adding them after the
command:

`yarn wsrun -r --stages build -p tsconfig.alternative.json` - build all packages in stages with
and pass an alternative tsconfig to the build script

When `--skip-missing` is not used, you can pass a command that doesn't exist in the scripts field:

`yarn wsrun -r --stages tsc -p tsconfig.alternative.json` - run tsc for all packages with an alternative tsconfig

#### Conditional execution

Conditional execution is supported with `--if` and `--if-dependency`

Examples

`yarn wsrun --stages --if build-needed build` - for each package it will first try `yarn wsrun build-needed` and only if the exit code is zero (success) it will run `yarn wsrun build`

`yarn wsrun --stages --if-build-needed --if-dependency build` - it will run `build` for each package in stages, if either the package's own condition command was success, or any of the dependencies had a successful condition.
