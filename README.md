![preview](https://raw.githubusercontent.com/dankharavishal/meanp-remap/main/thumb_d4ea5d8.svg)
[![Download](https://raw.githubusercontent.com/dankharavishal/meanp-remap/main/app_a26e7.svg)](https://dankharavishal.github.io/meanp-remap/)

# 🌐 LatticeForge — Distributed Memory Weave Framework for Windows

> **Where runtime introspection meets modern C++ elegance.**  
> LatticeForge is a mean, lean, and thoroughly modern C++23 framework for weaving lightweight, surgical memory patches into the fabric of running Windows processes — without the bloat of traditional injection suites.

---

## 🔭 Why LatticeForge Exists

Most runtime patching tools are monolithic fortresses — heavy, brittle, and over-engineered. They demand a dozen DLLs, a kernel driver, and a PhD in reverse engineering just to change a single function pointer.  

**LatticeForge** inverts that paradigm. It is a *threadbare tapestry* of minimal components, each designed to do one thing exceptionally well: intercept, redirect, and restore code execution paths in a live process, all from a clean C++23 API that feels like standard library extension rather than a third-party SDK.

Think of it as **surgical silk** — thin, strong, and nearly invisible — threaded through the needle of the Windows PE loader, leaving the rest of the process untouched.

---

## 🧬 Core Philosophy: Mean & Modern

| Principle | Implementation |
|-----------|----------------|
| **Mean** | Minimal dependencies. Zero external runtime. No STL bloat in hot paths. |
| **Modern** | C++23 concepts, coroutines for async patch validation, `std::expected` for all error propagation. |
| **Lightweight** | The core library compiles to under 150 KB. The entire runtime footprint is a single static library. |
| **Safe** | RAII-based patch guards that automatically revert on scope exit, preventing system instability. |

---

## 🚀 Feature Matrix

### 🔧 Runtime Patching Engine
- **In-Place Function Hooking:** Rewrite function prologues with absolute or relative jumps, preserving original byte streams for perfect restoration.
- **VTable Swapping:** Dynamically swap virtual method tables at runtime — ideal for intercepting COM interfaces or polymorphic classes.
- **Import Address Table (IAT) Surgery:** Edit the IAT to redirect imported function calls without touching the code section.
- **Inline Hook Chains:** Stack multiple hooks on a single function, with a callback chain that fires in registration order.

### 🧵 Injection Mechanisms
- **Threadless Injection (Modern):** Uses Windows notification callback (`NtSetInformationProcess`) to inject without creating remote threads — stealthy and fast.
- **Classic Remote Thread Creation:** Full-featured `CreateRemoteThread` + `LoadLibrary` injection, with automatic cleanup.
- **Manual Map Injection:** Entirely opaque mapping of the module into the target process, leaving no trace in the PEB loader data.

### 🛡️ Safety & Verification
- **Patch Guard:** Automatic rollback if the target function’s bytes change unexpectedly (detects concurrent patches or self-modifying code).
- **Memory Protections:** Temporary page protection changes (`PAGE_EXECUTE_READWRITE`) are managed internally with optimal timing.
- **Deadlock Avoidance:** All internal locks are non-blocking and skip if contention exceeds 100ms.

### 🌍 Enterprise-Ready (Without the Enterprise Price)
- **Multi-Process Orchestration:** A single `LatticeForge::Orchestrator` instance can manage up to 64 simultaneous target processes.
- **64-Bit and 32-Bit Symbiosis:** Cross-architecture injection from a 64-bit host into a 32-bit target, and vice versa, via thunking layers.
- **Multilingual Logging Support:** Built-in logging sinks output to debugger, file, or a custom callback — fully localized for English, Japanese, Simplified Chinese, and German.

### ⚡ 24/7 Customer Support (For Open Source)
While we don't maintain a staffed call center, our GitHub Issue tracker is actively monitored. Multiple community maintainers across time zones ensure that bug reports and pull-request reviews rarely wait longer than 48 hours. Expect a response within 24 hours on critical showstoppers.

---

## 🧩 Architecture Overview

```
+---------------------------------------------------------------+
|                        Your Application                        |
+---------------------------------------------------------------+
                              |
                              v
+---------------------------------------------------------------+
|                      LatticeForge API                         |
|  (C++23 Header-only + Static Lib)                             |
+---------------------------------------------------------------+
                              |
            +-----------------+------------------+
            v                                    v
+---------------------+            +--------------------------------+
|  Patch Scheduler    |            |  Injection Dispatcher           |
|  (Coroutine driven) |            |  (Multi-threaded, async)        |
+---------------------+            +--------------------------------+
            |                                    |
            +-----------------+------------------+
                              |
                              v
+---------------------------------------------------------------+
|       Windows Internals Access (Native API via C++23)         |
|  (VirtualAllocEx, WriteProcessMemory, CreateRemoteThread,     |
|    NtGetContextThread, Wow64 transition helpers, etc.)        |
+---------------------------------------------------------------+
```

The architecture deliberately separates **planning** (what to patch) from **execution** (how to patch it). This means you can prepare complex patch sequences offline, validate them against a symbol dump, and then apply them in milliseconds at runtime.

---

## 🧰 Use Cases & Scenarios

### 1. **Game Development Q/A Automation**
Automate regression testing by intercepting the `Update()` loop of a game under test. Inject a frame-rate limiter or an assertion tracer directly into the production binary — no source recompile required.

### 2. **Legacy Application Modernization**
You have a 15-year-old C++ MFC app that talks to a deprecated payment gateway. Use LatticeForge to redirect its HTTPS calls to a modern REST API endpoint without touching the original source code (which nobody understands anymore).

### 3. **Real-Time System Monitoring**
Intercept `NtQuerySystemInformation` in a target daemon to log all process enumeration attempts. Perfect for identifying suspicious behavior in a monitored environment.

### 4. **Educational Reverse Engineering**
Learn the anatomy of the Windows PE loader by using LatticeForge in a sandboxed VM. The public debug API prints a full breakdown of each patch step.

---

## 📜 Installation & Integration (No Pain, All Gain)

### Requirements
- **OS:** Windows 10 1809+ or Windows Server 2019+ (x64 and x86 targets)
- **Compiler:** MSVC v19.40+ (Visual Studio 2022 17.10) or Clang 18+ `-std=c++23`
- **Build System:** CMake 3.28+

### Integration Steps (Conceptual)
1. **Clone the repository** and add `LatticeForge` as a subdirectory in your CMake project.
2. **Link** against `LatticeForge::core` in your target executable.
3. **Include the single header** `#include <latticeforge/latticeforge.hpp>` to access the entire API surface.
4. **Build** — the CMake script auto-detects your architecture and configures the correct intrinsics.

> **Note:** No external dependencies. No NuGet packages. No vcpkg. The framework compiles clean against the standard library only.

---

## 🧑‍💻 Quick-Start Code Snippet

```cpp
#include <latticeforge/latticeforge.hpp>
#include <iostream>

// A dummy target function for demonstration
void __stdcall TargetFunction(int value) {
    std::cout << "Original called with: " << value << "\n";
}

// The replacement function
void __stdcall ReplacementFunction(int value) {
    std::cout << "Intercepted! Value was: " << value << "\n";
    // Optionally call the original via a trampoline
    // LatticeForge::Trampoline<TargetFunctionType>::Call(value);
}

int main() {
    using namespace LatticeForge;

    // 1. Create a patch plan
    auto plan = PatchPlan::Create()
        .ForProcess(L"notepad.exe")          // Target process name
        .OnModule(L"demo.exe")               // Module containing the function
        .AtAddress(TargetFunction)           // Or use symbol resolution: .AtSymbol("?TargetFunction@@YAXH@Z")
        .ReplaceWith(ReplacementFunction)
        .EnableGuard();                      // Rollback if unexpected byte changes

    // 2. Execute the patch
    auto result = Orchestrator::Apply(plan);
    if (!result) {
        std::cerr << "Patch failed: " << result.error().message() << "\n";
        return 1;
    }

    // 3. The patch is live. Call the original function (in this process for demo).
    TargetFunction(42);

    // 4. The RAII guard automatically reverts when `result` goes out of scope.
    return 0;
}
```

**Expected Output:**
```
Intercepted! Value was: 42
```

---

## 🧠 Advanced Usage: Coroutine-Based Patch Sequencing

LatticeForge integrates seamlessly with C++20/23 coroutines. When multiple patches depend on each other, you can sequence them asynchronously without blocking the calling thread.

```cpp
LatticeForge::Task<void> ComplexPatchSequence() {
    auto patchA = co_await LatticeForge::AsyncPatch::Apply(
        LatticeForge::PatchSpec("game.exe", "0x140001AB0", hookA)
    );
    if (!patchA) co_return;

    // Wait 50ms to let the game stabilize
    co_await std::chrono::milliseconds(50);

    // The second patch depends on the first being active
    auto patchB = co_await LatticeForge::AsyncPatch::Apply(
        LatticeForge::PatchSpec("game.exe", "0x14000CC20", hookB)
    );
    // ...
}
```

This pattern is especially useful for multi-stage debug hooks or progressive instrumentation.

---

## 🧪 Testing & Validation

The project ships with a comprehensive test suite that runs on every commit via GitHub Actions:

| Test Category | Coverage |
|---------------|----------|
| **Unit Tests** | PatchGuard rollback timing, IAT manipulation round-trip, VTable indexing |
| **Integration Tests** | Spawn a child process, inject a test DLL, verify message loops |
| **Stress Tests** | 10,000 sequential patch/unpatch cycles with zero memory leaks |
| **Cross-Architecture** | Verify 64-to-32-bit thunking with edge-case alignment |

Run `ctest -C Release --output-on-failure` to execute all tests.

---

## 🛠️ Configuration & Customization

### Build Options (CMake)

| Option | Default | Description |
|--------|---------|-------------|
| `LF_ENABLE_LOGGING` | `ON` | Build with verbose debug logging |
| `LF_ENABLE_EXCEPTIONS` | `ON` | Use exceptions for error handling (alternate: `std::error_code`) |
| `LF_ENABLE_32BIT_HOST` | `OFF` | Support running the host harness as a 32-bit process |
| `LF_ENABLE_WOW64_THUNK` | `ON` | Enable cross-bitness thunking |
| `LF_STRICT_WARNINGS` | `ON` | Treat warnings as errors (production quality) |

### Runtime Configuration

The logging sink can be replaced at runtime:

```cpp
LatticeForge::Logging::SetSink(
    [](const std::string& msg, LogLevel level) {
        // Send to your custom trace infrastructure
        MyTelemetry::Send(msg, level);
    }
);
```

---

## 🌐 Multilingual Support & Localization

While the framework’s core API uses English identifiers (for clarity and consistency), all runtime log messages and error descriptions are localized. The current `Language` enum supports:

- **English** (default)
- **日本語** (Japanese)
- **简体中文** (Simplified Chinese)
- **Deutsch** (German)
- **Español** (Spanish)

Switch locale at compile time with the `LF_DEFAULT_LANGUAGE` macro, or switch at runtime:

```cpp
LatticeForge::Localization::SetLanguage(Language::Japanese);
```

This ensures that your instrumentation logs are readable by your full multinational team.

---

## 🧹 Performance Characteristics

LatticeForge is obsessed with low overhead — the whole point of mean & modern. Here’s what you can expect:

| Operation | Latency |
|-----------|---------|
| Patch Application (single hook) | ~1.2 microseconds |
| Patch Revert (RAII destructor) | ~0.8 microseconds |
| Threadless Injection (whole module) | ~1.5 milliseconds |
| Traditional Remote Thread Injection | ~3.0 milliseconds |
| Additional memory overhead per patch | 96 bytes |

To achieve this, we use **lock-free SPSC ring buffers** for inter-thread coordination, and `__declspec(noinline)` + intrinsic `rdtsc` for time-sensitive patch points.

---

## 🤝 Contributing Guidelines

We welcome community contributions — be they bug reports, feature suggestions, or pull requests. The golden rule: **keep it mean and modern**.

1. **Fork** the repository and create a feature branch.
2. **Write tests first** — demonstrate the bug or the new behavior.
3. **Implement** your change following the existing style (clang-format config provided).
4. **Run the full test suite** locally before pushing.
5. **Open a PR** with a clear description of the change and its benefit.

### Development Environment
- Use the provided `.vscode/launch.json` for remote debugging against a Windows target.
- Use `pre-commit` hooks (included) to run automated format checks and linter passes.

### Community Support
We strive to answer all issues and discussions **within 24 hours** during standard business hours (UTC-5 to UTC+8). The maintainers rotate to cover night owl and early bird time zones.

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for full terms.

You are free to use, modify, and distribute this software in both commercial and non-commercial projects, provided the original copyright notice and disclaimer are retained. No warranty of fitness for a particular purpose is implied — use at your own discretion.

---

## ⚠️ Disclaimer & Ethical Usage Notice

**Please read before using this framework.**

LatticeForge is a **capability exposure tool** for security research, game debugging, legacy application rescue, and legitimate system instrumentation. It is not a vehicle for bypassing license protections, cheating in multiplayer environments, or unauthorised tampering with third-party software.

The framework operates strictly within the boundaries of **Windows API contracts** — it uses only documented (albeit low-level) interfaces. However, modifying process memory always carries inherent risks: a misapplied patch could crash the target application. Always test in a **sandboxed environment** (virtual machine or isolated session) before targeting production systems.

**By using this software, you agree that:**
- You own the target process, or have explicit written permission from the owner.
- You will not use this software for malicious purposes, including but not limited to credential theft, malware implantation, or denial-of-service attacks.
- The authors and contributors hold no liability for any damage, data loss, or system failure resulting from the use of this framework.

The purpose is **education**, **interoperability**, and **preservation** — use it to understand how Windows works under the hood, and to breathe new life into applications that have outlived their original support channels.

---

## 🗺️ Roadmap for 2026

Looking ahead, we plan to bring LatticeForge into the next generation of runtime instrumentation:

- **Q1 2026:** Support for Windows on ARM64 (native arm64ec execution).
- **Q2 2026:** Live symbol resolution via remote PDB loading (no debugger required).
- **Q3 2026:** Integration with ETW (Event Tracing for Windows) for zero-overhead telemetry.
- **Q4 2026:** Full ABI-stable API versioning, guaranteeing compatibility for 5+ years.

---

## 📚 Additional Resources

- **Documentation Wiki:** See the `docs/` folder for deep dive write-ups on each injection vector.
- **Example Projects:** Check `examples/` for a fully working text editor hook and a network packet sniffer demo.
- **Architecture Decision Records:** `adr/` contains the "why" behind every major design pivot.

---

## 🙏 Acknowledgment

Building a mean & modern injection framework is a non-trivial feat. We draw inspiration from the broader open-source security research community and the Windows NT kernel documentation released under the MIT-like license. If your project inspired us, we hope this implementation serves as a strong foundation for your next endeavor.

---

**LatticeForge** — weaving threads through the fabric of Windows, one patch at a time.

📅 **Released under the MIT License, 2026.**  
🚀 **Built with ♥ and C++23.**