tags: #architecture/compiling
## Compiler
- Substitui  as diretrizes por referências válidas e compila o arquivo em Assembly.

Compiler::Diretrizes e Assembly
<!--SR:!2024-08-11,4,270-->
## Assembler
- Substitui pseudo instruções como LI, MOV em instruções reais
- Converte o código Assembly em código de máquina
- Organiza o código em diferentes segmentos:
	- Headers: contém a posição e o tamanho dos outros segmentos
	- Text Segment: contém o código de máquina
	- Static Data and Dynamic Data: segmento reservado às variáveis, constantes e estruturas de dados durante a vida do programa
	- References: as instruções que dependem de um endereço absoluto
	- Symbol table: guarda os labels que ainda não foram definidos, as referências externas
	- Debbuging Info

Assembler::Instruções, machine-code e organização
<!--SR:!2024-08-11,4,270-->

Segmentos do programa Assembly::Header, text seg, data seg, ref, symbol tab e debug info
<!--SR:!2024-08-11,4,270-->

## Linker
Une as bibliotecas, já em formato de **objeto**, ao arquivo principal.

Linker::União
<!--SR:!2024-08-11,4,270-->
## Loader
1. Coleta os tamanhos dos segmentos de texto e memória do Header do executável
2. Cria espaço na memória
3. O kernel prepara os parâmetros e os coloca na stack
5. Uma rotina de inicialização chama o procedimento
6. O terreno é preparado com a inicialização dos registradores: stack pointer aponta para o próximo endereço livre da stack
7. Os parâmetros são passados para os registradores
8. Após o procedimento, a rotina finaliza o procedimento.

### Linked Libraries
* Carregadas na execução do programa
* Criam uma rotina *dummy* para cada rotina que não é local
* Essa rotina *dummy* aponta para um pedaço de código "trampolim", que coloca o ID (offset) da função real no registrador
* O código chama o **Loader** que encontra o procedimento e substitui o endereço no *Dummy* pelo endereço real
* Todas as funções da biblioteca tem seu ID salvo em uma tabela.

```RISC-V

swap:
	slli x6, x11, 3
	add  x6, x10, x6
	ld   x5, 0(x6)
	ld   x7, 8(x5)
	sd   x7, 0(x6)
	sd   x5, 0(x7)
	jalr x0, 0(x1)
	
```