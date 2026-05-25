---
name: ai-coding-safety-gate
description: Use for all AI coding work before code edits, patches, refactors, formatting, debugging, test-failure investigation, firmware or embedded troubleshooting, and any user report involving code problems, tests, waveform issues, data issues, jitter, unstable behavior, crashes, hangs, deadlocks, STM32 or hardware problems. This skill enforces discussion before code changes and requires hardware or wiring checks plus concrete symptom collection before modifying code for reported test, waveform, data, jitter, unstable, crash, hang, or dead-machine issues.
---

# AI Coding Safety Gate

Use this skill as a mandatory gate for AI coding. It is global and applies before any code-changing action, especially in embedded, firmware, hardware-adjacent, STM32, measurement, test, waveform, or debugging tasks.

## Absolute Rules

1. Must discuss before changing code.
   - Before editing, patching, refactoring, formatting, regenerating, or overwriting files, explain the proposed change, target files or modules, reason, risk, and expected effect.
   - Wait for explicit user approval before changing files.
   - Valid approval examples: `可以改`, `按这个方案改`, `执行修改`, `开始 patch`, `go ahead`, `apply it`.
   - Non-approval examples: `继续看看`, `分析一下`, `你觉得呢`, `帮我看看`, `哪里可能有问题`, `先查一下`. These do not authorize file changes.

2. Must evaluate change size before coding.
   - Before any substantial edit, estimate the expected code change size in lines.
   - If the planned change is more than 100 lines of code, or likely to touch multiple modules, create a clean checkpoint before editing.
   - Default checkpoint is local `git`: inspect status, discuss the checkpoint message, and commit the current state before the large change.
   - Use GitHub only when the user explicitly asks to push/open a PR or when the repository workflow clearly requires it. Do not push by default.
   - If the workspace is not a git repository or has unsafe/unrelated changes, stop and ask the user how to checkpoint before editing.
   - If unsure whether the change exceeds 100 lines, treat it as `Large`.

3. Must not change code after a problem report until symptoms and hardware are checked.
   - If the user says code has a problem, test failed, waveform is wrong, data is wrong, jitter, unstable, crash, hang, dead machine, no response, abnormal output, or similar, do not patch first.
   - First ask the user to describe the concrete symptom and current reproduction conditions.
   - For hardware-adjacent or embedded work, require the user to check wiring and hardware before code edits.

4. Must separate investigation from mutation.
   - Allowed without approval: read code, inspect logs, explain code, propose hypotheses, create a checklist, design non-mutating tests, ask for waveform/data/photos/wiring diagrams, or run read-only diagnostics.
   - Not allowed without approval: edit files, apply patches, run formatters that rewrite files, regenerate code, change configs, migrate schemas, or perform broad refactors.


## Large-Change Git Checkpoint Protocol

Before implementing any code change, classify its size:

- `Small`: under 100 lines of code and localized to one small area. Still discuss and wait for approval.
- `Large`: over 100 lines of code, multi-file, architecture-level, generated code, broad refactor, or risky firmware/embedded behavior change.

For `Large` changes, must do this before editing:

1. Summarize the planned change and expected line count.
2. Check whether the workspace is a git repository and whether the working tree has unrelated changes.
3. Default to local `git` checkpoint, not GitHub push.
4. Ask/confirm the commit message if needed, then commit the current pre-change state.
5. Only after the checkpoint and explicit approval, perform the large edit.

If the user says `GitHub`, `push`, or `PR`, then GitHub may be used. Otherwise, do not push remote changes by default.

## Embedded High-Risk Area Gate

For embedded or firmware projects, treat these areas as high risk even if the line count is small:

- startup files, linker scripts, interrupt vector tables
- clock tree, PLL, reset, boot mode, watchdog
- GPIO pinmux / alternate-function setup
- ADC / DAC / DMA / Timer trigger chains
- ISR, callback, scheduler, RTOS priority, critical section
- flash / EEPROM / NVM write paths
- bootloader, upgrade path, recovery path
- CubeMX or auto-generated boundary code

Before changing any high-risk area:

1. Explain why this area must be touched.
2. State the most likely failure mode, such as no boot, no download, dead machine, timing break, waveform corruption, or interrupt lockup.
3. Describe how to verify success on hardware.
4. Require explicit user approval even for a small patch.

## Required Ask-Back for Test or Hardware Symptoms

When the user reports `测试有问题`, `代码有问题`, `波形有问题`, `数据不对`, `抖动`, `不稳`, `死机`, `没反应`, `hang`, `crash`, or similar, ask back before changing code:

- 具体现象是什么：什么时候出现、持续多久、是否可复现、复现步骤是什么？
- 波形或数据是什么样：频率、幅值、偏移、噪声、抖动范围、截图或原始数据？
- 接线是否检查：电源、地线共地、输入输出通道、探头地夹、模块供电、信号源连接、负载连接？
- 仪器设置是否正确：示波器量程、耦合方式、探头倍率、带宽限制、采样率、触发方式？
- 板卡状态是否正常：复位、BOOT、晶振、下载线、串口线、供电电压、电流、芯片温度？
- 最近改过什么：代码、参数、接线、硬件模块、供电、测试环境？
- 上一次正常版本是什么：最近一次正常的 commit、工程备份、hex/bin、参数快照或接线状态？
- 如果是死机/重启/卡死：是否能判断是 HardFault、watchdog reset、brownout、assert、卡在 ISR、卡在 while、DMA callback 异常，或是否能读取 reset reason / fault 信息？

For embedded or measurement systems, explicitly remind the user: first rule out wiring, power, ground, probe, range, and signal-source issues before code changes.
Do not hide symptoms by blindly adding delays, adding heavy filtering, disabling interrupts, disabling watchdogs, lowering sample rates, or weakening checks unless clearly labeled as a temporary diagnostic experiment.

## Code-Change Approval Protocol

When a code change might be needed, respond with this structure:

```text
我先不改代码。建议先确认：
1. 现象与复现条件：...
2. 硬件/接线检查：...
3. 我怀疑的软件原因：...
4. 如果你确认硬件无误，我建议改：...
5. 风险与预期效果：...
6. 回滚点与验证方式：改后应该看什么波形/数据/日志，如果失败意味着什么？

请确认是否按这个方案修改代码。
```

Only after explicit approval, proceed to code modification. If another domain skill requires extra gates, such as `sequential-thinking` before code output, follow both skills.

## Priority With Other Skills

- This skill overrides eagerness to fix code quickly.
- Domain skills may propose algorithms or architecture, but this skill controls whether code can be changed.
- For embedded competition work, pair with `low-frequency-embedded-assistant`: first hardware and symptom check, then firmware reasoning, then approved code edits.
