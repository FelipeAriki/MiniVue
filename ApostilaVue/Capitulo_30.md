# Capítulo 30 — Render Functions e `h()`: Criando Interfaces sem Templates

> **Objetivo:** compreender profundamente como o Vue cria interfaces sem utilizar templates. Ao final deste capítulo você entenderá como funciona a função `h()`, como escrever Render Functions profissionais, como elas se conectam ao Virtual DOM e quando utilizá-las em projetos reais.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Entender como funciona uma Render Function.
* Explicar o papel da função `h()`.
* Criar componentes sem `<template>`.
* Entender quando Render Functions são superiores aos templates.
* Compreender como bibliotecas utilizam Render Functions.
* Implementar sua própria função `h()` na MiniVue.

---

# Pré-requisitos

* Capítulos 01 ao 29.

---

# Recapitulando

Até agora utilizamos.

```vue
<template>

<div>

{{ nome }}

</div>

</template>
```

Mas...

O Vue realmente interpreta HTML?

---

# A resposta

Não.

O Template.

É apenas.

Uma linguagem.

Mais amigável.

---

Internamente.

Tudo vira.

```javascript
render(){

    return ...

}
```

---

# O fluxo

```text
Template

↓

Compiler

↓

Render Function

↓

VNode

↓

Renderer

↓

DOM
```

---

# Então...

O Template.

É opcional.

---

Podemos escrever.

Diretamente.

A Render Function.

---

# Exemplo

Ao invés de:

```vue
<template>

<h1>

Vue 3

</h1>

</template>
```

Podemos escrever.

```javascript
render(){

    return h(

        "h1",

        "Vue 3"

    )

}
```

Resultado.

Exatamente.

O mesmo.

---

# O que é `h()`?

A função.

```javascript
h()
```

Significa.

```text
Hyperscript
```

---

Historicamente.

Hyperscript.

Era uma forma.

De representar.

HTML.

Utilizando.

Funções.

---

# Exemplo

```javascript
h(

"div",

"Olá"

)
```

Representa.

```html
<div>

Olá

</div>
```

---

# Outro exemplo

```javascript
h(

"button",

{

class:"primary"

},

"Salvar"

)
```

Produz.

```html
<button class="primary">

Salvar

</button>
```

---

# Como funciona?

A função.

Não cria.

HTML.

---

Ela cria.

Um.

```text
VNode
```

---

Exemplo.

```javascript
h(

"div"

)
```

Internamente.

É semelhante a.

```javascript
createVNode(

"div"

)
```

---

# Assinatura

Simplificando.

```javascript
h(

type,

props,

children

)
```

---

Onde.

```text
type
```

Pode ser.

* elemento
* componente
* Fragment
* Teleport
* Suspense

---

# Elementos

```javascript
h(

"div"

)
```

---

```javascript
h(

"button"

)
```

---

```javascript
h(

"input"

)
```

---

# Props

```javascript
h(

"button",

{

disabled:true,

class:"primary"

}

)
```

---

# Eventos

```javascript
h(

"button",

{

onClick(){

    salvar()

}

}

)
```

---

Observe.

Não usamos.

```vue
@click
```

---

Usamos.

```javascript
onClick
```

---

# Children

Texto.

```javascript
h(

"p",

"Olá"

)
```

---

Array.

```javascript
h(

"ul",

[

h("li","A"),

h("li","B")

]

)
```

---

# Componentes

Também podem.

Ser renderizados.

---

```javascript
h(

MeuBotao
)
```

---

Ou.

```javascript
h(

MeuBotao,

{

titulo:"Salvar"

}
)
```

---

# Slots

Também são.

Funções.

---

```javascript
h(

MeuCard,

{},

{

default(){

return

"Conteúdo"

}

}

)
```

---

Observe.

Slots.

São funções.

---

# Render Function

Um componente.

Pode ser escrito.

Assim.

```javascript
export default{

render(){

return

h(

"h1",

"Vue"

)

}

}
```

Sem.

Template.

---

# Composition API

Também funciona.

```javascript
export default{

setup(){

return()=>{

return h(

"h1",

"Vue"

)

}

}

}
```

---

Observe.

`setup()`

Retorna.

Uma função.

---

Essa função.

É.

A Render Function.

---

# JSX

Quem conhece React.

Provavelmente já viu.

```jsx
return(

<div>

Olá

</div>

)
```

---

Isso.

Também é.

Uma Render Function.

---

O JSX.

Compila.

Para.

```javascript
h(...)
```

---

# Template

```vue
<div>

Olá

</div>
```

↓

---

Render Function.

```javascript
h(

"div",

"Olá"

)
```

↓

---

VNode.

---

# Componentes dinâmicos

Imagine.

```javascript
const componente=

admin

?

Admin

:

User
```

---

Renderizamos.

```javascript
h(

componente
)
```

Muito simples.

---

# Loops

Template.

```vue
<li

v-for="item in lista"

>

{{ item }}

</li>
```

---

Render Function.

```javascript
lista.map(item=>

h(

"li",

item

)

)
```

---

# Condições

Template.

```vue
<div

v-if="admin"

>

Admin

</div>
```

---

Render Function.

```javascript
admin

?

h("div","Admin")

:

null
```

---

# Fragment

Também pode.

Ser criado.

---

```javascript
h(

Fragment,

null,

[

...

]

)
```

---

# Teleport

Também.

```javascript
h(

Teleport,

{

to:"#modal"

},

...

)
```

---

# Suspense

Também.

```javascript
h(

Suspense,

...

)
```

---

# Por que usar?

Na maioria.

Dos projetos.

Templates.

São melhores.

---

Mas existem.

Casos.

Onde.

Render Functions.

São superiores.

---

# Bibliotecas

PrimeVue.

Element Plus.

Naive UI.

Vuetify.

Utilizam.

Render Functions.

Em diversos pontos.

---

# Vue Router

Quando define.

Uma View.

Internamente.

Ele cria.

VNodes.

---

# Pinia

Também utiliza.

Render Functions.

Em alguns.

Componentes.

Auxiliares.

---

# Devtools

Utilizam.

Render Functions.

Para gerar.

Interfaces.

Dinamicamente.

---

# Quando utilizar?

Quando.

A interface.

É altamente.

Dinâmica.

---

Exemplo.

Um Table Builder.

---

Imagine.

```javascript
colunas

↓

API
```

Cada coluna.

É diferente.

---

Criar.

Templates.

Seria complicado.

---

Render Function.

Resolve.

Muito melhor.

---

# Outro exemplo

Um Form Builder.

---

Campos.

Vindos.

Do servidor.

---

Cada campo.

Possui.

Componentes.

Diferentes.

---

Render Functions.

São ideais.

---

# Como implementar?

Na MiniVue.

Podemos criar.

```javascript
function h(

type,

props,

children

){

return{

type,

props,

children

}

}
```

---

Observe.

Ela apenas.

Cria.

Um VNode.

---

Depois.

O Renderer.

Faz.

O restante.

---

# Fluxo

```text
h()

↓

VNode

↓

Patch

↓

Renderer

↓

DOM
```

---

# createVNode()

Na verdade.

Hoje.

A função.

```javascript
h()
```

É apenas.

Um atalho.

Para.

```javascript
createVNode()
```

---

# Arquivos reais

A implementação está em.

```text
packages/runtime-core/src
```

Arquivos importantes.

```text
h.ts

vnode.ts
```

---

Também vale a leitura de.

```text
componentRenderUtils.ts
```

Pois ele conecta.

As Render Functions.

Ao Renderer.

---

# Curiosidade

Antes da popularização dos templates declarativos, diversas bibliotecas JavaScript utilizavam APIs inspiradas em **Hyperscript** para construir interfaces. O Vue mantém essa tradição através da função `h()`, mas oferece os templates como uma camada mais amigável, compilando ambos para a mesma representação interna: **VNodes**.

---

# Resumo

Neste capítulo aprendemos que:

* Templates são compilados para Render Functions.
* A função `h()` cria VNodes.
* Render Functions podem substituir completamente os templates.
* JSX também é compilado para chamadas de `h()`.
* Bibliotecas utilizam Render Functions para gerar interfaces dinâmicas.
* `h()` é um atalho para `createVNode()`.

---

# Exercícios

## Exercício 1

Reescreva três componentes utilizando apenas Render Functions.

---

## Exercício 2

Implemente uma função `h()` simplificada na sua MiniVue.

---

## Exercício 3

Crie um componente que renderize dinamicamente uma lista de componentes utilizando `h()`.

---

## Exercício 4

Implemente suporte a slots em Render Functions.

---

## Exercício 5

Abra `packages/runtime-core/src/h.ts` e compare a implementação oficial com a da sua MiniVue.

---

# Desafio

Atualize sua **MiniVue** para:

* implementar `h()`;
* implementar `createVNode()`;
* permitir componentes escritos apenas com Render Functions;
* suportar elementos, componentes e arrays de filhos.

---

# Projeto do capítulo

Ao final deste capítulo sua MiniVue deverá permitir a criação de interfaces inteiramente por código, sem necessidade de templates, aproximando-se ainda mais da arquitetura utilizada pelo Vue oficial.

---

# Checklist

* [ ] Sei explicar o que é uma Render Function.
* [ ] Entendi o funcionamento da função `h()`.
* [ ] Consigo criar componentes sem `<template>`.
* [ ] Sei quando utilizar Render Functions em vez de templates.
* [ ] Minha MiniVue possui uma implementação básica de `h()`.

---

# Próximo capítulo

## **Capítulo 31 — JSX no Vue 3: Como Funciona, Como o Babel Compila e Quando Utilizar**

No próximo capítulo estudaremos **JSX** em profundidade. Você aprenderá como o Babel transforma JSX em chamadas para `h()`, como utilizar componentes, diretivas, eventos, slots e TypeScript com JSX no Vue, além de entender por que algumas bibliotecas preferem JSX em vez de templates. Será um mergulho completo na alternativa declarativa oferecida pelo ecossistema Vue.
