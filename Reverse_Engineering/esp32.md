# Duckware Team 
###### Solved by @Antonio-alexandre

> This CTF is about esp32 firmware reverse engineering

## Desafio: ESP32 Rev (Reverse Engineering, Code Analysis)
#### Introdução

Esse desafio consiste em extrair o firmware de uma placa esp32, e obter a flag escondida

#### Análise Inicial

Ao receber a placa física, o primeiro passo executado foi extrair o firmware diretamente da placa. Para isso, foi utilizado uma ferramenta chamada [esptool](https://github.com/espressif/esptool). Nessa ferramenta, foi utilizado o seguinte comando:

`python esptool.py --chip esp32 --port COM6 --baud 115200 read_flash 0x00000 0x400000 firmware.bin`

Após extrair o firmware, o próximo passo é analisar ele. Para isso, basta abrir no bloco de notas.

Ao executar esse passo e analisar mais a fundo o firmware, é possível concluir que ele gera uma rede Wi-Fi que é necessário conectar. Além disso, ele gera um site em `192.168.4.1`. 

Continuando a análise do firmware, a seguinte parte chama bastante atenção:

```
doctype html><html><head><meta charset='utf-8'><title>Login</title></head><body><h3>Login</h3><form action='/login' method='POST'>UsuÃ¡rio: <input type='text' name='user'><br>Senha: <input type='password' name='pass'><br><br><input type='submit' value='Entrar'></form></body></html> text/html <!doctype html><html><head><meta charset="utf-8"><title>Duckware_ExercÃ­cio</title></head><body><p>Se vocÃª estÃ¡ vendo isso Ã© porque estÃ¡ conectado na wifi.</p><p>    </p><p>Se conseguiu logar, Ã© mÃ©rito. Cada etapa exige atenÃ§Ã£o, preparo e execuÃ§Ã£o correta. Quem passa, passa porque fez o necessÃ¡rio - simples assim.</p><p><a href='/login'>Ir para login</a></p></body></html> 404 - Not Found text/plain %02x %02X:%02X:%02X:%02X:%02X:%02X [AP] Cliente conectado:  [AP] Cliente desconectado:  %2x pass Erro interno Login invÃ¡lido Erro ao iniciar AP AP iniciado. SSID:  IP do AP:  /login cff680d996115576ccad92f0a4bed8fcf9c5b4b99f4c3a2e9a918993fa86efbee8fef0e590 8ba1c38ded7f6546aef2e2a294e1b0c8 adminuser mT4sR8wKd02Z Curso-Hardware-Hacking-2 ,CH: ,OPEN ,WEP ,WPA_PSK ,WPA2_PSK ,WPA_WPA2_PSK ,WEAP ,WPA3_PSK ,WPA2_WPA3_PSK ,WAPI_PSK ,OWE ,WPA3_ENT_SUITE_B_192_BIT ,STA: ,RSSI: LR ,EAP arduino_events vector::_M_realloc_insert vector::_M_realloc_append  <UP  <DOWN DHCPC _OFF DHCPS ,AUTOUP ,GARP ,IP_MOD ,PPP ,BRIDGE ,V6_REP  PRIO:  ether  inet   netmask   broadcast  gateway   dns  inet6   type  GLOBAL LINK_LOCAL SITE_LOCAL UNIQUE_LOCAL IPV4_MAPPED_IPV6 esp32- %s%02X%02X%02X last bool WebServer::_collectHeader(const char*, const char*) /C:\Users\diogo\AppData\Local\Arduino15\packages\esp32\hardware\esp32\3.3.1\libraries\WebServer\src\Parsing.cpp 0x00 Content-Disposition blob 
```

Nessa parte, o seguinte trecho é o que será útil:

```
IP do AP:  /login cff680d996115576ccad92f0a4bed8fcf9c5b4b99f4c3a2e9a918993fa86efbee8fef0e590 8ba1c38ded7f6546aef2e2a294e1b0c8 adminuser mT4sR8wKd02Z Curso-Hardware-Hacking-2
```

Neste trecho, há as credenciais de login da rede Wi-Fi, e credenciais de login para algo que ainda não foi descoberto.

Conectando no Wi-Fi e acessando `192.168.4.1`, aparece a seguinte tela:

[![image.png](https://i.postimg.cc/zvMCWfcd/image.png)](https://postimg.cc/QKQKZjs5)

Ao clicar no link destacado, ele redireciona para o endpoint `/login`:

Acessando o link, a seguinte tela aparece:

[![image.png](https://i.postimg.cc/50XqtvP1/image.png)](https://postimg.cc/qhdCbtVZ)

Nessa tela, é necessário um login contendo usuário e senha. O usuário já foi descoberto, é `adminuser`. Porém, a senha é um hash. Para obter a senha, foi utilizado o site [hashes.com](https://hashes.com/en/decrypt/hash).

No site, basta colar o hash adquirido, e clicar em Submit & Search. Fazendo isso, obtém-se a seguinte resposta:

[![image.png](https://i.postimg.cc/qqFjxrRp/image.png)](https://postimg.cc/2L4QCPht)

Conclui-se então que o usuário e a senha são os seguintes:

```
Usuário: adminuser
Senha: correta
```

Acessando assim, a seguinte flag aparecerá:

```
DWCT{n00b_pR0_h4rdw4r3_h4ck1ng_vc_3h}
```