package(default_visibility = ["//visibility:public"])

load(
    "//tools/jdk:default_java_toolchain.bzl",
    "default_java_toolchain",
    "java_runtime_files",
    "DEFAULT_JAVACOPTS",
)

# Used to distinguish toolchains used for Java development, ie the JavaToolchainProvider.
toolchain_type(name = "toolchain_type")

# Used to distinguish toolchains used for Java execution, ie the JavaRuntimeInfo.
toolchain_type(name = "runtime_toolchain_type")

java_runtime_alias(name = "current_java_runtime")

java_host_runtime_alias(name = "current_host_java_runtime")

java_toolchain_alias(name = "current_java_toolchain")

filegroup(
    name = "BUILD-jdk",
    srcs = [":BUILD"],
)

# This is necessary to get the *host* Java runtime. Depending on
# //tools/jdk:current_java_runtime from an attribute with the host transition
# does not work because the dependency is determined based on the configuration
# *before* the transition.
alias(
    name = "java_runtime_alias",
    actual = "@bazel_tools//tools/jdk:current_java_runtime",
)

java_runtime_files(
    name = "jni_header",
    srcs = ["include/jni.h"],
)

java_runtime_files(
    name = "jni_md_header-darwin",
    srcs = ["include/darwin/jni_md.h"],
)

java_runtime_files(
    name = "jni_md_header-linux",
    srcs = ["include/linux/jni_md.h"],
)

java_runtime_files(
    name = "jni_md_header-windows",
    srcs = ["include/win32/jni_md.h"],
)

java_runtime_files(
    name = "jni_md_header-freebsd",
    srcs = ["include/freebsd/jni_md.h"],
)

alias(
    name = "java",
    actual = "@local_jdk//:java",
)

alias(
    name = "jar",
    actual = "@local_jdk//:jar",
)

alias(
    name = "javac",
    actual = "@local_jdk//:javac",
)

# On Windows, executables end in ".exe", but the label we reach it through
# must be platform-independent. Thus, we create a little filegroup that
# contains the appropriate platform-dependent file.
filegroup(
    name = "ijar",
    srcs = select({
        "//src/conditions:remote": ["//third_party/ijar:ijar"],
        "//conditions:default": glob(["ijar/*"]),
    }),
)

# On Windows, Java implementation of singlejar is used. We create a little
# filegroup that contains the appropriate platform-dependent file.
# Once https://github.com/bazelbuild/bazel/issues/2241 is fixed (that is,
# the native singlejar is used on windows), this file group can be reused since
# on Windows, executables end in ".exe", but the label we reach it through
# must be platform-independent.
filegroup(
    name = "singlejar",
    srcs = select({
        "//src/conditions:remote": ["//src/tools/singlejar:singlejar"],
        "//conditions:default": glob(["singlejar/*"]),
    }),
)

filegroup(
    name = "genclass",
    srcs = ["//tools/jdk:GenClass_deploy.jar"],
)

filegroup(
    name = "turbine",
    srcs = ["//tools/jdk:turbine_deploy.jar"],
)

filegroup(
    name = "javabuilder",
    srcs = ["//tools/jdk:JavaBuilder_deploy.jar"],
)

filegroup(
    name = "vanillajavabuilder",
    srcs = ["//tools/jdk:VanillaJavaBuilder_deploy.jar"],
)

BOOTCLASS_JARS = [
    "rt.jar",
    "resources.jar",
    "jsse.jar",
    "jce.jar",
    "charsets.jar",
]

# TODO(cushon): this isn't compatible with JDK 9
alias(
    name = "bootclasspath",
    actual = "@local_jdk//:bootclasspath",
)

alias(
    name = "extclasspath",
    actual = "@local_jdk//:extdir",
)

# TODO(cushon): migrate to extclasspath and delete
alias(
    name = "extdir",
    actual = "@local_jdk//:extdir",
)

filegroup(
    name = "langtools",
    srcs = ["//third_party/java/jdk/langtools:javac_jar"],
)

java_import(
    name = "langtools-neverlink",
    jars = [":langtools"],
    neverlink = 1,
)

alias(
    name = "jre",
    actual = "@local_jdk//:jre",
)

alias(
    name = "jdk",
    actual = "@local_jdk//:jdk",
)

alias(
    name = "host_jdk",
    actual = "@embedded_jdk//:jdk",
)

genrule(
    name = "platformclasspath",
    srcs = ["DumpPlatformClassPath.java"],
    outs = ["platformclasspath.jar"],
    cmd = """
set -eu
TMPDIR=$$(mktemp -d -t tmp.XXXXXXXX)
$(JAVABASE)/bin/javac -source 8 -target 8 \
    -cp $(JAVABASE)/lib/tools.jar \
    -d $$TMPDIR $<
$(JAVA) -XX:+IgnoreUnrecognizedVMOptions \
    --add-exports=jdk.compiler/com.sun.tools.javac.platform=ALL-UNNAMED \
    -cp $$TMPDIR DumpPlatformClassPath $@
rm -rf $$TMPDIR
""",
    toolchains = ["@bazel_tools//tools/jdk:current_host_java_runtime"],
    tools = ["@bazel_tools//tools/jdk:current_host_java_runtime"],
)

default_java_toolchain(
    name = "toolchain_hostjdk8",
    bootclasspath = [":platformclasspath"],
    jvm_opts = [
        "-Xbootclasspath/p:$(location @bazel_tools//third_party/java/jdk/langtools:javac_jar)",
    ],
    source_version = "8",
    target_version = "8",
)

default_java_toolchain(
    name = "toolchain",
    bootclasspath = [":platformclasspath"],
    source_version = "8",
    target_version = "8",
)

default_java_toolchain(
    name = "toolchain_java9",
    misc = DEFAULT_JAVACOPTS + [
        "--release",
        "9",
    ],
)

default_java_toolchain(
    name = "toolchain_java10",
    misc = DEFAULT_JAVACOPTS + [
        "--release",
        "10",
    ],
)

filegroup(
    name = "srcs",
    srcs = [
        "BUILD-jdk",  # Tools are build from the workspace for tests.
        "DumpPlatformClassPath.java",
        "default_java_toolchain.bzl",
        "proguard_whitelister.py",
        "proguard_whitelister_test.py",
        "proguard_whitelister_test_input.cfg",
    ],
)

filegroup(
    name = "package-srcs",
    srcs = glob(["**"]),
)

py_binary(
    name = "proguard_whitelister",
    srcs = [
        "proguard_whitelister.py",
    ],
    deps = [
        "//third_party/py/gflags",
    ],
)

py_test(
    name = "proguard_whitelister_test",
    srcs = ["proguard_whitelister_test.py"],
    data = ["proguard_whitelister_test_input.cfg"],
    deps = [
        ":proguard_whitelister",
    ],
)

# For java coverage
alias(
    name = "jacoco-blaze-agent",
    actual = "//third_party/java/jacoco:blaze-agent",
)

java_import(
    name = "JacocoCoverage",
    jars = [":JacocoCoverage_deploy.jar"],
)

test_suite(
    name = "windows_tests",
    tags = [
        "-no_windows",
        "-slow",
    ],
    visibility = ["//visibility:private"],
)

test_suite(
    name = "all_windows_tests",
    tests = [":windows_tests"],
    visibility = ["//tools:__pkg__"],
)
