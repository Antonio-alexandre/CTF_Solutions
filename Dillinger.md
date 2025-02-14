# DuckWare Team - Web Gauntlet (picoCTF 2024)
###### Solved by @Antonio-alexandre

> This is a CTF about SQL Injection, Web Exploitation

## Desafio: Web Gauntlet 1 (Exploração Web)
#### Introdução
Este desafio explora vulnerabilidades em ambientes que utilizam SQLite. É composto de 5 fases, e exige que o usuário faça login na conta "admin" utilizando de técnicas de SQL Injection. Porém, há filtros que impossibilitam o uso de determinados comandos, que são adicionados mais a cada rodada, aumentando assim a complexidade e exigindo mais conhecimento.
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

## Solution
[Discorra, de forma completa, como se resolve a questão]
>`[Insira a flag]`
 