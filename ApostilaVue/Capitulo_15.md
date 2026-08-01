# Capítulo 15 — O Runtime do Vue: Como um Componente Nasce, Renderiza e é Atualizado

> **Objetivo:** entender como o Vue transforma um componente (`.vue`) em uma instância viva na aplicação. Ao final deste capítulo você compreenderá como funciona o `createApp()`, como nasce um `ComponentInternalInstance`, como `setup()` é executado, como o Render Effect é criado e como o Scheduler atualiza automaticamente a interface.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Explicar o ciclo completo de criação de um componente.
* Entender o Runtime Core do Vue.
* Compreender o papel do `createApp()`.
* Entender a estrutura de um `ComponentInternalInstance`.
* Explicar como `setup()` funciona.
* Entender como nasce o Render Effect.
* Compreender como o Scheduler atualiza o DOM.

---

# Pré-requisitos

* Capítulos 01 ao 14.

---

# Mudança de fase

Até agora estudamos praticamente todo o pacote:

```text
@vue/reactivity
```

Foi nele que implementamos:

* `reactive()`
* `ref()`
* `computed()`
* `watch()`
* `effect()`

Mas isso, sozinho, não renderiza nada.

Ainda falta responder uma pergunta.

> Como um componente Vue realmente aparece na tela?

---

# A arquitetura do Vue

O Vue é dividido em vários pacotes.

```text
@vue/reactivity

↓

@vue/runtime-core

↓

@vue/runtime-dom
```

Cada um possui uma responsabilidade.

---

# @vue/reactivity

Responsável por:

* refs;
* reactive;
* computed;
* watch;
* effect.

Nenhuma linha de código conhece HTML.

---

# @vue/runtime-core

Responsável por:

* componentes;
* Virtual DOM;
* renderização;
* lifecycle;
* slots;
* props;
* emits;
* Scheduler.

Ainda não conhece o navegador.

---

# @vue/runtime-dom

Finalmente.

Este pacote sabe fazer:

```javascript
document.createElement()

appendChild()

removeChild()

setAttribute()
```

É ele que conversa com o navegador.

---

# Fluxo completo

```text
Seu componente

↓

Runtime Core

↓

Virtual DOM

↓

Runtime DOM

↓

DOM Real
```

---

# O ponto de entrada

Toda aplicação começa aqui.

```javascript
createApp(App)
```

Mas...

O que realmente acontece?

---

# Primeira etapa

O Vue cria uma aplicação.

```javascript
const app = createApp(App)
```

Internamente.

Algo parecido com:

```javascript
function createApp(root){

    return {

        mount(){}

    }

}
```

Ainda nada foi renderizado.

---

# mount()

Quando fazemos.

```javascript
app.mount("#app")
```

O Vue finalmente inicia a renderização.

---

# Fluxo

```text
createApp()

↓

mount()

↓

Componente raiz

↓

Render
```

---

# O componente

Imagine.

```vue
<script setup>

const contador = ref(0)

</script>

<template>

<p>{{ contador }}</p>

</template>
```

O Vue ainda não criou nada.

Ele apenas possui a definição do componente.

---

# Component Definition

Internamente.

Algo parecido com.

```javascript
const App = {

    setup(){},

    render(){}

}
```

Isso ainda não é um componente vivo.

É apenas uma definição.

---

# Surge a instância

Quando fazemos.

```javascript
mount()
```

O Vue cria.

```text
ComponentInternalInstance
```

Essa é a estrutura mais importante do Runtime.

---

# ComponentInternalInstance

Simplificando.

```javascript
const instance = {

}
```

Na prática.

Ela possui dezenas de propriedades.

---

# Algumas delas

```text
uid

type

parent

root

appContext

proxy

setupState

props

attrs

slots

ctx

render

subTree

effect

isMounted

update
```

Tudo que acontece no componente passa por essa instância.

---

# Visualizando

```text
App

↓

ComponentInternalInstance

↓

Tudo do componente
```

---

# uid

Cada componente recebe um identificador.

```text
1

2

3

4
```

Muito utilizado pelo Scheduler.

---

# parent

Imagine.

```vue
<App>

    <Header />

</App>
```

Temos.

```text
App

↓

Header
```

O Header guarda referência para o pai.

---

# root

Independentemente da profundidade.

Todo componente conhece o componente raiz.

---

# props

As Props ficam armazenadas aqui.

```javascript
instance.props
```

---

# attrs

Tudo que não for Prop.

Por exemplo.

```vue
<MeuBotao

class="ativo"

id="teste"

/>
```

Se não forem Props declaradas.

Entram em.

```javascript
instance.attrs
```

---

# slots

Todos os Slots.

```javascript
instance.slots
```

Também ficam armazenados.

---

# setupState

Imagine.

```javascript
setup(){

    const contador = ref(0)

    return {

        contador

    }

}
```

O retorno do Setup fica em.

```javascript
instance.setupState
```

---

# ctx

O contexto público.

Utilizado principalmente pelo Proxy.

---

# proxy

Quando escrevemos.

```vue
{{ contador }}
```

Na verdade.

O Vue faz.

```javascript
instance.proxy.contador
```

Esse Proxy decide.

```text
Existe em setupState?

↓

Existe em props?

↓

Existe em data?

↓

Existe em ctx?
```

---

# Fluxo

```text
contador

↓

Proxy

↓

setupState

↓

props

↓

ctx
```

---

# Executando o setup()

Agora.

O Vue chama.

```javascript
setup()
```

---

# O retorno

Se retornar.

```javascript
return {

    contador

}
```

Vai para.

```javascript
setupState
```

---

# E se retornar uma função?

Então.

Essa função torna-se o Render Function.

```javascript
setup(){

    return ()=>{

        ...

    }

}
```

---

# Template

Quando utilizamos.

```vue
<template>

<p>{{ contador }}</p>

</template>
```

O compilador gera.

```javascript
render(){

}
```

Ou seja.

No Runtime.

Template e Render Function tornam-se praticamente a mesma coisa.

---

# O Render Effect

Agora chegamos ao coração do Runtime.

O Vue cria.

```javascript
new ReactiveEffect(render)
```

Sim.

O Render inteiro do componente é apenas um Effect.

---

# Isso muda tudo

Imagine.

```vue
<p>{{ contador }}</p>
```

Durante o Render.

Executamos.

```javascript
contador.value
```

Lembra do capítulo sobre `track()`?

Agora.

O Render Effect torna-se dependente do Ref.

---

# Visualizando

```text
Render Effect

↓

contador.value

↓

track()

↓

Ref

↓

dep

↓

Render Effect
```

---

# Quando muda

```javascript
contador.value++
```

O Ref faz.

```text
trigger()

↓

Scheduler

↓

Render Effect
```

O componente inteiro é atualizado automaticamente.

---

# update()

Cada componente possui uma função.

```javascript
instance.update()
```

Ela executa.

```javascript
effect.run()
```

Ou.

Se houver Scheduler.

Agenda uma atualização.

---

# Primeira renderização

Fluxo.

```text
mount()

↓

setup()

↓

render()

↓

VNode

↓

Patch

↓

DOM
```

---

# Atualizações

Depois.

```text
contador++

↓

trigger()

↓

Scheduler

↓

update()

↓

render()

↓

Novo VNode

↓

Patch

↓

DOM
```

---

# subTree

Após renderizar.

O componente guarda.

```javascript
instance.subTree
```

Que contém.

```text
Virtual DOM anterior
```

---

# Próxima renderização

Novo Render.

↓

Novo Virtual DOM.

↓

Comparação.

```text
VNode antigo

↓

VNode novo

↓

Patch
```

---

# Patch

O Patch decide.

```text
O que realmente mudou?
```

Não recria tudo.

Atualiza apenas o necessário.

---

# isMounted

Durante a primeira renderização.

```javascript
instance.isMounted
```

Vale.

```text
false
```

Depois.

```text
true
```

Isso altera completamente o fluxo interno.

---

# Primeira renderização

```text
isMounted

↓

false

↓

Criar DOM
```

---

# Atualização

```text
isMounted

↓

true

↓

Patch
```

---

# O Scheduler

O Render Effect nunca executa diretamente.

Ele possui um Scheduler.

```javascript
new ReactiveEffect(

render,

scheduler

)
```

Assim.

Várias alterações consecutivas.

```javascript
contador.value++

contador.value++

contador.value++
```

Resultam em apenas uma renderização.

---

# Hierarquia

Imagine.

```text
App

↓

Layout

↓

Sidebar

↓

Menu
```

Cada componente possui.

```text
Instance

↓

Render Effect
```

Todos compartilham o mesmo Scheduler.

---

# Render Function

Simplificando.

O compilador transforma.

```vue
<p>{{ contador }}</p>
```

Em algo parecido com.

```javascript
render(){

    return h(

        "p",

        contador.value

    )

}
```

Ainda não estudaremos `h()`.

Isso acontecerá nos próximos capítulos.

---

# Arquitetura completa

```text
Componente

↓

Component Definition

↓

createComponentInstance()

↓

setup()

↓

setupState

↓

render()

↓

ReactiveEffect

↓

VNode

↓

Patch

↓

DOM
```

Esse é exatamente o caminho percorrido pelo Vue.

---

# Comparando

Nos capítulos anteriores.

```text
Ref

↓

Effect
```

Agora.

```text
Ref

↓

Render Effect

↓

Scheduler

↓

Render

↓

Virtual DOM

↓

Patch

↓

DOM
```

Toda a arquitetura começa a se conectar.

---

# Resumo

Neste capítulo aprendemos que:

* `createApp()` cria uma aplicação.
* `mount()` inicia a renderização.
* Cada componente possui um `ComponentInternalInstance`.
* `setup()` popula o `setupState`.
* O Render inteiro é um `ReactiveEffect`.
* O Scheduler controla as atualizações.
* Cada componente guarda seu Virtual DOM anterior em `subTree`.
* O Patch atualiza apenas o necessário.

---

# Exercícios

## Exercício 1

Desenhe o fluxo completo desde `createApp()` até a criação do DOM.

---

## Exercício 2

Explique por que o Render de um componente pode ser tratado como um `ReactiveEffect`.

---

## Exercício 3

Liste as principais propriedades de um `ComponentInternalInstance` e explique a função de cada uma.

---

## Exercício 4

Explique o papel de `subTree` durante uma atualização.

---

# Desafio

Atualize sua biblioteca **MiniVue** para suportar:

* `createApp()`;
* `mount()`;
* criação de uma instância de componente;
* `setup()`;
* `setupState`;
* `ComponentInternalInstance`;
* criação de um Render Effect;
* atualização através do Scheduler.

---

# Projeto do capítulo

Ao final deste capítulo sua biblioteca deverá ser capaz de:

* criar uma instância de componente;
* executar `setup()`;
* armazenar `setupState`;
* criar um Render Effect;
* reagir automaticamente às mudanças dos `ref()` e `reactive()`;
* diferenciar montagem inicial e atualizações.

---

# Checklist

* [ ] Entendi a diferença entre definição e instância de componente.
* [ ] Sei explicar como `createApp()` funciona.
* [ ] Entendi a estrutura de `ComponentInternalInstance`.
* [ ] Sei explicar por que o Render é um `ReactiveEffect`.
* [ ] Entendi como o Scheduler atualiza componentes.
* [ ] Minha MiniVue já consegue montar um componente simples.

---

# Próximo capítulo

## **Capítulo 16 — Virtual DOM: Como o Vue Cria, Compara e Atualiza a Árvore de VNodes**

Agora entraremos no coração do algoritmo de renderização do Vue. Você aprenderá o que é um **VNode**, como funciona a função `h()`, por que o Virtual DOM existe, como o algoritmo de **Patch** compara duas árvores, o papel das **Shape Flags**, **Patch Flags**, **Block Tree**, otimizações do compilador e como o Vue consegue atualizar milhares de elementos realizando o menor número possível de operações no DOM. Este será um dos capítulos mais importantes de todo o material.
