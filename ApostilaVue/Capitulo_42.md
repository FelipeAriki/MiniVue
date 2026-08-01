# Capítulo 42 — Vue Compiler Internals: Como Templates São Transformados em Código JavaScript

> **Objetivo:** compreender profundamente como funciona o **Compiler do Vue 3**. Ao final deste capítulo você entenderá como um template é convertido em uma Render Function, passando pelas etapas de **Parsing**, **AST**, **Transform**, **Optimization** e **Code Generation**. Você também implementará um compilador simplificado na sua MiniVue.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Explicar a arquitetura do Vue Compiler.
* Entender como funciona o Parser.
* Compreender a AST (Abstract Syntax Tree).
* Explicar o processo de Transform.
* Entender o Code Generator.
* Implementar um compilador simplificado.

---

# Pré-requisitos

* Capítulos 01 ao 41.

---

# Introdução

Até agora.

Estudamos.

O Runtime.

---

Mas.

Existe.

Outro.

Grande.

Pilar.

Do Vue.

---

O Compiler.

---

Sem ele.

Não existiriam.

Templates.

---

Quando escrevemos.

```vue
<div>

{{ mensagem }}

</div>
```

---

O navegador.

Não entende.

Templates Vue.

---

Ele entende.

JavaScript.

---

Então.

Quem transforma.

Esse template.

Em código?

---

Resposta.

O Compiler.

---

# O fluxo completo

```text
Template

↓

Parser

↓

AST

↓

Transform

↓

Optimize

↓

Code Generator

↓

Render Function
```

---

Tudo isso.

Acontece.

Antes.

Da aplicação.

Executar.

---

# O que é um Compiler?

Um Compiler.

Recebe.

Um código.

---

E produz.

Outro código.

---

Entrada.

```vue
<h1>Hello</h1>
```

---

Saída.

```javascript
render(){

return h("h1","Hello")

}
```

---

O Vue.

Não interpreta.

Templates.

Em Runtime.

---

Ele.

Os compila.

---

Por isso.

O desempenho.

É tão alto.

---

# Primeira etapa

## Parsing

O Parser.

Lê.

O template.

Caractere.

Por caractere.

---

Imagine.

```vue
<div>

<h1>

Vue

</h1>

</div>
```

---

O Parser.

Não cria.

DOM.

---

Ele apenas.

Lê.

Os tokens.

---

Fluxo.

```text
<

div

>

h1

...

```

---

É semelhante.

Ao funcionamento.

De um.

Compilador.

De linguagem.

---

# Tokens

Primeiro.

O texto.

É dividido.

Em unidades.

Chamadas.

```text
Tokens
```

---

Exemplo.

```vue
<div>Hello</div>
```

---

Resultado.

```text
<

div

>

Hello

</

div

>
```

---

Esses.

São os.

Tokens.

---

# AST

Depois.

Os Tokens.

São convertidos.

Em.

```text
AST

(Abstract Syntax Tree)
```

---

Ela representa.

A estrutura.

Do template.

---

Visualmente.

```text
Root

↓

div

↓

Text
```

---

Outro exemplo.

```vue
<div>

<span>

Hi

</span>

</div>
```

---

AST.

```text
Root

↓

div

↓

span

↓

Text
```

---

Observe.

Não existe.

HTML.

---

Existe.

Uma árvore.

De objetos.

---

# Estrutura

Simplificada.

```javascript
{

type:"Element",

tag:"div",

children:[...]

}
```

---

Texto.

```javascript
{

type:"Text",

content:"Hello"

}
```

---

Interpolação.

```javascript
{

type:"Interpolation",

content:"nome"

}
```

---

A AST.

É muito.

Mais rica.

---

Mas.

Essa ideia.

É suficiente.

Para começar.

---

# Por que usar AST?

Porque.

Manipular.

Objetos.

É muito.

Mais fácil.

Do que.

Manipular.

Texto.

---

Imagine.

Encontrar.

Todos.

Os elementos.

```vue
v-if
```

---

Na AST.

Basta.

Percorrer.

Os nós.

---

Muito mais.

Simples.

---

# Segunda etapa

## Transform

Agora.

O Vue.

Percorre.

Toda.

A AST.

---

Cada nó.

Pode ser.

Modificado.

---

Exemplo.

```vue
{{ nome }}
```

---

Transforma-se.

Em.

Um nó.

De expressão.

---

Outro exemplo.

```vue
v-if
```

---

É convertido.

Em lógica.

Condicional.

---

Outro.

```vue
v-for
```

---

Transforma-se.

Em.

Um loop.

---

# Exemplo

Template.

```vue
<li

v-for="item in itens"

>

{{ item }}

</li>
```

---

Depois.

Do Transform.

---

Temos.

Algo parecido.

Com.

```javascript
renderList(

itens,

item=>...

)
```

---

Observe.

O template.

Já começa.

A virar.

JavaScript.

---

# Otimizações

Durante.

O Transform.

O Compiler.

Também.

Aplica.

Otimizações.

---

Como.

Static Hoisting.

---

Patch Flags.

---

Block Tree.

---

Tudo.

Que estudamos.

No capítulo.

Anterior.

---

Essas.

Otimizações.

São inseridas.

Na AST.

---

# Code Generation

Agora.

Chegamos.

À última.

Etapa.

---

A AST.

É convertida.

Em código.

JavaScript.

---

Entrada.

```text
AST
```

---

Saída.

```javascript
function render(){

return createVNode(...)

}
```

---

Essa função.

É enviada.

Ao Runtime.

---

# Exemplo completo

Template.

```vue
<h1>

{{ nome }}

</h1>
```

---

Render Function.

Simplificada.

```javascript
function render(){

return createVNode(

"h1",

null,

toDisplayString(

nome

)

)

}
```

---

Observe.

Não existe.

Mais template.

---

Existe.

JavaScript.

Puro.

---

# toDisplayString()

Lembra.

Das interpolações?

---

O Compiler.

Gera.

Chamadas.

Para.

```javascript
toDisplayString()
```

---

Ela garante.

A conversão.

Para texto.

---

Exemplo.

```vue
{{ idade }}
```

---

Gera.

```javascript
toDisplayString(

idade

)
```

---

Assim.

Qualquer valor.

É exibido.

Corretamente.

---

# createVNode()

Também.

É inserido.

Pelo Compiler.

---

Template.

```vue
<div>

Hello

</div>
```

---

Resultado.

```javascript
createVNode(

"div",

null,

"Hello"

)
```

---

Quem executa.

Depois.

É o Runtime.

---

# Separação

Observe.

A arquitetura.

```text
Compiler

↓

Render Function

↓

Runtime

↓

DOM
```

---

O Compiler.

Nunca.

Manipula.

O DOM.

---

O Runtime.

Nunca.

Compila.

Templates.

---

Cada parte.

Possui.

Uma responsabilidade.

Muito clara.

---

# Single File Component

Quando.

Criamos.

```text
App.vue
```

---

O Vite.

Utiliza.

O compilador.

Do Vue.

---

Fluxo.

```text
App.vue

↓

Compiler

↓

JavaScript

↓

Bundle
```

---

Por isso.

Durante.

A execução.

---

Não existe.

Mais.

Arquivo.

`.vue`.

---

# Runtime Compiler

Existe.

Outra versão.

Do Vue.

---

Que contém.

O Compiler.

No navegador.

---

Ela permite.

Compilar.

Templates.

Em Runtime.

---

Por exemplo.

```javascript
app.component(

"Meu",

{

template:"<h1>Hi</h1>"

}
)
```

---

Essa abordagem.

É menos.

Performática.

---

Por isso.

Em produção.

Preferimos.

Compilar.

Durante.

O build.

---

# Como implementar?

Na MiniVue.

Podemos.

Começar.

Com.

Um Parser.

Muito simples.

---

Entrada.

```vue
<h1>Hello</h1>
```

---

Saída.

```javascript
{

tag:"h1",

text:"Hello"

}
```

---

Depois.

Criamos.

Um Generator.

---

```javascript
function generate(ast){

return `

h("${ast.tag}","${ast.text}")

`

}
```

---

Já temos.

Um compilador.

Muito básico.

---

Depois.

Podemos.

Adicionar.

* atributos;
* filhos;
* interpolações;
* diretivas;
* eventos;
* componentes.

---

Gradualmente.

Nossa MiniVue.

Passará.

A compilar.

Templates.

Como.

O Vue.

---

# Fluxo completo

```text
Template

↓

Parser

↓

Tokens

↓

AST

↓

Transform

↓

Optimize

↓

Code Generator

↓

Render Function

↓

Runtime

↓

DOM
```

---

# Performance

Grande parte.

Da velocidade.

Do Vue.

Existe.

Porque.

O Compiler.

Resolve.

Problemas.

Antes.

Da execução.

---

Quanto mais.

Informação.

Ele produz.

---

Mais simples.

Fica.

O Runtime.

---

Essa.

É uma.

Das maiores.

Vantagens.

Da arquitetura.

Do Vue.

---

# Código-fonte

Grande parte da implementação do Compiler está distribuída entre:

```text
packages/compiler-core
```

---

Especialmente.

```text
parse.ts
```

Responsável pelo Parser.

---

```text
transform.ts
```

Responsável pelas transformações da AST.

---

```text
codegen.ts
```

Responsável por gerar a Render Function.

---

Também vale estudar:

```text
ast.ts
```

---

```text
transformElement.ts
```

---

```text
transformExpression.ts
```

---

```text
transformText.ts
```

---

Esses arquivos representam o núcleo do processo de compilação do Vue.

---

# Curiosidade

O Compiler do Vue utiliza uma arquitetura baseada em **transformações independentes**. Cada recurso (`v-if`, `v-for`, componentes, eventos, interpolações, slots etc.) possui seu próprio transformador. Isso torna o compilador extremamente modular e facilita a evolução do framework sem precisar alterar toda a pipeline de compilação.

---

# Resumo

Neste capítulo aprendemos que:

* O Compiler transforma templates em JavaScript.
* O Parser converte texto em Tokens.
* Os Tokens são transformados em uma AST.
* A AST passa por transformações e otimizações.
* O Code Generator produz a Render Function.
* O Runtime apenas executa o código gerado.

---

# Exercícios

## Exercício 1

Implemente um Parser que reconheça elementos simples (`<div>Texto</div>`).

---

## Exercício 2

Crie uma estrutura de AST para representar elementos e textos.

---

## Exercício 3

Implemente um Code Generator que transforme essa AST em chamadas para `createVNode()`.

---

## Exercício 4

Adicione suporte a interpolações (`{{ mensagem }}`).

---

## Exercício 5

Leia os arquivos `parse.ts`, `transform.ts` e `codegen.ts` e identifique como a AST percorre toda a pipeline até se tornar uma Render Function.

---

# Desafio

Atualize sua **MiniVue** para suportar:

* Parser;
* Tokens;
* AST;
* Transform;
* Code Generator;
* geração automática de Render Functions.

---

# Projeto do capítulo

Ao final deste capítulo sua MiniVue deverá possuir um compilador funcional capaz de transformar templates simples em Render Functions executáveis, reproduzindo a arquitetura fundamental do Compiler do Vue.

---

# Checklist

* [ ] Sei explicar todas as etapas do Compiler.
* [ ] Entendi o papel da AST.
* [ ] Sei como ocorre o processo de Transform.
* [ ] Entendi como a Render Function é gerada.
* [ ] Minha MiniVue possui um compilador básico.

---

# Próximo capítulo

## **Capítulo 43 — Vue Compiler Deep Dive: Parser Internals, AST Avançada e Sistema de Transforms**

No próximo capítulo faremos um mergulho ainda mais profundo no Compiler do Vue. Estudaremos o Parser internamente, os diferentes tipos de nós da AST, o sistema de visitantes (*visitor pattern*), as transformações estruturais e como o Vue implementa recursos complexos como `v-if`, `v-for`, `v-model`, componentes, slots e eventos diretamente durante a compilação. Este será um dos capítulos mais avançados de toda a formação.
