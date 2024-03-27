# Basil tonic - Fresh `compile_commands.json` for your Bazel workspace

![bazel-compile-commands build status of main branch](https://github.com/kiron1/basil-tonic/actions/workflows/main.yaml/badge.svg)

Automatically generates `compile_command.json` files for an Bazel workspace
without modifications. This is the equivalent of _CMAKE_EXPORT_COMPILE_COMMANDS_
from CMake for Bazel.

Basil Tonic is a service which can listen to
[Bazel Build Events](https://bazel.build/remote/bep) and will write a
`compile_commands.json` to the Bazel workspace root after compilation finished.

Therefor `basil-tonic` is best suited for IDE use cases where the
`compile_commands.json` file is automatically updated with each build.

The
[Microsoft C/C++ Extension for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)
struggels sometimes with Bazel workspaces since not all include paths are at the
default locations. The `compile_commands.json` file can be used with `clangd` to
have a significant better LSP experience. For
[Visual Studio Code (VS Code)][vscode], the
[clangd extension][llvm-vs-code-extensions.vscode-clangd] can be used to utilize
the project compile information via the `compile_commands.json` file.

## Installation

- Download the latest Debian package from
  https://github.com/kiron1/basil-tonic/releases
- Install via:
  ```sh
  sudo dpkg --install bazel-compile-commands_*_amd64.deb
  ```
- Enable service:
  ```sh
  systemctl --user enable --now basil-tonic.socket
  ```

## Configure Bazel

Ensure when building a C/C++ workspace, that Bazel will send build events to
Basil Tonic.

Either add the relevant arguments when calling `bazel build ...` or add them to
one of the `.bazelrc` [files loaded by Bazel during startup][bazelrc].

For example in the `$HOME/.bazelrc` file:

```
build:basil-tonic --bes_backend=grpc://127.0.0.1:50151
build:basil-tonic --build_event_publish_all_actions
build:basil-tonic --nobes_lifecycle_events
build:basil-tonic --bes_timeout=2s

# The line below ensures that basil-tonic is used by default.
# Uncomment the line and use `bazel build --config=basil-tonic`
# to only use it on selected build invocations.
build --config=basil-tonic
```

## Configure VSCode

for [Visual Studio Code (VS Code)][vscode], use the
[clangd extension][llvm-vs-code-extensions.vscode-clangd]. `clangd` will
automatically pick up the `compile_commands.json` from the workspace root.

[vscode]: https://code.visualstudio.com/ "Visual Studio Code"
[llvm-vs-code-extensions.vscode-clangd]: https://marketplace.visualstudio.com/items?itemName=llvm-vs-code-extensions.vscode-clangd "clangd - C/C++ completion, navigation, and insights"
[bazelrc]: https://bazel.build/run/bazelrc "Write bazelrc configuration files"

## Usage workflow

When building a Bazel workspace, Bazel will send build events to Basil Tonic.
Basil Tonic will use the information from the build events protocol to build the
build commands database and extend the existing `compile_commands.json` file if
exists. The final `compile_commands.json` file will be written, when the build
is finished.

<!-- deno-fmt-ignore-start -->

> **Note**
> You might need to restart `clangd` when you started it _before_ any
> `compile_commands.json` existed.

> **Note**
> Bazel will only send build events for files it is compiling. If you buid your
> workspace already before using Basil Tonic Bazel will use the cached results
> and not compile the C/C++ files. Either run `bazel clean` to force a re-build
> of the workspace or use `bazel-copmpile-commands` to generate an initial
> `compile_commands.json` file.

<!-- deno-fmt-ignore-end -->

## Alternative tools

- [`bazel-compile-commands`](https://github.com/kiron1/bazel-compile-commands) -
  Sibling project of `basil-tonic`. Generate `compile_commands.json` by
  analysing the Bazel action graph.
- [Bear](https://github.com/rizsotto/Bear) - Can work when used with the
  [`--spawn_strategy=local`](https://docs.bazel.build/versions/main/user-manual.html#flag--spawn_strategy)
  Bazel flag.
- [hedronvision/bazel-compile-commands-extractor](https://github.com/hedronvision/bazel-compile-commands-extractor) -
  Can be integrated into Bazel files.
- [grailbio/bazel-compilation-database](https://github.com/grailbio/bazel-compilation-database) -
  Needs integration into your Bazel files

## Links

- [Format of the `compile_commands.json` format](https://clang.llvm.org/docs/JSONCompilationDatabase.html)
- [clangd](https://clangd.llvm.org/)
- [analysis_v2.proto](https://github.com/bazelbuild/bazel/blob/master/src/main/protobuf/analysis_v2.proto)
- [Build Event Protocol](https://bazel.build/remote/bep)

## License

This source code is under the [MIT](https://opensource.org/licenses/MIT) license
with the exceptions mentioned in "Third party source code in this repository".
