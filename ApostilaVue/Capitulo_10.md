# Capítulo 10 — Scheduler, Job Queue e Batch Updates: Como o Vue Atualiza o DOM Apenas Uma Vez

> **Objetivo:** compreender como o Vue evita renderizações desnecessárias utilizando um Scheduler, uma fila de Jobs e Microtasks. Ao final deste capítulo você entenderá exatamente por que várias alterações consecutivas no estado resultam em apenas uma atualização do DOM.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Entender o Scheduler do Vue.
* Explicar o conceito de Job Queue.
* Compreender Batch Updates.
* Entender o papel do `nextTick()`.
* Implementar um Scheduler simplificado.
* Explicar por que o Vue é tão eficiente.

---

# Pré-requisitos

* Capítulos 01 ao 09.

---

# O problema

Nossa implementação atual possui um grande problema.

Imagine.

```javascript
const state = reactive({

    contador: 0

})

effect(() => {

    console.log(state.contador)

})
```

Agora.

```javascript
state.contador++

state.contador++

state.contador++

state.contador++
```

O que acontece?

Nosso `trigger()` executa imediatamente o Effect.

Resultado.

```text
1

2

3

4
```

O Effect foi executado quatro vezes.

---

# Em um componente Vue

Imagine um componente.

```vue
<template>

<p>{{ contador }}</p>

</template>
```

Agora.

```javascript
contador.value++

contador.value++

contador.value++

contador.value++
```

Sem Scheduler.

Teríamos.

```text
Render

Render

Render

Render
```

Quatro renderizações.

---

# O custo

Cada renderização pode significar:

* Virtual DOM.
* Diff.
* Patch.
* Atualização do DOM.
* Repaint.
* Reflow.

Tudo isso quatro vezes.

Extremamente caro.

---

# O que o Vue faz?

Em vez de atualizar imediatamente.

Ele diz.

> "Vou esperar terminar todas as alterações."

Depois.

Atualiza apenas uma vez.

---

# Visualizando

Sem Scheduler.

```text
Mudança

↓

Render

↓

Mudança

↓

Render

↓

Mudança

↓

Render
```

Com Scheduler.

```text
Mudança

↓

Mudança

↓

Mudança

↓

Mudança

↓

Uma renderização
```

---

# Batch Updates

Esse processo chama-se:

```text
Batch Updates
```

Ou.

Atualizações em lote.

---

# Como fazer isso?

Precisamos de uma fila.

Imagine.

```text
Job Queue
```

Toda atualização será colocada nessa fila.

---

# Primeira implementação

```javascript
const queue = []
```

Sempre que um Effect precisar executar.

Não executamos imediatamente.

Fazemos.

```javascript
queue.push(effect)
```

---

# O problema

Imagine.

```javascript
state.contador++

state.contador++

state.contador++
```

A fila ficaria.

```text
Effect

↓

Effect

↓

Effect
```

O mesmo Effect três vezes.

---

# O Vue utiliza Set

Em vez de Array.

```javascript
const queue = new Set()
```

Agora.

```javascript
queue.add(effect)

queue.add(effect)

queue.add(effect)
```

Resultado.

```text
Effect
```

Apenas uma vez.

---

# Quando executar?

Não queremos executar imediatamente.

Precisamos esperar a Stack terminar.

Lembra do capítulo de Microtasks?

---

# Utilizando Promise

```javascript
Promise.resolve().then(() => {

    flushJobs()

})
```

Isso agenda a execução como Microtask.

---

# Implementando

```javascript
const queue = new Set()

function queueJob(job){

    queue.add(job)

}
```

Até aqui.

Ainda não executamos nada.

---

# flushJobs()

Agora.

```javascript
function flushJobs(){

    queue.forEach(job=>{

        job.run()

    })

    queue.clear()

}
```

Esse método executará todos os Jobs pendentes.

---

# Integrando

Sempre que ocorrer um trigger.

Em vez de.

```javascript
effect.run()
```

Faremos.

```javascript
queueJob(effect)
```

---

# Ainda falta algo

Imagine.

```javascript
state.contador++

state.contador++

state.contador++
```

Cada alteração faria.

```javascript
Promise.resolve()
```

Teríamos três Microtasks.

Também é desperdício.

---

# O Vue controla isso

Utilizando uma flag.

```javascript
let isFlushing = false
```

---

# Melhorando

```javascript
function queueJob(job){

    queue.add(job)

    if(!isFlushing){

        isFlushing = true

        Promise.resolve()

            .then(flushJobs)

    }

}
```

Agora.

Apenas uma Microtask será criada.

---

# Limpando

Depois.

```javascript
function flushJobs(){

    queue.forEach(job=>{

        job.run()

    })

    queue.clear()

    isFlushing = false

}
```

---

# Fluxo completo

Imagine.

```javascript
state.contador++

state.contador++

state.contador++
```

Primeira alteração.

```text
Queue

↓

Effect
```

Agenda Microtask.

---

Segunda alteração.

```text
Queue

↓

Effect
```

Nada acontece.

O Set impede duplicação.

---

Terceira alteração.

Mesmo resultado.

---

Depois que a Stack termina.

```text
Microtask

↓

flushJobs()

↓

Effect
```

Executa apenas uma vez.

---

# Visualizando

```text
Stack

↓

Mudança

↓

Mudança

↓

Mudança

↓

Stack vazia

↓

Microtask

↓

flushJobs()

↓

Render
```

---

# O Scheduler

Agora entra um novo conceito.

Observe.

```javascript
effect(() => {

    console.log(state.contador)

})
```

Queremos que alguns Effects sejam executados imediatamente.

Outros.

Agendados.

---

# Scheduler

Cada Effect pode possuir.

```javascript
scheduler(effect)
```

Em vez de.

```javascript
effect.run()
```

Chamamos.

```javascript
effect.scheduler()
```

---

# Exemplo

```javascript
const effect = new ReactiveEffect(

    fn,

    () => {

        queueJob(effect)

    }

)
```

Agora.

Quando ocorrer.

```javascript
trigger()
```

O Vue pergunta.

```text
Existe Scheduler?
```

Se sim.

↓

Utiliza Scheduler.

Caso contrário.

↓

Executa imediatamente.

---

# Trigger real

De forma simplificada.

```javascript
effects.forEach(effect=>{

    if(effect.scheduler){

        effect.scheduler()

    }

    else{

        effect.run()

    }

})
```

Essa é praticamente a implementação do Vue.

---

# Onde isso é usado?

O Render Effect dos componentes.

Nunca executa imediatamente.

Sempre utiliza Scheduler.

Por isso.

```javascript
contador.value++

contador.value++

contador.value++
```

Renderiza apenas uma vez.

---

# nextTick()

Agora tudo faz sentido.

Imagine.

```javascript
contador.value++
```

Logo depois.

```javascript
console.log(

    document.body.innerHTML

)
```

O DOM ainda não foi atualizado.

Porque o Scheduler ainda não executou.

---

# Solução

```javascript
await nextTick()
```

Agora.

A Microtask já terminou.

O DOM está sincronizado.

---

# Implementação simplificada

```javascript
function nextTick(){

    return Promise.resolve()

}
```

Na prática.

O Vue reutiliza a mesma Promise utilizada pelo Scheduler.

Assim evita criar Promises desnecessárias.

---

# Fluxo do nextTick

```text
Mudança

↓

Scheduler

↓

Promise

↓

flushJobs()

↓

DOM atualizado

↓

nextTick()

↓

Seu código continua
```

---

# Ordem de execução

Observe.

```javascript
contador.value++

console.log(1)

await nextTick()

console.log(2)
```

Resultado.

```text
1

Render

2
```

---

# Scheduler vs setTimeout

Alguns imaginam.

```javascript
setTimeout(

flushJobs

)
```

Isso seria pior.

Porque.

`setTimeout`

↓

Macrotask.

Enquanto.

```javascript
Promise.resolve()
```

↓

Microtask.

Muito mais rápida.

---

# O Vue utiliza somente um Scheduler?

Não.

Internamente existem diferentes filas para diferentes tipos de Jobs.

Por exemplo.

* Pre Flush Jobs.
* Component Update Jobs.
* Post Flush Jobs.

Veremos isso mais adiante.

---

# Ordem dos Jobs

O Vue também ordena Jobs.

Por quê?

Imagine.

```text
App

↓

Pai

↓

Filho
```

O componente pai deve renderizar antes do filho.

O Scheduler garante essa ordem.

---

# Evitando loops infinitos

Imagine.

```javascript
effect(()=>{

    state.contador++

})
```

O Scheduler também ajuda a detectar loops de atualização.

O Vue possui limites internos para impedir renderizações infinitas.

---

# Performance

O Scheduler é uma das principais razões pelas quais o Vue consegue lidar com milhares de alterações de estado mantendo excelente desempenho.

Ele reduz drasticamente:

* renderizações;
* cálculos de `computed`;
* execução de watchers;
* atualizações do DOM.

---

# Comparação

Sem Scheduler.

```text
100 alterações

↓

100 renders
```

Com Scheduler.

```text
100 alterações

↓

1 render
```

Essa otimização é um dos pilares da performance do Vue.

---

# Resumo

Neste capítulo aprendemos que:

* o Vue nunca renderiza imediatamente um componente;
* atualizações são colocadas em uma Job Queue;
* a fila utiliza `Set` para evitar duplicações;
* o Scheduler agenda a execução utilizando Microtasks;
* `nextTick()` aguarda o término desse processo;
* Batch Updates reduzem drasticamente o número de renderizações.

---

# Exercícios

## Exercício 1

Implemente uma `Job Queue` utilizando `Set`.

---

## Exercício 2

Implemente `queueJob()` utilizando `Promise.resolve()`.

---

## Exercício 3

Explique por que o Vue utiliza Microtasks em vez de `setTimeout()`.

---

## Exercício 4

Implemente um Scheduler para sua biblioteca `MiniVue Reactive`.

---

# Desafio

Evolua sua implementação para que:

* `trigger()` nunca execute Effects diretamente;
* todos os Effects sejam agendados pelo Scheduler;
* múltiplas alterações consecutivas resultem em apenas uma execução.

---

# Projeto do capítulo

Atualize sua biblioteca **MiniVue Reactive** adicionando:

* `Job Queue`;
* `Scheduler`;
* `queueJob()`;
* `flushJobs()`;
* `nextTick()`;
* prevenção de Jobs duplicados;
* Batch Updates.

Ao final deste projeto sua biblioteca já terá uma arquitetura muito próxima da utilizada pelo runtime oficial do Vue.

---

# Checklist

* [ ] Entendi o conceito de Batch Updates.
* [ ] Sei explicar a Job Queue.
* [ ] Sei implementar um Scheduler.
* [ ] Entendi por que o Vue utiliza Microtasks.
* [ ] Sei explicar o funcionamento de `nextTick()`.
* [ ] Minha implementação já suporta atualizações em lote.

---

# Próximo capítulo

## **Capítulo 11 — `ref()`: Implementando o Primeiro Primitive Reactivo do Vue**

Neste capítulo construiremos o `ref()` do zero. Você entenderá por que ele possui a propriedade `.value`, como o Vue torna valores primitivos reativos, como funciona o `RefImpl` internamente e como `track()` e `trigger()` são utilizados para transformar um simples número em um valor totalmente reativo. A partir daqui, sua biblioteca começará a expor APIs muito semelhantes às do Vue 3.
