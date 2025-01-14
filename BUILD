load("@//bazel:pkg_info.bzl", "pkg_variables", "pkg_version")
load("@bazel_skylib//rules:common_settings.bzl", "string_flag")
load("@bazel_skylib//rules:copy_file.bzl", "copy_file")
load("@rules_pkg//:mappings.bzl", "pkg_attributes", "pkg_filegroup", "pkg_files")
load("@rules_pkg//:pkg.bzl", "pkg_deb", "pkg_tar", "pkg_zip")
load("@rules_rust//rust:defs.bzl", "rust_clippy", "rustfmt_test")

package(default_visibility = ["//visibility:public"])

platform(
    name = "x86_64-apple-darwin",
    constraint_values = [
        "@platforms//cpu:x86_64",
        "@platforms//os:macos",
    ],
)

platform(
    name = "arm64-apple-darwin",
    constraint_values = [
        "@platforms//cpu:arm64",
        "@platforms//os:macos",
    ],
)

rust_clippy(
    name = "rust_clippy",
    testonly = True,
    deps = [
        "//basil_tonic",
        "//basil_tonic:libbasil_tonic_test",
        "//bazel_proto",
        "//compile_commands",
        "//compile_commands:compile_commands_test",
    ],
)

rustfmt_test(
    name = "rustfmt_test",
    targets = [
        "//basil_tonic",
        "//basil_tonic:libbasil_tonic_test",
        "//bazel_proto",
        "//compile_commands",
        "//compile_commands:compile_commands_test",
    ],
)

copy_file(
    name = "basil_tonic_bin",
    src = "//basil_tonic",
    out = "basil-tonic",
    allow_symlink = False,
    is_executable = True,
)

pkg_files(
    name = "bin_files",
    srcs = [
        ":basil_tonic_bin",
    ],
    attributes = pkg_attributes(mode = "0755"),
    prefix = "bin",
)

pkg_filegroup(
    name = "usr_files",
    srcs = [
        ":bin_files",
    ] + select({
        "@platforms//os:linux": ["//systemd"],
        "//conditions:default": [],
    }),
    prefix = select({
        "@platforms//os:macos": "/opt/basil-tonic",
        "@platforms//os:linux": "/usr",
        "//conditions:default": "",
    }),
)

pkg_filegroup(
    name = "pkg_files",
    srcs = [
        ":usr_files",
    ] + select({
        "@platforms//os:macos": ["//launchd"],
        "//conditions:default": [],
    }),
)

pkg_deb(
    name = "deb",
    architecture = select({
        "@platforms//cpu:aarch64": "arm64",
        "@platforms//cpu:x86_64": "amd64",
    }),
    built_using = "unzip (6.0.1)",
    data = ":tar",
    description = "Generates compile_commands.json file from a Bazel workspace",
    homepage = "https://github.com/kiron1/basil-tonic",
    maintainer = "Kiron <kiron1@gmail.com>",
    package = "basil-tonic",
    package_file_name = "basil-tonic_{version}_{architecture}.deb",
    package_variables = "@//:variables",
    version_file = "@//:version_file",
)

pkg_tar(
    name = "tar",
    srcs = [":pkg_files"],
)

pkg_zip(
    name = "zip",
    srcs = [":pkg_files"],
    package_file_name = "basil-tonic_{version}_{os}_{architecture}.zip",
    package_variables = "@//:variables",
)

alias(
    name = "pkg",
    actual =
        select({
            "@platforms//os:linux": ":deb",
            # Build a zip file for any other platforms:
            "//conditions:default": ":zip",
        }),
)

pkg_variables(
    name = "variables",
    architecture = select({
        "@platforms//cpu:x86_64": "amd64",
        "@platforms//cpu:aarch64": "arm64",
    }),
    os = select({
        "@platforms//os:linux": "linux",
        "@platforms//os:macos": "macos",
        "@platforms//os:windows": "windows",
        "//conditions:default": "unknown",
    }),
    version = "//:version",
)

pkg_version(
    name = "version_file",
    out = "version.txt",
    version = "//:version",
)

string_flag(
    name = "version",
    build_setting_default = "0",
    visibility = ["//visibility:public"],
)
