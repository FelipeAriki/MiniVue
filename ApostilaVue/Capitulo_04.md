# Capítulo 04 — A Call Stack em Profundidade

> **Objetivo:** compreender como o JavaScript executa funções internamente, como a Call Stack organiza a execução do código e por que esse conceito é indispensável para entender a reatividade, o Scheduler e o Runtime do Vue.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Explicar como uma função é executada.
* Entender o funcionamento da Call Stack.
* Explicar o conceito de Execution Context.
* Compreender Stack Frames.
* Identificar Stack Overflow.
* Entender recursão.
* Relacionar a Call Stack com o Runtime do Vue.

---

# Pré-requisitos

* Capítulo 01 — Como o JavaScript e o Navegador Funcionam
* Capítulo 02 — Event Loop
* Capítulo 03 — Microtasks e Macrotasks

---

# O que é a Call Stack?

A **Call Stack** é uma estrutura de dados do tipo **pilha (Stack)** utilizada pela Engine JavaScript para controlar a execução das funções.

Ela responde perguntas como:

* Qual função está sendo executada agora?
* Quem chamou essa função?
* Para qual ponto do código devemos voltar quando ela terminar?

Ela funciona seguindo o princípio:

> **LIFO — Last In, First Out**

Ou seja:

A última função que entra é a primeira que sai.

---

# Como imaginar uma Stack

Imagine uma pilha de pratos.

```text
┌──────────────┐
│ função C()   │
├──────────────┤
│ função B()   │
├──────────────┤
│ função A()   │
├──────────────┤
│ Global       │
└──────────────┘
```

Você nunca remove o prato do meio.

Sempre remove o último.

É exatamente assim que a Call Stack funciona.

---

# Primeiro exemplo

```javascript
function saudacao() {
    console.log("Olá")
}

saudacao()
```

Fluxo:

```text
Global

↓

saudacao()

↓

console.log()

↓

console.log termina

↓

saudacao termina

↓

Global
```

---

# Visualizando a Stack

Antes da chamada:

```text
┌──────────────┐
│ Global       │
└──────────────┘
```

Após chamar:

```javascript
saudacao()
```

```text
┌──────────────┐
│ saudacao()   │
├──────────────┤
│ Global       │
└──────────────┘
```

Dentro dela:

```javascript
console.log()
```

```text
┌─────────────────┐
│ console.log()   │
├─────────────────┤
│ saudacao()      │
├─────────────────┤
│ Global          │
└─────────────────┘
```

Depois que `console.log` termina:

```text
┌──────────────┐
│ saudacao()   │
├──────────────┤
│ Global       │
└──────────────┘
```

Depois:

```text
┌──────────────┐
│ Global       │
└──────────────┘
```

---

# Múltiplas chamadas

```javascript
function c() {
    console.log("C")
}

function b() {
    c()
}

function a() {
    b()
}

a()
```

A Stack evolui assim:

```text
Global
```

↓

```text
Global

↓

a()
```

↓

```text
Global

↓

a()

↓

b()
```

↓

```text
Global

↓

a()

↓

b()

↓

c()
```

↓

```text
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

Depois todas começam a sair.

---

# Stack Frames

Cada função adiciona um **Stack Frame**.

Um Stack Frame guarda:

* parâmetros;
* variáveis locais;
* endereço para retorno;
* contexto da função.

Exemplo:

```javascript
function soma(a, b) {

    const resultado = a + b

    return resultado

}
```

Durante sua execução a Engine guarda:

```text
Stack Frame

a = 10

b = 20

resultado = 30

return address
```

Quando a função termina.

Todo esse Frame é removido.

---

# Execution Context

Sempre que uma função é chamada.

A Engine cria um novo contexto.

Ele possui:

* variáveis locais;
* parâmetros;
* escopo léxico;
* referência ao contexto externo;
* valor de `this`.

Cada chamada possui seu próprio contexto.

Mesmo utilizando a mesma função.

---

# Exemplo

```javascript
function imprimir(numero){

    console.log(numero)

}

imprimir(10)

imprimir(20)
```

São criados dois Execution Contexts diferentes.

Um para cada chamada.

---

# Variáveis locais

Observe:

```javascript
function teste(){

    let nome = "Vue"

}
```

A variável `nome` existe apenas enquanto a função está na Stack.

Assim que ela termina.

Essa variável deixa de existir.

---

# O que acontece com objetos?

```javascript
function criar(){

    return {

        nome: "Vue"

    }

}
```

O objeto não fica na Stack.

Ele é armazenado na Heap.

Na Stack fica apenas a referência para ele.

Mais adiante estudaremos Heap e Garbage Collector.

---

# Recursão

Recursão é quando uma função chama ela mesma.

Exemplo.

```javascript
function contar(numero){

    if(numero === 0){

        return

    }

    contar(numero - 1)

}
```

Cada chamada cria um novo Stack Frame.

---

# Como fica a Stack

```text
contar(5)

↓

contar(4)

↓

contar(3)

↓

contar(2)

↓

contar(1)

↓

contar(0)
```

Depois.

Todas começam a retornar.

---

# Stack Overflow

Imagine:

```javascript
function infinito(){

    infinito()

}

infinito()
```

A Stack cresce.

E cresce.

E cresce.

Até atingir o limite.

Resultado.

```text
RangeError:

Maximum call stack size exceeded
```

Esse erro é conhecido como **Stack Overflow**.

---

# Por que existe limite?

A memória da Stack é limitada.

Se não existisse limite.

Uma função recursiva infinita poderia consumir toda a memória do computador.

---

# O Event Loop interrompe uma Stack?

Não.

Nunca.

Imagine:

```javascript
while(true){

}
```

Mesmo existindo:

* Promises
* Timers
* Eventos

Nada será executado.

Porque a Stack nunca ficou vazia.

---

# Relação com o Vue

Imagine:

```javascript
contador.value++
```

Quando isso acontece.

O Vue:

* detecta mudança;
* agenda atualização.

Mas essa atualização NÃO interrompe a função atual.

Ela será executada apenas quando a Stack terminar.

Esse comportamento depende diretamente do Event Loop.

---

# Chamadas dentro do Vue

Imagine um componente.

```text
render()

↓

patch()

↓

processElement()

↓

mountElement()

↓

createElement()
```

Cada chamada adiciona um novo Stack Frame.

Durante um render complexo.

Centenas de funções entram e saem da Stack.

---

# Debugando a Stack

Abra o DevTools.

Na aba **Sources**.

Adicione um breakpoint.

Quando a execução parar.

Você verá algo parecido:

```text
renderComponent

↓

patch

↓

processElement

↓

mountChildren

↓

mountElement
```

Isso é a Call Stack em tempo real.

Saber interpretá-la é uma habilidade essencial para depuração.

---

# Boas práticas

* Evite recursões sem condição de parada.
* Prefira funções pequenas.
* Não bloqueie a Stack com loops gigantes.
* Utilize operações assíncronas quando apropriado.

---

# Anti-patterns

### Recursão infinita

```javascript
function erro(){

    erro()

}
```

---

### Loops bloqueantes

```javascript
while(true){

}
```

---

### Processamento pesado na Thread Principal

```javascript
for(let i = 0; i < 10000000000; i++){

}
```

Isso bloqueia:

* Vue
* Renderização
* Eventos
* Timers
* Promises

---

# Performance

A Stack é extremamente rápida.

O problema não é utilizá-la.

O problema é mantê-la ocupada por muito tempo.

Quanto mais tempo a Thread Principal estiver ocupada:

* menos responsiva fica a aplicação;
* mais travamentos acontecem;
* pior será a experiência do usuário.

---

# Relação com os próximos capítulos

Agora você entende:

* como funções são executadas;
* como a Engine organiza chamadas;
* como o Event Loop depende da Stack.

Nos próximos capítulos veremos:

* Heap;
* Garbage Collector;
* Memory Management.

Esses conceitos serão fundamentais para compreender reatividade, proxies e vazamentos de memória em aplicações Vue.

---

# Exercícios

## Exercício 1

Desenhe a Stack para:

```javascript
function x(){

    y()

}

function y(){

    z()

}

function z(){

    console.log("Fim")

}

x()
```

---

## Exercício 2

Explique a diferença entre:

* Stack
* Stack Frame
* Execution Context

---

## Exercício 3

Pesquise qual é aproximadamente o limite de chamadas recursivas na Engine V8.

---

## Exercício 4

Utilize o Chrome DevTools para visualizar a Call Stack durante a execução de uma função.

---

# Desafio

Implemente uma função recursiva que percorra uma estrutura em árvore (por exemplo, uma árvore de categorias) e desenhe, no papel ou em um editor, como a Call Stack evolui durante a execução.

---

# Projeto do capítulo

Crie uma pequena aplicação para visualizar a Stack em tempo real.

Ela deve permitir:

* chamar funções encadeadas;
* visualizar entradas e saídas da Stack;
* simular uma recursão;
* demonstrar um Stack Overflow controlado.

---

# Checklist

* [ ] Sei explicar o que é Call Stack.
* [ ] Entendi Stack Frames.
* [ ] Sei o que é Execution Context.
* [ ] Consigo explicar recursão.
* [ ] Entendi Stack Overflow.
* [ ] Consigo relacionar a Stack com o Runtime do Vue.

---

# Próximo capítulo

## **Capítulo 05 — Heap, Garbage Collector e Gerenciamento de Memória**

Neste capítulo você entenderá onde objetos realmente são armazenados, como a memória é gerenciada pelo JavaScript, como funciona o Garbage Collector e por que vazamentos de memória acontecem em aplicações Vue. Esse conhecimento será essencial quando estudarmos `watch`, `effectScope`, `onUnmounted`, `KeepAlive` e otimização de aplicações de grande porte.
