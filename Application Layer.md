HTTP: incrementação do TCP
### Requests
```
GET /path_to_file.file HTTP 1/1\r\n
```

```
HTTP 1/1 200 ok\r\n
```

\r\n: são os caracteres *return carriage* e *line feed*. Cada sistema operacional implementa o comando de nova linha, mas para HTTP, o padrão é esse.

```C
// HEADERS
"Connection": "Keep-Alive | Close"
/*
Especifica se haverão mais requisicões do cliente nessa mesma conexão
*/

```