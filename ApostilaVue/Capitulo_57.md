# Capítulo 57 — Projeto Final VIII: Server-Side Rendering (SSR), Streaming, Hydration e a Arquitetura do Nuxt 3

> **Objetivo:** implementar um mecanismo completo de **Server-Side Rendering (SSR)** para a MiniVue, compreender o processo de **Hydration**, construir um renderer para o servidor, adicionar suporte a **Streaming SSR** e entender como essas tecnologias servem de base para frameworks como o **Nuxt 3**.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Entender profundamente o funcionamento do SSR.
* Implementar um Renderer para o servidor.
* Criar o processo de Hydration.
* Compreender Streaming SSR.
* Entender Partial Hydration e Islands Architecture.
* Conhecer a arquitetura utilizada pelo Nuxt 3.

---

# Pré-requisitos

* Capítulos 01 ao 56.

---

# Introdução

Até aqui.

Nossa MiniVue.

Sempre.

Executou.

No navegador.

---

O fluxo.

Era.

```text
Template

↓

Compiler

↓

VNode

↓

Renderer

↓

DOM
```

---

Mas.

Agora.

Queremos.

Renderizar.

Antes.

Do navegador.

---

No servidor.

---

Esse processo.

Recebe.

O nome.

De.

```text
SSR

Server Side Rendering
```

---

# O problema do SPA

Uma SPA.

Tradicional.

Segue.

Este fluxo.

```text
Servidor

↓

HTML vazio

↓

Download JavaScript

↓

Execução

↓

Renderização
```

---

Isso.

Significa.

Que.

O usuário.

Precisa.

Esperar.

---

Além disso.

Motores.

De busca.

Podem.

Ter dificuldade.

Para.

Indexar.

O conteúdo.

---

# Como o SSR resolve?

Agora.

O servidor.

Executa.

Nossa aplicação.

---

Fluxo.

```text
Servidor

↓

Render()

↓

HTML

↓

Cliente

↓

Tela
```

---

O usuário.

Recebe.

A página.

Pronta.

---

Muito antes.

Do JavaScript.

Ser carregado.

---

# O Renderer do servidor

No navegador.

Temos.

```javascript
render(vnode, container)
```

---

No servidor.

Criamos.

```javascript
renderToString(vnode)
```

---

O resultado.

É.

Uma string.

HTML.

---

Exemplo.

Entrada.

```vue
<App/>
```

---

Saída.

```html
<div>

<h1>MiniVue</h1>

<p>Hello SSR</p>

</div>
```

---

Nenhum.

DOM.

É criado.

---

Apenas.

Texto.

---

# Criando o SSR Renderer

Na MiniVue.

Podemos.

Criar.

```text
runtime-ssr/

renderer.js
```

---

A função.

Principal.

Será.

```javascript
renderToString(
vnode
)
```

---

Ela.

Percorre.

Toda.

A árvore.

---

Gerando.

HTML.

---

# Renderizando elementos

Imagine.

```vue
<div>

<p>Hello</p>

</div>
```

---

Geramos.

```html
<div>

<p>Hello</p>

</div>
```

---

O processo.

É.

Recursivo.

---

Cada filho.

Também.

É convertido.

Em.

HTML.

---

# Renderizando componentes

Agora.

Encontramos.

Um componente.

---

Executamos.

Sua.

Render Function.

---

Obtemos.

Outro.

VNode.

---

Continuamos.

A renderização.

---

Até chegar.

Apenas.

Em elementos.

HTML.

---

# Renderizando Props

Precisamos.

Converter.

As Props.

Em atributos.

---

Exemplo.

```vue
<input

disabled

id="name"
/>
```

---

Saída.

```html
<input

disabled

id="name"
/>
```

---

Tudo.

É convertido.

Em string.

---

# Escape de HTML

Outro.

Ponto.

Muito.

Importante.

---

Nunca.

Podemos.

Inserir.

Texto.

Sem.

Escape.

---

Exemplo.

```text
<
```

---

Precisa.

Virar.

```text
&lt;
```

---

Assim.

Evitamos.

Ataques.

Como.

XSS.

---

# Estado inicial

Nossa.

Aplicação.

Possui.

Estado.

---

Precisamos.

Enviá-lo.

Para.

O navegador.

---

Exemplo.

```javascript
window.__INITIAL_STATE__
```

---

Assim.

O cliente.

Recebe.

Os mesmos.

Dados.

Do servidor.

---

# O problema

A página.

Já existe.

---

Mas.

O JavaScript.

Ainda.

Não.

---

Como.

Conectar.

Os dois?

---

A resposta.

É.

Hydration.

---

# Hydration

Fluxo.

```text
Servidor

↓

HTML

↓

Browser

↓

Hydration

↓

Aplicação viva
```

---

Observe.

Que.

O DOM.

Já existe.

---

Não.

Criamos.

Tudo.

Novamente.

---

Apenas.

Conectamos.

Os eventos.

---

Criamos.

Os Effects.

---

Ligamos.

A Reactivity.

---

# Comparação

Renderização.

Tradicional.

```text
VNode

↓

DOM
```

---

Hydration.

```text
DOM existente

↓

VNode

↓

Eventos

↓

Reactivity
```

---

Muito.

Mais rápido.

---

# Detectando diferenças

Durante.

A Hydration.

Precisamos.

Comparar.

---

HTML.

---

VNode.

---

Caso.

Sejam.

Diferentes.

---

Executamos.

Um Patch.

---

Esse.

É chamado.

De.

```text
Hydration Mismatch
```

---

O Vue.

Mostra.

Avisos.

No modo.

De desenvolvimento.

---

# Streaming SSR

Agora.

Outra.

Grande.

Otimização.

---

Em vez.

De esperar.

Tudo.

Ficar pronto.

---

Enviamos.

O HTML.

Por partes.

---

Fluxo.

```text
Servidor

↓

Chunk 1

↓

Chunk 2

↓

Chunk 3
```

---

O navegador.

Começa.

A renderizar.

Enquanto.

Ainda.

Recebe.

Os dados.

---

Isso.

Reduz.

Muito.

O tempo.

De carregamento.

---

# Suspense + SSR

Agora.

Podemos.

Combinar.

Os dois.

---

```text
Promise

↓

Fallback

↓

Resolve

↓

Streaming
```

---

Assim.

O usuário.

Recebe.

Conteúdo.

Imediatamente.

---

Mesmo.

Com.

Componentes.

Assíncronos.

---

# Partial Hydration

Nem sempre.

Precisamos.

Hidratar.

Tudo.

---

Exemplo.

Uma página.

De blog.

---

Grande parte.

É estática.

---

Somente.

O formulário.

É interativo.

---

Então.

Hidratamos.

Apenas.

Essa parte.

---

Reduzindo.

Muito.

O JavaScript.

---

# Islands Architecture

Essa ideia.

Foi.

Levado.

Além.

---

Cada.

Parte.

Interativa.

É.

Uma ilha.

---

Visualmente.

```text
HTML

↓

Island

↓

HTML

↓

Island

↓

HTML
```

---

Cada ilha.

É.

Hidratada.

Separadamente.

---

Essa arquitetura.

É utilizada.

Por frameworks.

Modernos.

---

# Arquitetura do Nuxt 3

O Nuxt.

Combina.

Diversos.

Módulos.

---

```text
Vue

↓

Nitro

↓

SSR Renderer

↓

Streaming

↓

Hydration

↓

Application
```

---

O Nitro.

Executa.

A aplicação.

No servidor.

---

O Vue.

Continua.

Responsável.

Pela.

Hydration.

---

# Organização

Na MiniVue.

Criamos.

```text
packages/

runtime-ssr/

renderer.js

hydrate.js

stream.js
```

---

Cada módulo.

Possui.

Uma única.

Responsabilidade.

---

# Fluxo completo

Visualmente.

```text
Request

↓

Server

↓

renderToString()

↓

HTML

↓

Browser

↓

Hydration

↓

Reactivity

↓

Application
```

---

Essa.

É praticamente.

A arquitetura.

Utilizada.

Pelo Vue.

---

# Performance

SSR.

Melhora.

Diversos.

Indicadores.

---

Primeiro.

Conteúdo.

Visível.

---

SEO.

---

Tempo.

De carregamento.

---

Experiência.

Do usuário.

---

Entretanto.

Também.

Aumenta.

A complexidade.

Do servidor.

---

# Código-fonte

Os principais módulos do Vue relacionados ao SSR são:

```text
packages/server-renderer/
```

---

```text
packages/runtime-core/
```

---

```text
packages/runtime-dom/
```

---

Além deles, vale estudar os repositórios do **Nuxt 3** e do **Nitro**, que implementam o pipeline completo de renderização no servidor, streaming e inicialização da aplicação.

---

# Curiosidade

No Vue 3, o código responsável pelo **Server Renderer** é separado do Runtime utilizado no navegador. Isso permite que aplicações SSR não carreguem código desnecessário no cliente e que o mesmo componente possa ser renderizado tanto no servidor quanto no navegador utilizando renderizadores diferentes.

---

# Resumo

Neste capítulo aprendemos que:

* SSR renderiza HTML no servidor.
* `renderToString()` substitui o Renderer do navegador.
* Hydration conecta o HTML existente ao Runtime.
* Streaming SSR envia o HTML em partes.
* Partial Hydration reduz o JavaScript executado no cliente.
* Islands Architecture hidrata apenas regiões interativas.
* O Nuxt 3 utiliza esses conceitos para construir aplicações Vue de alta performance.

---

# Exercícios

## Exercício 1

Implemente uma função `renderToString()` capaz de converter VNodes em HTML.

---

## Exercício 2

Implemente um processo simplificado de Hydration que conecte eventos ao DOM existente.

---

## Exercício 3

Adicione suporte a Streaming SSR enviando o HTML em múltiplos blocos.

---

## Exercício 4

Implemente a serialização do estado inicial da aplicação para reutilização no cliente.

---

## Exercício 5

Crie um exemplo utilizando Partial Hydration em que apenas uma parte da página seja hidratada.

---

# Desafio

Transforme sua **MiniVue** em um framework universal implementando:

* Server-Side Rendering;
* `renderToString()`;
* Hydration;
* Streaming SSR;
* serialização do estado;
* suporte básico a Islands Architecture.

O objetivo é permitir que a mesma aplicação seja executada tanto no servidor quanto no navegador, aproveitando os benefícios de desempenho e SEO.

---

# Projeto do capítulo

Ao final deste capítulo sua MiniVue deverá suportar renderização no servidor, envio de HTML já pronto ao navegador, hidratação da aplicação e streaming de conteúdo, reproduzindo a arquitetura fundamental utilizada por frameworks modernos baseados em Vue.

---

# Checklist

* [ ] Implementei um Renderer para SSR.
* [ ] Entendi como funciona a Hydration.
* [ ] Adicionei suporte a Streaming SSR.
* [ ] Sei explicar Partial Hydration e Islands Architecture.
* [ ] Minha MiniVue pode executar tanto no servidor quanto no navegador.

---

# Próximo capítulo

## **Capítulo 58 — Projeto Final IX: Publicando sua MiniVue como um Framework Open Source**

No próximo capítulo você aprenderá a transformar a MiniVue em um projeto Open Source profissional: organização de monorepo, versionamento semântico, documentação, automação de releases, publicação no npm, GitHub Actions, governança do projeto e boas práticas para receber contribuições da comunidade.
