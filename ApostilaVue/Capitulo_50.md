# Capítulo 50 — Projeto Final I: Construindo uma MiniVue Completa — Arquitetura Geral e Integração dos Módulos

> **Objetivo:** iniciar a construção de uma MiniVue completa, integrando todos os módulos desenvolvidos ao longo do curso. Ao final deste capítulo você terá uma arquitetura profissional inspirada no Vue 3, entenderá como cada pacote se conecta aos demais e implementará o bootstrap do framework.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Integrar todos os módulos da MiniVue.
* Organizar o projeto seguindo a arquitetura do Vue.
* Entender o fluxo completo da aplicação.
* Implementar o bootstrap do framework.
* Preparar a MiniVue para receber aplicações reais.

---

# Pré-requisitos

* Capítulos 01 ao 49.

---

# Introdução

Durante.

Todo.

O curso.

Construímos.

Diversos.

Módulos.

Separadamente.

---

Reatividade.

---

Compiler.

---

Runtime.

---

Renderer.

---

Scheduler.

---

DevTools.

---

Agora.

Chegou.

O momento.

De unir.

Tudo.

---

A partir.

Deste capítulo.

Nossa.

MiniVue.

Deixa.

De ser.

Um conjunto.

De exemplos.

---

E passa.

A ser.

Um framework.

---

# Visão geral

Até agora.

Tínhamos.

Peças.

Independentes.

---

Agora.

Precisamos.

Montar.

A máquina.

Completa.

---

Visualmente.

```text
Application

↓

createApp()

↓

Runtime Core

↓

Renderer

↓

Runtime DOM

↓

Browser
```

---

Ao mesmo tempo.

Existe.

Outro fluxo.

```text
Template

↓

Compiler

↓

Render Function

↓

Runtime
```

---

E outro.

```text
Reactive State

↓

Scheduler

↓

Renderer

↓

DOM
```

---

Todos.

Precisam.

Conversar.

Entre si.

---

# Arquitetura final

Nossa.

MiniVue.

Terá.

A seguinte.

Estrutura.

```text
mini-vue/

packages/

shared/

reactivity/

compiler/

runtime-core/

runtime-dom/

renderer/

scheduler/

devtools/

server-renderer/

examples/

tests/
```

---

Cada módulo.

Possui.

Uma única.

Responsabilidade.

---

Essa.

É uma.

Das maiores.

Lições.

Do Vue.

---

# Fluxo completo

Quando.

O usuário.

Executa.

```javascript
createApp(App)
```

---

Acontece.

O seguinte.

```text
createApp()

↓

Application Instance

↓

Renderer

↓

Root Component

↓

VNode

↓

Patch()

↓

DOM
```

---

Durante.

Esse processo.

Todos.

Os módulos.

São utilizados.

---

# O papel do Compiler

O Compiler.

Não participa.

Da execução.

Da aplicação.

---

Ele participa.

Antes.

---

Fluxo.

```text
Template

↓

Compiler

↓

Render Function
```

---

Depois disso.

O Runtime.

Assume.

O controle.

---

# O papel da Reactivity

Sempre.

Que.

Um estado.

É alterado.

---

Executamos.

```javascript
ref.value++
```

---

O fluxo.

É.

```text
trigger()

↓

Scheduler

↓

Renderer

↓

Patch()

↓

DOM
```

---

Observe.

Como.

Compiler.

E Runtime.

São.

Independentes.

---

# Bootstrap

Toda.

Aplicação.

Começa.

Aqui.

```javascript
createApp(App)
```

---

Depois.

```javascript
.mount("#app")
```

---

Nosso.

Objetivo.

É implementar.

Essa API.

---

# Estrutura do createApp()

Começaremos.

Com.

```javascript
function createApp(rootComponent){

return{

mount(){}

}

}
```

---

Esse.

Será.

Nosso.

Ponto.

De entrada.

---

# Mount

Quando.

Chamamos.

```javascript
app.mount("#app")
```

---

Precisamos.

Criar.

O componente.

Raiz.

---

Depois.

Gerar.

Seu.

VNode.

---

Depois.

Executar.

O Renderer.

---

Fluxo.

```text
Root Component

↓

VNode

↓

Renderer

↓

DOM
```

---

# Criando a instância

Cada.

Aplicação.

Possui.

Uma instância.

---

Exemplo.

```javascript
const app={

component,

config,

provides,

plugins,

version

}
```

---

Observe.

Que.

Essa instância.

Controla.

Toda.

A aplicação.

---

# Configuração

Também.

Teremos.

Uma área.

Para.

Configurações.

---

Como.

```javascript
app.config
```

---

No futuro.

Ela poderá.

Conter.

---

Error Handler.

---

Warn Handler.

---

Global Properties.

---

Performance.

---

DevTools.

---

# Plugins

Outro.

Elemento.

Importante.

---

```javascript
app.use(plugin)
```

---

Inicialmente.

Podemos.

Implementar.

Uma versão.

Simples.

---

```javascript
plugin.install(app)
```

---

Assim.

Nossa.

MiniVue.

Já suporta.

Plugins.

---

# Componentes globais

Também.

Precisamos.

Permitir.

```javascript
app.component(

"MeuBotao",

Botao
)
```

---

Internamente.

Criamos.

Um registro.

---

```javascript
components.set(

name,

component

)
```

---

Depois.

O Renderer.

Consulta.

Esse registro.

---

# Diretivas globais

Da mesma forma.

```javascript
app.directive(

"focus",

directive
)
```

---

Tudo.

É armazenado.

Na aplicação.

---

# Provide global

Outra.

Responsabilidade.

Da aplicação.

---

```javascript
app.provide(

token,

value
)
```

---

Os componentes.

Poderão.

Consumir.

Esses valores.

Com.

```javascript
inject()
```

---

# Ciclo completo

Visualmente.

```text
createApp()

↓

Application

↓

mount()

↓

Component Instance

↓

Render Function

↓

VNode

↓

Renderer

↓

DOM
```

---

Essa.

Será.

A espinha dorsal.

Da MiniVue.

---

# DevTools

Durante.

O mount.

Também.

Podemos.

Emitir.

```text
app:init
```

---

Assim.

Nosso.

Mini DevTools.

Consegue.

Registrar.

A aplicação.

---

# Scheduler

O Renderer.

Não atualiza.

O DOM.

Diretamente.

---

Sempre.

Que possível.

Utilizamos.

O Scheduler.

---

Fluxo.

```text
State

↓

Reactive Effect

↓

Scheduler

↓

Renderer
```

---

# Organização

A comunicação.

Entre.

Os módulos.

Deve seguir.

Esta direção.

```text
Compiler

↓

Runtime Core

↓

Renderer

↓

Runtime DOM
```

---

Nunca.

O contrário.

---

Isso.

Evita.

Dependências.

Circulares.

---

# Dependências

Visualmente.

```text
shared

↓

reactivity

↓

runtime-core

↓

runtime-dom
```

---

O Runtime.

Pode.

Usar.

Reactivity.

---

Mas.

Reactivity.

Nunca.

Conhece.

O Runtime.

---

Essa.

É uma.

Regra.

Importante.

---

# Como implementar?

Na MiniVue.

Crie.

O arquivo.

```text
runtime-core/

createApp.js
```

---

Depois.

Implemente.

Uma versão.

Inicial.

```javascript
export function createApp(root){

return{

mount(){},

use(){},

component(){},

directive(){},

provide(){}

}

}
```

---

Gradualmente.

Vamos.

Expandir.

Essa API.

Até ficar.

Muito próxima.

Do Vue.

---

# Performance

Uma boa.

Arquitetura.

Não melhora.

Apenas.

A organização.

---

Ela também.

Facilita.

Otimizações.

---

Testes.

---

Refatorações.

---

Novas features.

---

E manutenção.

---

# Código-fonte

Os principais arquivos para este capítulo são:

```text
packages/runtime-core/src/apiCreateApp.ts
```

---

```text
packages/runtime-core/src/component.ts
```

---

```text
packages/runtime-core/src/renderer.ts
```

---

```text
packages/runtime-core/src/apiInject.ts
```

---

```text
packages/runtime-core/src/apiLifecycle.ts
```

---

Esses módulos formam a base da inicialização de uma aplicação Vue e mostram como o Runtime organiza a criação da aplicação, das instâncias de componentes e do processo de renderização.

---

# Curiosidade

Embora `createApp()` pareça uma função simples, ela é responsável por inicializar praticamente toda a infraestrutura do Vue. É nesse momento que são configurados plugins, componentes globais, diretivas, valores compartilhados (`provide`), integração com DevTools e o Renderer responsável por montar a aplicação.

---

# Resumo

Neste capítulo aprendemos que:

* A MiniVue passa a integrar todos os módulos desenvolvidos anteriormente.
* `createApp()` é o ponto de entrada da aplicação.
* O Runtime coordena Compiler, Reactivity, Renderer e Scheduler.
* A arquitetura em camadas reduz acoplamento e facilita evolução.
* A aplicação passa a suportar plugins, componentes globais, diretivas e `provide`.

---

# Exercícios

## Exercício 1

Implemente uma versão funcional de `createApp()`.

---

## Exercício 2

Adicione suporte a `app.use()` para registrar plugins.

---

## Exercício 3

Implemente um registro de componentes globais.

---

## Exercício 4

Adicione suporte a `app.provide()` e `inject()`.

---

## Exercício 5

Desenhe o diagrama completo do fluxo desde `createApp()` até a atualização do DOM.

---

# Desafio

Construa o bootstrap completo da sua **MiniVue**, integrando:

* Runtime Core;
* Reactivity;
* Renderer;
* Runtime DOM;
* Scheduler;
* DevTools;
* Sistema de Plugins.

O objetivo é que uma aplicação simples possa ser montada utilizando apenas a API pública do framework.

---

# Projeto do capítulo

Ao final deste capítulo sua MiniVue deverá possuir uma arquitetura integrada e um ponto de entrada (`createApp`) capaz de inicializar toda a infraestrutura do framework, servindo como base para aplicações completas.

---

# Checklist

* [ ] Entendi a arquitetura completa da MiniVue.
* [ ] Sei explicar como os módulos se conectam.
* [ ] Implementei um `createApp()` funcional.
* [ ] Minha MiniVue suporta plugins e componentes globais.
* [ ] O bootstrap da aplicação está integrado ao Runtime.

---

# Próximo capítulo

## **Capítulo 51 — Projeto Final II: Implementando um Sistema Completo de Componentes**

No próximo capítulo construiremos o sistema completo de componentes da MiniVue. Implementaremos instâncias de componentes, ciclo de vida, `setup()`, `render()`, gerenciamento de `props`, `slots`, `emit`, árvore de componentes e o processo completo de montagem e atualização, aproximando a MiniVue ainda mais da arquitetura real do Vue 3.
