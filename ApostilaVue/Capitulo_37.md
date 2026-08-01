# Capítulo 37 — Plugins no Vue 3: Arquitetura, Injeção Global e Extensibilidade do Framework

> **Objetivo:** compreender profundamente como funciona o sistema de **Plugins** do Vue 3. Ao final deste capítulo você entenderá como bibliotecas como **Pinia**, **Vue Router**, **Vue I18n** e **PrimeVue** são integradas ao framework, como funciona `app.use()`, como ocorre a instalação de plugins e como implementar um sistema semelhante na sua MiniVue.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Entender o propósito dos Plugins.
* Explicar como funciona `app.use()`.
* Compreender a instalação única de plugins.
* Utilizar `provide/inject` global.
* Entender `app.config.globalProperties`.
* Criar plugins profissionais.
* Implementar um sistema de plugins na MiniVue.

---

# Pré-requisitos

* Capítulos 01 ao 36.

---

# O problema

Imagine.

Uma biblioteca.

De tradução.

---

Sem Plugins.

Precisaríamos.

Importar.

Em todos.

Os componentes.

```javascript
import i18n from "./i18n"
```

---

Depois.

```javascript
i18n.translate(...)
```

---

Em centenas.

De arquivos.

---

Pouco prático.

---

Agora imagine.

Um Router.

---

Ou.

Um Store.

---

Ou.

Uma biblioteca.

De componentes.

---

Todos.

Precisariam.

Ser importados.

Manual.

Em cada.

Componente.

---

# A solução

O Vue.

Possui.

Um sistema.

De Plugins.

---

Que permite.

Instalar.

Recursos.

Uma única vez.

---

# Primeiro exemplo

```javascript
const app = createApp(App)

app.use(router)

app.mount("#app")
```

---

Outro.

```javascript
app.use(pinia)
```

---

Outro.

```javascript
app.use(i18n)
```

---

Outro.

```javascript
app.use(PrimeVue)
```

---

Observe.

Todos.

São instalados.

Da mesma forma.

---

# O que é um Plugin?

Um Plugin.

É um objeto.

Ou função.

Que recebe.

A aplicação.

---

Visualmente.

```text
createApp()

↓

App

↓

Plugin

↓

Configuração Global
```

---

# Estrutura

Um plugin.

Pode ser.

Uma função.

```javascript
function MeuPlugin(app){

}
```

---

Ou.

Um objeto.

```javascript
const MeuPlugin={

install(app){

}

}
```

---

O Vue.

Aceita.

Os dois.

---

# Internamente

Quando.

Chamamos.

```javascript
app.use(plugin)
```

---

O Vue.

Verifica.

Se existe.

```javascript
plugin.install
```

---

Se existir.

Executa.

```javascript
plugin.install(app)
```

---

Caso contrário.

Executa.

O próprio.

Plugin.

Como função.

---

# Simplificando

```javascript
function use(plugin){

if(plugin.install){

plugin.install(app)

}else{

plugin(app)

}

}
```

---

Essa é.

A ideia.

Principal.

---

# Instalação única

Imagine.

```javascript
app.use(router)

app.use(router)
```

---

O Router.

Não será.

Instalado.

Duas vezes.

---

O Vue.

Mantém.

Uma lista.

De plugins.

Já instalados.

---

Visualmente.

```text
Plugin A

✓
```

---

```text
Plugin B

✓
```

---

```text
Plugin A

Ignorado
```

---

# Implementação

Simplificada.

```javascript
const installed=

new Set()
```

---

```javascript
if(

installed.has(plugin)

){

return

}
```

---

Depois.

```javascript
installed.add(plugin)
```

---

Muito simples.

---

# app.config

Toda aplicação.

Possui.

Uma configuração.

Global.

---

```javascript
app.config
```

---

Dentro.

Existe.

```javascript
globalProperties
```

---

# Exemplo

```javascript
app.config.globalProperties.$api=api
```

---

Agora.

Em qualquer.

Componente.

```javascript
this.$api
```

---

Estará.

Disponível.

---

# Muito utilizado

Por.

Bibliotecas.

Como.

* Axios;
* APIs internas;
* Helpers;
* Clientes HTTP.

---

# provide()

Outra possibilidade.

É.

```javascript
app.provide(

"api",

api

)
```

---

Depois.

Qualquer.

Componente.

Pode fazer.

```javascript
inject("api")
```

---

Observe.

Não existe.

Import.

---

Tudo.

É injetado.

Automaticamente.

---

# Qual usar?

`globalProperties`.

É mais comum.

Na Options API.

---

`provide/inject`.

É recomendado.

Para.

Composition API.

---

Principalmente.

Quando.

Queremos.

Evitar.

Acoplamento.

---

# Exemplo real

Imagine.

Um plugin.

De autenticação.

```javascript
const AuthPlugin={

install(app){

app.provide(

"auth",

auth

)

}

}
```

---

Uso.

```javascript
const auth=

inject("auth")
```

---

Muito limpo.

---

# Outro exemplo

Plugin.

De tema.

```javascript
const ThemePlugin={

install(app){

app.provide(

"theme",

theme

)

}

}
```

---

Todos.

Os componentes.

Recebem.

O mesmo.

Objeto.

---

# Plugins famosos

## Vue Router

Instala.

* Router;
* navegação;
* RouterLink;
* RouterView.

---

## Pinia

Instala.

* Stores;
* contexto global;
* DevTools.

---

## PrimeVue

Instala.

* configuração global;
* temas;
* serviços;
* componentes.

---

## Vue I18n

Instala.

* tradução;
* locale;
* formatação.

---

Todos.

Utilizam.

`app.use()`.

---

# Passando opções

Também.

É possível.

Enviar.

Configurações.

---

```javascript
app.use(

plugin,

{

locale:"pt-BR"

}

)
```

---

O Vue.

Encaminha.

Esses parâmetros.

Para.

`install()`.

---

```javascript
install(

app,

options

){

}
```

---

# Exemplo

```javascript
const Logger={

install(

app,

options

){

console.log(

options

)

}

}
```

---

Uso.

```javascript
app.use(

Logger,

{

debug:true

}

)
```

---

# Componentes Globais

Um Plugin.

Também pode.

Registrar.

Componentes.

---

```javascript
app.component(

"Botao",

Botao

)
```

---

Assim.

Todos.

Os componentes.

Podem utilizá-lo.

---

Sem importar.

---

# Diretivas Globais

Também.

Pode registrar.

Diretivas.

---

```javascript
app.directive(

"focus",

focusDirective

)
```

---

Disponível.

Em toda.

A aplicação.

---

# Mixins Globais

Também.

Existe.

```javascript
app.mixin(...)
```

---

Embora.

Hoje.

Seja.

Pouco recomendado.

---

Composables.

Costumam.

Ser melhores.

---

# Fluxo

```text
Plugin

↓

install()

↓

App

↓

Provide

↓

Components

↓

Application
```

---

# MiniVue

Podemos criar.

Algo parecido.

Assim.

```javascript
class App{

plugins=

new Set()

}
```

---

Depois.

```javascript
use(plugin){

if(

this.plugins.has(plugin)

){

return

}

this.plugins.add(plugin)

plugin.install(

this

)

}
```

---

Já temos.

Um sistema.

Básico.

---

# Evoluindo

Adicionar.

```javascript
provide()
```

---

```javascript
component()
```

---

```javascript
directive()
```

---

```javascript
config
```

---

Sua MiniVue.

Ficará.

Muito próxima.

Da real.

---

# Fluxo completo

```text
createApp()

↓

App

↓

use()

↓

Plugin

↓

install()

↓

Configuração Global

↓

Componentes
```

---

# Performance

Os Plugins.

São instalados.

Apenas.

Uma vez.

---

Depois.

Os componentes.

Compartilham.

A mesma.

Instância.

---

Muito eficiente.

---

# Quando criar um Plugin?

Quando.

Seu código.

Precisa.

Ser reutilizado.

Por toda.

A aplicação.

---

Ou.

Quando.

Você está.

Criando.

Uma biblioteca.

---

Ou.

Quando.

Precisa.

Registrar.

Componentes.

Diretivas.

Ou serviços.

Globalmente.

---

# Quando NÃO criar

Não transforme.

Toda.

Função.

Em Plugin.

---

Se.

Um composable.

Resolve.

O problema.

---

Prefira.

O composable.

---

Plugins.

Devem.

Adicionar.

Capacidades.

À aplicação.

---

Não.

Resolver.

Lógica.

Local.

---

# Código-fonte

Grande parte da lógica está em:

```text
packages/runtime-core/src/apiCreateApp.ts
```

Vale estudar especialmente:

* `createApp()`;
* `app.use()`;
* `app.provide()`;
* registro de componentes;
* registro de diretivas;
* gerenciamento dos plugins instalados.

---

# Curiosidade

O método `app.use()` retorna a própria instância da aplicação. Isso permite o padrão de **encadeamento de chamadas (method chaining)**:

```javascript
createApp(App)
  .use(router)
  .use(pinia)
  .use(i18n)
  .mount("#app")
```

Esse estilo torna a inicialização da aplicação mais fluida e organizada.

---

# Resumo

Neste capítulo aprendemos que:

* Plugins adicionam funcionalidades globais à aplicação.
* `app.use()` aceita funções ou objetos com `install()`.
* O Vue impede instalações duplicadas.
* `globalProperties` disponibiliza recursos globais.
* `provide/inject` é a abordagem preferida para Composition API.
* Bibliotecas como Pinia e Vue Router utilizam esse mecanismo.

---

# Exercícios

## Exercício 1

Crie um plugin que registre um objeto `api` global utilizando `provide()`.

---

## Exercício 2

Crie um plugin que registre um componente global.

---

## Exercício 3

Implemente um plugin que registre uma diretiva `v-focus`.

---

## Exercício 4

Adicione suporte a opções (`app.use(plugin, options)`).

---

## Exercício 5

Leia `packages/runtime-core/src/apiCreateApp.ts` e identifique como o Vue controla a instalação única dos plugins.

---

# Desafio

Atualize sua **MiniVue** para suportar:

* `app.use()`;
* plugins baseados em função;
* plugins com `install()`;
* `provide()`;
* registro global de componentes;
* registro global de diretivas;
* configuração global (`app.config`).

---

# Projeto do capítulo

Ao final deste capítulo sua MiniVue deverá possuir um sistema completo de plugins, permitindo instalar bibliotecas, registrar componentes e diretivas globais, compartilhar dependências e reproduzir a arquitetura utilizada pelo Vue oficial.

---

# Checklist

* [ ] Sei explicar o propósito dos Plugins.
* [ ] Entendi como funciona `app.use()`.
* [ ] Sei a diferença entre `globalProperties` e `provide/inject`.
* [ ] Consigo criar plugins reutilizáveis.
* [ ] Minha MiniVue suporta um sistema básico de plugins.

---

# Próximo capítulo

## **Capítulo 38 — Vue Router por Dentro: Arquitetura, Matcher, Histórico e Navegação**

No próximo capítulo estudaremos a arquitetura interna do **Vue Router**. Você aprenderá como o sistema de rotas funciona internamente, como é feito o *route matching*, o gerenciamento do histórico (`History API`), navegação programática, *navigation guards*, rotas aninhadas, carregamento assíncrono e como implementar um roteador simplificado na sua MiniVue.
