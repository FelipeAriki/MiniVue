# Capítulo 18 — O Compilador do Vue: Como um Template se Transforma em Código JavaScript

> **Objetivo:** compreender como o compilador do Vue transforma um `<template>` em uma Render Function altamente otimizada. Ao final deste capítulo você entenderá como funcionam o Parser, a AST (Abstract Syntax Tree), as fases de transformação, a geração de código (Codegen) e por que boa parte da performance do Vue 3 vem do compilador.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Explicar o papel do Compiler.
* Entender a diferença entre Runtime e Compiler.
* Compreender o Parsing.
* Construir uma AST simplificada.
* Entender o Transform Pipeline.
* Compreender o Code Generation.
* Explicar Static Hoisting.
* Entender como nascem Patch Flags e Block Trees.

---

# Pré-requisitos

* Capítulos 01 ao 17.

---

# Uma dúvida comum

Imagine o componente.

```vue
<template>

<h1>{{ nome }}</h1>

</template>
```

Como isso vira:

```javascript
render() {

    return h(
        "h1",
        null,
        nome.value
    )

}
```

Quem faz essa transformação?

---

# O Compiler

O Vue possui um pacote específico.

```text
@vue/compiler-core
```

Responsável por transformar Templates em JavaScript.

---

# Arquitetura

O Vue é dividido em três grandes pilares.

```text
Compiler

↓

Runtime Core

↓

Runtime DOM
```

Até agora estudamos praticamente todo o Runtime.

Agora começaremos o Compiler.

---

# O fluxo completo

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
```

Esse fluxo acontece sempre.

---

# Em tempo de desenvolvimento

Quando utilizamos.

```vue
<template>

<p>Hello</p>

</template>
```

O compilador executa.

↓

Gera uma Render Function.

↓

O Runtime apenas executa essa função.

---

# O que é Parsing?

Parsing é o processo de interpretar texto.

Entrada.

```html
<div>

Olá

</div>
```

Saída.

Uma estrutura organizada.

---

# O Parser

Imagine.

Entrada.

```text
<div>Hello</div>
```

O Parser lê caractere por caractere.

```text
<

↓

d

↓

i

↓

v

↓

>

...
```

---

# Primeira etapa

O Parser identifica.

```text
Tag de abertura
```

Depois.

```text
Texto
```

Depois.

```text
Tag de fechamento
```

---

# Resultado

Em vez de texto.

Temos uma árvore.

```text
div

↓

Hello
```

Essa árvore recebe um nome.

```text
AST
```

---

# O que é AST?

AST significa.

```text
Abstract Syntax Tree
```

Ou.

Árvore Sintática Abstrata.

---

# Exemplo

Template.

```vue
<div>

<p>Hello</p>

</div>
```

AST.

```text
Root

↓

div

↓

p

↓

Hello
```

---

# Estrutura

Simplificando.

```javascript
{

type:"Root",

children:[

]

}
```

---

# Nó de Elemento

```javascript
{

type:"Element",

tag:"div",

children:[

]

}
```

---

# Nó de Texto

```javascript
{

type:"Text",

content:"Hello"

}
```

---

# Interpolação

Template.

```vue
{{ nome }}
```

AST.

```javascript
{

type:"Interpolation",

content:"nome"

}
```

---

# Expressões

Quando fazemos.

```vue
{{ idade + 1 }}
```

O Vue não guarda apenas texto.

Guarda uma expressão JavaScript.

---

# Estrutura

```javascript
{

type:"Expression",

content:"idade + 1"

}
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

```text
Root

↓

Element(div)

↓

Interpolation

↓

Expression(nome)
```

---

# Por que criar uma AST?

Porque transformar árvores é muito mais fácil do que transformar texto.

O compilador nunca mais trabalha diretamente com HTML.

Ele trabalha apenas com objetos JavaScript.

---

# Transform Pipeline

Depois do Parser.

Entra uma nova etapa.

```text
Transforms
```

---

# O objetivo

Modificar a AST.

Adicionar informações.

Otimizar.

Preparar para gerar código.

---

# Exemplo

Template.

```vue
<div>

{{ nome }}

</div>
```

Antes.

```text
Element

↓

Interpolation
```

Depois dos Transforms.

```text
Element

↓

Interpolation

↓

PatchFlag(TEXT)
```

O compilador adiciona informações que não existiam originalmente.

---

# Múltiplos Transforms

O Vue possui dezenas.

Por exemplo.

* transformElement
* transformText
* transformExpression
* transformIf
* transformFor
* transformSlotOutlet

Cada um é responsável por uma parte específica da linguagem.

---

# transformExpression

Imagine.

```vue
{{ nome }}
```

O compilador transforma.

```javascript
nome
```

Em algo parecido com.

```javascript
_ctx.nome
```

Assim a Render Function sabe onde buscar o valor.

---

# transformElement

Recebe.

```vue
<div class="ativo">

</div>
```

E prepara a estrutura para gerar.

```javascript
createElementVNode(...)
```

---

# transformText

Imagine.

```vue
<div>

Olá {{ nome }}

</div>
```

O compilador une textos estáticos e dinâmicos.

Gerando uma estrutura muito mais eficiente.

---

# transformIf

Recebe.

```vue
<div

v-if="mostrar"

>
```

Transforma em.

```javascript
mostrar

?

VNode

:

null
```

---

# transformFor

Recebe.

```vue
<li

v-for="item in lista"

>
```

Transforma em um loop JavaScript.

---

# Depois dos Transforms

A AST está muito mais rica.

Ela contém:

* informações do elemento;
* otimizações;
* Patch Flags;
* Block Tree;
* Helpers necessários.

---

# Code Generation

Agora entra a última etapa.

```text
Codegen
```

---

# Objetivo

Transformar a AST em JavaScript.

---

# Exemplo

AST.

```text
Element

↓

Text
```

Saída.

```javascript
return h(

"div",

null,

"Hello"

)
```

---

# Outro exemplo

Template.

```vue
<p>

{{ nome }}

</p>
```

Codegen.

```javascript
return h(

"p",

null,

_ctx.nome

)
```

---

# Helpers

O compilador não gera tudo manualmente.

Ele utiliza Helpers.

Por exemplo.

```javascript
createElementVNode()
```

Ou.

```javascript
toDisplayString()
```

---

# Interpolação

Template.

```vue
{{ nome }}
```

Render.

```javascript
toDisplayString(

_ctx.nome

)
```

Assim qualquer valor é convertido corretamente para texto.

---

# Static Hoisting

Agora uma das maiores otimizações.

Imagine.

```vue
<div>

<h1>Vue</h1>

<p>{{ contador }}</p>

</div>
```

O título nunca muda.

Então.

O compilador faz.

```javascript
const _hoisted_1 =

createElementVNode(

"h1",

null,

"Vue"

)
```

Na Render.

```javascript
return h(

"div",

null,

[

_hoisted_1,

...

]

)
```

O elemento é criado apenas uma vez.

---

# Sem Hoisting

Toda renderização recriaria o VNode.

---

# Com Hoisting

O mesmo objeto é reutilizado.

---

# Patch Flags

Outro trabalho do compilador.

Imagine.

```vue
<div>

{{ nome }}

</div>
```

O compilador já sabe.

Somente o texto muda.

Então gera.

```javascript
patchFlag:

TEXT
```

O Runtime economiza comparações.

---

# Block Tree

O compilador também identifica.

Quais partes são dinâmicas.

Exemplo.

```vue
<div>

<header>

Logo

</header>

<p>

{{ contador }}

</p>

</div>
```

O compilador registra apenas.

```text
contador
```

Como nó dinâmico.

---

# Por que isso é importante?

Porque o Runtime percorre apenas os nós dinâmicos.

Em vez da árvore inteira.

---

# createVNode()

Em versões recentes.

Grande parte dos Templates gera.

```javascript
createVNode()
```

Ou.

```javascript
createElementVNode()
```

Esses Helpers criam os VNodes que estudamos no capítulo anterior.

---

# O compilador conhece o DOM?

Não.

Ele apenas gera chamadas para Helpers.

Quem cria elementos reais continua sendo o Renderer.

---

# Fluxo completo

```text
Template

↓

Compiler

↓

Render Function

↓

Runtime

↓

VNode

↓

Renderer

↓

DOM
```

---

# Compiler vs Runtime

Compiler.

↓

Transforma código.

---

Runtime.

↓

Executa código.

---

# Runtime Only Build

Existe uma versão do Vue.

```text
vue.runtime.esm.js
```

Ela não possui Compiler.

Só Runtime.

Muito utilizada com Vite.

---

# Full Build

Outra versão.

```text
vue.esm.js
```

Possui:

* Compiler;
* Runtime.

Permite compilar Templates em tempo de execução.

---

# Vite

Quando utilizamos.

```vue
App.vue
```

O Vite executa o Compiler durante o build.

No navegador chega apenas a Render Function.

---

# Performance

Essa arquitetura permite.

* eliminar parsing no navegador;
* reduzir trabalho em runtime;
* adicionar otimizações impossíveis de descobrir durante a execução.

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

Static Hoisting

↓

Patch Flags

↓

Block Tree

↓

Codegen

↓

Render Function
```

---

# Comparando

Template.

```vue
<h1>

{{ nome }}

</h1>
```

Resultado simplificado.

```javascript
function render(){

    return createElementVNode(

        "h1",

        null,

        toDisplayString(

            _ctx.nome

        ),

        TEXT

    )

}
```

Essa função será executada pelo Runtime toda vez que o componente precisar ser atualizado.

---

# Curiosidade

Uma das maiores diferenças entre Vue e outras bibliotecas é justamente o compilador.

O Runtime do Vue faz menos trabalho porque o Compiler já antecipou muitas decisões durante o build.

É por isso que, tecnicamente, o Vue é considerado uma biblioteca **Compiler-Informed**: o compilador produz informações que permitem ao Runtime executar atualizações muito mais eficientes.

---

# Resumo

Neste capítulo aprendemos que:

* O Compiler transforma Templates em Render Functions.
* O Parser converte HTML em uma AST.
* Os Transforms enriquecem e otimizam essa AST.
* O Code Generator produz código JavaScript.
* O Compiler gera Static Hoisting, Patch Flags e Block Trees.
* O Runtime apenas executa a Render Function produzida pelo Compiler.

---

# Exercícios

## Exercício 1

Desenhe a AST do seguinte Template:

```vue
<div>

<p>{{ mensagem }}</p>

</div>
```

---

## Exercício 2

Implemente uma AST simplificada para elementos e textos.

---

## Exercício 3

Implemente um pequeno Parser que reconheça:

* abertura de tag;
* fechamento de tag;
* texto.

---

## Exercício 4

Implemente um Code Generator que transforme uma AST simples em chamadas para `h()`.

---

# Desafio

Atualize sua **MiniVue** para incluir um mini compilador capaz de:

* fazer Parsing de Templates simples;
* gerar uma AST;
* aplicar um Transform básico;
* produzir uma Render Function utilizando `h()`.

---

# Projeto do capítulo

Ao final deste capítulo sua biblioteca deverá conseguir:

* interpretar Templates simples;
* construir uma AST;
* transformar essa AST;
* gerar código JavaScript equivalente a uma Render Function;
* reutilizar o Runtime criado nos capítulos anteriores.

---

# Checklist

* [ ] Sei explicar a diferença entre Compiler e Runtime.
* [ ] Entendi como funciona o Parser.
* [ ] Sei explicar o que é uma AST.
* [ ] Entendi o papel dos Transforms.
* [ ] Sei explicar como funciona o Code Generation.
* [ ] Entendi por que Static Hoisting melhora a performance.
* [ ] Minha MiniVue já possui um compilador simplificado.

---

# Próximo capítulo

## **Capítulo 19 — Parsing na Prática: Construindo um Parser de Templates do Zero**

No próximo capítulo deixaremos a teoria e construiremos um **Parser funcional**, muito semelhante ao utilizado pelo Vue. Você implementará um cursor de leitura, consumirá caracteres, reconhecerá tags, atributos, textos, comentários e interpolações (`{{ }}`), construindo passo a passo uma AST real. A partir deste ponto, sua MiniVue começará a ter um compilador próprio.
