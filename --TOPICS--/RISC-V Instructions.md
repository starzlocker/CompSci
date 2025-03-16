tags: #risc-v #isa 
Levando em consideração a versão de 32-bits, essa ISA tem 5 tipos diferentes de instrução.

#### **R-Type**
- Instruções entre registradores
- Composto por FUNCT7, RS1, RS2, RSD, FUNCT3, OPCODE
- ADD e SUB
#### **I-Type**
- Instruções com imediatos
- Composto por IMM, RS1, RSD, FUNCT3 e OPCODE
- ADDI, SUBI e LW
- SW
#### **SB-Type**
- Instruções de branching
- Composto por IMM(12), RS1, RS2, FUNCT3 e OPCODE
- BEQ, BNQ
- Deslocamento em **half-words**
#### **UJ-Type**
- Jump incondicional
- Composto por IMM(20), RSD, OPCODE
- Deslocamento em **half-words**
#### **U-Type**
- Load Upper Bits
- Composto por IMM(20), RD e OPCODE