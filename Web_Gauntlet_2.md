# DuckWare Team - Web Gauntlet 2 (picoCTF 2024)
###### Solved by @Antonio-alexandre

> This is a CTF about SQL Injection, Web Exploitation

## Desafio: Web Gauntlet 2 (Exploração Web)
#### Introdução

Este desafio explora vulnerabilidades em ambientes que utilizam SQLite. É composto apenas de uma fase com filtros específicos que impossibilitam o uso de certas técnicas.
- [Página do desafio](https://play.picoctf.org/practice/challenge/174O)

#### Análise Inicial

A princípio, nos deparamos com dois links, um de filtros e um referente à aplicação. São eles:

- [Link para as tentativas de login](http://mercury.picoctf.net:26215/)
- [Link para os filtros](http://mercury.picoctf.net:26215/filter.php)

Ao acessar o link de filtros, nos deparamos com a seguinte informação: 
`Filters: or and true false union like = > < ; -- /* */ admin`

Assim, podemos já verificar quais operadores não poderemos utilizar.

Ao acessar o link da página de login, nos deparamos com a seguinte tela: 

[![Imagem8.png](https://i.postimg.cc/L5KJtDn0/Imagem8.png)](https://postimg.cc/WF8pjMmM)

#### Solução

Para acessarmos, é necessário utilizar o usuário `admin`, porém este está filtrado. Para contornar tal restrição, poderemos 'quebrar' o termo em dois, e utilizar o operador de concatenação `||` para juntar os dois fragmentos na consulta. 

O campo `Username` então, ficará assim: `ad'||'min'`.

Já o campo de senha, não poderemos utilizar apenas um operador de comentário no campo `Username` para contornar, pois todos estão filtrados. Sendo assim, é necessário utilizar uma expressão que sempre retorne verdadeiro para forçar a entrada.

Com o operador `OR` está filtrado, uma solução é utilizar o operador `NOT`. Sendo assim, ficará assim o campo `Password`: `a' IS NOT 'b`.

Ao executar, nos deparamos com o seguinte: 

[![Imagem9.png](https://i.postimg.cc/sgN7Bp9b/Imagem9.png)](https://postimg.cc/Mn1vNfLY)

Assim, poderemos ir novamente para o link de filtros.

Ao acessá-lo, aparece o seguinte código:

```
<?php
session_start();

if (!isset($_SESSION["winner2"])) {
    $_SESSION["winner2"] = 0;
}
$win = $_SESSION["winner2"];
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
    $_SESSION["winner2"] = 0;        // <- Don't refresh!
} else {
    $_SESSION["winner2"] = 0;
}

// picoCTF{0n3_m0r3_t1m3_fc0f841ee8e0d3e1f479f1a01a617ebb}
?>
```

Podemos verificar que há todos os filtros que fora mencionados anteriormente. 

Por último, nas últimas linhas do código está a flag.
>`picoCTF{0n3_m0r3_t1m3_fc0f841ee8e0d3e1f479f1a01a617ebb}`
 
