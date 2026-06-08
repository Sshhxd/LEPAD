# LEPAD
### LEPAD (Legacy Parameter Descent) -  Command-line parameters are automatically inherited and propagated through a sequence of binaries where each executable calls the next. The arguments provided to any binary in the sequence are carried forward unchanged through all remaining binaries, culminating in execution of the final command.

<img width="936" height="807" alt="image" src="https://github.com/user-attachments/assets/dde40bb4-6a79-4eaa-9801-1c58a7e2d169" />
*LEPAD chain execution showing argument propagation from first to last binary.*

# LEPAD - Legacy Parameter Descent

## Description

Command-line parameters are automatically inherited and propagated through a sequence of binaries where each executable calls the next. The arguments provided to any binary in the sequence are carried forward unchanged through all subsequent binaries, culminating in execution of the final command.

## Technical Background

### The ShellExecuteW Function

The propagation behavior is a direct consequence of how `ShellExecuteW` handles parameters when `lpParameters` is set to `NULL`.

```c
ShellExecuteW(
    NULL,           // hwnd (parent window handle)
    NULL,           // lpOperation (open, edit, etc.)
    L"wordpad.exe", // lpFile (the program to execute)
    NULL,           // lpParameters (command-line arguments)
    NULL,           // lpDirectory (working directory)
    SW_SHOWNORMAL   // nShowCmd (window state)
);
```
### Why NULL Matters

When `lpParameters` is `NULL`, the Windows API behaves as follows:

1. `ShellExecuteW` checks if the calling process received any command-line arguments
2. If arguments exist, they are automatically forwarded to the child process
3. The API performs no validation, modification, or filtering of these arguments
4. The child process receives the arguments as if they were passed directly

### The Patching Vector

If the original binary contains a **hardcoded string** to an executable which is passed as the `lpFile` parameter. It could be used as:

1. Copying the binary to a new location
2. Replacing the hardcoded string with any other executable name (e.g., cmd.exe)
3. The `NULL` `lpParameters` behavior remains unchanged

This creates an argument forwarding primitive. Any arguments given to the patched binary are passed untouched to the target binary specified in `lpFile`.

### Chain Construction

When multiple copies of the binary are patched to point to each other:

Binary 1 (patched to launch Binary 2)
  ↓
Binary 2 (patched to launch Binary 3)
  ↓
Binary 3 (patched to launch Binary 4)
  ↓
Binary 4 (patched to launch cmd.exe)

Each link in the chain maintains the same `NULL` `lpParameters` behavior. Therefore, arguments provided to any binary in the chain will propagate through all remaining links.

### Why No Registry Changes Are Needed

Known LOLBAS techniques for write.exe require modifying registry keys under **HKCU\Software\Microsoft\Windows\CurrentVersion\App Paths\wordpad.exe** or **HKCU\Software\Classes\exefile**. These methods:

- Leave forensic evidence in the registry
- Affect system-wide behavior
- Require cleanup after execution

**LEPAD requires no registry modifications.** The only changes are to the copied binary files themselves, which can be deleted after use.

## Core Properties

### Parameter Propagation

When a binary in the chain executes, it launches the next binary and automatically passes forward any command-line arguments it received. This propagation continues link by link until the end of the chain.

### Chain Position Independence

The parameter inheritance mechanism works regardless of which binary in the chain receives the initial arguments. You may start the chain at:

- The first binary
- The second binary
- The fifth binary
- Any binary within the chain

The arguments will always propagate forward from that point to the end.

### Self-Healing Propagation

Once arguments enter the chain at any position, they will continue to travel through all subsequent links. The final binary will always execute with the provided arguments, irrespective of how many copies exist in the chain or where the arguments were introduced.

## Technical Flow

### Scenario A - Arguments at First Binary

Binary 1 (with arguments) → Binary 2 → Binary 3 → Binary 4 (executes with arguments)

### Scenario B - Arguments at Third Binary

Binary 1 (no arguments) → Binary 2 (no arguments) → Binary 3 (with arguments) → Binary 4 (executes with arguments)

### Scenario C - Arguments at Last Binary

Binary 1 → Binary 2 → Binary 3 → Binary 4 (with arguments, executes directly)

## Chain Behavior

| Chain Length | Arguments Enter At | Result              |
|--------------|-------------------|----------------------|
| 3 copies | Position 1 | Propagates through 2 and 3      |
| 5 copies | Position 3 | Propagates through 4 and 5      |
| 10 copies | Position 7 | Propagates through 8, 9 and 10 |
| Any length | Any position | Forward propagation to end  |

## Key Characteristics

- Arguments only travel forward, never backward
- Binaries before the injection point remain unaffected and do not execute
- The final binary always receives and executes the arguments
- No argument modification occurs during propagation
- The chain does not need to be started from the beginning
- No registry modifications required
- Only the patched binary files are changed

## Practical Implications

This behavior means an operator can:

1. Pre-position a chain on a target system
2. Later activate any binary within the chain with arguments
3. Have those arguments reach and execute at the end of the chain
4. Execute commands without starting from binary one
5. Leave no registry artifacts behind

The chain maintains its propagation capability regardless of entry point.

## Limitations

- Arguments cannot travel to binaries earlier in the chain
- Each binary must be capable of launching the next binary
- The chain length is fixed at deployment time and cannot be dynamically extended
- Requires write access to a directory (e.g., Desktop, Temp, AppData)
