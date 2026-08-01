# Capítulo 40 — Performance no Vue 3: Como o Framework Consegue Ser Tão Rápido

> **Objetivo:** compreender profundamente todas as otimizações de performance implementadas no Vue 3. Ao final deste capítulo você entenderá como o Compiler gera código altamente otimizado, como o Renderer evita trabalho desnecessário e como o Runtime atualiza apenas o mínimo necessário para manter a interface sincronizada.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Explicar por que o Vue 3 é tão performático.
* Entender o papel do Compiler na otimização.
* Compreender Patch Flags.
* Entender Static Hoisting.
* Explicar Block Tree.
* Compreender Tree Flattening.
* Entender o Scheduler otimizado.
* Implementar algumas otimizações na MiniVue.

---

# Pré-requisitos

* Capítulos 01 ao 39.

---

# Introdução

Uma pergunta.

Muito comum.

É.

---

"Por que."

O Vue.

É tão rápido?

---

A resposta.

Não está.

Em apenas.

Um lugar.

---

Ela é resultado.

Da combinação.

De dezenas.

De otimizações.

---

Algumas acontecem.

Durante.

A compilação.

---

Outras.

Durante.

A renderização.

---

Outras.

Durante.

A atualização.

---

# Visão Geral

Todo o fluxo.

Do Vue.

Pode ser.

Representado.

Assim.

```text
Template

↓

Compiler

↓

Render Function

↓

VNode

↓

Renderer

↓

DOM
```

---

Cada etapa.

Possui.

Suas próprias.

Otimizações.

---

# Antes do Vue 3

Frameworks.

Mais antigos.

Seguiam.

Uma lógica.

Muito simples.

---

Sempre que.

O estado.

Mudava.

---

Comparavam.

Toda.

A árvore.

Virtual.

---

Mesmo.

Quando.

Apenas.

Um texto.

Mudava.

---

Visualmente.

```text
1000 Nodes

↓

Comparar

↓

1000 Nodes
```

---

Muito trabalho.

Desnecessário.

---

# A ideia do Vue 3

O Compiler.

Descobre.

O máximo.

Possível.

Em tempo.

De compilação.

---

Assim.

O Runtime.

Precisa.

Fazer.

Muito menos.

Trabalho.

---

Essa filosofia.

É conhecida.

Como.

```text
Compile-Time Optimization
```

---

# Primeira otimização

## Static Hoisting

Imagine.

```vue
<div>

<h1>Vue</h1>

<p>Curso</p>

<span>{{ contador }}</span>

</div>
```

---

Pergunta.

Quais elementos.

Nunca mudam?

---

Resposta.

```vue
<h1>Vue</h1>
```

---

E.

```vue
<p>Curso</p>
```

---

Somente.

O.

```vue
<span>
```

Muda.

---

Sem otimização.

Cada render.

Criaria.

Novamente.

Todos.

Os VNodes.

---

Algo parecido.

Com.

```javascript
render(){

return [

h("h1"),

h("p"),

h("span")

]

}
```

---

Mesmo.

Os elementos.

Estáticos.

---

Isso gera.

Alocação.

De memória.

Desnecessária.

---

# Static Hoisting

O Compiler.

Extrai.

Os elementos.

Estáticos.

---

Resultado.

```javascript
const _hoisted_1 =

createVNode("h1")

const _hoisted_2 =

createVNode("p")
```

---

Depois.

O Render.

Utiliza.

Esses objetos.

Já criados.

```javascript
return [

_hoisted_1,

_hoisted_2,

createVNode(...)

]
```

---

Observe.

Os dois.

Primeiros.

Nunca mais.

São recriados.

---

Benefícios.

* menos memória;
* menos objetos;
* menos GC;
* menos CPU.

---

# Patch Flags

Agora.

Imagine.

```vue
<div>

{{ nome }}

</div>
```

---

O Compiler.

Sabe.

Que somente.

O texto.

Pode mudar.

---

Então.

Ele gera.

Algo semelhante.

A.

```javascript
createVNode(

"div",

null,

nome,

TEXT
)
```

---

Esse.

`TEXT`.

É chamado.

De.

```text
Patch Flag
```

---

# O que é um Patch Flag?

É um número.

Que informa.

Ao Renderer.

O que.

Pode mudar.

---

Sem Patch Flags.

O Renderer.

Precisaria.

Comparar.

Tudo.

---

Com Patch Flags.

Ele sabe.

Exatamente.

O que atualizar.

---

Exemplo.

```text
TEXT
```

↓

Atualize.

Somente.

O texto.

---

Outro.

```text
CLASS
```

↓

Atualize.

Somente.

A classe.

---

Outro.

```text
STYLE
```

↓

Atualize.

Somente.

O estilo.

---

Outro.

```text
PROPS
```

↓

Atualize.

Apenas.

Algumas props.

---

# Exemplo

Imagine.

```vue
<div

:class="classe"

>

{{ nome }}

</div>
```

---

O Compiler.

Gera.

Algo parecido.

Com.

```text
CLASS

+

TEXT
```

---

O Renderer.

Não precisa.

Comparar.

O restante.

---

# Block Tree

Outra.

Grande.

Otimização.

---

Imagine.

Uma árvore.

Com.

1000 nós.

---

Apenas.

5.

São dinâmicos.

---

Por que.

Percorrer.

Todos?

---

Resposta.

Não faz sentido.

---

Então.

O Compiler.

Agrupa.

Os elementos.

Dinâmicos.

---

Visualmente.

Antes.

```text
1000 Nodes
```

---

Depois.

```text
1000 Nodes

↓

5 Dynamic Nodes
```

---

Isso.

É chamado.

De.

```text
Block Tree
```

---

Cada bloco.

Conhece.

Seus filhos.

Dinâmicos.

---

Assim.

O Renderer.

Percorre.

Apenas.

Eles.

---

# Tree Flattening

Outra otimização.

Muito importante.

---

Imagine.

Uma árvore.

Muito profunda.

```text
A

↓

B

↓

C

↓

D

↓

E
```

---

Percorrer.

Essa estrutura.

É caro.

---

O Compiler.

Cria.

Estruturas.

Mais planas.

---

Facilitando.

A navegação.

Durante.

O Patch.

---

Essa técnica.

É conhecida.

Como.

```text
Tree Flattening
```

---

# Cache de Handlers

Imagine.

```vue
<button

@click="salvar"

>
```

---

Sem otimização.

Cada render.

Criaria.

Uma nova.

Função.

---

```javascript
()=>salvar()
```

---

O Compiler.

Pode reutilizar.

A mesma.

Referência.

---

Assim.

Evita.

Novas.

Alocações.

---

# Cache de Expressões

Expressões.

Que nunca.

Mudam.

Também.

Podem.

Ser reutilizadas.

---

Menos.

Objetos.

Mais.

Performance.

---

# Scheduler

Lembra.

Do capítulo.

Sobre.

Scheduler?

---

Ele também.

É uma.

Grande.

Otimização.

---

Imagine.

```javascript
contador++

contador++

contador++
```

---

Sem Scheduler.

Teríamos.

3 renders.

---

Com Scheduler.

```text
3 alterações

↓

1 render
```

---

Economizando.

Muito.

Processamento.

---

# nextTick()

Também.

Participa.

Desse processo.

---

Ele espera.

A fila.

Ser processada.

---

Depois.

Executa.

O callback.

---

# Computed

Outra.

Otimização.

Importante.

---

Sem cache.

```javascript
total()
```

Seria.

Executado.

Sempre.

---

Com.

```javascript
computed()
```

---

O resultado.

É reutilizado.

Enquanto.

As dependências.

Não mudam.

---

# Watch

Também.

Evita.

Trabalho.

Desnecessário.

---

Ele executa.

Somente.

Quando.

A dependência.

Realmente muda.

---

# Chaves

Outro detalhe.

Fundamental.

---

Sempre.

Utilize.

```vue
:key
```

---

Isso.

Permite.

Que o Diff.

Reconheça.

Os elementos.

Corretamente.

---

Sem chave.

O algoritmo.

Precisa.

Fazer.

Mais trabalho.

---

# Fragment

Outro ganho.

Do Vue 3.

---

Agora.

Não existe.

Obrigação.

De um.

Elemento raiz.

---

Menos.

Nós.

Desnecessários.

No DOM.

---

# Teleport

Também.

Evita.

Estruturas.

Complexas.

No DOM.

---

Sem.

Perder.

Reatividade.

---

# Suspense

Permite.

Carregamento.

Assíncrono.

Sem bloquear.

Toda.

A interface.

---

# Async Components

Carregam.

Código.

Sob demanda.

---

Reduzindo.

O bundle.

Inicial.

---

# Compiler + Runtime

A grande.

Diferença.

Do Vue.

É.

---

O Compiler.

Ajuda.

O Runtime.

---

Quanto mais.

Informação.

O Compiler.

Descobre.

---

Menos.

O Runtime.

Precisa.

Descobrir.

Durante.

A execução.

---

Essa é.

Uma das.

Maiores.

Diferenças.

Em relação.

A bibliotecas.

Que fazem.

Mais trabalho.

Em Runtime.

---

# Fluxo completo

```text
Template

↓

Compiler

↓

Static Hoisting

↓

Patch Flags

↓

Block Tree

↓

Render Function

↓

Renderer

↓

Scheduler

↓

DOM
```

---

# MiniVue

Podemos.

Começar.

Com.

Uma otimização.

Simples.

---

Criar.

Um cache.

De nós.

Estáticos.

```javascript
const cache = new Map()
```

---

Depois.

Reutilizar.

Os mesmos.

VNodes.

---

Outra melhoria.

É adicionar.

Uma propriedade.

```javascript
patchFlag
```

---

Ao VNode.

---

Durante.

O Patch.

```javascript
if(

vnode.patchFlag===TEXT

){

// atualiza apenas o texto

}
```

---

Depois.

Podemos.

Adicionar.

Um sistema.

De blocos.

Com.

Lista.

De filhos.

Dinâmicos.

---

Mesmo.

Que simplificado.

---

Sua MiniVue.

Já ficará.

Muito próxima.

Da arquitetura.

Do Vue.

---

# Código-fonte

Grande parte das otimizações pode ser estudada em:

```text
packages/compiler-core
```

---

Especialmente.

```text
transformHoist.ts
```

---

```text
transformElement.ts
```

---

```text
patchFlags.ts
```

---

E no Runtime.

```text
packages/runtime-core/renderer.ts
```

---

Esses módulos mostram como o Compiler produz estruturas otimizadas e como o Renderer utiliza essas informações para minimizar o trabalho durante cada atualização.

---

# Curiosidade

Grande parte da velocidade do Vue 3 não vem de um algoritmo de Diff "mais rápido", mas sim do fato de que **o Runtime evita executar o Diff completo** sempre que possível. O Compiler entrega informações suficientes para que o Renderer saiba exatamente onde precisa trabalhar.

---

# Resumo

Neste capítulo aprendemos que:

* O Vue 3 combina otimizações de Compiler e Runtime.
* Static Hoisting evita recriar VNodes estáticos.
* Patch Flags informam exatamente o que pode mudar.
* Block Tree permite percorrer apenas nós dinâmicos.
* Tree Flattening reduz o custo de navegação na árvore.
* Scheduler agrupa atualizações em um único render.
* Computed, Watch e Async Components também contribuem para a performance.

---

# Exercícios

## Exercício 1

Analise o código gerado pelo Compiler para um componente simples e identifique quais VNodes foram hoisted.

---

## Exercício 2

Pesquise os principais `PatchFlags` utilizados pelo Vue e descreva em quais situações cada um é aplicado.

---

## Exercício 3

Implemente um cache de VNodes estáticos na sua MiniVue.

---

## Exercício 4

Adicione uma propriedade `patchFlag` aos VNodes e utilize-a para otimizar o processo de atualização.

---

## Exercício 5

Leia o `renderer.ts` e identifique em quais momentos o Vue utiliza `dynamicChildren` para evitar percorrer toda a árvore.

---

# Desafio

Atualize sua **MiniVue** para suportar:

* cache de VNodes estáticos;
* Patch Flags simplificados;
* Block Tree básico;
* otimização do processo de Patch;
* Scheduler com atualização em lote.

---

# Projeto do capítulo

Ao final deste capítulo sua MiniVue deverá possuir as primeiras otimizações de performance inspiradas diretamente na arquitetura do Vue 3, reduzindo trabalho desnecessário durante a renderização e aproximando seu funcionamento do framework oficial.

---

# Checklist

* [ ] Sei explicar por que o Vue 3 é tão rápido.
* [ ] Entendi Static Hoisting.
* [ ] Sei como funcionam Patch Flags.
* [ ] Entendi Block Tree e Tree Flattening.
* [ ] Minha MiniVue possui otimizações básicas de performance.

---

# Próximo capítulo

## **Capítulo 41 — Server-Side Rendering (SSR) e Hydration: Como o Vue Renderiza no Servidor**

No próximo capítulo estudaremos uma das partes mais avançadas do ecossistema Vue: **SSR (Server-Side Rendering)**. Você aprenderá como o Vue gera HTML no servidor, como funciona o processo de **Hydration**, quais problemas podem causar *hydration mismatch*, como o Renderer possui modos diferentes para servidor e cliente e como construir um mecanismo simplificado de SSR na sua MiniVue.
