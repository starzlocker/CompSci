## Low Level
- A implementação de ponteiros ocorre desde o baixo até o alto nível;

### Diferença na implementação de um loop com arrays Vs com ponteiros
```RISC-V
# Limpador de array (por array)
# x10 = array[0]
# x11 = tam (do array, lembra que é C)

    li x5, 0         #x5 = i
lp: slli x6, x5, 3   #x6 = i (doublews)
    addi x7, x10, x6 #x7 =array[i]
    addi x5, x5, 1   #x5 = i++
    sd x0, 0(x7)
    blt x6, x11, lp

# Limpador de array (por ponteiro)
# x10 = *p
# x11 = tam

# Versão em C para comparação
# for(int p = &array[0]; p < &array[tam]; p = p + 1){
#   *p = 0;
# }

    mv x5, x10         # x5 = &array[0]
    slli x6, x11, 3    # x6 = tam * 3
    addi x7, x10, x6   # x7 = array[tam]
lp: sd x0, 0(x5)
    addi x5, x5, 8     # x5 = *p + 1 (arit. de ponteiros)
    bltu x5, x7, lp
    

```
* A versão com ponteiros consegue eliminar a necessidade de incrementar e converter em bytes o índice, já que trabalha direto com o endereço. Assim, tendo o endereço inicial e o final, só precisamos incrementar por doublewords a cada loop;
* Compiladores modernos já fazem essa otimização em loops com arrays.