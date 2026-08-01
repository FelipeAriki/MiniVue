# Capítulo 38 — Vue Router por Dentro: Arquitetura, Matcher, Histórico e Navegação

> **Objetivo:** compreender profundamente como funciona o **Vue Router** internamente. Ao final deste capítulo você entenderá como o sistema de rotas faz o *matching* de URLs, utiliza a History API do navegador, executa *Navigation Guards*, gerencia componentes aninhados e como implementar um roteador simplificado na sua MiniVue.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Explicar como o Vue Router funciona internamente.
* Entender o Route Matcher.
* Compreender a History API.
* Dominar Navigation Guards.
* Entender rotas aninhadas.
* Implementar um Router simplificado na MiniVue.

---

# Pré-requisitos

* Capítulos 01 ao 37.

---

# O problema

Imagine uma aplicação.

```text
ERP
```

Possui.

* Login
* Dashboard
* Produtos
* Compras
* Financeiro
* Estoque

---

Sem um Router.

Precisaríamos.

Controlar.

Qual componente.

Exibir.

Manual.

```javascript
const page = ref("dashboard")
```

---

Depois.

```vue
<Dashboard

v-if="page==='dashboard'"

/>

<Produtos

v-if="page==='produtos'"

/>
```

---

À medida.

Que a aplicação.

Cresce.

---

Isso se torna.

Impraticável.

---

# A solução

O Vue.

Possui.

O Vue Router.

---

Ele faz.

O mapeamento.

Entre.

URLs.

E componentes.

---

# Primeiro exemplo

```javascript
const routes=[

{

path:"/",

component:Home

},

{

path:"/usuarios",

component:Usuarios

}

]
```

---

Agora.

Quando.

A URL.

É.

```text
/usuarios
```

---

O Router.

Renderiza.

```text
Usuarios.vue
```

---

# Fluxo

```text
URL

↓

Router

↓

Matcher

↓

Componente

↓

Renderer

↓

DOM
```

---

# O Matcher

O coração.

Do Router.

É chamado.

Matcher.

---

Sua função.

É responder.

Uma pergunta.

---

```text
Qual rota corresponde
à URL atual?
```

---

Exemplo.

Temos.

```text
/
```

---

```text
/produtos
```

---

```text
/produtos/:id
```

---

Quando.

A URL.

É.

```text
/produtos/25
```

---

O Matcher.

Percorre.

As rotas.

Até encontrar.

A correspondente.

---

Resultado.

```javascript
{

path:"/produtos/:id",

params:{

id:25

}

}
```

---

# Rotas Dinâmicas

Exemplo.

```javascript
{

path:"/usuarios/:id"

}
```

---

URL.

```text
/usuarios/15
```

---

Resultado.

```javascript
route.params.id
```

↓

```text
15
```

---

# Como funciona?

Simplificando.

O Matcher.

Transforma.

A rota.

Em.

Uma expressão.

Comparável.

---

```text
/usuarios/:id
```

↓

```text
/usuarios/*
```

↓

Comparação.

---

Na implementação.

Real.

É utilizada.

Uma estratégia.

Muito mais sofisticada.

Baseada.

Em parsing.

Da rota.

---

# Query String

Também existe.

```text
/produtos?page=2
```

---

Resultado.

```javascript
route.query.page
```

↓

```text
2
```

---

Observe.

O Matcher.

Também separa.

Os parâmetros.

Da Query.

---

# Hash

Outro exemplo.

```text
/docs#instalacao
```

---

Resultado.

```javascript
route.hash
```

↓

```text
#instalacao
```

---

# History API

Agora.

Uma pergunta.

---

Como.

A URL.

Muda.

Sem.

Recarregar.

A página?

---

A resposta.

É.

```text
History API
```

---

Ela possui.

Métodos.

Como.

```javascript
history.pushState()
```

---

```javascript
history.replaceState()
```

---

```javascript
history.back()
```

---

```javascript
history.forward()
```

---

# pushState()

Imagine.

```javascript
router.push(

"/usuarios"

)
```

---

Internamente.

O Router.

Executa.

```javascript
history.pushState(

state,

"",

"/usuarios"

)
```

---

Resultado.

A URL muda.

---

Mas.

A página.

Não recarrega.

---

# popstate

Quando.

O usuário.

Clica.

No botão.

Voltar.

---

O navegador.

Dispara.

```javascript
popstate
```

---

O Router.

Escuta.

Esse evento.

---

```javascript
window.addEventListener(

"popstate",

...

)
```

---

Quando.

O evento.

Acontece.

---

O Router.

Renderiza.

A nova rota.

---

# RouterView

Lembra.

Do componente.

```vue
<RouterView/>
```

---

Ele é.

Muito importante.

---

Sua função.

É renderizar.

O componente.

Correspondente.

À rota atual.

---

Fluxo.

```text
URL

↓

Matcher

↓

RouterView

↓

Componente
```

---

# RouterLink

Outro componente.

Muito utilizado.

```vue
<RouterLink

to="/usuarios"

>

Usuários

</RouterLink>
```

---

Internamente.

Ele gera.

Um.

```html
<a href="/usuarios">

Usuários

</a>
```

---

Mas.

Intercepta.

O clique.

---

Ao invés.

De navegar.

Normalmente.

---

Executa.

```javascript
router.push(...)
```

---

Assim.

Não ocorre.

Reload.

Da página.

---

# Navigation Guards

Agora.

Uma das partes.

Mais importantes.

---

Imagine.

Uma rota.

Protegida.

```text
/admin
```

---

O usuário.

Não está.

Logado.

---

Como impedir.

A navegação?

---

Utilizando.

Navigation Guards.

---

Exemplo.

```javascript
router.beforeEach(

(to,from)=>{

}
)
```

---

Fluxo.

```text
Clique

↓

beforeEach

↓

Autorizado?

↓

Sim

↓

Navegação

↓

Não

↓

Cancelar
```

---

# Outro exemplo

```javascript
router.beforeEach(

(to)=>{

if(

!usuarioLogado

){

return"/login"

}

}
)
```

---

Observe.

Nem sempre.

Cancelamos.

---

Também.

Podemos.

Redirecionar.

---

# Outros Guards

Existe.

```javascript
beforeResolve()
```

---

Executa.

Antes.

Da navegação.

Ser concluída.

---

Também.

```javascript
afterEach()
```

---

Executa.

Após.

A navegação.

---

Ideal.

Para.

Analytics.

Logs.

Título.

Da página.

---

# Guards por rota

Também.

Podemos.

Definir.

```javascript
{

path:"/admin",

beforeEnter(){

}

}
```

---

Somente.

Essa rota.

Será.

Protegida.

---

# Guards por componente

Também existem.

```javascript
beforeRouteEnter()
```

---

```javascript
beforeRouteLeave()
```

---

```javascript
beforeRouteUpdate()
```

---

Muito úteis.

Para.

Salvar.

Formulários.

Antes.

De sair.

Da página.

---

# Rotas Aninhadas

Imagine.

```text
/dashboard
```

---

Possui.

Menu.

Lateral.

---

Conteúdo.

Central.

---

Criamos.

```javascript
{

path:"/dashboard",

children:[...]

}
```

---

Visualmente.

```text
Dashboard

↓

RouterView

↓

Filho
```

---

Assim.

Cada rota.

Filha.

Renderiza.

Dentro.

Do pai.

---

# Lazy Loading

Lembra.

Do capítulo.

Anterior?

---

Também.

É utilizado.

No Router.

---

```javascript
component:

()=>import(

"./Usuarios.vue"

)
```

---

Cada página.

Será carregada.

Quando.

Necessário.

---

# Fluxo completo

```text
Clique

↓

RouterLink

↓

pushState()

↓

Matcher

↓

Navigation Guards

↓

Route

↓

RouterView

↓

Renderer

↓

DOM
```

---

# Como implementar?

Na MiniVue.

Começamos.

Assim.

```javascript
const routes=[]

function push(path){

currentRoute.value=

path

}
```

---

Depois.

Criamos.

Um Matcher.

---

```javascript
function match(path){

return routes.find(

r=>r.path===path

)

}
```

---

Depois.

Criamos.

Um componente.

Semelhante.

Ao.

```vue
<RouterView/>
```

---

```javascript
const route=

match(

currentRoute.value

)

return h(

route.component

)
```

---

Já temos.

Um Router.

Muito simples.

---

# Evoluindo

Depois.

Adicionar.

* params;
* query;
* hash;
* nested routes;
* guards;
* lazy loading.

---

Sua MiniVue.

Terá.

Um Router.

Muito parecido.

Com.

O oficial.

---

# Performance

O Matcher.

Não percorre.

Todas.

As rotas.

De forma ingênua.

---

Internamente.

O Vue Router.

Constrói.

Estruturas.

Otimizadas.

Para.

Encontrar.

Rotas.

Rapidamente.

---

Em aplicações.

Com centenas.

De rotas.

Isso faz.

Grande diferença.

---

# Código-fonte

Grande parte da arquitetura do Vue Router está distribuída entre:

```text
packages/router/src/matcher
```

---

A integração com o histórico está em módulos semelhantes a:

```text
history/html5.ts
```

---

Também vale estudar:

```text
RouterMatcher
```

---

```text
createRouter()
```

---

```text
createWebHistory()
```

---

Esses módulos mostram como o Router constrói o sistema de navegação e integra o History API ao Runtime.

---

# Curiosidade

Embora pareça apenas uma lista de rotas, o Vue Router cria internamente uma estrutura altamente otimizada para realizar o *matching* de URLs. Rotas dinâmicas, parâmetros opcionais, aliases e rotas aninhadas recebem diferentes níveis de prioridade para que a rota mais específica seja encontrada antes das mais genéricas.

---

# Resumo

Neste capítulo aprendemos que:

* O Vue Router associa URLs a componentes.
* O Matcher identifica qual rota corresponde à URL atual.
* A History API permite alterar a URL sem recarregar a página.
* `RouterLink` intercepta cliques e utiliza `pushState()`.
* `RouterView` renderiza o componente correspondente.
* Navigation Guards controlam o fluxo de navegação.
* Rotas aninhadas permitem layouts complexos.

---

# Exercícios

## Exercício 1

Implemente um Router simples utilizando um `ref` para controlar a rota atual.

---

## Exercício 2

Adicione suporte a rotas dinâmicas (`:id`).

---

## Exercício 3

Implemente `router.push()` utilizando `history.pushState()`.

---

## Exercício 4

Crie um `beforeEach()` que bloqueie acesso a uma rota protegida.

---

## Exercício 5

Leia o código do Vue Router e identifique como o `RouterMatcher` organiza as rotas e resolve conflitos entre rotas estáticas e dinâmicas.

---

# Desafio

Atualize sua **MiniVue** para suportar:

* `RouterView`;
* `RouterLink`;
* `push()`;
* `replace()`;
* params;
* query;
* hash;
* Navigation Guards;
* rotas aninhadas.

---

# Projeto do capítulo

Ao final deste capítulo sua MiniVue deverá possuir um roteador funcional capaz de controlar a URL da aplicação, renderizar componentes dinamicamente e executar um fluxo de navegação semelhante ao do Vue Router oficial.

---

# Checklist

* [ ] Sei explicar como funciona o Vue Router internamente.
* [ ] Entendi o funcionamento do Route Matcher.
* [ ] Sei como a History API evita recarregamentos da página.
* [ ] Entendi Navigation Guards e suas aplicações.
* [ ] Minha MiniVue possui um Router funcional.

---

# Próximo capítulo

## **Capítulo 39 — Pinia por Dentro: Arquitetura, Stores, Reatividade e Gerenciamento Global de Estado**

No próximo capítulo estudaremos a arquitetura interna do **Pinia**. Você aprenderá como funcionam as Stores, por que o Pinia substituiu o Vuex, como utiliza a reatividade do Vue, como registra Stores dinamicamente, como funciona o sistema de plugins do Pinia e como implementar um gerenciador de estado global inspirado nele na sua MiniVue.
