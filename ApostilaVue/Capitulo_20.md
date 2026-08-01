# Capítulo 20 — AST em Profundidade: Transform Pipeline, Visitors e Plugins do Compiler

> **Objetivo:** compreender como o Vue transforma uma AST de Template em uma AST preparada para geração de código. Ao final deste capítulo você entenderá como funciona o **Transform Pipeline**, o padrão **Visitor**, os **Node Transforms**, os **Directive Transforms**, a criação de Helpers e como desenvolver seus próprios plugins para o compilador.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Explicar a segunda etapa do Compiler.
* Entender o Transform Pipeline.
* Compreender o padrão Visitor.
* Criar Node Transforms.
* Criar Directive Transforms.
* Entender como Helpers são registrados.
* Criar plugins para o Compiler.

---

# Pré-requisitos

* Capítulos 01 ao 19.

---

# Recapitulando

No capítulo anterior produzimos uma AST.

Template.

```vue
<div>

    <p>{{ nome }}</p>

</div>
```

AST.

```text
Root

↓

Element(div)

↓

Element(p)

↓

Interpolation(nome)
```

Ainda não existe JavaScript.

Existe apenas uma árvore.

---

# O próximo passo

Precisamos transformar essa árvore em algo que possa gerar código.

Para isso existe o:

```text
Transform Pipeline
```

---

# Arquitetura

O Compiler agora funciona assim.

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
```

Hoje estudaremos a etapa do meio.

---

# O que é um Transform?

É uma função que recebe um nó da AST.

Pode:

* modificar;
* substituir;
* remover;
* adicionar informações.

---

# Exemplo

Entrada.

```text
Interpolation

↓

nome
```

Saída.

```text
Interpolation

↓

_ctx.nome
```

O nó continua existindo.

Mas agora está preparado para o Runtime.

---

# Como o Vue percorre a árvore?

Ele utiliza um padrão muito conhecido.

```text
Visitor Pattern
```

---

# O Visitor Pattern

Imagine uma árvore.

```text
Root

↓

div

↓

p

↓

Interpolation
```

O Visitor percorre todos os nós.

---

# Fluxo

```text
Root

↓

Element(div)

↓

Element(p)

↓

Interpolation
```

Cada nó é visitado exatamente na ordem correta.

---

# Estrutura simplificada

```javascript
function traverseNode(node){

}
```

Essa função recebe qualquer nó.

---

# Primeira decisão

```javascript
switch(node.type){

}
```

Dependendo do tipo.

Chamamos um Transform diferente.

---

# Exemplo

```javascript
switch(node.type){

case "Element":

...

case "Interpolation":

...

}
```

---

# Percorrendo filhos

Se um nó possui filhos.

Precisamos visitá-los.

```javascript
for(

const child

of node.children

){

    traverseNode(child)

}
```

Isso cria uma recursão.

---

# Visualizando

```text
Root

↓

div

↓

p

↓

Interpolation
```

A função chama a si mesma.

Até chegar ao último nó.

---

# Transform Context

O Vue cria um contexto.

Simplificando.

```javascript
const context={

    helpers:new Set(),

    nodeTransforms:[],

    directiveTransforms:{}

}
```

Esse contexto é compartilhado entre todos os Transforms.

---

# Helpers

Imagine.

```vue
{{ nome }}
```

Para gerar código.

Precisaremos utilizar.

```javascript
toDisplayString()
```

O Transform registra.

```javascript
context.helpers.add(

"toDisplayString"

)
```

Mais tarde.

O Code Generator fará os imports automaticamente.

---

# Node Transform

Um Node Transform recebe.

```javascript
function transformElement(

node,

context

){

}
```

Ele pode modificar o nó.

---

# Exemplo

Entrada.

```text
Element

↓

div
```

Saída.

```text
Element

↓

codegenNode
```

Agora o Elemento já sabe como gerar código.

---

# Outro exemplo

Interpolation.

```vue
{{ nome }}
```

Transforma em.

```javascript
{

type:"Interpolation",

content:"_ctx.nome"

}
```

---

# Pós-processamento

Existe um detalhe importante.

Um Transform pode retornar outra função.

```javascript
return ()=>{

}
```

Essa função será executada depois dos filhos.

---

# Por quê?

Imagine.

```vue
<div>

<p>

Hello

</p>

</div>
```

Primeiro processamos.

```text
p
```

Depois.

```text
div
```

Isso facilita diversas otimizações.

---

# Ordem

```text
Entrando

↓

Filhos

↓

Saindo
```

É chamada de.

```text
Exit Phase
```

---

# Visualizando

```text
Root

↓

↓

↓

Interpolation

↑

↑

↑
```

Descemos.

Depois subimos.

Durante a subida acontecem várias transformações.

---

# Directive Transform

Agora outro tipo.

Imagine.

```vue
<input

v-model="nome"

/>
```

O Parser criou.

```text
Directive

↓

model
```

O Transform converte.

```text
model

↓

Runtime Helper
```

---

# Outro exemplo

```vue
<button

@click="salvar"

/>
```

Parser.

↓

Directive.

Transform.

↓

Evento.

↓

onClick.

---

# Resultado

```javascript
{

props:{

onClick:

_ctx.salvar

}

}
```

---

# transformExpression()

Entrada.

```vue
{{ nome }}
```

Saída.

```javascript
_ctx.nome
```

---

# transformElement()

Entrada.

```vue
<div>

</div>
```

Saída.

```javascript
createElementVNode(...)
```

---

# transformText()

Imagine.

```vue
Olá {{ nome }}
```

Parser.

↓

```text
Text

Interpolation
```

Transform.

↓

```text
CompoundExpression
```

---

# Resultado

Algo parecido.

```javascript
"Olá "

+

_ctx.nome
```

Muito mais eficiente.

---

# transformIf()

Entrada.

```vue
<div

v-if="mostrar"

>
```

Saída.

```javascript
mostrar

?

VNode

:

null
```

---

# transformFor()

Entrada.

```vue
<li

v-for="item in lista"

>
```

Saída.

```javascript
renderList(

lista,

...
)
```

---

# transformSlot()

Entrada.

```vue
<slot/>
```

Saída.

Estruturas específicas para Slots.

---

# transformComponent()

Entrada.

```vue
<MeuBotao/>
```

Saída.

```javascript
resolveComponent(

"MeuBotao"

)
```

---

# Helpers registrados

Durante os Transforms.

O Context vai acumulando.

```javascript
{

helpers:Set(

...

)

}
```

Exemplo.

```text
createVNode

toDisplayString

resolveComponent

openBlock
```

---

# Ao final

O Code Generator sabe exatamente.

Quais funções importar.

---

# Criando um plugin

É extremamente simples.

```javascript
function transformMeuPlugin(

node,

context

){

}
```

Depois registramos.

```javascript
transform(

ast,

{

nodeTransforms:[

transformMeuPlugin

]

}
)
```

---

# Exemplo

Imagine.

Queremos transformar.

```vue
<titulo>

</titulo>
```

Automaticamente.

Em.

```vue
<h1>

</h1>
```

Nosso plugin faria exatamente isso.

---

# Outro exemplo

Podemos encontrar.

Todos os.

```vue
<img>
```

E adicionar.

```vue
loading="lazy"
```

Automaticamente.

Tudo durante o Compiler.

---

# Percorrendo a AST

Fluxo.

```text
Root

↓

Element

↓

Interpolation
```

Cada plugin é chamado para todos os nós.

---

# Múltiplos plugins

```javascript
nodeTransforms:[

transformExpression,

transformElement,

transformMeuPlugin,

transformText

]
```

Todos trabalham juntos.

---

# Ordem importa

Imagine.

Primeiro.

```text
transformExpression
```

Depois.

```text
transformText
```

Se invertermos.

O resultado pode mudar completamente.

---

# AST antes

```text
Interpolation

↓

nome
```

---

# AST depois

```text
Interpolation

↓

_ctx.nome

↓

Helper

↓

PatchFlag
```

Observe.

A árvore vai ficando cada vez mais rica.

---

# CodegenNode

No final dos Transforms.

Cada Elemento recebe.

```javascript
node.codegenNode
```

Esse campo será utilizado diretamente pelo Code Generator.

---

# Exemplo

Antes.

```javascript
{

type:"Element"

}
```

Depois.

```javascript
{

type:"Element",

codegenNode:{

...

}

}
```

---

# Fluxo completo

```text
Parser

↓

AST

↓

Visitor

↓

Node Transform

↓

Directive Transform

↓

Helpers

↓

CodegenNode

↓

Code Generator
```

---

# Arquivos reais do Vue

Grande parte dessa arquitetura está em:

```text
packages/compiler-core/src/transform.ts
```

E também.

```text
transforms/

├── transformElement.ts

├── transformExpression.ts

├── transformText.ts

├── vIf.ts

├── vFor.ts
```

Ao explorar esses arquivos, você reconhecerá praticamente todos os conceitos apresentados neste capítulo.

---

# Performance

Uma vantagem enorme dessa arquitetura.

Os Transforms acontecem apenas uma vez.

Durante o build.

O Runtime nunca mais precisa descobrir.

* se existe `v-if`;
* se existe `v-for`;
* como gerar um componente;
* como resolver Slots.

Tudo já está pronto.

---

# Comparando

Nossa MiniVue.

```text
AST

↓

Transform

↓

Code
```

Vue.

```text
AST

↓

Visitors

↓

Node Transforms

↓

Directive Transforms

↓

Helpers

↓

Patch Flags

↓

Block Tree

↓

CodegenNode

↓

Code
```

---

# Curiosidade

Os Transforms do Vue são independentes entre si.

Isso significa que é possível adicionar, remover ou substituir transformações sem alterar o restante do compilador. Essa arquitetura modular foi um dos grandes objetivos da reescrita do Vue 3 e tornou possível criar ferramentas como o compilador de `<script setup>`, macros experimentais e transformações específicas utilizadas por frameworks como Nuxt.

---

# Resumo

Neste capítulo aprendemos que:

* O Transform Pipeline modifica a AST.
* O Compiler utiliza o padrão Visitor.
* Existem Node Transforms e Directive Transforms.
* Os Helpers são registrados durante os Transforms.
* Cada Elemento recebe um `codegenNode`.
* É possível criar plugins personalizados para o Compiler.

---

# Exercícios

## Exercício 1

Implemente um Visitor que percorra todos os nós da AST.

---

## Exercício 2

Implemente um `transformExpression()` simplificado.

---

## Exercício 3

Crie um Transform que converta todos os elementos `<titulo>` em `<h1>`.

---

## Exercício 4

Implemente um sistema de registro de Helpers utilizando um `Set`.

---

## Exercício 5

Crie um plugin que adicione automaticamente o atributo `loading="lazy"` em todas as imagens.

---

# Desafio

Atualize sua **MiniVue Compiler** para suportar:

* Visitor Pattern;
* Transform Context;
* Node Transforms;
* registro de Helpers;
* `codegenNode`;
* plugins personalizados.

---

# Projeto do capítulo

Ao final deste capítulo sua biblioteca deverá conseguir:

* percorrer toda a AST;
* aplicar múltiplos Transforms;
* registrar Helpers automaticamente;
* enriquecer os nós com informações para geração de código;
* permitir a criação de plugins do compilador.

---

# Checklist

* [ ] Sei explicar o Transform Pipeline.
* [ ] Entendi o padrão Visitor.
* [ ] Sei criar Node Transforms.
* [ ] Entendi a diferença entre Node e Directive Transforms.
* [ ] Sei registrar Helpers.
* [ ] Entendi o papel do `codegenNode`.
* [ ] Minha MiniVue já possui um sistema de Transforms.

---

# Próximo capítulo

## **Capítulo 21 — Code Generation: Gerando Render Functions do Zero**

No próximo capítulo construiremos a última etapa do compilador: o **Code Generator**. Você aprenderá como transformar a AST enriquecida em código JavaScript válido, implementará um gerador de código semelhante ao do Vue (`generate()`), criará um contexto de escrita, emitirá Helpers, Render Functions e chamadas para `createElementVNode()`, entendendo exatamente como um `<template>` se transforma em uma função `render()` pronta para ser executada pelo Runtime.
