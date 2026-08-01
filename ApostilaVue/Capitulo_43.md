# Capítulo 43 — Vue Compiler Deep Dive: Parser Internals, AST Avançada e Sistema de Transforms

> **Objetivo:** compreender em profundidade a arquitetura interna do **Compiler do Vue 3**. Ao final deste capítulo você entenderá como o Parser funciona caractere por caractere, conhecerá todos os principais tipos de nós da AST, entenderá o Visitor Pattern utilizado pelo Compiler, verá como diretivas como `v-if`, `v-for`, `v-model`, `v-on`, `v-bind` e `slot` são transformadas em JavaScript e implementará uma versão muito mais completa do compilador na sua MiniVue.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Entender como o Parser do Vue funciona internamente.
* Conhecer todos os principais tipos de nós da AST.
* Compreender o Visitor Pattern.
* Entender o sistema de Transforms.
* Explicar como cada diretiva do Vue é compilada.
* Evoluir significativamente o Compiler da MiniVue.

---

# Pré-requisitos

* Capítulos 01 ao 42.

---

# Introdução

No capítulo anterior.

Vimos.

A pipeline.

Completa.

```text
Template

↓

Parser

↓

AST

↓

Transform

↓

Code Generator
```

---

Agora.

Vamos.

Entrar.

Dentro.

Do Compiler.

---

Vamos estudar.

O mesmo.

Código.

Que a equipe.

Do Vue.

Mantém.

---

# O Parser

Imagine.

Este template.

```vue
<div>

<h1>{{ titulo }}</h1>

<button @click="salvar">

Salvar

</button>

</div>
```

---

O Parser.

Não lê.

Linha.

Por linha.

---

Ele percorre.

Cada.

Caractere.

---

Algo parecido.

Com.

```text
<

d

i

v

>

...

```

---

Enquanto.

Percorre.

Vai alterando.

Seu estado.

---

Visualmente.

```text
Cursor

↓

<div>Hello</div>
 ^
```

---

Depois.

```text
<div>Hello</div>
    ^
```

---

Até.

Consumir.

Todo.

O template.

---

# Parsing Context

Internamente.

Existe.

Um objeto.

Semelhante.

A este.

```javascript
const context={

source,

offset,

line,

column

}
```

---

Ele informa.

Em qual.

Posição.

O Parser.

Está.

---

Toda função.

Recebe.

Esse contexto.

---

E avança.

O cursor.

---

# Tipos de Nós

Durante.

O Parsing.

Diversos.

Tipos.

De nós.

São criados.

---

Entre.

Os principais.

Temos.

```text
ROOT
```

---

```text
ELEMENT
```

---

```text
TEXT
```

---

```text
COMMENT
```

---

```text
INTERPOLATION
```

---

```text
ATTRIBUTE
```

---

```text
DIRECTIVE
```

---

Cada um.

Representa.

Uma parte.

Do template.

---

# Exemplo

```vue
<div class="card">

{{ nome }}

</div>
```

---

AST.

Simplificada.

```text
ROOT

↓

ELEMENT(div)

↓

ATTRIBUTE(class)

↓

INTERPOLATION(nome)
```

---

Observe.

Que.

Os atributos.

Também.

São nós.

---

# Element Node

Exemplo.

```javascript
{

type:"ELEMENT",

tag:"div",

props:[],

children:[]

}
```

---

# Text Node

```javascript
{

type:"TEXT",

content:"Hello"

}
```

---

# Interpolation Node

```javascript
{

type:"INTERPOLATION",

content:"nome"

}
```

---

# Directive Node

```javascript
{

type:"DIRECTIVE",

name:"if"

}
```

---

Ou.

```javascript
{

type:"DIRECTIVE",

name:"for"

}
```

---

# Visitor Pattern

Depois.

Que a AST.

Foi criada.

---

Ela precisa.

Ser percorrida.

---

O Vue.

Utiliza.

Uma técnica.

Muito conhecida.

---

```text
Visitor Pattern
```

---

O algoritmo.

Visita.

Cada nó.

---

Visualmente.

```text
Root

↓

Element

↓

Interpolation

↓

Text
```

---

Cada visitante.

Pode.

Modificar.

O nó.

---

Adicionar.

Informações.

---

Ou removê-lo.

---

# Traversal

O percurso.

É.

Profundidade.

Primeiro.

---

```text
A

↓

B

↓

C

↓

D
```

---

Somente.

Depois.

Volta.

Para.

O pai.

---

Esse algoritmo.

É conhecido.

Como.

```text
DFS

Depth First Search
```

---

# Transform Pipeline

Depois.

Do Parsing.

Começa.

A etapa.

De Transform.

---

Ela.

É composta.

Por.

Diversos.

Transformers.

---

Visualmente.

```text
AST

↓

transformIf()

↓

transformFor()

↓

transformSlot()

↓

transformElement()

↓

transformText()

↓

...
```

---

Cada.

Transformer.

Resolve.

Um problema.

Específico.

---

Essa arquitetura.

É extremamente.

Modular.

---

# Transform do v-if

Template.

```vue
<div

v-if="ativo"

>

Olá

</div>
```

---

Durante.

O Transform.

---

Esse nó.

Deixa.

De ser.

Um Element.

Simples.

---

E vira.

Uma estrutura.

Condicional.

---

Resultado.

Simplificado.

```javascript
ativo

?

createVNode(...)

:

null
```

---

Observe.

O template.

Já virou.

JavaScript.

---

# Transform do v-for

Template.

```vue
<li

v-for="item in lista"

>

{{ item }}

</li>
```

---

Transform.

↓

```javascript
renderList(

lista,

item=>...

)
```

---

O loop.

Agora.

É JavaScript.

---

# Transform do v-bind

Template.

```vue
<img

:src="foto"

>
```

---

Transforma-se.

Em.

```javascript
props:{

src:foto

}
```

---

# Transform do @click

Template.

```vue
<button

@click="salvar"

>
```

---

Transform.

↓

```javascript
props:{

onClick:salvar

}
```

---

Observe.

O símbolo.

```text
@click
```

---

Deixa.

De existir.

---

# Transform do v-model

Template.

```vue
<input

v-model="nome"

>
```

---

Transforma-se.

Em.

```javascript
modelValue

+

onUpdate:modelValue
```

---

Depois.

O Runtime.

Executa.

A sincronização.

---

# Transform de Slots

Template.

```vue
<Card>

<h1>Hello</h1>

</Card>
```

---

Transforma-se.

Em.

Funções.

---

```javascript
slots:{

default(){

...

}

}
```

---

Por isso.

Slots.

São.

Lazy.

---

Eles.

Só são.

Executados.

Quando.

Necessário.

---

# Exit Functions

Um detalhe.

Pouco conhecido.

---

Cada Transform.

Pode retornar.

Uma função.

---

Ela.

Será executada.

Depois.

Dos filhos.

---

Fluxo.

```text
Entrar

↓

Filhos

↓

Exit Function
```

---

Isso.

Permite.

Modificar.

A árvore.

Após.

Todo.

O processamento.

---

É uma.

Das partes.

Mais elegantes.

Do Compiler.

---

# Helpers

Durante.

Os Transforms.

---

O Compiler.

Descobre.

Quais.

Funções.

O Runtime.

Precisará.

---

Exemplo.

```javascript
toDisplayString()
```

---

Ou.

```javascript
renderList()
```

---

Ou.

```javascript
createVNode()
```

---

Esses.

Helpers.

São registrados.

Durante.

O Transform.

---

Depois.

São importados.

Automaticamente.

---

# Estrutura Geral

Visualmente.

```text
Template

↓

Parser

↓

AST

↓

Visitor

↓

TransformIf

↓

TransformFor

↓

TransformElement

↓

TransformText

↓

Helpers

↓

Code Generator
```

---

# Como implementar?

Na MiniVue.

Podemos.

Criar.

Uma lista.

De Transforms.

---

```javascript
const transforms=[

transformIf,

transformFor,

transformText

]
```

---

Depois.

Percorremos.

A AST.

---

```javascript
for(

const transform

of transforms

){

transform(node)

}
```

---

Cada.

Transform.

Pode alterar.

O nó.

---

Ou criar.

Novos.

Nós.

---

Gradualmente.

Nossa MiniVue.

Passará.

A possuir.

Um Compiler.

Baseado.

Na mesma.

Arquitetura.

Do Vue.

---

# Performance

A divisão.

Do Compiler.

Em pequenos.

Transforms.

Permite.

Adicionar.

Novos recursos.

Sem alterar.

Todo.

O compilador.

---

Essa arquitetura.

É um dos.

Principais motivos.

Pelos quais.

O Vue.

Continua.

Evoluindo.

Com facilidade.

---

# Código-fonte

Grande parte dessa arquitetura pode ser estudada em:

```text
packages/compiler-core/src/parse.ts
```

---

```text
packages/compiler-core/src/transform.ts
```

---

```text
packages/compiler-core/src/transforms
```

---

Dentro dessa pasta, os principais módulos são:

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

```text
vIf.ts
```

---

```text
vFor.ts
```

---

```text
vSlot.ts
```

---

Cada arquivo é responsável por uma transformação específica da AST, tornando o compilador altamente modular.

---

# Curiosidade

O Compiler do Vue é baseado em uma arquitetura de **plugins internos**. Cada transform funciona de forma muito semelhante a um plugin, recebendo um nó da AST, podendo modificá-lo, registrar Helpers e retornar uma função de saída (*Exit Function*). Essa estratégia permite que novas funcionalidades sejam adicionadas ao compilador com impacto mínimo no restante da pipeline.

---

# Resumo

Neste capítulo aprendemos que:

* O Parser percorre o template caractere por caractere.
* A AST possui diversos tipos de nós.
* O Compiler utiliza o Visitor Pattern para percorrer a árvore.
* Cada diretiva possui seu próprio Transform.
* Os Helpers do Runtime são registrados durante a compilação.
* O sistema de Exit Functions permite transformações em duas fases.

---

# Exercícios

## Exercício 1

Implemente novos tipos de nós (`ATTRIBUTE`, `DIRECTIVE` e `COMMENT`) na AST da sua MiniVue.

---

## Exercício 2

Implemente um Visitor que percorra toda a árvore utilizando DFS.

---

## Exercício 3

Crie um `transformIf()` que converta um nó com `v-if` em uma estrutura condicional.

---

## Exercício 4

Implemente um `transformFor()` que converta um `v-for` em uma chamada para `renderList()`.

---

## Exercício 5

Leia os módulos `vIf.ts`, `vFor.ts` e `transformElement.ts` e identifique em qual momento cada um registra os Helpers utilizados posteriormente pelo Code Generator.

---

# Desafio

Atualize sua **MiniVue** para suportar:

* novos tipos de nós da AST;
* Visitor Pattern;
* pipeline de Transforms;
* `v-if`;
* `v-for`;
* `v-bind`;
* `v-on`;
* `v-model`;
* sistema de Helpers.

---

# Projeto do capítulo

Ao final deste capítulo sua MiniVue deverá possuir uma pipeline de compilação modular, capaz de transformar diretivas em estruturas JavaScript e gerar uma AST muito mais próxima da utilizada pelo Vue 3.

---

# Checklist

* [ ] Entendi como o Parser percorre o template.
* [ ] Conheço os principais tipos de nós da AST.
* [ ] Sei explicar o Visitor Pattern.
* [ ] Entendi como funcionam os Transforms do Compiler.
* [ ] Minha MiniVue possui uma pipeline modular de compilação.

---

# Próximo capítulo

## **Capítulo 44 — Runtime Dom Internals: Como o Vue Manipula o DOM Real**

No próximo capítulo estudaremos o **Runtime DOM**, a camada responsável por conectar o Runtime Core ao navegador. Você entenderá como o Vue cria elementos, atualiza atributos, registra eventos, manipula classes e estilos, trata propriedades especiais (`value`, `checked`, `selected`), implementa o sistema de eventos, otimiza operações no DOM e separa completamente a lógica do Runtime Core da plataforma em que está sendo executado. Este será o início do estudo da arquitetura multiplataforma do Vue.
