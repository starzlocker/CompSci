tags: #risc-v #isa
**caller**: o programa que chama o procedimento e passa os parâmetros
**callee**: o procedimento
**return-address**: endereço de retorno do procedimento, no registrador x1. Normalmente o PC incrementado em uma instrução.

O *caller* chama o *callee*, um LABEL, passando os parâmetros nos registradores específicos x10 ao x17 e salvando o *return-address* no registrador x1.

```RISC-V
addi x10, 10
jal x1, PROCEDURE

PROCEDURE:
	addi x10, 10
	jalr x0, 0(x1)
```

![[Untitled.png]]
