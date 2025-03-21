#### HTTP codes
200
300 - Redirecionamento, {"Location"} header
400 - Erros

HTTP Chunks
Além do Content-eength, exuste o envio de dados em chunks com tamanho predefinido por um hexadecimal.

```C
req = "...headers, MENOS O Content-Length"
"Transfer-encoding: chuncked"
"\r\n"
"a"
"Hello motor"
```