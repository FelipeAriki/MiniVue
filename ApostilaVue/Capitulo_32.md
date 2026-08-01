# Capítulo 32 — Teleport: Renderizando Componentes Fora da Árvore do DOM

> **Objetivo:** compreender profundamente o funcionamento do **Teleport** no Vue 3. Ao final deste capítulo você entenderá por que ele existe, como funciona internamente no Renderer, como preserva a árvore de componentes enquanto move elementos no DOM e como implementar uma versão simplificada na MiniVue.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Entender o propósito do Teleport.
* Explicar por que ele existe.
* Utilizar Teleport corretamente.
* Compreender seu funcionamento interno.
* Entender como ele se integra ao Renderer.
* Implementar uma versão simplificada na MiniVue.

---

# Pré-requisitos

* Capítulos 01 ao 31.

---

# Recapitulando

Até agora aprendemos.

```text
Template

↓

Render Function

↓

VNode

↓

Renderer

↓

DOM
```

Sempre assumimos.

Que um componente.

Renderiza.

Dentro.

Do componente pai.

Mas...

Isso nem sempre é desejado.

---

# O problema

Imagine um modal.

```text
App

└── Página

      └── Formulário

            └── Modal
```

O Modal.

Está dentro.

Do formulário.

---

No DOM.

Teremos algo parecido.

```html
<body>

<div id="app">

<div>

<form>

<div class="modal">

...

</div>

</form>

</div>

</div>

</body>
```

---

# Qual o problema?

O Modal.

Pode sofrer.

Com:

* `overflow: hidden`
* `z-index`
* `position`
* `transform`
* clipping
* stacking context

---

# Exemplo

Imagine.

```css
.container{

overflow:hidden;

}
```

O Modal.

Será cortado.

Mesmo usando.

```css
position:fixed;
```

---

# A solução

Mover.

O elemento.

Para outro lugar.

Do DOM.

---

Mas...

Sem perder.

Sua posição.

Na árvore.

De componentes.

---

É exatamente isso.

Que o Teleport faz.

---

# Primeiro exemplo

```vue
<Teleport to="body">

<Modal/>

</Teleport>
```

---

Visualmente.

A árvore.

Continua.

```text
App

↓

Página

↓

Modal
```

---

Mas.

No DOM.

Temos.

```html
<body>

<div id="app">

...

</div>

<div class="modal">

...

</div>

</body>
```

Observe.

O Modal.

Saiu.

Do App.

---

# Importante

O Teleport.

Move.

A renderização.

---

Ele não move.

O componente.

---

A árvore lógica.

Permanece.

Exatamente igual.

---

# Isso significa

O componente.

Continua tendo.

---

Props.

---

Emits.

---

Inject.

---

Provide.

---

Lifecycle.

---

Refs.

---

Tudo.

Continua funcionando.

---

# Fluxo

```text
Árvore de Componentes

↓

App

↓

Modal

↓

VNode

↓

Teleport

↓

BODY
```

---

# A propriedade

```vue
<Teleport

to="#modal"

/>
```

---

Pode receber.

---

Selector CSS.

---

HTMLElement.

---

Outro destino.

---

# Outro exemplo

```vue
<Teleport

to="#sidebar"

>

<Menu/>

</Teleport>
```

O Menu.

Será renderizado.

Na Sidebar.

---

# Desabilitando

Existe.

A propriedade.

```vue
<Teleport

:disabled="true"

/>
```

---

Quando.

```javascript
disabled=true
```

O componente.

Renderiza.

No lugar original.

---

Muito útil.

Para.

Testes.

---

SSR.

---

Layouts.

---

# Múltiplos Teleports

Podemos fazer.

```vue
<Teleport to="body">

<Modal1/>

</Teleport>

<Teleport to="body">

<Modal2/>

</Teleport>
```

---

Resultado.

Ambos.

Serão inseridos.

No mesmo destino.

---

# Como funciona?

O Compiler.

Produz.

Um VNode.

Especial.

---

Ao invés de.

```javascript
type:"div"
```

Temos.

```javascript
type:Teleport
```

---

O Renderer.

Reconhece.

Esse tipo.

---

Fluxo.

```text
VNode

↓

ShapeFlag

↓

TELEPORT

↓

processTeleport()
```

---

# Dentro do Renderer

Existe uma função.

Semelhante a.

```javascript
processTeleport(

oldVNode,

newVNode,

container

)
```

---

Ela decide.

---

Montar.

---

Atualizar.

---

Mover.

---

Remover.

---

# Montagem

Primeiro.

O Vue.

Localiza.

O destino.

---

```javascript
const target=

document.querySelector(

props.to

)
```

---

Depois.

Renderiza.

Os filhos.

No destino.

---

Não.

No container original.

---

# Observe

O componente.

Continua.

Filho.

Do componente pai.

---

Mas.

Os elementos.

São inseridos.

Em outro.

Container.

---

# Atualização

Quando.

O estado muda.

---

A Render Function.

Continua.

Gerando.

Novos VNodes.

---

O Patch.

Continua.

Executando.

Normalmente.

---

A única diferença.

É.

O container.

---

# Mudando o destino

Imagine.

```vue
<Teleport

:to="destino"

/>
```

---

Antes.

```javascript
destino="#left"
```

---

Depois.

```javascript
destino="#right"
```

---

O Renderer.

Move.

Todos.

Os filhos.

---

Sem recriá-los.

---

# Lifecycle

Um detalhe.

Muito importante.

---

Mesmo renderizando.

No BODY.

---

O componente.

Continua.

Sendo filho.

Do componente pai.

---

Logo.

```javascript
onMounted()
```

Continua.

Executando.

Normalmente.

---

Também.

```javascript
onUnmounted()
```

---

Watch.

---

Computed.

---

Provide.

---

Inject.

---

Tudo.

Continua igual.

---

# Por quê?

Porque.

Teleport.

Move.

Elementos.

---

Não.

Instâncias.

De componentes.

---

# Exemplo

```vue
<App>

<Teleport>

<Modal/>

</Teleport>

</App>
```

---

Árvore lógica.

```text
App

↓

Modal
```

---

Árvore DOM.

```text
BODY

↓

Modal
```

---

São.

Duas árvores.

Diferentes.

---

# Eventos

Imagine.

```vue
<button

@click="abrir"
/>
```

Dentro.

Do Modal.

---

O evento.

Continua.

Funcionando.

---

Porque.

Está ligado.

À instância.

Do componente.

---

Não.

À posição.

No DOM.

---

# Casos reais

Modais.

---

Dropdowns.

---

Tooltips.

---

Context Menus.

---

Toasts.

---

Notificações.

---

Popovers.

---

Date Pickers.

---

Quase todas.

As bibliotecas.

De UI.

Utilizam.

Teleport.

---

# PrimeVue

Componentes.

Como.

```text
Dialog

OverlayPanel

ConfirmPopup

Tooltip
```

Utilizam.

Teleport.

Ou estratégias.

Semelhantes.

---

# Como implementar?

Na MiniVue.

Podemos começar.

Assim.

```javascript
function processTeleport(

vnode

){

const target=

document.querySelector(

vnode.props.to

)

mountChildren(

vnode.children,

target

)

}
```

---

É.

Uma implementação.

Muito simples.

---

Mas já demonstra.

O conceito.

---

# Diferença

Elemento comum.

```text
VNode

↓

Container

↓

DOM
```

---

Teleport.

```text
VNode

↓

Target

↓

DOM
```

---

# Fluxo completo

```text
Render Function

↓

VNode

↓

Renderer

↓

processTeleport()

↓

Target

↓

DOM
```

---

# Integração com Shape Flags

Internamente.

O VNode.

Recebe.

Uma flag.

```text
TELEPORT
```

---

O Renderer.

Pergunta.

```javascript
if(

shapeFlag & TELEPORT

){

processTeleport()

}
```

---

Assim.

Não precisa.

Descobrir.

O tipo.

Toda vez.

---

# Código-fonte

A implementação principal.

Está em.

```text
packages/runtime-core/src/components/Teleport.ts
```

Vale observar.

As funções.

```text
process()

move()

remove()

hydrate()
```

Além da integração.

Com.

```text
renderer.ts
```

---

# Curiosidade

Apesar do nome sugerir "teletransporte", nenhum componente é realmente movido pela aplicação. O que muda é apenas o **local onde os nós do DOM são inseridos**. A árvore de componentes continua exatamente a mesma, permitindo que reatividade, ciclo de vida, `provide/inject` e comunicação entre componentes funcionem sem qualquer alteração.

---

# Resumo

Neste capítulo aprendemos que:

* Teleport permite renderizar elementos fora do componente pai.
* A árvore de componentes permanece inalterada.
* Apenas a posição dos elementos no DOM muda.
* O Renderer trata Teleport como um tipo especial de VNode.
* Bibliotecas de UI utilizam Teleport para overlays, modais e popups.
* É possível implementar uma versão simplificada utilizando um destino alternativo no Renderer.

---

# Exercícios

## Exercício 1

Crie um modal utilizando `Teleport`.

---

## Exercício 2

Implemente um componente Toast utilizando `Teleport`.

---

## Exercício 3

Adicione suporte a um destino dinâmico (`:to`) em sua MiniVue.

---

## Exercício 4

Implemente a propriedade `disabled`.

---

## Exercício 5

Abra `packages/runtime-core/src/components/Teleport.ts` e identifique como o Vue trata montagem, atualização e movimentação de Teleports.

---

# Desafio

Atualize sua **MiniVue** para suportar:

* `Teleport`;
* destinos dinâmicos;
* múltiplos Teleports para o mesmo alvo;
* movimentação entre destinos sem recriação dos elementos.

---

# Projeto do capítulo

Ao final deste capítulo sua MiniVue deverá permitir renderizar qualquer subtree em outro ponto do DOM, preservando a árvore lógica de componentes e aproximando ainda mais sua arquitetura da implementação oficial do Vue.

---

# Checklist

* [ ] Sei explicar por que o Teleport existe.
* [ ] Entendi a diferença entre árvore de componentes e árvore do DOM.
* [ ] Sei utilizar `Teleport` em aplicações reais.
* [ ] Entendi como o Renderer trata um VNode do tipo Teleport.
* [ ] Minha MiniVue possui uma implementação básica de `Teleport`.

---

# Próximo capítulo

## **Capítulo 33 — KeepAlive: Cache Inteligente de Componentes e Gerenciamento de Estado**

No próximo capítulo estudaremos um dos componentes internos mais sofisticados do Vue: **KeepAlive**. Você aprenderá como o Vue mantém componentes montados em memória, como funciona o cache interno, as estratégias de ativação e desativação (`activated`/`deactivated`), políticas de descarte (`LRU Cache`) e como implementar um sistema de cache semelhante na sua MiniVue.
