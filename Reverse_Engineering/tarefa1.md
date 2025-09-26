# Duckware Team - MarsAnalytica
###### Solved by @Antonio-alexandre

> This CTF is about Reverse-Engineering

## Desafio: MarsAnalytica
#### Introdução

Neste desafio, é fornecido um arquivo aparentemente simples, com o nome de tarefa1 (anteriormente chamado marsanalytica). Como se trata de um exercício de engenharia reversa, precisaremos descobrir um CitizenID em um binário ofuscado.

#### Análise Inicial

Ao tentar executar o arquivo, ele exibe a seguinte tela:

[![image.png](https://i.postimg.cc/BZK8JxTf/image.png)](https://postimg.cc/23CjwL90)

Pode-se concluir então, que o código recebe um ID de entrada, faz a validação dele internamente, e caso esteja incorreto, resulta no travamento do programa.

Neste caso, é necessário então analisar toda a estrutura a fim de descobrir o `CitizenID` correto.

#### Solução

Logo de início, é importante analisar as informações básicas que pode-se obter do binário.

Para isso, foram utilizados os seguintes comandos:

[![image.png](https://i.postimg.cc/brNF9zHQ/image.png)](https://postimg.cc/0MFf814Q)

Como pode-se notar, o arquivo é um executável `ELF` "comum".

O próximo passo, é executar o comando `strings` para ver se tem alguma informação útil.

Ao executar, logo no final do arquivo tem as seguintes informações:

[![image.png](https://i.postimg.cc/15bJf0T0/image.png)](https://postimg.cc/XpcwP5nX)

Percebe-se que a string `UPX!` se repete. [UPX](https://upx.github.io/) nada mais é que um empacotador de arquivos de código aberto. Nesse caso, é necessário extrair o arquivo original. Para isso, foi utilizado o comando:

`upx -d tarefa1 -o tarefa1_2`

[![image.png](https://i.postimg.cc/SsDhWQzh/image.png)](https://postimg.cc/2bbMYDwc)

O próximo passo executado, foi executar o binário no r2.

[![image.png](https://i.postimg.cc/8cWcVrs6/image.png)](https://postimg.cc/0KkPYjYk)

Feito isso, foram obtidas algumas informações interessantes.

Uma delas, é a função sym.imp.getchar. Analisando essa função, é possível entender que é ela que recebe cada caractere e faz a verificação. 

Feito isso, basta testar o código em tempo de execução.

Para isso, novamente foi utilizado o r2.

[![image.png](https://i.postimg.cc/hvbJddnw/image.png)](https://postimg.cc/SYRN0jJ7)

Instalei um breakpoint no endereço onde é realizada a verificação do bit: `0x004031a6`

Após isso, foram realizados testes para cada bit contendo cada letra do alfabeto, através do comandos:

```
dr rsi    
dr edx    
dc
```

Foi obtido então, a seguinte sequência de flags:

`[7,8,13,15,16,26,27,22,21,4,18,28,23,29,9,1,25,30,17]`

Isso significa que cada caractere digitado vai diretamente para uma posição no buffer. No caso do código, ele aplica operações internas e faz a comparação com os valores que já foram estabelecidos.

Aqui está um trecho de exemplo, que faz essa verificação:

`(buf[14] * buf[6])*((buf[12]-buf[10])^buf[13]) - 0x3fcf == 0`

Então, apenas foi utilizado o seguinte código para descobrir a string correta:

```
from z3 import *

solver = Solver()

# Flag order
order=[7,8,13,15,16,26,27,22,21,4,18,28,23,29,9,1,25,30,17]
flag={}
for i in order:
	flag[i] = BitVec("c_%d" % i,8)

#char constraints
for k in flag.keys():
	solver.add(flag[k] >= 32, flag[k] <= 126)
	
#equations
solver.add((flag[9] * flag[27])*((flag[23]-flag[18])^flag[29]) == 0x3fcf)
...
solution=''
if solver.check() == z3.sat:
    model=solver.model()
    for i in order:
        solution+=chr(model[flag[i]].as_signed_long())
print solution
```

(Créditos do código para [scud](https://re-dojo.github.io/post/2018-10-28-NSEC-2018-mars-analytica/))

Esse script resulta na seguinte string:

`q4Eo-eyMq-1dd0-leKx`

Executando ela no binário, obtivemos a seguinte resposta:

[![image.png](https://i.postimg.cc/sf99SBwb/image.png)](https://postimg.cc/XXqC6vMg)

#### Conclusão

Foi um desafio complexo, de grande dificuldade técnica e de compreendimento. Aprendi bastante, principalmente como utilizar o r2 da maneira correta para essa verificação.

É necessária uma análise minuciosa de cada função para entender o fluxo do código, e esse foi o principal empecilho, visto que o código estava inteiro ofuscado.