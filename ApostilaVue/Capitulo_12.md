# Capítulo 12 — `reactive()`: Implementando Objetos Reativos Completos

> **Objetivo:** implementar uma versão extremamente próxima do `reactive()` do Vue 3. Ao final deste capítulo você entenderá como o Vue cria Proxies, evita criar múltiplos Proxies para o mesmo objeto, implementa reatividade profunda (*Deep Reactive*), `readonly()`, `shallowReactive()`, `markRaw()`, `toRaw()` e como toda essa arquitetura funciona internamente.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Implementar um `reactive()` próximo ao oficial.
* Entender o cache de Proxies.
* Explicar por que o Vue utiliza `WeakMap`.
* Compreender Deep Reactivity.
* Implementar `readonly()`.
* Entender `shallowReactive()`.
* Explicar `markRaw()` e `toRaw()`.
* Compreender os *Reactive Flags* utilizados pelo Vue.

---

# Pré-requisitos

* Capítulos 01 ao 11.

---

# Revisando

Até agora implementamos:

```javascript
const usuario = reactive({

    nome: "Felipe"

})
```

Sabemos que o Vue cria um Proxy.

```javascript
return new Proxy(obj,{
    get(){},
    set(){}
})
```

Mas existe um problema enorme.

---

# O problema

Imagine.

```javascript
const usuario = {

    nome:"Felipe"

}

const a = reactive(usuario)

const b = reactive(usuario)
```

Pergunta.

Quantos Proxies foram criados?

Nossa implementação criaria dois.

```text
Proxy A

↓

usuario

Proxy B

↓

usuario
```

Isso é desperdício.

---

# O Vue nunca faz isso

Para cada objeto existe apenas um Proxy.

Sempre.

---

# Como resolver?

Precisamos de um cache.

Imagine.

```text
Objeto

↓

Proxy
```

Sempre que alguém chamar:

```javascript
reactive(usuario)
```

Primeiro verificamos.

```text
Esse objeto já possui Proxy?
```

Se sim.

Retornamos o existente.

---

# WeakMap

Para isso o Vue utiliza.

```javascript
const reactiveMap = new WeakMap()
```

Estrutura.

```text
WeakMap

↓

Objeto Original

↓

Proxy
```

---

# Primeira implementação

```javascript
function reactive(target){

    const existing = reactiveMap.get(target)

    if(existing){

        return existing

    }

    const proxy = new Proxy(target,handlers)

    reactiveMap.set(target,proxy)

    return proxy

}
```

Agora.

```javascript
reactive(usuario)
```

Sempre retorna exatamente o mesmo Proxy.

---

# Por que WeakMap?

Poderíamos usar:

```javascript
Map
```

Mas existe um problema.

---

# Memory Leak

Imagine.

```javascript
let usuario = {

}
```

Depois.

```javascript
usuario = null
```

O objeto não possui mais referências.

O Garbage Collector deveria removê-lo.

---

# O problema do Map

Se utilizarmos.

```javascript
Map
```

Ainda existe.

```text
Map

↓

Objeto
```

Resultado.

Nunca será removido.

Memory Leak.

---

# WeakMap resolve isso

No WeakMap.

As chaves são fracas.

Quando o objeto deixa de existir.

O Garbage Collector remove automaticamente sua entrada.

Por isso praticamente toda a arquitetura reativa do Vue utiliza `WeakMap`.

---

# Deep Reactivity

Observe.

```javascript
const usuario = reactive({

    endereco:{

        cidade:"São Paulo"

    }

})
```

Pergunta.

`endereco` também é reativo?

Sim.

---

# Como?

No getter.

Quando encontramos um objeto.

Retornamos.

```javascript
reactive(objeto)
```

Automaticamente.

---

# Implementação

```javascript
get(target,key,receiver){

    const result = Reflect.get(

        target,

        key,

        receiver

    )

    if(

        typeof result === "object"

        &&

        result !== null

    ){

        return reactive(result)

    }

    return result

}
```

Perceba.

A reatividade profunda acontece durante a leitura.

---

# Lazy Reactive

O Vue não percorre todo o objeto.

Imagine.

```javascript
const usuario = {

    a:{},

    b:{},

    c:{},

    d:{}

}
```

Ele não cria quatro Proxies imediatamente.

Somente quando alguém faz.

```javascript
usuario.a
```

O Proxy é criado.

Isso chama-se:

```text
Lazy Proxy Creation
```

Uma importante otimização de desempenho.

---

# Reactive Flags

Como saber se um objeto já é reativo?

O Vue adiciona propriedades internas.

Exemplo.

```javascript
__v_isReactive
```

---

# Exemplo

```javascript
proxy.__v_isReactive
```

Retorna.

```text
true
```

Essas propriedades são chamadas de *Reactive Flags*.

---

# Principais Flags

```text
__v_isReactive

__v_isReadonly

__v_raw

__v_skip
```

Cada uma possui uma finalidade específica.

---

# __v_raw

Imagine.

```javascript
const proxy = reactive(usuario)
```

Como recuperar o objeto original?

O Vue permite.

```javascript
toRaw(proxy)
```

Internamente.

```javascript
proxy.__v_raw
```

Retorna.

```javascript
usuario
```

---

# Implementando

No getter.

```javascript
if(

key==="__v_raw"

){

    return target

}
```

---

# toRaw()

Agora.

```javascript
function toRaw(proxy){

    return proxy.__v_raw

}
```

Muito simples.

---

# readonly()

Agora um novo conceito.

Imagine.

```javascript
const usuario = readonly({

    nome:"Vue"

})
```

Quando alguém faz.

```javascript
usuario.nome = "React"
```

Nada acontece.

Ou o Vue emite um Warning em desenvolvimento.

---

# Implementação

```javascript
set(){

    console.warn(

        "Objeto readonly"

    )

    return true

}
```

Todo o restante continua funcionando.

Inclusive a leitura.

---

# Visualizando

```text
get()

↓

Permitido

set()

↓

Bloqueado
```

---

# readonly profundo

Assim como `reactive()`.

`readonly()` também é profundo.

```javascript
const usuario = readonly({

    endereco:{

        cidade:"SP"

    }

})
```

Também impede.

```javascript
usuario.endereco.cidade = "RJ"
```

---

# shallowReactive()

Agora.

```javascript
const usuario = shallowReactive({

    endereco:{

        cidade:"SP"

    }

})
```

Apenas.

```javascript
usuario
```

É reativo.

Mas.

```javascript
usuario.endereco
```

É um objeto comum.

---

# Diferença

reactive.

```text
Todos os níveis
```

shallowReactive.

```text
Somente primeiro nível
```

---

# Quando utilizar?

Em objetos gigantes.

Ou quando outra biblioteca controla os objetos internos.

Exemplo.

* Leaflet
* Chart.js
* Monaco Editor
* OpenLayers

---

# markRaw()

Imagine.

```javascript
const mapa = new Mapa()
```

Não queremos.

```javascript
reactive(mapa)
```

Porque pode quebrar a biblioteca.

O Vue permite.

```javascript
markRaw(mapa)
```

---

# Como funciona?

Internamente.

```javascript
mapa.__v_skip = true
```

Depois.

```javascript
if(

target.__v_skip

){

    return target

}
```

O objeto nunca será transformado em Proxy.

---

# isReactive()

Implementação.

```javascript
function isReactive(valor){

    return !!valor.__v_isReactive

}
```

---

# isReadonly()

Mesma ideia.

```javascript
return !!valor.__v_isReadonly
```

---

# Proxy Handlers

Na implementação real.

O Vue possui vários conjuntos de handlers.

Exemplo.

```text
Mutable Handlers

Readonly Handlers

Shallow Handlers

Collection Handlers
```

Todos reutilizam boa parte da lógica.

---

# ReactiveFactory

Na implementação oficial existe uma função chamada.

```text
createReactiveObject()
```

Ela recebe parâmetros.

```javascript
createReactiveObject(

target,

isReadonly,

handlers,

proxyMap

)
```

Assim.

O Vue evita duplicação de código.

---

# Fluxo completo

```text
reactive()

↓

WeakMap

↓

Existe Proxy?

↓

Sim

↓

Retorna

↓

Não

↓

Cria Proxy

↓

Salva WeakMap

↓

Retorna
```

---

# Fluxo do getter

```text
Proxy.get()

↓

track()

↓

Reflect.get()

↓

Objeto?

↓

Sim

↓

reactive()

↓

Retorna Proxy

↓

Não

↓

Retorna valor
```

---

# Performance

As três maiores otimizações do `reactive()` são:

* Cache de Proxies.
* Lazy Proxy Creation.
* WeakMap.

Sem essas otimizações.

O desempenho cairia drasticamente em aplicações grandes.

---

# Comparando

Nossa primeira implementação.

```javascript
reactive(obj)
```

↓

Sempre cria Proxy.

---

Vue.

```javascript
reactive(obj)
```

↓

Verifica cache.

↓

Retorna existente.

↓

Cria apenas se necessário.

---

# Curiosidade

Se fizer.

```javascript
const proxy = reactive(obj)

reactive(proxy)
```

O Vue não cria outro Proxy.

Ele identifica que já recebeu um objeto reativo e retorna a própria instância.

Isso evita uma cadeia infinita de Proxies.

---

# Resumo

Neste capítulo aprendemos que:

* O Vue nunca cria dois Proxies para o mesmo objeto.
* `WeakMap` é utilizado como cache.
* Objetos internos tornam-se reativos sob demanda (*Lazy Reactive*).
* `readonly()` compartilha grande parte da implementação de `reactive()`.
* `shallowReactive()` limita a reatividade ao primeiro nível.
* `markRaw()` impede a criação de Proxies.
* `toRaw()` recupera o objeto original.
* Os *Reactive Flags* identificam objetos reativos.

---

# Exercícios

## Exercício 1

Implemente um cache de Proxies utilizando `WeakMap`.

---

## Exercício 2

Implemente `toRaw()`.

---

## Exercício 3

Implemente `isReactive()`.

---

## Exercício 4

Implemente `readonly()` reutilizando o máximo possível da lógica de `reactive()`.

---

# Desafio

Refatore sua biblioteca **MiniVue Reactive** para suportar:

* Cache de Proxies.
* `WeakMap`.
* Deep Reactive.
* Lazy Proxy Creation.
* `readonly()`.
* `shallowReactive()`.
* `markRaw()`.
* `toRaw()`.

---

# Projeto do capítulo

Ao finalizar este capítulo sua biblioteca deverá possuir uma implementação de `reactive()` muito próxima da utilizada pelo Vue 3, incluindo:

* `createReactiveObject()`;
* Cache com `WeakMap`;
* *Reactive Flags*;
* `readonly()`;
* `shallowReactive()`;
* `isReactive()`;
* `isReadonly()`;
* `markRaw()`;
* `toRaw()`.

---

# Checklist

* [ ] Sei explicar por que o Vue utiliza `WeakMap`.
* [ ] Entendi o cache de Proxies.
* [ ] Sei implementar *Deep Reactive*.
* [ ] Entendi a diferença entre `reactive()` e `shallowReactive()`.
* [ ] Sei quando utilizar `markRaw()`.
* [ ] Entendi como `readonly()` funciona internamente.
* [ ] Minha implementação de `reactive()` já se aproxima muito da oficial.

---

# Próximo capítulo

## **Capítulo 13 — `computed()`: Implementando Computed do Zero**

Neste capítulo construiremos o `computed()` exatamente como o Vue faz. Você aprenderá sobre **Lazy Evaluation**, **Dirty Checking**, **Cache de Valores**, **ReactiveEffect com Scheduler**, dependências encadeadas (*Computed Chains*), `Writable Computed`, invalidação de cache e como o Vue consegue recalcular um `computed` apenas quando realmente necessário. Esse é um dos mecanismos mais elegantes e sofisticados do runtime do Vue 3.
