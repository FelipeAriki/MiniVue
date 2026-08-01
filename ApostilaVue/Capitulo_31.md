# Capítulo 31 — JSX no Vue 3: Como Funciona, Como o Babel Compila e Quando Utilizar

> **Objetivo:** compreender profundamente o JSX no Vue 3. Ao final deste capítulo você será capaz de escrever aplicações inteiras utilizando JSX, entender como ele é compilado para `h()`, conhecer as diferenças em relação ao JSX do React e saber quando essa abordagem é realmente vantajosa.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Entender o que é JSX.
* Explicar como o Babel transforma JSX em Render Functions.
* Utilizar JSX com Composition API.
* Trabalhar com eventos, slots e diretivas em JSX.
* Entender as diferenças entre Vue JSX e React JSX.
* Saber quando utilizar JSX em projetos reais.

---

# Pré-requisitos

* Capítulos 01 ao 30.

---

# Recapitulando

No capítulo anterior aprendemos.

```text
Template

↓

Render Function

↓

h()

↓

VNode
```

Mas escrever isto.

```javascript
h(
    "div",
    [
        h("h1","Vue"),
        h("button","Salvar")
    ]
)
```

Pode ficar difícil de ler.

---

# A solução

Existe uma sintaxe.

Muito mais agradável.

Chamada.

```text
JSX
```

---

# Exemplo

Ao invés de escrever.

```javascript
h(
    "h1",
    "Vue 3"
)
```

Podemos escrever.

```jsx
<h1>Vue 3</h1>
```

Muito mais próximo.

Do HTML.

---

# Importante

JSX.

Não é HTML.

---

Também.

Não é Template.

---

JSX.

É JavaScript.

---

# O que acontece?

Quando escrevemos.

```jsx
<div>

Olá

</div>
```

O navegador.

Nunca vê isso.

---

Primeiro.

O Babel.

Transforma.

Em.

```javascript
h(
    "div",
    "Olá"
)
```

---

Depois.

O Vue cria.

```text
VNode
```

---

Depois.

O Renderer.

Atualiza.

O DOM.

---

# Fluxo

```text
JSX

↓

Babel

↓

Render Function

↓

h()

↓

VNode

↓

Patch

↓

DOM
```

---

# Configuração

Em projetos Vue.

Normalmente utilizamos.

O plugin.

```text
@vitejs/plugin-vue-jsx
```

---

No Vite.

```javascript
import vue from "@vitejs/plugin-vue"

import vueJsx from "@vitejs/plugin-vue-jsx"

export default{

plugins:[

vue(),

vueJsx()

]

}
```

---

# Primeiro componente

```javascript
import{

defineComponent

}from"vue"

export default defineComponent({

setup(){

return()=>{

return(

<h1>

Vue 3

</h1>

)

}

}

})
```

---

Observe.

`setup()`.

Retorna.

Uma função.

---

Essa função.

É a Render Function.

---

# Elementos

```jsx
<div></div>
```

↓

```javascript
h("div")
```

---

```jsx
<button>

Salvar

</button>
```

↓

```javascript
h(

"button",

"Salvar"

)
```

---

# Props

```jsx
<input

value="Felipe"

/>
```

↓

```javascript
h(

"input",

{

value:"Felipe"

}

)
```

---

# Classes

```jsx
<div

class="card"

/>
```

---

Também podemos.

```jsx
<div

class={[

"card",

ativo&&"ativo"

]}

/>
```

---

# Styles

```jsx
<div

style={{

color:"red",

fontSize:"18px"

}}

/>
```

---

# Eventos

Templates.

```vue
@click
```

---

JSX.

```jsx
<button

onClick={salvar}

/>
```

---

Outro exemplo.

```jsx
<input

onInput={

atualizar

}

/>
```

---

Observe.

Sempre usamos.

```text
onNomeEvento
```

---

# Eventos inline

```jsx
<button

onClick={()=>{

contador.value++

}}

/>
```

---

# Diretivas

Aqui.

Existe.

Uma diferença.

---

Não podemos escrever.

```vue
v-if
```

---

Em JSX.

Utilizamos.

JavaScript.

---

Exemplo.

```jsx
{

admin

&&

<div>

Admin

</div>

}
```

---

Ou.

```jsx
{

admin

?

<div>

Admin

</div>

:

null

}
```

---

# Loops

Template.

```vue
v-for
```

---

JSX.

```jsx
lista.map(item=>

<li>

{item.nome}

</li>

)
```

---

Muito parecido.

Com React.

---

# Interpolação

Template.

```vue
{{ nome }}
```

---

JSX.

```jsx
{

nome.value

}
```

---

# Componentes

```jsx
<MeuBotao/>
```

↓

```javascript
h(MeuBotao)
```

---

Props.

```jsx
<MeuBotao

titulo="Salvar"

/>
```

---

Slots.

```jsx
<MeuCard>

Conteúdo

</MeuCard>
```

---

Também podemos.

Passar.

Funções.

---

```jsx
<MeuCard>

{{

default:()=>(
<div>

Conteúdo

</div>
)

}}

</MeuCard>
```

---

# Fragment

Sem elemento pai.

```jsx
<>

<h1/>

<p/>

</>
```

↓

Fragment.

---

# Renderização dinâmica

```jsx
const Comp=

admin

?

Admin

:

User

return<Comp/>
```

---

Muito elegante.

---

# Componentes funcionais

```javascript
const Botao=()=>(

<button>

Salvar

</button>

)
```

---

Não existe.

Estado interno.

---

Apenas.

Renderização.

---

# Composição

Podemos usar.

Tudo.

Da Composition API.

---

```jsx
const nome=

ref("Felipe")
```

↓

```jsx
return()=>(

<h1>

{nome.value}

</h1>

)
```

---

# Watch

Também funciona.

---

```javascript
watch(

contador,

...

)
```

---

Nada muda.

---

# Computed

```javascript
const total=

computed(...)
```

↓

```jsx
{

total.value

}
```

---

# Diferença para React

Visualmente.

São parecidos.

---

Mas.

Internamente.

São diferentes.

---

React.

Compila.

Para.

```javascript
React.createElement()
```

---

Vue.

Compila.

Para.

```javascript
h()
```

---

Outra diferença.

React.

Não possui.

Diretivas.

---

Vue.

Também não.

Quando usamos.

JSX.

---

Tudo.

É.

JavaScript.

---

# Quando utilizar?

A maioria.

Dos projetos.

Utiliza.

Templates.

---

Mas.

JSX.

É excelente.

Para.

---

Builders.

---

Tables.

---

Trees.

---

Menus.

---

Renderização.

Muito dinâmica.

---

# Bibliotecas

Grande parte.

Das bibliotecas.

Utiliza.

JSX.

Ou.

Render Functions.

---

Porque.

É muito mais fácil.

Gerar.

Interface.

Por código.

---

# Performance

Existe diferença?

---

Praticamente.

Não.

---

Ambos.

Produzem.

```text
VNode
```

---

A diferença.

Está.

Na experiência.

De desenvolvimento.

---

# Vantagens

* Muito flexível.
* Todo o poder do JavaScript.
* Excelente para componentes altamente dinâmicos.
* Facilita abstrações complexas.

---

# Desvantagens

* Menor legibilidade para quem prefere HTML.
* Curva de aprendizado maior.
* Mais verboso em componentes simples.

---

# Como o Babel funciona?

Simplificando.

Ele percorre.

A AST.

Do JSX.

---

Transforma.

Cada elemento.

Em.

```javascript
h(...)
```

---

Depois.

O Vue.

Executa.

A Render Function.

Normalmente.

---

# Fluxo completo

```text
JSX

↓

Parser

↓

AST

↓

Babel Plugin

↓

Render Function

↓

h()

↓

VNode

↓

Patch

↓

DOM
```

---

# Arquivos reais

O suporte oficial está no plugin:

```text
@vue/babel-plugin-jsx
```

Principais arquivos para estudo:

```text
packages/babel-plugin-jsx/
```

Vale observar como o plugin transforma:

* elementos JSX;
* eventos;
* slots;
* fragments;
* componentes.

---

# Curiosidade

Embora o JSX seja frequentemente associado ao React, ele **não pertence ao React**. JSX é apenas uma extensão de sintaxe para JavaScript. Cada framework implementa sua própria transformação. No Vue, o JSX gera chamadas para `h()`, enquanto no React moderno ele gera chamadas para funções do runtime JSX do React.

---

# Resumo

Neste capítulo aprendemos que:

* JSX é uma sintaxe para escrever Render Functions.
* O Babel transforma JSX em chamadas para `h()`.
* JSX utiliza JavaScript para condições e loops.
* Templates e JSX produzem VNodes equivalentes.
* JSX é especialmente útil para componentes altamente dinâmicos.

---

# Exercícios

## Exercício 1

Reescreva três componentes do seu projeto utilizando JSX.

---

## Exercício 2

Implemente um componente de tabela completamente em JSX.

---

## Exercício 3

Crie um componente que renderize dinamicamente diferentes componentes utilizando variáveis.

---

## Exercício 4

Compare o código gerado por um template e por um componente JSX equivalente.

---

## Exercício 5

Explore o `@vue/babel-plugin-jsx` e identifique onde ocorre a transformação de elementos JSX em chamadas para `h()`.

---

# Desafio

Atualize sua **MiniVue** para permitir componentes escritos exclusivamente com Render Functions, simulando a saída produzida pelo Babel para JSX.

---

# Projeto do capítulo

Ao final deste capítulo sua MiniVue deverá aceitar componentes baseados em Render Functions, tornando-se compatível com a ideia central por trás do JSX.

---

# Checklist

* [ ] Sei explicar o que é JSX.
* [ ] Entendi como o Babel transforma JSX em `h()`.
* [ ] Consigo utilizar JSX com Composition API.
* [ ] Sei escrever eventos, loops e condições em JSX.
* [ ] Entendi quando JSX é mais adequado do que templates.

---

# Próximo capítulo

## **Capítulo 32 — Teleport: Renderizando Componentes Fora da Árvore do DOM**

No próximo capítulo estudaremos o **Teleport**, uma das funcionalidades mais elegantes do Vue 3. Você entenderá por que modais, tooltips, dropdowns e overlays utilizam Teleport, como ele funciona internamente no Renderer, como preserva a árvore de componentes mesmo renderizando em outro ponto do DOM e como implementar uma versão simplificada desse recurso na MiniVue.
