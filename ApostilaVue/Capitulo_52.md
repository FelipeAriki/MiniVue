# Capítulo 52 — Projeto Final III: Integrando o Compiler ao Runtime (Template → AST → Render Function → DOM)

> **Objetivo:** integrar definitivamente o Compiler ao Runtime da MiniVue. Ao final deste capítulo sua MiniVue será capaz de receber um template, compilá-lo em uma Render Function e utilizá-la para gerar e atualizar o DOM, reproduzindo o fluxo utilizado pelo Vue 3.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Integrar Compiler e Runtime.
* Executar a pipeline completa de compilação.
* Transformar Templates em Render Functions.
* Entender como o Vue conecta compilação e renderização.
* Implementar um compilador em tempo de execução (Runtime Compiler).
* Finalizar uma das partes mais importantes da MiniVue.

---

# Pré-requisitos

* Capítulos 01 ao 51.

---

# Introdução

Até aqui.

Construímos.

O Compiler.

---

Construímos.

O Runtime.

---

Construímos.

O Renderer.

---

Mas.

Ainda.

Existe.

Uma separação.

Entre eles.

---

Agora.

Vamos.

Conectá-los.

---

Esse.

É um dos.

Momentos.

Mais importantes.

Do framework.

---

Porque.

Finalmente.

Um template.

Poderá.

Ser executado.

---

# O fluxo completo

Visualmente.

```text
Template

↓

Parser

↓

AST

↓

Transforms

↓

Code Generator

↓

Render Function

↓

Runtime

↓

VNode

↓

Renderer

↓

DOM
```

---

Todo.

Esse fluxo.

Agora.

Passará.

A funcionar.

Como.

Uma única.

Pipeline.

---

# Antes da integração

Até aqui.

Nosso Runtime.

Esperava.

Receber.

Algo assim.

```javascript
render(){

return h("div")
}
```

---

Mas.

O usuário.

Quer escrever.

```vue
<template>

<div>{{count}}</div>

</template>
```

---

Quem.

Transforma.

Esse template.

Na Render Function?

---

O Compiler.

---

# Compiler + Runtime

Visualmente.

```text
Template

↓

Compiler

↓

Render Function

↓

Runtime
```

---

A Render Function.

É a ponte.

Entre.

Os dois.

---

# Componentes

Imagine.

O componente.

```javascript
const App={

template:`

<div>

{{count}}

</div>

`

}
```

---

Ele.

Ainda.

Não possui.

Uma função.

Render.

---

Precisamos.

Criá-la.

---

# Fluxo de montagem

Quando.

Executamos.

```javascript
createApp(App)
```

---

O Runtime.

Verifica.

---

Existe.

```javascript
render
```

?

---

Se existir.

Usa.

Diretamente.

---

Caso contrário.

---

Existe.

```javascript
template
```

?

---

Se existir.

Compila.

Automaticamente.

---

Fluxo.

```text
template

↓

compile()

↓

render()

↓

mount()
```

---

# compile()

Nossa função.

Pode começar.

Assim.

```javascript
function compile(template){

...

}
```

---

Ela executa.

Internamente.

Todo.

O Compiler.

---

```text
parse()

↓

transform()

↓

generate()
```

---

Retornando.

Uma.

Render Function.

---

# Exemplo

Entrada.

```html
<div>

{{msg}}

</div>
```

---

Saída.

```javascript
function render(){

return h(

"div",

ctx.msg

)

}
```

---

Essa função.

É anexada.

Ao componente.

---

```javascript
component.render=

compile(

component.template

)
```

---

A partir.

Desse momento.

O Runtime.

Não percebe.

Diferença.

---

# Runtime Compiler

Essa técnica.

Recebe.

O nome.

De.

```text
Runtime Compiler
```

---

Porque.

O template.

É compilado.

Durante.

A execução.

---

Existe.

Outra opção.

---

# Build Compiler

Em aplicações.

Vue modernas.

Normalmente.

O template.

É compilado.

Antes.

Da execução.

---

Fluxo.

```text
.vue

↓

Vite

↓

Compiler

↓

JavaScript

↓

Browser
```

---

Nesse caso.

O navegador.

Nunca.

Recebe.

O template.

---

Recebe.

Apenas.

A Render Function.

---

Isso.

Melhora.

A performance.

---

# Nossa MiniVue

Inicialmente.

Implementaremos.

O Runtime Compiler.

---

Porque.

Ele é.

Mais simples.

Para entender.

---

Depois.

Será fácil.

Migrar.

Para.

Build Time.

---

# Pipeline interna

Visualmente.

```text
Template

↓

Lexer

↓

Parser

↓

AST

↓

Transforms

↓

Code Generator

↓

new Function()

↓

Render Function
```

---

Observe.

Que.

A última etapa.

Cria.

Uma função.

Executável.

---

# Gerando a função

Imagine.

Que.

O Code Generator.

Produziu.

```javascript
return h(

"div",

ctx.msg

)
```

---

Podemos.

Criar.

A função.

Assim.

```javascript
new Function(

"ctx",

codigo

)
```

---

Obtendo.

Uma função.

Executável.

---

# Integração

No Runtime.

Implementamos.

Algo semelhante.

```javascript
if(

!component.render

){

component.render=

compile(

component.template

)

}
```

---

Depois.

Tudo.

Continua.

Igual.

---

```javascript
render()

↓

VNode

↓

Patch()

↓

DOM
```

---

# Cache

Compilar.

É caro.

---

Por isso.

O Vue.

Nunca.

Compila.

O mesmo.

Template.

Duas vezes.

---

Podemos.

Criar.

Um cache.

---

```javascript
Map()
```

---

Fluxo.

```text
Template

↓

Cache?

↓

Sim

↓

Render Function

↓

Não

↓

Compile()

↓

Salvar

↓

Render Function
```

---

Assim.

Melhoramos.

Muito.

O desempenho.

---

# Componentes filhos

O Compiler.

Também.

Precisa.

Reconhecer.

```vue
<Button/>
```

---

Gerando.

```javascript
resolveComponent(

"Button"

)
```

---

O Runtime.

Resolve.

Esse componente.

Na aplicação.

---

# Diretivas

Também.

São.

Transformadas.

---

Exemplo.

```vue
v-if
```

---

Vira.

Chamadas.

Para.

Helpers.

Internos.

---

O Runtime.

Executa.

Esses helpers.

Durante.

A renderização.

---

# Slots

Da mesma forma.

---

```vue
<slot/>
```

---

É convertido.

Em.

Chamadas.

Para.

Render Slots.

---

Todo.

O trabalho.

Pesado.

Foi realizado.

Pelo Compiler.

---

# Eventos

Exemplo.

```vue
<button

@click="save"
/>
```

---

Compila.

Para.

```javascript
onClick:

ctx.save
```

---

O Runtime.

Apenas.

Registra.

O evento.

---

# Reatividade

Durante.

A Render Function.

---

Lemos.

```javascript
ctx.count
```

---

Isso.

Dispara.

O tracking.

---

Fluxo.

```text
Render

↓

track()

↓

Reactive Effect

↓

Dependências
```

---

Depois.

Quando.

O estado.

Muda.

---

Executamos.

```text
trigger()

↓

Scheduler

↓

Renderer
```

---

Sem.

Recompilar.

O template.

---

Observe.

Que.

O Compiler.

É utilizado.

Apenas.

Uma vez.

---

Depois.

Tudo.

É controlado.

Pelo Runtime.

---

# Como implementar?

Na MiniVue.

Crie.

O arquivo.

```text
compiler/

compile.js
```

---

Depois.

Implemente.

O fluxo.

```javascript
compile(){

parse()

transform()

generate()

}
```

---

No Runtime.

Atualize.

A montagem.

```javascript
if(

!component.render

){

component.render=

compile(

component.template

)

}
```

---

A integração.

Está.

Concluída.

---

# Organização

Visualmente.

```text
Compiler

↓

Render Function

↓

Runtime Core

↓

Renderer

↓

Runtime DOM
```

---

Essa.

É exatamente.

A arquitetura.

Utilizada.

Pelo Vue.

---

# Performance

A separação.

Entre.

Compiler.

E Runtime.

Permite.

---

Templates.

Pré-compilados.

---

Bundle menor.

---

Execução.

Mais rápida.

---

Cache.

De Render Functions.

---

Menor.

Uso.

De CPU.

---

# Código-fonte

Os principais arquivos relacionados à integração entre Compiler e Runtime são:

```text
packages/compiler-core/src/compile.ts
```

---

```text
packages/compiler-core/src/codegen.ts
```

---

```text
packages/runtime-core/src/component.ts
```

---

```text
packages/runtime-core/src/componentRenderUtils.ts
```

---

```text
packages/vue/src/index.ts
```

---

O pacote `vue` é responsável por conectar o Runtime ao Compiler quando a build utilizada inclui suporte à compilação em tempo de execução.

---

# Curiosidade

O Vue distribui diferentes builds para cenários distintos. A versão utilizada com ferramentas como Vite normalmente não inclui o Runtime Compiler, pois os templates já foram compilados durante o processo de build. Já a versão completa ("full build") incorpora o compilador, permitindo escrever templates diretamente no navegador.

---

# Resumo

Neste capítulo aprendemos que:

* O Compiler produz Render Functions consumidas pelo Runtime.
* O Runtime verifica se o componente possui `render()` ou `template`.
* O Runtime Compiler compila templates durante a execução.
* Em aplicações modernas, a compilação normalmente ocorre durante o build.
* A Render Function é criada apenas uma vez e reutilizada em todas as renderizações.
* Compiler e Runtime permanecem desacoplados, comunicando-se através da Render Function.

---

# Exercícios

## Exercício 1

Implemente uma função `compile()` que execute `parse()`, `transform()` e `generate()`.

---

## Exercício 2

Adicione suporte ao Runtime Compiler na montagem dos componentes.

---

## Exercício 3

Implemente um cache para Render Functions compiladas.

---

## Exercício 4

Faça um componente que possua apenas `template` e verifique se ele é compilado automaticamente.

---

## Exercício 5

Desenhe toda a pipeline desde o template até a atualização do DOM após uma alteração reativa.

---

# Desafio

Atualize sua **MiniVue** para suportar:

* Runtime Compiler;
* compilação automática de templates;
* cache de Render Functions;
* integração completa entre Compiler e Runtime;
* componentes escritos exclusivamente com `template`.

---

# Projeto do capítulo

Ao final deste capítulo sua MiniVue será capaz de receber componentes escritos com templates, compilá-los automaticamente em Render Functions e renderizá-los utilizando o Runtime e o Renderer, reproduzindo a arquitetura central do Vue 3.

---

# Checklist

* [ ] Entendi como o Compiler se integra ao Runtime.
* [ ] Sei explicar o fluxo Template → AST → Render Function → DOM.
* [ ] Minha MiniVue suporta Runtime Compiler.
* [ ] Implementei cache para Render Functions compiladas.
* [ ] Compiler e Runtime estão totalmente integrados.

---

# Próximo capítulo

## **Capítulo 53 — Projeto Final IV: Implementando um Virtual DOM Completo com Otimizações Avançadas**

No próximo capítulo elevaremos a MiniVue a um novo nível implementando um **Virtual DOM completo**, incluindo algoritmos de diff mais sofisticados, otimizações de atualização, Patch Flags, Block Tree, Static Hoisting e outras técnicas utilizadas pelo Vue 3 para reduzir drasticamente o trabalho do Renderer e alcançar alta performance.
