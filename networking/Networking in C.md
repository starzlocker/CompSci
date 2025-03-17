- sin_family: a familia de endereços usada (AF_INET e AF_INET6)
- sin_port: a porta, deve ser convertida para big-endian com htons
- sin_addr.s_addr: no cliente é o endereço com o qual ele se conecta e no servidr é o endereço que espera as conexões
### HTON* e NTOH*
*host to network long/short* e *network to host short/long*
Essas funções convertem binários com multiplos bytes (como um long e um short) para a ordem de bytes usada na rede (*big-endian*), essa ordem determina como os bytes de uma estrutura multibyte são ordenados, começando do byte mais significativo (mais à direita) ou do menos significativo (à esquerda). Basicamente, os dois modos são big e little endian:

``` 
A ordem natural de um binário é big-endian:
|78|56|34|12|

Little endian
|12|34|56|78|
```

Para evitar erros de comunicação: sempre que se passa uma estrutura desse tipo pela rede, é necessário converter essas estruturas e converte-las de volta no recebimento.