# Capítulo 54 — Projeto Final V: Implementando Suspense, Teleport, KeepAlive e Transitions na MiniVue

> **Objetivo:** implementar os componentes especiais do Vue 3 na MiniVue. Ao final deste capítulo você compreenderá como **Suspense**, **Teleport**, **KeepAlive** e **Transition** funcionam internamente, como interagem com o Renderer e o Scheduler e como essas funcionalidades são construídas na arquitetura do Vue.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Implementar o Teleport.
* Entender o funcionamento do KeepAlive.
* Implementar um Suspense simplificado.
* Criar um sistema de Transition.
* Integrar esses recursos ao Renderer.
* Aproximar ainda mais sua MiniVue do Vue 3.

---

# Pré-requisitos

* Capítulos 01 ao 53.

---

# Introdução

Até aqui.

Nossa MiniVue.

Possui.

---

Compiler.

---

Runtime.

---

Renderer.

---

Reactivity.

---

Scheduler.

---

Virtual DOM.

---

Mas.

Ainda faltam.

Alguns.

Dos recursos.

Mais sofisticados.

Do Vue.

---

Eles.

Não são.

Componentes.

Comuns.

---

São.

Componentes.

Especiais.

---

Eles.

Alteram.

O comportamento.

Do Renderer.

---

# Componentes especiais

O Vue.

Possui.

Alguns.

Componentes.

Que.

Não renderizam.

HTML.

Diretamente.

---

Em vez disso.

Eles.

Controlam.

O Runtime.

---

Os principais.

São.

```text
Teleport

KeepAlive

Suspense

Transition
```

---

Todos.

São tratados.

Pelo Renderer.

De maneira.

Especial.

---

# Teleport

Imagine.

O componente.

```vue
<Teleport to="body">

<Modal/>

</Teleport>
```

---

Visualmente.

Parece.

Que.

O Modal.

Está.

Dentro.

Do componente.

---

Mas.

No DOM.

Ele será.

Movido.

---

Resultado.

```text
App

↓

Teleport

↓

Modal
```

---

DOM.

```text
<body>

<div id="app"/>

<div class="modal"/>

</body>
```

---

Observe.

Que.

A árvore.

De componentes.

Não muda.

---

Somente.

A posição.

No DOM.

---

# Como funciona?

O Renderer.

Detecta.

Que.

O VNode.

É.

```text
Teleport
```

---

Ao invés.

De inserir.

Os filhos.

No pai.

---

Ele procura.

O destino.

---

```javascript
document.querySelector(
target
)
```

---

Depois.

Executa.

```javascript
mount(

children,

target
)
```

---

O componente.

Continua.

No mesmo.

Lugar.

Da árvore.

---

Mas.

O DOM.

Foi.

Redirecionado.

---

# KeepAlive

Agora.

Imagine.

```vue
<KeepAlive>

<Page/>

</KeepAlive>
```

---

Sem.

KeepAlive.

---

Quando.

O componente.

Sai.

Da tela.

---

Ele.

É destruído.

---

Depois.

É recriado.

---

Com.

KeepAlive.

---

Isso.

Não acontece.

---

A instância.

É preservada.

---

# Fluxo

Sem.

KeepAlive.

```text
Mount

↓

Unmount

↓

Destroy
```

---

Com.

KeepAlive.

```text
Mount

↓

Deactivate

↓

Activate
```

---

Observe.

Que.

O componente.

Nunca.

É destruído.

---

# Cache

Internamente.

O KeepAlive.

Mantém.

Um cache.

---

```javascript
const cache=

new Map()
```

---

Quando.

O componente.

É removido.

---

Ele.

É armazenado.

---

Depois.

Se voltar.

---

É reutilizado.

---

Sem.

Criar.

Nova instância.

---

# Estrutura

Visualmente.

```text
KeepAlive

↓

Cache

↓

Component Instance
```

---

O Renderer.

Consulta.

Primeiro.

O cache.

---

Se existir.

---

Reutiliza.

---

Caso contrário.

---

Cria.

Uma nova.

Instância.

---

# Suspense

Agora.

Imagine.

```vue
<Suspense>

<AsyncPage/>

<Fallback/>

</Suspense>
```

---

O componente.

Ainda.

Não terminou.

De carregar.

---

O Renderer.

Não pode.

Exibi-lo.

---

Então.

Renderiza.

O fallback.

---

Fluxo.

```text
Async Component

↓

Pending

↓

Fallback
```

---

Quando.

A Promise.

Resolve.

---

Executamos.

```text
Resolve

↓

Renderer

↓

Patch()
```

---

Agora.

O fallback.

É removido.

---

O componente.

É exibido.

---

# Como implementar?

Na MiniVue.

Podemos.

Criar.

Um contador.

De dependências.

---

```javascript
pending++
```

---

Quando.

Uma Promise.

Resolve.

---

```javascript
pending--
```

---

Quando.

O contador.

Chega.

A zero.

---

Renderizamos.

O conteúdo.

Principal.

---

# Transition

Agora.

Outro.

Componente.

Especial.

---

```vue
<Transition>

<Card/>

</Transition>
```

---

O objetivo.

Não é.

Renderizar.

HTML.

---

Mas.

Controlar.

Animações.

---

# Entrada

Durante.

O Mount.

---

Executamos.

```text
beforeEnter

↓

enter

↓

afterEnter
```

---

# Saída

Durante.

O Unmount.

---

Executamos.

```text
beforeLeave

↓

leave

↓

afterLeave
```

---

Somente.

Depois.

Removemos.

O elemento.

---

# Classes CSS

Normalmente.

O Vue.

Adiciona.

Classes.

Como.

```text
fade-enter-from

fade-enter-active

fade-enter-to
```

---

Depois.

Remove.

Automaticamente.

---

O mesmo.

Acontece.

Na saída.

---

```text
fade-leave-from

fade-leave-active

fade-leave-to
```

---

# Integração

Observe.

Que.

Transition.

Não modifica.

O Compiler.

---

Ela modifica.

O Renderer.

---

Fluxo.

```text
Renderer

↓

Hooks

↓

CSS

↓

DOM
```

---

# Scheduler

Mesmo.

Durante.

Uma animação.

---

As atualizações.

Continuam.

Passando.

Pelo Scheduler.

---

Assim.

Mantemos.

A consistência.

Da aplicação.

---

# DevTools

Todos.

Esses.

Componentes.

Também.

São registrados.

---

Permitindo.

Visualizar.

---

Teleport.

---

Suspense.

---

KeepAlive.

---

Transitions.

---

Na árvore.

De componentes.

---

# Organização

Na MiniVue.

Crie.

Os arquivos.

```text
runtime-core/

components/

Teleport.js

KeepAlive.js

Suspense.js

Transition.js
```

---

Cada.

Arquivo.

Possui.

Uma única.

Responsabilidade.

---

Depois.

Atualize.

O Renderer.

---

```text
VNode

↓

Tipo especial?

↓

Sim

↓

Renderer Especial

↓

Não

↓

Renderer Normal
```

---

Essa.

É exatamente.

A estratégia.

Utilizada.

Pelo Vue.

---

# Performance

Esses.

Componentes.

Não.

Afetam.

Os demais.

---

Porque.

São tratados.

Somente.

Quando.

Encontrados.

Pelo Renderer.

---

Isso.

Mantém.

O custo.

Muito baixo.

---

# Código-fonte

Os principais arquivos para estudar são:

```text
packages/runtime-core/src/components/Teleport.ts
```

---

```text
packages/runtime-core/src/components/KeepAlive.ts
```

---

```text
packages/runtime-core/src/components/Suspense.ts
```

---

```text
packages/runtime-dom/src/components/Transition.ts
```

---

```text
packages/runtime-dom/src/components/BaseTransition.ts
```

---

Esses módulos implementam praticamente toda a lógica dos componentes especiais do Vue 3.

---

# Curiosidade

Embora `Teleport`, `KeepAlive`, `Suspense` e `Transition` sejam utilizados como componentes na API pública do Vue, internamente eles funcionam muito mais como **extensões do Renderer** do que como componentes convencionais. Cada um possui um fluxo de renderização próprio e altera o comportamento padrão da montagem, atualização ou remoção de elementos.

---

# Resumo

Neste capítulo aprendemos que:

* `Teleport` altera o destino da renderização no DOM sem modificar a árvore de componentes.
* `KeepAlive` reutiliza instâncias através de um cache.
* `Suspense` coordena componentes assíncronos e fallbacks.
* `Transition` controla animações de entrada e saída.
* Todos esses recursos são tratados diretamente pelo Renderer.
* A MiniVue passa a suportar praticamente todos os componentes especiais presentes no Vue 3.

---

# Exercícios

## Exercício 1

Implemente um `Teleport` simplificado que renderize elementos em outro nó do DOM.

---

## Exercício 2

Implemente um cache básico para `KeepAlive` utilizando `Map`.

---

## Exercício 3

Crie um `Suspense` simplificado que exiba um fallback enquanto uma Promise não for resolvida.

---

## Exercício 4

Implemente os hooks `beforeEnter`, `enter` e `afterEnter` para uma `Transition`.

---

## Exercício 5

Atualize o Renderer para detectar automaticamente VNodes especiais e delegar a renderização para a implementação correspondente.

---

# Desafio

Atualize sua **MiniVue** para suportar:

* Teleport;
* KeepAlive;
* Suspense;
* Transition;
* cache de componentes;
* componentes assíncronos;
* animações controladas pelo Runtime.

---

# Projeto do capítulo

Ao final deste capítulo sua MiniVue deverá oferecer suporte aos principais componentes especiais do Vue 3, permitindo cache de componentes, renderização em destinos alternativos, carregamento assíncrono e transições de entrada e saída integradas ao Renderer.

---

# Checklist

* [ ] Entendi como o Teleport altera apenas o DOM.
* [ ] Sei implementar um cache para KeepAlive.
* [ ] Compreendi o funcionamento interno do Suspense.
* [ ] Implementei uma Transition básica.
* [ ] Minha MiniVue suporta os principais componentes especiais do Vue.

---

# Próximo capítulo

## **Capítulo 55 — Projeto Final VI: Construindo um Ecossistema Completo para a MiniVue (Router, Store, CLI, DevTools e Plugins)**

No próximo capítulo expandiremos a MiniVue para além do framework principal. Construiremos um ecossistema completo inspirado no Vue, implementando um **Mini Router**, uma **Mini Store** inspirada no Pinia, um sistema de **Plugins**, uma **CLI** para criação de projetos e integrações com o **Mini DevTools**, aproximando a experiência de desenvolvimento da oferecida pelo ecossistema oficial do Vue.
