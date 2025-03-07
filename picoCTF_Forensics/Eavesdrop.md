# DuckWare Team - Eavesdrop (picoCTF 2024)
###### Solved by @Antonio-alexandre

> This is a CTF about Network Traffic Analysis and Packet Sniffing

## Desafio: Eavesdrop (Sniffing de Pacotes e Análise de Tráfego de rede)
#### Introdução

Neste desafio, precisaremos analisar um arquivo de captura de tráfego de rede (.pcap), assim buscando maneiras de encontrar a flag.

- [Página do desafio](https://play.picoctf.org/practice/challenge/264)

#### Análise Inicial

A princípio, nos declaramos com o seguinte link que nos permite baixar o arquivo a ser utilizado:

- [Link do arquivo](https://artifacts.picoctf.net/c/133/capture.flag.pcap)

Logo de cara, como sabemos que se trata de um desafio de `Forensics`, é sempre bom utilizar o comando `strings -n 7 "arquivo"` (o -n 7 é para que ele mostre strings de no mínimo 7 caracteres) para analisar as strings escondidas do arquivo. Ao utilizar esse comando, isso acontece:

[![Imagem5.png](https://i.postimg.cc/d1Pq4Gw1/Imagem5.png)](https://postimg.cc/06nRNM6R)

Podemos notar a princípio três coisas:

- Trata-se de uma conversa
- Logo na quinta linha, aparece o seguinte comando: `openssl des3 -d -salt -in file.des3 -out file.txt -k supersecretpassword123`. Ele será muito importante daqui para frente.
- Ele menciona a porta 9002. Pode ser que ela seja útil.

Para esse desafio, utilizaremos a ferramenta `Wireshark` para analisar o histórico do tráfego de rede. Ao abrir a ferramenta e inserir o arquivo a ser utilizado, nos deparamos com o seguinte:

[![Imagem7.png](https://i.postimg.cc/4d6XMnx1/Imagem7.png)](https://postimg.cc/TLP85dmy)

O próximo passo, é procurar a comunicação a partir da porta 9002. Assim que encontramos, basta clicar com o botão direito, ir em Follow e TCP Stream.

[![Imagem6.png](https://i.postimg.cc/J4nW3pZx/Imagem6.png)](https://postimg.cc/t7Lfptrn)

Então, basta escolher a opção `Raw` na visualização, para visualizar como dados binários puros. E então, basta salvar o arquivo como `.des3`, que é um modelo de criptografia que foi mencionado no comando que nos deram nas strings.

[![Imagem8.png](https://i.postimg.cc/Bvhzv0wG/Imagem8.png)](https://postimg.cc/nsD2d6Jd)

E então, basta executar o comando que foi mencionado nas strings.

Ao abrir o `file.txt` que foi gerado, aparecerá a string.

[![Imagem9.png](https://i.postimg.cc/RCJYv27j/Imagem9.png)](https://postimg.cc/CdY7NP6N)

>`picoCTF{nc_73115_411_5786acc3}`
 
