# Building `rug` (GMP/MPFR) on Windows 11 64-bit

**Copy-paste-ready markdown guide**

| Goal | Use the `rug` crate in a Rust project on Windows without compilation errors. |
|------|------------------------------------------------------------------------------|
| Works on | Windows 10/11, 64-bit (GNU toolchain)                                       |
| Time | ~10 minutes once MSYS2 is installed                                         |
| Result | `cargo build` finishes successfully and produces a runnable `.exe`          |

---

## 1. Install MSYS2 (once)

1. Download **msys2-x86_64-*.exe** from [https://www.msys2.org](https://www.msys2.org).  
2. Run the installer → **keep the default path** `C:\msys64`.  
3. **Finish the installer** and **close** the terminal it opens.

---

## 2. Open the **correct** terminal

| Icon | Name in Start Menu | Purpose |
|------|--------------------|---------|
| ![mingw64](https://www.msys2.org/assets/images/mingw64.png) | **MSYS2 MinGW 64-bit** | Build 64-bit MinGW binaries |

> ❗ Do NOT use "MSYS2 MSYS" or "MSYS2 MinGW 32-bit".

---

## 3. One-command install of **everything** you need

In the **MSYS2 MinGW 64-bit** terminal, copy-paste and hit Enter:

```bash
pacman -Syu base base-devel m4 diffutils make pkgconf mingw-w64-x86_64-gcc mingw-w64-x86_64-toolchain mingw-w64-x86_64-gmp mingw-w64-x86_64-mpfr mingw-w64-x86_64-pkgconf
```

Press Enter to confirm any prompts during the installation.

---

## 4. Install Rust GNU target (once)

Open PowerShell or CMD (not MSYS2):

```powershell
rustup toolchain install stable-x86_64-pc-windows-gnu
rustup default stable-x86_64-pc-windows-gnu
```

---

## 5. Add MinGW bin folder to PATH

**Pick one method:**

### One-time (current shell only):
```powershell
$Env:PATH += ";C:\msys64\mingw64\bin"
```

### Permanent (recommended):
1. `Win + S` → search "Environment Variables"
2. Edit **User variables** → **Path** → **New** → `C:\msys64\mingw64\bin` → **OK**
3. Open a new PowerShell window.

---

## 6. Configure your project

In `Cargo.toml`:

```toml
[dependencies]
rug = "1.28"                     # pulls correct gmp-mpfr-sys automatically

[dependencies.gmp-mpfr-sys]
version = "1.6"                  # same version rug uses
features = ["use-system-libs"]   # tells it *not* to compile from source
```

---

## 7. Build

```powershell
cargo clean   # optional but avoids stale cache
cargo build
```

**You should see:**
```
   Compiling gmp-mpfr-sys v1.6.7
   Compiling rug v1.28.0
   Compiling your_project v0.1.0
    Finished dev [unoptimized + debuginfo] target(s) in …
```

---

## Troubleshooting

### If you get "linker not found" errors:
- Verify PATH contains `C:\msys64\mingw64\bin`
- Restart your terminal after changing PATH

### If build still tries to compile from source:
- Check that `features = ["use-system-libs"]` is correctly set
- Ensure the MSYS2 packages are installed in the correct terminal

### If you see "permission denied" errors:
- Run MSYS2 terminal as Administrator
- Or check file permissions in the MSYS2 installation directory

---

## Verification

Test that everything works by creating a simple test program:

```rust
use rug::Integer;

fn main() {
    let a = Integer::from(123);
    let b = Integer::from(456);
    let result = a * b;
    println!("123 * 456 = {}", result);
}
```

Run with:
```powershell
cargo run
```

Expected output:
```
123 * 456 = 56088
```
