# Capítulo 19 — Parsing na Prática: Construindo um Parser de Templates do Zero

> **Objetivo:** construir um Parser funcional semelhante ao utilizado pelo Vue 3. Ao final deste capítulo você será capaz de transformar um template em uma AST, implementando o mesmo fluxo usado pelo `@vue/compiler-core`.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Implementar um parser de templates.
* Criar um cursor de leitura.
* Consumir caracteres.
* Interpretar elementos HTML.
* Interpretar atributos.
* Interpretar textos.
* Interpretar interpolações (`{{ }}`).
* Interpretar comentários.
* Produzir uma AST navegável.

---

# Pré-requisitos

* Capítulos 01 ao 18.

---

# Onde paramos?

No capítulo anterior vimos que o Compiler funciona assim.

```text
Template

↓

Parser

↓

AST

↓

Transforms

↓

Codegen
```

Agora construiremos o Parser.

---

# Nosso objetivo

Dado este template.

```html
<div>

    <h1>Hello</h1>

    <p>{{ nome }}</p>

</div>
```

Queremos gerar algo parecido com:

```javascript
{
    type: "Root",
    children: [
        {
            type: "Element",
            tag: "div",
            children: [
                {
                    type: "Element",
                    tag: "h1",
                    children: [
                        {
                            type: "Text",
                            content: "Hello"
                        }
                    ]
                },
                {
                    type: "Element",
                    tag: "p",
                    children: [
                        {
                            type: "Interpolation",
                            content: "nome"
                        }
                    ]
                }
            ]
        }
    ]
}
```

---

# O Parser trabalha com um Cursor

O Vue nunca divide o template inteiro em pedaços.

Ele utiliza um cursor.

Imagine.

```text
<div>Hello</div>
 ^
```

O cursor sempre aponta para a posição atual.

---

# Estrutura inicial

```javascript
function createParserContext(source){

    return {

        source

    }

}
```

Nosso contexto possui apenas uma informação.

O texto restante.

---

# Exemplo

Inicialmente.

```text
<div>Hello</div>
```

Depois de consumir `<div>`.

```text
Hello</div>
```

Depois de consumir `Hello`.

```text
</div>
```

Observe.

Sempre reduzimos a string.

---

# advanceBy()

A função mais importante do Parser.

```javascript
function advanceBy(

    context,

    n

){

    context.source =

        context.source.slice(n)

}
```

Ela move o cursor.

---

# Exemplo

Entrada.

```text
<div>Hello
```

Executando.

```javascript
advanceBy(context,5)
```

Resultado.

```text
Hello
```

---

# Primeira decisão

O que existe na posição atual?

Se começa com.

```text
<
```

↓

Elemento.

---

Se começa com.

```text
{{
```

↓

Interpolação.

---

Caso contrário.

↓

Texto.

---

# Fluxo

```text
Template

↓

"<"

↓

Elemento
```

Ou.

```text
"{{"

↓

Interpolação
```

Ou.

```text
Texto
```

---

# parseChildren()

Essa função controla praticamente todo o Parser.

```javascript
function parseChildren(

    context

){

    const nodes = []

    while(

        !isEnd(context)

    ){

    }

    return nodes

}
```

Ela continua lendo até chegar ao final.

---

# Dentro do loop

```javascript
while(

    !isEnd(context)

){

    let node

}
```

Agora decidimos.

Qual nó criar.

---

# Identificando um Elemento

```javascript
if(

    context.source.startsWith("<")

){

    node = parseElement(context)

}
```

---

# Identificando Interpolação

```javascript
else if(

    context.source.startsWith("{{")

){

    node = parseInterpolation(context)

}
```

---

# Caso contrário

É texto.

```javascript
else{

    node = parseText(context)

}
```

Depois.

```javascript
nodes.push(node)
```

---

# parseText()

Imagine.

```text
Hello World</div>
```

Precisamos consumir.

Apenas.

```text
Hello World
```

---

# Como descobrir onde termina?

Texto termina quando encontramos.

```text
<
```

Ou.

```text
{{
```

---

# Exemplo

```text
Hello {{nome}}
```

Texto.

↓

```text
Hello
```

---

# Implementação simplificada

```javascript
const end =

context.source.indexOf("<")
```

Depois.

```javascript
const content =

context.source.slice(

0,

end

)
```

Consumimos.

```javascript
advanceBy(

context,

content.length

)
```

---

# Resultado

```javascript
{

type:"Text",

content

}
```

---

# parseInterpolation()

Entrada.

```text
{{ nome }}
```

Primeiro removemos.

```text
{{
```

---

```javascript
advanceBy(

context,

2

)
```

Agora temos.

```text
nome }}
```

---

# Procurando o fechamento

```javascript
const closeIndex =

context.source.indexOf("}}")
```

---

# Conteúdo

```javascript
const content =

context.source

.slice(

0,

closeIndex

)

.trim()
```

Resultado.

```text
nome
```

---

# Consumindo

Depois removemos.

```text
nome
```

Depois.

```text
}}
```

---

# Resultado

```javascript
{

type:"Interpolation",

content

}
```

---

# parseElement()

Agora vem a parte interessante.

Entrada.

```text
<div>Hello</div>
```

---

# Primeiro

Consumimos.

```text
<div>
```

Precisamos descobrir.

```text
div
```

---

# Regex simplificada

```javascript
/^<([a-z]+)/i
```

Ela encontra.

```text
div
```

---

# Depois

Criamos.

```javascript
{

type:"Element",

tag:"div",

children:[]
}
```

---

# Agora

Precisamos interpretar.

Tudo que existe.

Até.

```text
</div>
```

---

# Recursão

Chamamos novamente.

```javascript
element.children =

parseChildren(

context

)
```

Perceba.

Um Elemento pode conter.

Outros Elementos.

---

# Exemplo

```html
<div>

<p>

Hello

</p>

</div>
```

Temos.

```text
div

↓

p

↓

Hello
```

Tudo graças à recursão.

---

# Quando parar?

Precisamos descobrir.

Se encontramos.

```text
</div>
```

---

# isEnd()

Implementação simplificada.

```javascript
function isEnd(

context

){

    return

    context.source.startsWith("</")
}
```

Mais tarde melhoraremos.

---

# Consumindo o fechamento

Encontramos.

```text
</div>
```

Removemos.

```javascript
advanceBy(

context,

6

)
```

Agora voltamos para o pai.

---

# Recursão completa

```text
parseElement(div)

↓

parseChildren()

↓

parseElement(p)

↓

parseChildren()

↓

parseText()

↓

Retorna

↓

Retorna

↓

Retorna
```

É exatamente assim que o Vue funciona.

---

# Comentários

Template.

```html
<!-- comentário -->
```

O Parser identifica.

```text
<!--
```

Depois procura.

```text
-->
```

Resultado.

```javascript
{

type:"Comment",

content:"comentário"

}
```

---

# CDATA

Em SVG e XML.

Existe.

```html
<![CDATA[

...

]]>
```

O Vue também interpreta.

---

# Doctype

```html
<!DOCTYPE html>
```

Também possui um parser específico.

---

# Espaços

Imagine.

```html
<div>





Hello

</div>
```

O Vue precisa decidir.

Esses espaços são importantes?

Nem sempre.

Existe uma etapa específica para normalização.

---

# Atributos

Agora.

```html
<div

class="card"

id="app"

>
```

Precisamos interpretar.

* class
* id

---

# parseAttributes()

Fluxo.

```text
class

↓

=

↓

"card"
```

Resultado.

```javascript
{

name:"class",

value:"card"

}
```

---

# Diretivas

O Parser também reconhece.

```vue
v-if
```

```vue
v-for
```

```vue
v-model
```

```vue
@click
```

Mas ainda não sabe o que significam.

Ele apenas registra.

---

# Exemplo

```vue
<button

@click="salvar"

>
```

AST.

```javascript
{

type:"Directive",

name:"on",

arg:"click",

exp:"salvar"

}
```

A transformação acontecerá nos próximos capítulos.

---

# O Root

No início.

Criamos.

```javascript
{

type:"Root",

children:[]
}
```

Depois.

```javascript
root.children =

parseChildren(

context

)
```

Agora temos uma AST completa.

---

# Fluxo completo

```text
Template

↓

Context

↓

advanceBy()

↓

parseChildren()

↓

parseElement()

↓

parseText()

↓

parseInterpolation()

↓

AST
```

---

# Comparando com o Vue

Nossa versão.

```text
Template

↓

Parser

↓

AST
```

Vue.

```text
baseParse()

↓

ParserContext

↓

parseChildren()

↓

parseElement()

↓

parseAttributes()

↓

parseInterpolation()

↓

parseComment()

↓

AST
```

Muito parecido.

---

# Arquivos do Vue

Grande parte desse código está em:

```text
packages/compiler-core/src/parser.ts
```

Ao abrir esse arquivo no GitHub, você reconhecerá praticamente todos os conceitos implementados neste capítulo.

---

# Curiosidade

O Parser do Vue não utiliza um parser HTML pronto.

Ele possui sua própria implementação.

Isso permite compreender perfeitamente as diretivas do Vue (`v-if`, `v-for`, `v-bind`, `v-model`, `v-slot`, etc.) e gerar uma AST já preparada para as próximas fases do compilador.

---

# Resumo

Neste capítulo aprendemos que:

* O Parser utiliza um cursor de leitura.
* `advanceBy()` move o cursor.
* `parseChildren()` controla a leitura da árvore.
* `parseElement()` interpreta elementos.
* `parseText()` interpreta textos.
* `parseInterpolation()` interpreta `{{ }}`.
* O Parser gera uma AST que será utilizada pelos Transforms.

---

# Exercícios

## Exercício 1

Implemente `advanceBy()`.

---

## Exercício 2

Implemente `parseText()`.

---

## Exercício 3

Implemente `parseInterpolation()`.

---

## Exercício 4

Implemente `parseElement()` utilizando recursão.

---

## Exercício 5

Implemente suporte básico a atributos (`class`, `id` e atributos personalizados).

---

# Desafio

Atualize sua **MiniVue Compiler** para suportar:

* `ParserContext`;
* `advanceBy()`;
* `parseChildren()`;
* `parseElement()`;
* `parseText()`;
* `parseInterpolation()`;
* comentários HTML;
* atributos simples.

---

# Projeto do capítulo

Ao final deste capítulo sua biblioteca deverá conseguir:

* ler um template caractere por caractere;
* interpretar elementos aninhados;
* construir uma AST recursiva;
* reconhecer textos, interpolações e comentários;
* produzir uma estrutura semelhante à utilizada pelo `@vue/compiler-core`.

---

# Checklist

* [ ] Sei explicar como funciona um Parser.
* [ ] Entendi o papel do `ParserContext`.
* [ ] Sei implementar `advanceBy()`.
* [ ] Sei interpretar elementos, textos e interpolações.
* [ ] Entendi como a recursão constrói a árvore da AST.
* [ ] Minha MiniVue já possui um Parser funcional.

---

# Próximo capítulo

## **Capítulo 20 — AST em Profundidade: Transform Pipeline, Visitors e Plugins do Compiler**

No próximo capítulo entraremos na segunda fase do compilador: os **Transforms**. Você aprenderá como o Vue percorre a AST utilizando um sistema de *visitors*, como diretivas (`v-if`, `v-for`, `v-model`, `v-bind`, `v-on`) são transformadas em estruturas de Runtime, como criar seus próprios *compiler plugins* e como o compilador prepara a AST para a geração de código. Esse é o ponto em que a AST deixa de representar apenas o template e passa a representar o código JavaScript que será executado pelo Runtime.
