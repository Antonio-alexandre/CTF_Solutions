# Duckware Team - Another Android Applaketion
###### Solved by @Antonio-alexandre

> This CTF is about reverse engineering, code analysis

## Desafio: 1 (Reverse Engineering, Code Analysis)
#### Introdução

Esse desafio consiste em analisar o código de um arquivo apk em Java a fim de descobrir uma flag escondida.

#### Análise Inicial

Logo de início, é possível notar que o arquivo se trata de um .apk (basicamente um zip). O primeiro passo então é extrair os arquivos que tem dentro.

Fazendo isso, obtém-se:

[![image.png](https://i.postimg.cc/y60RCxHf/image.png)](https://postimg.cc/c64CQs6Y)

#### Solução

Para obter o código fonte, é necessário abrir o arquivo classes.dex. Esse tipo de arquivo .dex, é característico de sistemas que possuem a linguagem Java/Kotlin. Para abrir o arquivo então, basta utilizar o jadx:

[![image.png](https://i.postimg.cc/gkJ6XDhv/image.png)](https://postimg.cc/Rqk02wXF)

Dando uma analisada rápida, é possível notar que o código possui funções parecidas com nomes parecidos. Cada método valida alguns caracteres da flag. Para descobrir a flag original, basta descobrir um input que seja validado corretamente pela lógica do código.

Além disso, há uma função escondida lá no final com nome de `nop`, que aparentemente faz o mesmo que o resto.

Analisando mais aprofundadamente, é possível notar a função:

`com.lake.ctf.MainActivity$$ExternalSyntheticLambda0`

Abrindo a função, há o seguinte trecho de código que denuncia o tamanho da flag:

```
    /* renamed from: lambda$onCreate$0$com-lake-ctf-MainActivity, reason: not valid java name */
    /* synthetic */ void m4lambda$onCreate$0$comlakectfMainActivity(EditText editText, TextView textView, View view) {
        String string = editText.getText().toString();
        Log.i("LAKECTF", "flag: " + string);
        if (string.length() != 55) {
            textView.setText("flag is wrong...");
            return;
        }
        boolean zTest = Test(string);
        boolean zTest2 = Test(string);
        boolean zTest3 = Test(string);
        boolean zTest4 = Test(string);
        boolean zTest5 = Test(string);
        boolean zTest6 = Test(string);
        boolean zTest7 = Test(string);
        boolean zTest8 = Test(string);
        boolean zTest9 = Test(string);
        boolean zTest10 = Test(string);
        boolean zTest11 = Test(string);
        boolean zTest12 = Test(string);
        //...
        boolean zTest78 = Test(string);
        boolean zTest79 = Test(string);
        boolean zTest80 = Test(string);
        if (zTest && zTest2 && zTest3 && zTest4 && zTest5 && zTest6 && zTest7 && zTest8 && zTest9 && zTest10 && zTest11 && zTest12 && zTest13 && zTest14 && zTest15 && zTest16 && zTest17 && zTest18 && zTest19 && zTest20 && zTest21 && zTest22 && zTest23 && zTest24 && zTest25 && zTest26 && zTest27 && zTest28 && zTest29 && zTest30 && zTest31 && zTest32 && zTest33 && zTest34 && zTest35 && zTest36 && zTest37 && zTest38 && zTest39 && zTest40 && zTest41 && zTest42 && zTest43 && zTest44 && zTest45 && zTest46 && zTest47 && zTest48 && zTest49 && zTest50 && zTest51 && zTest52 && zTest53 && zTest54 && zTest55 && zTest56 && zTest57 && zTest58 && zTest59 && zTest60 && zTest61 && zTest62 && zTest63 && zTest64 && zTest65 && zTest66 && zTest67 && zTest68 && zTest69 && zTest70 && zTest71 && zTest72 && zTest73 && zTest74 && zTest75 && zTest76 && zTest77 && zTest78 && zTest79 && zTest80) {
            textView.setText("flag correct!");
        } else {
            textView.setText("flag is wrong...");
        }
    }
}
```

Porém, para chegar a qualquer conclusão é necessário analisar outros aspectos. 

Dentro da pasta /lib, há a biblioteca: `libohgreat2.so` que é referenciada em alguns trechos do código fonte. Isso significa que talvez haja algo de interessante lá.

Abrindo o arquivo no ghidra:

Dentro da função MainActivity_Init, é possível notar o seguinte:

```
undefined8 Java_com_lake_ctf_MainActivity_Init(void)

{
  char *pcVar1;
  undefined8 *puVar2;
  
  puVar2 = (undefined8 *)calloc(0x50,1);
  aaa = puVar2;
  *puVar2 = 0x3934313035653638;
  puVar2[1] = 0x3133313636383536;
  puVar2[2] = 0x3533623065396132;
  puVar2[3] = 0x3666343864383535;
  puVar2[4] = 0x3937616433643663;
  puVar2[5] = 0x3639613235356637;
  pcVar1 = (char *)(puVar2 + 6);
  pcVar1[0] = '5';
  pcVar1[1] = '7';
  pcVar1[2] = 'f';
  pcVar1[3] = 'e';
  pcVar1[4] = '0';
  pcVar1[5] = '5';
  pcVar1[6] = '5';
  pcVar1[7] = '8';
  pcVar1 = (char *)(puVar2 + 7);
  pcVar1[0] = 'c';
  pcVar1[1] = 'a';
  pcVar1[2] = '4';
  pcVar1[3] = '0';
  pcVar1[4] = 'c';
  pcVar1[5] = 'd';
  pcVar1[6] = 'e';
  pcVar1[7] = 'f';
  *(undefined1 *)(puVar2 + 8) = 0;
  puVar2 = (undefined8 *)calloc(0x50,1);
  bbb = puVar2;
  *puVar2 = 0x3566613462313330;
  puVar2[1] = 0x6130336365373931;
  puVar2[2] = 0x6663383466363239;
  puVar2[3] = 0x6437613131653034;
  puVar2[4] = 0x3834303037346362;
  puVar2[5] = 0x3330303465313261;
  pcVar1 = (char *)(puVar2 + 6);
  pcVar1[0] = 'b';
  pcVar1[1] = '7';
  pcVar1[2] = 'a';
  pcVar1[3] = '3';
  pcVar1[4] = 'c';
  pcVar1[5] = '0';
  pcVar1[6] = '7';
  pcVar1[7] = 'c';
  pcVar1 = (char *)(puVar2 + 7);
  pcVar1[0] = '5';
  pcVar1[1] = 'd';
  pcVar1[2] = 'a';
  pcVar1[3] = 'b';
  pcVar1[4] = '1';
  pcVar1[5] = 'b';
  pcVar1[6] = 'a';
  pcVar1[7] = 'a';
  *(undefined1 *)(puVar2 + 8) = 0;
  return 0;
}
```

Nesse código, são atribuídos dois valores a duas variáveis, aaa e bbb, que coincidem com nomes encontrados nas classes do Java.

Abrindo a função `do_nop`, aparece o seguinte:

```
void __cdecl do_nop(long *param_1,undefined8 param_2,undefined8 param_3,undefined8 param_4)

{
  char *__src;
  char *__src_00;
  
  __src = (char *)(**(code **)(*param_1 + 0x548))(param_1,param_3,0);
  __src_00 = (char *)(**(code **)(*param_1 + 0x548))(param_1,param_4,0);
  strcpy(aaa,__src);
  strcpy(bbb,__src_00);
  return;
}
```

Essa função atribui os novos valores de aaa e bbb, baseados nos valores das strings passados pelo Java.

Para prosseguir então, é necessário acessar as função e o método que contém os nomes iguais aos valores de aaa e bbb. São elas:

```
aaa=86e50149658661312a9e0b35558d84f6c6d3da797f552a9657fe0558ca40cdef

public class Check86e50149658661312a9e0b35558d84f6c6d3da797f552a9657fe0558ca40cdef

bbb=031b4af5197ec30a926f48cf40e11a7dbc470048a21e4003b7a3c07c5dab1baa

Check031b4af5197ec30a926f48cf40e11a7dbc470048a21e4003b7a3c07c5dab1baa
```

```
static boolean Check031b4af5197ec30a926f48cf40e11a7dbc470048a21e4003b7a3c07c5dab1baa(String str) {
    if ((str.charAt(1) - str.charAt(43)) - str.charAt(50) == -78) {
        nop("4e07408562bedb8b60ce05c1decfe3ad16b72230967de01f640b7e4729b49fce", "3ada92f28b4ceda38562ebf047c6ff05400d4c572352a1142eedfef67d21e662");
        return true;
    }
    nop("eb1e33e8a81b697b75855af6bfcdbcbf7cbbde9f94962ceaec1ed8af21f5a50f", "9400f1b21cb527d7fa3d3eabba93557a18ebe7a2ca4e471cfe5e4c5b4ca7f767");
    return false;
}
```

É possível concluir então, que os caracteres da flag precisam satisfazer a condição presente acima.

Porém, como é bem complexo de ser realizado manualmente, dá para automatizar com técnicas de parse.

Por fim, para a conclusão final, foi utilizado o seguinte código (crédito para [pawlos](https://allthingsreversed.io/20251130-AnotherAndroidApplaketion.html)):

```
def parse_condition(condition_string, z3_variables):
    cleaned_string = condition_string.replace(' ', '')
    
    main_pattern = re.compile(r"(.+?)([=!<>]{1,2})([-]?\d+)$")
    main_match = main_pattern.match(cleaned_string)
    
    if not main_match:
        return {"error": "Non matched condition."}

    lhs_string = main_match.group(1)
    comparison_operator = main_match.group(2)
    result_value = int(main_match.group(3))

    first_char_match = re.search(r"str\.charAt\((\d+)\)", lhs_string)
    if not first_char_match:
        return {"error": "No str.charAt."}

    first_index = int(first_char_match.group(1))
    
    rest_of_lhs = lhs_string[first_char_match.end():]
    
    subsequent_pattern = re.compile(r"([+\-*/])str\.charAt\((\d+)\)")
    subsequent_matches = subsequent_pattern.findall(rest_of_lhs)
    
    indexes = [first_index]
    arithmetic_operators = []
    
    for op, idx in subsequent_matches:
        arithmetic_operators.append(op)
        indexes.append(int(idx))

    z3_constraint = None
    
    current_z3_expr = z3_variables[first_index]
    
    for i in range(len(arithmetic_operators)):
        op = arithmetic_operators[i]
        next_var = z3_variables[indexes[i+1]]
        
        if op == '+':
            current_z3_expr += next_var
        elif op == '-':
            current_z3_expr -= next_var
        elif op == '*':
            current_z3_expr *= next_var
        elif op == '/':
            current_z3_expr /= next_var
    
    if comparison_operator == '==':
        z3_constraint = (current_z3_expr == result_value)
    elif comparison_operator == '!=':
        z3_constraint = (current_z3_expr != result_value)
    elif comparison_operator == '<':
        z3_constraint = (current_z3_expr < result_value)
    elif comparison_operator == '>':
        z3_constraint = (current_z3_expr > result_value)
    elif comparison_operator == '<=':
        z3_constraint = (current_z3_expr <= result_value)
    elif comparison_operator == '>=':
        z3_constraint = (current_z3_expr >= result_value)
        
    return z3_constraint
```

Executando esse código, aparece o seguinte:

```
:)
b’EPFL{Wh1_3v3n_b0th3r_w1th_J4v4_1n_th3_f1rst_Pl4c3?????}’
```

