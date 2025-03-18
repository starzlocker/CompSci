- Portas são endereços de 16 bits
- Sockets são a interface pela qual computadores conversam na rede;
- São identificados pelo endereço IP, porta e protocolo
- Um mesmo socket pode ser acessado de diferentes aplicações no host
### Pointers
- **&** points to a single data, structures don't need

## Criando Sockets
- PF_INET (PROTOCOL_FAMILY_INTERNET) e AF_INET (ADDRESS_FAMILY_INTERNET) - Existe um esquema de endereçamento para cada familia de protocolo (embora exista a possibilidade de ampliar), inclusive, AF_INET e PF_INET são intercambiáveis
- IPPROTO_TCP e IPPROTO_UDP são os protocolos ponta-a-ponta
- O valor de retorno do socket é um *int*
- Opaque handle? File descriptor? Socket descriptor? PID?

```C
int socket = socket(AF_INET, SOCK_STREAM, 0) // returns a positive integer for success
socket.close() // 0 on success, -1 otherwise

struct sockaddr
{
	unsigned short sa_family; // Familia de protocolos
	char sa_data[14];         // Endereço (específico da família ?)
}

struct in_addr
{
	unsigned long s_addr; // Endereço de internet (32 bits - 4 bytes)
}

struct sockaddr_in
{
	unsigned short sin_family; // Familia de endereços (AF_INET)
	unsigned short sin_port;   // endereço da porta (16 bits - 2 bytes)
	struct in_addr sin_addr;   // endereço de IP do socket (32 bits - 4 bytes)
	char sin_zero[8];          // 8 bytes não usados (padding?)
}

```

![[Pasted image 20250312213919.png]] 

A estrutura sockaddr e sockaddr_in tem o formato compatível e na verdade, se tratam de diferentes visualizações dos mesmo dados. **Por isso você pode fazer cast de sockadrr_in para sockadrr**, e usando a *sin_family*, as funções do socket interpretam o campo blob para saber como interpretar os dados que foram passados.
## Cliente TCP
Os passos para uma conexão TCP são:
- Criação do socket com *socket(PF_INET, SOCK_STREAM, IPPROTO_TCP)*
- Estabelecimento de uma conexão com *connect*
- Envio e recebimento de dados com *send()* e *recv()*
- Encerramento da conexão com *close()*

```C 
int socket(int protocol_family, int type, int protocol)

int close(int socket)

int connect(int socket, struct sockaddr * external_address, unsigned int address_length)


```
- *socket* é o descritor gerado pelo *socket()*
- *external_address* é um ponteiro para sockaddr_in (casteado para sockaddr) que contem o endereço de internet externo e a porta
- *address_length* é o tamanho da estrutura dado por *sizeof(struct sockaddr_in)*

```C
int send(int socket, const void *msg, unsigned int msg_length, int flags)

/*
socket: é o descritor do socket
*msg: é um ponteiro pra mensagem aka string
*/

int recv(int socket, void *rcv_buffer, unsigned int buffer_length, int flags)
```

**SEND()**
- socket: descritor do socket;
- \*msg: é o ponteiro para a estrutura de dados (string)
- msg_length: é o sizeof dessa estrutura
- flags: ?

**RECV()**
- socket;
- \*rcv_buffer: ponteiro para área de memória (parecido com uma cadeia de chars aka string) que vai armazenar a mensagem quando essa chegar
- buffer_length: tamanho esperado da estrutura
- - sin_family: a familia de endereços usada (AF_INET e AF_INET6)
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

#### Padding
- Structs são alinhadas pela maior estrutura