# Capítulo 24 — Reatividade Avançada: Implementando `ref()`, `reactive()`, `computed()` e `watch()` do Zero

> **Objetivo:** compreender profundamente a engine de reatividade do Vue 3 implementando suas principais APIs do zero. Ao final deste capítulo você entenderá praticamente todo o funcionamento do pacote `@vue/reactivity`, incluindo `track()`, `trigger()`, `ReactiveEffect`, `ref()`, `reactive()`, `computed()`, `watch()`, `watchEffect()` e o Scheduler.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Explicar como funciona toda a reatividade do Vue.
* Implementar `ReactiveEffect`.
* Implementar `track()` e `trigger()`.
* Criar `reactive()`.
* Criar `ref()`.
* Criar `computed()`.
* Criar `watchEffect()`.
* Criar `watch()`.
* Entender o Scheduler.

---

# Pré-requisitos

* Capítulos 01 ao 23.

---

# O coração do Vue

Existe um pacote inteiro responsável apenas pela reatividade.

```text
@vue/reactivity
```

Todo o restante do Vue depende dele.

Inclusive:

* Components
* Runtime
* Pinia
* VueUse
* Devtools

---

# Arquitetura

```text
Reactive Object

↓

track()

↓

ReactiveEffect

↓

trigger()

↓

Scheduler

↓

Render
```

Todo o sistema gira em torno desse fluxo.

---

# A ideia principal

Imagine.

```javascript
const usuario = reactive({

    nome: "Felipe"

})
```

Depois.

```javascript
effect(()=>{

    console.log(

        usuario.nome

    )

})
```

Resultado.

```text
Felipe
```

Agora.

```javascript
usuario.nome = "Lucas"
```

Automaticamente.

```text
Lucas
```

Mas...

Como isso acontece?

---

# O conceito

Sempre que uma propriedade é lida.

↓

Precisamos registrar.

Quem está usando.

---

Sempre que uma propriedade muda.

↓

Precisamos avisar.

Quem depende dela.

---

Esse é todo o segredo da reatividade.

---

# Dependências

Imagine.

```javascript
effect(()=>{

    console.log(

        usuario.nome

    )

})
```

Existe uma dependência.

```text
usuario.nome

↓

effect()
```

---

# Mudança

Quando fazemos.

```javascript
usuario.nome = "João"
```

Precisamos encontrar.

Todos os efeitos registrados.

E executá-los novamente.

---

# O banco de dependências

O Vue utiliza uma estrutura parecida com esta.

```javascript
WeakMap
```

Visualmente.

```text
WeakMap

↓

Target

↓

Map

↓

Key

↓

Set

↓

Effects
```

---

# Exemplo

```text
WeakMap

↓

usuario

↓

nome

↓

effect1

effect2

effect3
```

Cada propriedade possui seu próprio conjunto de efeitos.

---

# Estrutura real

Simplificando.

```javascript
const targetMap =

new WeakMap()
```

---

# Dentro dela

Cada objeto.

Possui.

```javascript
Map()
```

---

Cada propriedade.

Possui.

```javascript
Set()
```

---

Resultado.

```text
WeakMap

↓

Map

↓

Set
```

Essa estrutura é extremamente eficiente.

---

# O efeito ativo

Agora entra uma variável muito importante.

```javascript
let activeEffect
```

Ela representa.

O efeito atualmente sendo executado.

---

# Exemplo

```javascript
effect(()=>{

    console.log(

        usuario.nome

    )

})
```

Antes de executar.

```javascript
activeEffect = efeito
```

Depois.

Executamos.

```javascript
effect()
```

Quando terminar.

```javascript
activeEffect = undefined
```

---

# Implementando ReactiveEffect

Classe simplificada.

```javascript
class ReactiveEffect{

    constructor(fn){

        this.fn = fn

    }

    run(){

        activeEffect = this

        this.fn()

        activeEffect = undefined

    }

}
```

---

# Criando effect()

Agora.

```javascript
function effect(fn){

    const e =

        new ReactiveEffect(fn)

    e.run()

    return e

}
```

---

# Testando

```javascript
effect(()=>{

    console.log("Hello")

})
```

Resultado.

```text
Hello
```

Ainda não existe reatividade.

Mas temos a infraestrutura.

---

# Agora entra o Proxy

Lembra do capítulo de `reactive()`?

```javascript
const obj =

new Proxy(...)
```

Interceptamos.

```javascript
get()
```

E.

```javascript
set()
```

---

# get()

Quando alguém lê.

```javascript
usuario.nome
```

Executamos.

```javascript
track()
```

---

# set()

Quando alguém altera.

```javascript
usuario.nome = "João"
```

Executamos.

```javascript
trigger()
```

---

# track()

Objetivo.

Registrar.

Quem depende daquela propriedade.

---

# Assinatura

```javascript
track(

target,

key

)
```

---

# Primeiro passo

Encontramos.

```javascript
depsMap
```

Se não existir.

Criamos.

---

# Depois

Encontramos.

```javascript
dep
```

Que é um.

```javascript
Set()
```

---

# Finalmente

```javascript
dep.add(

activeEffect
)
```

Agora o efeito está registrado.

---

# trigger()

Agora.

Precisamos executar.

Todos os efeitos.

---

# Assinatura

```javascript
trigger(

target,

key
)
```

---

# Fluxo

Encontramos.

```text
WeakMap

↓

Map

↓

Set
```

Depois.

```javascript
for(

const effect

of dep

){

effect.run()

}
```

Pronto.

Temos reatividade.

---

# Fluxo completo

```javascript
usuario.nome
```

↓

```javascript
get()
```

↓

```javascript
track()
```

↓

Dependência registrada.

---

Depois.

```javascript
usuario.nome = "Maria"
```

↓

```javascript
set()
```

↓

```javascript
trigger()
```

↓

Todos os efeitos executam novamente.

---

# Implementando reactive()

```javascript
function reactive(target){

    return new Proxy(target,{

        get(obj,key){

            track(obj,key)

            return obj[key]

        },

        set(obj,key,value){

            obj[key]=value

            trigger(obj,key)

            return true

        }

    })

}
```

Agora temos um sistema funcional.

---

# O problema

Objetos funcionam.

Mas e isto?

```javascript
const contador = ref(0)
```

Não existe propriedade.

Precisamos criar uma.

---

# Implementando ref()

Na prática.

Um `ref` é apenas um objeto.

```javascript
{

value:0

}
```

---

# Classe

```javascript
class RefImpl{

    constructor(value){

        this._value = value

    }

    get value(){

    }

    set value(){

    }

}
```

---

# Getter

```javascript
get value(){

    track(this,"value")

    return this._value

}
```

---

# Setter

```javascript
set value(v){

    this._value = v

    trigger(this,"value")

}
```

---

# Resultado

```javascript
const contador = ref(0)

effect(()=>{

console.log(

contador.value

)

})
```

Mudando.

```javascript
contador.value++
```

Resultado.

```text
1
```

---

# Computed

Agora.

```javascript
const total = computed(()=>{

return preco.value *

quantidade.value

})
```

---

# O segredo

Computed também utiliza.

```text
ReactiveEffect
```

---

Mas.

Existe cache.

---

# Primeira execução

Calcula.

↓

Guarda.

↓

Retorna.

---

# Segunda execução

Nada mudou.

↓

Retorna o cache.

---

# Quando muda

```javascript
preco.value++
```

↓

Cache inválido.

↓

Calcula novamente.

---

# Estrutura

```javascript
class ComputedRef{

}
```

Internamente possui.

```javascript
ReactiveEffect
```

E um campo.

```javascript
dirty
```

---

# Dirty Flag

```text
true
```

↓

Precisa recalcular.

---

```text
false
```

↓

Pode reutilizar.

---

# watchEffect()

Implementação conceitual.

```javascript
watchEffect(()=>{

console.log(

contador.value

)

})
```

Na prática.

É praticamente.

```javascript
effect()
```

Com algumas funcionalidades extras.

---

# watch()

Agora.

```javascript
watch(

contador,

(novo,antigo)=>{

}
)
```

Existe uma diferença importante.

---

# effect()

Executa imediatamente.

---

# watch()

Observa uma fonte específica e entrega o valor anterior e o novo valor ao callback.

---

# Scheduler

Até agora.

Quando chamamos.

```javascript
trigger()
```

Executamos imediatamente.

---

Mas imagine.

```javascript
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
```

Três renderizações.

---

# Com Scheduler

Resultado.

```text
Render
```

Apenas uma.

---

# Como?

Os efeitos entram em uma fila.

```text
Queue

↓

effect1

effect2

effect3
```

---

# Depois

O Scheduler faz.

```text
Microtask

↓

Executa fila
```

---

# queueJob()

Simplificando.

```javascript
const queue =

new Set()
```

Quando um efeito dispara.

```javascript
queue.add(effect)
```

O `Set` evita duplicatas.

---

# flushJobs()

Depois.

```javascript
for(

const job

of queue

){

job.run()

}
```

Fila limpa.

---

# Microtasks

O Vue utiliza.

```javascript
Promise.resolve()
```

Para agendar.

```javascript
Promise

.resolve()

.then(flushJobs)
```

Assim.

Múltiplas alterações geram apenas uma atualização.

---

# Flush Timing

O `watch()` permite escolher quando executar.

```javascript
flush:"pre"
```

Antes da renderização.

---

```javascript
flush:"post"
```

Depois da renderização.

---

```javascript
flush:"sync"
```

Imediatamente.

---

# Computed + Scheduler

Quando uma dependência muda.

O Scheduler.

↓

Marca.

```text
dirty = true
```

Mas não recalcula imediatamente.

Só recalcula quando alguém acessa.

```javascript
computed.value
```

---

# Arquitetura completa

```text
Proxy

↓

track()

↓

WeakMap

↓

ReactiveEffect

↓

trigger()

↓

Scheduler

↓

Render Function

↓

DOM
```

---

# Comparando

Nossa MiniVue.

```text
Proxy

↓

track

↓

trigger
```

Vue.

```text
Proxy

↓

track

↓

ReactiveEffect

↓

Scheduler

↓

Computed

↓

Watch

↓

Renderer
```

---

# Arquivos reais

Grande parte dessa implementação está em.

```text
packages/reactivity/src
```

Arquivos importantes.

```text
effect.ts

reactive.ts

ref.ts

computed.ts

watch.ts

effectScope.ts
```

---

# Curiosidade

Embora `ref()` e `reactive()` pareçam APIs completamente diferentes, ambas utilizam exatamente o mesmo mecanismo de dependências (`track()` e `trigger()`). A diferença é que `reactive()` intercepta propriedades usando `Proxy`, enquanto `ref()` encapsula um único valor por meio do getter e setter da propriedade `value`.

---

# Resumo

Neste capítulo aprendemos que:

* Toda a reatividade do Vue gira em torno de `track()` e `trigger()`.
* `ReactiveEffect` representa uma computação reativa.
* `WeakMap → Map → Set` armazena as dependências.
* `reactive()` utiliza `Proxy`.
* `ref()` utiliza getters e setters.
* `computed()` implementa cache através da `dirty flag`.
* `watch()` e `watchEffect()` utilizam efeitos reativos.
* O Scheduler evita renderizações desnecessárias.

---

# Exercícios

## Exercício 1

Implemente uma versão simplificada de `effect()`.

---

## Exercício 2

Implemente `track()` utilizando `WeakMap`.

---

## Exercício 3

Implemente `trigger()` executando todos os efeitos registrados.

---

## Exercício 4

Implemente um `ref()` simplificado.

---

## Exercício 5

Implemente um `computed()` com cache utilizando uma `dirty flag`.

---

# Desafio

Atualize sua **MiniVue Reactivity** para suportar:

* `ReactiveEffect`;
* `track()`;
* `trigger()`;
* `reactive()`;
* `ref()`;
* `computed()`;
* um Scheduler simples baseado em microtasks.

---

# Projeto do capítulo

Ao final deste capítulo sua biblioteca deverá conseguir:

* criar objetos reativos;
* registrar dependências automaticamente;
* reagir a alterações;
* implementar valores computados com cache;
* agrupar múltiplas atualizações em uma única execução através do Scheduler.

---

# Checklist

* [ ] Sei explicar o funcionamento de `track()` e `trigger()`.
* [ ] Entendi a estrutura `WeakMap → Map → Set`.
* [ ] Sei implementar `ReactiveEffect`.
* [ ] Entendi como `ref()` funciona internamente.
* [ ] Sei explicar o cache de `computed()`.
* [ ] Entendi o Scheduler e as microtasks.
* [ ] Minha MiniVue já possui uma engine de reatividade funcional.

---

# Próximo capítulo

## **Capítulo 25 — A Engine de Reatividade em Profundidade: `effectScope()`, `customRef()`, `shallowRef()`, `readonly()`, `markRaw()`, `toRef()`, `toRefs()` e Otimizações Internas**

No próximo capítulo exploraremos as APIs mais avançadas do `@vue/reactivity`, incluindo gerenciamento de escopo de efeitos (`effectScope()`), criação de referências personalizadas (`customRef()`), reatividade rasa (`shallowRef()` e `shallowReactive()`), objetos somente leitura (`readonly()`), exclusão de objetos da reatividade (`markRaw()`), utilitários (`toRef()`, `toRefs()`, `toValue()`) e diversas otimizações internas utilizadas pelo Vue para manter alta performance mesmo em aplicações muito grandes.
