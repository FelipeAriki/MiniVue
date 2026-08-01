# Capítulo 44 — Runtime DOM Internals: Como o Vue Manipula o DOM Real

> **Objetivo:** compreender profundamente o funcionamento do **Runtime DOM** do Vue 3. Ao final deste capítulo você entenderá como o Runtime Core é desacoplado do navegador, como elementos são criados, como atributos, propriedades, estilos e eventos são atualizados, por que o Vue consegue funcionar em diferentes plataformas e como implementar um Runtime DOM simplificado na sua MiniVue.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Entender a arquitetura Runtime Core × Runtime DOM.
* Explicar por que o Vue é multiplataforma.
* Compreender como o Vue cria elementos.
* Entender como atributos e propriedades são atualizados.
* Explicar o sistema de eventos do Runtime.
* Implementar um Runtime DOM simplificado.

---

# Pré-requisitos

* Capítulos 01 ao 43.

---

# Introdução

Até agora.

Conhecemos.

O Compiler.

---

Também.

O Runtime Core.

---

Mas.

Existe.

Outra camada.

Muito importante.

---

O.

```text
Runtime DOM
```

---

Ela.

É responsável.

Por conversar.

Com o navegador.

---

# A arquitetura

O Vue.

É dividido.

Em camadas.

```text
Template

↓

Compiler

↓

Runtime Core

↓

Runtime DOM

↓

Browser
```

---

Observe.

O Runtime Core.

Não conhece.

O navegador.

---

Ele.

Não chama.

```javascript
document.createElement()
```

---

Nem.

```javascript
element.appendChild()
```

---

Quem faz.

Isso.

É o Runtime DOM.

---

# Por que separar?

Imagine.

Que o Vue.

Precisasse.

Rodar.

No.

```text
Node.js
```

---

Ou.

```text
Native
```

---

Ou.

```text
Mini Programs
```

---

Ou.

```text
Canvas
```

---

Se.

Todo.

O código.

Dependesse.

Do DOM.

---

Nada disso.

Seria.

Possível.

---

Por isso.

O Runtime Core.

É totalmente.

Independente.

Da plataforma.

---

# Renderer

Lembra.

Do Renderer?

---

Ele.

Recebe.

Um conjunto.

De operações.

---

Chamadas.

De.

```text
Host Operations
```

---

Exemplo.

```javascript
hostCreateElement()
```

---

```javascript
hostInsert()
```

---

```javascript
hostRemove()
```

---

```javascript
hostSetText()
```

---

O Runtime Core.

Nunca.

Chama.

O DOM.

Diretamente.

---

# No navegador

As operações.

São implementadas.

Assim.

```javascript
hostCreateElement(tag){

return document.createElement(tag)

}
```

---

Já.

```javascript
hostInsert(

el,

parent

){

parent.appendChild(el)

}
```

---

Observe.

O Renderer.

Não sabe.

Como.

Isso funciona.

---

Ele apenas.

Chama.

As operações.

---

# Fluxo

```text
Renderer

↓

hostCreateElement()

↓

Runtime DOM

↓

Browser
```

---

# Criando elementos

Imagine.

Este template.

```vue
<div>

<h1>Hello</h1>

</div>
```

---

O Compiler.

Produz.

VNodes.

---

Depois.

O Renderer.

Executa.

```javascript
hostCreateElement(

"div"

)
```

---

Depois.

```javascript
hostCreateElement(

"h1"

)
```

---

Depois.

```javascript
hostSetText()

```

---

Depois.

```javascript
hostInsert()

```

---

Resultado.

O DOM.

É criado.

---

# Atualizando atributos

Imagine.

```vue
<img

:src="foto"

>
```

---

Quando.

A imagem.

Muda.

---

O Runtime.

Executa.

Algo parecido.

Com.

```javascript
patchProp(

img,

"src",

old,

novo

)
```

---

Observe.

Não existe.

Um método.

Específico.

Para.

Cada atributo.

---

Existe.

Um único.

```javascript
patchProp()
```

---

Ele decide.

O que fazer.

---

# Atributo ou propriedade?

Existe.

Uma diferença.

Muito importante.

---

HTML possui.

Atributos.

---

DOM possui.

Propriedades.

---

Exemplo.

```html
<input value="10">
```

---

Atributo.

```text
value="10"
```

---

Depois.

Do elemento.

Ser criado.

---

O navegador.

Mantém.

Uma propriedade.

```javascript
input.value
```

---

Nem sempre.

Alterar.

O atributo.

Altera.

A propriedade.

---

Por isso.

O Vue.

Precisa.

Saber.

Qual atualizar.

---

# Exemplo

```vue
<input

v-model="nome"

>
```

---

O Vue.

Atualiza.

```javascript
input.value
```

---

Não.

```html
value=""
```

---

Essa diferença.

É essencial.

---

# Classes

Imagine.

```vue
<div

:class="classe"

>
```

---

O Runtime.

Executa.

```javascript
el.className=

classe
```

---

Ou.

Quando necessário.

```javascript
classList
```

---

Dependendo.

Da situação.

---

# Estilos

Outro exemplo.

```vue
<div

:style="estilo"

>
```

---

Internamente.

```javascript
el.style.color="red"
```

---

Ou.

```javascript
el.style.width="100px"
```

---

Quando.

Um estilo.

Desaparece.

---

O Vue.

Também.

O remove.

---

# Eventos

Imagine.

```vue
<button

@click="salvar"

>
```

---

Durante.

O Patch.

---

O Runtime.

Executa.

```javascript
addEventListener(

"click",

handler

)
```

---

Quando.

O evento.

É removido.

---

Executa.

```javascript
removeEventListener()
```

---

# Invokers

Existe.

Uma otimização.

Pouco conhecida.

---

Imagine.

Que.

O handler.

Mude.

---

Sem otimização.

O Vue.

Precisaria.

Remover.

O evento.

---

Depois.

Adicionar.

Outro.

---

Em vez disso.

Ele cria.

Um objeto.

Chamado.

```text
Invoker
```

---

Visualmente.

```text
Click

↓

Invoker

↓

Função Atual
```

---

Quando.

O handler.

Muda.

---

O Vue.

Troca.

A função.

---

Sem remover.

O listener.

Do DOM.

---

Isso.

É muito.

Mais rápido.

---

# Eventos múltiplos

Também.

É possível.

```vue
@click.once

@click.capture

@click.passive
```

---

O Compiler.

Transforma.

Esses modificadores.

---

Em opções.

Do.

```javascript
addEventListener()
```

---

# Text Nodes

Texto.

Também.

É um nó.

Do DOM.

---

Para.

Criá-lo.

O Runtime.

Executa.

```javascript
document.createTextNode()
```

---

Depois.

Atualiza.

Com.

```javascript
node.nodeValue
```

---

# Comentários

Comentários.

Também.

São nós.

---

Eles.

São utilizados.

Principalmente.

Por.

Fragments.

Suspense.

SSR.

---

O Runtime.

Cria.

```javascript
document.createComment()
```

---

# Fragments

Lembra.

Do Fragment?

---

Ele.

Não gera.

Elemento.

Pai.

---

Então.

O Runtime.

Precisa.

Controlar.

Onde.

O Fragment.

Começa.

E termina.

---

Para isso.

São utilizados.

Comentários.

Como marcadores.

---

Visualmente.

```html
<!--fragment-->

<h1></h1>

<p></p>

<!--fragment-->
```

---

# SVG

Outro detalhe.

Importante.

---

Criar.

```html
<div>
```

É diferente.

De criar.

```html
<svg>
```

---

O Runtime.

Detecta.

Esse caso.

---

E utiliza.

```javascript
document.createElementNS()
```

---

Em vez.

De.

```javascript
createElement()
```

---

# Namespaces

O mesmo.

Vale.

Para.

```text
MathML
```

---

Cada.

Namespace.

Possui.

Regras.

Próprias.

---

# Como implementar?

Na MiniVue.

Podemos.

Criar.

Uma interface.

---

```javascript
const host={

createElement,

insert,

remove,

patchProp,

setText

}
```

---

Depois.

O Renderer.

Recebe.

Esse objeto.

---

```javascript
createRenderer(

host

)
```

---

Assim.

O Renderer.

Nunca.

Depende.

Do navegador.

---

Depois.

Implementamos.

```javascript
createElement(tag){

return

document.createElement(tag)

}
```

---

Já temos.

Nosso.

Runtime DOM.

---

# Arquitetura

Visualmente.

```text
Renderer

↓

Host API

↓

DOM API

↓

Browser
```

---

Se.

Quisermos.

Criar.

Outro.

Renderer.

---

Basta.

Trocar.

A Host API.

---

Todo.

O restante.

Continua.

Funcionando.

---

# Performance

O Runtime DOM.

Também.

Possui.

Diversas.

Otimizações.

---

Entre elas.

* cache de Invokers;
* atualização seletiva de props;
* atualização incremental de estilos;
* reutilização de listeners;
* integração com Patch Flags.

---

Essas.

Otimizações.

Reduzem.

Muito.

O custo.

De atualização.

Do DOM.

---

# Código-fonte

Grande parte do Runtime DOM pode ser estudada em:

```text
packages/runtime-dom
```

---

Especialmente.

```text
nodeOps.ts
```

Responsável pelas operações básicas sobre o DOM.

---

```text
patchProp.ts
```

Responsável por decidir como atualizar atributos, propriedades, estilos e eventos.

---

Também vale estudar:

```text
modules/class.ts
```

---

```text
modules/style.ts
```

---

```text
modules/events.ts
```

---

```text
modules/attrs.ts
```

---

```text
modules/props.ts
```

---

Cada módulo trata uma categoria específica de atualização, mantendo o Runtime DOM altamente modular.

---

# Curiosidade

O Runtime Core **não importa nenhuma API do navegador**. Toda comunicação com a plataforma acontece através das Host Operations fornecidas ao `createRenderer()`. Essa decisão arquitetural permite reutilizar praticamente todo o Runtime do Vue em ambientes completamente diferentes, como renderizadores para aplicações nativas, Canvas e até terminais.

---

# Resumo

Neste capítulo aprendemos que:

* O Runtime Core é independente da plataforma.
* O Runtime DOM implementa as operações específicas do navegador.
* O Renderer trabalha através de Host Operations.
* `patchProp()` centraliza a atualização de atributos, propriedades, estilos e eventos.
* O Vue utiliza Invokers para otimizar listeners.
* Fragments, SVG e namespaces possuem tratamentos específicos.

---

# Exercícios

## Exercício 1

Implemente um objeto `host` contendo `createElement`, `insert`, `remove` e `setText`.

---

## Exercício 2

Implemente uma versão simplificada de `patchProp()` que suporte `class`, `style` e atributos comuns.

---

## Exercício 3

Adicione suporte a eventos utilizando `addEventListener()` e `removeEventListener()`.

---

## Exercício 4

Implemente um cache simples de Invokers para evitar remover e adicionar listeners quando apenas a função do evento mudar.

---

## Exercício 5

Leia `patchProp.ts`, `events.ts` e `nodeOps.ts` e identifique como o Runtime DOM decide entre atualizar uma propriedade do DOM ou um atributo HTML.

---

# Desafio

Atualize sua **MiniVue** para suportar:

* Runtime DOM desacoplado;
* Host Operations;
* `patchProp()`;
* atualização de classes e estilos;
* sistema de eventos otimizado;
* suporte básico a Fragments e SVG.

---

# Projeto do capítulo

Ao final deste capítulo sua MiniVue deverá possuir um Runtime DOM modular, desacoplado do Renderer e capaz de manipular o DOM de forma semelhante ao Vue 3, servindo como base para a criação de renderizadores para outras plataformas.

---

# Checklist

* [ ] Entendi a separação entre Runtime Core e Runtime DOM.
* [ ] Sei explicar o papel das Host Operations.
* [ ] Entendi como o Vue atualiza atributos e propriedades.
* [ ] Sei como funciona o sistema de eventos do Runtime.
* [ ] Minha MiniVue possui um Runtime DOM desacoplado.

---

# Próximo capítulo

## **Capítulo 45 — Vue Internals: Scheduler Avançado, Effect Scope, Flush Timing e Pipeline Completa de Atualizações**

No próximo capítulo estudaremos o ciclo completo de atualização do Vue 3. Você aprenderá como o Scheduler organiza os jobs, como funcionam as filas de atualização, `flush: "pre"`, `"post"` e `"sync"`, `EffectScope`, limpeza automática de efeitos, prevenção de loops infinitos e toda a pipeline de atualização desde a alteração de um `ref()` até a modificação final do DOM. Este é um dos capítulos que consolidam a compreensão completa da arquitetura interna do Vue.
