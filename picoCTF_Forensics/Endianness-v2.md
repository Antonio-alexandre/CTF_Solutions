# DuckWare Team - Endianness-v2 (picoCTF 2024)
###### Solved by @Antonio-alexandre

> This is a CTF about Binary Exploitation and Endianness

## Desafio: Endianness-v2 (Exploração Binária)
#### Introdução

Este desafio faz com que o usuário descubra como manipular e entender como funcionam arquivos com conceitos de [Endianness](https://www.freecodecamp.org/news/what-is-endianness-big-endian-vs-little-endian/).

Endianness é um conceito que define a ordem em que os bytes serão organizados e armazenados na memória. Ela se divide em duas opções:

- Little-endian (LSB)
- Big-endian (MSB)

Em resumo, a diferença entre os dois é qual byte virá primeiro. No Little-endian, o byte menos significativo (Less significative byte - LSB) é armazenado primeiro na memória. Já no Big-endian, o byte mais significado (Most significative byte - MSB) virá primeiro. Por exemplo:

- Tomamos o número 0x12345678 como exemplo. Em sistemas little-endian, o byte menos significativo virá primeiro. Nesse caso, seria organizado da seguinte maneira `78 56 34 12`. Já em sistemas big-endian, o byte mais significado vem primeiro. Usando o mesmo exemplo, seria organizado da seguinte forma `12 34 56 78`.

Vamos ao desafio. Aqui está o link da página no PicoCTF:

- [Página do desafio](https://play.picoctf.org/practice/challenge/415)

#### Análise Inicial

A princípio, nos declaramos com o seguinte link que nos permite baixar o arquivo a ser utilizado:

- [Link do arquivo](https://artifacts.picoctf.net/c_titan/35/challengefile)

Juntamente desse link, há a seguinte descrição:

- Aqui está um arquivo que foi recuperado de um sistema de 32 bits que organizou os bytes de uma maneira estranha. Nem temos certeza de que tipo de arquivo é.

#### Solução

As primeiras conclusões que podemos chegar, é que é apenas um arquivo comum. 

Porém, há certos procedimentos padrão que fazemos sempre que nos deparamos com um arquivo assim.

Ao executar o comando: `file challengefile`, o terminal nos retorna a seguinte mensagem: `challengefile: data`.

O próximo procedimento, é utilizar a ferramenta exiftool. Ela irá servir para verificar os [metadados](https://www.datacamp.com/pt/blog/what-is-metadata) da imagem. 

Ao executar o comando, nos deparamos com o seguinte:

[![Imagem1.png](https://i.postimg.cc/FKBjt4jn/Imagem1.png)](https://postimg.cc/gw3wR1tv)

Certo, podemos chegar a uma conclusão que será muito importante. É tratado como um arquivo JPEG.

A próxima etapa, é verificar os valores em hexadecimal do arquivo. Para isso, utilizaremos o comando: `hexeditor challengefile`. O que esse comando faz é abrir uma interface que nos possibilita visualizar os valores hexadecimais. Ao executarmos o comando, isto aparece:

[![Imagem2.png](https://i.postimg.cc/fyTZY6PD/Imagem2.png)](https://postimg.cc/WFKQvfjH)

Podemos notar uma coisa: o cabeçalho JPEG está invertido. Era para estar organizado da seguinte maneira:

- `FF D8 FF E0 00 10 4A 46 49 46 00 01`

Ou seja, podemos concluir que foi armazenado usando conversão errada de endianness, o que corrompeu o arquivo. Para isso, precisaremos inverter esses bytes. Utilizaremos o site [Cyberchef](https://cyberchef.org/). Nele, iremos utilizar a ferramenta `Swap endianness`.

[![Imagem3.png](https://i.postimg.cc/QN2vGBs1/Imagem3.png)](https://postimg.cc/MXY91pnp)

O próximo passo é salvar o arquivo como imagem. Por fim, ao abrir a imagem nos deparamos com o seguinte:

[![Imagem4.png](https://i.postimg.cc/Y0Ynq3kg/Imagem4.png)](https://postimg.cc/RJCQgwWV)

E então, descobrimos a flag que é:

>`picoCTF{cert!f1Ed_iNd!4n_s0rrY_3nDian_188d7b8c}`
 