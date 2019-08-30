---
layout: post
title: Clang Static Analyzer
date: 2017-03-20 09:46:59 +0800
categories: 开发知识
tags: clang iOS Mobile StaticAnalyzer
---

A brief post about [Clang Static Analyzer](http://clang-analyzer.llvm.org/) and [scan-build](http://clang-analyzer.llvm.org/scan-build.html) tool.

Clang Static Analyzer is a source code analysis tool that finds bugs in C, C++, and Objective-C programs. Yep, no Swift yet.

## Xcode and xcodebuild

You may have used it already since a stable build of clang static analyzer comes bundled with Xcode. It’s ⌘⇧B (Command + Shift + B) shortcut in Xcode or `analyze` action when building from command line.

``` sh
xcodebuild analyze \
    -project MyProject.xcodeproj \
    -scheme MyScheme \
    -sdk iphonesimulator \
    -destination "name=iPhone 6s Plus,OS=9.2" \
    -configuration Debug
```

Note the use of `-sdk` and `-destination` options. Latest Xcode versions may play dumb when used from command line and will want explicit destination specified. By default the build destination will be `Generic iOS Device` and will most likely require code signing.

When running Analyze action in Xcode you get a beautiful report with nice arrows rendered right on top of the source code. While running from command line the analyzer errors will show up in build log.

The build log by itself should be enough for integration with CI servers, but you can actually get more out of standard `xcodebuild analyze` action. Using `CLANG_ANALYZER_OUTPUT` and `CLANG_ANALYZER_OUTPUT_DIR` you can control in which form analyzer creates report and where. Valid options for `CLANG_ANALYZER_OUTPUT` are `text`, `html` and `plist`. You can combine multiple options using dashes, e.g. `plist-html`.

``` sh
xcodebuild analyze \
    -project MyProject.xcodeproj \
    -scheme MyScheme \
    -sdk iphonesimulator \
    -destination "name=iPhone 6s Plus,OS=9.2" \
    -configuration Debug \
    CLANG_ANALYZER_OUTPUT=plist-html \
    CLANG_ANALYZER_OUTPUT_DIR="$(pwd)/clang"
```

There will be a lot of output files in `clang` directory, using `find` you can locate HTML report.

``` sh
find clang -name "*.html"
# It will find a file named like report-f27e58.html
```

Open it in a browser and it looks just like the one in Xcode.


## Under the Hood

If you are curious what are those not-so-much-documented `CLANG_ANALYZER_` build settings, this section may shed some light for you.

`CLANG_ANALYZER_OUTPUT=html` translates into `-Xclang -analyzer-output=html` when `xcodebuild` composes and executes clang command. Similar story for `CLANG_ANALYZER_OUTPUT_DIR`. You can call `clang -cc1 --help` yourself, there are a lot of interesting things in the help message. For example, running `clang -cc1 -analyze -analyzer-checker-help` will list all the available checkers. If you look at build log closely now, you will see how xcodebuild configures all those checkers using `-Xclang -analyzer-config` and `-Xclang -analyzer-checker` flags.

With this knowledge, you should be able to tweak default Xcode Analyze configuration by modifying Xcode build settings. To be honest, I never had to do that. If you really want more control over static analyzer, you should look at [scan-build](http://clang-analyzer.llvm.org/scan-build.html) tool.

## scan-build

*If Xcode comes with a static analyzer already available, why bother and use another tool?*

`scan-build` has a newer version of static analyzer with a number of experimental checks enabled. For example, using Xcode 7.2.1 I got 0 analyzer warnings when running Analyze action on a given project. For the very same project `scan-build` finds 6 warnings related to nullability and detects possible `nil` values passed to methods that don’t expect `nil`.


### Installation

Clang Static Analyzer is not available for installation via Homebrew directly. You can get it as part of `llvm` package if installed with `--with-clang` option, but this is not recommended, because Homebrew `llvm` version will not be the same as Apple LLVM version. You won’t be able to build your iOS projects with this toolchain right away.

So I would recommend to use the [custom Homebrew tap](https://github.com/mgrebenets/homebrew-scan-build) instead.

``` sh
brew tap mgrebenets/scan-build
brew install scan-build
```

Run `scan-build` to make sure installation is successful.

### Usage

You can use it in two ways, from Xcode or from command line.

To learn how to use it from Xcode [read this documentation](http://clang-analyzer.llvm.org/xcode.html). I never actually did it. Aware that others mention [a number of caveats](http://loufranco.com/blog/xcode-better-build-and-analyze) with this approach, some not so obvious configuration tweaks required to make it work properly.
I’m more interested in running it from command line and generating reports for CI tasks. Here’s [the original documentation](http://clang-analyzer.llvm.org/scan-build.html) with some good examples.

The basic usage is supposed to be as simple as this

``` sh
# Build a scheme
scan-build -k -v -v xcodebuild clean build \
    -project MyProject.xcodeproj \
    -scheme MyScheme \
    -destination "name=iPhone 6s Plus,OS=9.2" \ 
    -configuration Debug
```

Note the use of `clean build` and not just `build`. I have discovered that `clean` action may not be necessary and for me just running `build` action works as well and yields the same results.

By default `scan-build` will use static analyzer version bundled with its own installation (`clang-check`). You may choose to specify an explicit path to static analyzer executable or fall back to using the version bundled with Xcode.

``` sh
# Use analyzer bundled with xcode
scan-build -k -v -v --use-analyzer Xcode \
  xcodebuild clean build \
  -project MyProject.xcodeproj \
  -scheme MyScheme \
  -destination "name=iPhone 6s Plus,OS=9.2" \        
  -configuration Debug

# Use explicit path to clang-check executable
CLANG_CHECK=$(which clang-check)

scan-build -k -v -v --use-analyzer ${CLANG_CHECK} \
  xcodebuild clean build \
  -project MyProject.xcodeproj \
  -scheme MyScheme \
  -destination "name=iPhone 6s Plus,OS=9.2" \        
  -configuration Debug
```

### Reports

Use `-o` (output) option to specify output directory for reports.

``` sh
mkdir -p clang-reports

# Build a target
scan-build -k -v -v \
  --use-analyzer Xcode \
  -o clang-reports \
  xcodebuild clean build \
  -project MyProject.xcodeproj \
  -scheme MyScheme \
  -destination "name=iPhone 6s Plus,OS=9.2" \        
  -configuration Debug
```

There are dozens of other options you can use to customize `scan-build`, try `scan-build -h` to see all of them.

It is also supposed to be easier to customize checkers with `scan-build`. I had never had a chance to do that, you must have some compelling reason to go beyond default configuration.


### Troubleshooting

If you get an error like this: `clang: error: unknown argument: '-fembed-bitcode-marker'`, make sure you have specified both `-sdk` and `-destination` options to `xcodebuild`.

Same applies for `xcodebuild` failing to find imports like `#import <UIKit/UIKit.h>`.
