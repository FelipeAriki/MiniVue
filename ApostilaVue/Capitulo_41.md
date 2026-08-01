# Capítulo 41 — Server-Side Rendering (SSR) e Hydration: Como o Vue Renderiza no Servidor

> **Objetivo:** compreender profundamente como funciona o **SSR (Server-Side Rendering)** do Vue 3. Ao final deste capítulo você entenderá como o Vue renderiza HTML no servidor, como ocorre o processo de **Hydration**, quais problemas causam *Hydration Mismatch*, como o Runtime trabalha em dois ambientes diferentes e como implementar um SSR simplificado na sua MiniVue.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Explicar a diferença entre CSR, SSR e SSG.
* Entender como o Vue renderiza HTML no servidor.
* Compreender o processo de Hydration.
* Identificar Hydration Mismatch.
* Entender Streaming SSR.
* Implementar um SSR simplificado na MiniVue.

---

# Pré-requisitos

* Capítulos 01 ao 40.

---

# Introdução

Imagine.

Que você.

Abra.

Uma aplicação.

Vue.

---

O navegador.

Recebe.

Algo assim.

```html
<body>

<div id="app"></div>

<script src="app.js"></script>

</body>
```

---

Depois.

Baixa.

Todo.

O JavaScript.

---

Só então.

A aplicação.

É renderizada.

---

Esse modelo.

É chamado.

```text
CSR

(Client Side Rendering)
```

---

# Problema

Imagine.

Uma aplicação.

Muito grande.

---

O navegador.

Precisa.

Baixar.

Todo.

O JavaScript.

---

Executar.

O Runtime.

---

Executar.

O Renderer.

---

Só depois.

Mostrar.

A página.

---

Resultado.

Primeira.

Renderização.

Mais lenta.

---

Principalmente.

Em conexões.

Lentas.

---

# Outro problema

Motores.

De busca.

Precisam.

Ler.

O HTML.

---

Se o HTML.

Está vazio.

```html
<div id="app"></div>
```

---

O robô.

Possui.

Pouco conteúdo.

Para indexar.

---

Isso.

Afeta.

SEO.

---

# A solução

Renderizar.

A página.

No servidor.

---

Assim.

O navegador.

Recebe.

HTML pronto.

---

Exemplo.

```html
<body>

<div id="app">

<h1>Produtos</h1>

<ul>

<li>Notebook</li>

<li>Mouse</li>

</ul>

</div>

</body>
```

---

Observe.

O HTML.

Já existe.

---

Antes mesmo.

Do JavaScript.

---

Isso.

É chamado.

```text
SSR

(Server Side Rendering)
```

---

# Fluxo CSR

```text
Browser

↓

Download JS

↓

Runtime

↓

Renderer

↓

HTML
```

---

# Fluxo SSR

```text
Browser

↓

Recebe HTML

↓

Mostra Página

↓

Download JS

↓

Hydration
```

---

Observe.

O HTML.

Chega.

Pronto.

---

Muito mais.

Rápido.

---

# Como funciona?

No servidor.

O Vue.

Não possui.

DOM.

---

Não existe.

```javascript
window
```

---

Nem.

```javascript
document
```

---

Existe apenas.

JavaScript.

---

Então.

O Vue.

Utiliza.

Outro Renderer.

---

Ao invés.

De criar.

Elementos.

Do DOM.

---

Ele cria.

Strings.

HTML.

---

Exemplo.

Template.

```vue
<h1>{{ titulo }}</h1>
```

---

No navegador.

```javascript
document.createElement(...)
```

---

No servidor.

Resultado.

```html
<h1>Vue 3</h1>
```

---

Tudo.

É gerado.

Como texto.

---

# Renderer do Servidor

Fluxo.

```text
Template

↓

Compiler

↓

Render Function

↓

SSR Renderer

↓

HTML
```

---

Observe.

Não existe.

DOM.

---

Apenas.

Strings.

---

# Exemplo

No servidor.

```javascript
renderToString(app)
```

---

Resultado.

```html
<div>

Olá Mundo

</div>
```

---

Esse HTML.

É enviado.

Ao navegador.

---

# Agora surge um problema

O HTML.

Existe.

---

Mas.

Ainda.

Não possui.

Eventos.

---

O botão.

```vue
<button @click="salvar">
```

---

Ainda.

Não funciona.

---

Por quê?

---

Porque.

O JavaScript.

Ainda.

Não foi.

Executado.

---

# Hydration

Quando.

O bundle.

É carregado.

---

O Vue.

Percorre.

Todo.

O HTML.

---

Ligando.

Cada.

VNode.

Ao.

DOM.

Existente.

---

Visualmente.

```text
HTML

↓

VNode

↓

Eventos

↓

Reatividade

↓

Aplicação Viva
```

---

Esse processo.

É chamado.

```text
Hydration
```

---

# Muito importante

Hydration.

Não recria.

O DOM.

---

Ela.

Reutiliza.

O HTML.

Já existente.

---

Isso.

É muito.

Mais rápido.

---

# Fluxo completo

```text
Servidor

↓

HTML

↓

Browser

↓

Render

↓

Download JS

↓

Hydration

↓

Aplicação
```

---

# Como a Hydration funciona?

Imagine.

O HTML.

```html
<h1>Vue</h1>
```

---

O Vue.

Cria.

O VNode.

---

Depois.

Compara.

O VNode.

Com.

O HTML.

Existente.

---

Se ambos.

Forem.

Iguais.

---

Apenas.

Conecta.

Os eventos.

---

Sem recriar.

Os elementos.

---

# Hydration Mismatch

Agora.

Imagine.

Que.

No servidor.

Foi gerado.

```html
<p>10</p>
```

---

Mas.

No cliente.

O estado.

É.

```html
<p>20</p>
```

---

Resultado.

```text
Hydration Mismatch
```

---

O Vue.

Detecta.

Que.

O HTML.

É diferente.

---

E exibe.

Um aviso.

No console.

---

# Principais causas

Utilizar.

```javascript
Math.random()
```

Durante.

A renderização.

---

Ou.

```javascript
Date.now()
```

---

Ou.

Dados.

Diferentes.

Entre.

Servidor.

E cliente.

---

Exemplo.

Servidor.

```javascript
Math.random()

↓

0.32
```

---

Cliente.

```javascript
Math.random()

↓

0.91
```

---

Resultado.

HTML.

Diferente.

---

# Como evitar?

Os dados.

Precisam.

Ser.

Determinísticos.

---

Ou.

Ser enviados.

Do servidor.

Para o cliente.

---

Assim.

Ambos.

Renderizam.

O mesmo.

Resultado.

---

# Streaming SSR

Outra.

Grande.

Otimização.

---

Imagine.

Uma página.

Muito grande.

---

Sem Streaming.

O servidor.

Precisa.

Gerar.

Tudo.

---

Depois.

Enviar.

---

Fluxo.

```text
Render Completo

↓

Enviar HTML
```

---

Com Streaming.

```text
Render Parcial

↓

Enviar

↓

Continuar

↓

Enviar

↓

Continuar
```

---

O navegador.

Começa.

A renderizar.

Muito antes.

---

Melhorando.

A percepção.

De velocidade.

---

# Suspense

Lembra.

Do capítulo.

Anterior?

---

Ele também.

Funciona.

Com SSR.

---

Enquanto.

Os dados.

São carregados.

---

O servidor.

Pode.

Renderizar.

Fallbacks.

---

Depois.

Enviar.

O restante.

---

# Estado Inicial

Imagine.

Uma Store.

Do Pinia.

---

Ela.

Precisa.

Existir.

Também.

No cliente.

---

Então.

O servidor.

Serializa.

O estado.

---

Exemplo.

```javascript
window.__INITIAL_STATE__
```

---

Depois.

O cliente.

Reconstrói.

As Stores.

Com.

Esses dados.

---

Esse processo.

É chamado.

```text
Hydration do Estado
```

---

# Nuxt

O framework.

Mais famoso.

Para SSR.

No Vue.

É.

```text
Nuxt
```

---

Ele automatiza.

* SSR;
* Hydration;
* Rotas;
* Code Splitting;
* Streaming;
* SEO.

---

Grande parte.

Do trabalho.

É feito.

Automaticamente.

---

# Como implementar?

Na MiniVue.

Podemos criar.

Um Renderer.

Que.

Ao invés.

De criar.

DOM.

---

Retorna.

Strings.

---

Exemplo.

```javascript
function renderToString(vnode){

return `<div>${vnode.children}</div>`

}
```

---

Depois.

No navegador.

Criamos.

Uma função.

```javascript
hydrate()

```

---

Ela.

Percorre.

O DOM.

Existente.

---

Associa.

Cada nó.

Ao VNode.

---

Depois.

Liga.

Eventos.

---

Assim.

Não precisamos.

Criar.

Novos.

Elementos.

---

# Fluxo interno

```text
Template

↓

Compiler

↓

VNode

↓

SSR Renderer

↓

HTML

↓

Browser

↓

Hydration

↓

Eventos

↓

Reatividade
```

---

# Performance

SSR.

Melhora.

Principalmente.

* First Contentful Paint (FCP);
* Largest Contentful Paint (LCP);
* SEO;
* percepção de velocidade.

---

Por outro lado.

Aumenta.

O trabalho.

No servidor.

---

Cada requisição.

Precisa.

Ser renderizada.

---

Por isso.

Nem toda.

Aplicação.

Precisa.

De SSR.

---

# Quando usar?

Utilize.

SSR.

Quando.

* SEO é importante;
* a primeira renderização deve ser rápida;
* páginas públicas precisam ser indexadas.

---

Prefira.

CSR.

Quando.

* aplicações internas;
* ERPs;
* sistemas administrativos;
* dashboards autenticados.

---

Nesses casos.

O custo.

Do SSR.

Nem sempre.

Compensa.

---

# Código-fonte

Grande parte da implementação do SSR pode ser estudada em:

```text
packages/server-renderer
```

Especialmente:

```text
renderToString.ts
```

---

Também vale analisar:

```text
renderToNodeStream.ts
```

---

```text
render.ts
```

---

No Runtime, observe como o processo de hidratação é tratado em:

```text
packages/runtime-core
```

e

```text
packages/runtime-dom
```

Esses módulos mostram como o Vue reutiliza o HTML existente e conecta os VNodes ao DOM durante a Hydration.

---

# Curiosidade

A Hydration não significa "renderizar novamente". O objetivo é **reaproveitar o HTML gerado pelo servidor**, associando cada nó existente aos VNodes correspondentes e registrando eventos e reatividade. Quando ocorre um *Hydration Mismatch*, o Vue pode precisar descartar parte desse HTML e recriá-lo, reduzindo os benefícios do SSR.

---

# Resumo

Neste capítulo aprendemos que:

* CSR renderiza no navegador.
* SSR renderiza no servidor.
* O servidor gera HTML como texto.
* A Hydration conecta VNodes ao HTML existente.
* Dados diferentes entre servidor e cliente causam *Hydration Mismatch*.
* Streaming SSR melhora a entrega de páginas grandes.
* SSR melhora SEO e a primeira renderização.

---

# Exercícios

## Exercício 1

Implemente uma função `renderToString(vnode)` que converta uma árvore simples de VNodes em HTML.

---

## Exercício 2

Implemente uma função `hydrate()` que percorra um HTML existente e associe os VNodes correspondentes.

---

## Exercício 3

Crie um exemplo que gere um *Hydration Mismatch* utilizando `Math.random()` e explique por que ele ocorre.

---

## Exercício 4

Pesquise como o Nuxt serializa o estado inicial da aplicação e como ele é restaurado no cliente.

---

## Exercício 5

Leia os arquivos do `packages/server-renderer` e identifique onde ocorre a conversão dos VNodes em HTML.

---

# Desafio

Atualize sua **MiniVue** para suportar:

* `renderToString()`;
* Renderer para servidor;
* `hydrate()`;
* reaproveitamento do HTML existente;
* serialização do estado inicial.

---

# Projeto do capítulo

Ao final deste capítulo sua MiniVue deverá ser capaz de renderizar uma aplicação em HTML puro no servidor e reutilizar esse HTML no cliente por meio de um processo simplificado de Hydration, reproduzindo os conceitos fundamentais do SSR do Vue.

---

# Checklist

* [ ] Sei explicar a diferença entre CSR e SSR.
* [ ] Entendi como funciona a Hydration.
* [ ] Sei identificar um *Hydration Mismatch*.
* [ ] Entendi Streaming SSR.
* [ ] Minha MiniVue possui um Renderer básico para servidor.

---

# Próximo capítulo

## **Capítulo 42 — Vue Compiler Internals: Como Templates São Transformados em Código JavaScript**

No próximo capítulo estudaremos um dos temas mais avançados de todo o Vue 3: o **Compiler**. Você aprenderá como o Vue faz o *parsing* dos templates, constrói a AST (*Abstract Syntax Tree*), executa transformações, aplica otimizações como *Static Hoisting* e *Patch Flags* e, por fim, gera a *Render Function*. Também implementaremos um compilador simplificado na sua MiniVue.
