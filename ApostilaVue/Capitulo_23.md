# Capítulo 23 — `<script setup>` em Profundidade: Macros, Hoisting e Transformações do Compilador

> **Objetivo:** compreender completamente como o `<script setup>` funciona internamente. Ao final deste capítulo você será capaz de ler o código gerado pelo compilador, entender cada transformação realizada pelo `compiler-sfc` e explicar por que `<script setup>` é uma das maiores evoluções do Vue 3.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Explicar como funciona o `<script setup>`.
* Entender o processo de compilação.
* Compreender o Binding Analysis.
* Entender Hoisting.
* Explicar todas as Compiler Macros.
* Saber como o `return` do `setup()` é gerado.
* Entender as otimizações realizadas pelo compilador.

---

# Pré-requisitos

* Capítulos 01 ao 22.

---

# Recapitulando

Quando escrevemos:

```vue
<script setup>

const nome = "Felipe"

</script>
```

O Runtime nunca recebe isso.

O Compiler transforma tudo antes.

---

# A maior mentira do Vue

Muitos desenvolvedores acreditam que existe um componente executando exatamente esse código.

Não existe.

O navegador nunca executa um `<script setup>`.

Ele executa apenas JavaScript gerado pelo compilador.

---

# O Pipeline

```text
App.vue

↓

parse()

↓

compileScript()

↓

JavaScript

↓

setup()

↓

Runtime
```

---

# O primeiro passo

O compilador cria uma AST do `<script setup>`.

Código.

```javascript
const nome = "Felipe"

const idade = 24
```

AST.

```text
Program

↓

VariableDeclaration

↓

Identifier(nome)

↓

Literal("Felipe")
```

---

# Binding Analysis

Essa é uma das partes mais importantes.

O compilador percorre toda a AST procurando declarações.

Por exemplo.

```javascript
const nome = "Felipe"

const idade = 24

function salvar(){}

const contador = ref(0)
```

---

# Resultado

O Compiler registra.

```javascript
{

nome,

idade,

salvar,

contador

}
```

Esses são chamados de:

```text
Bindings
```

---

# O que é um Binding?

É qualquer variável declarada no `<script setup>`.

Por exemplo.

```javascript
const usuario = {}

let total = 0

function abrir(){}

class Pessoa{}
```

Tudo isso vira um Binding.

---

# Por que isso importa?

Porque o Compiler precisa descobrir.

Quais variáveis estarão disponíveis dentro do Template.

---

# Exemplo

```vue
<script setup>

const nome = "Felipe"

</script>

<template>

{{ nome }}

</template>
```

O Template utiliza.

```text
nome
```

Então o Compiler sabe.

Essa variável precisa ser exposta para o `render()`.

---

# Geração automática do return

Você nunca escreve.

```javascript
return {

nome

}
```

Mas o Compiler gera.

Resultado.

```javascript
setup(){

const nome="Felipe"

return{

nome

}

}
```

---

# Como ele sabe?

Graças ao Binding Analysis.

---

# Outro exemplo

```javascript
const nome = ref("Felipe")

const idade = ref(24)

const salvar = ()=>{}
```

Resultado.

```javascript
return{

nome,

idade,

salvar

}
```

Tudo automaticamente.

---

# Variáveis não utilizadas

Imagine.

```javascript
const nome = "Felipe"

const segredo = "123"

</script>

<template>

{{ nome }}

</template>
```

O Template nunca usa.

```text
segredo
```

O compilador consegue identificar isso durante a análise.

Isso permite otimizações posteriores pelo bundler.

---

# Hoisting

Outro conceito importante.

Imagine.

```javascript
const props = defineProps({

nome:String

})
```

Esse código não fica dentro do `setup()`.

Ele sobe.

Isso é chamado.

```text
Hoisting
```

---

# Visualizando

Antes.

```javascript
setup(){

defineProps()

}
```

Depois.

```javascript
props:{

...

}

setup(){

...
}
```

---

# Por que fazer isso?

Porque `props` pertence ao componente.

Não à execução do `setup()`.

---

# Hoisting de constantes

Imagine.

```javascript
const PI = 3.14
```

O Compiler pode mover essa constante para fora do `setup()`.

Resultado.

```javascript
const PI = 3.14

export default{

setup(){

...

}

}
```

Agora essa constante é criada apenas uma vez.

---

# Hoisting de imports

Imports também são movidos.

```javascript
import{

ref

}

from"vue"
```

Sempre aparecem no topo.

---

# Compiler Macros

Agora veremos cada Macro detalhadamente.

---

# defineProps()

Entrada.

```javascript
const props = defineProps({

nome:String,

idade:Number

})
```

Resultado.

```javascript
export default{

props:{

nome:String,

idade:Number

},

setup(props){

...

}

}
```

---

# Tipagem

Também funciona.

```typescript
const props = defineProps<{

nome:string

}>()
```

O compilador utiliza os tipos para gerar uma definição equivalente quando possível e alimentar a inferência do TypeScript.

---

# defineEmits()

Entrada.

```javascript
const emit = defineEmits([

"save",

"cancel"

])
```

Resultado.

```javascript
setup(

props,

{

emit

}

)
```

---

# defineExpose()

Entrada.

```javascript
defineExpose({

abrir,

fechar

})
```

Resultado.

```javascript
setup(

props,

{

expose

}

){

expose({

abrir,

fechar

})

}
```

---

# defineOptions()

Entrada.

```javascript
defineOptions({

name:"MeuComponente"

})
```

Resultado.

```javascript
export default{

name:"MeuComponente"

}
```

---

# defineSlots()

Essa Macro praticamente desaparece.

Ela serve principalmente para:

* tipagem;
* inferência;
* IDE.

---

# defineModel()

Entrada.

```javascript
const valor = defineModel()
```

Resultado simplificado.

```javascript
props:{

modelValue:...

}

emits:[

"update:modelValue"

]
```

---

# Múltiplos Models

Também funciona.

```javascript
defineModel(

"titulo"

)
```

Resultado.

```javascript
titulo

↓

update:titulo
```

---

# useSlots()

Diferente de `defineSlots()`.

```javascript
useSlots()
```

Existe em Runtime.

Ela precisa ser importada.

---

# useAttrs()

Também existe em Runtime.

```javascript
import{

useAttrs

}
```

Não é Macro.

---

# Macros x Runtime

Macros.

```text
defineProps

defineEmits

defineModel

defineExpose

defineSlots

defineOptions
```

Não existem em Runtime.

---

Runtime APIs.

```text
ref

reactive

computed

watch

watchEffect

provide

inject

useSlots

useAttrs
```

Precisam ser importadas.

---

# Imports automáticos?

Não.

O Vue não importa automaticamente APIs de Runtime.

Você ainda precisa escrever.

```javascript
import{

ref

}

from"vue"
```

---

# Top Level Await

Outra funcionalidade.

```javascript
const usuario =

await carregarUsuario()
```

O Compiler transforma isso.

Em um `async setup()`.

---

# Resultado

```javascript
async setup(){

const usuario=

await carregarUsuario()

}
```

---

# Misturando Script e Script Setup

É permitido.

```vue
<script>

export default{

inheritAttrs:false

}

</script>

<script setup>

...

</script>
```

O Compiler une ambos.

---

# Ordem

Primeiro.

```text
<script>
```

Depois.

```text
<script setup>
```

Tudo vira um único componente.

---

# O retorno automático

Imagine.

```javascript
const nome="Felipe"

const idade=24

function salvar(){}
```

O Compiler gera.

```javascript
return{

nome,

idade,

salvar

}
```

Automaticamente.

---

# O que não entra?

Imports.

```javascript
import{

ref

}
```

Não entram.

---

Macros.

Também não.

---

Funções privadas utilizadas apenas internamente também podem ser omitidas quando não são referenciadas pelo template nem precisam ser expostas.

---

# Template

Quando o Compiler encontra.

```vue
{{ nome }}
```

Ele procura.

```text
Binding(nome)
```

Encontrou.

↓

```javascript
_ctx.nome
```

---

# Se não encontrar?

Será tratado como:

* prop;
* componente;
* variável global permitida;
* erro de compilação (dependendo do contexto).

---

# Arquitetura

```text
Script Setup

↓

AST

↓

Binding Analysis

↓

Compiler Macros

↓

Hoisting

↓

Generate setup()

↓

Generate return()

↓

Runtime
```

---

# Arquivos reais

Grande parte dessa implementação está em.

```text
packages/compiler-sfc/src/compileScript.ts
```

Lá você encontrará funções responsáveis por:

* analisar bindings;
* processar macros;
* gerar `setup()`;
* criar o retorno automático;
* transformar `await`.

---

# Performance

O `<script setup>` não é apenas mais curto.

Ele também reduz trabalho em Runtime.

Como o compilador conhece todo o escopo antecipadamente, diversas decisões são tomadas durante o build.

---

# Comparando

Script tradicional.

```javascript
export default{

setup(){

...

}

}
```

Script Setup.

```vue
<script setup>

...

</script>
```

Após a compilação.

Os dois geram praticamente a mesma estrutura.

A diferença está na experiência de desenvolvimento e nas otimizações feitas durante a compilação.

---

# Curiosidade

Uma das etapas mais sofisticadas do `compileScript()` é a análise de escopo (*scope analysis*). O compilador precisa saber exatamente onde cada identificador foi declarado para evitar colisões de nomes, preservar o comportamento do JavaScript e decidir corretamente quais variáveis devem ser expostas ao template. É essa análise que permite ao Vue oferecer um `<script setup>` extremamente simples sem perder previsibilidade.

---

# Resumo

Neste capítulo aprendemos que:

* `<script setup>` é compilado antes da execução.
* O compilador realiza uma análise completa dos bindings.
* O `return` do `setup()` é gerado automaticamente.
* Compiler Macros são removidas durante a compilação.
* Hoisting reduz trabalho em Runtime.
* `top-level await` é transformado em `async setup()`.
* O Runtime nunca executa o código original do `<script setup>`.

---

# Exercícios

## Exercício 1

Explique a diferença entre `defineProps()` e `useAttrs()`.

---

## Exercício 2

Implemente uma análise simples de bindings em uma AST JavaScript.

---

## Exercício 3

Transforme manualmente um `<script setup>` em um `export default` com `setup()`.

---

## Exercício 4

Liste quais APIs do Vue são Compiler Macros e quais existem em Runtime.

---

## Exercício 5

Explique por que `top-level await` exige um `async setup()`.

---

# Desafio

Atualize sua **MiniVue Compiler** para suportar:

* análise de bindings;
* geração automática do `return`;
* uma macro simplificada de `defineProps()`;
* transformação de `top-level await` em `async setup()`.

---

# Projeto do capítulo

Ao final deste capítulo sua biblioteca deverá conseguir:

* analisar um `<script setup>` simplificado;
* identificar bindings;
* gerar automaticamente uma função `setup()`;
* transformar uma macro básica em opções do componente;
* produzir um componente equivalente ao escrito manualmente.

---

# Checklist

* [ ] Sei explicar como funciona o `<script setup>`.
* [ ] Entendi o Binding Analysis.
* [ ] Sei explicar o Hoisting.
* [ ] Entendi todas as Compiler Macros.
* [ ] Sei diferenciar Macros de APIs de Runtime.
* [ ] Entendi como o `return` é gerado automaticamente.
* [ ] Minha MiniVue já possui um compilador básico para `<script setup>`.

---

# Próximo capítulo

## **Capítulo 24 — Reatividade Avançada: Implementando `ref()`, `reactive()`, `computed()` e `watch()` do Zero**

Nos próximos capítulos deixaremos temporariamente o compilador para mergulhar na **engine de reatividade** do Vue em um nível ainda mais profundo. Você implementará versões completas de `ref()`, `reactive()`, `effect()`, `computed()`, `watch()`, `watchEffect()`, `effectScope()`, `customRef()`, `shallowRef()`, `readonly()` e do sistema de agendamento (*scheduler*), compreendendo praticamente todo o código do pacote `@vue/reactivity`. Esse será um dos capítulos mais técnicos e importantes de toda a jornada rumo à especialização em Vue 3.
