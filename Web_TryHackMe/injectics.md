# Duckware Team - injectics
###### Solved by @Antonio-alexandre

> This is a injection attack room in TryHackMe

## Desafio: injectics - Tryhackme
#### Introdução

Essa sala do tryhackme consiste em utilizar de habilidades de pentesting para executar injections em uma aplicação web.

A princípio, se trata de uma página comum, com login e registro, além de outras abas como Eventos, Atletas, entre outras:

[![image.png](https://i.postimg.cc/jjcGsXM6/image.png)](https://postimg.cc/VSS7gjQ5)

#### Primeiros passos

O primeiro passo executado, foi utilizar o nmap para mapear as portas disponíveis na aplicação, através do seguinte comando:

`nmap -A -T4 IP`

```
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 22:42:db:22:36:6f:3f:7f:95:51:90:33:3b:27:01:bd (RSA)
|   256 cd:29:83:02:66:b4:49:54:02:02:66:67:79:c7:e1:1b (ECDSA)
|_  256 aa:6d:74:16:7f:33:e3:f5:d1:4b:1f:35:f1:07:42:7c (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Injectics Leaderboard
```

Após isso, para obter mais informações, foi utilizado o dirbuster para verificar quais diretórios estavam disponíveis. Para isso, utiliza-se o comando:

`dirsearch -u http://IP/`

```
[22:42:36] 301 -  311B  - /js  ->  http://10.80.137.235/js/                 
[22:42:42] 403 -  278B  - /.ht_wsr.txt                                      
[22:42:42] 403 -  278B  - /.htaccess.bak1                                   
[22:42:42] 403 -  278B  - /.htaccess.orig                                   
[22:42:42] 403 -  278B  - /.htaccess.save
[22:42:42] 403 -  278B  - /.htaccess.sample
[22:42:42] 403 -  278B  - /.htaccess_extra                                  
[22:42:42] 403 -  278B  - /.htaccess_sc                                     
[22:42:42] 403 -  278B  - /.htaccess_orig
[22:42:42] 403 -  278B  - /.htaccessOLD
[22:42:42] 403 -  278B  - /.htaccessOLD2                                    
[22:42:42] 403 -  278B  - /.htaccessBAK                                     
[22:42:42] 403 -  278B  - /.htm
[22:42:42] 403 -  278B  - /.html
[22:42:42] 403 -  278B  - /.htpasswds                                       
[22:42:42] 403 -  278B  - /.httr-oauth                                      
[22:42:42] 403 -  278B  - /.htpasswd_test                                   
[22:42:46] 403 -  278B  - /.php                                             
[22:43:28] 200 -   48B  - /composer.json                                    
[22:43:28] 200 -    9KB - /composer.lock                                    
[22:43:32] 301 -  312B  - /css  ->  http://10.80.137.235/css/               
[22:43:33] 302 -    0B  - /dashboard.php  ->  dashboard.php                 
[22:43:44] 301 -  314B  - /flags  ->  http://10.80.137.235/flags/           
[22:43:55] 301 -  319B  - /javascript  ->  http://10.80.137.235/javascript/ 
[22:43:56] 403 -  278B  - /js/                                              
[22:43:59] 200 -    1KB - /login.php                                        
[22:44:22] 302 -    0B  - /logout.php  ->  index.php                        
[22:44:01] 200 -    1KB - /mail.log                                         
[22:44:15] 301 -  319B  - /phpmyadmin  ->  http://10.80.137.235/phpmyadmin/ 
[22:44:19] 200 -    3KB - /phpmyadmin/doc/html/index.html                   
[22:44:19] 200 -    3KB - /phpmyadmin/                                      
[22:44:19] 200 -    3KB - /phpmyadmin/index.php
[22:44:29] 403 -  278B  - /server-status                                    
[22:44:29] 403 -  278B  - /server-status/                                   
[22:44:44] 403 -  278B  - /vendor/                                          
[22:44:44] 200 -    0B  - /vendor/composer/autoload_classmap.php            
[22:44:44] 200 -    1KB - /vendor/composer/LICENSE
[22:44:44] 200 -    0B  - /vendor/composer/autoload_static.php
[22:44:44] 200 -    0B  - /vendor/composer/ClassLoader.php
[22:44:44] 200 -    0B  - /vendor/composer/autoload_files.php
[22:44:44] 200 -    0B  - /vendor/composer/autoload_psr4.php
[22:44:44] 200 -    0B  - /vendor/composer/autoload_namespaces.php          
[22:44:44] 200 -    0B  - /vendor/composer/autoload_real.php
[22:44:44] 200 -    0B  - /vendor/autoload.php
[22:44:44] 200 -   12KB - /vendor/composer/installed.json 
```

Após isso, um passo crucial é analisar o código fonte da página.

Ao abrir, é possível encontrar o seguinte:

```
<!-- Website developed by John Tim - dev@injectics.thm-->

<!-- Mails are stored in mail.log file-->
    <!-- Bootstrap JS and dependencies -->
```

Isso significa que está exposto o arquivo log de e-mails. Acessando esse arquivo:

```
From: dev@injectics.thm
To: superadmin@injectics.thm
Subject: Update before holidays

Hey,

Before heading off on holidays, I wanted to update you on the latest changes to the website. I have implemented several enhancements and enabled a special service called Injectics. This service continuously monitors the database to ensure it remains in a stable state.

To add an extra layer of safety, I have configured the service to automatically insert default credentials into the `users` table if it is ever deleted or becomes corrupted. This ensures that we always have a way to access the system and perform necessary maintenance. I have scheduled the service to run every minute.

Here are the default credentials that will be added:

| Email                     | Password 	              |
|---------------------------|-------------------------|
| superadmin@injectics.thm  | superSecurePasswd101    |
| dev@injectics.thm         | devPasswd123            |

Please let me know if there are any further updates or changes needed.

Best regards,
Dev Team

dev@injectics.thm
```

Nesse e-mail, contém as senhas padrão, e também deixa explícito que se algo ocorrer com a tabela `users`, todos os usuários voltam às senhas padrão.

Outra informação útil, é a de que o sistema utiliza mysql, pois o `/phpmyadmin` utiliza mysql como db padrão.

Outro ponto interessante, é ao acessar o endpoint `/composer.json`, pois ele deixa explícito que está sendo utilizado twig nessa aplicação:

```
require
twig/twig "2.14.0"
```

##### What is the flag value after logging into the admin panel?

Para essa questão, é necessário acessar o painel de admin. Como os e-mails de admin já foram descobertos, o próximo passo é tentar acessar de alguma outra maneira. Nesse caso, foi utilizado técnicas de SQLI. 

Bastou então abrir o BurpSuite, interceptar a requisição da página de login, e executar o seguinte:

[![image.png](https://i.postimg.cc/mkrsCn5R/image.png)](https://postimg.cc/34zVHthb)

Fazendo isso, ele redireciona para o dashboard.php:

[![image.png](https://i.postimg.cc/9FKhwJt0/image.png)](https://postimg.cc/1fGTL0d1)

Como procedimento padrão, já que se trata de SQLI, o próximo passo foi tentar editar qualquer tabela e enviar um payload para ver como ele se comporta:

[![image.png](https://i.postimg.cc/Jn4g2kcf/image.png)](https://postimg.cc/2qM2q3tG)

Como ele deu erro por conta de validação, mudei o payload para o seguinte:

`1; SELECT 1;`

E ele me redirecionou para o dashboard novamente.

Basta então abrir novamente a página de edição, e digitar o seguinte em algum dos campos:

`1; DROP table users;`

Ele retorna o seguinte:

`Seems like database or some important table is deleted. InjecticsService is running to restore it. Please wait for 1-2 minutes.`

Basta então tentar acessar o painel de admin com a senha padrão que o e-mail continha, obtendo assim a primeira flag:

[![image.png](https://i.postimg.cc/zff6dhk2/image.png)](https://postimg.cc/xXW68chL)

##### What is the content of the hidden text file in the flags folder?

Para a segunda flag, como a aplicação utiliza o twig, e sabendo que ele é vulnerável a SSTI, basta acessar a aba profile, para editar o perfil, e realizar um teste simples como `{{7*7}}`:

[![image.png](https://i.postimg.cc/Hn1SC8bq/image.png)](https://postimg.cc/9R1PdM8b)

O próximo passo, foi pesquisar e tentar encontrar algum payload que exibisse o texto contido no documento /flags.

Para isso, o payload utilizado foi o seguinte:

[![image.png](https://i.postimg.cc/C1kNJMWy/image.png)](https://postimg.cc/8JkMzGpw)

Fazendo tudo isso, obtém-se a segunda flag.

#### Conclusão

##### 1 - Vulnerabilidades encontradas:

* Exposição de Informações Sensíveis e Logs: A presença de arquivos como mail.log e composer.json acessíveis publicamente permitiu a descoberta de credenciais administrativas padrão e das versões exatas das bibliotecas utilizadas (Twig 2.14.0).

* SQL Injection (SQLi): O formulário de login e os campos de edição no dashboard não tratavam adequadamente os caracteres especiais, permitindo o desvio de autenticação (Authentication Bypass) e a execução de comandos destrutivos, como o DROP TABLE, que comprometeu a integridade do banco de dados.

* Server-Side Template Injection (SSTI): Devido ao uso de uma versão vulnerável do motor de renderização Twig, foi possível injetar expressões que o servidor executou diretamente. Isso permitiu a leitura de arquivos locais do sistema operacional através do perfil do usuário.

##### 2 - Como Remediar:

* Implementar Prepared Statements (Consultas Parametrizadas): Para impedir SQLi, o código deve separar os dados da instrução SQL. Em PHP, deve-se utilizar PDO ou MySQLi com bind parameters, garantindo que entradas de usuários nunca sejam concatenadas diretamente na query.

* Sanitização e Escapamento de Templates: Para corrigir o SSTI, deve-se configurar o Twig para não permitir funções perigosas de execução de código ou leitura de arquivos dentro dos templates. É fundamental atualizar o Twig para versões que corrigem escapes de contexto e implementar uma sandbox rigorosa.

* Arquivos de log (mail.log) e manifestos de dependências (composer.json) devem ser movidos para fora do diretório raiz do servidor web ou protegidos via regras no .htaccess.

* Desativar a listagem de diretórios no servidor Apache para evitar a descoberta de arquivos sensíveis.

* Princípio do Menor Privilégio: O usuário do banco de dados utilizado pela aplicação web não deve ter permissões para deletar tabelas (DROP). As permissões devem ser restritas apenas ao necessário para o funcionamento do sistema (SELECT, INSERT, UPDATE).