# Capítulo 45 — Vue Internals: Scheduler Avançado, Effect Scope, Flush Timing e Pipeline Completa de Atualizações

> **Objetivo:** compreender completamente o ciclo de atualização do Vue 3. Ao final deste capítulo você entenderá como uma alteração em um `ref()` percorre todo o sistema reativo até atualizar o DOM, como funciona o Scheduler, as filas de execução, `flush: "pre"`, `"post"` e `"sync"`, `EffectScope`, limpeza automática de efeitos e os mecanismos internos que evitam atualizações desnecessárias e loops infinitos.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Explicar toda a pipeline de atualização do Vue.
* Entender profundamente o Scheduler.
* Compreender Job Queue e Microtasks.
* Dominar `flush: "pre"`, `"post"` e `"sync"`.
* Entender `EffectScope`.
* Implementar um Scheduler avançado na MiniVue.

---

# Pré-requisitos

* Capítulos 01 ao 44.

---

# Introdução

Até agora.

Aprendemos.

Como funciona.

A reatividade.

---

Também.

O Renderer.

---

O Compiler.

---

O Runtime.

---

Agora.

Vamos responder.

Uma pergunta.

Muito importante.

---

Quando fazemos.

```javascript
contador.value++
```

---

Como.

Essa alteração.

Chega.

Ao DOM?

---

A resposta.

Passa.

Por uma.

Pipeline.

Muito sofisticada.

---

# Fluxo completo

```text
ref.value++

↓

trigger()

↓

Reactive Effect

↓

Scheduler

↓

Job Queue

↓

Renderer

↓

Patch

↓

DOM
```

---

Cada etapa.

Possui.

Uma responsabilidade.

Muito específica.

---

# Primeira etapa

Imagine.

```javascript
const contador = ref(0)

contador.value++
```

---

O Proxy.

Detecta.

A escrita.

---

Internamente.

Executa.

```javascript
trigger()
```

---

Esse.

É o início.

Da atualização.

---

# Trigger

O `trigger()`.

Localiza.

Todos.

Os efeitos.

Dependentes.

---

Visualmente.

```text
contador

↓

Effect A

↓

Effect B

↓

Effect C
```

---

Todos.

São encontrados.

Através.

Do Dependency Map.

---

# Reactive Effect

Cada componente.

Possui.

Um.

Reactive Effect.

---

Quando.

O trigger.

É executado.

---

O efeito.

Não renderiza.

Imediatamente.

---

Ele.

É enviado.

Para.

O Scheduler.

---

# Por quê?

Imagine.

```javascript
contador.value++

contador.value++

contador.value++
```

---

Sem Scheduler.

Teríamos.

```text
Render

↓

Render

↓

Render
```

---

Três.

Renderizações.

---

Com Scheduler.

```text
3 mudanças

↓

1 atualização
```

---

Esse.

É um dos.

Maiores ganhos.

De performance.

Do Vue.

---

# Job Queue

O Scheduler.

Mantém.

Uma fila.

---

Visualmente.

```text
Queue

↓

Job A

↓

Job B

↓

Job C
```

---

Cada Job.

Representa.

Uma atualização.

---

# Deduplicação

Imagine.

Que.

O mesmo.

Componente.

Receba.

Cinco.

Atualizações.

---

O Scheduler.

Não adiciona.

Cinco Jobs.

---

Ele verifica.

Se.

O Job.

Já existe.

---

Resultado.

```text
Queue

↓

Job
```

---

Apenas.

Uma execução.

---

# Como funciona?

Simplificando.

```javascript
const queue = []

if(!queue.includes(job)){

queue.push(job)

}
```

---

Na implementação.

Real.

São utilizadas.

Estruturas.

Mais eficientes.

---

# Microtasks

Outra.

Parte.

Fundamental.

---

O Vue.

Não executa.

A fila.

Imediatamente.

---

Ele agenda.

Uma.

Microtask.

---

Utilizando.

```javascript
Promise.resolve()
```

---

Fluxo.

```text
Trigger

↓

Queue

↓

Microtask

↓

Flush Jobs
```

---

Assim.

Todas.

As alterações.

Do mesmo.

Tick.

São agrupadas.

---

# nextTick()

Lembra.

Do.

```javascript
nextTick()
```

---

Ele.

Espera.

A fila.

Ser processada.

---

Exemplo.

```javascript
contador.value++

await nextTick()
```

---

Após.

O `await`.

O DOM.

Já está.

Atualizado.

---

# Flush Timing

Os Watchers.

Podem.

Executar.

Em momentos.

Diferentes.

---

Existem.

Três.

Modos.

---

```text
pre
```

---

```text
post
```

---

```text
sync
```

---

# flush: "pre"

É.

O padrão.

---

Executa.

Antes.

Da atualização.

Do DOM.

---

Fluxo.

```text
Watcher

↓

Renderer

↓

DOM
```

---

Muito útil.

Para.

Preparar.

Dados.

---

# flush: "post"

Agora.

O Watcher.

Executa.

Depois.

Da atualização.

---

Fluxo.

```text
Renderer

↓

DOM

↓

Watcher
```

---

Ideal.

Para.

Ler.

O DOM.

Atualizado.

---

Exemplo.

```javascript
watch(

estado,

callback,

{

flush:"post"

}

)
```

---

# flush: "sync"

Nesse modo.

O Watcher.

Executa.

Imediatamente.

---

Sem.

Scheduler.

---

Fluxo.

```text
Trigger

↓

Watcher
```

---

Deve.

Ser usado.

Com cuidado.

---

Pode.

Gerar.

Perda.

De performance.

---

# Effect Scope

Outro.

Recurso.

Muito importante.

---

Imagine.

Um componente.

---

Dentro dele.

Existem.

```text
watch()

watchEffect()

computed()

effects
```

---

Quando.

O componente.

É destruído.

---

Todos.

Esses.

Efeitos.

Precisam.

Ser removidos.

---

É exatamente.

O papel.

Do.

```text
EffectScope
```

---

# Funcionamento

Visualmente.

```text
Component

↓

Effect Scope

↓

Effect

↓

Watcher

↓

Computed
```

---

Quando.

O Scope.

É destruído.

---

Tudo.

É limpo.

Automaticamente.

---

# Exemplo

```javascript
const scope = effectScope()

scope.run(()=>{

watch(...)

watchEffect(...)

})
```

---

Depois.

```javascript
scope.stop()
```

---

Todos.

Os efeitos.

São removidos.

---

Muito útil.

Para.

Evitar.

Memory Leak.

---

# Pipeline completa

Imagine.

```javascript
contador.value++
```

---

Fluxo.

```text
ref

↓

Proxy

↓

trigger()

↓

Effects

↓

Scheduler

↓

Queue

↓

Microtask

↓

Renderer

↓

Patch

↓

DOM
```

---

Essa.

É uma.

Das partes.

Mais importantes.

Da arquitetura.

Do Vue.

---

# Loops infinitos

Imagine.

```javascript
watch(

contador,

()=>{

contador.value++

}

)
```

---

Sem proteção.

Teríamos.

```text
Trigger

↓

Watcher

↓

Trigger

↓

Watcher

↓

∞
```

---

O Vue.

Possui.

Mecanismos.

Para detectar.

Execuções.

Excessivas.

---

Quando.

Um limite.

É ultrapassado.

---

Um erro.

É lançado.

---

Protegendo.

A aplicação.

---

# Ordem dos Jobs

Nem todos.

Os Jobs.

Possuem.

A mesma.

Prioridade.

---

Por exemplo.

Componentes.

Pais.

Precisam.

Renderizar.

Antes.

Dos filhos.

---

O Scheduler.

Ordena.

Os Jobs.

Antes.

Da execução.

---

Garantindo.

Consistência.

---

# Scheduler Recursivo

Durante.

Um Render.

Podem surgir.

Novos.

Jobs.

---

O Scheduler.

Continua.

Processando.

Até.

A fila.

Ficar.

Vazia.

---

Mas.

Sem executar.

O mesmo.

Job.

Duas vezes.

No mesmo.

Ciclo.

---

# Como implementar?

Na MiniVue.

Podemos.

Começar.

Assim.

```javascript
const queue = new Set()
```

---

Adicionar.

Jobs.

```javascript
function queueJob(job){

queue.add(job)

}
```

---

Depois.

Agendar.

```javascript
Promise.resolve()

.then(flushJobs)
```

---

Executar.

```javascript
function flushJobs(){

for(

const job

of queue

){

job()

}

queue.clear()

}
```

---

Já temos.

Um Scheduler.

Básico.

---

Depois.

Adicionar.

* prioridades;
* flush timing;
* EffectScope;
* nextTick();
* deduplicação;
* prevenção de loops.

---

Gradualmente.

Nossa MiniVue.

Terá.

Um Scheduler.

Muito parecido.

Com.

O Vue.

---

# Performance

Grande parte.

Da performance.

Do Vue.

Não vem.

Apenas.

Da reatividade.

---

Mas.

Da forma.

Como.

O Scheduler.

Agrupa.

Atualizações.

---

Sem ele.

Grandes.

Aplicações.

Renderizariam.

Centenas.

De vezes.

Por segundo.

---

# Código-fonte

Grande parte da implementação pode ser estudada em:

```text
packages/runtime-core/src/scheduler.ts
```

---

Também vale analisar:

```text
packages/reactivity/src/effect.ts
```

---

```text
packages/reactivity/src/effectScope.ts
```

---

```text
packages/runtime-core/src/apiWatch.ts
```

---

Esses arquivos mostram como o Vue organiza a execução dos efeitos, controla a fila de atualização, implementa `nextTick()` e gerencia o ciclo de vida dos efeitos reativos.

---

# Curiosidade

O Scheduler do Vue foi projetado para garantir que **cada componente seja renderizado no máximo uma vez por ciclo de atualização**, mesmo que dezenas de propriedades reativas sejam modificadas. Essa decisão é um dos fatores que permitem que aplicações grandes mantenham boa performance mesmo sob alta frequência de atualizações.

---

# Resumo

Neste capítulo aprendemos que:

* `trigger()` inicia a pipeline de atualização.
* O Scheduler agrupa múltiplas alterações em um único ciclo.
* A Job Queue evita renderizações repetidas.
* `nextTick()` aguarda o processamento da fila.
* `flush: "pre"`, `"post"` e `"sync"` controlam o momento de execução dos Watchers.
* `EffectScope` gerencia automaticamente o ciclo de vida dos efeitos.
* O Vue protege a aplicação contra loops infinitos de atualização.

---

# Exercícios

## Exercício 1

Implemente uma Job Queue utilizando `Set` para evitar Jobs duplicados.

---

## Exercício 2

Implemente um `nextTick()` baseado em `Promise.resolve()`.

---

## Exercício 3

Adicione suporte a prioridades simples para os Jobs da sua fila.

---

## Exercício 4

Implemente um `EffectScope` que registre efeitos e os interrompa com `stop()`.

---

## Exercício 5

Leia `scheduler.ts` e identifique como o Vue organiza a ordem de execução dos componentes e impede renderizações duplicadas.

---

# Desafio

Atualize sua **MiniVue** para suportar:

* Scheduler avançado;
* Job Queue com deduplicação;
* `nextTick()`;
* `flush: "pre"`, `"post"` e `"sync"`;
* `EffectScope`;
* proteção contra loops infinitos.

---

# Projeto do capítulo

Ao final deste capítulo sua MiniVue deverá possuir um sistema de atualização altamente otimizado, capaz de agrupar renderizações, controlar a ordem de execução dos efeitos e gerenciar automaticamente o ciclo de vida da reatividade, reproduzindo a arquitetura utilizada pelo Vue 3.

---

# Checklist

* [ ] Sei explicar toda a pipeline de atualização do Vue.
* [ ] Entendi profundamente o Scheduler.
* [ ] Sei quando usar `flush: "pre"`, `"post"` e `"sync"`.
* [ ] Entendi como funciona o `EffectScope`.
* [ ] Minha MiniVue possui um Scheduler avançado.

---

# Próximo capítulo

## **Capítulo 46 — Custom Renderer: Criando um Renderer para Canvas, Terminal e Outras Plataformas**

No próximo capítulo você aprenderá um dos recursos mais elegantes da arquitetura do Vue: os **Custom Renderers**. Estudaremos como o `createRenderer()` permite reutilizar todo o Runtime Core para renderizar aplicações em plataformas diferentes do DOM, como Canvas, Terminal, aplicações nativas e ambientes customizados. Também construiremos um renderer completo para uma plataforma simplificada na MiniVue, consolidando o entendimento da arquitetura multiplataforma do Vue.
