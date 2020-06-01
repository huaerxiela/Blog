---
title: ARMv8学习记录二
date: 2020-03-01 20:37:23
tags: 汇编
category: ARMv8汇编
---

<!-- TOC -->

- [基本知识](#基本知识)
    - [ARMv8寄存器](#armv8寄存器)
        - [AArch64通用寄存器](#aarch64通用寄存器)
        - [状态寄存器(SPSR)](#状态寄存器spsr)
    - [条件码](#条件码)
- [指令](#指令)
    - [数据处理指令](#数据处理指令)
        - [算术和逻辑运算](#算术和逻辑运算)
        - [乘法和除法指令](#乘法和除法指令)
        - [移位操作](#移位操作)
        - [位和字节操作指令](#位和字节操作指令)
        - [条件执行](#条件执行)
    - [内存访问指令](#内存访问指令)
        - [加载和存储指令](#加载和存储指令)
        - [寻址模式](#寻址模式)
    - [流程控制指令](#流程控制指令)
    - [System](#system)
- [参考](#参考)

<!-- /TOC -->

前面的环境搭建好了，下面主要学习 ARMv8 的一些基本知识和指令

# 基本知识
## ARMv8寄存器
寄存器名称描述

| 位宽   | 分类       |                |                 |
| ------ | ---------- | -------------- | --------------- |
| 32-bit | Wn（通用） | WZR（0寄存器） | WSP（堆栈指针） |
| 64-bit | Xn（通用） | XZR（0寄存器） | SP（堆栈指针）  |

### AArch64通用寄存器
AArch64通用寄存器共31个 `X0-X30` , 其中各寄存器的作用如下表。

| 寄存器    | 描述                   |
| --------- | ---------------------- |
| `X0` – `X7`   | 参数和返回值           |
| `X8` – `X18`  | 临时寄存器(X8一般保存结构体首地址) |
| `X19` – `X28` | 被调用者保存的寄存器   |
| `X29`       | 帧指针(`FP`)                 |
| `X30`       | 返回地址(`LR`)               |
| `SP`        | 栈指针                 |

### 状态寄存器(SPSR)
1. PSTATE at AArch64

![](ARMv8学习记录二/2020-03-09-15-57-08.png)

2. PSTATE at AArch32

![](ARMv8学习记录二/2020-03-09-16-00-41.png)


## 条件码

| Code | Encoding | Meaning (when set by CMP)                            | Meaning (when set by FCMP)                              | Condition flags      |
| ---- | -------- | ---------------------------------------------------- | ------------------------------------------------------- | -------------------- |
| `EQ`   | 0b0000   | Equal to.                                            | Equal to.                                               | `Z = 1`                 |
| `NE`   | 0b0001   | Not equal to.                                        | Unordered, or not equal to.                             | `Z = 0`                |
| `CS`   | 0b0010   | Carry set (identical to HS).                         | Greater than, equal to, or unordered (identical to HS). | `C = 1`                |
| `HS`   | 0b0010   | Greater than, equal to (unsigned) (identical to CS). | Greater than, equal to, or unordered (identical to CS). | `C = 1`                |
| `CC`   | 0b0011   | Carry clear (identical to LO).                       | Less than (identical to LO).                            | `C = 0`                |
| `LO`   | 0b0011   | Unsigned less than (identical to CC).                | Less than (identical to CC).                            | `C = 0`                |
| `MI`   | 0b0100   | Minus, Negative.                                     | Less than.                                              | `N = 1`                |
| `PL`   | 0b0101   | Positive or zero.                                    | Greater than, equal to, or unordered.                   | `N = 0`                |
| `VS`   | 0b0110   | Signed overflow.                                     | Unordered. (At least one argument was NaN).             | `V = 1`                |
| `VC`   | 0b0111   | No signed overflow.                                  | Not unordered. (No argument was NaN).                   | `V = 0`                |
| `HI`   | 0b1000   | Greater than (unsigned).                             | Greater than or unordered.                              | `(C = 1) && (Z = 0)`   |
| `LS`   | 0b1001   | Less than or equal to (unsigned).                    | Less than or equal to.                                  | `(C = 0) || (Z = 1)` |
| `GE`   | 0b1010   | Greater than or equal to (signed).                   | Greater than or equal to.                               | `N==V`                 |
| `LT`   | 0b1011   | Less than (signed).                                  | Less than or unordered.                                 | `N!=V`                 |
| `GT`   | 0b1100   | Greater than (signed).                               | Greater than.                                           | `(Z==0) && (N==V)`     |
| `LE`   | 0b1101   | Less than or equal to (signed).                      | Less than, equal to or unordered.                       | `(Z==1) || (N!=V)`   |
| `AL`   | 0b1110   | Always executed.                                     | Default. Always executed.                               | `Any`                  |
| `NV`   | 0b1111   | Always executed.                                     | Always executed.                                        | `Any`                  |


# 指令

## 数据处理指令

### 算术和逻辑运算

| Type | Instructions                 |
| ---- | ---------------------------- |
| 算术 | `ADD`, `SUB`, `ADC`, `SBC`, `NEG`, `RSB` |
| 逻辑 | `AND`, `BIC`, `ORR`, `ORN`, `EOR`, `EON` |
| 比较 | `CMP`, `CMN`, `TST`                |
| 移动 | `MOV`, `MVN`                     |

`ADC` 和 `SBC` 运算执行加法和减法，并且将进位条件标志位用作输入。
```
ADC{S}: Rd = Rn + Rm + C
SBC{S}: Rd = Rn - Rm - 1 + C
```

例子
```
CMN w0, #-3 // 等价CMP W0, #3 即 x0 == 3
ADD W0, W1, W2, LSL #3 // W0 = W1 + (W2 << 3)
SUBS X0, X4, X3, ASR #2 // X0 = X4 - (X3 >> 2), set flags
MOV X0, X1 // Copy X1 to X0
CMP W3, W4 // Set flags based on W3 - W4
ADD W0, W5, #27 // W0 = W5 + 27
MNEG w0, w1, w2   //w0 = w0 - w1 * w2
NEG w0, w1  //w0 = -w1
// x1 = x0 % 16
and     x1, x0, 15

// x0 == 0?
tst     x0, x0
beq     zero

// x0 = 0
eor     x0, x0, x0
```

### 乘法和除法指令

| Opcode                | Description                           |
| --------------------- | ------------------------------------- |
| **Multiply instructions**                                       |
| `MADD`                  | Multiply add                          |
| `MNEG`                  | Multiply negate                       |
| `MSUB`                  | Multiply subtract                     |
| `MUL`                   | Multiply                              |
| `SMADDL`                | Signed multiply-add long              |
| `SMNEGL`                | Signed multiply-negate long           |
| `SMSUBL`                | Signed multiply-subtract long         |
| `SMULH`                 | Signed multiply returning high half   |
| `SMULL`                 | Signed multiply long                  |
| `UMADDL`                | Unsigned multiply-add long            |
| `UMNEGL`                | Unsigned multiply-negate long         |
| `UMSUBL`                | Unsigned multiply-subtract long       |
| `UMULH`                 | Unsigned multiply returning high half |
| `UMULL`                 | Unsigned multiply long                |
| **Divide instructions**                                         |
| `SDIV`                  | Signed divide                         |
| `UDIV`                  | Unsigned divide                       |

```
MUL X0, X1, X2  // X0 = X1 * X2
MNEG X0, X1, X2  // X0 = -(X1 * X2)

UDIV W0, W1, W2  // W0 = W1 / W2 (unsigned, 32-bit divide)
SDIV X0, X1, X2  // X0 = X1 / X2 (signed, 64-bit divide)
```

### 移位操作

| Instruction | Description            |
| ----------- | ---------------------- |
| **Shift**                              |
| `ASR`         | Arithmetic shift right |
| `LSL`         | Logical shift left     |
| `LSR`         | Logical shift right    |
| `ROR`         | Rotate right           |
| **Move**                               |
| `MOV`         | Move                   |
| `MVN`         | Bitwise NOT            |

```
// x1 = x0 / 8
lsr     x1, x0, 3

// x1 = x0 * 4
lsl     x1, x0, 2
```

![](ARMv8学习记录二/2020-03-09-19-15-20.png)


### 位和字节操作指令

| Mnemonic       | Operands             | Instruction                                                                                                                                                                                                                                                    |
| -------------- | -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `BFI`            | `Rd, Rn, #lsb, #width` | Bitfield Insert copies any number of low-order bits from a source register into the same number of adjacent bits at any position in the destination register, leaving other bits unchanged.                                                                    |
| `BFM`            | `Rd, Rn, #immr, #imms` | Bitfield Move copies any number of low-order bits from a source register into the same number of adjacent bits at any position in the destination register, leaving other bits unchanged.                                                                      |
| `BFXIL`          | `Rd, Rn, #lsb, #width` | Bitfield extract and insert at low end copies any number of low-order bits from a source register into the same number of adjacent bits at the low end in the destination register, leaving other bits unchanged.                                              |
| `CLS`            | `Rd, Rn`               | Count leading sign bits.                                                                                                                                                                                                                                       |
| `CLZ`            | `Rd, Rn`               | Count leading zero bits.                                                                                                                                                                                                                                       |
| `EXTR`           | `Rd, Rn, Rm, #lsb`     | Extract register extracts a register from a pair of registers.                                                                                                                                                                                                 |
| `RBIT`           | `Rd, Rn`               | Reverse Bits reverses the bit order in a register.                                                                                                                                                                                                             |
| `REV16`          | `Rd, Rn`               | Reverse bytes in 16-bit halfwords reverses the byte order in each 16-bit halfword of a register.                                                                                                                                                               |
| `REV32`          | `Rd, Rn`               | Reverse bytes in 32-bit words reverses the byte order in each 32-bit word of a register.                                                                                                                                                                       |
| `REV64`          | `Rd, Rn`               | Reverse Bytes reverses the byte order in a 64-bit general-purpose register.                                                                                                                                                                                    |
| `SBFIZ`          | `Rd, Rn, #lsb, #width` | Signed Bitfield Insert in Zero zeroes the destination register and copies any number of contiguous bits from a source register into any position in the destination register, sign-extending the most significant bit of the transferred value. Alias of SBFM. |
| `SBFM`           | `Wd, Wn, #immr, #imms` | Signed Bitfield Move copies any number of low-order bits from a source register into the same number of adjacent bits at any position in the destination register, shifting in copies of the sign bit in the upper bits and zeros in the lower bits.           |
| `SBFX`           | `Rd, Rn, #lsb, #width` | Signed Bitfield Extract extracts any number of adjacent bits at any position from a register, sign-extends them to the size of the register, and writes the result to the destination register.                                                                |
| `{S,U}XT{B,H,W}` | `Rd, Rn`               | (S)igned/(U)nsigned eXtend (B)yte/(H)alfword/(W)ord extracts an 8-bit,16-bit or 32-bit value from a register, zero-extends it to the size of the register, and writes the result to the destination register. Alias of UBFM.                                   |

![](ARMv8学习记录二/2020-03-09-21-46-28.png)

![](ARMv8学习记录二/2020-03-09-21-48-58.png)

![](ARMv8学习记录二/2020-03-09-21-49-14.png)


### 条件执行

| Mnemonic         | Operands              | Instruction                                                                                                                                                                                                                                |
| ---------------- | --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `CCMN` (immediate) | `Rn, #imm, #nzcv, cond` | Conditional Compare Negative (immediate) sets the value of the condition flags to the result of the comparison of a register value and a negated immediate value if the condition is TRUE, and an immediate value otherwise.               |
| `CCMN` (register)  | `Rn, Rm, #nzcv, cond`   | Conditional Compare Negative (register) sets the value of the condition flags to the result of the comparison of a register value and the inverse of another register value if the condition is TRUE, and an immediate value otherwise.    |
| `CCMP` (immediate) | `Rn, #imm, #nzcv, cond` | Conditional Compare (immediate) sets the value of the condition flags to the result of the comparison of a register value and an immediate value if the condition is TRUE, and an immediate value otherwise.                               |
| `CCMP` (register)  | `Rn, Rm, #nzcv, cond`   | Conditional Compare (register) sets the value of the condition flags to the result of the comparison of two registers if the condition is TRUE, and an immediate value otherwise.                                                          |
| `CSEL`             | `Rd, Rn, Rm, cond`      | Conditional Select returns, in the destination register, the value of the first source register if the condition is TRUE, and otherwise returns the value of the second source register.                                                   |
| `CSINC`            | `Rd, Rn, Rm, cond`      | Conditional Select Increment returns, in the destination register, the value of the first source register if the condition is TRUE, and otherwise returns the value of the second source register incremented by 1. Used by CINC and CSET. |
| `CSINV`            | `Rd, Rn, Rm, cond`      | Conditional Select Invert returns, in the destination register, the value of the first source register if the condition is TRUE, and otherwise returns the bitwise inversion value of the second source register. Used by CINV and CSETM.  |
| `CSNEG`            | `Rd, Rn, Rm, cond`      | Conditional Select Negation returns, in the destination register, the value of the first source register if the condition is TRUE, and otherwise returns the negated value of the second source register. Used by CNEG.                    |
| `CSET`             | `Rd, cond`              | Conditional Set sets the destination register to 1 if the condition is TRUE, and otherwise sets it to 0.                                                                                                                                   |
| `CSETM`            | `Rd, cond`              | Conditional Set Mask sets all bits of the destination register to 1 if the condition is TRUE, and otherwise sets all bits to 0.                                                                                                            |
| `CINC`             | `Rd, Rn, cond`          | Conditional Increment returns, in the destination register, the value of the source register incremented by 1 if the condition is TRUE, and otherwise returns the value of the source register.                                            |
| `CINV`             | `Rd, Rn, cond`          | Conditional Invert returns, in the destination register, the bitwise inversion of the value of the source register if the condition is TRUE, and otherwise returns the value of the source register.                                       |
| `CNEG`             | `Rd, Rn, cond`          | Conditional Negate returns, in the destination register, the negated value of the source register if the condition is TRUE, and otherwise returns the value of the source register.                                                        |

```
CSINC X0, X1, X0, NE  // Set the return register X0 to X1 if Zero flag clear, else increment X0
CINC X0, X0, LS  // If less than or same (LS) then X0 = X0 + 1
CSET W0, EQ  // If the previous comparison was equal (Z=1) then W0 = 1, 
// else W0 = 0
CSETM X0, NE  // If not equal then X0 = -1, else X0 = 0 

//if (i == 0) r = r + 2; else r = r - 1;
CMP w0, #0  // if (i == 0)
SUB w2, w1, #1  // r = r - 1
ADD w1, w1, #2  // r = r + 2
CSEL w1, w1, w2, EQ  // select between the two results
```

## 内存访问指令
### 加载和存储指令
| Mnemonic                | Operands                                    | Instruction                                                                                                                                                                                                                                     |
| ----------------------- | ------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `LDR(B|H|SB|SH|SW)` | `Wt, [Xn|SP], #simm`                       | Load Register (immediate) loads a word or doubleword from memory and writes it to a register. The address that is used for the load is calculated from a base register and an immediate offset.                                                 |
| `LD(B|H|SB|SH|SW)`  | `Wt, [Xn|SP, (Wm|Xm){, extend {amount}}]` | Load Register (register) calculates an address from a base register value and an offset register value, loads a byte/half-word/word from memory, and writes it to a register. The offset register value can optionally be shifted and extended. |
| `STR(B|H|SB|SH|SW)` | `Wt, [Xn|SP], #simm`                       | Store Register (immediate) stores a word or a doubleword from a register to memory. The address that is used for the store is calculated from a base register and an immediate offset.                                                          |
| `STR(B|H|SB|SH|SW)` | `Wt, [Xn|SP, (Wm|Xm){, extend {amount}}]` | Store Register (immediate) stores a word or a doubleword from a register to memory. The address that is used for the store is calculated from a base register and an immediate offset.                                                          |
| `LDP`                   | `Wt1, Wt2, [Xn|SP], #imm`                  | Load Pair of Registers calculates an address from a base register value and an immediate offset, loads two 32-bit words or two 64-bit doublewords from memory, and writes them to two registers.                                                |
| `STP`                   | `Wt1, Wt2, [Xn|SP], #imm`                  | Store Pair of Registers calculates an address from a base register value and an immediate offset, and stores two 32-bit words or two 64-bit doublewords to the calculated address, from two registers                                           |

```
// load a byte from x1
ldrb    w0, [x1]

// load a signed byte from x1
ldrsb   w0, [x1]

// store a 32-bit word to address in x1
str     w0, [x1]

// load two 32-bit words from stack, advance sp by 8
ldp     w0, w1, [sp], 8

// store two 64-bit words at [sp-96] and subtract 96 from sp 
stp     x0, x1, [sp, -96]!

// load 32-bit immediate from literal pool
ldr     w0, =0x12345678
```

### 寻址模式
| Addressing Mode                | Immediate       | Register                | Extended Register              |
| ------------------------------ | --------------- | ----------------------- | ------------------------------ |
| Base register only (no offset) | `[base{, 0}]`   |                         |                                |
| Base plus offset               | `[base{, imm}]` | `[base, Xm{, LSL imm}]` | `[base, Wm, (S|U)XTW {#imm}]` |
| Pre-indexed                    | `[base, imm]!`  |                         |                                |
| Post-indexed                   | `[base], imm`   | `[base], Xm a`          |                                |
| Literal (PC-relative)          | `label`         |                         |                                |

```
// load a byte from x1
ldrb   w0, [x1]

// load a half-word from x1
ldrh   w0, [x1]

// load a word from x1
ldr    w0, [x1]

// load a doubleword from x1
ldr    x0, [x1]

// load a byte from x1 plus 1
ldrb   w0, [x1, 1]

// load a half-word from x1 plus 2
ldrh   w0, [x1, 2]

// load a word from x1 plus 4
ldr    w0, [x1, 4]

// load a doubleword from x1 plus 8
ldr    x0, [x1, 8]

// load a doubleword from x1 using x2 as index
// w2 is multiplied by 8
ldr    x0, [x1, x2, lsl 3]

// load a doubleword from x1 using w2 as index
// w2 is zero-extended and multiplied by 8
ldr    x0, [x1, w2, uxtw 3]
// load a byte from x1 plus 1, then advance x1 by 1
ldrb   w0, [x1, 1]!

// load a half-word from x1 plus 2, then advance x1 by 2
ldrh   w0, [x1, 2]!

// load a word from x1 plus 4, then advance x1 by 4
ldr    w0, [x1, 4]!

// load a doubleword from x1 plus 8, then advance x1 by 8
ldr    x0, [x1, 8]!
// load a byte from x1, then advance x1 by 1
ldrb   w0, [x1], 1

// load a half-word from x1, then advance x1 by 2
ldrh   w0, [x1], 2

// load a word from x1, then advance x1 by 4
ldr    w0, [x1], 4

// load a doubleword from x1, then advance x1 by 8
ldr    x0, [x1], 8

  // load address of label
adr    x0, label

// load address of label
adrp   x0, label
```

## 流程控制指令
| Mnemonic | Operands        | Instruction                                                                                                                                                                                                                                                                                                          |
| -------- | --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `B`        | `label`           | Branch causes an unconditional branch to a label at a PC-relative offset, with a hint that this is not a subroutine call or return.                                                                                                                                                                                  |
| `B.cond`   | `label`           | Branch conditionally to a label at a PC-relative offset, with a hint that this is not a subroutine call or return.                                                                                                                                                                                                   |
| `BL`       | `label`           | Branch with Link branches to a PC-relative offset, setting the register X30 to PC+4. It provides a hint that this is a subroutine call.                                                                                                                                                                              |
| `BLR`      | `Xn`              | Branch with Link to Register calls a subroutine at an address in a register, setting register X30 to PC+4.                                                                                                                                                                                                           |
| `BR`       | `Xn`              | Branch to Register branches unconditionally to an address in a register, with a hint that this is not a subroutine return.                                                                                                                                                                                           |
| `CBNZ`     | `Rn, label`       | Compare and Branch on Nonzero compares the value in a register with zero, and conditionally branches to a label at a PC-relative offset if the comparison is not equal. It provides a hint that this is not a subroutine call or return. This instruction does not affect the condition flags.                       |
| `CBZ`      | `Rn, label`       | Compare and Branch on Zero compares the value in a register with zero, and conditionally branches to a label at a PC-relative offset if the comparison is equal. It provides a hint that this is not a subroutine call or return. This instruction does not affect condition flags.                                  |
| `RET`      | `Xn`              | Return from subroutine branches unconditionally to an address in a register, with a hint that this is a subroutine return.                                                                                                                                                                                           |
| `TBNZ`     | `Rn, #imm, label` | Test bit and Branch if Nonzero compares the value of a bit in a general-purpose register with zero, and conditionally branches to a label at a PC-relative offset if the comparison is not equal. It provides a hint that this is not a subroutine call or return. This instruction does not affect condition flags. |
| `TBZ`      | `Rn, #imm, label` | Test bit and Branch if Zero compares the value of a test bit with zero, and conditionally branches to a label at a PC-relative offset if the comparison is equal. It provides a hint that this is not a subroutine call or return. This instruction does not affect condition flags.                                 |

## System
| Mnemonic | Instruction                                                                                                                                           |
| -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `MSR`      | Move general-purpose register to System Register allows the PE to write an AArch64 System register from a general-purpose register.                   |
| `MRS`      | Move System Register allows the PE to read an AArch64 System register into a general-purpose register.                                                |
| `SVC`      | Supervisor Call causes an exception to be taken to EL1.                                                                                               |
| `NOP`      | No Operation does nothing, other than advance the value of the program counter by 4. This instruction can be used for instruction alignment purposes. |

# 参考
[A Guide to ARM64 / AArch64 Assembly on Linux with Shellcodes and Cryptography](https://modexp.wordpress.com/2018/10/30/arm64-assembly/)

[DEN0024A_v8_architecture_PG.pdf](https://static.docs.arm.com/den0024/a/DEN0024A_v8_architecture_PG.pdf)

















