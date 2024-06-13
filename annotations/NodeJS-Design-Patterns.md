# Como o JavaScript funciona

## I/O is slow

Operações de I/O são as mais lentas entre as operações computacionais fundamentais. Enquanto para o computador acessar a RAM leva um tempo na ordem de nanosegundos para se acessar dados em disco ou em rede o tempo está na ordem dos milissegundos.
Porém operações I/O não são muito custosas em termos de CPU.

## Blocking I/O

Chamadas de funções correspondentes com requisições I/O irão bloquear a execução da thread até a operação terminar.
Serviços web que são implementados usando `blocking I/O` não estará apto a receber multiplas conexões na mesma thread. Isso se deve porque cada operação I/O irá bloquear o processamento de qualquer outra conexão.

## Non-blocking I/O

Outro mecanismo para acessar recursos é chamado de `non-blocking I/O`, nesse modo de operação as chamadas do sistema sempre retorna imediatamente sem esperar o dado para ser lido ou escrito. Se nenhum resultado estiver disponivel no momento da chamada ele retornará uma constante pré-definida.
O padrão mais comum para lidar com esse tipo de `non-blocking I/O` é ativamente pesquisar o recurso em loop ate realmente algum dado ser retornado. Isso é chamado de `busy-waiting`. Porém com essa técnica simples irá consumir uma quantidade preciosa de CPU por iteração em que o recurso estará indisponível a maior parte do tempo.

## Event demultiplexing

O `event demultiplexing` observa varios recursos e retorna um novo evento (ou um conjunto de eventos) quando a operação de leitura ou escrita executada sobre um dos recursos é finalizada.
Uma vantagem é que ele é sincrono, ou seja, ele bloqueia até que haja novos eventos para serem processados.

## The reactor pattern

A ideia principal por tras do `reactor pattern` é de ter um `handler` associado a cada operação I/O. Um `handler` em NodeJS é representado pela função `callback`. O `handler` será invocado assim que o evento é produzido e processado pelo loop de eventos.
A aplicação expressa interesse em acessar um recurso em um determinado momento (sem bloquear), e fornece um `handler`, que vai então ser invocado em um outro momento quando a operação se completar.

## Libuv, the I/O engine of Node.js

Cada sistema operacional possui sua propria interface para o `event demultiplexer`: `epoll` no Linux, `kqueue` no macOs e o `I/O completion port (IOCP) API` no Windows.
Porém cada operação I/O pode se comportar um pouco diferente dependendo do tipo de recurso, até para operações dentro do mesmo sistema operacional. Em sistemas Unix, `filesystem` regulares nao suportam operações `non-blocking`, então para simular esse comportamento é necessario usar um thread separado fora do loop de eventos.
Devido a essas inconsistencias é necessario um alto nivel de abstração para a construção do `event multiplexer`. **Assim o NodeJS criou a lib nativa chamada `libuv`, com o objetivo de fazer o NodeJS compativel com a maioria dos sistemas operacionais e normalizar o comportamento das operações `non-blocking` em diferentes tipos de recursos. `Libuv` então representa uma engine de baixo nivel do NodeJS e é provavelmente o componente mais importante que o NodeJS é baseado**

## The recipe for Node.js

O `reactor pattern` e `libuv` são fundações basicas do NodeJS, mas existem mais 3 componentes para construir a plataforma completa:

- Um conjunto de ligações responsaveis por agrupar e export a `libuv` e outras funcionalidades de baixo nivel do JavaScript
- `V8`, que é a engine do JavaScript criada pelo Google para o navegador Chrome. Uma das razoes pela qual o NodeJS é tão rapido e eficiente.
- Uma lib `core` do JavaScript que implementa APIs de mais alto nivel

# JavaScript no NodeJS

Rodando o JavaScript no NodeJS temos acesso a coisas diferentes ao roda-lo no navegador e vice versa, por exemplo, em NodeJS nao temos acesso ao `window` ou `document`, e no caso do Navegador nao temos acesso a serviços providos pelo sistema operacional, onde o NodeJS pode virtualmente ter acesso a esses servições exportados pelo sistema operacional.

## The module system

O `module system` original do NodeJS é chamado de `CommonJS` e ele usa o `require` como palavra chave para importar funções, variaveis e classes exportadas por modulos internos e outros modulos do sistema.
Hoje em dia o JavaScript possui o chamado `ECMAScript modules (ESM ou ESmodule)` onde usa o `import` ao invés do `require`.

## Full access to operating system services

Devido ao NodeJS usar o JavaScript sem os limites do navegador, podemos ter acesso aos arquivos do sistema graças ao modulo `fs`, ou podemos escrever aplicacoes que usem o baixo nivel sockets TCP ou UDP graças aos modulos `net` e `dgram`, podemos tambem criar servers HTTP(S) com os modulos `http` e `https` ou utilizar algoritmos de hashing e encriptação do OpenSSL com o modulo `crypto`. Podemos tambem acessar alguns componentes internos do V8 utilizando o modulo `v8` ou rodar codigo em diferentes contextos do V8 com o modulo `vm`.
Pode-se rodar outros processos utilizando o modulo `child_process` ou recuperar informações de processamento sobre nossas proprias aplicações usando a variavel global `process`. Sobre a variavel global `process` podemos ter acesso às variaveis ambientes utilizadas no processo (`process.env`) ou os argumentos utilizados via linha de comando passados para a aplicação no momento de seu lançamento (com `process.argv`).

## Running native code

O NodeJS oferece suporte para implementar modulos nativos graças a interface `N-API`. Isso permite com que possamos reutilizar, com um pouco de esforço, uma vasta gama de bibliotecas open source e permite com que possamos reaproveitar codigos legados em C/C++ sem a necessidade de migra-los.
Onde codigos nativos sao ainda muito importante para acessar em baixo nivel e comunicar com drivers de hardwares ou com portas, como por exemplo portas USB ou serial.

# CommonJS modules

Dois principais conceitos:

- `require` é uma função que permite importar um modulo de qualquer lugar do sistema de arquivos
- `exports` e `module.exports` sao variaveis especiais que podem ser usadas para exportar funcionalidades publicas do modulo atual

## The require function is synchronous
