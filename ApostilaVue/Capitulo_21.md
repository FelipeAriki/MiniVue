# Capítulo 21 — Code Generation: Gerando Render Functions do Zero

> **Objetivo:** compreender e implementar a última etapa do Compiler do Vue. Ao final deste capítulo você será capaz de transformar uma AST enriquecida em uma Render Function válida, entendendo exatamente como o Vue gera o código JavaScript que será executado pelo Runtime.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Explicar o funcionamento do Code Generator.
* Implementar um gerador de código.
* Criar um contexto de escrita.
* Gerar Helpers automaticamente.
* Gerar Render Functions.
* Produzir código JavaScript válido.
* Entender como o Vue gera `createElementVNode()`, `createVNode()` e outros Helpers.

---

# Pré-requisitos

* Capítulos 01 ao 20.

---

# Onde estamos?

Nosso Compiler já executa.

```text
Template

↓

Parser

↓

AST

↓

Transforms
```

Agora precisamos produzir JavaScript.

---

# O objetivo

Entrada.

```vue
<div>

{{ nome }}

</div>
```

Saída.

```javascript
function render(_ctx){

    return createElementVNode(

        "div",

        null,

        toDisplayString(

            _ctx.nome

        )

    )

}
```

---

# O Code Generator

O Vue possui uma função chamada.

```javascript
generate()
```

Ela recebe.

```javascript
generate(

    ast

)
```

E devolve.

```javascript
{

code,

map

}
```

---

# Estrutura

Simplificando.

```javascript
function generate(

    ast

){

}
```

---

# Primeira etapa

Criamos um contexto.

```javascript
const context = {

    code:""

}
```

---

# Por que um contexto?

Porque vamos escrevendo código aos poucos.

---

# Exemplo

Inicialmente.

```text
""
```

Depois.

```text
function render(
```

Depois.

```text
function render(_ctx){
```

Depois.

```text
return ...
```

---

# push()

O contexto possui uma função.

```javascript
context.push(

texto

)
```

Implementação.

```javascript
function push(

code

){

    context.code += code

}
```

---

# Exemplo

```javascript
push(

"function render(){"

)
```

Depois.

```javascript
push(

"}"

)
```

Resultado.

```javascript
function render(){

}
```

---

# Helpers

Lembra dos capítulos anteriores?

Durante os Transforms registramos.

```javascript
context.helpers
```

Exemplo.

```text
createElementVNode

toDisplayString
```

---

# Agora

Precisamos gerar.

```javascript
import{

createElementVNode,

toDisplayString

}from"vue"
```

Ou.

Na versão Runtime.

Referências internas.

---

# Fluxo

```text
Helpers

↓

Imports

↓

Render Function
```

---

# Gerando a função

Primeiro escrevemos.

```javascript
function render(
```

Depois.

```javascript
_ctx
```

Depois.

```javascript
){
```

---

# Resultado

```javascript
function render(

_ctx

){

}
```

---

# Agora

Geramos o corpo.

```javascript
return
```

---

# Quem gera o restante?

Uma função recursiva.

```javascript
genNode()
```

---

# Estrutura

```javascript
function genNode(

node,

context

){

}
```

Ela recebe qualquer nó da AST.

---

# Primeira decisão

```javascript
switch(

node.type

){

}
```

Dependendo do tipo.

Chamamos outro Generator.

---

# Exemplo

```javascript
switch(

node.type

){

case "Element":

...

case "Text":

...

case "Interpolation":

...

}
```

---

# genText()

Entrada.

```javascript
{

content:"Hello"

}
```

Saída.

```javascript
"Hello"
```

---

# Implementação

```javascript
push(

JSON.stringify(

node.content

)

)
```

---

# genInterpolation()

Entrada.

```vue
{{ nome }}
```

Saída.

```javascript
toDisplayString(

_ctx.nome

)
```

---

# Implementação

```javascript
push(

"toDisplayString("

)

genNode(

node.content

)

push(")")
```

---

# genExpression()

Entrada.

```javascript
_ctx.nome
```

Saída.

```javascript
_ctx.nome
```

Apenas escreve.

---

# genElement()

Agora.

Entrada.

```vue
<div>

Hello

</div>
```

Saída.

```javascript
createElementVNode(

"div",

null,

"Hello"

)
```

---

# Passos

Escrevemos.

```javascript
createElementVNode(
```

Depois.

Tag.

```javascript
"div"
```

Depois.

Props.

```javascript
null
```

Depois.

Children.

---

# Resultado

```javascript
createElementVNode(

"div",

null,

"Hello"

)
```

---

# Children

Podem ser.

* texto;
* interpolação;
* array;
* componente.

Precisamos gerar todos.

---

# Exemplo

```vue
<div>

<h1/>

<p/>

</div>
```

Resultado.

```javascript
createElementVNode(

"div",

null,

[

...

]

)
```

---

# Arrays

Criamos.

```javascript
[
```

Depois.

Cada filho.

---

# Implementação

```javascript
for(

const child

of node.children

){

    genNode(

child

)

}
```

---

# Componentes

Entrada.

```vue
<MeuBotao/>
```

Saída.

```javascript
createVNode(

MeuBotao

)
```

Observe.

Não usamos.

```javascript
createElementVNode()
```

---

# Comentários

Entrada.

```html
<!-- teste -->
```

Saída.

```javascript
createCommentVNode(

"teste"

)
```

---

# Fragment

Imagine.

```vue
<h1/>

<p/>
```

Dois elementos.

Sem pai.

O Compiler gera.

```javascript
Fragment
```

---

# Resultado

```javascript
createVNode(

Fragment,

null,

[...]

)
```

---

# Slots

Entrada.

```vue
<slot/>
```

Saída.

Chamadas específicas.

```javascript
renderSlot()
```

---

# Condições

Entrada.

```vue
<div

v-if="ok"

>
```

Depois dos Transforms.

Temos.

```javascript
ok

?

VNode

:

null
```

O Generator apenas escreve.

---

# Loops

Entrada.

```vue
<li

v-for="item in lista"

>
```

Depois do Transform.

Temos.

```javascript
renderList(...)
```

O Generator apenas gera o código.

---

# Observe

O Code Generator não interpreta diretivas.

Quem faz isso.

São os Transforms.

O Generator apenas escreve JavaScript.

---

# Indentação

O Vue produz código bonito.

Para isso.

O contexto possui.

```javascript
indent()
```

E.

```javascript
deindent()
```

---

# Exemplo

Resultado.

```javascript
function render(){

    return ...

}
```

Em vez de.

```javascript
function render(){return...}
```

---

# newline()

Outra função.

```javascript
newline()
```

Ela adiciona.

```text
\n
```

Mais espaços.

---

# Contexto completo

Simplificando.

```javascript
const context={

code:"",

indentLevel:0,

push(){},

newline(){},

indent(){},

deindent(){}

}
```

---

# Fluxo completo

```text
AST

↓

genNode()

↓

genElement()

↓

genText()

↓

genInterpolation()

↓

Código
```

---

# Exemplo completo

Template.

```vue
<div>

{{ nome }}

</div>
```

AST.

↓

Transforms.

↓

Code Generator.

↓

Resultado.

```javascript
function render(

_ctx

){

return createElementVNode(

"div",

null,

toDisplayString(

_ctx.nome

)

)

}
```

---

# Depois

Essa função será executada pelo Runtime.

Lembra do Capítulo 15?

Agora tudo se conecta.

```text
Render Function

↓

Reactive Effect

↓

VNode

↓

Patch

↓

DOM
```

---

# Arquitetura completa

```text
Template

↓

Parser

↓

AST

↓

Transforms

↓

Code Generator

↓

Render Function

↓

Runtime

↓

VNode

↓

DOM
```

---

# Source Maps

O Vue também gera.

```javascript
{

code,

map

}
```

O Source Map permite que ferramentas como DevTools e Vite relacionem o código gerado com o arquivo `.vue` original, facilitando o debug.

---

# Compilação em Desenvolvimento

Durante o desenvolvimento.

O Code Generator adiciona.

* nomes mais legíveis;
* informações de localização;
* mensagens de erro.

---

# Compilação em Produção

Em produção.

O código é otimizado.

* menos verificações;
* menos mensagens;
* menor tamanho.

---

# Arquivos reais

Grande parte dessa implementação está em.

```text
packages/compiler-core/src/codegen.ts
```

Ao abrir esse arquivo no código-fonte do Vue, você reconhecerá:

* `generate()`;
* `genNode()`;
* `genFunctionPreamble()`;
* `genNodeList()`;
* `genExpression()`;
* `genInterpolation()`;
* `genElement()`.

---

# Comparando

Nossa MiniVue.

```text
AST

↓

Generator

↓

Código
```

Vue.

```text
AST

↓

Generator Context

↓

Helpers

↓

Imports

↓

Render Function

↓

Source Maps

↓

Código Final
```

---

# Curiosidade

Uma Render Function gerada pelo Vue pode parecer complexa à primeira vista, mas ela é apenas JavaScript comum. Não existe nenhuma "mágica" durante a execução: o Runtime apenas chama essa função, obtém uma árvore de VNodes e a envia para o algoritmo de Patch que estudamos anteriormente.

---

# Resumo

Neste capítulo aprendemos que:

* O Code Generator transforma a AST em JavaScript.
* O Generator utiliza um contexto de escrita.
* `genNode()` delega a geração para funções específicas.
* Os Helpers registrados nos Transforms são utilizados durante a geração.
* O resultado final é uma Render Function executada pelo Runtime.
* O Vue também gera Source Maps e diferentes versões para desenvolvimento e produção.

---

# Exercícios

## Exercício 1

Implemente um `GeneratorContext` com:

* `push()`;
* `newline()`;
* `indent()`;
* `deindent()`.

---

## Exercício 2

Implemente `genText()`.

---

## Exercício 3

Implemente `genInterpolation()`.

---

## Exercício 4

Implemente `genElement()` para elementos simples.

---

## Exercício 5

Implemente um `generate()` que produza uma Render Function para uma AST contendo:

* elementos;
* textos;
* interpolações.

---

# Desafio

Atualize sua **MiniVue Compiler** para suportar:

* `GeneratorContext`;
* `generate()`;
* `genNode()`;
* `genElement()`;
* `genText()`;
* `genInterpolation()`;
* geração de uma Render Function executável.

---

# Projeto do capítulo

Ao final deste capítulo sua biblioteca deverá conseguir:

* converter uma AST em código JavaScript;
* gerar uma Render Function funcional;
* reutilizar Helpers registrados durante os Transforms;
* produzir código legível;
* conectar completamente o Compiler ao Runtime.

---

# Checklist

* [ ] Sei explicar como funciona o Code Generator.
* [ ] Entendi o papel do `GeneratorContext`.
* [ ] Sei implementar `genNode()`.
* [ ] Sei gerar código para elementos, textos e interpolações.
* [ ] Entendi como os Helpers são utilizados.
* [ ] Minha MiniVue já gera uma Render Function executável.

---

# Próximo capítulo

## **Capítulo 22 — Compilando um SFC (.vue): Como o Vue Processa `<template>`, `<script setup>` e `<style>`**

A partir do próximo capítulo deixaremos o `compiler-core` e entraremos no universo dos **Single File Components (SFC)**. Você aprenderá como o Vue interpreta um arquivo `.vue`, como funciona o `@vue/compiler-sfc`, o pipeline de compilação do `<template>`, `<script>`, `<script setup>` e `<style>`, como macros como `defineProps`, `defineEmits`, `defineExpose`, `defineOptions`, `defineSlots` e `defineModel` são transformadas em JavaScript, e como ferramentas como Vite conseguem importar um arquivo `.vue` como se fosse um módulo JavaScript comum. Esse é o próximo grande passo para entender toda a arquitetura do ecossistema Vue 3.
