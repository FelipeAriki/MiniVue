# Capítulo 03 — Microtasks e Macrotasks

> **Objetivo:** entender por que `Promise.then()` executa antes de `setTimeout()`, como o Event Loop prioriza tarefas e como esse conhecimento explica o funcionamento de `nextTick()`, do Scheduler e da reatividade do Vue.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Entender a diferença entre Microtasks e Macrotasks.
* Explicar por que Promises possuem prioridade sobre `setTimeout`.
* Prever a ordem de execução de códigos assíncronos.
* Compreender a base do `nextTick()`.
* Entender por que o Vue utiliza Microtasks para atualizar a interface.

---

# Pré-requisitos

* Capítulo 01 — Como o JavaScript e o Navegador Funcionam
* Capítulo 02 — Event Loop

---

# Revisando o Event Loop

No capítulo anterior vimos que o Event Loop verifica continuamente:

```text
A Call Stack está vazia?
```

Se estiver:

↓

Executa a próxima tarefa.

Mas...

Existe um detalhe que omitimos.

**Não existe apenas uma fila.**

Existem várias.

E algumas possuem prioridade sobre outras.

---

# O problema

Observe o código abaixo.

```javascript
console.log("A")

setTimeout(() => {

    console.log("B")

},0)

Promise.resolve().then(() => {

    console.log("C")

})

console.log("D")
```

O que será impresso?

Muita gente responde:

```text
A

D

B

C
```

Está errado.

O resultado correto é:

```text
A

D

C

B
```

Por quê?

---

# Existem duas filas principais

O navegador mantém duas filas importantes.

```text
Call Stack

↓

Microtask Queue

↓

Macrotask Queue
```

As Microtasks possuem prioridade.

Sempre.

---

# Macrotasks

Também chamadas de:

* Tasks
* Macro Queue

Exemplos:

```javascript
setTimeout()

setInterval()

setImmediate() (Node.js)

MessageChannel

I/O
```

Essas tarefas esperam sua vez.

---

# Microtasks

São tarefas pequenas.

Executadas imediatamente após a Stack ficar vazia.

Exemplos:

```javascript
Promise.then()

Promise.catch()

Promise.finally()

queueMicrotask()

MutationObserver
```

O Vue também utiliza esse mecanismo.

---

# A prioridade

Imagine:

```text
Call Stack vazia
```

O Event Loop faz:

```text
Existe Microtask?

↓

SIM

↓

Executa TODAS

↓

Existe Macrotask?

↓

Executa UMA

↓

Repete
```

Isso é extremamente importante.

---

# Fluxo visual

```text
Call Stack

↓

Microtask Queue

↓

Microtask

↓

Microtask

↓

Microtask

↓

Macrotask Queue

↓

setTimeout

↓

setInterval
```

As Microtasks sempre esvaziam primeiro.

---

# Exemplo

```javascript
console.log(1)

setTimeout(()=>{

    console.log(2)

},0)

Promise.resolve().then(()=>{

    console.log(3)

})

console.log(4)
```

Vamos acompanhar.

---

Primeiro:

```text
console.log(1)
```

Resultado:

```text
1
```

---

Depois:

```javascript
setTimeout(...)
```

Vai para:

Macrotask Queue.

---

Depois:

```javascript
Promise.resolve()
```

Vai para:

Microtask Queue.

---

Depois:

```javascript
console.log(4)
```

Resultado:

```text
1

4
```

Agora a Stack ficou vazia.

O Event Loop verifica.

Existe Microtask?

Sim.

Executa.

Resultado:

```text
1

4

3
```

Agora verifica novamente.

Não existem mais Microtasks.

Existe Macrotask?

Sim.

Executa.

Resultado final.

```text
1

4

3

2
```

---

# Outro exemplo

```javascript
setTimeout(()=>{

console.log("Timer")

},0)

Promise.resolve().then(()=>{

console.log("Promise")

})

queueMicrotask(()=>{

console.log("Microtask")

})
```

Resultado.

```text
Promise

Microtask

Timer
```

Observe que tanto `Promise` quanto `queueMicrotask` executam antes do Timer.

---

# A regra de ouro

O Event Loop nunca executa uma Macrotask enquanto existir uma Microtask pendente.

Nunca.

---

# Então `setTimeout(0)` significa zero?

Não.

Significa:

> Coloque essa função na fila de Macrotasks.

Ela será executada quando:

* a Stack estiver vazia;
* todas as Microtasks tiverem terminado.

---

# Como isso afeta o Vue?

Imagine:

```javascript
contador.value++

contador.value++

contador.value++

contador.value++
```

Sem um Scheduler.

O Vue faria:

```text
Render

Render

Render

Render
```

Seria extremamente caro.

---

# O que o Vue faz?

Em vez disso:

```text
Mudança

↓

Mudança

↓

Mudança

↓

Mudança

↓

Uma única atualização
```

Ele agenda essa atualização utilizando Microtasks.

Assim o DOM é atualizado apenas uma vez.

---

# É por isso que nextTick existe

Imagine:

```javascript
contador.value++
```

Logo depois:

```javascript
console.log(document.body.innerHTML)
```

Você pode receber o DOM antigo.

Porque o Vue ainda não executou a atualização.

Quando usamos:

```javascript
await nextTick()
```

Estamos dizendo:

> Espere o Vue terminar todas as Microtasks antes de continuar.

Mais adiante estudaremos sua implementação.

---

# Como o Vue implementa isso?

De forma simplificada.

Ele agenda uma atualização.

```javascript
Promise.resolve().then(flushJobs)
```

Ou equivalente.

Assim a atualização acontece:

* rapidamente;
* apenas uma vez;
* após todas as alterações do estado.

Essa decisão melhora significativamente a performance.

---

# Comparação

## Macrotask

Mais lenta.

Menor prioridade.

Exemplos:

```javascript
setTimeout

setInterval
```

---

## Microtask

Mais rápida.

Maior prioridade.

Exemplos:

```javascript
Promise

queueMicrotask

MutationObserver
```

---

# Armadilhas

## Erro 1

Achar que:

```javascript
setTimeout(fn,0)
```

Executa imediatamente.

Não executa.

---

## Erro 2

Achar que Promise é "mais rápida".

Não.

Ela apenas possui prioridade maior.

---

## Erro 3

Esquecer que milhares de Microtasks podem bloquear o navegador.

Exemplo.

```javascript
function loop(){

Promise.resolve().then(loop)

}

loop()
```

Isso pode impedir o Event Loop de processar outras tarefas.

---

# Performance

Microtasks são excelentes.

Mas não são gratuitas.

Se você criar milhares delas.

O navegador continuará ocupado.

O Vue possui mecanismos para evitar esse tipo de problema.

---

# Relação com próximos capítulos

Agora você já entende:

* Event Loop;
* Microtasks;
* Macrotasks.

Nos próximos capítulos veremos:

* `nextTick()`;
* Scheduler do Vue;
* Batch Updates;
* Render Pipeline.

Todos dependem diretamente destes conceitos.

---

# Exercícios

## Exercício 1

Explique a diferença entre:

* Microtask
* Macrotask

---

## Exercício 2

Qual será a saída?

```javascript
console.log("A")

setTimeout(()=>{

console.log("B")

})

Promise.resolve().then(()=>{

console.log("C")

})

console.log("D")
```

Explique passo a passo.

---

## Exercício 3

Pesquise outras APIs que utilizam Microtasks.

---

## Exercício 4

Explique por que o Vue prefere Microtasks para atualizar o DOM.

---

# Desafio

Escreva cinco exemplos diferentes utilizando:

* Promise
* setTimeout
* queueMicrotask

E tente prever a saída antes de executar.

---

# Projeto do capítulo

Crie um visualizador do Event Loop.

A interface deve possuir:

* Call Stack
* Microtask Queue
* Macrotask Queue

E permitir visualizar a ordem em que cada tarefa é executada.

---

# Checklist

* [ ] Sei diferenciar Microtasks e Macrotasks.
* [ ] Entendi a prioridade das filas.
* [ ] Sei prever a ordem de execução de Promises e Timers.
* [ ] Entendi por que `nextTick()` utiliza Microtasks.
* [ ] Estou preparado para estudar a Call Stack em profundidade e, posteriormente, o Scheduler do Vue.

---

# Próximo capítulo

## **Capítulo 04 — A Call Stack em Profundidade**

Você entenderá como funções são empilhadas, desempilhadas, como ocorre a recursão, stack overflow, execução de closures e por que compreender a Call Stack é essencial para entender a reatividade e o ciclo de renderização do Vue.
