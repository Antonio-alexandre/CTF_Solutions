# Duckware Team - lotto (pwnable)
###### Solved by @Antonio-alexandre

> This CTF is about pwn, code vulnerability

## Desafio: lotto (Pwn, vulnerabilidade de código)
#### Introdução

Neste desafio, precisaremos analisar e identificar vulnerabilidades em um código a fim de tomar conta e vantagem delas para identificar a flag.

- [Página do desafio](http://pwnable.kr/play.php)

#### Análise Inicial

A princípio, nos deparamos com a seguinte descrição: 

> Mamãe! Fiz um programa de loteria para minha lição de casa.
Você quer jogar?

Além do seguinte código para acessar o servidor:

`ssh lotto@pwnable.kr -p2222 (pw:guest)`

#### Solução

Ao nos conectarmos ao servidor, o primeiro passo é executar o comando `ls` para verificar quais arquivos estão presentes.

Ao executar o comando, nos deparamos com três arquivos:

`flag` `lotto` `lotto.c`

Para o primeiro passo, executaremos o arquivo chamado `lotto`. Para isso, basta digitar `./lotto` no terminal.

Feito isso, nos deparamos com o seguinte menu:

[![Imagem1.png](https://i.postimg.cc/50fq9Bb4/Imagem1.png)](https://postimg.cc/Y4ymbmvZ)

Digitaremos então o número `2` para ver do que se trata. Feito isso, aparecerá a seguinte mensagem:

[![Imagem2.png](https://i.postimg.cc/RZfccs0C/Imagem2.png)](https://postimg.cc/nXFjx17N)

Tentaremos então digitar `1` no terminal, e enviar qualquer sequência de números aleatórios para ver o que acontece.

[![Imagem3.png](https://i.postimg.cc/Fskc4W3b/Imagem3.png)](https://postimg.cc/YhkjNxXj)

Como não conseguimos resposta alguma, é preciso analisar o código. Para isso, basta digitar `cat lotto.c` no terminal. O código é o seguinte:

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>

unsigned char submit[6];

void play(){

        int i;
        printf("Submit your 6 lotto bytes : ");
        fflush(stdout);

        int r;
        r = read(0, submit, 6);

        printf("Lotto Start!\n");
        //sleep(1);

        // generate lotto numbers
        int fd = open("/dev/urandom", O_RDONLY);
        if(fd==-1){
                printf("error. tell admin\n");
                exit(-1);
        }
        unsigned char lotto[6];
        if(read(fd, lotto, 6) != 6){
                printf("error2. tell admin\n");
                exit(-1);
        }
        for(i=0; i<6; i++){
                lotto[i] = (lotto[i] % 45) + 1;         // 1 ~ 45
        }
        close(fd);

        // calculate lotto score
        int match = 0, j = 0;
        for(i=0; i<6; i++){
                for(j=0; j<6; j++){
                        if(lotto[i] == submit[j]){
                                match++;
                        }
                }
        }

        // win!
        if(match == 6){
                system("/bin/cat flag");
        }
        else{
                printf("bad luck...\n");
        }

}

void help(){
        printf("- nLotto Rule -\n");
        printf("nlotto is consisted with 6 random natural numbers less than 46\n");
        printf("your goal is to match lotto numbers as many as you can\n");
        printf("if you win lottery for *1st place*, you will get reward\n");
        printf("for more details, follow the link below\n");
        printf("http://www.nlotto.co.kr/counsel.do?method=playerGuide#buying_guide01\n\n");
        printf("mathematical chance to win this game is known to be 1/8145060.\n");
}

int main(int argc, char* argv[]){

        // menu
        unsigned int menu;

        while(1){

                printf("- Select Menu -\n");
                printf("1. Play Lotto\n");
                printf("2. Help\n");
                printf("3. Exit\n");

                scanf("%d", &menu);

                switch(menu){
                        case 1:
                                play();
                                break;
                        case 2:
                                help();
                                break;
                        case 3:
                                printf("bye\n");
                                return 0;
                        default:
                                printf("invalid menu\n");
                                break;
                }
        }
        return 0;
}
```

Ao analisarmos o código por completo, nos deparamos com a seguinte função:

```
// calculate lotto score
int match = 0, j = 0;
for(i=0; i<6; i++){
        for(j=0; j<6; j++){
                if(lotto[i] == submit[j]){
                        match++;
                }
        }
}
```

Nesse trecho, o código compara cara número que foi gerado (array lotto) com os números digitados (array submit). Isto é, para cada número gerado, ele verifica se há uma correspondência no array de números enviados. Com isso, podemos notar algumas falhas:

- Essa lógica não verifica se os números correspondentes estão na posição desejada, ou se são únicos. Isto é, a variável que conta a quantidade de acertos é incrementada mesmo que não haja correspondência.
- Há uma falta de verificação de unicidade dos números digitados. Isto é, caso eu envie `222222` por exemplo, ele incrementa a quantidade de correspondências mesmo que não tenha acertado todos os dígitos.

Porém, como a função não aceita valores como `1, 2, 3, 4, etc,` é necessário enviar valores em `ASCII` que correspondem ao limite estabelecido (`até 46`). No entanto, os números que conhecemos (de 0 a 9), possuem valores que vão de `48` a `57`. Isto é, precisaremos enviar outros caracteres em que os seus valores em `ASCII` não ultrapassem `46`.

Aqui estão alguns exemplos da tabela `ASCII` de valores aceitos:

[![Imagem4.png](https://i.postimg.cc/bNLrD4kf/Imagem4.png)](https://postimg.cc/ykgs2QLL)

Com essas informações, basta tentar repetidas vezes enviar uma sequência de valores repetidos (é válido ressaltar que a chance de acerto é de `1/46`).

Feito isso, ao acertarmos a sequência, nos deparamos com a seguinte mensagem:

[![Imagem5.png](https://i.postimg.cc/15R5QMTC/Imagem5.png)](https://postimg.cc/23JfddCh)


Assim então, temos acesso a `flag`:

>`sorry mom... I FORGOT to check duplicate numbers... :(`
 