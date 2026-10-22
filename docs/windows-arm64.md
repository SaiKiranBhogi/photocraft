# Windows ARM64 development

PhotoCraft already has a Windows ARM64 release target and a scheduled/package validation workflow. Windows packages target `aarch64-pc-windows-msvc`; the package script checks that both executables have the ARM64 PE machine type before producing an installer. See [the workflow](../.github/workflows/windows-arm64.yml) and [package script](../packaging/windows/package.ps1).

## Build on a Windows ARM64 device

1. Install Rust stable with the MSVC host toolchain using [rustup](https://rustup.rs/). In PowerShell:

   ```powershell
   rustup toolchain install stable
   rustup default stable
   rustup target add aarch64-pc-windows-msvc
   ```

2. In Visual Studio Installer, install Visual Studio 2022 or Build Tools 2022 with **Desktop development with C++**, the **MSVC v143 ARM64/ARM64EC build tools**, and a current **Windows 10/11 SDK**. Include **C++ Clang Compiler for Windows** and ensure `clang-cl.exe` is on `PATH`; the `aws-lc-sys` dependency uses Clang for its ARMv8 crypto routines.

3. Open a new Developer PowerShell and confirm the tools resolve:

   ```powershell
   rustc -vV
   cargo --version
   where.exe clang-cl
   where.exe rc
   ```

   The Rust host should be `aarch64-pc-windows-msvc` on an ARM64 installation. The explicit target below ensures the output is ARM64 even if you are cross-compiling from x64.

4. Build and run PhotoCraft:

   ```powershell
   cargo build --release --locked -p photocraft -p photocraft-cli --features heif --target aarch64-pc-windows-msvc
   cargo run --release --target aarch64-pc-windows-msvc -p photocraft
   ```

5. To produce the MSI and portable ZIP, install .NET and WiX 5.0.2 with its UI and Util extensions as listed in the package script, then run:

   ```powershell
   pwsh packaging/windows/package.ps1 -Arch arm64
   ```

The packaging script validates the PE machine type (`0xAA64`) for the app and CLI. Run the resulting executable on Windows 11 ARM64 to execute it natively. The shared GitHub workflow also cross-compiles on Windows x64 and installs/runs the package on a native `windows-11-arm` runner.

## Other tools

Node.js, Python, and CMake are not required by PhotoCraft's Windows Rust build or packaging workflow. WiX requires the .NET SDK; Windows packaging also requires the Windows SDK utilities. A cross-compile may use x64-hosted Visual Studio tools, but the Rust target and resulting application are ARM64.
