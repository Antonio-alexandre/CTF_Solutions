# Duckware Team
###### Solved by @Antonio-alexandre

> This is a CSRF room in TryHackMe

## Desafio: CSRF - Tryhackme
#### Introdução

Essa sala do tryhackme consiste em aprender e entender como funciona a vulnerabilidade CSRF, métodos de exploit e maneiras de se defender.

#### Task 01 - Introduction

Nessa primeira etapa, é apresentado o que a sala vai ensinar, assim como credenciais e informações relevantes para acessar a máquina virtual.

#### Task 02 - Overview of CSRF Attack

Na segunda etapa, a sala explica diretamente ao usuário a maneira que o CSRF funciona. Em resumo, consiste de uma vulnerabilidade onde o atacante consegue fazer com que o navegador execute algo em um site verificado onde o usuário está autenticado. Isso ocorre porque o navegador inclui cookies automaticamente, permitindo assim que o atacante envie requisições não autorizadas se passando pelo usuário.

Além disso, contém as seguintes perguntas:

###### 1 - Which of the following is a possible effect of CSRF? Write the correct option only.

a. Unauthorised Access

b. Exploiting Trust

c. Stealthy Exploitation

d. All of the above

R: d

Explicação:

```
a. Está correta, pois o atacante consegue acessar e controlar a ação do usuário, o que pode ocasionar em consequências graves, como perder dinheiro, reputação ou até mesmo responder por crimes.

b. Está correta pois o atacante consegue se aproveitar da "confiança" que os sites depositam sobre os usuários, permitindo assim que ele passe por cima da segurança do site.

c. Está correta pois essa é uma vulnerabilidade que ocorre de maneira silenciosa, muitas vezes onde o usuário nem percebe que está sofrendo o ataque.
```

##### 2 - Does the attacker usually know the web application requests and response format while launching a CSRF attack (yea/nay)?

R: yea

Explicação:

```Para que o usuário consiga executar o CSRF, é necessário ele entender a estrutura das requisições e das respostas do site.```

#### Task 03 - Types of CSRF Attack

Nessa etapa, é apresentado dois tipos de ataques CSRF. São eles:

- CSRF convencional: Este tipo concentra-se em ações que alteram o estado da aplicação (como mudar senhas ou realizar transações), enganando a vítima para que envie um formulário malicioso. O navegador do usuário envia automaticamente os cookies de autenticação, explorando a confiança do servidor na solicitação que parece legítima.

- CSRF assíncrono (XMLHttpRequest): Comum em aplicações web dinâmicas que utilizam JavaScript, XMLHttpRequest (AJAX) ou Fetch API, este ataque inicia operações sem a recarga completa da página. O atacante usa scripts maliciosos para fazer uma chamada assíncrona (como uma solicitação POST) em nome do usuário autenticado para realizar ações indesejadas, como alterar as configurações de e-mail, por exemplo.

Também é mencionado o Flash-based CSRF, que consistia de uma técnica que aproveitava falhas de segurança nos componentes do Adobe Flash Player para executar ataques CSRF. 

Embora o Flash tenha sido historicamente usado para conteúdo interativo, streaming de vídeo e animações, vulnerabilidades em seu design permitiram que arquivos maliciosos (.swf) postados no site de um atacante enviassem solicitações não autorizadas a outros sites, explorando a autenticação do usuário. 

No entanto, o suporte ao flash player foi encerrado em 31 de dezembro de 2020.

Contém duas perguntas:

##### 1 - What is usually the extension of a malicious flash file used during a CSRF attack?

R: .swf


Explicação: `Em ataques CSRF Flash-based, era injetado arquivos maliciosos .swf (flash file) na aplicação.`

##### 2 - Which type of CSRF exploitation is carried out when operations are initiated without a complete page request-response cycle?

R: Asynchronous

Explicação: `Essa questão fala sobre o tipo de exploit CSRF onde as aplicações não precisam de um ciclo completo de requisição-resposta. Isso é característica de CSRF assíncronos.`


### Task 04 - Basic CSRF - Hidden Link/Image Exploitation

Nessa etapa, é apresentado o CSRF por exploração de imagem/link escondido. Em resumo, ele consiste de uma técnica sorrateira de ataque onde um invasor utiliza engenharia social para induzir um usuário autenticado a clicar em um link ou a carregar uma imagem minúscula (0x0 pixel) em uma página maliciosa. 

A URL de destino do link/imagem é uma ação indesejada no site alvo (como uma transferência bancária) e, como o navegador da vítima inclui automaticamente os cookies de sessão, o servidor do site executa a ação forjada. 

A falha reside na falta de validação por parte do servidor de que a solicitação é legítima, e a solução principal é a implementação de Tokens CSRF (Cross-Site Request Forgery), que são valores únicos e ocultos que o servidor espera receber e validar em cada requisição para garantir que ela se originou de uma fonte confiável.

Ela contém três perguntas:

##### 1 - What is the flag value after a successful transfer from Josh's account?

R: THM{SUCCESSFUL_ATTACK}

Explicação: `Para essa questão, basta acessar a caixa de entrada do Josh (descrita no texto da sala) e acessar o e-mail de phishing (está com tag de inseguro): Congratulations! You Might Be Our Lucky Winner.`

[![image.png](https://i.postimg.cc/2SZg6yQB/image.png)](https://postimg.cc/75qKtxQx)

[![image.png](https://i.postimg.cc/sXwN8hrT/image.png)](https://postimg.cc/7GJ9Jbbz)


##### 2 - What is the flag value once the CSRF attack is detected?

R: THM{INVALID_CSRF_TOKEN}

Explicação: `Para essa questão, basta acessar o e-mail considerado seguro com a mensagem: Congratulations! You Might Be Our Lucky Winner.`

[![image.png](https://i.postimg.cc/2SZg6yQB/image.png)](https://postimg.cc/75qKtxQx)

[![image.png](https://i.postimg.cc/9FvNv2vs/image.png)](https://postimg.cc/xc3RzwjP)

##### 3 - Does the hidden image exploitation require the img tag's src attribute to be linked toward a legitimate image file (yea/nay)?

R: nay

Explicação: `Não precisa necessariamente ser uma imagem legítima, mas as vezes só o atributo com algum pixel ou algo do tipo.`

### Task 05 - Double Submit Cookie Bypass

Nessa quinta etapa, é apresentado o bypass de token CSRF. Resumidamente, apesar da implementação de Tokens CSRF (usando a técnica Double Submit Cookies, onde o token no cookie deve corresponder ao token no campo oculto do formulário) fortalecer a segurança, atacantes podem explorá-la se a geração do token for previsível ou se outras vulnerabilidades (como XSS ou falhas na política Same-Origin) existirem.

O cenário de ataque demonstra o bypass da defesa quando o token é gerado de forma insegura (facilmente decodificado para um dado previsível, como o número da conta). O atacante faz o seguinte:

- Engenharia Reversa do Token: Descobre que o token CSRF é o número da conta do usuário codificado em Base64.

- Injeção de Cookie em Subdomínio: Usa um subdomínio controlado (attacker.mybank.thm) para injetar um cookie CSRF legítimo no navegador da vítima, contendo o valor Base64 do número da conta da vítima.

- Execução do Payload: Engana a vítima (Josh) para clicar em um e-mail de engenharia social. Esse link direciona para uma página do atacante que contém um formulário de mudança de senha com o token no campo oculto (o mesmo valor Base64 injetado no cookie). O formulário é auto-submetido para a URL do banco.

Como o servidor valida apenas se o cookie e o valor do formulário correspondem (sem validar a origem segura do token), o ataque é bem-sucedido, permitindo que o atacante mude a senha e assuma o controle total da conta. A mitigação essencial é garantir que a geração do token CSRF use algoritmos criptograficamente fortes e imprevisíveis.

Além disso, contém algumas perguntas:

##### 1 - What is the decoded value of the CSRF token for Josh's account?

R: GB82MYBANK5699

Explicação: `É necessário logar na conta do Josh em mybank.thm:8080. Após isso, abrir os devtools (F12) e traduzir o valor de csrf-token que está em base64.`

[![image.png](https://i.postimg.cc/nrnnK716/image.png)](https://postimg.cc/gXB9mxQD)

[![image.png](https://i.postimg.cc/FzPXj6zf/image.png)](https://postimg.cc/9wqk2JGV)

##### 2 - What is the updated password for Josh's account?

R: GB82MYBANK5697

Explicação: `Ao entrar no link falso de troca de senha, ele mostra a opção de salvar a senha, e lá está a senha atualizada.`

[![image.png](https://i.postimg.cc/4ds3Prf8/image.png)](https://postimg.cc/SX1qN1z8)

##### 3 - What is the flag value if someone clicks on malicious links after the IT team of MyBank has successfully employed a random token generation algorithm?

R: THM{SECURED_CSRF}

Explicação: `Ao entrar no link de troca de senha que está seguro, a flag aparece na tela.`

[![image.png](https://i.postimg.cc/pdtVVgFt/image.png)](https://postimg.cc/ts2G2vkr)

##### 4 - What is the hidden field name that the MyBank IT team added to avoid CSRF attacks?

R: csrf_token

Explicação: `Como podemos ver na questão 01, csrf_token é o token que é gerado.`

[![image.png](https://i.postimg.cc/nrnnK716/image.png)](https://postimg.cc/gXB9mxQD)

##### 5 - Is it technically possible to hijack a session cookie in a web application (yea/nay)?

R: yea

Explicação: `Sim, há vários métodos para se obter um cookie de sessão, como XSS por exemplo.`

### Task 06 - Samesite Cookie Bypass

Nesta etapa, é apresentado o atributo SameSite. Em resumo, ele se trata de um mecanismo de defesa crucial que controla quando os cookies são enviados em solicitações cross-site, ajudando a prevenir ataques CSRF e vazamentos de dados cross-origin.

###### Tipos de Atributos SameSite: 

- `Strict`: Oferece o nível mais alto de proteção. O cookie só é enviado se a solicitação for proveniente da mesma origem que o definiu (contexto first-party), bloqueando efetivamente todas as solicitações cross-site.

- `Lax`: Fornece um nível moderado de proteção. Permite que o cookie seja enviado apenas em navegações de nível superior (ex: clicar em um link) e com métodos HTTP seguros como GET, HEAD e OPTIONS. Ele impede o envio de cookies em solicitações POST cross-site.

- `None`: Comportamento de "globo-trotter". Permite que o cookie seja enviado em solicitações first-party e cross-site. Requer o atributo Secure (HTTPS) para prevenir riscos de segurança durante o trânsito.

###### Cenários de Exploração do SameSite=Lax:

Apesar de ser uma defesa, a configuração Lax pode ser explorada:

- `Exploração Lax (GET Request)`: Se um cookie sensível (como um cookie de logout) estiver definido como Lax, um atacante pode usar engenharia social para fazer a vítima clicar em um link (tag) que executa uma solicitação GET para a ação alvo (ex: logout.php). Como se trata de uma navegação de nível superior, o navegador envia o cookie Lax, forçando o logout da vítima.

- `Exploração Lax + POST (Janela de 2 Minutos)`: O padrão moderno do Chrome trata cookies sem um atributo SameSite explícito como Lax. No entanto, ele faz uma exceção de 2 minutos logo após a criação ou modificação de um cookie sem o atributo SameSite, permitindo que ele seja enviado em solicitações POST cross-site. Um atacante pode encadear ações (ex: forçar o logout para atualizar um cookie dentro da janela de 2 minutos) e, em seguida, fazer uma solicitação POST maliciosa (ex: para modificar outro cookie), explorando essa janela de tempo antes que a restrição Lax entre em vigor.

Além disso, contém um passo a passo detalhado de como botar em prática no contexto da sala. Esta etapa possui quatro questões:

##### 1 - What is the logout cookie value?

R: 7kRt2x9LpQyW

Explicação: `Basta acessar a conta do banco do Josh, e abrir o DevTools (F12). Lá vai estar o valor do cookie.`

[![image.png](https://i.postimg.cc/6pFc8hWB/image.png)](https://postimg.cc/Hr4XKX8N)

##### 2 - What is the flag value after successfully logging out Josh from the mybank.thm application?

R: THM{LOGGED_OUT}

Explicação: `Basta acessar o link da pesquisa para ter chance de ganhar uma ferrari que está no email. A flag irá aparecer na tela.`


[![image.png](https://i.postimg.cc/zvfKtLzg/image.png)](https://postimg.cc/WFQdt1JN)

[![image.png](https://i.postimg.cc/zfnCVZ2d/image.png)](https://postimg.cc/WD1FHKvr)

##### 3 - Once logged into the mybank.thm:8080 application, change the logout cookie value to hellothm, and try to click on the malicious link. What is the flag value after detecting a CSRF attack?

R: THM{ATTACK_DETECTED}

Explicação: `Ao trocar o valor do cookie (nos DevTools) e tentar abrir o link, a flag aparece na tela.`

[![image.png](https://i.postimg.cc/4ynHvrPM/image.png)](https://postimg.cc/N2q0BCW7)

[![image.png](https://i.postimg.cc/26kZyqBt/image.png)](https://postimg.cc/Y4P9yCRN)


#### 4 - After updating the isBanned cookie value to true through a CSRF attack, what is the flag value?

R: THM{USER_IS_B@NNED}

Explicação: `Para essa questão, é necessário alterar o valor do cookie isBanned para true, no site que deu logout através de CSRF. Após isso, basta acessar o email com o cenário de teste que a flag aparecerá.`

[![image.png](https://i.postimg.cc/YqYYXwkR/image.png)](https://postimg.cc/2Vj16M1L)

[![image.png](https://i.postimg.cc/ZnyK5ZjX/image.png)](https://postimg.cc/sGsRTkkJ)

### Task 07 - Few Additional Exploitation Techniques

Nesta etapa, é ensinado que CSRF em um contexto AJAX ocorre quando o navegador do usuário autenticado é enganado para enviar uma requisição assíncrona (via XMLHttpRequest ou Fetch API) a um site confiável, executando uma ação não intencional, como atualizar uma senha. 

Embora a Política de Mesma Origem (SOP) normalmente restrinja requisições cross-origin, o CSRF pode ter sucesso ao explorar o fato de que o navegador ainda anexa os cookies de autenticação nas requisições, mesmo que sejam bloqueadas por SOP/CORS.

###### Bypass de SOP/CORS e Header Referer

- `Bypass de SOP/CORS`: O ataque pode ser facilitado por configurações inadequadas do CORS no lado do servidor, como definir o cabeçalho Access-Control-Allow-Origin: * (permitindo requisições de qualquer origem). Embora essa configuração possa ser necessária para APIs públicas, ela se torna uma vulnerabilidade de CSRF se não for acompanhada de medidas anti-CSRF adequadas. O uso de Access-Control-Allow-Origin: * e Access-Control-Allow-Credentials: true é estritamente proibido pela especificação CORS por motivos de segurança.

- `Header Referer Bypass`: Alguns sites tentam impedir o CSRF verificando o cabeçalho Referer para garantir que a requisição venha do próprio domínio. No entanto, essa defesa é fraca, pois o cabeçalho Referer pode ser manipulado ou omitido por extensões de navegador, ferramentas de privacidade ou configurações de meta tags do site.

Contém algumas questões:

##### 1 - What is the general policy name that forbids cross-origin requests?

R: same-origin policy

Explicação: Como está no resumo acima, a SOP proíbe as requisições cross-origin.

##### 2 - Strictly speaking, while updating an email address of Josh through the API endpoint, is Access-Control-Allow-Origin: * a vulnerable header (yea/nay)?

R: yea

Explicação: No contexto do Josh, se a requisição possuir um header em que o domínio bata com o do site, ele permite que a requisição seja autorizada.

##### 3 - What is the header called that stores the URL of the last page the user visited before making a request?

R: refererer

Explicação: Esse header referencia a URL da última página que o usuário acessou.

### Task 08 - Defence Mechanisms

Nesta etapa, é ensinado maneiras de se defender. O CSRF desempenha um papel crítico para pentesters, permitindo simular ataques onde usuários executam ações não autorizadas em sites confiáveis. 

Explorar o CSRF avalia a eficácia das defesas e identifica falhas no gerenciamento de sessão e nas medidas anti-CSRF.

###### Maneiras de se defender: 

- `Pentesters/Red Teamers`: Devem realizar Testes Ativos de CSRF, avaliar mecanismos de Validação de Limite (garantindo que tokens anti-CSRF estejam presentes e verificados), analisar Headers de Segurança (como CORS e Referer) e examinar o Gerenciamento de Sessão.

- `Secure Coders`: Devem integrar Tokens Anti-CSRF únicos e imprevisíveis em todas as requisições. Devem também definir o atributo SameSite dos cookies como 'Strict' ou 'Lax', implementar uma Política de Referrer (Referrer Policy) rigorosa, utilizar Content Security Policy (CSP) para mitigar injeção de scripts, e considerar o uso do Padrão Double-Submit Cookie e CAPTCHAS como camadas adicionais de defesa.

Contém apenas uma questão:

##### 1 - Is it a good practice to keep the anti-CSRF token predictable so that secure coders can quickly implement them? (yea/nay)

R: nay

Explicação: Se deixar o token muito previsível, facilitará que atacantes consigam descobrir o padrão e invadir mais usuários.

### Conclusão

Nesta sala, é ensinado temas sobre CSRF, que são de suma importância, pois é uma vulnerabilidade crítica e difícil de ser detectada, por ser silenciosa.

