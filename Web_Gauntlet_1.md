# DuckWare Team - Web Gauntlet (picoCTF 2024)
###### Solved by @Antonio-alexandre

> This is a CTF about SQL Injection, Web Exploitation

## Desafio: Web Gauntlet 1 (Exploração Web)
#### Introdução
Este desafio explora vulnerabilidades em ambientes que utilizam SQLite. É composto de 5 fases, e exige que o usuário faça login na conta "admin" utilizando de técnicas de SQL Injection. Porém, há filtros que impossibilitam o uso de determinados comandos, que são adicionados a cada rodada, aumentando assim a complexidade e exigindo mais conhecimento.
- [Página do desafio](https://play.picoctf.org/practice/challenge/88)
### Análise Inicial
A princípio, são introduzidas algumas informações. São elas:
- [Link para as tentativas de login](http://jupiter.challenges.picoctf.org:19593/)
- [Link para o acesso aos filtros](http://jupiter.challenges.picoctf.org:19593/filter.php)

Além das seguintes dicas:
1. Você não tem permissão para fazer login com credenciais válidas.
2. Anote as injeções que você usa para o caso de perder seu progresso.
3. Para alguns filtros, pode ser difícil ver os caracteres, sempre (sempre) olhe para o hexadecimal bruto na resposta.
4. sqlite
5. Se o seu cookie continuar sendo redefinido, tente usar uma janela privada do navegador.

## Primeira fase

Ao acessar o link referente aos filtros, irá aparecer a mensagem: 
`Round1: or`

Ao acessar o link da aplicação, o usuário irá se deparar com uma tela de login:

[![Imagem1.png](https://i.postimg.cc/ZKxdLPbW/Imagem1.png)](https://postimg.cc/Czd5hqf0)

Caso tente logar como admin, irá se resultar na seguinte tela:

[![Imagem2.png](https://i.postimg.cc/8cV7Rcxd/Imagem2.png)](https://postimg.cc/N9pGYssL)

Assim, temos a primeira pista de como prosseguir.

`SELECT * FROM users WHERE username='admin' AND password='a'`

Esse comando, se refere a um método de consulta SQL, onde busca todos os usuários registrados na tabela `users` onde o `'username'` é igual a `'admin'` e a senha igual a `'a'` (credenciais que o usuário colocou).

Assim, temos o conhecimento de que a consulta e injeção serão feitas através do campo `Username`.

## Round 1
Juntando todas as informações que fora fornecidas, sabemos que não poderemos utilizar o operador OR na nossa injeção. 

Uma opção a ser utilizada, é o comando admin' --, porém antes é necessário entender como ele funciona.

Quando executamos esse comando no campo de username, o que ele faz é enviar a seguinte consulta para o banco de dados:
`SELECT * FROM users WHERE username = 'admin'--' AND password = 'aa';`

O operador `--` nada mais é do que uma opção de comentário em SQLite, que acaba ignorando qualquer coisa que venha após ele na consulta.

Essa é uma informação muito importante, pois é necessário que a consulta ignore a parte referente à senha, onde o comando enviado se resume a: 
`SELECT * FROM users WHERE username = 'admin';`

Ao digitarmos a injeção no campo Username, nos é retornado a seguinte mensagem: 

[![Imagem3.png](https://i.postimg.cc/5NTZnzsM/Imagem3.png)](https://postimg.cc/cr79JvqD)

Nada mais é do que uma confirmação de que a inserção foi correta. Assim, prosseguiremos para a fase 2.

## Round 2

No round 2, temos a mesma tela novamente. Ao tentar executar o mesmo comando, nos deparamos com o seguinte:

[![Imagem4.png](https://i.postimg.cc/hPzYzjcZ/Imagem4.png)](https://postimg.cc/rKTj7qd5)

Não aconteceu nada. Isso significa que foi adicionado uma informação nos filtros. Ao checar novamente a página, verificamos que agora aparece o seguinte: 
`Round2: or and like = --`

Então, tentaremos a seguinte abordagem: `admin'/*`. O `/*` presente, nada mais é do que outra sintaxe para o mesmo operador de comentário, executando assim a mesma requisição explicada na fase 1.

Assim, poderemos prosseguir para a próxima fase.

## Round 3

Já na fase 3, a página de filtros é a seguinte:
`Round3: or and = like > < --`

Ou seja, teremos que alterar um pouco a abordagem. Utilizaremos a seguinte injeção: `admin';`. O `;` presente, indica o fim da instrução enviada, o que acaba por ignorar tudo que vem posteriormente, fazendo exatamente o mesmo das outras fases.

Ao executar o comando, nos deparamos com o seguinte: 

[![Imagem5.png](https://i.postimg.cc/bJ5TpxDy/Imagem5.png)](https://postimg.cc/N91TxrkS)

Isto indica que poderemos prosseguir para a próxima fase.

## Round 4

Na fase 4, novamente iremos verificar a página de filtros antes de mais nada. Nos é apresentado o seguinte: 
`Round4: or and = like > < -- admin`

Isto é, não poderemos utilizar a palavra `admin` completa. Uma forma de enviar essa informação, é utilizar o operador de concatenação `||`, que nada mais é que um operador para juntar dois 'textos'. A injeção ficará assim: `ad'||'min';`. Como podemos ver, ela junta dois conceitos abordados tanto na fase 4, quanto na fase 3.

Ao executar a injeção, nos deparamos com essa tela: 

[![Imagem6.png](https://i.postimg.cc/MK9rFQtG/Imagem6.png)](https://postimg.cc/zVRjL3sm)

Assim, poderemos ir para a última fase.

## Round 5

Assim como a fase 4, poderemos utilizar a mesma injeção, pois a página de filtros apenas está filtrando os seguintes operadores: `Round5: or and = like > < -- union admin`

Ao executar a injeção utilizada na fase 4, aparece o seguinte:

[![Imagem7.png](https://i.postimg.cc/MKg1HZpD/Imagem7.png)](https://postimg.cc/KRr1s2Lk)

Assim, significa que concluímos o desafio e poderemos ir verificar novamente a página de filtros.

## Final

Ao entrarmos novamente na página de filtros, nos deparamos com o seguinte código: 

```
<?php
session_start();

if (!isset($_SESSION["round"])) {
    $_SESSION["round"] = 1;
}
$round = $_SESSION["round"];
$filter = array("");
$view = ($_SERVER["PHP_SELF"] == "/filter.php");

if ($round === 1) {
    $filter = array("or");
    if ($view) {
        echo "Round1: ".implode(" ", $filter)."<br/>";
    }
} else if ($round === 2) {
    $filter = array("or", "and", "like", "=", "--");
    if ($view) {
        echo "Round2: ".implode(" ", $filter)."<br/>";
    }
} else if ($round === 3) {
    $filter = array(" ", "or", "and", "=", "like", ">", "<", "--");
    // $filter = array("or", "and", "=", "like", "union", "select", "insert", "delete", "if", "else", "true", "false", "admin");
    if ($view) {
        echo "Round3: ".implode(" ", $filter)."<br/>";
    }
} else if ($round === 4) {
    $filter = array(" ", "or", "and", "=", "like", ">", "<", "--", "admin");
    // $filter = array(" ", "/**/", "--", "or", "and", "=", "like", "union", "select", "insert", "delete", "if", "else", "true", "false", "admin");
    if ($view) {
        echo "Round4: ".implode(" ", $filter)."<br/>";
    }
} else if ($round === 5) {
    $filter = array(" ", "or", "and", "=", "like", ">", "<", "--", "union", "admin");
    // $filter = array("0", "unhex", "char", "/*", "*/", "--", "or", "and", "=", "like", "union", "select", "insert", "delete", "if", "else", "true", "false", "admin");
    if ($view) {
        echo "Round5: ".implode(" ", $filter)."<br/>";
    }
} else if ($round >= 6) {
    if ($view) {
        highlight_file("filter.php");
    }
} else {
    $_SESSION["round"] = 1;
}

// picoCTF{y0u_m4d3_1t_cab35b843fdd6bd889f76566c6279114}
?> 
```

Podemos então verificar como os filtros impostos funcionam no código.

Por último, nas últimas linhas do código está a flag.
>`picoCTF{y0u_m4d3_1t_cab35b843fdd6bd889f76566c6279114}`
 
