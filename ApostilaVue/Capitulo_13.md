# Capítulo 13 — `computed()`: Implementando o `computed` do Zero

> **Objetivo:** implementar uma versão extremamente próxima do `computed()` do Vue 3. Ao final deste capítulo você entenderá por que um `computed` não é apenas uma função, como funciona o cache interno, o mecanismo de *Lazy Evaluation*, o *Dirty Checking*, o Scheduler e por que um `computed` só é recalculado quando realmente necessário.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Explicar como `computed()` funciona internamente.
* Implementar um `ComputedRefImpl`.
* Entender Lazy Evaluation.
* Compreender Dirty Checking.
* Implementar cache de valores.
* Explicar Computed Chains.
* Implementar Writable Computed.
* Entender como o Vue evita recálculos desnecessários.

---

# Pré-requisitos

* Capítulos 01 ao 12.

---

# O problema

Imagine.

```javascript
const total = computed(() => {

    return produtos.value.length

})
```

Pergunta.

Quando essa função executa?

Logo na criação?

Quando alguém acessa?

Sempre que qualquer estado muda?

---

# Primeira tentativa

Poderíamos implementar assim.

```javascript
function computed(getter){

    return {

        get value(){

            return getter()

        }

    }

}
```

Agora.

```javascript
total.value
```

Executa.

```javascript
getter()
```

Sempre.

---

# O problema

Imagine.

```javascript
const total = computed(() => {

    console.log("Calculando")

    return produtos.value.length

})
```

Depois.

```javascript
console.log(total.value)

console.log(total.value)

console.log(total.value)
```

Resultado.

```text
Calculando

Calculando

Calculando
```

O cálculo acontece três vezes.

---

# O Vue nunca faz isso

Observe.

```javascript
console.log(total.value)

console.log(total.value)

console.log(total.value)
```

No Vue.

Resultado.

```text
Calculando
```

Apenas uma vez.

---

# Como?

Utilizando cache.

---

# Visualizando

Primeiro acesso.

```text
computed

↓

getter()

↓

Cache
```

Segundo acesso.

```text
computed

↓

Cache
```

O getter nem é executado.

---

# Primeira implementação

Precisamos guardar.

```javascript
class ComputedRefImpl{

    constructor(getter){

        this._value = undefined

    }

}
```

---

# Getter

```javascript
get value(){

    return this._value

}
```

Ainda não funciona.

Precisamos saber.

O cache já foi calculado?

---

# Dirty Flag

O Vue utiliza uma variável.

```javascript
_dirty
```

Ela indica.

```text
true

↓

Cache inválido
```

Ou.

```text
false

↓

Cache válido
```

---

# Inicialmente

```javascript
class ComputedRefImpl{

    constructor(){

        this._dirty = true

    }

}
```

---

# Melhorando o Getter

```javascript
get value(){

    if(this._dirty){

        this._value = getter()

        this._dirty = false

    }

    return this._value

}
```

Agora.

Primeiro acesso.

↓

Calcula.

↓

Salva.

↓

Retorna.

Segundo acesso.

↓

Retorna cache.

---

# Testando

```javascript
const total = computed(() => {

    console.log("Calculando")

    return contador.value * 2

})
```

Depois.

```javascript
console.log(total.value)

console.log(total.value)

console.log(total.value)
```

Resultado.

```text
Calculando

2

2

2
```

Muito melhor.

---

# Ainda existe um problema

Imagine.

```javascript
contador.value++
```

Nosso cache continua.

```text
2
```

Mesmo que o resultado correto agora seja.

```text
4
```

---

# Precisamos invalidar o cache

Sempre que uma dependência mudar.

Fazemos.

```javascript
_dirty = true
```

Mas...

Como descobrir isso?

---

# ReactiveEffect

Lembra do capítulo anterior?

Vamos reutilizar.

```javascript
new ReactiveEffect(getter)
```

Agora.

O getter torna-se um Effect.

---

# O Scheduler

Quando uma dependência mudar.

Não queremos recalcular imediatamente.

Queremos apenas.

```text
Marcar como sujo
```

---

# Implementação

```javascript
this.effect = new ReactiveEffect(

    getter,

    () => {

        this._dirty = true

    }

)
```

Perceba.

O Scheduler não recalcula.

Ele apenas invalida o cache.

---

# Fluxo

Mudança.

↓

Scheduler.

↓

_dirty = true.

↓

Nada mais.

---

# Quando recalcula?

Somente quando alguém faz.

```javascript
total.value
```

Então.

```text
_dirty?

↓

Sim

↓

getter()

↓

Atualiza cache
```

Isso chama-se.

```text
Lazy Evaluation
```

---

# O que significa Lazy?

Preguiçoso.

O Vue só trabalha quando realmente precisa.

---

# Fluxo completo

Primeira leitura.

```text
value

↓

_dirty

↓

true

↓

getter()

↓

Cache

↓

false
```

Nova leitura.

```text
value

↓

_dirty

↓

false

↓

Cache
```

Mudança.

```text
contador++

↓

Scheduler

↓

_dirty = true
```

Próxima leitura.

↓

Calcula novamente.

---

# Implementação completa

```javascript
get value(){

    if(this._dirty){

        this._dirty = false

        this._value =

            this.effect.run()

    }

    return this._value

}
```

Muito parecido com o Vue.

---

# Computed também é reativo

Imagine.

```javascript
const dobro = computed(() => {

    return contador.value * 2

})
```

Depois.

```javascript
effect(() => {

    console.log(

        dobro.value

    )

})
```

Agora.

Quem depende do Computed?

O Effect.

---

# Então...

O próprio Computed precisa possuir dependências.

Assim como um Ref.

---

# Estrutura

```text
Computed

↓

dep

↓

Set

↓

Effects
```

Perceba.

É igual ao Ref.

---

# Getter atualizado

```javascript
get value(){

    trackRefValue(this)

    ...
}
```

Agora.

Quem utiliza o Computed será registrado.

---

# Scheduler atualizado

Quando.

```javascript
contador.value++
```

O Scheduler faz.

```javascript
this._dirty = true

triggerRefValue(this)
```

Agora.

Todos que dependem do Computed serão atualizados.

---

# Computed Chains

Imagine.

```javascript
const dobro = computed(...)

const triplo = computed(() => {

    return dobro.value + contador.value

})
```

Temos.

```text
contador

↓

dobro

↓

triplo
```

Tudo continua funcionando.

Porque.

Computed também participa do sistema reativo.

---

# Writable Computed

Existe outra forma.

```javascript
const nome = computed({

    get(){

        return primeiro.value +

        ultimo.value

    },

    set(valor){

        ...

    }

})
```

Agora.

```javascript
nome.value = "Felipe Ariki"
```

Também funciona.

---

# Implementação

```javascript
constructor(

getter,

setter

){

    this.setter = setter

}
```

---

# Setter

```javascript
set value(valor){

    this.setter(valor)

}
```

---

# Readonly Computed

Quando fazemos.

```javascript
computed(() => {

})
```

Não existe setter.

Então.

```javascript
set value(){

    console.warn(

        "Readonly Computed"

    )

}
```

---

# Computed dentro de Computed

Imagine.

```javascript
const a = computed(...)

const b = computed(...)

const c = computed(...)
```

O Vue consegue montar toda a árvore de dependências automaticamente.

---

# Por que Computed é tão rápido?

Porque.

Enquanto nada muda.

Ele nunca recalcula.

Mesmo que você faça.

```javascript
for(

let i=0

i<100000

i++

){

    total.value

}
```

O getter executa apenas uma vez.

---

# Comparando com método

Método.

```javascript
function total(){

}
```

Executa.

Toda vez.

---

Computed.

```javascript
computed()
```

Executa.

Somente quando necessário.

---

# Erro comum

Fazer.

```javascript
computed(()=>{

    console.log("teste")

})
```

Esperando que execute imediatamente.

Não.

Enquanto ninguém acessar.

```javascript
.value
```

Nada acontece.

---

# Performance

Imagine.

```javascript
const listaFiltrada = computed(() => {

    return lista.value

        .filter(...)

        .sort(...)

})
```

Mesmo sendo uma operação pesada.

Enquanto `lista` não mudar.

Ela nunca será executada novamente.

---

# Estrutura final

```text
ComputedRefImpl

↓

_value

↓

_dirty

↓

dep

↓

ReactiveEffect

↓

Scheduler
```

Essa é praticamente a implementação oficial.

---

# Comparando com o Vue

Nossa implementação.

```text
Computed

↓

Getter

↓

Cache
```

Vue.

```text
ComputedRefImpl

↓

ReactiveEffect

↓

Scheduler

↓

Dirty Flag

↓

dep

↓

trackRefValue()

↓

triggerRefValue()
```

Muito próxima da implementação real.

---

# Resumo

Neste capítulo aprendemos que:

* `computed()` utiliza cache.
* O Scheduler não recalcula imediatamente.
* `_dirty` indica quando o cache é inválido.
* `computed()` também é reativo.
* Computeds podem depender de outros Computeds.
* O Vue utiliza Lazy Evaluation para maximizar desempenho.

---

# Exercícios

## Exercício 1

Implemente uma classe `ComputedRefImpl`.

---

## Exercício 2

Implemente o mecanismo de Dirty Checking.

---

## Exercício 3

Adicione cache ao seu `computed()`.

---

## Exercício 4

Implemente suporte para Writable Computed.

---

# Desafio

Atualize sua biblioteca **MiniVue Reactive** para suportar:

* `computed()`;
* `ComputedRefImpl`;
* cache;
* `_dirty`;
* Scheduler;
* Lazy Evaluation;
* Writable Computed;
* Computed Chains.

---

# Projeto do capítulo

Ao final deste capítulo sua biblioteca deverá oferecer uma implementação de `computed()` extremamente próxima da utilizada pelo Vue 3, incluindo:

* `ComputedRefImpl`;
* `ReactiveEffect`;
* Scheduler;
* cache de resultados;
* Dirty Flag;
* `trackRefValue()`;
* `triggerRefValue()`;
* suporte a `computed(getter)` e `computed({ get, set })`.

---

# Checklist

* [ ] Sei explicar por que `computed()` utiliza cache.
* [ ] Entendi o conceito de Lazy Evaluation.
* [ ] Sei implementar Dirty Checking.
* [ ] Entendi o papel do Scheduler em `computed()`.
* [ ] Sei a diferença entre um método e um `computed()`.
* [ ] Minha implementação já se aproxima da utilizada pelo Vue 3.

---

# Próximo capítulo

## **Capítulo 14 — `watch()` e `watchEffect()`: Implementando o Sistema de Observação do Vue**

Neste capítulo construiremos o sistema completo de observação do Vue. Você aprenderá como funcionam `watch()` e `watchEffect()`, quais são suas diferenças, como o Vue implementa *Deep Watch*, opções como `immediate`, `once` e `flush` (`pre`, `post`, `sync`), funções de limpeza (`onCleanup`/`onInvalidate`) e como todo esse mecanismo se integra ao Scheduler e ao ciclo de atualização dos componentes. Esse capítulo completa praticamente toda a camada de reatividade pública do Vue 3.
