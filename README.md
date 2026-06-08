# LEPAD
### LEPAD (Legacy Parameter Descent) -  Command-line parameters are automatically inherited and propagated through a sequence of binaries where each executable calls the next. The arguments provided to any binary in the sequence are carried forward unchanged through all remaining binaries, culminating in execution of the final command.

<img width="936" height="807" alt="image" src="https://github.com/user-attachments/assets/dde40bb4-6a79-4eaa-9801-1c58a7e2d169" />
*LEPAD chain execution showing argument propagation from first to last binary.*


# LEPAD - Legacy Parameter Descent

## Description

Command-line parameters are automatically inherited and propagated through a sequence of binaries where each executable calls the next. The arguments provided to **any** binary in the sequence are carried forward unchanged through all subsequent binaries, culminating in execution of the final command.

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

| Chain Length | Arguments Enter At | Result |
|--------------|-------------------|--------|
| 3 copies | Position 1 | Propagates through 2 and 3 |
| 5 copies | Position 3 | Propagates through 4 and 5 |
| 10 copies | Position 7 | Propagates through 8, 9 and 10 |
| Any length | Any position | Forward propagation to end |

## Key Characteristics

- Arguments only travel forward, never backward
- Binaries before the injection point remain unaffected and do not execute
- The final binary always receives and executes the arguments
- No argument modification occurs during propagation
- The chain does not need to be started from the beginning

## Practical Implications

This behavior means an operator can:

1. Pre-position a chain on a target system
2. Later activate any binary within the chain with arguments
3. Have those arguments reach and execute at the end of the chain
4. Execute commands without starting from binary one

The chain maintains its propagation capability regardless of entry point.

## Limitations

- Arguments cannot travel to binaries earlier in the chain
- Each binary must be capable of launching the next binary
- The chain length is fixed at deployment time and cannot be dynamically extended
