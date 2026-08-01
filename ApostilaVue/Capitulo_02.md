# Capítulo 02 — Event Loop: O Coração da Execução do JavaScript

> **Objetivo:** compreender exatamente como o JavaScript executa código e como o Event Loop funciona. Este é um dos capítulos mais importantes de todo o livro, pois praticamente toda a reatividade do Vue depende desses conceitos.

---

# Objetivos

Ao final deste capítulo você será capaz de:

* Explicar por que o JavaScript é single-thread.
* Entender Call Stack.
* Entender Queue.
* Entender Event Loop.
* Explicar como `setTimeout()` funciona.
* Explicar por que Promises executam antes de `setTimeout`.
* Compreender a base necessária para entender `nextTick()` e o Scheduler do Vue.

---

# Pré-requisitos

* Capítulo 01 – Como o JavaScript e o Navegador Funcionam

---

# O maior mito sobre JavaScript

Muita gente acredita que JavaScript executa várias coisas ao mesmo tempo.

Não executa.

Na realidade, JavaScript executa **uma instrução por vez**.

Exemplo:

```javascript
console.log("A")
console.log("B")
console.log("C")
```

Resultado:

```
A
B
C
```

Sempre nessa ordem.

Nunca:

```
A
C
B
```

---

# O que significa Single Thread?

JavaScript possui apenas **uma thread principal**.

Imagine um caixa de supermercado.

Existe apenas um caixa.

Enquanto uma pessoa está sendo atendida, ninguém mais é atendido.

O JavaScript funciona da mesma maneira.

```
Thread Principal

↓

Tarefa 1

↓

Tarefa 2

↓

Tarefa 3

↓

Tarefa 4
```

Uma tarefa precisa terminar antes da próxima começar.

---

# O problema

Imagine:

```javascript
console.log("Início")

while(true){

}

console.log("Fim")
```

O que acontece?

O navegador trava.

O JavaScript nunca chega na última linha.

Isso acontece porque existe apenas uma thread executando o código.

---

# A Call Stack

Todo código executado entra primeiro na **Call Stack**.

Imagine uma pilha.

```
Stack

↓

função3()

↓

função2()

↓

função1()

↓

Global
```

A última função que entra é sempre a primeira que sai.

LIFO.

Last In.

First Out.

---

# Exemplo

```javascript
function c(){
    console.log("C")
}

function b(){
    c()
}

function a(){
    b()
}

a()
```

Fluxo:

```
Global

↓

a()

↓

b()

↓

c()

↓

console.log()
```

Depois:

```
console.log sai

↓

c sai

↓

b sai

↓

a sai

↓

Global
```

---

# A Stack nunca executa duas funções ao mesmo tempo

Mesmo que existam centenas de funções.

Tudo passa pela mesma pilha.

Isso é importante.

Porque o Vue também utiliza essa pilha.

---

# O problema das operações lentas

Imagine:

```javascript
fetch("/usuarios")
```

O navegador precisa:

* enviar requisição
* esperar internet
* receber resposta

Isso pode demorar segundos.

O JavaScript deveria parar tudo?

Não.

---

# É aqui que entram as Web APIs

Quando fazemos:

```javascript
setTimeout(() => {

},1000)
```

Acontece:

```
JavaScript

↓

Web API

↓

Timer

↓

Callback
```

A Stack fica livre.

Pode continuar executando outras tarefas.

---

# Exemplo

```javascript
console.log("A")

setTimeout(() => {

    console.log("B")

},1000)

console.log("C")
```

Resultado:

```
A

C

B
```

Muita gente acha que o JavaScript "pulou" o código.

Não.

Ele apenas registrou o timer.

Quem ficou esperando foi o navegador.

---

# A Callback Queue

Quando o timer termina.

A função NÃO volta imediatamente.

Ela vai para uma fila.

```
Callback Queue

↓

console.log("B")
```

Ela fica esperando.

---

# Quem move essa função para a Stack?

O Event Loop.

---

# O Event Loop

O Event Loop faz apenas uma pergunta.

```
A Stack está vazia?
```

Se:

Não.

↓

Espera.

Se:

Sim.

↓

Pega a próxima tarefa da fila.

↓

Coloca na Stack.

↓

Executa.

Ele faz isso milhares de vezes por segundo.

---

# Fluxo completo

```
JavaScript

↓

Call Stack

↓

Web API

↓

Callback Queue

↓

Event Loop

↓

Call Stack novamente
```

Esse ciclo nunca para enquanto a página estiver aberta.

---

# Exemplo passo a passo

```javascript
console.log("A")

setTimeout(() => {

    console.log("B")

},0)

console.log("C")
```

Primeiro:

```
Stack

↓

console.log("A")
```

Resultado:

```
A
```

Depois:

```
setTimeout
```

Vai para Web API.

A Stack continua.

Depois:

```
console.log("C")
```

Resultado:

```
A

C
```

Somente depois.

Quando a Stack fica vazia.

O Event Loop move a callback.

Resultado final.

```
A

C

B
```

---

# Então por que 0ms não executa imediatamente?

Porque:

0ms significa:

> coloque essa callback na fila o mais rápido possível.

Mas.

Ela ainda precisa esperar:

* Stack vazia.
* Event Loop.

---

# O Event Loop nunca interrompe uma função

Imagine:

```javascript
for(let i=0;i<1000000000;i++){

}
```

Mesmo existindo callbacks esperando.

Elas NÃO executam.

O Event Loop respeita a Stack.

Primeiro termina.

Depois executa.

---

# Como isso influencia o Vue?

Imagine:

```javascript
contador.value++
```

O Vue não atualiza o DOM imediatamente.

Ele agenda uma atualização.

Isso evita renderizações desnecessárias.

Mais adiante veremos que o Vue possui um Scheduler próprio que trabalha em conjunto com o Event Loop.

---

# O Event Loop é suficiente?

Ainda não.

Existe outro detalhe.

Muito importante.

Existem duas filas.

* Macrotasks
* Microtasks

E elas possuem prioridades diferentes.

Esse será o assunto do próximo capítulo.

---

# Resumo

Aprendemos que:

* JavaScript possui apenas uma thread.
* Tudo passa pela Call Stack.
* Operações lentas são enviadas para Web APIs.
* O Event Loop monitora a Stack.
* Quando a Stack fica vazia, tarefas retornam para execução.

Esse conhecimento será utilizado em praticamente todos os capítulos seguintes.

---

# Exercícios

## Exercício 1

Explique com suas palavras:

* Call Stack
* Queue
* Event Loop

---

## Exercício 2

Qual será a saída?

```javascript
console.log(1)

setTimeout(()=>{

console.log(2)

},0)

console.log(3)
```

Explique o motivo.

---

## Exercício 3

Desenhe o fluxo de execução de:

```javascript
function a(){

b()

}

function b(){

c()

}

function c(){

console.log("Fim")

}

a()
```

Mostre a evolução da Stack.

---

## Exercício 4

Pesquise:

Por que JavaScript foi criado como Single Thread?

---

# Desafio

Implemente um pequeno simulador desenhando no papel:

* Stack
* Web API
* Queue
* Event Loop

Execute mentalmente diferentes códigos.

---

# Projeto do capítulo

Crie uma página contendo:

* Botão "Adicionar Timer"
* Botão "Executar Código"

Mostre visualmente:

* quando uma função entra na Stack;
* quando vai para Web API;
* quando entra na Queue;
* quando retorna para execução.

O objetivo é visualizar o funcionamento do Event Loop.

---

# Checklist

* [ ] Sei explicar o que é Call Stack.
* [ ] Entendi o conceito de Single Thread.
* [ ] Sei explicar Web APIs.
* [ ] Entendi Callback Queue.
* [ ] Sei explicar Event Loop.
* [ ] Consigo prever a ordem de execução de `setTimeout()`.

---

# Próximo capítulo

## **Capítulo 03 — Microtasks e Macrotasks**

Neste capítulo você entenderá por que `Promise.then()` executa antes de `setTimeout()`, como funcionam as filas de prioridade do JavaScript e por que isso é fundamental para compreender `nextTick()`, `watch()`, `watchEffect()` e o Scheduler interno do Vue.
