# Capítulo 14 — `watch()` e `watchEffect()`: Implementando o Sistema de Observação do Vue

> **Objetivo:** compreender e implementar completamente o sistema de observação do Vue 3. Ao final deste capítulo você entenderá exatamente como funcionam `watch()`, `watchEffect()`, `deep`, `immediate`, `once`, `flush`, `onCleanup()` e por que essas APIs são diferentes do `computed()`.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Implementar `watch()` do zero.
* Implementar `watchEffect()`.
* Entender a diferença entre ambos.
* Explicar Deep Watch.
* Implementar `immediate`.
* Implementar `once`.
* Entender `flush: pre`, `post` e `sync`.
* Compreender `onCleanup()`.

---

# Pré-requisitos

* Capítulos 01 ao 13.

---

# O problema

Imagine.

```javascript
const contador = ref(0)
```

Queremos executar um código sempre que ele mudar.

```javascript
contador.value++
```

Por exemplo.

```javascript
console.log("Mudou")
```

Como fazer isso?

---

# Primeira ideia

Poderíamos utilizar um Effect.

```javascript
effect(() => {

    console.log(contador.value)

})
```

Mas existe um problema.

O Effect executa imediatamente.

E toda vez que qualquer dependência mudar.

Nem sempre queremos isso.

---

# Surge o watch()

```javascript
watch(contador, (novo, antigo) => {

    console.log(novo)

})
```

Agora.

Executamos apenas quando houver mudança.

---

# A diferença fundamental

`computed()`

↓

Calcula um valor.

---

`watch()`

↓

Executa efeitos colaterais (*side effects*).

---

# O que é um Side Effect?

Qualquer operação que altera algo fora da função.

Por exemplo:

* Fazer uma requisição HTTP.
* Atualizar o `localStorage`.
* Salvar dados.
* Alterar o título da página.
* Enviar métricas.
* Escrever no console.
* Iniciar um timer.

Essas operações não pertencem ao `computed()`.

---

# Como implementar?

Internamente.

`watch()` utiliza um `ReactiveEffect`.

Mas com uma diferença.

O Scheduler não executa diretamente o callback.

Ele agenda um Job.

---

# Estrutura simplificada

```javascript
function watch(source, callback){

}
```

---

# O Source

O primeiro parâmetro pode ser:

```javascript
watch(ref)
```

Ou.

```javascript
watch(reactive)
```

Ou.

```javascript
watch(() => usuario.nome)
```

Ou.

```javascript
watch([a,b,c])
```

Todos esses casos são suportados.

---

# Primeiro passo

Precisamos transformar o Source em uma função.

```javascript
const getter = () => source.value
```

Quando o Source é um Ref.

---

# Criando o Effect

```javascript
const effect = new ReactiveEffect(

    getter,

    scheduler

)
```

---

# Guardando o valor antigo

Precisamos saber.

```javascript
oldValue
```

Inicialmente.

```javascript
oldValue = effect.run()
```

Assim armazenamos o primeiro valor.

---

# O Scheduler

Quando uma dependência mudar.

Executamos.

```javascript
const newValue = effect.run()

callback(

    newValue,

    oldValue

)

oldValue = newValue
```

Agora temos um Watch funcionando.

---

# Fluxo

```text
Mudança

↓

Scheduler

↓

Getter

↓

Novo valor

↓

Callback

↓

Atualiza oldValue
```

---

# Exemplo

```javascript
const idade = ref(20)

watch(idade,(novo,antigo)=>{

    console.log(

        antigo,

        novo

    )

})
```

Depois.

```javascript
idade.value = 21
```

Resultado.

```text
20

21
```

---

# watchEffect()

Agora outro caso.

```javascript
watchEffect(()=>{

    console.log(

        contador.value

    )

})
```

Perceba.

Não existe Source.

---

# Como funciona?

O próprio callback torna-se o Getter.

Ou seja.

Tudo que for acessado.

Será observado.

---

# Visualizando

```javascript
watchEffect(()=>{

    console.log(nome.value)

    console.log(idade.value)

})
```

Dependências.

```text
nome

idade
```

Automaticamente.

---

# Diferença

watch.

```javascript
watch(

() => usuario.nome

)
```

Você escolhe.

---

watchEffect.

```javascript
watchEffect(()=>{

    usuario.nome

    usuario.idade

})
```

O Vue descobre sozinho.

---

# Quando usar?

watch.

Quando você sabe exatamente o que observar.

---

watchEffect.

Quando deseja observar tudo que for utilizado.

---

# immediate

Por padrão.

```javascript
watch()
```

Não executa imediatamente.

---

Mas.

```javascript
watch(

contador,

callback,

{

    immediate:true

}

)
```

Executa assim que é criado.

---

# Implementação

```javascript
if(

options.immediate

){

    job()

}
```

Caso contrário.

Guardamos apenas o valor inicial.

---

# once

No Vue 3.4 foi adicionada a opção.

```javascript
once:true
```

Depois da primeira execução.

O Watch é interrompido automaticamente.

---

# Implementação

```javascript
if(

options.once

){

    stop()

}
```

---

# stop()

Todo Watch retorna uma função.

```javascript
const stop = watch(...)
```

Depois.

```javascript
stop()
```

Todas as dependências são removidas.

O Effect deixa de existir.

---

# Deep Watch

Agora um caso interessante.

```javascript
watch(

usuario,

callback

)
```

Se mudarmos.

```javascript
usuario.endereco.cidade
```

O Watch deve executar.

Mas.

O Getter apenas acessa.

```javascript
usuario
```

---

# Como resolver?

Precisamos percorrer todo o objeto.

---

# traverse()

Implementação simplificada.

```javascript
function traverse(valor){

    if(

        typeof valor

        !==

        "object"

    ){

        return valor

    }

    for(

        const key

        in valor

    ){

        traverse(

            valor[key]

        )

    }

    return valor

}
```

Agora.

Todas as propriedades são lidas.

Consequentemente.

Todas são registradas pelo `track()`.

---

# O Vue faz isso?

Sim.

Mas de maneira muito mais sofisticada.

Incluindo:

* Map
* Set
* Arrays
* WeakMap
* Objetos circulares

---

# Objetos circulares

Imagine.

```javascript
obj.a = obj
```

Nossa implementação entraria em recursão infinita.

O Vue utiliza.

```javascript
const seen = new Set()
```

Para evitar isso.

---

# Flush

Outro recurso importante.

```javascript
watch(

contador,

callback,

{

flush:"pre"

}

)
```

---

# Existem três modos

```text
pre

post

sync
```

---

# sync

Executa imediatamente.

```text
Mudança

↓

Watch
```

---

# pre

Executa antes da atualização do DOM.

Fluxo.

```text
Mudança

↓

Watch

↓

Render
```

É o padrão do Vue.

---

# post

Executa depois do DOM.

Fluxo.

```text
Mudança

↓

Render

↓

Watch
```

Muito utilizado quando precisamos acessar elementos atualizados.

---

# Scheduler

O Scheduler decide em qual fila colocar o Job.

```text
Pre Queue

Render Queue

Post Queue
```

Essa arquitetura é compartilhada com o sistema de renderização.

---

# Cleanup

Imagine.

```javascript
watch(

id,

async()=>{

    await fetch(...)

}
)
```

Depois.

```javascript
id.value++
```

Nova requisição.

Mas a anterior ainda está em andamento.

---

# Problema

A resposta antiga pode chegar depois.

Sobrescrevendo dados recentes.

---

# Solução

```javascript
watch(

id,

(

novo,

antigo,

onCleanup

)=>{

}
)
```

---

# Utilizando

```javascript
watch(

id,

async(

id,

_,

onCleanup

)=>{

    const controller =

        new AbortController()

    onCleanup(()=>{

        controller.abort()

    })

}
)
```

Sempre antes da próxima execução.

O Cleanup é chamado.

---

# Fluxo

```text
Mudança

↓

Cleanup antigo

↓

Novo callback
```

---

# onCleanup vs onUnmounted

São coisas diferentes.

`onCleanup`.

↓

Entre execuções do Watch.

---

`onUnmounted`.

↓

Quando o componente deixa de existir.

---

# watch múltiplo

O Vue aceita.

```javascript
watch(

[

nome,

idade

],

([novoNome,novaIdade])=>{

}
)
```

Cada Source possui seu próprio Getter.

---

# Watch em Getter

Muito comum.

```javascript
watch(

()=>{

    return usuario.nome

},

callback
)
```

Nesse caso.

O Getter já define exatamente a dependência.

Não precisamos de `deep`.

---

# Erro comum

Fazer.

```javascript
watch(

usuario.nome,

callback
)
```

Isso não funciona.

Porque `usuario.nome` já foi avaliado.

O correto é.

```javascript
watch(

()=>usuario.nome,

callback
)
```

---

# Comparação

| API         | Executa imediatamente | Possui valores antigo/novo | Descobre dependências                                             |
| ----------- | --------------------- | -------------------------- | ----------------------------------------------------------------- |
| effect      | Sim                   | Não                        | Sim                                                               |
| watchEffect | Sim                   | Não                        | Sim                                                               |
| watch       | Não (por padrão)      | Sim                        | Não (exceto quando o source é uma função que acessa dependências) |
| computed    | Sob demanda           | Não                        | Sim                                                               |

---

# Internamente

O Vue implementa.

```text
doWatch()
```

Todas as APIs.

```text
watch()

watchEffect()

watchPostEffect()

watchSyncEffect()
```

Chamam.

```text
doWatch()
```

Mudando apenas algumas opções.

---

# Arquitetura

```text
watch()

↓

Getter

↓

ReactiveEffect

↓

Scheduler

↓

Job

↓

Callback
```

---

# Performance

`deep: true` pode ser muito caro em objetos grandes.

Sempre prefira.

```javascript
watch(

()=>usuario.nome
)
```

Em vez de.

```javascript
watch(

usuario,

...

{

deep:true

}
)
```

Sempre que possível.

---

# Resumo

Neste capítulo aprendemos que:

* `watch()` é utilizado para Side Effects.
* `watchEffect()` descobre dependências automaticamente.
* `watch()` trabalha com valores antigo e novo.
* O Vue implementa Deep Watch percorrendo objetos.
* `flush` controla quando o callback será executado.
* `onCleanup()` evita efeitos colaterais antigos.
* Todas essas APIs são construídas sobre `ReactiveEffect`.

---

# Exercícios

## Exercício 1

Implemente um `watch()` simplificado utilizando `ReactiveEffect`.

---

## Exercício 2

Adicione suporte para `immediate`.

---

## Exercício 3

Implemente `watchEffect()`.

---

## Exercício 4

Implemente uma função `traverse()` para suportar `deep: true`.

---

# Desafio

Atualize sua biblioteca **MiniVue Reactive** para suportar:

* `watch()`;
* `watchEffect()`;
* `immediate`;
* `once`;
* `deep`;
* `flush`;
* `onCleanup()`;
* `stop()`.

---

# Projeto do capítulo

Ao final deste capítulo sua biblioteca deverá implementar praticamente toda a API pública de observação do Vue 3, incluindo:

* `watch()`;
* `watchEffect()`;
* `watchPostEffect()`;
* `watchSyncEffect()`;
* `doWatch()`;
* `cleanup`;
* `deep traversal`;
* múltiplas fontes de observação;
* Scheduler integrado.

---

# Checklist

* [ ] Sei explicar a diferença entre `watch()` e `watchEffect()`.
* [ ] Entendi quando utilizar `computed()` e quando utilizar `watch()`.
* [ ] Sei implementar `immediate`.
* [ ] Entendi como funciona `deep: true`.
* [ ] Sei explicar os modos `flush`.
* [ ] Entendi o papel de `onCleanup()`.
* [ ] Minha biblioteca já implementa praticamente toda a camada pública de reatividade do Vue.

---

# Próximo capítulo

## **Capítulo 15 — O Runtime do Vue: Como um Componente Nasce, Renderiza e é Atualizado**

Até aqui reconstruímos praticamente todo o sistema reativo. A partir do próximo capítulo mudaremos de foco: sairemos da camada de reatividade e entraremos no **Runtime Core** do Vue. Você aprenderá como um componente é criado, como surge a instância interna (`ComponentInternalInstance`), como `setup()` é executado, como o Render Effect é criado, como nasce a árvore de Virtual DOM e como o Scheduler se integra ao ciclo completo de renderização. A partir desse ponto estaremos estudando a arquitetura interna do Vue praticamente arquivo por arquivo.
