---
name: ai-coding-safety-gate
description: Use for all AI coding work before code edits, patches, refactors, formatting, debugging, test-failure investigation, firmware or embedded troubleshooting, and any user report involving code problems, tests, waveform issues, data issues, jitter, unstable behavior, crashes, hangs, deadlocks, STM32, or hardware problems. This skill enforces discussion before code changes and requires hardware or wiring checks plus concrete symptom collection before modifying code for reported test, waveform, data, jitter, unstable, crash, hang, or dead-machine issues.
---

# AI Coding Safety Gate

Use this skill as a mandatory gate for AI coding. It applies before any code-changing action, especially in embedded, firmware, hardware-adjacent, STM32, measurement, test, waveform, or debugging tasks.

## Absolute Rules

1. Must discuss before changing code.
   - Before editing, patching, refactoring, formatting, regenerating, or overwriting files, explain the proposed change, target files or modules, reason, risk, and expected effect.
   - Wait for explicit user approval before changing files.
   - Valid approval examples: `go ahead`, `apply it`, `proceed`, `use that plan`, `start patching`, `please modify the code`.
   - Non-approval examples: `take a look`, `analyze it`, `what do you think`, `check it first`, `where might the problem be`, `keep investigating`. These do not authorize file changes.

2. Must evaluate change size before coding.
   - Before any substantial edit, estimate the expected code change size in lines.
   - If the planned change is more than 100 lines of code, or likely to touch multiple modules, create a clean checkpoint before editing.
   - Default checkpoint is local `git`: inspect status, discuss the checkpoint message, and commit the current state before the large change.
   - Use GitHub only when the user explicitly asks to push, open a PR, or use a remote workflow. Do not push by default.
   - If the workspace is not a git repository or has unsafe or unrelated changes, stop and ask the user how to checkpoint before editing.
   - If unsure whether the change exceeds 100 lines, treat it as `Large`.

3. Must not change code after a problem report until symptoms and hardware are checked.
   - If the user says code has a problem, a test failed, a waveform is wrong, data is wrong, there is jitter, unstable behavior, a crash, a hang, a dead machine, no response, abnormal output, or similar, do not patch first.
   - First ask the user to describe the concrete symptom and current reproduction conditions.
   - For hardware-adjacent or embedded work, require the user to check wiring and hardware before code edits.

4. Must separate investigation from mutation.
   - Allowed without approval: read code, inspect logs, explain code, propose hypotheses, create a checklist, design non-mutating tests, ask for waveform or data captures, request photos or wiring diagrams, or run read-only diagnostics.
   - Not allowed without approval: edit files, apply patches, run formatters that rewrite files, regenerate code, change configs, migrate schemas, or perform broad refactors.

## Large-Change Git Checkpoint Protocol

Before implementing any code change, classify its size:

- `Small`: under 100 lines of code and localized to one small area. Still discuss and wait for approval.
- `Large`: over 100 lines of code, multi-file, architecture-level, generated code, broad refactor, or risky firmware or embedded behavior change.

For `Large` changes, must do this before editing:

1. Summarize the planned change and expected line count.
2. Check whether the workspace is a git repository and whether the working tree has unrelated changes.
3. Default to a local `git` checkpoint, not a GitHub push.
4. Ask for or confirm the commit message if needed, then commit the current pre-change state.
5. Only after the checkpoint and explicit approval, perform the large edit.

If the user says `GitHub`, `push`, or `PR`, then GitHub may be used. Otherwise, do not push remote changes by default.

## Embedded High-Risk Area Gate

For embedded or firmware projects, treat these areas as high risk even if the line count is small:

- startup files, linker scripts, interrupt vector tables
- clock tree, PLL, reset, boot mode, watchdog
- GPIO pinmux or alternate-function setup
- ADC, DAC, DMA, or timer trigger chains
- ISR, callback, scheduler, RTOS priority, or critical section
- flash, EEPROM, or NVM write paths
- bootloader, upgrade path, or recovery path
- CubeMX or auto-generated boundary code

Before changing any high-risk area:

1. Explain why this area must be touched.
2. State the most likely failure mode, such as no boot, no download, dead machine, timing break, waveform corruption, or interrupt lockup.
3. Describe how to verify success on hardware.
4. Require explicit user approval even for a small patch.

## Required Ask-Back for Test or Hardware Symptoms

When the user reports a test problem, code problem, waveform issue, bad data, jitter, unstable behavior, a dead machine, no response, a hang, a crash, or similar, ask back before changing code:

- What exactly is the symptom: when does it happen, how long does it last, is it reproducible, and what are the reproduction steps?
- What does the waveform or data look like: frequency, amplitude, offset, noise, jitter range, screenshots, or raw captures?
- Has wiring been checked: power, common ground, input and output channels, probe ground clip, module supply, signal-source connection, load connection?
- Are the instrument settings correct: oscilloscope range, coupling, probe ratio, bandwidth limit, sample rate, trigger mode?
- Is the board state normal: reset, boot mode, crystal, debug cable, UART cable, supply voltage, current, chip temperature?
- What changed most recently: code, parameters, wiring, hardware modules, supply, or test environment?
- What was the last known good state: the last working commit, project backup, hex or bin, parameter snapshot, or wiring setup?
- For dead, rebooting, or frozen systems: is it likely a HardFault, watchdog reset, brownout, assert, ISR lockup, infinite loop, DMA callback issue, or can the reset reason or fault information be read?

For embedded or measurement systems, explicitly remind the user: first rule out wiring, power, ground, probe, range, and signal-source issues before code changes.
Do not hide symptoms by blindly adding delays, adding heavy filtering, disabling interrupts, disabling watchdogs, lowering sample rates, or weakening checks unless clearly labeled as a temporary diagnostic experiment.

## Code-Change Approval Protocol

When a code change might be needed, respond with this structure:

```text
I will not modify code yet. Please confirm:
1. Symptom and reproduction conditions: ...
2. Hardware and wiring checks: ...
3. Suspected software cause: ...
4. If hardware is confirmed clean, I propose changing: ...
5. Risk and expected effect: ...
6. Rollback point and verification method: what waveform, data, or log should change, and what failure would mean.

Please confirm whether I should modify the code using this plan.
```

Only after explicit approval, proceed to code modification. If another domain skill requires extra gates, such as `sequential-thinking` before code output, follow both skills.

## Priority With Other Skills

- This skill overrides eagerness to fix code quickly.
- Domain skills may propose algorithms or architecture, but this skill controls whether code can be changed.
- For embedded competition work, pair with `low-frequency-embedded-assistant`: first hardware and symptom checks, then firmware reasoning, then approved code edits.
