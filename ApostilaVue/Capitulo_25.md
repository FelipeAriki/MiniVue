# Capítulo 25 — A Engine de Reatividade em Profundidade: `effectScope()`, `customRef()`, `shallowRef()`, `readonly()`, `markRaw()`, `toRef()`, `toRefs()` e Otimizações Internas

> **Objetivo:** dominar as APIs mais avançadas do pacote `@vue/reactivity`. Ao final deste capítulo você entenderá como o Vue gerencia ciclos de vida de efeitos, cria referências customizadas, trabalha com reatividade rasa, objetos somente leitura, exclusão da reatividade e diversas otimizações utilizadas internamente pelo framework.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Explicar o funcionamento do `effectScope()`.
* Implementar `readonly()`.
* Implementar `shallowReactive()` e `shallowRef()`.
* Criar `customRef()`.
* Entender `markRaw()`.
* Explicar `toRef()`, `toRefs()` e `toValue()`.
* Compreender otimizações internas da engine.

---

# Pré-requisitos

* Capítulos 01 ao 24.

---

# Recapitulando

No capítulo anterior implementamos:

* `effect()`
* `track()`
* `trigger()`
* `reactive()`
* `ref()`
* `computed()`
* `watch()`
* Scheduler

Agora veremos APIs utilizadas principalmente em bibliotecas e aplicações complexas.

---

# O problema

Imagine.

```javascript
const stop1 = watch(...)

const stop2 = watchEffect(...)

const stop3 = effect(...)
```

Quando o componente for destruído.

Precisamos executar.

```javascript
stop1()

stop2()

stop3()
```

Mas...

E se existirem 150 effects?

---

# Surge o Effect Scope

Existe uma API chamada.

```javascript
effectScope()
```

Ela agrupa vários efeitos.

---

# Exemplo

```javascript
const scope = effectScope()

scope.run(() => {

    watch(...)

    watchEffect(...)

    effect(...)

})
```

Agora existe apenas.

```javascript
scope.stop()
```

Todos os efeitos são destruídos.

---

# Como funciona?

Existe uma variável global.

```javascript
let activeEffectScope
```

Sempre que criamos um novo Scope.

```javascript
activeEffectScope = novoScope
```

Todos os efeitos criados dentro dele ficam registrados.

---

# Estrutura

```javascript
class EffectScope {

    effects = []

    scopes = []

}
```

---

# Criando um Effect

Quando executamos.

```javascript
effect(...)
```

O Vue verifica.

```javascript
if(activeEffectScope){

    activeEffectScope.effects.push(effect)

}
```

Pronto.

O efeito pertence ao Scope.

---

# stop()

Depois.

```javascript
scope.stop()
```

Executa.

```javascript
effect.stop()
```

Para todos os efeitos registrados.

---

# Escopos Aninhados

Também é possível.

```javascript
scopeA

↓

scopeB

↓

scopeC
```

Ao destruir o pai.

Todos os filhos também são destruídos.

---

# readonly()

Outra API muito importante.

Imagine.

```javascript
const usuario = readonly({

    nome:"Felipe"

})
```

Depois.

```javascript
usuario.nome = "Lucas"
```

Resultado.

Nada acontece.

Em desenvolvimento o Vue também emite um aviso no console.

---

# Implementação

É praticamente o mesmo Proxy.

```javascript
get()
```

Continua funcionando.

---

Mas.

```javascript
set()
```

Nunca altera o valor.

```javascript
return true
```

Ou exibe um Warning.

---

# Fluxo

```text
Objeto

↓

Proxy

↓

get()

↓

track()
```

---

```text
set()

↓

Ignorado
```

---

# shallowReadonly()

Agora.

```javascript
const estado = shallowReadonly({

    usuario:{

        nome:"Felipe"

    }

})
```

---

Apenas.

O primeiro nível.

É readonly.

---

Resultado.

```javascript
estado.usuario = {}
```

↓

Bloqueado.

---

Mas.

```javascript
estado.usuario.nome = "Lucas"
```

↓

Permitido.

---

# shallowReactive()

Mesma ideia.

```javascript
const estado = shallowReactive({

    usuario:{

        nome:"Felipe"

    }

})
```

---

Apenas.

O primeiro nível.

É Proxy.

---

Internamente.

```javascript
estado.usuario
```

Não recebe outro Proxy.

---

# Por quê?

Performance.

---

# Quando usar?

Imagine.

```javascript
const resposta = await fetch(...)
```

Recebemos.

50 MB de JSON.

Não faz sentido transformar tudo em Proxy.

Então.

```javascript
shallowReactive()
```

Pode economizar memória e processamento.

---

# shallowRef()

Agora.

```javascript
const lista = shallowRef([])
```

Observe.

```javascript
lista.value.push(...)
```

Não dispara atualização.

---

Mas.

```javascript
lista.value = []
```

Dispara.

---

# Diferença

`ref()`.

↓

Tudo reativo.

---

`shallowRef()`.

↓

Somente.

```javascript
value
```

É observado.

---

# triggerRef()

Existe outra API.

```javascript
triggerRef(lista)
```

Ela força.

```javascript
trigger()
```

Mesmo sem trocar o objeto.

---

# customRef()

Agora uma API extremamente poderosa.

Imagine.

```javascript
const texto = customRef(...)
```

Você controla.

* quando registrar dependências;
* quando disparar atualizações.

---

# Estrutura

```javascript
customRef(

(track,trigger)=>{

}
)
```

---

# Exemplo

Um debounce.

```javascript
const busca = customRef((track, trigger) => {

    let value = ""

    let timer

    return {

        get() {

            track()

            return value

        },

        set(v) {

            clearTimeout(timer)

            timer = setTimeout(() => {

                value = v

                trigger()

            },300)

        }

    }

})
```

Agora.

A atualização ocorre somente após 300 ms.

---

# markRaw()

Outro recurso.

```javascript
const mapa = markRaw(

new Map()

)
```

Resultado.

Nunca será convertido em Proxy.

---

# Quando usar?

Bibliotecas externas.

```javascript
const chart =

markRaw(

new Chart(...)

)
```

Ou.

```javascript
const editor =

markRaw(

new MonacoEditor()

)
```

Objetos muito grandes ou de terceiros costumam ser marcados como *raw* para evitar interferência da reatividade.

---

# Como funciona?

Internamente.

O Vue adiciona.

Uma marca.

```javascript
__v_skip
```

Quando `reactive()` encontra essa marca.

↓

Retorna o objeto original.

---

# toRef()

Agora.

Imagine.

```javascript
const usuario = reactive({

    nome:"Felipe"

})
```

Queremos.

```javascript
usuario.nome
```

Como.

```javascript
ref
```

---

Usamos.

```javascript
const nome =

toRef(

usuario,

"nome"

)
```

Agora.

```javascript
nome.value
```

Aponta diretamente para.

```javascript
usuario.nome
```

---

# Observe

Mudando.

```javascript
nome.value = "Lucas"
```

Também muda.

```javascript
usuario.nome
```

Porque.

É apenas uma referência.

---

# Implementação conceitual

```javascript
class ObjectRef{

    get value(){

        return obj[key]

    }

    set value(v){

        obj[key]=v

    }

}
```

---

# toRefs()

Agora.

```javascript
const usuario = reactive({

    nome:"Felipe",

    idade:24

})
```

---

Executando.

```javascript
const {

nome,

idade

}=toRefs(usuario)
```

Cada propriedade.

Vira.

```javascript
ref
```

---

# Muito utilizado

Principalmente.

Dentro de Composables.

---

# Problema clássico

Isto.

```javascript
const {

nome

}=usuario
```

Perde a reatividade.

---

Mas isto.

```javascript
const {

nome

}=toRefs(usuario)
```

Mantém.

---

# toValue()

Outra API recente.

Aceita.

* valor;
* ref;
* getter.

---

Exemplo.

```javascript
toValue(10)
```

↓

```text
10
```

---

```javascript
toValue(ref(10))
```

↓

```text
10
```

---

```javascript
toValue(()=>10)
```

↓

```text
10
```

Muito utilizada por bibliotecas como VueUse.

---

# isReactive()

Outra API.

```javascript
isReactive(obj)
```

Retorna.

```javascript
true
```

Ou.

```javascript
false
```

---

# isReadonly()

Também existe.

```javascript
isReadonly(obj)
```

---

# isRef()

```javascript
isRef(valor)
```

---

# unref()

Atalho.

```javascript
unref(ref(10))
```

↓

```text
10
```

Equivalente a.

```javascript
ref.value
```

Quando o argumento é realmente um `ref`.

---

# proxyRefs()

Muito utilizada internamente.

Ela permite.

```javascript
usuario.nome
```

Em vez de.

```javascript
usuario.nome.value
```

Quando uma propriedade é um `ref`.

---

# O Template

Lembra.

```vue
{{ contador }}
```

Sem.

```javascript
contador.value
```

Grande parte dessa experiência é possível graças ao desempacotamento automático realizado em contexto de template e por utilitários internos relacionados.

---

# Flags internas

O Vue utiliza diversas marcas.

Como.

```text
__v_isRef

__v_isReactive

__v_isReadonly

__v_skip
```

Para identificar rapidamente o tipo de objeto.

---

# Performance

Por que tantas APIs?

Porque nem toda aplicação precisa.

De reatividade profunda.

Em objetos gigantes.

Ou.

Em bibliotecas externas.

Cada API resolve um cenário específico.

---

# Arquitetura completa

```text
reactive()

↓

readonly()

↓

shallowReactive()

↓

shallowReadonly()

↓

markRaw()

↓

ref()

↓

shallowRef()

↓

customRef()

↓

effectScope()
```

---

# Arquivos reais

Grande parte dessa implementação está em.

```text
packages/reactivity/src
```

Arquivos importantes.

```text
effectScope.ts

ref.ts

reactive.ts

baseHandlers.ts

collectionHandlers.ts

reactiveFlags.ts
```

---

# Comparando

Nossa MiniVue.

```text
reactive()

↓

ref()

↓

computed()
```

Vue.

```text
reactive()

↓

readonly()

↓

shallowReactive()

↓

markRaw()

↓

customRef()

↓

effectScope()

↓

Scheduler

↓

Renderer
```

---

# Curiosidade

Uma das razões pelas quais bibliotecas como **Pinia** conseguem liberar automaticamente recursos quando uma store deixa de ser utilizada é justamente o uso de `effectScope()`. Em vez de controlar cada `watch` individualmente, elas agrupam todos os efeitos em um único escopo e encerram tudo com apenas uma chamada para `scope.stop()`.

---

# Resumo

Neste capítulo aprendemos que:

* `effectScope()` agrupa efeitos reativos.
* `readonly()` impede alterações em objetos reativos.
* `shallowReactive()` e `shallowRef()` tornam apenas o primeiro nível reativo.
* `customRef()` permite controlar manualmente `track()` e `trigger()`.
* `markRaw()` impede que objetos sejam transformados em Proxy.
* `toRef()` e `toRefs()` preservam a reatividade ao trabalhar com propriedades.
* Existem diversas APIs auxiliares como `isRef()`, `unref()` e `toValue()`.

---

# Exercícios

## Exercício 1

Implemente um `readonly()` simplificado utilizando `Proxy`.

---

## Exercício 2

Implemente um `shallowReactive()`.

---

## Exercício 3

Implemente um `toRef()` para propriedades de um objeto reativo.

---

## Exercício 4

Implemente um `effectScope()` simplificado.

---

## Exercício 5

Crie um `customRef()` que implemente debounce de 500 ms.

---

# Desafio

Atualize sua **MiniVue Reactivity** para suportar:

* `readonly()`;
* `shallowReactive()`;
* `shallowRef()`;
* `effectScope()`;
* `markRaw()`;
* `toRef()`;
* `toRefs()`;
* `customRef()`.

---

# Projeto do capítulo

Ao final deste capítulo sua biblioteca deverá conseguir:

* controlar grupos de efeitos;
* criar objetos somente leitura;
* oferecer reatividade rasa;
* excluir objetos da reatividade;
* preservar reatividade durante desestruturação;
* permitir referências personalizadas.

---

# Checklist

* [ ] Sei explicar `effectScope()`.
* [ ] Entendi a diferença entre `ref()` e `shallowRef()`.
* [ ] Sei quando utilizar `markRaw()`.
* [ ] Entendi `readonly()` e `shallowReadonly()`.
* [ ] Sei utilizar `toRef()` e `toRefs()`.
* [ ] Entendi `customRef()`.
* [ ] Minha MiniVue já implementa as APIs avançadas de reatividade.

---

# Próximo capítulo

## **Capítulo 26 — O Renderer do Vue: Implementando `createRenderer()` do Zero**

Até agora entendemos o **Compiler** e a **Engine de Reatividade**. No próximo capítulo uniremos tudo isso construindo o **Renderer** do Vue, implementando `createRenderer()` praticamente do zero. Você aprenderá como o Vue consegue renderizar não apenas para o DOM, mas também para **NativeScript**, **Canvas**, **WebGL**, **Terminal**, **PDF** e outras plataformas, entendendo por que o Runtime do Vue é completamente independente da plataforma de destino. Este é um dos capítulos mais importantes para compreender a arquitetura completa do framework.
