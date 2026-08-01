# Capítulo 26 — O Renderer do Vue: Implementando `createRenderer()` do Zero

> **Objetivo:** compreender profundamente como o Vue renderiza interfaces em qualquer plataforma. Ao final deste capítulo você entenderá como funciona o `@vue/runtime-core`, implementará um Renderer simplificado, compreenderá o Virtual DOM, o processo de Patch e descobrirá por que o Vue consegue renderizar para DOM, NativeScript, Canvas, WebGL, Terminal e qualquer outro ambiente.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Explicar a arquitetura do Renderer do Vue.
* Entender o papel do `@vue/runtime-core`.
* Implementar um `createRenderer()` simplificado.
* Entender como funciona o Patch.
* Implementar montagem e atualização de elementos.
* Compreender por que o Vue é multiplataforma.
* Explicar como o DOM Renderer é apenas uma implementação do Runtime.

---

# Pré-requisitos

* Capítulos 01 ao 25.

---

# Recapitulando

Até agora estudamos:

```text
Template

↓

Compiler

↓

Render Function

↓

Reatividade

↓

Reactive Effect
```

Mas...

Ainda falta uma peça.

Quem transforma a Render Function em elementos visíveis?

---

# O Renderer

Existe um pacote inteiro chamado.

```text
@vue/runtime-core
```

Ele não conhece.

* HTML
* CSS
* DOM
* Browser

Ele conhece apenas.

```text
VNode
```

---

# Outro pacote

Existe também.

```text
@vue/runtime-dom
```

Esse sim.

Conhece.

* document
* Element
* Node
* HTML
* Browser

---

# Arquitetura

```text
Render Function

↓

VNode

↓

Runtime Core

↓

Host Operations

↓

DOM
```

Observe.

O Runtime Core nunca acessa diretamente.

```javascript
document.createElement()
```

---

# Por quê?

Porque o Vue não nasceu apenas para Web.

Ele pode renderizar.

* Browser
* Mobile
* Desktop
* Canvas
* WebGL
* PDF
* Terminal

---

# A grande abstração

Imagine.

```javascript
document.createElement("div")
```

Isso existe apenas no navegador.

---

O Runtime Core nunca faz isso.

Ele chama.

```javascript
hostCreateElement("div")
```

Quem implementa.

É o Runtime DOM.

---

# createRenderer()

A função principal.

É.

```javascript
createRenderer(options)
```

---

# O parâmetro

Recebe.

```javascript
{

createElement,

insert,

remove,

patchProp,

setElementText

}
```

Essas funções são chamadas de.

```text
Host Operations
```

---

# Exemplo

DOM.

```javascript
createElement(tag){

return document.createElement(tag)

}
```

---

Canvas.

```javascript
createElement(tag){

return new Shape(tag)

}
```

---

Terminal.

```javascript
createElement(tag){

return new TerminalNode(tag)

}
```

O Runtime Core continua igual.

---

# Fluxo

```text
VNode

↓

Renderer

↓

Host Operations

↓

Plataforma
```

---

# Implementando

Começamos.

```javascript
function createRenderer(options){

    return{

        render

    }

}
```

---

# Desestruturando

```javascript
const{

createElement,

insert,

remove,

patchProp,

setElementText

}=options
```

Agora.

O Renderer nunca conhece.

O DOM.

---

# render()

Primeira versão.

```javascript
function render(vnode,container){

    patch(

        null,

        vnode,

        container

    )

}
```

---

# O Patch

Toda atualização passa por.

```javascript
patch()
```

Essa é provavelmente.

A função mais importante.

De todo o Runtime.

---

# Assinatura

```javascript
patch(

n1,

n2,

container

)
```

Onde.

```text
n1
```

É.

VNode antiga.

---

E.

```text
n2
```

É.

VNode nova.

---

# Primeira renderização

```text
n1

↓

null
```

---

Logo.

Precisamos montar.

---

# Atualização

Agora.

```text
n1

↓

VNode antiga

↓

VNode nova
```

Precisamos comparar.

---

# Primeira decisão

```javascript
if(n1==null){

mountElement()

}
```

Caso contrário.

```javascript
patchElement()
```

---

# mountElement()

Recebe.

```javascript
mountElement(

vnode,

container

)
```

---

# Primeiro passo

Criar elemento.

```javascript
const el =

createElement(

vnode.type

)
```

Observe.

Não usamos.

```javascript
document.createElement()
```

---

# Depois

Precisamos salvar.

```javascript
vnode.el = el
```

Essa referência será utilizada.

Nas próximas atualizações.

---

# Texto

Se.

```javascript
children

é string
```

Executamos.

```javascript
setElementText(

el,

children

)
```

---

# Filhos

Se.

```javascript
children

é Array
```

Executamos.

```javascript
patch()

Para cada filho.
```

---

# Props

Agora.

```javascript
props
```

Precisam ser aplicadas.

---

# Exemplo

```javascript
{

id:"app",

class:"container"

}
```

Executamos.

```javascript
patchProp(

el,

"id",

null,

"app"

)
```

---

Depois.

```javascript
patchProp(

el,

"class",

null,

"container"

)
```

---

# Inserção

Finalmente.

```javascript
insert(

el,

container

)
```

Elemento montado.

---

# Atualização

Agora.

Temos.

```text
VNode antiga

↓

VNode nova
```

---

Primeiro.

Reutilizamos.

O mesmo elemento.

```javascript
const el =

n2.el = n1.el
```

---

# Depois

Atualizamos.

Props.

---

# patchProps()

Recebe.

```javascript
oldProps

newProps
```

---

Exemplo.

Antes.

```javascript
{

class:"red"

}
```

Depois.

```javascript
{

class:"blue"

}
```

Executamos.

```javascript
patchProp(

el,

"class",

"red",

"blue"

)
```

---

# Removendo Props

Antes.

```javascript
{

title:"teste"

}
```

Depois.

```javascript
{}
```

Chamamos.

```javascript
patchProp(

el,

"title",

"teste",

null

)
```

---

# Atualizando Texto

Antes.

```text
Felipe
```

Depois.

```text
Lucas
```

Executamos.

```javascript
setElementText(

el,

"Lucas"

)
```

---

# Atualizando Filhos

Agora.

O caso complicado.

```text
Array

↓

Array
```

Precisamos comparar.

Cada filho.

---

# Patch Recursivo

```javascript
patch(

oldChild,

newChild,

el

)
```

Chamado.

Para cada posição.

---

# O Virtual DOM

Lembre.

Nunca manipulamos.

HTML diretamente.

Sempre manipulamos.

```text
VNode
```

---

# Fluxo completo

```text
Estado

↓

Render Function

↓

VNode

↓

Patch

↓

Host Operations

↓

DOM
```

---

# Componentes

Até agora.

Renderizamos apenas elementos.

Mas.

```javascript
{

type:MeuComponente

}
```

Também é um VNode.

---

# Como distinguir?

Se.

```javascript
typeof vnode.type

==="string"
```

↓

Elemento.

---

Caso contrário.

↓

Componente.

---

# mountComponent()

Agora.

Criamos.

A instância.

Do componente.

---

# Estrutura

```javascript
instance={

vnode,

props,

setupState,

render

}
```

---

# Executando setup()

Chamamos.

```javascript
setup()
```

Obtendo.

```javascript
setupState
```

---

# Depois

Executamos.

```javascript
render()
```

---

Resultado.

Outra.

```text
VNode
```

---

Observe.

Componentes.

Nunca renderizam HTML.

Eles renderizam.

Mais VNodes.

---

# Árvore

```text
App

↓

Header

↓

Menu

↓

Item
```

Cada componente.

Produz.

Novos VNodes.

---

# Renderização

No final.

Tudo vira.

```text
VNode

↓

Elemento

↓

DOM
```

---

# Atualizações

Quando uma dependência muda.

```text
Reactive Effect

↓

render()

↓

Nova VNode

↓

patch()

↓

DOM
```

---

# O Reactive Effect

Lembra.

Do capítulo anterior.

```javascript
effect(

componentUpdateFn

)
```

---

Esse.

É o elo.

Entre.

Reatividade.

E.

Renderer.

---

# Fluxo completo

```text
ref()

↓

track()

↓

trigger()

↓

ReactiveEffect

↓

render()

↓

VNode

↓

patch()

↓

DOM
```

Agora.

Toda arquitetura do Vue faz sentido.

---

# Host Operations

DOM.

```javascript
createElement()

insert()

remove()

patchProp()
```

---

Canvas.

```javascript
drawRect()

drawCircle()

drawText()
```

---

Terminal.

```javascript
write()

moveCursor()

clear()
```

O Runtime Core permanece igual.

---

# Arquitetura Final

```text
Compiler

↓

Render Function

↓

Runtime Core

↓

Renderer

↓

Host Operations

↓

Plataforma
```

---

# Arquivos reais

Grande parte dessa implementação está em.

```text
packages/runtime-core/src
```

Arquivos importantes.

```text
renderer.ts

component.ts

componentRenderUtils.ts

vnode.ts

scheduler.ts
```

---

# O renderer do DOM

Já a implementação específica do navegador está em.

```text
packages/runtime-dom/src
```

Arquivos.

```text
nodeOps.ts

patchProp.ts

modules/
```

---

# Comparando

Nossa MiniVue.

```text
Render

↓

Elemento
```

Vue.

```text
Render Function

↓

VNode

↓

Patch

↓

Renderer

↓

Host Operations

↓

DOM
```

---

# Curiosidade

O fato de o Vue separar o **Runtime Core** das **Host Operations** é o que permite a existência de projetos como **Vue Native**, renderizadores para **Canvas**, motores experimentais para **WebGL** e até implementações para **Terminal**. Toda a inteligência de atualização de componentes é reutilizada; apenas a camada responsável por criar, atualizar e remover elementos muda conforme a plataforma.

---

# Resumo

Neste capítulo aprendemos que:

* O Runtime Core não conhece o DOM.
* O Renderer é baseado em Host Operations.
* `createRenderer()` permite criar renderizadores para qualquer plataforma.
* Toda atualização passa pela função `patch()`.
* Componentes geram VNodes, não HTML.
* O Reactive Effect conecta a reatividade ao Renderer.
* O Runtime DOM é apenas uma implementação do Runtime Core.

---

# Exercícios

## Exercício 1

Implemente um `createRenderer()` simplificado.

---

## Exercício 2

Implemente `mountElement()`.

---

## Exercício 3

Implemente `patchProps()`.

---

## Exercício 4

Implemente `patch()` distinguindo montagem e atualização.

---

## Exercício 5

Implemente um renderer que escreva elementos em uma estrutura JSON em vez do DOM.

---

# Desafio

Atualize sua **MiniVue** para suportar:

* `createRenderer()`;
* Host Operations injetáveis;
* montagem de elementos;
* atualização de props;
* atualização de texto;
* montagem básica de componentes.

---

# Projeto do capítulo

Ao final deste capítulo sua biblioteca deverá conseguir:

* renderizar VNodes em qualquer plataforma através de Host Operations;
* montar e atualizar elementos;
* montar componentes simples;
* conectar a engine de reatividade ao processo de renderização.

---

# Checklist

* [ ] Sei explicar por que o Runtime Core não conhece o DOM.
* [ ] Entendi o papel de `createRenderer()`.
* [ ] Sei implementar `mountElement()`.
* [ ] Entendi como funciona `patch()`.
* [ ] Sei diferenciar elementos de componentes.
* [ ] Entendi como a reatividade dispara novas renderizações.
* [ ] Minha MiniVue possui um renderer básico independente da plataforma.

---

# Próximo capítulo

## **Capítulo 27 — Virtual DOM e Diffing: O Algoritmo de Patch Completo do Vue 3**

No próximo capítulo entraremos no coração do renderer: o algoritmo de **diffing** do Vue 3. Você aprenderá como o framework compara duas árvores de VNodes, como funciona o patch de filhos, por que as `key`s são fundamentais, como o Vue utiliza a **Longest Increasing Subsequence (LIS)** para minimizar movimentações no DOM e por que seu algoritmo é considerado um dos mais eficientes entre os frameworks modernos. Esse será um dos capítulos mais avançados de toda a série.
