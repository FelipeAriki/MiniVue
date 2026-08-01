# Capítulo 28 — Otimizações do Compiler: Patch Flags, Shape Flags, Block Tree e Renderização Otimizada

> **Objetivo:** compreender como o Vue 3 utiliza informações geradas pelo **Compiler** para tornar o Renderer extremamente eficiente. Ao final deste capítulo você entenderá como funcionam **Patch Flags**, **Shape Flags**, **Static Hoisting**, **Tree Flattening** e **Block Tree**, que são algumas das maiores inovações arquiteturais do Vue 3.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Entender como Compiler e Renderer trabalham juntos.
* Explicar o conceito de Patch Flags.
* Entender Shape Flags.
* Compreender Static Hoisting.
* Explicar Tree Flattening.
* Entender Block Tree.
* Saber por que o Vue evita percorrer toda a árvore em cada atualização.

---

# Pré-requisitos

* Capítulos 01 ao 27.

---

# O problema

Imagine este componente.

```vue
<template>

<div>

<h1>Vue 3</h1>

<p>{{ usuario }}</p>

<button @click="salvar">

Salvar

</button>

</div>

</template>
```

Quando.

```javascript
usuario.value = "Felipe"
```

muda.

Será que o Vue compara tudo?

* h1
* p
* button

?

---

# A resposta

Não.

Esse seria um desperdício enorme.

---

# O Compiler sabe

Durante a compilação.

O Vue já sabe.

```text
h1

↓

Nunca muda
```

---

Também sabe.

```text
button

↓

Nunca muda
```

---

E sabe.

```text
p

↓

Dinâmico
```

Então.

Por que comparar tudo?

---

# A solução

O Compiler gera informações extras.

Chamadas.

```text
Patch Flags
```

---

# O fluxo

```text
Template

↓

Compiler

↓

VNode

+

Patch Flags

↓

Renderer

↓

Patch
```

Agora.

O Renderer sabe exatamente.

O que verificar.

---

# Sem Patch Flags

Imagine.

```text
VNode

↓

Compara Props

↓

Compara Classes

↓

Compara Styles

↓

Compara Eventos

↓

Compara Children
```

Tudo.

Sempre.

---

# Com Patch Flags

O Compiler informa.

```text
Este elemento

↓

Possui apenas

Texto dinâmico
```

O Renderer compara.

Somente.

O texto.

---

# Exemplo

Template.

```vue
<p>

{{ nome }}

</p>
```

Compiler.

```javascript
createElementVNode(

"p",

null,

toDisplayString(nome),

1

)
```

Observe.

O último número.

```text
1
```

É uma Patch Flag.

---

# O que significa?

```text
1

↓

TEXT
```

O Renderer sabe.

Que apenas.

O texto.

Pode mudar.

---

# Outro exemplo

```vue
<div

:class="classe"

>

</div>
```

Compiler.

```javascript
createElementVNode(

"div",

{

class:classe

},

null,

2

)
```

Patch Flag.

```text
2

↓

CLASS
```

---

# Mais exemplos

```text
1

↓

TEXT
```

---

```text
2

↓

CLASS
```

---

```text
4

↓

STYLE
```

---

```text
8

↓

PROPS
```

---

```text
16

↓

FULL_PROPS
```

---

```text
32

↓

HYDRATE_EVENTS
```

---

```text
64

↓

STABLE_FRAGMENT
```

---

```text
128

↓

KEYED_FRAGMENT
```

---

```text
256

↓

UNKEYED_FRAGMENT
```

---

```text
512

↓

NEED_PATCH
```

---

```text
1024

↓

DYNAMIC_SLOTS
```

---

# Bit Flags

Observe.

```text
1

2

4

8

16

32

64
```

Todos.

São.

Potências de dois.

---

# Por quê?

Porque podemos combinar.

```text
TEXT

+

CLASS
```

Resultado.

```text
1 | 2

=

3
```

Agora.

O Renderer sabe.

Que precisa atualizar.

Texto.

E.

Classe.

---

# Bitwise

O Vue verifica.

```javascript
if(

patchFlag & TEXT

){

...
}
```

Essa operação.

É extremamente rápida.

---

# Shape Flags

Agora.

Outro tipo de Flag.

---

Enquanto Patch Flags.

Informam.

O que mudou.

---

Shape Flags.

Informam.

O que o VNode é.

---

Exemplo.

```javascript
{

type:"div"

}
```

Possui.

```text
ELEMENT
```

---

```javascript
{

type:MeuComponente

}
```

Possui.

```text
COMPONENT
```

---

Também existem.

```text
TEXT_CHILDREN

ARRAY_CHILDREN

SLOTS_CHILDREN

TELEPORT

SUSPENSE
```

---

# Exemplo

```text
VNode

↓

ShapeFlag

↓

ELEMENT

+

ARRAY_CHILDREN
```

O Renderer.

Não precisa descobrir.

Já sabe.

---

# Sem Shape Flags

Precisaria fazer.

```javascript
typeof vnode.type

instanceof

Array.isArray

...
```

Toda vez.

---

Com Shape Flags.

É apenas.

```javascript
if(

shapeFlag & ARRAY_CHILDREN

)
```

Muito mais rápido.

---

# Static Hoisting

Agora.

Uma das otimizações.

Mais inteligentes.

---

Imagine.

```vue
<h1>

Vue 3

</h1>
```

Esse elemento.

Nunca muda.

---

Por que recriá-lo.

Em toda renderização?

---

O Compiler.

Move.

Esse VNode.

Para fora.

Da Render Function.

---

Antes.

```javascript
render(){

return

createVNode(...)
}
```

---

Depois.

```javascript
const _hoisted_1=

createVNode(...)
```

Render.

```javascript
render(){

return

_hoisted_1

}
```

---

Resultado.

O objeto.

É criado.

Uma única vez.

---

# Tree Flattening

Imagine.

```text
DIV

↓

SECTION

↓

ARTICLE

↓

P

↓

SPAN
```

Mas.

Apenas.

```text
SPAN
```

É dinâmico.

---

Sem otimização.

O Patch.

Percorreria.

Toda a árvore.

---

O Compiler.

Cria.

Uma lista.

Apenas.

Dos nós dinâmicos.

---

Resultado.

```text
SPAN
```

É encontrado.

Diretamente.

---

# Block Tree

Agora.

A maior inovação.

Do Vue 3.

---

O Compiler.

Divide.

A árvore.

Em.

```text
Blocks
```

---

Cada Block.

Contém.

Somente.

Nós dinâmicos.

---

Visualmente.

```text
App

↓

Header

↓

Block

↓

Button

↓

Input
```

---

O Renderer.

Ignora.

Todo o restante.

---

# openBlock()

O Compiler gera.

```javascript
openBlock()
```

---

Depois.

```javascript
createElementBlock(...)
```

---

Por quê?

Porque.

Cada Block.

Possui.

Uma lista.

Dos filhos dinâmicos.

---

# dynamicChildren

Internamente.

Existe.

```javascript
dynamicChildren
```

---

Exemplo.

```text
DIV

↓

dynamicChildren

↓

INPUT

BUTTON
```

---

Quando atualiza.

O Renderer.

Percorre.

Apenas.

Essa lista.

---

# Sem Block Tree

```text
1000 nós

↓

Patch

↓

1000 comparações
```

---

# Com Block Tree

```text
1000 nós

↓

10 dinâmicos

↓

10 comparações
```

---

# Exemplo

Template.

```vue
<div>

<h1>

Vue

</h1>

<p>

{{ nome }}

</p>

<footer>

2026

</footer>

</div>
```

---

O Block.

Contém.

Apenas.

```text
P
```

---

Nem.

```text
H1
```

Nem.

```text
FOOTER
```

Entram.

Na atualização.

---

# Compiler + Renderer

Agora.

Tudo faz sentido.

---

Compiler.

```text
Analisa

↓

Marca

↓

Otimiza
```

---

Renderer.

```text
Lê Flags

↓

Atualiza

↓

DOM
```

---

# Fluxo completo

```text
Template

↓

Compiler

↓

Patch Flags

↓

Shape Flags

↓

Block Tree

↓

Renderer

↓

Patch

↓

DOM
```

---

# Arquivos reais

Grande parte dessas otimizações está em.

```text
packages/compiler-core/src
```

Arquivos.

```text
transform.ts

hoistStatic.ts

transformElement.ts

ast.ts
```

---

No Runtime.

```text
packages/runtime-core/src
```

Arquivos.

```text
renderer.ts

vnode.ts
```

---

# Curiosidade

Essas otimizações são uma das principais razões pelas quais o Vue 3 conseguiu melhorar significativamente o desempenho em relação ao Vue 2. Em vez de depender apenas do Virtual DOM, o Vue combina **informações produzidas pelo Compiler** com o **Renderer**, reduzindo drasticamente o trabalho necessário durante cada atualização.

---

# Resumo

Neste capítulo aprendemos que:

* O Compiler produz informações utilizadas pelo Renderer.
* Patch Flags indicam exatamente o que pode mudar.
* Shape Flags descrevem o tipo do VNode.
* Static Hoisting evita recriar VNodes estáticos.
* Tree Flattening reduz a quantidade de nós percorridos.
* Block Tree permite atualizar apenas os nós realmente dinâmicos.

---

# Exercícios

## Exercício 1

Explique a diferença entre Patch Flags e Shape Flags.

---

## Exercício 2

Implemente um sistema simples de Flags utilizando operações bitwise.

---

## Exercício 3

Modifique sua MiniVue para ignorar VNodes marcados como estáticos.

---

## Exercício 4

Implemente uma lista simplificada de `dynamicChildren`.

---

## Exercício 5

Abra `packages/compiler-core/src/transformElement.ts` e identifique onde o Compiler decide quais Patch Flags gerar.

---

# Desafio

Atualize sua **MiniVue** para suportar:

* Patch Flags;
* Shape Flags;
* Static Hoisting;
* uma versão simplificada de Block Tree.

---

# Projeto do capítulo

Ao final deste capítulo sua biblioteca deverá:

* marcar VNodes estáticos;
* ignorar comparações desnecessárias;
* armazenar filhos dinâmicos;
* reduzir significativamente o trabalho do algoritmo de Patch.

---

# Checklist

* [ ] Entendi Patch Flags.
* [ ] Entendi Shape Flags.
* [ ] Sei explicar Static Hoisting.
* [ ] Entendi Block Tree.
* [ ] Sei explicar Tree Flattening.
* [ ] Minha MiniVue já possui otimizações básicas de renderização.

---

# Próximo capítulo

## **Capítulo 29 — Longest Increasing Subsequence (LIS): Implementando o Algoritmo de Movimentação de Nós do Vue 3**

No próximo capítulo faremos um mergulho completo no algoritmo **Longest Increasing Subsequence**, implementando-o do zero e acompanhando, linha por linha, como o Vue utiliza essa técnica dentro de `patchKeyedChildren()` para minimizar movimentações no DOM. Será um dos capítulos mais avançados de toda a série, conectando teoria de algoritmos com a implementação real do Vue.
