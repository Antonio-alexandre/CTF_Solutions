# DuckWare Team - Web Gauntlet 2 (picoCTF 2024)
###### Solved by @Antonio-alexandre

> This is a CTF about SQL Injection, Web Exploitation

## Desafio: Web Gauntlet 2 (Exploração Web)
#### Introdução

Este desafio explora vulnerabilidades em ambientes que utilizam SQLite. É composto apenas de uma fase com filtros específicos de operadores e um limite de 25 caracteres que impossibilitam o uso de certas técnicas e exigem uma abordagem ainda mais curta.
- [Página do desafio](https://play.picoctf.org/practice/challenge/128)

#### Análise Inicial

A princípio, nos deparamos com dois links, um de filtros e um referente à aplicação. São eles:

- [Link para as tentativas de login](http://mercury.picoctf.net:28715/)
- [Link para os filtros](http://mercury.picoctf.net:28715/filter.php)

Ao acessar o link de filtros, nos deparamos com a seguinte informação: 
`Filters: or and true false union like = > < ; -- /* */ admin`

Assim, podemos já verificar quais operadores não poderemos utilizar.

Ao acessar o link da página de login, nos deparamos com a seguinte tela: 

[![Imagem10.png](https://i.postimg.cc/dtXVrnpR/Imagem10.png)](https://postimg.cc/ZC8mhPT0)

#### Solução

Além dos filtros, há uma limitação de 25 caracteres que é muito importante de ser lembrada.

Para acessarmos, é necessário utilizar o usuário admin, porém este está filtrado. Para contornar tal restrição, poderemos 'quebrar' o termo em dois, e utilizar o operador de concatenação || para juntar os dois fragmentos na consulta.

O campo Username então, ficará assim: `ad'||'min'`.

Já o campo de senha, poderá ser digitado qualquer coisa até 25 caracteres neste desafio. Utilizaremos a letra `a`.

Ao tentarmos enviar a requisição, normalmente, ele recusa e nos retorna `not admin`.

Para a solução deste desafio, utilizaremos o burpsuite e um conceito chamado Null Byte Injection.

Null Byte Injection nada mais é do que uma técnica para enviar dados que seriam filtrados normalmente. O que ela faz é basicamente enviar [bytes nulos](https://pt.wikipedia.org/wiki/Caractere_nulo). É mais uma abordagem para encerrar um comando, sem o uso de comentários. 

Para isso, faremos o passo a passo básico. Tentaremos enviar as informações com o modo Intercept do Proxy do BurpSuite ativado. Esse modo Intercept nada mais é do que um modo que nos possibilita ver e editar as informações que iremos enviar antes de serem enviadas. Como o caracter de Null Byte (`%00`) não é reconhecido normalmente pelo site, utilizaremos ele para enviar.

Ao tentar enviar, o burpsuite receberá as informações a serem enviadas. Ficará assim:

[![Imagem11.png](https://i.postimg.cc/44YddrF4/Imagem11.png)](https://postimg.cc/NLwYCPcZ)

Prosseguiremos então, editando a linha: `user=ad%27%7C%7C%27min%27&pass=a` para: `user=ad%27%7C%7C%27min%27%00&pass=a` e clicando em  `Forward ` no topo da tela para enviar ao site.

Ao realizar esse passo a passo, nos deparamos com a seguinte tela: 

[![Imagem12.png](https://i.postimg.cc/NM5gKHDC/Imagem12.png)](https://postimg.cc/FdXtq120)

`Detalhe importante: caso não apareça a flag na página de filtros, limpe os cookies do site, faça login novamente e repita o processo`

Assim, poderemos prosseguir para a página de filtros.

Na página de filtros, nos é apresentado o seguinte código:

```
<?php
session_start();

if (!isset($_SESSION["winner3"])) {
    $_SESSION["winner3"] = 0;
}
$win = $_SESSION["winner3"];
$view = ($_SERVER["PHP_SELF"] == "/filter.php");

if ($win === 0) {
    $filter = array("or", "and", "true", "false", "union", "like", "=", ">", "<", ";", "--", "/*", "*/", "admin");
    if ($view) {
        echo "Filters: ".implode(" ", $filter)."<br/>";
    }
} else if ($win === 1) {
    if ($view) {
        highlight_file("filter.php");
    }
    $_SESSION["winner3"] = 0;        // <- Don't refresh!
} else {
    $_SESSION["winner3"] = 0;
}

// picoCTF{k3ep_1t_sh0rt_2a78ea34c84da0bf585ada4cb9a6f8fb}
?>
```

Podemos verificar que há todos os filtros e limitações que fora mencionados anteriormente. 

Por último, nas últimas linhas do código está a flag.
>`picoCTF{k3ep_1t_sh0rt_2a78ea34c84da0bf585ada4cb9a6f8fb}`
 
