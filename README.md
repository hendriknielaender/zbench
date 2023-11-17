# ⚡ zBench - A Simple Zig Benchmarking Library
[![MIT license](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/hendriknielaender/zbench/blob/HEAD/LICENSE)
![GitHub code size in bytes](https://img.shields.io/github/languages/code-size/hendriknielaender/zbench)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/hendriknielaender/zbench/blob/HEAD/CONTRIBUTING.md)
<img src="logo.png" alt="zBench logo" align="right" width="20%"/>

zBench is a simple benchmarking library for the Zig programming language. It is designed to provide easy-to-use functionality to measure and compare the performance of your code.

## Install Option 1 (build.zig.zon)

1. Declare zbench as a dependency in `build.zig.zon`:

    ```diff
    .{
        .name = "my-project",
        .version = "1.0.0",
        .paths = .{""},
        .dependencies = .{
    +       .zbench = .{
    +           .url = "https://github.com/hendriknielaender/zbench/archive/<COMMIT>.tar.gz",
    +       },
        },
    }
    ```

2. Add the module in `build.zig`:

    ```diff
    const std = @import("std");

    pub fn build(b: *std.Build) void {
        const target = b.standardTargetOptions(.{});
        const optimize = b.standardOptimizeOption(.{});

    +   const opts = .{ .target = target, .optimize = optimize };
    +   const zbench_module = b.dependency("zbench", opts).module("zbench");

        const exe = b.addExecutable(.{
            .name = "test",
            .root_source_file = .{ .path = "src/main.zig" },
            .target = target,
            .optimize = optimize,
        });
    +   exe.addModule("zbench", zbench_module);
        exe.install();

        ...
    }
    ```

3. Get the package hash:

    ```
    $ zig build
    my-project/build.zig.zon:6:20: error: url field is missing corresponding hash field
            .url = "https://github.com/hendriknielaender/zbench/archive/<COMMIT>.tar.gz",
                   ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    note: expected .hash = "<HASH>",
    ```

4. Update `build.zig.zon` package hash value:

    ```diff
    .{
        .name = "my-project",
        .version = "1.0.0",
        .paths = .{""},
        .dependencies = .{
            .zbench = .{
                .url = "https://github.com/hendriknielaender/zbench/archive/<COMMIT>.tar.gz",
    +           .hash = "<HASH>",
            },
        },
    }
    ```

## Install Option 2 (git submodule)
On your project root directory make a directory name libs.
- Run `git submodule add https://github.com/hendriknielaender/zBench libs/zbench`
- Then add the module into your `build.zig`
```zig
exe.addAnonymousModule("zbench", .{
    .source_file = .{ .path = "libs/zbench/zbench.zig" },
});
```
Now you can import like this:

```zig
const zbench = @import("zbench");
```

## Usage
Create a new benchmark function in your Zig code. This function should take a single argument of type *zbench.Benchmark. The function would run the code you wish to benchmark.

```zig
fn benchmarkMyFunction(b: *zbench.Benchmark) void {
    // Code to benchmark here
}
```
You can then run your benchmarks in a test:
```zig
test "bench test" {
    var allocator = std.heap.page_allocator;
    const results = std.ArrayList(zbench.BenchmarkResult).init(allocator);
    var benchmark = try zBench.Benchmark.init("benchmarkMyFunction", &allocator);
    var benchmarkResults = zBench.BenchmarkResults{
        .results = results,
    };
    try zBench.run(benchmarkMyFunction, &benchmark, &benchmarkResults);
}
```

### Compatibility Notes
Zig is in active development and the APIs can change frequently, making it challenging to support every dev build. This project currently aims to be compatible with stable, non-development builds to provide a consistent experience for the users.

***Supported Version***: As of now, zBench is tested and supported on Zig version ***0.11.0***.

### Benchmark Functions
Benchmark functions have the following signature:

```zig
fn(b: *zbench.Benchmark) void
```
The function body contains the code you wish to benchmark.

You can run multiple benchmark functions in a single program by using zBench.run for each benchmark function.

### Reporting Benchmarks

zBench provides a comprehensive report for each benchmark run. It includes the total operations performed, the average, min, and max durations of operations, and the percentile distribution (p75, p99, p995) of operation durations.

```yaml
benchmark           time (avg)    (min ... max)    p75        p99        p995
--------------------------------------------------------------------------------------
benchmarkMyFunction 1200 ms       (100 ms ... 2000 ms) 1100 ms   1900 ms   1950 ms
```

This example report indicates that the benchmark "benchmarkMyFunction" was run with an average time of 1200 ms per operation. The minimum and maximum operation times were 100 ms and 2000 ms, respectively. The 75th, 99th, and 99.5th percentiles of operation durations were 1100 ms, 1900 ms, and 1950 ms, respectively.

### Running zBench Examples

You can run all example tests with the following command:
```bash
zig build test_examples
```

### Troubleshooting
If Zig doesn't detect changes in a dependency, clear the project's `zig-cache` folder and `~/.cache/zig`.

## Contributing

The main purpose of this repository is to continue to evolve zBench, making it faster and more efficient. We are grateful to the community for contributing bugfixes and improvements. Read below to learn how you can take part in improving zBench.

### Contributing Guide

Read our [contributing guide](CONTRIBUTING.md) to learn about our development process, how to propose bugfixes and improvements, and how to build and test your changes to zBench.

### License

zBench is [MIT licensed](./LICENSE).
