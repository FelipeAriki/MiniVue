# Capítulo 51 — Projeto Final II: Implementando um Sistema Completo de Componentes

> **Objetivo:** construir o sistema completo de componentes da MiniVue, reproduzindo a arquitetura utilizada pelo Vue 3. Ao final deste capítulo você será capaz de implementar instâncias de componentes, `setup()`, `render()`, `props`, `slots`, `emit`, ciclo de vida e o processo completo de montagem e atualização.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Entender a arquitetura de uma Component Instance.
* Implementar `setup()`.
* Implementar `props` e `emit`.
* Compreender o fluxo completo de montagem.
* Criar uma árvore de componentes.
* Aproximar sua MiniVue da arquitetura real do Vue 3.

---

# Pré-requisitos

* Capítulos 01 ao 50.

---

# Introdução

Até agora.

Nossa MiniVue.

Consegue.

Inicializar.

Uma aplicação.

---

Mas.

Ainda.

Não existe.

O conceito.

Mais importante.

Do Vue.

---

O componente.

---

Todo.

O Runtime.

Foi criado.

Para.

Gerenciar.

Componentes.

---

Neste capítulo.

Vamos.

Construir.

Essa estrutura.

---

# O que é um componente?

Muitos.

Pensam.

Que um componente.

É apenas.

Um objeto.

---

Na verdade.

É muito mais.

---

Existe.

Uma definição.

---

```javascript
const App = {

setup(){},

render(){}

}
```

---

Mas.

Quando.

A aplicação.

É executada.

---

O Runtime.

Cria.

Uma instância.

---

Essa instância.

É quem.

Mantém.

Todo.

O estado.

---

# Definição × Instância

Visualmente.

```text
Component Definition

↓

Component Instance
```

---

A definição.

É reutilizada.

---

A instância.

É única.

Para.

Cada componente.

---

# Exemplo

Imagine.

```vue
<Card/>

<Card/>

<Card/>
```

---

Existe.

Uma única.

Definição.

---

Mas.

Três.

Instâncias.

---

```text
Card

↓

Instance 1

Instance 2

Instance 3
```

---

Cada.

Uma possui.

Seu próprio.

Estado.

---

# Criando uma Component Instance

Na MiniVue.

Podemos.

Começar.

Assim.

```javascript
function createComponentInstance(vnode){

return{

vnode,

type:vnode.type,

parent:null,

appContext:null,

props:{},

attrs:{},

slots:{},

setupState:{},

ctx:{},

proxy:null,

emit:null,

isMounted:false,

subTree:null,

update:null

}

}
```

---

Observe.

Que.

A instância.

Não guarda.

Apenas.

Os dados.

Do componente.

---

Ela também.

Guarda.

Informações.

Do Runtime.

---

# Campos importantes

Cada.

Campo.

Possui.

Uma função.

---

```text
props
```

Dados.

Recebidos.

Do pai.

---

```text
slots
```

Conteúdo.

Projetado.

---

```text
setupState
```

Retorno.

Do `setup()`.

---

```text
ctx
```

Contexto.

Interno.

---

```text
proxy
```

Objeto.

Utilizado.

Pelo Render.

---

```text
subTree
```

VNode.

Gerado.

Pelo componente.

---

```text
update
```

Reactive Effect.

Responsável.

Pela renderização.

---

# Fluxo de montagem

Visualmente.

```text
VNode

↓

createComponentInstance()

↓

setupComponent()

↓

setup()

↓

render()

↓

VNode

↓

Patch()
```

---

Esse.

É o ciclo.

Mais importante.

Do Runtime.

---

# setupComponent()

Depois.

De criar.

A instância.

---

Executamos.

```javascript
setupComponent(instance)
```

---

Essa função.

Inicializa.

---

Props.

---

Slots.

---

Emit.

---

Setup.

---

Proxy.

---

# Inicializando Props

Imagine.

```vue
<Card

:title="titulo"
/>
```

---

Na montagem.

---

Copiamos.

Os valores.

---

```javascript
instance.props={

title:"Vue"

}
```

---

Depois.

Congelamos.

A referência.

Sempre.

Que possível.

---

# Inicializando Slots

Da mesma forma.

---

```vue
<Card>

<h1/>

</Card>
```

---

O conteúdo.

É armazenado.

---

```javascript
instance.slots
```

---

Posteriormente.

Será utilizado.

Pela Render Function.

---

# Emit

Todo componente.

Possui.

Uma função.

---

```javascript
emit()
```

---

Na MiniVue.

Podemos.

Criar.

```javascript
instance.emit=(event,...args)=>{

...
}
```

---

Depois.

Encontramos.

O listener.

No componente.

Pai.

---

Executamos.

O callback.

---

# setup()

Agora.

Chegamos.

Ao coração.

Do componente.

---

Chamamos.

```javascript
const result=

setup(

props,

context

)
```

---

O contexto.

Pode conter.

---

emit.

---

slots.

---

attrs.

---

expose.

---

# O retorno

O `setup()`.

Pode retornar.

Um objeto.

---

```javascript
return{

contador,

incrementar

}
```

---

Esse objeto.

É armazenado.

Em.

```javascript
setupState
```

---

Ou.

Pode retornar.

Uma função.

---

```javascript
return()=>{

...
}
```

---

Nesse caso.

Ela se torna.

A Render Function.

---

# Proxy

Depois.

Criamos.

O Proxy.

Da instância.

---

Por quê?

---

Porque.

No template.

Escrevemos.

```vue
{{contador}}
```

---

Não.

```javascript
setupState.contador
```

---

O Proxy.

Procura.

Na ordem.

---

```text
setupState

↓

props

↓

data

↓

ctx

↓

globalProperties
```

---

Assim.

A experiência.

Do desenvolvedor.

É simplificada.

---

# Render

Depois.

Chamamos.

```javascript
render.call(proxy)
```

---

O resultado.

É um.

VNode.

---

```text
Render()

↓

VNode
```

---

Esse VNode.

Recebe.

O nome.

De.

```text
subTree
```

---

Porque.

Representa.

A árvore.

Interna.

Do componente.

---

# Patch

Agora.

O Renderer.

Recebe.

Essa árvore.

---

```text
subTree

↓

Patch()

↓

DOM
```

---

A montagem.

Está.

Concluída.

---

# Atualizações

Quando.

Uma prop.

Ou.

Um estado.

Muda.

---

Executamos.

O Reactive Effect.

---

Fluxo.

```text
Reactive Effect

↓

Render()

↓

Novo subTree

↓

Patch()
```

---

O Patch.

Compara.

---

```text
subTree antigo

↓

subTree novo
```

---

Atualizando.

Somente.

O necessário.

---

# Component Tree

Cada instância.

Conhece.

Seu pai.

---

Também.

Seus filhos.

---

Visualmente.

```text
App

↓

Header

↓

Menu

↓

Button
```

---

Essa estrutura.

É utilizada.

Pelo.

---

Provide.

---

Inject.

---

DevTools.

---

Lifecycle.

---

Error Handling.

---

# Lifecycle

Durante.

A montagem.

Disparamos.

Os hooks.

---

```text
beforeCreate

↓

created

↓

beforeMount

↓

mounted
```

---

Nas atualizações.

---

```text
beforeUpdate

↓

updated
```

---

Na remoção.

---

```text
beforeUnmount

↓

unmounted
```

---

Todos.

São executados.

Pela instância.

---

# Como implementar?

Na MiniVue.

Crie.

Os arquivos.

```text
runtime-core/

component.js

componentProps.js

componentSlots.js

componentEmit.js

componentRender.js
```

---

Cada.

Arquivo.

Terá.

Uma única.

Responsabilidade.

---

Depois.

Implemente.

O fluxo.

```text
createComponentInstance()

↓

setupComponent()

↓

setup()

↓

render()
```

---

Gradualmente.

Nossa MiniVue.

Terá.

Uma arquitetura.

Muito próxima.

Da oficial.

---

# Performance

A separação.

Entre.

Definição.

E instância.

Permite.

---

Reutilização.

---

Melhor.

Gerenciamento.

De memória.

---

Atualizações.

Mais rápidas.

---

Menor.

Acoplamento.

---

# Código-fonte

Os principais arquivos para estudar são:

```text
packages/runtime-core/src/component.ts
```

---

```text
packages/runtime-core/src/componentProps.ts
```

---

```text
packages/runtime-core/src/componentSlots.ts
```

---

```text
packages/runtime-core/src/componentRenderUtils.ts
```

---

```text
packages/runtime-core/src/componentEmits.ts
```

---

```text
packages/runtime-core/src/componentOptions.ts
```

---

Esses módulos implementam praticamente toda a lógica relacionada à criação, configuração e atualização das instâncias de componentes do Vue.

---

# Curiosidade

No Vue 3, uma Component Instance contém muito mais informações do que normalmente imaginamos. Além do estado do componente, ela gerencia dependências reativas, contexto da aplicação, escopos de efeitos (`EffectScope`), integração com DevTools, hooks de ciclo de vida, Scheduler e referências para otimizações internas do Renderer.

---

# Resumo

Neste capítulo aprendemos que:

* Um componente possui uma definição e uma instância.
* A Component Instance concentra todo o estado necessário para o Runtime.
* `setupComponent()` inicializa props, slots, emit, proxy e `setup()`.
* A Render Function gera uma `subTree`, utilizada pelo Renderer.
* O Reactive Effect é responsável por atualizar componentes.
* A árvore de componentes é utilizada por diversos recursos do framework.

---

# Exercícios

## Exercício 1

Implemente `createComponentInstance()` contendo os principais campos da instância.

---

## Exercício 2

Implemente `setupComponent()` inicializando `props`, `slots` e `emit`.

---

## Exercício 3

Adicione suporte ao retorno em objeto e em função do `setup()`.

---

## Exercício 4

Implemente um Proxy que procure propriedades em `setupState`, `props` e `ctx`.

---

## Exercício 5

Monte dois componentes aninhados e desenhe a árvore completa de instâncias criada durante a execução.

---

# Desafio

Atualize sua **MiniVue** para suportar:

* Component Instances completas;
* `setup()`;
* `props`;
* `slots`;
* `emit`;
* Proxy da instância;
* Render Function;
* ciclo de atualização baseado em `Reactive Effect`.

---

# Projeto do capítulo

Ao final deste capítulo sua MiniVue deverá possuir um sistema completo de componentes, com instâncias independentes, gerenciamento de estado, renderização e integração com o Runtime, reproduzindo a base da arquitetura utilizada pelo Vue 3.

---

# Checklist

* [ ] Entendi a diferença entre definição e instância de componente.
* [ ] Sei implementar `createComponentInstance()`.
* [ ] Entendi o fluxo `setupComponent() → setup() → render()`.
* [ ] Minha MiniVue suporta `props`, `slots` e `emit`.
* [ ] O sistema de componentes está integrado ao Runtime.

---

# Próximo capítulo

## **Capítulo 52 — Projeto Final III: Integrando o Compiler ao Runtime (Template → AST → Render Function → DOM)**

No próximo capítulo concluiremos um dos ciclos mais importantes do framework: conectaremos o **Compiler** ao **Runtime**. Você aprenderá como um template percorre todas as etapas — parsing, geração da AST, transforms, geração da Render Function e execução pelo Renderer — até produzir o DOM. Ao final, sua MiniVue será capaz de renderizar componentes escritos com templates, consolidando toda a pipeline de compilação do Vue 3.
