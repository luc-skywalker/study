
**每个 VMCS** 可以是 `ordinary VMCS`(**普通 VMCS**)或者`shadow VMCS`(**影子VMCS**).

**VMCS 的类型** 由VMCS区域中的 `shadow-VMCS indicator`(**shadow-VMCS 指示符**)确定(这是 VMCS 区域的前 4 个字节的第 31 位的值; 请参见表24-1):

* 0 表示普通的 VMCS,
* 1 表示 shadow VMCS.

shadow VMCS 仅在支持对 `VM-execution control` 控制字段的 "`VMCS shadowing`" 设置为1的处理器上受支持(请参见第`3.5.2`节).

`shadow VMCS` 与 `ordinary VMCS` 在以下两个方面有所不同:

* `ordinary VMCS` 可以用于 **VM-entry**, 但是 `shadow VMCS` **不能**. 当 `current VMCS` 是 shadow VMCS 时, 尝试执行VM进入失败(请参见第26.1节).

* VMREAD和VMWRITE指令可以在 **VMX non-root 操作模式！！！**中使用, 以访问 **shadow VMCS**, 但不能访问  **ordinary VMCS** . 此事实是由于以下原因造成的:

    * 如果`VM-execution control` 控制字段的 "`VMCS shadowing`" 为**0**, 则在 `VMX non-root` 模式(**guest模式**)中执行**VMREAD**和**VMWRITE**指令始终会导致`VM-exit`(请参见第25.1.3节).

    * `VM-execution control` 控制字段的 "`VMCS shadowing`" 为**1**, 则在 `VMX non-root`(**guest模式**)操作中执行VMREAD和VMWRITE指令可以访问VMCS链接指针引用的VMCS(请参见第30.3节).

    * `VM-execution control` 控制字段的 "`VMCS shadowing`" 为**1**, 则 VM entry 可确保 VMCS 链接指针引用的任何VMCS都是 **shadow VMCS** (请参阅第26.3.1.5节).

在**VMX root操作模式**(**host模式**)中, 可以使用**VMREAD**和**VMWRITE**指令访问**两种类型的 VMCS**.

软件**不应修改**处于 active 状态的VMCS的VMCS region中的`shadow-VMCS indicator`. 这样做可能会导致VMCS损坏(请参见24.11.1节). 在**修改**`shadow-VMCS indicator`之前, 软件应为VMCS执行**VMCLEAR**以确保其不是 active 状态.

