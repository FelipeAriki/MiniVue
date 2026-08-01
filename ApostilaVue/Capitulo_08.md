Excelente! A partir daqui começa a parte mais importante do livro. Os próximos capítulos vão praticamente reconstruir o runtime reativo do Vue 3. Este capítulo será bem mais extenso que os anteriores porque ele explica o coração do framework.

---

# Capítulo 08 — Construindo o Sistema de Reatividade do Zero (Parte 1): `track()` e `trigger()`

> **Objetivo:** implementar do zero o mecanismo que torna o Vue reativo. Ao final deste capítulo você entenderá exatamente como o Vue sabe quais componentes, `computed` e `watchers` precisam ser atualizados quando um estado muda.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Entender o conceito de dependência reativa.
* Explicar o papel de `track()`.
* Explicar o papel de `trigger()`.
* Entender o conceito de Effect.
* Construir um sistema reativo simplificado.
* Compreender a arquitetura interna do Vue.

---

# Pré-requisitos

* Capítulos 01 ao 07.

---

# O problema

Imagine um componente.

```javascript
const state = reactive({

    contador: 0

})
```

Depois.

```javascript
console.log(state.contador)
```

Mais tarde.

```javascript
state.contador++
```

Como o Vue sabe que precisa atualizar exatamente quem utilizava `contador`?

Essa é a pergunta que vamos responder.

---

# O conceito de Dependência

Sempre que uma propriedade é lida.

Existe alguém interessado nela.

Por exemplo.

```javascript
console.log(state.contador)
```

Nesse momento.

O Vue pensa.

> "Alguém utilizou `contador`. Vou lembrar disso."

Esse processo chama-se:

```text
track()
```

---

# Quando uma propriedade muda

Depois.

```javascript
state.contador++
```

O Vue pensa.

> "Quem estava utilizando contador?"

↓

Atualiza todos.

Esse processo chama-se:

```text
trigger()
```

---

# Visualizando

```text
Leitura

↓

track()

↓

Guardar dependência

↓

Mudança

↓

trigger()

↓

Executar efeitos
```

Todo o sistema reativo do Vue gira em torno dessas duas funções.

---

# O que é um Effect?

Um Effect é uma função que depende de dados reativos.

Por exemplo.

```javascript
effect(() => {

    console.log(state.contador)

})
```

Essa função depende de:

```text
contador
```

Sempre que ele mudar.

Essa função precisa executar novamente.

---

# Nosso primeiro Effect

Vamos criar uma versão extremamente simples.

```javascript
let activeEffect = null

function effect(fn){

    activeEffect = fn

    fn()

    activeEffect = null

}
```

Agora podemos escrever.

```javascript
effect(() => {

    console.log("Executando")

})
```

Resultado.

```text
Executando
```

Até aqui.

Nada de especial.

---

# O que é activeEffect?

Imagine.

```javascript
effect(() => {

    console.log(state.contador)

})
```

Enquanto essa função executa.

O Vue guarda.

```text
activeEffect

↓

Função atual
```

Assim.

Quando o Proxy detectar uma leitura.

Ele saberá:

> "Quem está lendo?"

---

# Implementando track()

```javascript
function track(){

    console.log(

        "Registrando dependência"

    )

}
```

Ainda não iremos armazenar nada.

Primeiro.

Vamos entender o fluxo.

---

# Integrando ao Proxy

```javascript
const proxy = new Proxy(state,{

    get(target,key){

        track()

        return Reflect.get(

            target,

            key

        )

    }

})
```

Agora.

```javascript
effect(()=>{

    console.log(

        proxy.contador

    )

})
```

Resultado.

```text
Registrando dependência

0
```

---

# Mas onde guardar essa dependência?

Precisamos responder:

Qual propriedade?

↓

Quais Effects utilizam essa propriedade?

---

# Primeira estrutura

Podemos imaginar.

```text
contador

↓

Effect A

↓

Effect B

↓

Effect C
```

Quando `contador` mudar.

Executamos:

A

↓

B

↓

C

---

# Utilizando Map

```javascript
const deps = new Map()
```

Cada chave.

Representa uma propriedade.

```text
Map

contador

↓

Set
```

---

# Por que Set?

Porque o mesmo Effect pode acessar uma propriedade várias vezes.

Exemplo.

```javascript
effect(()=>{

    console.log(

        state.contador

    )

    console.log(

        state.contador

    )

})
```

Queremos armazenar apenas uma vez.

Set resolve isso.

---

# Melhorando track()

```javascript
const deps = new Map()

function track(key){

    if(!activeEffect){

        return

    }

    let effects = deps.get(key)

    if(!effects){

        effects = new Set()

        deps.set(

            key,

            effects

        )

    }

    effects.add(activeEffect)

}
```

Agora.

Estamos realmente registrando dependências.

---

# Visualizando

Depois deste código.

```javascript
effect(()=>{

    console.log(

        state.contador

    )

})
```

Nossa estrutura fica.

```text
Map

contador

↓

Set

↓

effect()
```

---

# Agora falta trigger()

Quando alguém fizer.

```javascript
state.contador++
```

Precisamos localizar.

```text
contador

↓

Set

↓

Executar todas
```

---

# Implementando trigger()

```javascript
function trigger(key){

    const effects = deps.get(key)

    if(!effects){

        return

    }

    effects.forEach(effect=>{

        effect()

    })

}
```

Agora.

Já conseguimos reagir às mudanças.

---

# Integrando ao Proxy

```javascript
set(target,key,value){

    Reflect.set(

        target,

        key,

        value

    )

    trigger(key)

    return true

}
```

---

# Testando

```javascript
const state = {

    contador:0

}

const reactiveState = new Proxy(state,{

    get(target,key){

        track(key)

        return Reflect.get(

            target,

            key

        )

    },

    set(target,key,value){

        Reflect.set(

            target,

            key,

            value

        )

        trigger(key)

        return true

    }

})
```

Depois.

```javascript
effect(()=>{

    console.log(

        reactiveState.contador

    )

})
```

Resultado.

```text
0
```

Agora.

```javascript
reactiveState.contador++
```

Resultado.

```text
1
```

Sem chamarmos nenhuma função manualmente.

---

# O que acabou de acontecer?

Fluxo completo.

```text
effect()

↓

activeEffect

↓

Leitura

↓

track()

↓

Guardar Effect

↓

Mudança

↓

trigger()

↓

Executar Effect
```

Esse é o coração do Vue.

---

# Um problema

Observe.

```javascript
effect(()=>{

    console.log(

        reactiveState.contador

    )

})
```

Depois.

```javascript
reactiveState.nome = "Felipe"
```

Nosso sistema executaria o Effect mesmo que ele não utilize `nome`.

Isso acontece porque ainda estamos simplificando.

Mais adiante resolveremos isso utilizando uma estrutura mais sofisticada.

---

# Como o Vue resolve?

O Vue utiliza.

```text
WeakMap

↓

Map

↓

Set
```

Estrutura.

```text
WeakMap

↓

Objeto

↓

Map

↓

Propriedade

↓

Set

↓

Effects
```

Visualmente.

```text
WeakMap

└── state

     ├── contador

     │      │

     │      ▼

     │   Set

     │

     └── nome

            │

            ▼

         Set
```

Isso permite milhares de objetos reativos.

---

# Por que WeakMap?

Porque.

Se o objeto deixar de existir.

O Garbage Collector poderá removê-lo.

Sem Memory Leak.

---

# Como o Vue chama isso?

Na implementação oficial.

Essa estrutura recebe o nome de:

```text
targetMap
```

Você verá exatamente esse nome no código-fonte do Vue.

---

# Arquitetura simplificada

```text
reactive()

↓

Proxy

↓

get()

↓

track()

↓

targetMap

↓

set()

↓

trigger()

↓

Executar Effects
```

Esses são os pilares da reatividade.

---

# Comparando com React

React.

```text
setState()

↓

Render
```

Vue.

```text
Proxy

↓

track()

↓

trigger()

↓

Atualizar apenas quem depende
```

Essa é uma das razões pelas quais o Vue consegue ser extremamente eficiente.

---

# Performance

Registrar dependências possui custo.

Mas.

Depois que elas estão registradas.

O Vue atualiza apenas quem realmente depende daquele estado.

Isso evita renderizações desnecessárias.

---

# Resumo

Neste capítulo construímos o primeiro núcleo do sistema reativo.

Aprendemos que:

* `effect()` representa código reativo.
* `track()` registra dependências.
* `trigger()` dispara atualizações.
* O Vue utiliza `WeakMap → Map → Set`.
* Toda a reatividade nasce dessa estrutura.

---

# Exercícios

## Exercício 1

Implemente `track()` utilizando `Map` e `Set`.

---

## Exercício 2

Implemente `trigger()` que execute todos os Effects registrados.

---

## Exercício 3

Explique por que `Set` é utilizado em vez de `Array`.

---

## Exercício 4

Desenhe a estrutura:

```text
WeakMap

↓

Map

↓

Set
```

e explique o papel de cada camada.

---

# Desafio

Implemente um sistema reativo capaz de registrar dependências para múltiplos objetos e múltiplas propriedades.

Exemplo.

```javascript
stateA.nome

stateA.idade

stateB.total

stateB.produtos
```

Cada propriedade deve possuir seu próprio conjunto de Effects.

---

# Projeto do capítulo

Inicie a construção da biblioteca **MiniVue Reactive**.

Nesta etapa implemente:

* `reactive()`
* `effect()`
* `track()`
* `trigger()`
* `targetMap`

Nos próximos capítulos essa biblioteca evoluirá até suportar:

* `ref`
* `computed`
* `watch`
* Scheduler
* Lazy Effects
* Dependency Cleanup

---

# Checklist

* [ ] Entendi o conceito de Effect.
* [ ] Sei explicar `track()`.
* [ ] Sei explicar `trigger()`.
* [ ] Entendi a estrutura `WeakMap → Map → Set`.
* [ ] Consigo implementar um sistema reativo simples.

---

# Próximo capítulo

## **Capítulo 09 — Construindo o Sistema de Reatividade do Zero (Parte 2): `ReactiveEffect`, Cleanup e Dependency Tracking**

Neste capítulo evoluiremos nossa implementação para ficar muito mais próxima do código-fonte oficial do Vue. Você aprenderá como funciona a classe `ReactiveEffect`, por que o Vue faz limpeza de dependências (`cleanup`), como evita registros duplicados e como lida corretamente com dependências que mudam entre execuções. A partir desse ponto estaremos estudando o runtime do Vue praticamente linha por linha.
