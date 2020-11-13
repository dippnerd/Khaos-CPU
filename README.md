# 16-bit Harvard RISC Processor (Khaos)
### By Aaron Dippner
#### 4/10/08

This document is a complete explanation on the 16-bit Harvard RISC processor that I have designed. In this document I will give tables and detailed descriptions on the instruction set architecture, how it is laid out, the complete register set, how the processor works along with other details about the architecture. I will also give some basic examples of how to use the assembler that has been designed to go along with programming for this processor. By the end of this document, it should be easily understandable what makes this processor useful for general purpose programming.

One thing I have learned from designing this processor is the extreme level of detail and scrutiny one must undertake when designing such a thing. First and foremost one must know what have an idea in mind for what the processor will be used for. As I wanted mine to only be general purpose, I had to piece together a set of instructions that can undertake most general tasks, but if more complex tasks are needed, one can synthesize them with combining a series of the existing instructions in this architecture.

Early on I was pretty set on designing a 16-bit processor, based on the Reduced Instruction Set Computing (RISC) design. I chose RISC mostly due to personal beliefs in the efficiency and better design of the architecture, compared to Complex Instruction Set Computing. The major difference between the two is that RISC uses a limited number of simple instructions that can be completed in a very minimal amount of time, compared to CISC which uses a large set of complex instructions which perform multiple tasks but take much more time to complete. By using RISC, I feel not only can one keep the design small and simple, but also the performance potential is generally going to be higher.

After settling on the basics of RISC or CISC and the size of the processor, I had a challenge choosing between the Von Neumann architecture and the Harvard architecture. The difference here is, a classic Von Neumann architecture stores both the data and instructions in the same memory area, whereas the Harvard architecture separates the data and instructions. Ultimately I chose the Harvard architecture because I felt it would give better performance, as data and instructions can be accessed at the same time if necessary, plus it keeps both separate, so there is no worry of code modifying an instruction or piece of data in the same area. While designing the processor, I did try both methods back and forth multiple times, but ultimately I felt the Harvard architecture was the best design for this processor.

When designing this processor, I admittedly left behind all ideas of designing it around materials and parts that exist, because I do not plan to physical design this chip beyond just on the computer. Instead, I made this design rather ideal, by giving it memory modules exactly 16-bit bits in word size and address size, and over-using circuitry in a few areas just to see it actually work. In my next design, I plan to put more effort towards cleaning up unnecessary circuitry by reducing or combining certain areas, as well as focusing on a more realistic, practical application for design.

Once I had settled on all the needed architectural designs for the processor, I wrote down the Instruction Set Architecture Op-Code Layout, which explains the bit-positions for each operation code and how the following bits interact with the processor. I separated op-codes into four-bit pieces and operands typically into four-bit pieces to represent destination register, first and second source registers, or in branching cases, a bit-mask that represents where in the condition register it will compare the desired value. I used an eight-bit offset for instruction jumps within the operand for some operations as well.

Below is the table I designed to explain the op-codes. When a register is held within parenthesis, it means the value in that register will act as a pointer to an address in the data memory. On the Jump/Link and Jump/Return instructions, the word "Stack" is in parenthesis next to the register, signifying the programmer should preferably use the stack register for this register. In the branch instructions, the destination register is field has brackets around it, stating "Rs1 (CR) Mask." The value specified here will decode into one of 16 bits on the condition register and compare that bit with either zero or one (non-zero) depending on which instruction you choose. Offset fields specify an eight-bit value that the operation will jump to if true (in the case of branches). Do to the nature of the program counter, these values will always be incremented by one more beyond the offset (e.g. an offset of 4 will jump by 5).

#### Instruction Set Architecture Op-code Layout

Op |  1 |  2 |  3 |  4 |  5 |  6 |  7 |  8 |  9 | 10 | 11 | 12 | 13 | 14 | 15 | 16
-- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | --
LD |  0 |  0 |  0 |  0 | **<** | **Rd** | **-** | **>** |    |    |    |    |    |    |    |
LW |  1 |  0 |  0 |  0 | **<** | **Rd** | **-** | **>** |    |    |    |    | **<** | **(Rs2)** | **-** | **>**
SW |  0 |  1 |  0 |  0 |    |    |    |    | **<** | **Rs1** | **-** | **>**  | **<** | **(Rs2)** | **-** | **>**
JMP |  1 |  1 |  0 |  0 |   |   |   |   |  **<** |  **Offset** | **-**  | **-**  | **-**  | **-**  | **-**  | **>**
JL |  0 |  0 |  1 |  0 | **<** | **Rd (Stack)** | **-** | **>** |  **<** |  **Offset** | **-**  | **-**  | **-**  | **-**  | **-**  | **>**
JR | 1 |  0 |  1 |  0 |    |    |    |    | **<** | **Rs1 (Stack)** | **-** | **>**  |   |   |   |  
BRZ |  0 |  1 |  1 |  0 | **<** | **[Rs1 (CR)** | **Mask]** | **>** |  **<** |  **Offset** | **-**  | **-**  | **-**  | **-**  | **-**  | **>**
BRN |  1 |  1 |  1 |  0 | **<** | **[Rs1 (CR)** | **Mask]** | **>** |  **<** |  **Offset** | **-**  | **-**  | **-**  | **-**  | **-**  | **>**
ADD |  0 |  0 |  0 |  1 | **<** | **Rd** | **-** | **>** | **<** | **Rs1** | **-** | **>**  | **<** | **Rs2** | **-** | **>**
SUB |  1 |  0 |  0 |  1 | **<** | **Rd** | **-** | **>** | **<** | **Rs1** | **-** | **>**  | **<** | **Rs2** | **-** | **>**
AND |  0 |  1 |  0 |  1 | **<** | **Rd** | **-** | **>** | **<** | **Rs1** | **-** | **>**  | **<** | **Rs2** | **-** | **>**
OR |  1 |  1 |  0 |  1 | **<** | **Rd** | **-** | **>** | **<** | **Rs1** | **-** | **>**  | **<** | **Rs2** | **-** | **>**
XOR |  0 |  0 |  1 |  1 | **<** | **Rd** | **-** | **>** | **<** | **Rs1** | **-** | **>**  | **<** | **Rs2** | **-** | **>**
NOT |  1 |  0 |  1 |  1 | **<** | **Rd** | **-** | **>** | **<** | **Rs1** | **-** | **>**  |   |  |   |  
RTL |  0 |  1 |  1 |  1 | **<** | **Rd** | **-** | **>** | **<** | **Rs1** | **-** | **>**  |   |  |   |  
RTR |  1 |  1 |  1 |  1 | **<** | **Rd** | **-** | **>** | **<** | **Rs1** | **-** | **>**  |   |  |   |  


As can be seen, there are a total of 16 instructions listed. From the top of the list they are as follows:  Load Data word from address in program counter, Load Word from address in Rs2, Store word from Rs1 to address in Rs2, Jump to offset, Link program counter value to register (stack register preferred) and jump to offset, Jump/Return to value in register (stack register preferred) plus one, branch to offset if value in Condition Register bit specified by mask is zero (otherwise increment by one), branch to offset if value in Condition Register bit specified by mask is one (non-zero) (otherwise increment by one), add the values in Rs1 and Rs2 and place result in Rd, Subtract by same method, do logical AND by same method, do logical OR by same method, do logical XOR by same method, do logical NOT to value in Rs1 and place in Rd, Rotate value left and Rotate Value Right. When using the rotate value commands, if one merely desires to shift, the condition register needs to be set to zero first as to not include a carry with the rotate. Now that you understand the instruction operation-codes, below you will find a table with the register set and the mnemonics used to represent each one.


#### Register Set

Name | Mnemonic | Usage
---- | -------- | -----
Zero | $0 | Returns numberic value of zero
General Purpose | $1-$13 | Can be used for any general data type, no special purpose
Condition | $CR | Used for branching functions to store return condition
Stack Pointer | $SP | Used to store pointer for Jump/Link operations


The registers listed start from 0 and end at 15 for addressing, but the included assembler recognizes the mnemonic values accordingly. The Zero register is not actually a register, so nothing can be written to it, but when read, it always returns zero. General purpose registers 1-13 can store any type of data for any purpose except the branch commands, which only use the condition register. The condition register is where resulting values like carry, borrow, greater than, equal to and less than will be stored upon any arithmetic/logical operation automatically. The stack pointer does not have to be used but is just a preferred register for use when using Jump/Link and Jump/Return instructions. Below is a table explaining the bit-layout for the condition register values.


#### Condition Register Values

0 | 1 | 2 | 3 | 4 | 5-15
-- | -- | -- | -- | -- | -- 
Add Carry | Subtract Borrow | A > Zero | A = Zero | A < Zero | Unused


When using arithmetic/logical operations, depending on how they result, a bit or bits will be stored in the appropriately listed fields above. When using a branch command, the bit mask must specify one of these numbered fields to compare the bit stored in it with either zero or non-zero (one) accordingly. Fields 5-15 are unused unless the programmer writes a custom value to this register first.

 Now that we have a general understanding of the architecture, we can get into some more detailed explanations of how everything interacts. When both memory modules have been loaded with the program, it is necessary to press the reset just to make sure the processor is set back to its initial position and all registers are cleared. At this point it can be started, and will automatically fetch the instruction at address 0. This processor uses the standard Fetch-Decode-Execute-Store cycles for each instruction, but I have paired the clock to a counter so that on the rising and falling edge of the clock 2 cycles are completed, e.g. Fetch and Decode happen in one full cycle, and Execute and Store happen in the next. I did this to try optimizing efficiency in the processor without pipelining it (at least, not pipelining it yet). After 2 complete clock cycles, one instruction is completed and automatically fetches the next.

 Some things to keep in mind about this processor:

* While there is no instruction to copy one register to another, an easy thing to do is use the add command and specify the source register you want to copy, and the zero register as the second source, then the destination will receive a copy of the source register unchanged.
* The subtraction command is useful for comparing 2 values, but keep in mind if the highest most bit is used, comparing is signed and may result improperly according to two’s complement math.
* The Jump/Link command stores the program counter value, but doesn’t need to jump by more than 1, so if an offset of 0 is specified it will simply continue on to the next instruction, allowing the programmer to use that value for something else if desired.
* The jump return can jump to the value in any register, not just the stack register.
* The stack is technically not very deep, so if one wishes to go deeply into a subroutine, etc one must save the previous stack pointer value first.
* This processor is general purpose, so in theory it can accomplish virtually any task, but it may just take more complex steps to accomplish what is needed.
* While the comparator is signed, unsigned ALU operations can be done, but the condition register may not always be accurate.
* While there is no No-Operation instruction present, keep in mind an instruction of pure 0’s will result in no operation, because it is not possible to load any value into the zero register. Use this where needed to space a program out.


In the next section I have details on the assembler I have written for this processor. While the assembler has some bits that need corrected, I mostly wrote it just to save time so I may or may not improve all issues it has, as it works well enough. The assembler uses the mnemonics listed in the tables above and translates each line into a set of 1’s and 0’s, then arranges them appropriately and converts them to hexadecimal for the Logisim program’s memory modules to read properly. The assembler as of now does not account for a NOOP command, but I plan to implement one shortly. It also does not write properly to whatever format Logisim reads, so the texts files that come out have to still be entered manually into Logisim, however, I hope to fix that issue before too long as well. While this processor has no real practical use and likely will end here, I do debate writing another language that can be easier to program for and then translate into assembly, which can then be interpreted to binary. The current assembly also does not offer comments yet, but that will either go into the new language or else just not get added.
 
To use the assembler, it is as simple as running the program, which has three fields at the top. The first is the source file for your assembly; the next two are to specify destinations for the instruction and data memory files. Once all three are specified, the assemble button can be pushed. The block below will list what all was translated and say “Done” once it is finished. From here, you can just retrieve your two files and individually enter their data into the instruction and data memory modules of the processor.
 
I chose the name Khaos for this processor, as it represents the Greek deity by the same name. Khaos in Greek mythology was the nothingness which all else sprang from, the beginning so to speak. I felt this is an appropriate for my first processor, as it is the beginning for me, and will become what all else will come from. Future processors will always be based at least remotely from what I’ve learned from this processor. I also find it slightly ironic because typically people view the English word “chaos” as the opposite of order, yet this processor is practically complete order in and of itself. I was debating the name “Kronos” for the Greek god of eternal time, but decided that also would be ironic because they call this type of processor a finite state machine, which is not so eternal obviously.


## Assembly Examples

- When using the LD instruction in assembly, it should be written as such:
  - **LD $2, 8**
    - This method specifies Load Data “real number 8” into General Purpose Register 2. To specify a hexadecimal number it must be input as such:
- The Load Word assembly should be used as such:
  - **LW $2, $3**
    - The assembler will automatically set it up to Load Word found in the address in General Purpose Register 3 and load the Data into register 2.
- Store Word
  - **SW $2, $3**
    - This will store the value in General Purpose Register (GPR) 2 into the data memory, at the address stored in GPR 3.
- Jump
  - **JMP 8**
    - This will take the “real number 8” and add it to the contents of the Program Counter, then proceed to jump there.
- Jump/Link
  - **JL $SP, 8**
    - This will store the current PC value to Stack Pointer register, then jump to the offset value of 8, by adding it with the value of the PC. This can be done in Hex like others.
- Jump Return
  - **JR $SP**
    - This Jumps directly to the value stored in the Stack Pointer. This can also be used with other registers to add to program complexity, but the Stack Pointer is the preferred register to use for the Jump Return. It simply replaces the value in the PC to the value stored in the register specified.
- Branch If Zero/Non-Zero
  - **BRZ 3,8**
    - This branches to the offset value of 8 by adding it to the PC, provided the value stored in the Condition Register specifies a Zero result. This and the BRN can only use the Condition Register, so the programmer does not need to specify it. Only the type of Branch condition, and where to branch to. The specified value in the middle is a mask for which bit of the CR to test, 0-4. Refer to the CR value table for which bits mean what. Do note this command, BRZ, compares the bit with a Zero, BRN tests the value with a 1. The offset value can only be an 8-bit real number.
  - **BRN 2,5**
    - This is the same as the BRZ command, but instead it compares the value in the Condition Register, and if result specified is NOT Zero, the branch is made in the same way as the BRZ command.
- ALU Functions; This will only specify one or two ALU functions, as they all operate identically
  - **ADD $4, $2, $3**
    - This will ADD the values stored in GPRs 2 and 3, and store the results into GPR 4.
  - **AND $3, $2, $0**
    - This will do an AND compare of the GPR 2 with the GPR 0, which always returns 0, and stores the results into GPR 3.



### Images
* [Main](main.gif)
* [ALU](alu.gif)
* [Control Unit](controlunit.gif)
* [Program Counter](progcounter.gif)
* [Register File](regfile.gif)
