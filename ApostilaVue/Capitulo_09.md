# Capítulo 09 — Construindo o Sistema de Reatividade do Zero (Parte 2): `ReactiveEffect`, Cleanup e Dependency Tracking

> **Objetivo:** evoluir o sistema reativo criado no capítulo anterior para ficar muito mais próximo da implementação real do Vue 3. Neste capítulo você entenderá por que um simples `track()` e `trigger()` não são suficientes e como o Vue resolve problemas complexos de gerenciamento de dependências.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Entender por que o Vue utiliza `ReactiveEffect`.
* Explicar o conceito de Effects ativos.
* Implementar Cleanup de dependências.
* Compreender o problema das dependências antigas.
* Entender Effects aninhados.
* Aproximar sua implementação da arquitetura real do Vue.

---

# Pré-requisitos

* Capítulos 01 ao 08.

---

# Revisando nossa implementação

Terminamos o capítulo anterior com algo parecido com isto.

```javascript
let activeEffect = null

function effect(fn){

    activeEffect = fn

    fn()

    activeEffect = null

}
```

Funciona.

Mas possui vários problemas.

---

# O primeiro problema

Imagine.

```javascript
const state = reactive({

    contador:0

})

effect(()=>{

    console.log(state.contador)

})
```

Depois.

```javascript
effect(()=>{

    console.log(state.contador)

})
```

Agora existem dois Effects.

Tudo certo.

Mas...

Como remover um deles?

Não conseguimos.

---

# Segundo problema

Imagine.

```javascript
let mostrarNome = true

effect(()=>{

    if(mostrarNome){

        console.log(state.nome)

    }

})
```

Inicialmente.

```text
Dependências

↓

nome
```

Depois.

```javascript
mostrarNome = false
```

Na próxima execução.

```javascript
effect(()=>{

    if(false){

        console.log(state.nome)

    }

})
```

A propriedade `nome` nem foi utilizada.

Mas o Effect continua registrado nela.

Temos uma dependência "fantasma".

---

# O problema visualmente

Primeira execução.

```text
nome

↓

Effect
```

Segunda execução.

```text
nome

↓

Effect
```

Mesmo que o código não utilize mais `nome`.

Isso é um erro.

---

# O que o Vue faz?

Antes de registrar novas dependências.

Ele remove todas as antigas.

Esse processo recebe o nome de:

```text
cleanup
```

---

# Antes de entender o Cleanup

Precisamos melhorar nosso Effect.

Em vez de armazenar apenas uma função.

Vamos criar uma classe.

---

# ReactiveEffect

O Vue utiliza uma classe chamada:

```text
ReactiveEffect
```

Uma versão simplificada.

```javascript
class ReactiveEffect{

    constructor(fn){

        this.fn = fn

    }

    run(){

        activeEffect = this

        this.fn()

        activeEffect = null

    }

}
```

Agora.

```javascript
const effect = new ReactiveEffect(()=>{

    console.log("Vue")

})

effect.run()
```

---

# Por que usar uma classe?

Porque agora podemos armazenar informações.

Por exemplo.

```javascript
class ReactiveEffect{

    constructor(fn){

        this.fn = fn

        this.deps = []

    }

}
```

Observe.

Cada Effect conhece todas as dependências que possui.

---

# O que é deps?

Imagine.

```javascript
effect(()=>{

    console.log(state.nome)

    console.log(state.idade)

})
```

Esse Effect depende de.

```text
nome

idade
```

Então.

```javascript
effect.deps
```

Pode conter.

```text
[
 Set(nome),

 Set(idade)
]
```

Isso será essencial.

---

# Melhorando track()

Antes.

```javascript
effects.add(activeEffect)
```

Agora.

Depois de adicionar.

Também fazemos.

```javascript
activeEffect.deps.push(effects)
```

Assim.

O Effect sabe exatamente em quais Sets está registrado.

---

# Visualizando

```text
Effect

↓

deps

↓

Set(nome)

↓

Set(idade)
```

Ao mesmo tempo.

```text
Set(nome)

↓

Effect
```

Perceba.

A relação é bidirecional.

---

# Agora podemos fazer Cleanup

Implementação.

```javascript
function cleanup(effect){

    effect.deps.forEach(dep=>{

        dep.delete(effect)

    })

    effect.deps.length = 0

}
```

O que acontece?

Percorremos todos os Sets.

Removemos o Effect.

Depois limpamos a lista.

---

# Fluxo

Antes.

```text
Effect

↓

Set(nome)

↓

Set(idade)

↓

Set(email)
```

Após.

```text
Effect

↓

deps vazio
```

Todos os Sets também deixam de conter esse Effect.

---

# Integrando ao run()

Agora.

```javascript
run(){

    cleanup(this)

    activeEffect = this

    this.fn()

    activeEffect = null

}
```

Sempre que um Effect executa.

Suas dependências são reconstruídas.

---

# Exemplo

```javascript
let mostrar = true

effect(()=>{

    if(mostrar){

        console.log(state.nome)

    }

})
```

Primeira execução.

```text
nome

↓

Effect
```

Depois.

```javascript
mostrar = false
```

Nova execução.

Cleanup.

↓

Remove.

↓

Executa novamente.

↓

Nenhuma leitura.

Resultado.

Nenhuma dependência registrada.

Exatamente como deveria ser.

---

# Outro problema

Imagine.

```javascript
effect(()=>{

    console.log(state.nome)

    console.log(state.nome)

})
```

Nosso código faria.

```javascript
deps.push(set)
```

Duas vezes.

Teríamos.

```text
deps

↓

Set(nome)

↓

Set(nome)
```

Duplicado.

---

# Como o Vue resolve?

Antes de adicionar.

Ele verifica.

```javascript
if(

    !effects.has(activeEffect)

){

    effects.add(activeEffect)

}
```

Somente depois.

```javascript
activeEffect.deps.push(effects)
```

Isso evita registros duplicados.

---

# Effects aninhados

Agora um problema mais difícil.

Imagine.

```javascript
effect(()=>{

    console.log(state.nome)

    effect(()=>{

        console.log(state.idade)

    })

})
```

Temos dois Effects ativos.

Quem é o activeEffect?

---

# Nosso código quebra

Porque fazemos.

```javascript
activeEffect = effectA
```

Depois.

```javascript
activeEffect = effectB
```

Quando o segundo termina.

O primeiro desapareceu.

---

# Como o Vue resolve?

Utilizando uma pilha.

```text
Effect Stack
```

---

# Visualizando

Inicialmente.

```text
[]
```

Executa.

```text
[effectA]
```

Depois.

```text
[effectA

effectB]
```

Quando B termina.

```text
[effectA]
```

Agora o primeiro continua ativo.

---

# Implementação simplificada

```javascript
const effectStack = []
```

No run.

```javascript
effectStack.push(this)

activeEffect = this
```

Ao terminar.

```javascript
effectStack.pop()

activeEffect =

effectStack[
effectStack.length-1
]
```

Agora.

Effects aninhados funcionam perfeitamente.

---

# Outro detalhe importante

Imagine.

```javascript
effect(()=>{

    state.contador++

})
```

O Effect modifica a própria dependência.

Isso pode gerar.

```text
trigger()

↓

effect()

↓

trigger()

↓

effect()

↓

trigger()

↓

∞
```

Loop infinito.

---

# Como o Vue evita isso?

Durante o trigger.

Ele ignora.

```javascript
if(

effect !== activeEffect

)
```

Assim.

Um Effect não dispara ele mesmo.

---

# O Fluxo completo

```text
run()

↓

cleanup()

↓

push()

↓

activeEffect

↓

Proxy.get()

↓

track()

↓

WeakMap

↓

Map

↓

Set

↓

Proxy.set()

↓

trigger()

↓

run()
```

Esse fluxo é praticamente igual ao runtime do Vue.

---

# Organização interna

Cada Effect possui.

```text
ReactiveEffect

↓

fn

↓

deps

↓

active

↓

scheduler

↓

parent

↓

onStop
```

Ainda estudaremos vários desses campos.

---

# Por que isso é importante?

Porque:

* computed é um ReactiveEffect.
* watch é um ReactiveEffect.
* watchEffect é um ReactiveEffect.
* render de componente é um ReactiveEffect.

Tudo no Vue deriva dessa classe.

---

# Comparação

Nossa implementação.

```text
Function
```

Vue.

```text
ReactiveEffect

↓

run()

↓

stop()

↓

scheduler

↓

cleanup

↓

parent

↓

deps
```

Muito mais poderosa.

---

# Performance

O Cleanup parece caro.

Mas evita:

* milhares de dependências inválidas;
* Memory Leaks;
* renderizações desnecessárias.

No longo prazo.

Ele economiza muito processamento.

---

# Resumo

Neste capítulo aprendemos que:

* um Effect precisa conhecer suas dependências;
* Cleanup remove registros antigos;
* Effects utilizam uma pilha para suportar chamadas aninhadas;
* o Vue evita loops infinitos durante `trigger()`;
* `ReactiveEffect` é a base de praticamente todo o sistema reativo.

---

# Exercícios

## Exercício 1

Implemente a classe `ReactiveEffect` com o método `run()`.

---

## Exercício 2

Implemente a função `cleanup()`.

---

## Exercício 3

Explique por que um Effect precisa conhecer seus próprios Sets de dependência.

---

## Exercício 4

Implemente suporte para Effects aninhados utilizando uma pilha.

---

# Desafio

Evolua sua biblioteca **MiniVue Reactive** para suportar:

* `ReactiveEffect`;
* `cleanup`;
* `effectStack`;
* prevenção de registros duplicados;
* prevenção de loops infinitos simples.

---

# Projeto do capítulo

Refatore completamente a implementação criada no Capítulo 08 para que sua arquitetura fique semelhante ao runtime oficial do Vue.

Ao final do projeto sua biblioteca deverá possuir:

* `ReactiveEffect`;
* `effect()`;
* `cleanup()`;
* `track()`;
* `trigger()`;
* `effectStack`;
* `WeakMap → Map → Set`.

---

# Checklist

* [ ] Entendi por que uma função simples não é suficiente.
* [ ] Sei explicar `ReactiveEffect`.
* [ ] Entendi `cleanup()`.
* [ ] Sei explicar por que o Vue utiliza `effectStack`.
* [ ] Entendi como o Vue evita dependências antigas.
* [ ] Minha implementação já se aproxima da arquitetura oficial do Vue.

---

# Próximo capítulo

## **Capítulo 10 — Scheduler, Job Queue e Batch Updates: Como o Vue Atualiza o DOM Apenas Uma Vez**

Neste capítulo veremos um dos mecanismos mais elegantes do Vue 3: o **Scheduler**. Você entenderá por que várias alterações consecutivas no estado geram apenas uma renderização, como funciona a Job Queue, o papel do `nextTick()`, o uso de Microtasks com `Promise.resolve()` e como o Vue evita renderizações redundantes mesmo em aplicações extremamente complexas. Esse é o componente que conecta todo o sistema reativo ao processo de renderização do framework.
