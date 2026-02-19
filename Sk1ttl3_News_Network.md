# DuckWare Team - Sk1ttl3 News Network
###### Solved by @Antonio-alexandre

> This is a CTF about SQLInjection

## Desafio: Sk1ttl3 News Network (Exploração Web)
#### Introdução

Este desafio se trata de um site aparentemente comum de notícias sobre o mundo tech em Night City.

- [Página do desafio](https://snewspaper.discloud.app/)

#### Análise Inicial

A princípio, ao acessar o site, a questão apresenta um site comum, sem nada muito estranho.

[![image.png](https://i.postimg.cc/9QxRvJXG/image.png)](https://postimg.cc/RWHZt1HF)

Ao entrar em qualquer dessas notícias, não tem nada muito interessante, além de um pequeno IDOR, que permite interagir com o id direto pela URL.

[![image.png](https://i.postimg.cc/FszQTZqQ/image.png)](https://postimg.cc/Lhc7Xz2y)

Ao inspecionar o código fonte com o DevTools, aparece o seguinte:

[![image.png](https://i.postimg.cc/QCWJMvC3/image.png)](https://postimg.cc/SYkMTTcT)

É possível notar que há um <!--admin.php-->.

Ou seja, há acesso ao painel administrativo que a questão pede.

Ao acessar o painel, aparece o seguinte:

[![image.png](https://i.postimg.cc/MpYNxfgt/image.png)](https://postimg.cc/0zbc0jkJ)

É um painel supostamente simples, porém que não há nenhuma dica sobre login ou senha para acessar.

O primeiro passo quando se depara com um painel desse, é testar um SQLI. Ao tentar algo como:

`' --`

[![image.png](https://i.postimg.cc/136RKBnq/image.png)](https://postimg.cc/4YfTfz1f)

É possível notar que o site quebra. Ele retorna erro 500.

Quando tentamos um usuario qualquer e uma senha qualquer, ele apenas redireciona para a página novamente com código 200:

[![image.png](https://i.postimg.cc/5y6bQSW5/image.png)](https://postimg.cc/y34ww9dW)

Então, significa que será necessário um payload que consiga contornar o sistema.

Após diversas tentativas, o payload encontrado foi:

`' OR 1=1/*`

(Lembrando que `/*` em qslite é comentário.)

Mas como se obtém a informação de que é SQLite?

No código fonte da página, que aparece abrindo o devtools, há o seguinte:

`<!-- anotação: trocar sqlite por mysql qdo eu conseguir pagar por um servidor melhor! -->`

Tentando enviar o payload mencionado acima, acontece o seguinte:

[![image.png](https://i.postimg.cc/HxJTVtg9/image.png)](https://postimg.cc/9Rj6kZzD)

Enfim então a flag é encontrada!!!

>>> DUCK{SK1TTL3_N3W5_2077_SQL1NJ3CT10N_H45_B33N_H4CK3D}

### Conclusão

Este desafio se tratou de um desafio até que bem simples envolvendo SQLInjection. Foi um pouco desafiador encontrar o payload necessário para acessar o painel, pois obtive dificuldades em pensar que seria algo simples como foi. Porém, se tratou de um desafio divertido.


