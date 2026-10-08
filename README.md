> This repository is a backup of the GitHub source. Original repository: https://github.com/OpenVapeCN/VapeV4.21

# Vape 4.21 Product Recovery

A research-oriented recovery project for the Vape 4.21 Java layer and the Windows x64 native bridge layer.

### It is not Vape's official source code, original release package, or a vendor-signed artifact, and it is not guaranteed to behave identically to the original product.

> This project is intended for software recovery, compatibility analysis, and testing in your own environment. It should only be
> used in isolated instances that you own and are authorized to test, and you are responsible for confirming your local laws,
> software licenses, and server rules.

## Current Status

| Scope | Status |
| --- | --- |
| Java source | 2,939 sample owned-package source files, compiles normally via Gradle |
| Resources | 230 mappings, fonts, textures, shaders, sounds, and localization resources |
| Recovery baseline | `baseline/final.jar`, with a fixed SHA-256 verified at build time |
| Injection payload | Self-contained Shadow JAR, targeting Java 8 class-file major 52 |
| Native bridge | Windows x64 JNI/JVMTI DLL and a `LoadLibraryW` injector |
| Runtime verification | Build and payload structure verified; full in-game behavior still requires further testing |

Currently supports **Minecraft 1.7.10 Forge, 1.8.9 Forge, and 1.12.2 Forge**, and supports injection into Forge-enabled
Lunar Client instances. Vanilla and Fabric are not supported. Support for Minecraft 1.16.5 is poor; some mappings, rendering,
and module features may not work correctly. All target instances must use a 64-bit JVM.

## Requirements

Required to compile and verify only the Java layer:

- JDK 17, used as the Gradle toolchain; output is compiled with `--release 8` by default
- The Gradle Wrapper bundled with the project; the build script pins Gradle 8.8
- Network access to Maven Central and the Gradle Plugin Portal

Building the native bundle additionally requires:

- Windows x64
- Visual Studio 2022 C++ x64 toolchain and the Windows SDK
- CMake 3.21 or later
- A JDK that includes the JNI/JVMTI headers; JDK 8 is recommended when testing against 1.7.10, 1.8.9, and 1.12.2

## Quick Start

From the repository root in PowerShell:

```powershell
.\gradlew.bat clean build verifyInjectionPayload
```

This command does the following:

1. Compiles the recovery source and processes all resources.
2. Verifies the SHA-256 of `baseline/final.jar` and the recovery metadata.
3. Checks the source file count and any residual fatal CFR decompilation markers.
4. Generates the injection JAR including runtime dependencies.
5. Confirms the payload contains the required packages and that all classes can be loaded by Java 8.

The main Java artifacts are located in `build/libs/`. To generate IntelliJ IDEA project configuration, run:

```powershell
.\gradlew.bat idea
```

## Building the Native Test Bundle

```powershell
.\gradlew.bat prepareInjectionBundle -PtargetRelease=8 `
  -PnativeJavaHome="C:\Program Files\Java\jdk1.8.0_301"
```

The complete test bundle is output to `build/injection/`:

```text
Vape421Native.dll
Vape421Injector.exe
README.md
```

The DLL embeds the Java injection JAR as `RCDATA`, so no separate payload placement is required. The native bridge layer only
implements the interfaces recovered from the sample's nine-entry `RegisterNatives` table; additional Java native declarations
that were not registered in the sample are not implemented speculatively. See [`native/README.md`](native/README.md) for more
details.

## Running in an Isolated Environment

After starting a supported Forge instance using a 64-bit JVM, or a Forge-enabled Lunar Client instance, run the following from
`build/injection/`:

```powershell
.\Vape421Injector.exe <pid> .\Vape421Native.dll
```

The injector only performs `LoadLibraryW`. Once the DLL is loaded, it waits for the JVM and the Minecraft `Client thread`, loads
the embedded JAR through its context ClassLoader, registers the nine native methods, and calls
`gg.vape.runtime.NativeBridge.start()`. Execution results are written to `vape421-native.log` in the same directory as the DLL.

## Common Verification Tasks

| Command | Purpose |
| --- | --- |
| `.\gradlew.bat check` | Compilation, baseline hash, source coverage, and recovery quality checks |
| `.\gradlew.bat injectionJar` | Build the self-contained Java injection payload |
| `.\gradlew.bat verifyInjectionPayload` | Check dependency integrity and Java 8 bytecode version |
| `.\gradlew.bat buildNative` | Build the x64 DLL and the injector |
| `.\gradlew.bat prepareInjectionBundle` | Assemble the native bundle ready for isolated testing |

The SHA-256 of the recovery baseline is:

```text
DEB51671044A6EAE4275A61217AF270F9256FD0D96036EF016DE5E9F6BFE42CE
```


## License

This repository is provided under [CC0 1.0 Universal](LICENSE). Where applicable, CC0 only covers content that the repository
contributors have the right to dispose of; third-party libraries, trademarks, fonts, textures, and other pre-existing materials
remain subject to their respective rights.
