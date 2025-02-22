# DuckWare Team - m00nwalk2 (picoCTF 2024)
###### Solved by @Antonio-alexandre

> This is a CTF about Steganography, Signal transmission

## Desafio: m00nwalk2 (Esteganografia, Transmissão de sinais)
#### Introdução

Neste desafio, precisaremos analisar e procurar mensagens escondidas em arquivos de áudio com informações desconhecidas, utilizando de técnicas e meios para interpretá-las.

- [Página do desafio](https://play.picoctf.org/practice/challenge/28)

#### Análise Inicial

A princípio, nos deparamos com a seguinte descrição: 

- Revisite a última transmissão. Achamos que esta [transmissão](https://jupiter.challenges.picoctf.org/static/a33c9e5dae30c560704e6f2ffaba35c7/message.wav) contém uma mensagem oculta. Existem também algumas pistas: [pista 1](https://jupiter.challenges.picoctf.org/static/a33c9e5dae30c560704e6f2ffaba35c7/clue1.wav), [pista 2](https://jupiter.challenges.picoctf.org/static/a33c9e5dae30c560704e6f2ffaba35c7/clue2.wav), [pista 3](https://jupiter.challenges.picoctf.org/static/a33c9e5dae30c560704e6f2ffaba35c7/clue3.wav).

Além da seguinte dica:

- Use as pistas para extrair a outra flag do arquivo .wav

#### Solução

Logo ao acessar os links, recebemos quatro arquivos de áudio: message.wav, clue1.wav, clue2.wav e clue3.wav.

Ao acessá-los, apenas um som desconhecido e estranho toca. Porém, precisamos entender do que se trata e como prosseguir.

O termo `moonwalk` pode ser interpretado como o ato de `andar na lua` do homem. Partindo desse pressuposto, descobrimos que as imagens que foram transmitidas na operação `Apollo 11` utilizaram um método chamado `SSTV` (acrônimo para `slow-scan television` ou `televisão de varredura lenta`). Precisamos então entender como esse método funciona. Encontramos através de pesquisa a seguinte informação:

> O SSTV, ou Slow Scan Television, é uma técnica única que permite a transmissão de imagens estáticas por rádio. Ao contrário da televisão tradicional que transmite múltiplos quadros por segundo, o SSTV envia uma única imagem em um período de tempo, convertendo essa imagem em tons de áudio para a transmissão. Esta técnica é amplamente utilizada em comunicações de rádio de longa distância, especialmente onde a largura de banda é limitada.

Precisamos então de uma maneira para transmitir a imagem. Usaremos então dois softwares, um chamado `QSSTV` e o outro `Pavucontrol`. O `QSSTV` serve para receber e transmitir `SSTV` e `HAMDRM` (também conhecido como `DSSTV`). Já o `Pavucontrol` servirá para tocar o áudio e transmití-lo para o `QSSTV`. Precisaremos então configurá-los.

Para instalar o `QSSTV`, utilizamos o comando : `sudo apt-get install qsstv`

Basta então abrir ambos os dois e executar o seguinte comando: `pactl load-module module-null-sink sink_name=virtual-cable`
Em resumo, o que esse comando faz é conectar com o servidor de som `PulseAudio`, criar um `módulo nulo` (um dispositivo virtual que serviria como `buraco negro` de áudio), e atribuir o nome `virtual-cable`. 

Após executar esse comando, podemos notar que ele criou um dispositivo no `Pavucontrol`:

[![Imagem20.png](https://i.postimg.cc/dtwr1yMG/Imagem20.png)](https://postimg.cc/JtYGYtzh)

Feito isso, voltaremos ao `QSSTV`. No `QSSTV`, iremos em: `Options -> Configuration -> Sound`, e selecionaremos a opção: `PulseAudio`.

[![Imagem22.png](https://i.postimg.cc/4xz3h8zd/Imagem22.png)](https://postimg.cc/pmLvwJcb)

Feito isso, retornaremos ao `Pavucontrol`. Na opção `Recording`, selecionaremos que o `QSSTV` precisa capturar áudio do dispositivo `virtual-cable`:

[![Imagem23.png](https://i.postimg.cc/sgXr3wwq/Imagem23.png)](https://postimg.cc/SJw5r70f)

Após isso, voltaremos ao `QSSTV`, selecionaremos a opção `Autoslant` e trocaremos o mode para `Auto`:

[![Imagem24.png](https://i.postimg.cc/nLDNL57R/Imagem24.png)](https://postimg.cc/mcTmXmmC)

Com tudo isso feito, executaremos o comando: `paplay -d virtual-cable message.wav` para reproduzir o áudio `message.wav` no `Pavucontrol`.

[![Imagem25.png](https://i.postimg.cc/nzK54D3q/Imagem25.png)](https://postimg.cc/vgB3McFm)

Como podemos ver, há uma flag:

> picoCTF{beep_boop_im_in_space}

No entanto, essa flag não funciona caso eu tente inserí-la no picoCTF. Então, executaremos novamente o comando para cada um dos outros arquivos:

> clue1.wav:

[![Imagem26.png](https://i.postimg.cc/Jzy2TgTr/Imagem26.png)](https://postimg.cc/DST60CkH)

> clue2.wav: 

[![Imagem27.png](https://i.postimg.cc/gJKSXP0D/Imagem27.png)](https://postimg.cc/S2J733C2)

> clue3.wav:

[![Imagem28.png](https://i.postimg.cc/yYxQdtCm/Imagem28.png)](https://postimg.cc/dLMmNW13)

Todas as imagens são dicas de como prosseguir. A primeira, nos mostrou uma senha: hidden_stegosaurus. É uma referência nítida a [Esteganografia](https://pt.wikipedia.org/wiki/Esteganografia).

Analisando a terceira imagem, verificamos que há o seguinte texto: `Alan Eliasen the FutureBoy`. Pesquisando isso, nos deparamos com o seguinte site:

[![Imagem29.png](https://i.postimg.cc/CxfcC81d/Imagem29.png)](https://postimg.cc/87NdNjLV)

Neste site, há várias ferramentas. Porém, dentre elas há uma que nos será útil: a de `Esteganografia`. Então, nos deparamos com a seguinte tela:

[![Imagem30.png](https://i.postimg.cc/d3x9rQc8/Imagem30.png)](https://postimg.cc/D41sKTZZ)

Podemos enviar o audio `message.wav` para o site, utilizando a senha que nos fora fornecida anteriormente. 

Ao inserir todas as informações, ele nos retorna a `flag`.

>`picoCTF{the_answer_lies_hidden_in_plain_sight}`
 
