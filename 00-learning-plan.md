# Xcode 调试学习计划

制定日期：2026-11-06。

## 学习目标与已有基础

已有基础：使用 Xcode 调用栈、断点和基础 LLDB 命令，做过简单 mock；了解 Mach-O 和符号表、UIKit UI Debugging；知道 Instruments 的用途，但在大型 App 中使用效果有限；有一些汇编基础，ARM 汇编较少；会基础 Metal 资产调试；主要技术栈是 Objective-C 和 C/C++，Swift 有基础。

本次学习围绕 Objective-C，按需结合 C/C++。Swift、UIKit UI Debugging 和 Metal 不单独展开；遇到相关诊断问题时补充必要背景。

目标是建立从源码、编译产物、运行时行为到调试器观察和诊断证据的联系，能够：

- 在 crash、hang、内存压力、逻辑错误和调试器自身异常之间选择合适的调查路径。
- 从日志、报告和调用栈提出可验证的假设，而不是只停留在查看变量。
- 理解 Clang、LLVM、链接器、dyld、LLDB 之间的关系。
- 解释断点、表达式求值、栈回溯和变量显示的依据与局限。
- 在大型 App 中缩小问题范围，设计成本可控的验证实验。
- 区分 iOS 和 macOS 的共通原理与设备、权限、生命周期方面的差异。

## 如何使用官网文档

采用“按知识依赖安排学习顺序，按具体问题阅读官方文档”的方式。

Apple 的 [Diagnosing issues using crash reports and device logs](https://developer.apple.com/documentation/xcode/diagnosing-issues-using-crash-reports-and-device-logs) 入口按 Essentials、Crash reports、Device logs 组织，主要面向获取与分析诊断材料。它适合作为诊断参考，页面层级并不直接对应由浅入深的课程分级。

每个主题先带着一个具体问题读相关 Apple 文档，再用小实验连接机制。这个入口没有系统展开的编译、调试器和运行时知识，由对话补充；需要时引用 Apple 的其他资料、LLVM/Clang/LLDB 上游文档或源码。不能把“此入口没有展开”理解为“Apple 全部文档都没有”。

区分三类信息：官方明确说明的行为、特定版本中核验过的实现、帮助理解的简化模型。Apple 工具链与上游 LLVM 的版本和实现可能不同，实验时记录实际环境。

## 学习与文档约定

- 这份文档只记录学习计划，不提前写教程正文。
- 12 个 topic 文档目前保持空白；讲解、追问和官网之外的知识补充先在对话中进行。
- 用户明确通知整理或写入某个主题后，再将对应内容写入该 topic 文档；学完一节不自动写入。
- 整理时只保存已讲过的结论、关键证据、实验结果和仍待验证的问题，不把计划中的内容写成已验证事实。
- 示例代码和命令先在对话中给出；需要落地实验工程时，再按当时确定的范围实施。

## 学习顺序

| 阶段 | 对应 topic | 核心问题与补充方向 | 实践安排 | 阶段完成标准 |
|---|---|---|---|---|
| 1. 认识诊断证据 | [01 报告与设备日志](01-crash-reports-and-device-logs.md)、[02 异常与终止](02-mach-exceptions-signals-and-termination.md) | 报告能告诉我们什么？Mach exception、signal、OC exception、系统终止之间有什么联系？ | 阅读一份 OC crash report，区分事实与推测；比较附加调试器和独立运行时的可见现象。 | 能指出异常信息、相关线程和镜像信息，提出下一项取证动作；知道栈顶不一定是根因。 |
| 2. 从编译产物还原现场 | [03 编译、链接与加载](03-clang-compilation-linking-and-dyld.md)、[04 符号化](04-macho-dwarf-dsym-and-symbolication.md) | 一份 `.m` 文件怎样变成可运行代码？符号表、DWARF、dSYM 各自提供什么？ | 观察一个最小 OC 程序的编译与链接命令；匹配二进制与 dSYM 的 UUID，验证地址到符号的还原。 | 能解释构建链路和符号化需要的材料，区分缺少符号、缺少源码行信息与材料不匹配。 |
| 3. 理解调试器如何观察进程 | [05 LLDB 与断点](05-lldb-debugger-and-breakpoint-internals.md)、[06 OC 运行时与表达式](06-objc-runtime-arc-and-lldb-expressions.md) | LLDB 与 Clang 怎样协作？断点为何能暂停进程？读取变量和执行表达式有什么区别？ | 观察断点解析与命中；比较 `frame variable`、`p`、`po`、`expr`，检查表达式是否执行了目标进程中的代码。 | 能区分检查已有状态与执行代码，解释表达式可能产生的副作用，并选择合适的观察方式。 |
| 4. 理解调试失真与失败 | [07 ARM64、回溯与优化](07-arm64-stack-unwinding-and-optimized-code.md)、[08 调试不稳定](08-debugger-instability-and-troubleshooting.md) | 栈如何回溯？为什么优化后单步、行号和变量不符合直觉？“debug 不稳定”具体发生在哪一层？ | 对比同一段代码在不同优化配置下的表现；结合寄存器和少量 ARM64 指令解释调用过程；分析一个断点或表达式失败场景。 | 能从构建与符号、优化、进程状态、表达式求值、连接与权限等实际证据中缩小范围。 |
| 5. 定位内存与并发问题 | [09 内存错误](09-memory-bugs-zombies-and-sanitizers.md)、[10 并发、hang 与 watchdog](10-concurrency-deadlocks-hangs-and-watchdog.md) | 内存何时被破坏，何时才被发现？怎样区别竞争、死锁、长任务和系统超时终止？ | 比较 Zombies 与 ASan 对不同错误的证据；用一个阻塞案例练习多线程栈和重复采样；条件允许时使用 TSan。 | 能按错误类型选工具，从多线程证据提出因果假设，并用一次有针对性的实验验证。 |
| 6. 应用于大型 App | [11 内存压力与性能](11-memory-pressure-jetsam-and-performance.md)、[12 日志与定位方法](12-logging-and-systematic-debugging.md) | jetsam 与非法内存访问有什么不同？Instruments 数据很多时怎样找到有效证据？ | 阅读 jetsam 报告；围绕一个明确问题选择日志、signpost、采样或 Instruments 模板；完成一个综合定位练习。 | 能说明材料的证明范围，提出范围可控的调查方案，并用证据评估修复，而不只看问题是否暂时消失。 |

第 12 个 topic 的“证据 → 假设 → 实验 → 复核”方法从第一节就使用，最后再系统整理。已有 Mach-O、断点与 UIKit 知识按理解情况快速带过，将更多时间放在机制与实际定位上。

## 每次学习如何进行

建议每次 45–75 分钟，总计约 12–18 次；这只是可调整的节奏，不是固定排期。一个 topic 可以拆成多次，也可以在已有基础扎实时合并。

1. **一个具体问题**：先描述现象和现有证据，确认本节要回答什么。
2. **一小段官方材料**：只读与当前问题有关的段落，解释字段或工具行为的含义。
3. **机制补充**：用 OC/C++ 示例连接编译、运行时和操作系统知识，必要时展开 ARM64。
4. **一个小实验**：先预测结果，再观察；记录会影响结果的 Xcode、系统、架构和构建配置。
5. **复述与迁移**：用自己的话解释观察结果，再讨论放到大型 App 中应怎样取证。

没有可运行环境时，可以先做报告阅读或代码推演；推演结果需要明确标注，不代替实际运行验证。涉及 sanitizers、真机、模拟器或系统工具时，先核验当前平台支持和限制。

推进标准是能解释结果并选择下一步验证动作；无需记住全部命令。每次优先解决一个因果链，避免同时引入过多工具。

## 官方材料与补充资料

以下是阅读入口，不要求一次读完，也不作为额外作业清单。

| 用途 | 资料 |
|---|---|
| 获取 crash report 与诊断日志 | [Acquiring crash reports and diagnostic logs](https://developer.apple.com/documentation/xcode/acquiring-crash-reports-and-diagnostic-logs) |
| 阅读报告和字段 | [Analyzing a crash report](https://developer.apple.com/documentation/xcode/analyzing-a-crash-report)、[Examining the fields in a crash report](https://developer.apple.com/documentation/xcode/examining-the-fields-in-a-crash-report) |
| 异常类型和常见模式 | [Understanding the exception types in a crash report](https://developer.apple.com/documentation/xcode/understanding-the-exception-types-in-a-crash-report)、[Identifying the cause of common crashes](https://developer.apple.com/documentation/xcode/identifying-the-cause-of-common-crashes) |
| 符号化 | [Adding identifiable symbol names to a crash report](https://developer.apple.com/documentation/xcode/adding-identifiable-symbol-names-to-a-crash-report) |
| 按需读取原始报告格式 | [Interpreting the JSON format of a crash report](https://developer.apple.com/documentation/xcode/interpreting-the-json-format-of-a-crash-report) |
| 内存压力与系统终止 | [Identifying high-memory use with jetsam event reports](https://developer.apple.com/documentation/xcode/identifying-high-memory-use-with-jetsam-event-reports) |
| 系统日志 | [Logging](https://developer.apple.com/documentation/os/logging) |
| Clang 编译选项与工具链背景 | [Clang Compiler User’s Manual](https://clang.llvm.org/docs/UsersManual.html) |
| 编译器生成的调试信息 | [LLVM Source Level Debugging](https://llvm.org/docs/SourceLevelDebugging.html) |
| LLDB 使用与表达式基础 | [LLDB Tutorial](https://lldb.llvm.org/use/tutorial.html) |
| 内存错误与数据竞争检测 | [AddressSanitizer](https://clang.llvm.org/docs/AddressSanitizer.html)、[ThreadSanitizer](https://clang.llvm.org/docs/ThreadSanitizer.html) |

涉及 LLDB 断点实现、表达式编译、OC runtime、ARM64 ABI 和栈回溯的细节时，再定位对应版本的官方说明或源码，不预先要求通读源码。

## 第一节安排

从 topic 01 开始：**拿到一份 OC crash report，先看哪些信息，怎样决定接下来调查什么？**

先在对话中确认使用的 Xcode/系统版本和设备类型，再选用示例报告或用户已有的脱敏报告。练习提取异常信息、终止信息（如报告包含）、相关线程、调用栈和二进制镜像信息；把直接观察到的事实与需要验证的推测分开。

第一节的目标：能够用几句话说明“目前知道什么、还不知道什么、下一步应获取什么证据”。教程正文及官网之外的机制补充先在对话中进行，等待用户通知后再整理进 topic 文档。
