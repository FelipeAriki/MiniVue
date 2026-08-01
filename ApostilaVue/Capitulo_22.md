# Capítulo 22 — Single File Components (SFC): Como o Vue Compila Arquivos `.vue`

> **Objetivo:** compreender profundamente como um arquivo `.vue` é interpretado pelo Vue. Ao final deste capítulo você entenderá como funciona o `@vue/compiler-sfc`, como `<template>`, `<script>`, `<script setup>` e `<style>` são compilados e como ferramentas como Vite transformam um `.vue` em um módulo JavaScript comum.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Explicar o que é um SFC.
* Entender o funcionamento do `@vue/compiler-sfc`.
* Compreender o pipeline completo de compilação de um `.vue`.
* Explicar como funciona o `<script setup>`.
* Entender as Compiler Macros.
* Explicar como o Vite processa arquivos `.vue`.
* Compreender como os estilos são compilados.

---

# Pré-requisitos

* Capítulos 01 ao 21.

---

# O que é um SFC?

SFC significa.

```text
Single File Component
```

Ou.

```text
Componente em Arquivo Único
```

---

# Exemplo

```vue
<template>

<div>

{{ nome }}

</div>

</template>

<script setup>

const nome = "Felipe"

</script>

<style>

div{

color:red;

}

</style>
```

Esse arquivo parece simples.

Mas o Vue nunca executa esse arquivo diretamente.

---

# Uma dúvida comum

Quando escrevemos.

```text
App.vue
```

Como o navegador entende isso?

A resposta.

Ele não entende.

---

# O navegador entende apenas

* JavaScript
* CSS
* HTML

Não existe suporte nativo para:

```text
.vue
```

---

# Então quem faz isso?

Existe um compilador específico.

```text
@vue/compiler-sfc
```

---

# Arquitetura

Até agora estudamos.

```text
compiler-core
```

Agora adicionamos.

```text
compiler-sfc
```

---

# Diferença

compiler-core.

↓

Compila Templates.

---

compiler-sfc.

↓

Compila arquivos `.vue`.

---

# Fluxo completo

```text
App.vue

↓

compiler-sfc

↓

Template

↓

compiler-core

↓

Render Function

↓

JavaScript
```

---

# Primeira etapa

O SFC Compiler lê o arquivo inteiro.

```vue
<template>

...

</template>

<script>

...

</script>

<style>

...

</style>
```

---

# O Parser

Primeiro ele separa os blocos.

Resultado.

```javascript
{

template:{},

script:{},

styles:[]

}
```

---

# Exemplo

```vue
<template>

<div/>

</template>

<script>

export default{}

</script>

<style>

...

</style>
```

Produz.

```javascript
{

descriptor:{

template,

script,

styles

}

}
```

Esse objeto é chamado.

```text
SFC Descriptor
```

---

# O Descriptor

Simplificando.

```javascript
{

filename,

template,

script,

scriptSetup,

styles,

customBlocks

}
```

Ele representa completamente o arquivo.

---

# customBlocks

Também podemos criar.

```vue
<i18n>

...

</i18n>
```

Ou.

```vue
<docs>

...

</docs>
```

Tudo isso aparece em.

```javascript
customBlocks
```

Plugins podem utilizar esses blocos.

---

# Pipeline

Depois de criar o Descriptor.

Cada bloco segue um caminho diferente.

```text
Template

↓

compileTemplate()
```

---

```text
Script

↓

compileScript()
```

---

```text
Style

↓

compileStyle()
```

Tudo acontece separadamente.

---

# compileTemplate()

Recebe.

```vue
<template>

<h1>{{ nome }}</h1>

</template>
```

Resultado.

```javascript
function render(){

...
}
```

Esse é exatamente o Compiler que estudamos.

---

# compileStyle()

Recebe.

```css
div{

color:red;

}
```

Produz CSS processado.

---

# compileScript()

Agora começa a mágica.

---

# Antes do Vue 3

Escrevíamos.

```vue
<script>

export default{

setup(){

}

}

</script>
```

---

# Hoje

Escrevemos.

```vue
<script setup>

const nome="Felipe"

</script>
```

Mas...

Como isso funciona?

---

# Não existe Script Setup no Runtime

Essa é uma informação extremamente importante.

```text
<script setup>

não existe

em Runtime.
```

É apenas sintaxe.

---

# O Compiler transforma

Entrada.

```vue
<script setup>

const nome="Felipe"

</script>
```

Saída.

```javascript
export default{

setup(){

const nome="Felipe"

return{

nome

}

}

}
```

O Runtime nunca sabe que existiu `<script setup>`.

---

# Compiler Macros

Agora entra outro conceito.

Imagine.

```javascript
defineProps({

nome:String

})
```

Onde essa função está?

---

# Ela não existe

Você nunca faz.

```javascript
import{

defineProps

}
```

Porque.

Ela não existe em Runtime.

---

# Compiler Macro

```javascript
defineProps()
```

É uma instrução para o Compiler.

---

# Exemplo

Entrada.

```javascript
const props=

defineProps({

nome:String

})
```

Saída.

```javascript
export default{

props:{

nome:String

},

setup(props){

...

}

}
```

---

# defineEmits()

Entrada.

```javascript
const emit=

defineEmits([

"save"

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

abrir

})
```

Compiler.

↓

```javascript
expose({

abrir

})
```

---

# defineOptions()

Entrada.

```javascript
defineOptions({

inheritAttrs:false

})
```

Resultado.

```javascript
export default{

inheritAttrs:false

}
```

---

# defineSlots()

Outra Macro.

```javascript
defineSlots()
```

Serve apenas para tipos e análise do compilador.

Não existe em Runtime.

---

# defineModel()

Nova Macro.

Entrada.

```javascript
const valor=

defineModel()
```

Compiler.

↓

Gera automaticamente.

* prop;
* emit;
* sincronização.

Sem escrever todo o código manualmente.

---

# Todas as Macros

```text
defineProps

defineEmits

defineExpose

defineSlots

defineOptions

defineModel
```

São removidas pelo Compiler.

---

# Imports

Outro detalhe.

Imagine.

```javascript
import{

ref

}

from"vue"
```

O Compiler analisa.

Quais imports realmente são utilizados.

---

# Exemplo

```javascript
import{

ref,

computed,

watch

}
```

Se usamos apenas.

```javascript
ref
```

Os demais podem ser eliminados pelo bundler durante o tree-shaking.

---

# CSS Scoped

Agora outro assunto.

Imagine.

```vue
<style scoped>

button{

color:red;

}

</style>
```

Como isso funciona?

---

# O Compiler gera

HTML.

```html
<button

data-v-123abc

>
```

CSS.

```css
button[data-v-123abc]{

color:red;

}
```

Agora o CSS afeta apenas aquele componente.

---

# CSS Modules

Também podemos escrever.

```vue
<style module>

...
```

Resultado.

Um objeto.

```javascript
$style.botao
```

---

# Múltiplos Styles

Um componente pode possuir.

```vue
<style/>

<style scoped/>

<style module/>
```

Todos são compilados.

---

# Hot Module Replacement

Quando alteramos.

```text
App.vue
```

O Vite recompila.

Somente.

O bloco alterado.

---

# Exemplo

Mudou apenas.

```css
color:red;
```

O Vite recompila.

↓

Style.

Não recompila.

Template.

Nem Script.

---

# Template

Mudou apenas.

```vue
{{ nome }}
```

O Vite recompila apenas.

```text
Render Function
```

---

# Arquitetura do Vite

Quando fazemos.

```javascript
import App from "./App.vue"
```

Na verdade.

O Plugin Vue intercepta.

Esse import.

---

# Fluxo

```text
App.vue

↓

Plugin Vue

↓

compiler-sfc

↓

JavaScript

↓

ES Module
```

O navegador nunca vê um `.vue`.

---

# Resultado simplificado

Depois da compilação.

Temos algo parecido.

```javascript
const script={

setup(){}

}

script.render=render

export default script
```

Observe.

O Script e o Template tornam-se um único objeto.

---

# Template + Script

No arquivo original.

Eles parecem separados.

Depois da compilação.

Viram.

```javascript
{

setup,

render

}
```

---

# Depois

O Runtime recebe.

```javascript
{

setup,

render
}
```

Que é exatamente o formato estudado nos capítulos anteriores.

---

# Arquivos reais

Grande parte dessa implementação está em.

```text
packages/compiler-sfc/src
```

Arquivos importantes.

```text
parse.ts

compileScript.ts

compileTemplate.ts

compileStyle.ts
```

---

# Arquitetura completa

```text
App.vue

↓

parse()

↓

SFC Descriptor

↓

compileScript()

↓

compileTemplate()

↓

compileStyle()

↓

Merge

↓

Componente Final
```

---

# Comparando

Nossa MiniVue.

```text
Template

↓

Compiler
```

Vue.

```text
.vue

↓

Descriptor

↓

Compiler SFC

↓

Compiler Core

↓

Runtime
```

---

# Curiosidade

O `<script setup>` não torna o Runtime mais rápido.

A vantagem está na compilação: como o compilador conhece toda a estrutura do componente antes da execução, ele consegue eliminar código desnecessário, transformar macros em JavaScript comum e gerar uma função `setup()` mais enxuta do que seria escrita manualmente em muitos casos.

---

# Resumo

Neste capítulo aprendemos que:

* Um `.vue` é processado pelo `@vue/compiler-sfc`.
* O arquivo é dividido em um `SFC Descriptor`.
* Cada bloco (`template`, `script` e `style`) possui um compilador específico.
* `<script setup>` é apenas açúcar sintático compilado para uma função `setup()`.
* As Compiler Macros não existem em Runtime.
* O CSS Scoped funciona através de atributos gerados automaticamente.
* O Plugin Vue do Vite transforma um `.vue` em um módulo JavaScript comum.

---

# Exercícios

## Exercício 1

Desenhe o pipeline completo desde um arquivo `.vue` até a Render Function.

---

## Exercício 2

Explique por que `defineProps()` não precisa ser importado.

---

## Exercício 3

Mostre como um `<script setup>` simples é transformado em `export default`.

---

## Exercício 4

Explique como o CSS Scoped impede que estilos vazem para outros componentes.

---

## Exercício 5

Pesquise o código-fonte do `compileScript.ts` e identifique onde as Compiler Macros são processadas.

---

# Desafio

Atualize sua **MiniVue** para suportar um formato simplificado de SFC:

* leitura de um arquivo `.vue`;
* separação de `<template>`, `<script>` e `<style>`;
* compilação independente de cada bloco;
* união do resultado em um componente final.

---

# Projeto do capítulo

Ao final deste capítulo sua biblioteca deverá conseguir:

* interpretar um arquivo `.vue` simplificado;
* criar um Descriptor;
* compilar template e script separadamente;
* unir `setup()` e `render()` em um único componente;
* compreender o papel do `compiler-sfc` no ecossistema Vue.

---

# Checklist

* [ ] Sei explicar o que é um SFC.
* [ ] Entendi o papel do `@vue/compiler-sfc`.
* [ ] Sei explicar como funciona o `SFC Descriptor`.
* [ ] Entendi como `<script setup>` é compilado.
* [ ] Sei explicar as Compiler Macros.
* [ ] Entendi como funciona o CSS Scoped.
* [ ] Sei explicar como o Vite importa um arquivo `.vue`.

---

# Próximo capítulo

## **Capítulo 23 — `<script setup>` em Profundidade: Macros, Hoisting e Transformações do Compilador**

No próximo capítulo faremos um mergulho completo no `<script setup>`. Você entenderá exatamente como cada macro é transformada pelo compilador, como funcionam **hoisting**, análise estática de bindings, resolução de imports, geração automática do `return` do `setup()`, diferenças entre `<script>` e `<script setup>`, limitações, armadilhas e otimizações. Ao final, você será capaz de ler o código gerado pelo compilador praticamente linha por linha.
