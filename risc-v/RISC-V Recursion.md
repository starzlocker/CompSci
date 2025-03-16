tags: #risc-v #isa
A função fatorial em RISC fica assim:

Vamos seguir o fluxo do código para `n = 2`, detalhando cada passo e mostrando o estado dos registradores e da pilha.

#### Inicialização

- `n = 2`
- `x10` = 2
#### Primeiro Chamado (n = 2)

1. **Aloca espaço na pilha:**
    
    ```
    addi sp, sp, -16 // aloca espaço para dois registradores
    
    ```
    
    - `sp` = `sp` - 16
2. **Salva registradores na pilha:**
    
    ```
    sd x1, 8(sp) // endereço de retorno atual
    sd x10, 0(sp) // valor de n atual
    
    ```
    
    - `sp[8]` = `x1`
    - `sp[0]` = 2
3. **Subtrai 1 de `n`:**
    
    ```
    addi x5, x10, -1 // x5 = n - 1
    
    ```
    
    - `x5` = 1
4. **Verifica se `n > 0`:**
    
    ```
    bge x5, x0, L1
    
    ```
    
    - `x5` >= 0 é verdadeiro, então pula para `L1`.

#### Bloco Recursivo

1. **Decrementa `n`:**
    
    ```
    L1: addi x10, x10, -1 // n >= 1: argument gets (n − 1)
    
    ```
    
    - `x10` = 1
2. **Chama recursivamente a função `fact`:**
    
    ```
    jal x1, fact
    
    ```
    
    - `x1` armazena o endereço de retorno.

#### Segundo Chamado (n = 1)

1. **Aloca espaço na pilha:**
    
    ```
    addi sp, sp, -16
    
    ```
    
    - `sp` = `sp` - 16 (agora `sp` está em `sp - 32`)
2. **Salva registradores na pilha:**
    
    ```
    sd x1, 8(sp)
    sd x10, 0(sp)
    
    ```
    
    - `sp[8]` = `x1` (endereço de retorno do segundo chamado)
    - `sp[0]` = 1
3. **Subtrai 1 de `n`:**
    
    ```
    addi x5, x10, -1 // x5 = n - 1
    
    ```
    
    - `x5` = 0
4. **Verifica se `n > 0`:**
    
    ```
    bge x5, x0, L1
    
    ```
    
    - `x5` >= 0 é verdadeiro, então pula para `L1`.

#### Bloco Recursivo

1. **Decrementa `n`:**
    
    ```
    L1: addi x10, x10, -1 // n >= 1: argument gets (n − 1)
    
    ```
    
    - `x10` = 0
2. **Chama recursivamente a função `fact`:**
    
    ```
    jal x1, fact
    
    ```
    
    - `x1` armazena o endereço de retorno.

#### Terceiro Chamado (n = 0, Caso Base)

1. **Aloca espaço na pilha:**
    
    ```
    addi sp, sp, -16
    
    ```
    
    - `sp` = `sp` - 16 (agora `sp` está em `sp - 48`)
2. **Salva registradores na pilha:**
    
    ```
    sd x1, 8(sp)
    sd x10, 0(sp)
    
    ```
    
    - `sp[8]` = `x1` (endereço de retorno do terceiro chamado)
    - `sp[0]` = 0
3. **Subtrai 1 de `n`:**
    
    ```
    addi x5, x10, -1 // x5 = n - 1
    
    ```
    
    - `x5` = -1
4. **Verifica se `n > 0`:**
    
    ```
    bge x5, x0, L1
    
    ```
    
    - `x5` >= 0 é falso, então não pula.

#### Caso Base

1. **Define `x10` para 1:**
    
    ```
    addi x10, x0, 1
    
    ```
    
    - `x10` = 1
2. **Desaloca o espaço da pilha:**
    
    ```
    addi sp, sp, 16
    
    ```
    
    - `sp` = `sp` + 16 (agora `sp` está em `sp - 32`)
3. **Retorna ao chamador:**
    
    ```
    jalr x0, 0(x1)
    
    ```
    
    - Retorna ao chamador com `x10` = 1

#### Retorno ao Segundo Chamado (n = 1)

1. **Continua após a chamada recursiva:**
    
    ```
    addi x6, x10, 0
    
    ```
    
    - `x6` = 1
2. **Restaura `n` e o `x1`:**
    
    ```
    ld x10, 0(sp)
    ld x1, 8(sp)
    
    ```
    
    - `x10` = 1
    - `x1` restaurado (endereço de retorno do primeiro chamado)
3. **Desaloca o espaço da pilha:**
    
    ```
    addi sp, sp, 16
    
    ```
    
    - `sp` = `sp` + 16 (agora `sp` está em `sp - 16`)
4. **Calcula `n * fact(n - 1)`:**
    
    ```
    mul x10, x10, x6
    
    ```
    
    - `x10` = 1 * 1 = 1
5. **Retorna ao chamador:**
    
    ```
    jalr x0, 0(x1)
    
    ```
    
    - Retorna ao chamador com `x10` = 1

#### Retorno ao Primeiro Chamado (n = 2)

1. **Continua após a chamada recursiva:**
    
    ```
    addi x6, x10, 0
    
    ```
    
    - `x6` = 1
2. **Restaura `n` original:**
    
    ```
    ld x10, 0(sp)
    ld x1, 8(sp)
    
    ```
    
    - `x10` = 2
    - `x1` restaurado (endereço de retorno do chamador original)
3. **Desaloca o espaço da pilha:**
    
    ```
    addi sp, sp, 16
    
    ```
    
    - `sp` = `sp` + 16 (agora `sp` está no valor original)
4. **Calcula `n * fact(n - 1)`:**
    
    ```
    mul x10, x10, x6
    
    ```
    
    - `x10` = 2 * 1 = 2
5. **Retorna ao chamador:**
    
    ```
    jalr x0, 0(x1)
    
    ```
    
    - Retorna ao chamador original com `x10` = 2

#### Resumo

Para `n = 2`:

1. **Primeira chamada:** Aloca espaço, verifica `n >= 0`, chama recursivamente com `n-1`.
2. **Segunda chamada:** Aloca espaço, verifica `n > 0`, chama recursivamente com `n-1`.
3. **Terceira chamada:** Caso base, retorna 1.
4. **Retorna ao segundo chamado:** Calcula `1 * 1 = 1`, retorna.
5. **Retorna ao primeiro chamado:** Calcula `2 * 1 = 2`, retorna.

O resultado final é `x10 = 2`, que é o fatorial de 2.