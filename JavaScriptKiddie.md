# DuckWare Team - Java Script Kiddie (picoCTF 2024)
###### Solved by @Antonio-alexandre

> This is a CTF about SQL Injection, Web Exploitation

## Desafio: Java Script Kiddie (Exploração Web)
#### Introdução

Este desafio faz com que o usuário entenda como códigos estruturados em JavaScript funcionam, a fim de encontrar padrões e falhas, além de propor a compreensão do modo que arquivos png são estruturados, através de bytes e números hexadecimais.

- [Página do desafio](https://play.picoctf.org/practice/challenge/29)

#### Análise Inicial

A princípio, nos declaramos com o seguinte link que nos leva à página do desafio:

- [Link da aplicação](http://jupiter.challenges.picoctf.org:58112/)

Juntamente desse link, há a seguinte dica:

- Este é apenas um problema de JavaScript.

Ao abrirmos o link, nos deparamos com a seguinte tela:

[![Imagem17.png](https://i.postimg.cc/Y9dBzqnG/Imagem17.png)](https://postimg.cc/DJ4N2FKn)

Nessa tela, há a opção de digitar e enviar algo. Ao tentar digitar alguma letra como `ab`, ele não retorna nada. Porém, ao digitar algum número de exemplo como `1234567`, ele nos retorna o seguinte:

[![Imagem18.png](https://i.postimg.cc/Dw1VYcpK/Imagem18.png)](https://postimg.cc/3krbk2HL)

Ele gera um arquivo de uma imagem "quebrada". Ao abrir esse arquivo em outra aba do navegador, isso nos é mostrado:

[![Imagem19.png](https://i.postimg.cc/k5y07gd1/Imagem19.png)](https://postimg.cc/8FjKHDcM)

Não tem muito o que analisar, pois é apenas uma imagem quebrada. Qualquer entrada que seja algum número, e tente enviar irá retornar qualquer coisa.

Nesse caso, a única opção que nos resta é ir atrás do código fonte. Aqui está ele:

```
<html>
	<head>    
		<script src="jquery-3.3.1.min.js"></script>
		<script>
			var bytes = [];
			$.get("bytes", function(resp) {
				bytes = Array.from(resp.split(" "), x => Number(x));
			});

			function assemble_png(u_in){
				var LEN = 16;
				var key = "0000000000000000";
				var shifter;
				if(u_in.length == LEN){
					key = u_in;
				}
				var result = [];
				for(var i = 0; i < LEN; i++){
					shifter = key.charCodeAt(i) - 48;
					for(var j = 0; j < (bytes.length / LEN); j ++){
						result[(j * LEN) + i] = bytes[(((j + shifter) * LEN) % bytes.length) + i]
					}
				}
				while(result[result.length-1] == 0){
					result = result.slice(0,result.length-1);
				}
				document.getElementById("Area").src = "data:image/png;base64," + btoa(String.fromCharCode.apply(null, new Uint8Array(result)));
				return false;
			}
		</script>
	</head>
	<body>

		<center>
			<form action="#" onsubmit="assemble_png(document.getElementById('user_in').value)">
				<input type="text" id="user_in">
				<input type="submit" value="Submit">
			</form>
			<img id="Area" src=""/>
		</center>

	</body>
</html>
```

Chegamos a algumas conclusões:

Esse código em JavaScript, recebe um conjunto de bytes direto do servidor, processa-os de acordo com a entrada do usuário e gera uma imagem PNG.

Vamos dissecar o código:

O seguinte trecho:

```
$.get("bytes", function(resp) {
	bytes = Array.from(resp.split(" "), x => Number(x));
});
```

A função dele é realizar uma requisição `get` para o bytes, separando tudo com espaços, e armazená-los em um array.

Assim que o usuário insere uma chave (16 caracteres), chama a função assemble_png().

Essa função organiza os bytes utilizando um `shifter` (deslocamento). O trecho que utiliza o shifter é esse: `shifter = key.charCodeAt(i) - 48;`
 
Então, esses bytes são armazenados no array `result[]`, convertidos para Base64 e atribuídos no src da imagem.

O trecho que faz essa conversão é esse: `btoa(String.fromCharCode.apply(null, new Uint8Array(result)))`.

Portanto, há duas maneiras de resolver:
- Refazendo o código para que ele mostre a imagem corretamente.
- Descobrindo os 16 caracteres necessários para mostrar a imagem correta.

#### Solução

Antes de mais nada, é necessário entender como é feita uma PNG.

Todo [PNG](https://en.wikipedia.org/wiki/PNG) é formado por números hexadecimais divididos em 4 categorias: PNG signature, IMG Header, IMG Data e IMG End. Tais números são organizados nas seguintes posições:

[![Imagem20.png](https://i.postimg.cc/9f7QFTfm/Imagem20.png)](https://postimg.cc/7CqkK57d)

Isso nada mais é do que os códigos hex de um único pixel vermelho. No código está descrito que são necessários 16 dígitos para gerar uma imagem. São esses os 16 primeiros bytes da imagem: `89 50 4E 47 0D 0A 1A 0A 00 00 00 0D 49 48 44 52`.

Uma maneira de adquirir os códigos hex, é alterando o final do link de `/?` para `/bytes`. Fazendo isso, conseguimos os seguintes códigos: 

```
128 252 182 115 177 211 142 252 189 248 130 93 154 0 68 90 131 255 204 170 239 167 18 51 233 43 0 26 210 72 95 120 227 7 195 126 207 254 115 53 141 217 0 11 118 192 110 0 0 170 248 73 103 78 10 174 208 233 156 187 185 65 228 0 137 128 228 71 159 10 111 10 29 96 71 238 141 86 91 82 0 214 37 114 7 0 238 114 133 0 140 0 38 36 144 108 164 141 63 2 69 73 15 65 68 0 249 13 0 64 111 220 48 0 55 255 13 12 68 41 66 120 188 0 73 27 173 72 189 80 0 148 0 64 26 123 0 32 44 237 0 252 36 19 52 0 78 227 98 88 1 185 1 128 182 177 155 44 132 162 68 0 1 239 175 248 68 91 84 18 223 223 111 83 26 188 241 12 0 197 57 89 116 96 223 96 161 45 133 127 125 63 80 129 69 59 241 157 0 105 57 23 30 241 62 229 128 91 39 152 125 146 216 91 5 217 16 48 159 4 198 23 108 178 199 14 6 175 51 154 227 45 56 140 221 0 230 228 99 239 132 198 133 72 243 93 3 86 94 246 156 153 123 1 204 200 233 143 127 64 164 203 36 24 2 169 121 122 159 40 4 25 64 0 241 9 94 220 254 221 122 8 22 227 140 221 248 250 141 66 78 126 190 73 248 105 5 14 26 19 119 223 103 165 69 177 68 61 195 239 115 199 126 61 41 242 175 85 211 11 5 250 93 79 194 78 245 223 255 189 0 128 9 150 178 0 112 247 210 21 36 0 2 252 144 59 101 164 185 94 232 59 150 255 187 1 198 171 182 228 147 73 149 47 92 133 147 254 173 242 39 254 223 214 196 135 248 34 146 206 63 127 127 22 191 92 88 69 23 142 167 237 248 23 215 148 166 59 243 248 173 210 169 254 209 157 174 192 32 228 41 192 245 47 207 120 139 28 224 249 29 55 221 109 226 21 129 75 41 113 192 147 45 144 55 228 126 250 127 197 184 155 251 19 220 11 241 171 229 213 79 135 93 49 94 144 38 250 121 113 58 114 77 111 157 146 242 175 236 185 60 67 173 103 233 234 60 248 27 242 115 223 207 218 203 115 47 252 241 152 24 165 115 126 48 76 104 126 42 225 226 211 57 252 239 21 195 205 107 255 219 132 148 81 171 53 79 91 27 174 235 124 213 71 221 243 212 38 224 124 54 77 248 252 88 163 44 191 109 63 189 231 251 189 242 141 246 249 15 0 2 230 7 244 161 31 42 182 219 15 221 164 252 207 53 95 99 60 190 232 78 255 197 16 169 252 100 164 19 158 32 189 126 140 145 158 116 245 68 94 149 111 252 74 135 189 83 74 71 218 99 220 208 87 24 228 11 111 245 1 0 98 131 46 22 94 71 244 22 147 21 83 155 252 243 90 24 59 73 247 223 127 242 183 251 124 28 245 222 199 248 122 204 230 79 219 147 11 225 202 239 24 132 55 89 221 143 151 137 63 150 79 211 8 16 4 60 63 99 65 0 2
```

Após isso, basta converter tudo para hexadecimal.

Com os valores em mãos, iremos utilizar a ferramenta HxD.

Ao inserirmos os bytes na ferramenta, nos deparamos com a seguinte tela:

[![Imagem21.png](https://i.postimg.cc/zf5cVYzy/Imagem21.png)](https://postimg.cc/5X73RGFJ)

Após isso, podemos verificar diretamente nas colunas (de cima para baixo), a posição dos bytes que fora destacados anteriormente.

[![Imagem22.png](https://i.postimg.cc/0N2hV1LL/Imagem22.png)](https://postimg.cc/jDm8jmqX)

Para descobrir a chave, pegamos a posição de cada um dos 16 caracteres destacados anteriormente, e subtraímos 1, pois esse cálculo é chamado no seguinte trecho de código: `result = result.slice(0,result.length-1)`.

Assim, chegaremos na seguinte chave:

> 4894748485167104

Ao enviar a chave no campo de texto, chegamos no seguinte código: 

[![Imagem23.png](https://i.postimg.cc/g0SQ5Bpr/Imagem23.png)](https://postimg.cc/rd4fRjKL)

Ao escanear o qr code, obtemos a flag:

>`picoCTF{7b15cfb95f05286e37a22dda25935e1e}`
 
