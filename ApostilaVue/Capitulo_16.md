# Capítulo 16 — Virtual DOM: Como o Vue Cria, Compara e Atualiza a Árvore de VNodes

> **Objetivo:** compreender profundamente como o Vue renderiza componentes utilizando o Virtual DOM. Ao final deste capítulo você entenderá como funcionam os **VNodes**, a função `h()`, o algoritmo `patch()`, as **Shape Flags**, **Patch Flags**, **Block Tree** e as otimizações do compilador que fazem o Vue 3 ser extremamente rápido.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Explicar o que é um Virtual DOM.
* Implementar um VNode simplificado.
* Entender a função `h()`.
* Explicar como funciona o algoritmo `patch()`.
* Entender Shape Flags.
* Entender Patch Flags.
* Compreender o Block Tree.
* Explicar por que o compilador do Vue é tão importante.

---

# Pré-requisitos

* Capítulos 01 ao 15.

---

# O problema

Imagine um componente.

```vue
<template>

<h1>{{ nome }}</h1>

</template>
```

Depois.

```javascript
nome.value = "Felipe"
```

O Vue precisa responder uma pergunta.

> O que exatamente mudou na tela?

---

# A solução mais simples

Poderíamos fazer.

```text
Apagar todo o HTML

↓

Criar tudo novamente
```

Funciona.

Mas é extremamente lento.

---

# Outra solução

Comparar o HTML antigo com o novo.

Mas...

Como comparar HTML?

Muito complicado.

---

# Surge o Virtual DOM

Em vez de comparar HTML.

O Vue compara objetos JavaScript.

---

# Visualizando

HTML.

```html
<div>

<h1>Vue</h1>

<p>Hello</p>

</div>
```

Virtual DOM.

```javascript
{

type:"div",

children:[

{

type:"h1",

children:"Vue"

},

{

type:"p",

children:"Hello"

}

]

}
```

Muito mais fácil de comparar.

---

# O que é um VNode?

VNode significa.

```text
Virtual Node
```

Cada elemento HTML.

↓

Um objeto JavaScript.

---

# Exemplo

```html
<button>

Salvar

</button>
```

Transforma-se em.

```javascript
{

type:"button",

props:null,

children:"Salvar"

}
```

---

# A função h()

O Vue possui uma função.

```javascript
h()
```

Ela cria VNodes.

---

# Exemplo

```javascript
h(

"div",

null,

"Olá"

)
```

Resultado.

```javascript
{

type:"div",

props:null,

children:"Olá"

}
```

---

# Implementação simplificada

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

# Template

Quando escrevemos.

```vue
<div>

Olá

</div>
```

O compilador gera.

```javascript
render(){

    return h(

        "div",

        null,

        "Olá"

    )

}
```

---

# Componentes também são VNodes

Imagine.

```vue
<MeuBotao />
```

Compilado.

```javascript
h(

MeuBotao

)
```

Observe.

O type não é mais uma string.

É um componente.

---

# Dois tipos de VNode

Elemento.

```javascript
{

type:"div"

}
```

Componente.

```javascript
{

type:MeuComponente

}
```

---

# Estrutura real

Um VNode do Vue possui dezenas de propriedades.

Simplificando.

```javascript
{

type,

props,

key,

ref,

children,

shapeFlag,

patchFlag,

el

}
```

---

# el

Após montar.

O VNode guarda referência para o elemento real.

```text
VNode

↓

el

↓

HTMLElement
```

---

# key

Muito utilizada em listas.

```vue
<li

v-for="item in lista"

:key="item.id"

/>
```

Ela será estudada em detalhes posteriormente.

---

# Primeira renderização

Fluxo.

```text
Render

↓

VNode

↓

DOM
```

---

# Segunda renderização

Agora.

```text
Render

↓

Novo VNode
```

Temos.

```text
VNode antigo

↓

VNode novo
```

Agora entra o algoritmo.

```text
patch()
```

---

# O Patch

Sua missão.

```text
Comparar duas árvores
```

E atualizar somente o necessário.

---

# Fluxo

```text
VNode Antigo

↓

Patch

↓

VNode Novo

↓

DOM Atualizado
```

---

# Primeira decisão

Os tipos são iguais?

```javascript
old.type

===

new.type
```

---

# Se forem diferentes

Exemplo.

Antes.

```html
<div>
```

Depois.

```html
<section>
```

Não existe reaproveitamento.

O Vue remove.

↓

Cria novamente.

---

# Se forem iguais

Agora compara.

* Props
* Children
* Eventos
* Texto

---

# Exemplo

Antes.

```html
<button>

Salvar

</button>
```

Depois.

```html
<button>

Enviar

</button>
```

O botão continua.

Apenas o texto muda.

---

# Resultado

O Vue faz.

```javascript
button.textContent =

"Enviar"
```

Sem recriar o botão.

---

# Shape Flags

O Vue utiliza Bits.

Por exemplo.

```text
Elemento

↓

1
```

Componente.

↓

2

Texto.

↓

4

Array.

↓

8

Slots.

↓

16

---

# Por quê?

Imagine.

```javascript
if(

vnode.children

instanceof Array

)
```

Isso custa mais.

Com Bits.

```javascript
if(

shapeFlag & ARRAY_CHILDREN

)
```

Muito mais rápido.

---

# Exemplo

```javascript
const ShapeFlags={

ELEMENT:1,

STATEFUL_COMPONENT:2,

TEXT_CHILDREN:4,

ARRAY_CHILDREN:8

}
```

---

# Um VNode

```javascript
{

shapeFlag:9

}
```

9 significa.

```text
1

+

8
```

Elemento.

↓

Array de filhos.

---

# Patch Flags

Outro conceito importante.

Eles são gerados pelo compilador.

---

# Exemplo

Imagine.

```vue
<div

class="ativo"

>

{{ nome }}

</div>
```

O compilador sabe.

A classe nunca muda.

Somente.

```text
nome
```

---

# Então ele gera

Algo parecido.

```javascript
patchFlag:

TEXT
```

Assim.

Durante o Patch.

O Vue nem compara atributos.

Atualiza apenas o texto.

---

# Sem Patch Flags

Precisaria comparar.

* class
* style
* props
* eventos
* texto

---

# Com Patch Flags

Somente.

```text
Texto
```

Muito mais rápido.

---

# Block Tree

Outra otimização.

Imagine.

```vue
<div>

<h1>Logo</h1>

<p>{{ contador }}</p>

<footer>Fim</footer>

</div>
```

Logo.

↓

Nunca muda.

Footer.

↓

Nunca muda.

Somente.

```text
contador
```

---

# O compilador cria

```text
Bloco

↓

Nós dinâmicos
```

Na atualização.

O Vue percorre apenas.

```text
Nós dinâmicos
```

Ignorando o restante.

---

# Visualizando

Sem Block Tree.

```text
div

↓

h1

↓

p

↓

footer
```

Todos comparados.

---

Com Block Tree.

```text
Bloco

↓

p
```

Somente.

O elemento dinâmico.

---

# Patch Element

Quando dois elementos possuem o mesmo tipo.

O Vue faz.

```javascript
patchElement(

oldVNode,

newVNode

)
```

Essa função.

* compara props;
* compara filhos;
* aplica atualizações.

---

# Patch Children

Agora outro caso.

Antes.

```html
<ul>

<li>A</li>

</ul>
```

Depois.

```html
<ul>

<li>A</li>

<li>B</li>

</ul>
```

O algoritmo entra em.

```javascript
patchChildren()
```

Esse algoritmo é um dos mais sofisticados do Vue.

Terá um capítulo exclusivo.

---

# mountElement()

Na primeira renderização.

O Vue chama.

```javascript
mountElement()
```

Ela faz.

```javascript
createElement()

setProps()

mountChildren()

append()
```

---

# patchElement()

Na atualização.

Nunca cria novamente.

Faz apenas.

```javascript
patchProps()

patchChildren()
```

---

# mountComponent()

Se o VNode representa um componente.

O Vue chama.

```javascript
mountComponent()
```

Criando uma nova instância.

---

# patchComponent()

Se o componente já existe.

Executa.

```javascript
updateComponent()
```

Sem desmontá-lo.

---

# O Renderer

Existe uma função enorme.

```text
createRenderer()
```

Ela recebe.

```javascript
{

createElement,

insert,

remove,

setText,

setElementText

}
```

Assim.

O Runtime Core não depende do navegador.

---

# Isso permite

Criar renderizadores para.

* Browser.
* NativeScript.
* Mini Programs.
* Electron.
* Canvas.
* Custom Renderers.

Tudo reutilizando o mesmo Runtime.

---

# Fluxo completo

```text
Template

↓

Compilador

↓

Render Function

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

# Performance

As maiores otimizações do Vue 3 são:

* Shape Flags.
* Patch Flags.
* Block Tree.
* Static Hoisting.
* Scheduler.
* Reactive System.

Todas trabalham juntas.

---

# Comparando

Nossa implementação.

```text
Render

↓

VNode

↓

Patch
```

Vue.

```text
Render

↓

VNode

↓

Shape Flags

↓

Patch Flags

↓

Block Tree

↓

Patch

↓

Renderer

↓

DOM
```

---

# Curiosidade

Muitas pessoas dizem.

> "O Vue usa Virtual DOM."

Mais correto seria.

> "O Vue utiliza um Virtual DOM altamente otimizado pelo compilador."

Grande parte da velocidade do Vue não vem apenas do Virtual DOM, mas das informações extras que o compilador adiciona para reduzir drasticamente o trabalho durante as atualizações.

---

# Resumo

Neste capítulo aprendemos que:

* Todo elemento HTML torna-se um VNode.
* `h()` cria VNodes.
* O `patch()` compara duas árvores de VNodes.
* `Shape Flags` identificam rapidamente o tipo de um nó.
* `Patch Flags` informam exatamente o que pode mudar.
* O `Block Tree` evita percorrer partes estáticas da árvore.
* O Renderer converte VNodes em elementos reais do DOM.

---

# Exercícios

## Exercício 1

Implemente uma função `h()` que crie VNodes simples.

---

## Exercício 2

Implemente um `mountElement()` simplificado.

---

## Exercício 3

Implemente um `patchElement()` que atualize apenas o texto.

---

## Exercício 4

Explique a diferença entre Shape Flags e Patch Flags.

---

# Desafio

Atualize sua **MiniVue** para suportar:

* `h()`;
* criação de VNodes;
* `mountElement()`;
* `patchElement()`;
* `mountComponent()`;
* `patchComponent()`;
* `shapeFlag`;
* cache do elemento real (`el`).

---

# Projeto do capítulo

Ao final deste capítulo sua biblioteca deverá conseguir:

* criar VNodes;
* montar elementos no DOM;
* atualizar texto sem recriar elementos;
* distinguir componentes de elementos;
* armazenar referências ao DOM real;
* executar um `patch()` simplificado.

---

# Checklist

* [ ] Sei explicar o que é um VNode.
* [ ] Entendi como funciona a função `h()`.
* [ ] Sei explicar o algoritmo `patch()`.
* [ ] Entendi a função das Shape Flags.
* [ ] Entendi a função das Patch Flags.
* [ ] Sei explicar o conceito de Block Tree.
* [ ] Minha MiniVue já consegue renderizar e atualizar elementos simples.

---

# Próximo capítulo

## **Capítulo 17 — O Algoritmo de Diff: Como o Vue Atualiza Listas com Máxima Eficiência**

Agora estudaremos o algoritmo mais famoso do Vue: o **Diff de listas**. Você aprenderá como funcionam `patchKeyedChildren()`, `patchUnkeyedChildren()`, a importância da propriedade `key`, o algoritmo de **Longest Increasing Subsequence (LIS)**, movimentação mínima de elementos, inserções, remoções e por que o Vue consegue atualizar listas enormes realizando o menor número possível de operações no DOM. Este é um dos capítulos mais avançados e importantes de toda a arquitetura do Vue 3.
