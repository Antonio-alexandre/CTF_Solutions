# DuckWare Team - Matryoshka Doll (picoCTF 2024)
###### Solved by @Antonio-alexandre

> This is a CTF about Steganography, Image manipulation

## Desafio: Matryoshka Doll (Binwalk)
#### Introdução

Neste desafio, precisaremos analisar e procurar arquivos escondidos em uma imagem comum.

- [Página do desafio](https://play.picoctf.org/practice/challenge/129)

#### Análise Inicial

A princípio, nos deparamos com o seguinte link para download da imagem:

- [Link do arquivo](https://mercury.picoctf.net/static/205adad23bf9d8303081a0e71c9beab8/dolls.jpg)

Além de duas dicas:

- Espere, você pode ocultar arquivos dentro de arquivos? Mas como você os encontra?
- Certifique-se de enviar a flag como picoCTF{XXXXX}

Ao baixar o arquivo, nos deparamos com a seguinte imagem:

[![dolls.png](https://i.postimg.cc/Nffp5Vms/dolls.png)](https://postimg.cc/5jTwrssT)

Como se trata de uma boneca [Maryoshka](https://pt.wikipedia.org/wiki/Matriosca), podemos interpretar que o que buscamos está realmente "dentro" dessa imagem, escondido entre camadas.

#### Solução

A princípio, precisamos ver que tipo de arquivo se trata realmente

Para isso, utilizamos o comando: `file dolls.jpg`. Ele nos retorna isto: `dolls.jpg: PNG image data, 594 x 1104, 8-bit/color RGBA, non-interlaced`. Como podemos ver, trata-se de um arquivo jpg comum. 

Após isso, utilizamos o comando `strings -n 7 dolls.jpg` para verificar se não há nada nas strings. 

[![Imagem10.png](https://i.postimg.cc/kGRzZpsP/Imagem10.png)](https://postimg.cc/D4Fx45Rx)

Podemos notar que há sim um arquivo escondido. Para que conseguimos extraí-lo, utilizaremos a ferramenta `binwalk`. `binwalk` é uma ferramenta muito utilizada em esteganografia, análise de imagens de disco, de arquivos comprimidos ou de firmware.

Utilizaremos então o comando `binwalk -e dolls.jpg` (o -e serve para extrair automaticamente os arquivos dentro de um binário).

Esse comando nos gerou uma pasta:

[![Imagem11.png](https://i.postimg.cc/brMDYs12/Imagem11.png)](https://postimg.cc/V0qLDsGY)

Dentro dessa pasta, nos deparamos com mais uma pasta, chamada `base_images` com essa imagem dentro, chamada `2_c.jpg`:

[![Imagem12.png](https://i.postimg.cc/kGkG3LdQ/Imagem12.png)](https://postimg.cc/H89dbZnn)

Repetiremos então o mesmo processo. Executamos o comando `strings -n 7 2_c.jpg` para verificar as strings:

[![Imagem14.png](https://i.postimg.cc/x1GChspm/Imagem14.png)](https://postimg.cc/Wd3TJ7vp)

Podemos notar então que há sim outros arquivos escondidos. Então precisaremos prosseguir para o próximo passo.

Ao executar o binwalk novamente, porém alterando o nome da imagem de `dolls.jpg` para `2_c.jpg`, nos gera outra pasta.

[![Imagem13.png](https://i.postimg.cc/kgPGjrs9/Imagem13.png)](https://postimg.cc/mtVTD5K6)

Dentro dessa pasta, nos deparamos com outra imagem, porém menor: 

[![Imagem15.png](https://i.postimg.cc/KYJYv12v/Imagem15.png)](https://postimg.cc/VS0wKLv3)

Podemos então notar que estamos tendo certo progresso, pois as bonecas estão diminuindo.

Repetiremos então o processo de analisar as strings (lembrar de alterar o nome do arquivo de `2_c.jpg` para `3_c.jpg`):

[![Imagem16.png](https://i.postimg.cc/qBt2STsC/Imagem16.png)](https://postimg.cc/4ngKHjFJ)

Novamente há mais um arquivo escondido. Iremos então executar o `binwalk`. Ele nos gerou mais uma pasta com mais uma imagem:

[![Imagem17.png](https://i.postimg.cc/T3xmK5Zr/Imagem17.png)](https://postimg.cc/YhXjV029)

Novamente então iremos verificar as strings:

[![Imagem18.png](https://i.postimg.cc/bvNtTcYr/Imagem18.png)](https://postimg.cc/fVpyzP9Q)

Pera lá! Não há mais nenhuma imagem dentro. Significa que finalmente conseguimos chegar ao fim das bonecas. Executaremos então o binwalk para extrair o `.txt`. Ele nos gerou mais uma pasta:

[![Imagem19.png](https://i.postimg.cc/L4ZZvVSt/Imagem19.png)](https://postimg.cc/fkDLRmJy)

Dentro desta pasta então, finalmente está o que buscávamos. A flag!!!

[![Imagem21.png](https://i.postimg.cc/7LWfn5Qt/Imagem21.png)](https://postimg.cc/TyndR3xn)

>`picoCTF{96fac089316e094d41ea046900197662}`
 
