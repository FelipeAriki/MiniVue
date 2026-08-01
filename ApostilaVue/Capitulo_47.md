# Capítulo 47 — Vue DevTools Internals: Instrumentação, Debugging e Integração com o Runtime

> **Objetivo:** compreender profundamente como o **Vue DevTools** funciona internamente. Ao final deste capítulo você entenderá como o Runtime do Vue expõe informações para ferramentas externas, como componentes, estado reativo, Pinia e Vue Router são inspecionados, como eventos de performance são registrados e como construir um sistema básico de inspeção na sua MiniVue.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Entender a arquitetura do Vue DevTools.
* Explicar como o Runtime é instrumentado.
* Compreender como a árvore de componentes é construída.
* Entender a inspeção de estado reativo.
* Explicar como Pinia e Vue Router se integram ao DevTools.
* Implementar um sistema básico de inspeção na MiniVue.

---

# Pré-requisitos

* Capítulos 01 ao 46.

---

# Introdução

Imagine.

Que você.

Abra.

O navegador.

---

Clique.

Na aba.

Vue.

Do DevTools.

---

Instantaneamente.

Você vê.

Toda.

A árvore.

De componentes.

---

Também.

Props.

---

State.

---

Refs.

---

Computed.

---

Watchers.

---

Eventos.

---

Performance.

---

Como.

O DevTools.

Consegue.

Enxergar.

Tudo isso?

---

A resposta.

É simples.

---

O Runtime.

Foi projetado.

Para.

Ser.

Instrumentado.

---

# O que significa instrumentação?

Instrumentar.

Um sistema.

Significa.

Adicionar.

Pontos.

De observação.

---

Sem alterar.

Seu comportamento.

---

Exemplo.

```text id="1xv0gr"
Renderer

↓

Evento

↓

DevTools
```

---

O Runtime.

Continua.

Funcionando.

---

Mas.

Também.

Envia.

Informações.

---

# Arquitetura

Visualmente.

```text id="dyhgn0"
Runtime

↓

DevTools Hook

↓

Bridge

↓

Vue DevTools

↓

Interface
```

---

Observe.

O Runtime.

Não conhece.

A interface.

Do DevTools.

---

Ele.

Apenas.

Publica.

Eventos.

---

# DevTools Hook

Durante.

A inicialização.

O Vue.

Procura.

Um objeto.

Global.

---

Semelhante.

A.

```javascript id="jchd7f"
window.__VUE_DEVTOOLS_GLOBAL_HOOK__
```

---

Se existir.

---

O Runtime.

Registra.

A aplicação.

---

Caso contrário.

Nada acontece.

---

Por isso.

O DevTools.

Não impacta.

A produção.

Quando ausente.

---

# Registro da aplicação

Quando.

Chamamos.

```javascript id="tyv7ob"
createApp(App)

.mount("#app")
```

---

O Runtime.

Emite.

Algo parecido.

Com.

```text id="sfrjlwm"
app:init
```

---

O DevTools.

Recebe.

Esse evento.

---

E registra.

A aplicação.

---

# Árvore de componentes

Depois.

O Runtime.

Percorre.

Todos.

Os componentes.

---

Visualmente.

```text id="qmfwpf"
App

↓

Header

↓

Sidebar

↓

Dashboard

↓

Card
```

---

Cada.

Instância.

É registrada.

---

Com.

Seu.

ID.

---

Nome.

---

Props.

---

State.

---

Parent.

---

Children.

---

# Estrutura

Simplificada.

```javascript id="sjlmfa"
{

id:1,

name:"Card",

props:{},

state:{},

children:[]

}
```

---

Essa.

Estrutura.

É enviada.

Ao DevTools.

---

# Atualizações

Imagine.

```javascript id="rhrjpn"
contador.value++
```

---

Depois.

Da renderização.

---

O Runtime.

Emite.

Um evento.

---

```text id="ckwgq0"
component:update
```

---

Assim.

O DevTools.

Atualiza.

A interface.

---

Sem.

Reconstruir.

Toda.

A árvore.

---

# Selecionando um componente

Quando.

Você.

Clica.

Em.

```text id="wmybhi"
Card
```

---

O DevTools.

Solicita.

Mais.

Informações.

---

O Runtime.

Responde.

Com.

---

Props.

---

Data.

---

Refs.

---

Computed.

---

Methods.

---

Watchers.

---

Tudo.

Em tempo.

Real.

---

# Estado reativo

Imagine.

```javascript id="snjlwm"
const nome=

ref("Felipe")
```

---

O DevTools.

Não recebe.

A referência.

Diretamente.

---

Ele recebe.

Uma representação.

Serializável.

---

Exemplo.

```javascript id="0bqvrr"
{

type:"Ref",

value:"Felipe"

}
```

---

Assim.

A interface.

Consegue.

Exibir.

O valor.

---

# Computed

Também.

São.

Instrumentados.

---

Exemplo.

```javascript id="i0m6lq"
computed(()=>{

...

})
```

---

O DevTools.

Mostra.

---

Valor atual.

---

Dependências.

---

Estado.

---

Se.

Está.

Cacheado.

---

# Watchers

Os Watchers.

Também.

Podem.

Ser.

Exibidos.

---

Principalmente.

Durante.

Debug.

---

O Runtime.

Sabe.

Quais.

Effects.

Estão ativos.

---

E.

Os envia.

Ao DevTools.

---

# Eventos

Outro.

Recurso.

Muito útil.

---

Quando.

Um componente.

Emite.

```javascript id="mjlwm8"
emit("save")
```

---

O Runtime.

Pode.

Registrar.

Esse evento.

---

Permitindo.

Visualizar.

A sequência.

De ações.

---

# Timeline

Existe.

Outra aba.

Muito importante.

---

Timeline.

---

Ela registra.

Eventos.

Como.

---

Mount.

---

Update.

---

Unmount.

---

Navigation.

---

Pinia.

---

Custom Events.

---

Visualmente.

```text id="d3qg1t"
10:00 Mount

10:01 Update

10:02 Save

10:03 Route
```

---

Isso.

Facilita.

Encontrar.

Problemas.

---

# Performance

Durante.

Cada render.

---

O Runtime.

Pode registrar.

---

Tempo inicial.

---

Tempo final.

---

Quantidade.

De componentes.

---

Quantidade.

De efeitos.

---

Tudo.

É enviado.

Para.

A Timeline.

---

# Pinia

O Pinia.

Também.

Utiliza.

A mesma.

Infraestrutura.

---

Cada Store.

É registrada.

---

Visualmente.

```text id="tdjlwm"
Stores

↓

Auth

↓

Cart

↓

Products
```

---

Ao alterar.

O estado.

---

O DevTools.

Mostra.

Antes.

E depois.

---

Facilitando.

O Debug.

---

# Vue Router

O Router.

Também.

Envia.

Eventos.

---

Exemplo.

```text id="g3bqlf"
beforeEach

↓

afterEach

↓

navigation
```

---

Assim.

A Timeline.

Mostra.

Toda.

A navegação.

---

# Bridge

Entre.

O Runtime.

E.

A Interface.

Existe.

Uma camada.

Chamada.

```text id="zjlwmq"
Bridge
```

---

Ela.

Transporta.

Mensagens.

---

Sem.

Acoplar.

O Runtime.

À interface.

---

Visualmente.

```text id="jepdvn"
Runtime

↓

Bridge

↓

DevTools UI
```

---

# Backend

O DevTools.

Possui.

Um Backend.

---

Responsável.

Por conversar.

Com.

O Runtime.

---

E um.

Frontend.

---

Responsável.

Por mostrar.

As informações.

---

Essa separação.

Facilita.

A manutenção.

---

# Como implementar?

Na MiniVue.

Podemos.

Criar.

Um Hook.

Global.

---

```javascript id="djlwm3"
window.__MINIVUE_DEVTOOLS__={}
```

---

Depois.

Sempre.

Que um.

Componente.

For criado.

---

Chamamos.

```javascript id="zrf8mt"
emit(

"component:add",

instance

)
```

---

Quando.

Atualizar.

---

```javascript id="wh7q9s"
emit(

"component:update"

)
```

---

Quando.

Remover.

---

```javascript id="1pwkgo"
emit(

"component:remove"

)
```

---

Agora.

Podemos.

Criar.

Uma interface.

---

Que apenas.

Escuta.

Esses eventos.

---

Sem alterar.

O Runtime.

---

# Estrutura

Visualmente.

```text id="3yohwx"
Runtime

↓

Hook

↓

Eventos

↓

MiniVue DevTools
```

---

Depois.

Podemos.

Adicionar.

---

Timeline.

---

Stores.

---

Router.

---

Performance.

---

Computed.

---

Watchers.

---

Gradualmente.

Nossa.

MiniVue.

Terá.

Seu próprio.

DevTools.

---

# Performance

Toda.

Essa instrumentação.

É opcional.

---

Quando.

O DevTools.

Não existe.

---

O Runtime.

Praticamente.

Ignora.

Esses.

Eventos.

---

Mantendo.

Alta.

Performance.

---

# Código-fonte

Grande parte da integração pode ser estudada em:

```text id="sllb72"
packages/runtime-core/src/devtools.ts
```

---

Também vale analisar:

```text id="qk7vzu"
packages/runtime-core/src/component.ts
```

---

```text id="h9r2jm"
packages/runtime-core/src/apiCreateApp.ts
```

---

No ecossistema do DevTools, também é interessante estudar os pacotes responsáveis pela comunicação entre Runtime e interface, que utilizam uma arquitetura baseada em eventos e uma camada intermediária (Bridge).

---

# Curiosidade

O Vue DevTools não acessa diretamente os objetos internos do Runtime. Em vez disso, ele se comunica através de um protocolo de eventos, permitindo que diferentes versões do Vue e do DevTools evoluam com menor acoplamento. Essa arquitetura também facilita a integração de bibliotecas como Pinia e Vue Router.

---

# Resumo

Neste capítulo aprendemos que:

* O Runtime é instrumentado através de Hooks.
* O DevTools recebe eventos em vez de acessar diretamente o Runtime.
* A árvore de componentes é construída a partir das instâncias registradas.
* Pinia e Vue Router utilizam a mesma infraestrutura de comunicação.
* A Timeline registra renderizações, navegações e eventos.
* A instrumentação é opcional e praticamente sem custo quando o DevTools não está presente.

---

# Exercícios

## Exercício 1

Implemente um Hook global para registrar componentes montados.

---

## Exercício 2

Implemente eventos para atualização e remoção de componentes.

---

## Exercício 3

Crie uma árvore hierárquica representando todos os componentes ativos.

---

## Exercício 4

Adicione uma Timeline simples registrando mounts, updates e unmounts.

---

## Exercício 5

Implemente um painel que exiba Props e State do componente selecionado.

---

# Desafio

Atualize sua **MiniVue** para suportar:

* Hook global de DevTools;
* registro de componentes;
* árvore de componentes;
* Timeline;
* inspeção de Props, State e Refs;
* eventos de atualização.

---

# Projeto do capítulo

Ao final deste capítulo sua MiniVue deverá possuir um sistema básico de inspeção semelhante ao Vue DevTools, capaz de registrar componentes, acompanhar atualizações e fornecer informações úteis para depuração da aplicação.

---

# Checklist

* [ ] Entendi como o Vue DevTools se comunica com o Runtime.
* [ ] Sei explicar o papel do DevTools Hook.
* [ ] Entendi como a árvore de componentes é construída.
* [ ] Sei como Pinia e Vue Router se integram ao DevTools.
* [ ] Minha MiniVue possui um sistema básico de inspeção.

---

# Próximo capítulo

## **Capítulo 48 — Vue Core Source Code Tour: Navegando e Entendendo Todo o Repositório Oficial**

No próximo capítulo faremos um tour completo pelo **repositório oficial do Vue 3**. Você aprenderá a localizar rapidamente qualquer funcionalidade no código-fonte, entenderá a organização dos pacotes (`compiler-core`, `compiler-dom`, `runtime-core`, `runtime-dom`, `reactivity`, `shared`, `server-renderer` etc.), conhecerá as convenções utilizadas pela equipe do Vue e desenvolverá a habilidade de navegar pelo código do framework como um mantenedor do projeto.
