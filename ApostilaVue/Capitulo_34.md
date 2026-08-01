# Capítulo 34 — Suspense: Renderização Assíncrona e Coordenação de Componentes

> **Objetivo:** compreender profundamente o funcionamento do **Suspense** no Vue 3. Ao final deste capítulo você entenderá como o Vue coordena componentes assíncronos, controla estados de carregamento, integra o Scheduler ao Renderer e como implementar uma versão simplificada desse mecanismo na MiniVue.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Entender por que o Suspense existe.
* Explicar o conceito de componentes assíncronos.
* Utilizar `Suspense` em aplicações reais.
* Entender como o Renderer controla dependências assíncronas.
* Explicar como o Scheduler participa desse processo.
* Implementar uma versão simplificada na MiniVue.

---

# Pré-requisitos

* Capítulos 01 ao 33.

---

# Recapitulando

Até agora.

Todo componente.

Era renderizado.

Imediatamente.

```text
Render Function

↓

VNode

↓

Renderer

↓

DOM
```

Mas...

E quando.

Um componente.

Depende.

De uma operação.

Assíncrona?

---

# O problema

Imagine.

Um componente.

Que precisa.

Buscar dados.

Da API.

```javascript
async setup(){

    const usuario =

    await buscarUsuario()

    return{

        usuario

    }

}
```

---

Enquanto.

A requisição.

Não termina.

O componente.

Não pode.

Ser renderizado.

---

Sem Suspense.

Precisaríamos.

Controlar isso.

Manualmente.

---

Exemplo.

```vue
<template>

<div v-if="loading">

Carregando...

</div>

<Usuario

v-else

/>

</template>
```

---

Funciona.

Mas.

Escala mal.

---

Imagine.

Cinco componentes.

Assíncronos.

Na mesma tela.

---

Teríamos.

Cinco estados.

De loading.

---

Cinco.

Tratamentos.

---

Muito código.

---

# A solução

O Vue.

Introduziu.

```vue
<Suspense>

<Usuario/>

</Suspense>
```

---

Agora.

O próprio.

Framework.

Coordena.

A renderização.

---

# Primeiro exemplo

```vue
<Suspense>

<template #default>

<Usuario/>

</template>

<template #fallback>

Loading...

</template>

</Suspense>
```

---

Fluxo.

```text
Início

↓

Fallback

↓

Promise

↓

Resolveu

↓

Componente
```

---

# Observe

Enquanto.

O componente.

Espera.

A Promise.

---

O usuário.

Visualiza.

O fallback.

---

Quando.

A Promise.

Resolve.

---

O Vue.

Troca.

Automaticamente.

O conteúdo.

---

# Visualmente

```text
Loading...

↓

Usuário

↓

Tabela

↓

Botões
```

---

Tudo.

Sem código.

Manual.

---

# O que pode ser assíncrono?

Principalmente.

```javascript
async setup()
```

---

Também.

```javascript
defineAsyncComponent()
```

---

E componentes.

Que dependem.

De recursos.

Externos.

---

# Exemplo

```javascript
const MeuPainel =

defineAsyncComponent(

()=>import("./Painel.vue")

)
```

---

Enquanto.

O arquivo.

É carregado.

---

O Suspense.

Mostra.

O fallback.

---

# Múltiplos componentes

Imagine.

```vue
<Suspense>

<Usuario/>

<Pedidos/>

<Produtos/>

</Suspense>
```

---

O Suspense.

Espera.

Todos.

Terminarem.

---

Somente depois.

Renderiza.

O conteúdo.

---

# Fluxo

```text
Usuário

↓

Esperando

Pedidos

↓

Esperando

Produtos

↓

Esperando

↓

Todos terminaram

↓

Renderizar
```

---

# O Renderer

Quando.

Encontra.

Um Suspense.

---

Não renderiza.

Imediatamente.

---

Primeiro.

Conta.

As dependências.

Assíncronas.

---

Enquanto existir.

Pelo menos.

Uma Promise.

Pendente.

---

Renderiza.

O fallback.

---

# Internamente

Existe.

Um objeto.

Semelhante.

A.

```javascript
SuspenseBoundary
```

---

Ele controla.

* estado;
* dependências;
* fallback;
* conteúdo principal;
* troca entre ambos.

---

# Estrutura

Simplificando.

```javascript
{

pending:true,

deps:3,

fallback,

content

}
```

---

Cada Promise.

Resolvida.

Diminui.

O contador.

---

```text
3

↓

2

↓

1

↓

0
```

---

Quando chega.

Em.

```text
0
```

O conteúdo.

É liberado.

---

# Scheduler

Lembra.

Do capítulo.

Sobre Scheduler?

---

Ele.

Também participa.

---

Quando.

Uma Promise.

Resolve.

---

O Scheduler.

Agenda.

Uma atualização.

---

Fluxo.

```text
Promise

↓

Scheduler

↓

Queue

↓

Patch

↓

DOM
```

---

Assim.

O Vue.

Evita.

Renderizações.

Desnecessárias.

---

# Fallback

O fallback.

É apenas.

Outro VNode.

---

Exemplo.

```vue
<template #fallback>

<Spinner/>

</template>
```

---

Nada impede.

Que seja.

Complexo.

---

```vue
<Skeleton/>

```

---

```vue
<TabelaLoading/>

```

---

```vue
<CardLoading/>

```

---

Tudo funciona.

---

# Erros

Suspense.

Não substitui.

Tratamento.

De erros.

---

Para isso.

Utilizamos.

```javascript
onErrorCaptured()
```

---

Ou.

Error Boundaries.

---

# Lazy Loading

Uma utilização.

Muito comum.

É.

```javascript
defineAsyncComponent()
```

---

Assim.

O JavaScript.

Só é baixado.

Quando necessário.

---

Isso reduz.

O bundle inicial.

---

# Performance

Imagine.

Uma aplicação.

Com.

20 páginas.

---

Sem.

Lazy Loading.

---

Todo.

O JavaScript.

É baixado.

Na primeira visita.

---

Com.

Componentes.

Assíncronos.

---

Cada página.

É carregada.

Quando.

Necessário.

---

Resultado.

Melhor.

Tempo.

De carregamento.

---

# Integração

O Suspense.

Conversa.

Com.

```text
Renderer

↓

Scheduler

↓

Async Components

↓

VNode

↓

Patch
```

---

Ele.

Não substitui.

Nenhum.

Desses.

---

Ele coordena.

Todos.

---

# Como implementar?

Na MiniVue.

Podemos começar.

Assim.

```javascript
async function render(

component

){

if(

component.async

){

mostrarFallback()

await component.load()

}

renderComponent()

}
```

---

Muito simples.

---

Mas.

Já demonstra.

O conceito.

---

# Evoluindo

Depois.

Podemos criar.

```javascript
class SuspenseBoundary{

}
```

---

Responsável.

Por controlar.

Promises.

---

Fallback.

---

Conteúdo.

---

Trocas.

---

# Fluxo completo

```text
Render Function

↓

VNode

↓

Suspense

↓

Pending?

↓

Sim

↓

Fallback

↓

Promise

↓

Scheduler

↓

Renderer

↓

DOM
```

---

# Casos reais

Suspense.

É excelente.

Para.

* dashboards;
* páginas administrativas;
* lazy loading;
* componentes pesados;
* gráficos;
* mapas;
* editores ricos;
* visualizadores de documentos.

---

# Quando evitar

Não utilize.

Quando.

O componente.

É simples.

E síncrono.

---

Nem.

Para esconder.

Problemas.

De arquitetura.

---

Suspense.

Não substitui.

Boa modelagem.

---

# Código-fonte

Grande parte da implementação está em:

```text
packages/runtime-core/src/components/Suspense.ts
```

Também vale estudar a integração com:

```text
renderer.ts
```

e observar como o Scheduler é utilizado durante a resolução das dependências assíncronas.

---

# Curiosidade

O Suspense do Vue foi inspirado em conceitos semelhantes aos do React, mas possui uma implementação adaptada ao modelo de reatividade do Vue. Internamente, ele mantém uma estrutura chamada **Suspense Boundary**, responsável por controlar o conteúdo principal, o fallback e o número de dependências assíncronas pendentes.

---

# Resumo

Neste capítulo aprendemos que:

* Suspense coordena componentes assíncronos.
* O fallback é exibido enquanto existem dependências pendentes.
* O Scheduler agenda a atualização quando as Promises são resolvidas.
* `defineAsyncComponent()` e `async setup()` integram-se naturalmente ao Suspense.
* O Renderer utiliza uma estrutura interna para controlar esse fluxo.

---

# Exercícios

## Exercício 1

Crie um componente utilizando `async setup()` e renderize-o com `Suspense`.

---

## Exercício 2

Utilize `defineAsyncComponent()` para carregar uma página sob demanda.

---

## Exercício 3

Implemente um fallback personalizado utilizando Skeletons.

---

## Exercício 4

Implemente uma `SuspenseBoundary` simplificada na MiniVue.

---

## Exercício 5

Leia `packages/runtime-core/src/components/Suspense.ts` e identifique como o Vue controla o número de dependências pendentes.

---

# Desafio

Atualize sua **MiniVue** para suportar:

* componentes assíncronos;
* `async setup()`;
* fallback;
* contador de dependências;
* atualização automática após resolução das Promises.

---

# Projeto do capítulo

Ao final deste capítulo sua MiniVue deverá ser capaz de coordenar componentes assíncronos, exibir conteúdo temporário durante o carregamento e substituir automaticamente o fallback pelo conteúdo principal quando todas as dependências forem resolvidas.

---

# Checklist

* [ ] Sei explicar por que o Suspense existe.
* [ ] Entendi como funciona o fallback.
* [ ] Sei utilizar `defineAsyncComponent()`.
* [ ] Entendi o papel do Scheduler no Suspense.
* [ ] Minha MiniVue possui uma implementação básica de Suspense.

---

# Próximo capítulo

## **Capítulo 35 — Async Components: Lazy Loading, Code Splitting e Estratégias de Carregamento**

No próximo capítulo estudaremos os **Componentes Assíncronos** em profundidade. Você aprenderá como funciona `defineAsyncComponent()`, estratégias de *lazy loading*, *code splitting*, carregamento sob demanda, tratamento de erros, timeouts, componentes de loading, prefetch, preload e como essas funcionalidades se integram ao bundler (Vite/Rollup) e ao Runtime do Vue.
