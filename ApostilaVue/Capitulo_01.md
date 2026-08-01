# Capítulo 01 — Como o JavaScript e o Navegador Realmente Funcionam

> **Objetivo:** compreender o ambiente de execução do JavaScript antes de iniciar os estudos sobre Vue. Todo o funcionamento do Vue depende destes conceitos.

---

# Objetivos do capítulo

Ao concluir este capítulo você será capaz de:

* Explicar como uma página web é carregada.
* Diferenciar JavaScript da Engine JavaScript.
* Explicar o papel do navegador.
* Entender o que são Web APIs.
* Entender como o DOM é criado.
* Compreender por que o Vue depende desses conceitos.
* Preparar a base para Event Loop e Reatividade.

---

# Pré-requisitos

Nenhum.

Este é o primeiro capítulo técnico do livro.

---

# O maior erro de quem começa Vue

A maioria aprende algo parecido com isso:

```vue
<script setup>
import { ref } from 'vue'

const contador = ref(0)
</script>
```

E pensa:

> "Legal, o Vue atualiza a tela automaticamente."

Mas...

**Como?**

Quem atualizou?

Quando?

Quem percebeu a mudança?

Quem alterou o HTML?

Quem decidiu quando renderizar?

O navegador?

O JavaScript?

O Vue?

Todas essas perguntas começam a ser respondidas neste capítulo.

---

# Antes do Vue existe o navegador

Quando acessamos um site:

```
https://meusite.com
```

O navegador faz muito mais do que apenas mostrar HTML.

Ele possui dezenas de componentes internos.

Simplificando:

```
                Navegador
                     │
      ┌──────────────┼──────────────┐
      │              │              │
      ▼              ▼              ▼
 JavaScript      Render Engine    Web APIs
   Engine
```

Cada um possui uma responsabilidade diferente.

---

# O ciclo completo de uma página

Quando você acessa:

```
https://meusite.com
```

O navegador executa algo parecido com:

```
Requisição HTTP

↓

Recebe HTML

↓

Parser HTML

↓

Cria o DOM

↓

Baixa CSS

↓

Cria CSSOM

↓

Baixa JavaScript

↓

Executa JavaScript

↓

Renderiza a página

↓

Usuário interage

↓

JavaScript responde

↓

Nova renderização
```

Todo framework frontend trabalha em cima desse fluxo.

Vue não é diferente.

---

# O que é JavaScript?

JavaScript é apenas uma linguagem.

Ele define:

* Sintaxe
* Operadores
* Objetos
* Funções
* Classes
* Promises
* Proxy
* Reflect
* Arrays
* Maps

Mas...

JavaScript NÃO define:

* HTML
* DOM
* fetch
* localStorage
* setTimeout
* requestAnimationFrame
* addEventListener

Isso costuma surpreender muita gente.

---

# O que é uma Engine JavaScript?

A linguagem precisa de alguém para executá-la.

Essa "alguém" é a Engine.

Exemplos:

| Navegador | Engine         |
| --------- | -------------- |
| Chrome    | V8             |
| Edge      | V8             |
| Firefox   | SpiderMonkey   |
| Safari    | JavaScriptCore |

A Engine recebe seu código.

Exemplo:

```js
const soma = 10 + 20

console.log(soma)
```

Ela interpreta.

Compila.

Otimiza.

Executa.

---

# JavaScript não conhece o DOM

Imagine executar isto:

```js
document.querySelector("h1")
```

Parece JavaScript.

Mas não é.

A variável `document` não existe na linguagem.

Quem cria isso é o navegador.

---

# O ambiente de execução

O navegador entrega vários objetos prontos.

Exemplo:

```
window

document

navigator

history

location

fetch

localStorage

sessionStorage

console

setTimeout

setInterval
```

Todos são disponibilizados pelo navegador.

---

# As Web APIs

As Web APIs são funcionalidades disponibilizadas pelo navegador.

Alguns exemplos:

```
DOM API

Fetch API

History API

Storage API

Clipboard API

Canvas API

Geolocation API

WebSocket API

Intersection Observer API

Mutation Observer API
```

O Vue utiliza várias delas.

---

# Exemplo

Quando fazemos:

```js
setTimeout(() => {
    console.log("Olá")
},1000)
```

O JavaScript não controla o relógio.

O navegador faz isso.

Fluxo:

```
JavaScript

↓

setTimeout()

↓

Web API

↓

1 segundo

↓

Callback

↓

Event Loop

↓

JavaScript executa callback
```

Esse assunto será aprofundado no próximo capítulo.

---

# O DOM

DOM significa:

Document Object Model

O navegador transforma:

```html
<body>

<h1>Título</h1>

<p>Texto</p>

</body>
```

em algo parecido com:

```
Document

└── html

    └── body

        ├── h1

        └── p
```

Essa árvore pode ser modificada.

---

# Exemplo

HTML

```html
<h1>Vue</h1>
```

JavaScript

```js
const titulo = document.querySelector("h1")

titulo.textContent = "Vue 3"
```

O navegador atualiza o DOM.

Depois redesenha a tela.

---

# Onde o Vue entra?

Imagine isto:

```vue
<h1>{{ nome }}</h1>
```

Quando:

```js
nome.value = "Felipe"
```

acontece algo parecido com:

```
Estado mudou

↓

Vue detecta

↓

Agenda atualização

↓

Virtual DOM

↓

Diff

↓

DOM real

↓

Renderização
```

Tudo isso depende do navegador.

---

# Renderização

Após alterar o DOM:

```js
titulo.textContent = "Novo"
```

o navegador ainda precisa:

* recalcular estilos;
* recalcular layout;
* pintar pixels;
* enviar para GPU.

Esse processo possui custo.

Por isso o Vue evita alterações desnecessárias.

---

# Por que Virtual DOM existe?

Imagine atualizar 300 elementos.

Sem Virtual DOM.

```
JavaScript

↓

DOM

↓

DOM

↓

DOM

↓

DOM

↓

DOM
```

Muitas operações.

Com Virtual DOM.

```
Estado

↓

Virtual DOM

↓

Comparação

↓

Somente diferenças

↓

DOM
```

Muito mais eficiente.

Esse conceito será estudado profundamente mais adiante.

---

# Relação entre navegador e Vue

Sem navegador:

* não existe DOM;
* não existe renderização;
* não existe Web API;
* não existe interação.

O Vue apenas organiza tudo isso.

---

# Resumo

Até aqui aprendemos que:

* JavaScript é apenas uma linguagem.
* A Engine executa essa linguagem.
* O navegador fornece Web APIs.
* O DOM representa a página.
* O Vue trabalha sobre o DOM.
* O navegador continua responsável pela renderização.

---

# Exercícios

## Exercício 1

Explique com suas palavras a diferença entre:

* JavaScript
* Engine
* Navegador

---

## Exercício 2

Liste pelo menos dez Web APIs.

---

## Exercício 3

Desenhe a árvore DOM desta página.

```html
<body>

<header>

<h1></h1>

</header>

<main>

<section>

<p></p>

</section>

</main>

</body>
```

---

## Exercício 4

Pesquise qual Engine seu navegador utiliza.

---

# Desafio

Crie uma página contendo:

* botão;
* contador;
* título.

Sem utilizar Vue.

Apenas JavaScript puro.

Você deverá:

* alterar texto;
* alterar classes;
* alterar atributos;
* manipular eventos.

---

# Projeto do capítulo

Construir uma pequena aplicação em JavaScript puro que demonstre:

* manipulação do DOM;
* eventos;
* criação de elementos;
* remoção de elementos;
* atualização dinâmica.

Esse projeto servirá de comparação quando começarmos Vue.

---

# O que estudar antes do próximo capítulo

Leia na MDN:

* DOM
* Window
* Document
* Fetch API
* Web APIs (visão geral)

Não é necessário decorar.

Apenas entender o propósito.

---

# Checklist

* [ ] Sei explicar a diferença entre JavaScript e navegador.
* [ ] Entendi o papel da Engine.
* [ ] Sei o que são Web APIs.
* [ ] Sei o que é DOM.
* [ ] Entendi como uma página é carregada.
* [ ] Estou preparado para estudar Event Loop.

---

# Próximo capítulo

> **Capítulo 02 — Event Loop**

Você entenderá como o JavaScript executa código, como tarefas assíncronas são processadas e por que esse conhecimento é essencial para compreender `watch`, `nextTick`, atualizações do DOM e o scheduler do Vue.

---

**A partir do próximo capítulo vou manter exatamente esse nível (ou superior), tratando cada assunto como um capítulo de livro.**
