# Capítulo 48 — Vue Core Source Code Tour: Navegando e Entendendo Todo o Repositório Oficial

> **Objetivo:** dominar a organização do código-fonte oficial do Vue 3. Ao final deste capítulo você será capaz de encontrar rapidamente qualquer funcionalidade dentro do repositório, compreender a responsabilidade de cada pacote, entender o fluxo entre Compiler, Runtime e Reactivity e desenvolver a habilidade de estudar novas funcionalidades diretamente na implementação oficial do framework.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Navegar com confiança pelo repositório do Vue.
* Entender a arquitetura de todos os pacotes.
* Localizar rapidamente qualquer funcionalidade.
* Compreender o fluxo entre os módulos.
* Estudar novas features diretamente no código-fonte.
* Utilizar o repositório como principal fonte de aprendizado.

---

# Pré-requisitos

* Capítulos 01 ao 47.

---

# Introdução

Até aqui.

Aprendemos.

Como.

O Vue.

Funciona.

---

Agora.

Vamos.

Aprender.

Onde.

Tudo isso.

Está.

No código.

---

Porque.

Um especialista.

Não depende.

Apenas.

Da documentação.

---

Ele consegue.

Ler.

O código.

Do framework.

---

# Estrutura geral

Na raiz.

Do projeto.

Encontramos.

```text
packages/
```

---

Essa pasta.

Contém.

Praticamente.

Todo.

O Vue.

---

Visualmente.

```text
packages/

├── compiler-core
├── compiler-dom
├── compiler-sfc
├── compiler-ssr
├── reactivity
├── runtime-core
├── runtime-dom
├── runtime-test
├── runtime-core
├── server-renderer
├── shared
├── vue
└── ...
```

---

Cada pasta.

Tem.

Uma responsabilidade.

Muito clara.

---

# shared

Começaremos.

Pela menor.

---

```text
packages/shared
```

---

Ela contém.

Funções.

Utilizadas.

Por.

Todos.

Os outros.

Pacotes.

---

Exemplos.

```javascript
isArray()

isObject()

isString()

hasOwn()

camelize()

capitalize()

hyphenate()
```

---

Sempre.

Que você.

Vir.

Uma função.

Muito simples.

---

Provavelmente.

Ela está.

Aqui.

---

# reactivity

Agora.

O coração.

Do Vue.

---

```text
packages/reactivity
```

---

Aqui.

Encontramos.

Todo.

O sistema.

Reativo.

---

Arquivos.

Importantes.

```text
effect.ts
```

Reactive Effects.

---

```text
reactive.ts
```

Proxy.

---

```text
ref.ts
```

Refs.

---

```text
computed.ts
```

Computed.

---

```text
watch.ts
```

Watch.

---

```text
effectScope.ts
```

Effect Scope.

---

Sempre.

Que surgir.

Uma dúvida.

Sobre.

Reatividade.

---

Comece.

Por aqui.

---

# runtime-core

Depois.

Temos.

O cérebro.

Da renderização.

---

```text
packages/runtime-core
```

---

Esse.

É provavelmente.

O pacote.

Mais importante.

Do Vue.

---

Ele contém.

Componentes.

---

VNode.

---

Renderer.

---

Scheduler.

---

Lifecycle.

---

Slots.

---

Provide.

---

Inject.

---

Suspense.

---

KeepAlive.

---

Teleport.

---

Tudo.

Sem conhecer.

O navegador.

---

Arquivos.

Essenciais.

```text
renderer.ts
```

---

```text
component.ts
```

---

```text
vnode.ts
```

---

```text
scheduler.ts
```

---

```text
apiWatch.ts
```

---

```text
apiLifecycle.ts
```

---

```text
componentRenderUtils.ts
```

---

# runtime-dom

Agora.

Entramos.

Na camada.

Do navegador.

---

```text
packages/runtime-dom
```

---

Aqui.

Encontramos.

As Host Operations.

---

Arquivos.

Mais importantes.

```text
nodeOps.ts
```

---

```text
patchProp.ts
```

---

```text
modules/

├── attrs.ts
├── class.ts
├── style.ts
├── props.ts
├── events.ts
```

---

Tudo.

Relacionado.

Ao DOM.

Está.

Aqui.

---

# compiler-core

Agora.

O compilador.

---

```text
packages/compiler-core
```

---

É aqui.

Que.

Os templates.

Viram.

JavaScript.

---

Arquivos.

Mais importantes.

```text
parse.ts
```

Parser.

---

```text
ast.ts
```

AST.

---

```text
transform.ts
```

Pipeline.

---

```text
codegen.ts
```

Code Generator.

---

Também.

Existe.

A pasta.

```text
transforms/
```

---

Que contém.

```text
transformElement.ts

transformExpression.ts

transformText.ts

vIf.ts

vFor.ts

vSlot.ts
```

---

Cada.

Feature.

Possui.

Seu próprio.

Transform.

---

# compiler-dom

O.

Compiler Core.

É genérico.

---

Mas.

O navegador.

Possui.

Regras.

Próprias.

---

Elas ficam.

Em.

```text
packages/compiler-dom
```

---

Exemplo.

```vue
v-model
```

---

Em.

```html
<input>
```

---

Possui.

Tratamento.

Diferente.

De.

```vue
<MyInput>
```

---

Essas.

Diferenças.

Estão.

Aqui.

---

# compiler-sfc

Agora.

Os arquivos.

`.vue`.

---

São tratados.

Por.

```text
packages/compiler-sfc
```

---

Ele.

Divide.

O arquivo.

Em.

```vue
<template>

<script>

<style>
```

---

Cada parte.

Segue.

Uma pipeline.

Diferente.

---

Também.

Implementa.

```text
<script setup>
```

---

Macros.

Como.

```javascript
defineProps()

defineEmits()

defineExpose()

defineSlots()

defineModel()
```

---

Grande parte.

Das macros.

É resolvida.

Neste pacote.

---

# compiler-ssr

Quando.

Usamos.

SSR.

---

Existe.

Outro.

Compiler.

---

```text
packages/compiler-ssr
```

---

Ele gera.

Código.

Voltado.

Para.

Renderização.

No servidor.

---

# server-renderer

Depois.

Temos.

O Renderer.

Do servidor.

---

```text
packages/server-renderer
```

---

Aqui.

Encontramos.

```text
renderToString()
```

---

Streaming.

---

Hydration.

---

Serialização.

Do estado.

---

# vue

Agora.

Uma curiosidade.

---

Existe.

Também.

O pacote.

```text
packages/vue
```

---

Ele.

É praticamente.

Um ponto.

De entrada.

---

Reexportando.

As APIs.

Dos demais.

Pacotes.

---

Quando.

Você faz.

```javascript
import {

ref,

computed,

watch

}

from "vue"
```

---

Essas APIs.

Vêm.

Deste pacote.

---

# Fluxo completo

Agora.

Vamos juntar.

Tudo.

---

```text
App.vue

↓

compiler-sfc

↓

compiler-core

↓

Render Function

↓

runtime-core

↓

runtime-dom

↓

Browser
```

---

Já.

A reatividade.

Segue.

Outro fluxo.

---

```text
ref()

↓

reactivity

↓

runtime-core

↓

renderer

↓

runtime-dom
```

---

Observe.

Como.

Os módulos.

Conversam.

Entre si.

---

Mas.

Sem.

Acoplamento.

---

# Como estudar o código?

Uma boa.

Estratégia.

É sempre.

Começar.

Pela API.

---

Exemplo.

```javascript
watch()
```

---

Procure.

Onde.

Ela.

É exportada.

---

Depois.

Onde.

É implementada.

---

Depois.

Quais funções.

Ela chama.

---

Depois.

Quais módulos.

Ela utiliza.

---

Repita.

Esse processo.

Para qualquer.

API.

---

# Ordem recomendada

Para estudar.

O repositório.

Siga.

Esta ordem.

```text
shared

↓

reactivity

↓

runtime-core

↓

runtime-dom

↓

compiler-core

↓

compiler-dom

↓

compiler-sfc

↓

server-renderer
```

---

Essa sequência.

Segue.

A dependência.

Natural.

Entre.

Os módulos.

---

# Como contribuir?

Quando.

Você quiser.

Contribuir.

Para o Vue.

---

Primeiro.

Descubra.

Qual pacote.

É responsável.

Pela feature.

---

Depois.

Leia.

Os testes.

---

Só então.

Leia.

A implementação.

---

Essa abordagem.

É muito.

Mais eficiente.

---

# Onde ficam os testes?

Cada pacote.

Possui.

Sua própria.

Pasta.

```text
__tests__
```

---

Os testes.

São.

Uma das.

Melhores.

Formas.

De entender.

O comportamento.

Esperado.

---

# Ferramentas úteis

Durante.

A leitura.

Do código.

Utilize.

* "Go to Definition";
* "Find References";
* busca global;
* Call Hierarchy;
* Type Hierarchy (quando aplicável);
* Debugger.

---

Essas ferramentas.

Economizam.

Horas.

De estudo.

---

# Como implementar?

Na MiniVue.

Crie.

A mesma.

Organização.

---

```text
mini-vue/

reactivity/

runtime-core/

runtime-dom/

compiler/

shared/
```

---

Mesmo.

Que.

Os arquivos.

Sejam.

Pequenos.

---

Você.

Começará.

A pensar.

Como.

A equipe.

Do Vue.

---

# Performance

A separação.

Em pacotes.

Reduz.

Acoplamento.

---

Facilita.

Testes.

---

Permite.

Reutilização.

---

E torna.

A evolução.

Do framework.

Muito.

Mais simples.

---

# Código-fonte

Os diretórios mais importantes para estudar são:

```text
packages/reactivity
```

---

```text
packages/runtime-core
```

---

```text
packages/runtime-dom
```

---

```text
packages/compiler-core
```

---

```text
packages/compiler-sfc
```

---

```text
packages/server-renderer
```

---

Ao dominar esses pacotes, você compreenderá praticamente toda a arquitetura do Vue 3.

---

# Curiosidade

Embora muitos desenvolvedores imaginem que o Vue seja um único bloco de código, ele é composto por diversos pacotes independentes. Essa organização permite que partes do framework, como o sistema de reatividade, sejam utilizadas isoladamente em outros projetos, sem necessidade de carregar todo o Runtime do Vue.

---

# Resumo

Neste capítulo aprendemos que:

* O código do Vue está organizado em pacotes independentes.
* `shared` contém utilitários reutilizáveis.
* `reactivity` implementa todo o sistema reativo.
* `runtime-core` contém a lógica principal do framework.
* `runtime-dom` conecta o Runtime ao navegador.
* `compiler-core`, `compiler-dom` e `compiler-sfc` formam a pipeline de compilação.
* `server-renderer` implementa SSR e Hydration.

---

# Exercícios

## Exercício 1

Navegue pelo pacote `reactivity` e identifique o fluxo completo de execução de um `ref()`.

---

## Exercício 2

Siga a implementação de `watch()` desde sua exportação até a criação do Reactive Effect.

---

## Exercício 3

Localize onde `createVNode()` é implementado e onde ele é utilizado pelo Renderer.

---

## Exercício 4

Abra `compiler-core` e siga o fluxo desde `parse()` até `generate()`.

---

## Exercício 5

Desenhe um diagrama mostrando como os pacotes do Vue se relacionam entre si.

---

# Desafio

Reorganize sua **MiniVue** seguindo uma arquitetura inspirada no repositório oficial:

* `shared`;
* `reactivity`;
* `runtime-core`;
* `runtime-dom`;
* `compiler`;
* `server-renderer`;
* `devtools`.

Explique por que cada módulo existe e quais dependências ele possui.

---

# Projeto do capítulo

Ao final deste capítulo sua MiniVue deverá possuir uma estrutura de projeto semelhante à do Vue 3, facilitando futuras implementações e tornando o código muito mais organizado, escalável e próximo de um framework profissional.

---

# Checklist

* [ ] Sei localizar qualquer funcionalidade no repositório do Vue.
* [ ] Entendi a responsabilidade de cada pacote.
* [ ] Sei seguir o fluxo de uma API desde a exportação até sua implementação.
* [ ] Minha MiniVue possui uma arquitetura inspirada no Vue oficial.
* [ ] Consigo utilizar o código-fonte do Vue como material de estudo.

---

# Próximo capítulo

## **Capítulo 49 — Como Pensar Como um Core Maintainer do Vue: Arquitetura, Decisões de Design e Evolução do Framework**

No próximo capítulo deixaremos de analisar apenas o código e passaremos a estudar **como a equipe do Vue toma decisões arquiteturais**. Você aprenderá os princípios de design utilizados no framework, como novas funcionalidades são propostas por meio de RFCs, como mudanças são avaliadas, quais compromissos entre desempenho, simplicidade e compatibilidade precisam ser considerados e como desenvolver a mentalidade de um mantenedor do Vue. Este será um dos capítulos mais importantes para quem deseja atingir um nível realmente especialista.
