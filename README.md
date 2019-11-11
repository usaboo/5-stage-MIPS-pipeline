# 5-stage-MIPS-pipeline
This repository gives the user a CPP code that implements the 5 stage MIPS pipeline

This coded simulator supports a subset of the MIPS instruction set that models the execution of each instruction with cycle accuracy.

Our MIPS pipeline has the following 5 stages:

1. Fetch (IF): fetches an instruction from instruction memory. Updates PC.
2. Decode (ID/RF): reads from the register RF and generates control signals required in
subsequent stages. In addition, branches are resolved in this stage by checking for the
branch condition and computing the effective address.
3. Execute (EX): performs an ALU operation.
4. Memory (MEM): loads or stores a 32-bit word from data memory.
5. Writeback (WB): writes back data to the RF.


Each pipeline stages takes inputs from flip-flops. The input flip-flops for each pipeline stage are described in the tables below.

￼
IF Stage Input Flip-Flops
Flip-Flop Name                             Bit-width                          Functionality
PC                                            32                              Current value of PC
nop                                            1                              If set, IF stage performs a nop

ID/RF Stage Input Flip-Flops
Flip-Flop Name                             Bit-width                          Functionality
Instr                                         32                              32-bit instruction read from IMEM
nop                                            1                              If set, ID/RF stage performs a nop

EX Stage Input Flip-Flops
Flip-Flop Name                             Bit-width                          Functionality
￼￼￼Read_data1, Read_data2                        32                              32-bit data values read from RF
￼￼￼Imm                                           16                              ￼￼￼16-bit immediate for I-Type instructions. Don’t care for R-type instructions
￼￼￼Rs, Rt                                         5                              Addresses of source registers rs, rt. Note that these are defined for both R-type and I-type instructions
￼￼￼Wrt_reg_addr                                   5                              Address of the instruction’s destination register. Don’t care if the instruction does not update RF
￼￼￼alu_op                                         1                              Set for addu, lw, sw; unset for subu
￼￼￼is_I_type                                      ￼￼￼￼1                              Set if the instruction is an i-type instruction
￼￼￼wrt_enable                                     ￼￼￼￼￼1                              Set if the instruction updates RF
￼￼￼rd_mem, wrt_mem                                1                              rd_mem is set for lw instruction and wrt_mem for sw instructions
￼￼￼nop                                            1                              If set, EX stage performs a nop
￼

MEM Stage Input Flip-Flops
￼￼￼Flip-Flop Name                            ￼￼￼￼￼￼￼Bit-width                           Functionality
￼￼￼ALUresult                                     32                              32-bit ALU result. Don’t care for beq instructions
￼￼￼Store_data                                    32                              32-bit value to be stored in DMEM for sw instruction. Don’t care otherwise
￼￼￼Rs, Rt                                         5                              ￼￼￼Addresses of source registers rs, rt. Note that these are defined for both R-type and I-type instructions
￼￼￼Wrt_reg_addr                                   ￼￼￼￼5                              Address of the instruction’s destination register. Don’t care if the instruction does not update RF
￼￼￼wrt_enable                                     1                              Set if the instruction updates RF
￼￼￼rd_mem, wrt_mem                                ￼￼￼￼1                              rd_mem is set for lw instruction and wrt_mem for sw instructions
￼￼￼nop                                            ￼￼￼￼￼￼1                              If set, MEM stage performs a nop
￼￼

WB Stage Input Flip-Flops
￼￼￼Flip-Flop Name                            ￼￼￼￼￼￼￼Bit-width￼￼￼￼                          Functionality
￼￼￼Wrt_data                                      32                             32-bit data to be written back to RF. Don’t care for sw and beq
Rs, Rt                                         5                             Addresses of source registers rs, rt. Note that these are defined for both R-type and I-type instructions
Wrt_reg_addr                                   5                             Address of the instruction’s destination register. Don’t care if the instruction does not update RF
wrt_enable                                     1                             Set if the instruction updates RF
nop                                            1                             If set, MEM stage performs a nop


Dealing with Hazards
The processor deals with two types of hazards :-
1. RAW Hazards: RAW hazards are dealt with using either only forwarding (if possible) or, if not, using stalling + forwarding. 
2. Control Flow Hazards: You will assume that branch conditions are resolved in the ID/RF stage of the pipeline. The processor deals with beq instructions as follows:
a. Branches are always assumed to be NOT TAKEN. That is, when a beq is fetched in the IF stage, the PC is speculatively updated as PC+4.
b. Branch conditions are resolved in the ID/RF stage. We will assume that every beq instruction has no RAW dependency with its previous two instructions. In other words, we do NOT have to deal with RAW hazards for branches!
c. Two operations are performed in the ID/RF stage: (i) Read_data1 and Read_data2 are compared to determine the branch outcome; (ii) the effective branch address is computed.
d. If the branch is NOT TAKEN, execution proceeds normally. However, if the branch is TAKEN, the speculatively fetched instruction from PC+4 is quashed in its ID/RF stage using the nop bit and the next instruction is fetched from the effective branch address. Execution now proceeds normally.
