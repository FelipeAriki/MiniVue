# Capítulo 39 — Pinia por Dentro: Arquitetura, Stores, Reatividade e Gerenciamento Global de Estado

> **Objetivo:** compreender profundamente a arquitetura do **Pinia**, o gerenciador de estado oficial do Vue 3. Ao final deste capítulo você entenderá como as Stores são criadas, como utilizam a reatividade do Vue, por que o Pinia substituiu o Vuex, como funciona seu sistema interno e como implementar uma versão simplificada na sua MiniVue.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Explicar por que o Pinia existe.
* Entender a arquitetura das Stores.
* Compreender como o Pinia utiliza a reatividade do Vue.
* Explicar Actions, Getters e State internamente.
* Criar plugins para o Pinia.
* Implementar um sistema semelhante na MiniVue.

---

# Pré-requisitos

* Capítulos 01 ao 38.

---

# O problema

Imagine.

Uma aplicação.

```text
ERP
```

Possui.

* Login
* Dashboard
* Compras
* Estoque
* Financeiro
* RH

---

Todos.

Precisam.

Saber.

Quem é.

O usuário.

---

Sem.

Estado global.

Precisaríamos.

Passar.

Props.

Por dezenas.

De componentes.

---

Visualmente.

```text
App

↓

Layout

↓

Dashboard

↓

Tabela

↓

Botão
```

---

O usuário.

Precisaria.

Passar.

Por toda.

A árvore.

---

Esse problema.

É conhecido.

Como.

```text
Prop Drilling
```

---

# A solução

Criar.

Um estado.

Global.

---

Que qualquer.

Componente.

Possa acessar.

---

É exatamente.

O papel.

Do Pinia.

---

# Primeiro exemplo

```javascript
export const

useUsuarioStore=

defineStore(

"usuario",

{

state:()=>({

nome:"",

idade:0

})

}
)
```

---

Uso.

```javascript
const usuario=

useUsuarioStore()
```

---

Agora.

Qualquer.

Componente.

Possui acesso.

Ao mesmo.

Estado.

---

# Fluxo

```text
Componentes

↓

Store

↓

Reactive

↓

Renderer

↓

DOM
```

---

# O que é uma Store?

Uma Store.

É um objeto.

Reativo.

Compartilhado.

---

Visualmente.

```text
Store

↓

State

↓

Actions

↓

Getters
```

---

# State

Representa.

Os dados.

---

```javascript
state:()=>({

contador:0

})
```

---

Internamente.

O Pinia.

Transforma.

Esse objeto.

Em.

```javascript
reactive()
```

---

Assim.

Qualquer alteração.

Dispara.

Atualizações.

---

# Exemplo

```javascript
store.contador++
```

---

O Vue.

Detecta.

A alteração.

---

Depois.

Atualiza.

Os componentes.

Automaticamente.

---

# Actions

Representam.

Métodos.

---

```javascript
actions:{

incrementar(){

this.contador++

}

}
```

---

Nada mais.

São.

Funções.

Ligadas.

À Store.

---

# Getters

Representam.

Valores.

Derivados.

---

```javascript
getters:{

dobro(){

return

this.contador*2

}

}
```

---

Internamente.

São criados.

Como.

```javascript
computed()
```

---

Assim.

São.

Cacheados.

Automaticamente.

---

# Fluxo interno

```text
State

↓

Computed

↓

Getter

↓

Componentes
```

---

# Por que substituir o Vuex?

O Vuex.

Possuía.

Uma arquitetura.

Mais rígida.

---

Precisávamos.

Criar.

```text
State
```

↓

```text
Mutations
```

↓

```text
Actions
```

↓

```text
Modules
```

---

No Pinia.

Basta.

```javascript
store.contador++
```

---

Muito mais.

Simples.

---

# Instalação

```javascript
const pinia=

createPinia()

app.use(

pinia

)
```

---

Observe.

Ele é.

Apenas.

Um Plugin.

---

Assim.

Como estudamos.

No capítulo.

Anterior.

---

# Como funciona?

Quando.

Chamamos.

```javascript
createPinia()
```

---

É criada.

Uma instância.

Global.

---

Ela contém.

Principalmente.

Um Map.

De Stores.

---

Visualmente.

```text
Pinia

↓

Map

↓

Stores
```

---

# Registro

Quando.

Executamos.

```javascript
useUsuarioStore()
```

---

O Pinia.

Pergunta.

```javascript
map.has(

"id"

)
```

---

Se existir.

Retorna.

A Store.

---

Caso contrário.

Cria.

Uma nova.

---

Resultado.

Sempre existe.

Uma única.

Instância.

Por Store.

---

# Fluxo

```text
useStore()

↓

Existe?

↓

Sim

↓

Retornar

↓

Não

↓

Criar

↓

Registrar
```

---

# Implementação

Simplificada.

```javascript
const stores=

new Map()
```

---

Depois.

```javascript
if(

stores.has(id)

){

return

stores.get(id)

}
```

---

Caso contrário.

```javascript
stores.set(

id,

novaStore

)
```

---

Muito elegante.

---

# Reatividade

Lembra.

Do capítulo.

Sobre.

Proxy?

---

O Pinia.

Não implementa.

Outro sistema.

De reatividade.

---

Ele reutiliza.

O Runtime.

Do Vue.

---

Principalmente.

```javascript
reactive()
```

---

```javascript
computed()
```

---

```javascript
effect()
```

---

Por isso.

É extremamente.

Leve.

---

# Setup Store

Além.

Da sintaxe.

Clássica.

Também existe.

```javascript
defineStore(

"id",

()=>{

const contador=

ref(0)

function inc(){

contador.value++

}

return{

contador,

inc

}

}
)
```

---

Essa abordagem.

É chamada.

```text
Setup Store
```

---

Ela utiliza.

A Composition API.

Diretamente.

---

# Option Store

A primeira.

Sintaxe.

É chamada.

```text
Option Store
```

---

Muito semelhante.

À Options API.

---

Hoje.

Ambas.

São utilizadas.

---

# Plugins

O Pinia.

Também possui.

Plugins.

---

Exemplo.

```javascript
pinia.use(

plugin

)
```

---

Um plugin.

Pode adicionar.

Novas propriedades.

À Store.

---

Exemplo.

```javascript
store.$persist
```

---

Ou.

```javascript
store.$reset
```

---

Tudo.

Via Plugin.

---

# DevTools

Outro detalhe.

Muito interessante.

---

O Pinia.

Se integra.

Às.

Vue DevTools.

---

Cada alteração.

Na Store.

É registrada.

---

Facilitando.

Debug.

---

# Hot Module Replacement

Durante.

O desenvolvimento.

---

As Stores.

Podem ser.

Atualizadas.

Sem perder.

O estado.

---

Isso melhora.

Muito.

A experiência.

Do desenvolvedor.

---

# SSR

O Pinia.

Também suporta.

Server Side Rendering.

---

O estado.

É serializado.

No servidor.

---

Depois.

Hidratado.

No cliente.

---

# Persistência

O Pinia.

Não persiste.

Dados.

Automaticamente.

---

Mas.

Plugins.

Como.

```text
pinia-plugin-persistedstate
```

---

Permitem.

Salvar.

No.

```text
LocalStorage
```

---

Ou.

```text
SessionStorage
```

---

# Fluxo completo

```text
Componente

↓

useStore()

↓

Pinia

↓

Map

↓

Store

↓

Reactive

↓

Renderer

↓

DOM
```

---

# Como implementar?

Na MiniVue.

Podemos começar.

Assim.

```javascript
const stores=

new Map()

function defineStore(

id,

factory

){

return()=>{

if(

stores.has(id)

){

return

stores.get(id)

}

const store=

reactive(

factory()

)

stores.set(

id,

store

)

return store

}

}
```

---

Já temos.

Uma Store.

Global.

---

# Evoluindo

Depois.

Adicionar.

* Getters;
* Actions;
* Plugins;
* Persistência;
* DevTools;
* HMR.

---

Sua MiniVue.

Terá.

Um sistema.

Muito parecido.

Com.

O Pinia.

---

# Performance

O Pinia.

Atualiza.

Apenas.

Os componentes.

Que realmente.

Utilizam.

O estado.

---

Isso acontece.

Graças.

Ao sistema.

De dependências.

Da reatividade.

Do Vue.

---

# Código-fonte

Grande parte da arquitetura do Pinia pode ser estudada em:

```text
packages/pinia/src/store.ts
```

Também vale analisar:

```text
createPinia.ts
```

---

```text
rootStore.ts
```

---

```text
plugins.ts
```

---

Esses arquivos mostram como as Stores são registradas, reutilizadas e conectadas ao sistema de reatividade do Vue.

---

# Curiosidade

Ao contrário do Vuex, o Pinia **não utiliza mutations**. As alterações acontecem diretamente no estado reativo. Isso reduz bastante a complexidade da API e permite melhor inferência de tipos, integração com a Composition API e um código muito mais enxuto.

---

# Resumo

Neste capítulo aprendemos que:

* O Pinia resolve o problema do gerenciamento global de estado.
* Cada Store é uma instância única registrada em um `Map`.
* O estado utiliza `reactive()`.
* Getters são implementados com `computed()`.
* Actions são métodos da Store.
* O Pinia reutiliza completamente o sistema de reatividade do Vue.

---

# Exercícios

## Exercício 1

Implemente uma Store de contador utilizando `defineStore()`.

---

## Exercício 2

Adicione suporte a Getters utilizando `computed()`.

---

## Exercício 3

Implemente Actions que alterem o estado.

---

## Exercício 4

Crie um sistema de registro de Stores utilizando `Map`.

---

## Exercício 5

Leia os arquivos principais do Pinia e identifique como ele garante que cada Store exista apenas uma vez durante a execução da aplicação.

---

# Desafio

Atualize sua **MiniVue** para suportar:

* `defineStore()`;
* Stores globais;
* Getters;
* Actions;
* Plugins;
* persistência opcional;
* integração com DevTools (simulada).

---

# Projeto do capítulo

Ao final deste capítulo sua MiniVue deverá possuir um gerenciador de estado global inspirado no Pinia, capaz de compartilhar estado entre componentes, criar Stores reutilizáveis e atualizar automaticamente a interface quando os dados forem alterados.

---

# Checklist

* [ ] Sei explicar por que o Pinia substituiu o Vuex.
* [ ] Entendi como as Stores são registradas.
* [ ] Sei a diferença entre State, Getters e Actions.
* [ ] Entendi como o Pinia reutiliza a reatividade do Vue.
* [ ] Minha MiniVue possui um sistema básico de Stores.

---

# Próximo capítulo

## **Capítulo 40 — Performance no Vue 3: Otimizações do Compiler, Renderer e Runtime**

No próximo capítulo estudaremos um dos temas mais importantes para se tornar especialista em Vue: **Performance**. Você aprenderá como o Compiler gera código otimizado, como funcionam **Patch Flags**, **Static Hoisting**, **Tree Flattening**, **Block Tree**, atualização seletiva de VNodes, otimizações do Scheduler e diversas técnicas utilizadas internamente pelo Vue para renderizar aplicações extremamente rápidas.
