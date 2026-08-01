# Capítulo 35 — Async Components: Lazy Loading, Code Splitting e Estratégias de Carregamento

> **Objetivo:** compreender profundamente como funcionam os **Componentes Assíncronos** no Vue 3. Ao final deste capítulo você entenderá como `defineAsyncComponent()` funciona internamente, como o Vite realiza *code splitting*, como tratar erros, timeouts, componentes de carregamento e como tudo isso se integra ao Runtime do Vue.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Entender o conceito de Componentes Assíncronos.
* Explicar o funcionamento do `defineAsyncComponent()`.
* Compreender Lazy Loading e Code Splitting.
* Configurar componentes de loading e erro.
* Entender timeouts e tentativas de recuperação.
* Implementar um componente assíncrono simplificado na MiniVue.

---

# Pré-requisitos

* Capítulos 01 ao 34.

---

# O problema

Imagine uma aplicação.

```text
ERP
```

Possui.

* Dashboard
* Financeiro
* Compras
* Estoque
* RH
* Fiscal
* Produção

---

Cada módulo.

Possui centenas.

De componentes.

---

Sem otimização.

Todo JavaScript.

É baixado.

Na primeira visita.

---

Mesmo que.

O usuário.

Nunca acesse.

O módulo.

Fiscal.

---

# Consequência

Imagine.

```text
Bundle

↓

8 MB
```

---

O navegador.

Precisa baixar.

Tudo.

---

Mesmo usando.

Apenas.

A tela.

De Login.

---

# A solução

Carregar.

Somente.

Quando necessário.

---

Isso é chamado.

```text
Lazy Loading
```

---

# Outro conceito

Separar.

O código.

Em vários arquivos.

---

Isso é chamado.

```text
Code Splitting
```

---

# Exemplo

Ao invés de.

```javascript
import Dashboard from "./Dashboard.vue"
```

Utilizamos.

```javascript
const Dashboard = ()=>

import("./Dashboard.vue")
```

Observe.

Não existe.

Import imediato.

---

Existe.

Uma função.

Que retorna.

Uma Promise.

---

# O Vue

Fornece.

Uma API.

Para isso.

```javascript
defineAsyncComponent()
```

---

# Primeiro exemplo

```javascript
import{

defineAsyncComponent

}from"vue"

const Dashboard=

defineAsyncComponent(

()=>import("./Dashboard.vue")

)
```

---

Agora.

O componente.

Só será.

Baixado.

Quando renderizado.

---

# Fluxo

```text
Render

↓

Promise

↓

Download

↓

Compilar

↓

Renderizar
```

---

# O que retorna?

`defineAsyncComponent()`.

Não retorna.

O componente.

---

Retorna.

Um Wrapper.

---

Visualmente.

```text
Componente

↓

Wrapper

↓

Loader

↓

Componente real
```

---

# Internamente

Simplificando.

Algo parecido.

Com.

```javascript
function defineAsyncComponent(loader){

return{

loader

}

}
```

---

Claro.

A implementação.

Real.

É muito.

Mais complexa.

---

# Integração

Quando.

O Renderer.

Encontra.

Esse Wrapper.

---

Ele executa.

```javascript
loader()
```

---

Resultado.

```text
Promise
```

---

Enquanto.

Ela.

Não resolve.

---

O componente.

Ainda.

Não existe.

---

# Loading

Podemos mostrar.

Outro componente.

Enquanto isso.

```javascript
defineAsyncComponent({

loader:

()=>import("./Tabela.vue"),

loadingComponent:

Loading

})
```

---

Fluxo.

```text
Loading

↓

Tabela
```

---

Muito elegante.

---

# Error Component

Também existe.

```javascript
defineAsyncComponent({

loader:

()=>import("./Tabela.vue"),

errorComponent:

Erro

})
```

---

Caso.

A Promise.

Falhe.

---

O Vue.

Renderiza.

Erro.

---

# Timeout

Imagine.

Que o servidor.

Não respondeu.

---

Podemos definir.

```javascript
timeout:5000
```

---

Depois de.

5 segundos.

---

O Vue.

Considera.

Que houve.

Falha.

---

# Delay

Existe.

Outro recurso.

Muito útil.

```javascript
delay:200
```

---

Significa.

Espere.

200ms.

Antes.

De mostrar.

O Loading.

---

Por quê?

---

Imagine.

Uma conexão.

Muito rápida.

---

O componente.

Carrega.

Em.

100ms.

---

Sem delay.

Teríamos.

```text
Loading

↓

Tela
```

Muito rápido.

Gerando.

Flickering.

---

Com delay.

Nunca veremos.

O Loading.

---

Experiência.

Muito melhor.

---

# Retry

Também podemos.

Tentar novamente.

---

```javascript
defineAsyncComponent({

loader,

onError(

error,

retry,

fail,

attempts

){

}
})
```

---

Exemplo.

```javascript
onError(

error,

retry,

fail,

attempts

){

if(

attempts<3

){

retry()

}else{

fail()

}

}
```

---

Muito útil.

Para.

Problemas.

Temporários.

De rede.

---

# Suspense

Lembra.

Do capítulo.

Anterior?

---

Ele trabalha.

Perfeitamente.

Com.

Async Components.

---

Fluxo.

```text
Async Component

↓

Suspense

↓

Fallback

↓

Promise

↓

Render
```

---

# Vite

Agora.

Vamos entender.

Quem realmente.

Divide.

O código.

---

Não é.

O Vue.

---

É.

O Bundler.

---

No nosso caso.

```text
Vite
```

---

O Vite.

Utiliza.

Rollup.

Na produção.

---

Quando encontra.

```javascript
import()
```

---

Ele cria.

Outro.

Arquivo.

JavaScript.

---

Visualmente.

Antes.

```text
app.js
```

---

Depois.

```text
app.js

dashboard.js

usuarios.js

financeiro.js

estoque.js
```

---

Cada módulo.

É carregado.

Separadamente.

---

# Benefício

Primeira carga.

Muito menor.

---

Usuário.

Baixa.

Somente.

O necessário.

---

# Router

O uso.

Mais comum.

É.

```javascript
{

path:"/usuarios",

component:

()=>import(

"./Usuarios.vue"

)

}
```

---

Cada rota.

Vira.

Um chunk.

Independente.

---

# Chunk

Um chunk.

Nada mais é.

Do que.

Um arquivo.

JavaScript.

Gerado.

Pelo bundler.

---

# Fluxo completo

```text
import()

↓

Rollup

↓

Chunk

↓

HTTP

↓

Browser

↓

Execute

↓

Render
```

---

# Prefetch

Em algumas situações.

Queremos.

Baixar.

Antes.

Do usuário.

Precisar.

---

Isso é chamado.

```text
Prefetch
```

---

O navegador.

Baixa.

Em segundo plano.

---

# Preload

Outro conceito.

É.

```text
Preload
```

---

Aqui.

O download.

É prioritário.

---

Muito usado.

Para recursos.

Necessários.

Logo após.

A navegação.

---

# Como o Vue sabe?

Na verdade.

Não sabe.

---

Quem controla.

Chunks.

Prefetch.

Preload.

É o Bundler.

---

O Vue.

Apenas.

Solicita.

O módulo.

---

# MiniVue

Podemos criar.

Algo simples.

Assim.

```javascript
function defineAsyncComponent(

loader

){

return{

async:true,

loader

}

}
```

---

Depois.

O Renderer.

Executa.

```javascript
const comp=

await loader()
```

---

Em seguida.

Renderiza.

O componente.

---

# Evoluindo

Depois.

Podemos adicionar.

```javascript
loadingComponent
```

---

```javascript
errorComponent
```

---

```javascript
timeout
```

---

```javascript
retry()
```

---

Exatamente.

Como o Vue.

---

# Fluxo interno

```text
Render

↓

Async Wrapper

↓

Loader()

↓

Promise

↓

Resolveu?

↓

Sim

↓

Render

↓

Fim
```

---

Caso contrário.

```text
Erro

↓

Error Component
```

---

# Performance

Aplicações grandes.

Podem reduzir.

Significativamente.

O tempo.

De carregamento.

---

Imagine.

```text
Bundle

↓

12 MB
```

---

Após.

Code Splitting.

```text
Inicial

↓

600 KB
```

---

O restante.

Será baixado.

Sob demanda.

---

Essa estratégia.

É um dos motivos.

Pelos quais.

Grandes aplicações Vue.

Conseguem manter.

Boa performance.

Mesmo possuindo.

Centenas de componentes.

---

# Código-fonte

Grande parte da implementação está em:

```text
packages/runtime-core/src/apiAsyncComponent.ts
```

Vale estudar especialmente:

* `defineAsyncComponent()`;
* gerenciamento de estado interno;
* tratamento de timeout;
* controle de erros;
* integração com `Suspense`.

Também observe como o Renderer trata componentes assíncronos durante o processo de montagem.

---

# Curiosidade

O `defineAsyncComponent()` **não é apenas um `await import()`**. Internamente ele cria um componente intermediário responsável por controlar carregamento, erro, timeout, tentativas de recuperação e integração com o `Suspense`, funcionando como uma camada inteligente entre o Renderer e o componente real.

---

# Resumo

Neste capítulo aprendemos que:

* Componentes assíncronos utilizam `defineAsyncComponent()`.
* O carregamento acontece apenas quando necessário.
* O bundler é responsável pelo Code Splitting.
* O Vue fornece suporte para loading, erro, timeout e retry.
* `Suspense` integra-se naturalmente aos componentes assíncronos.
* O Wrapper criado pelo Vue controla todo o ciclo de carregamento.

---

# Exercícios

## Exercício 1

Transforme três componentes do seu projeto em componentes assíncronos utilizando `defineAsyncComponent()`.

---

## Exercício 2

Configure `loadingComponent` e `errorComponent` para um componente carregado sob demanda.

---

## Exercício 3

Implemente um timeout de 5 segundos e teste o comportamento.

---

## Exercício 4

Implemente uma lógica de retry com até três tentativas.

---

## Exercício 5

Leia `packages/runtime-core/src/apiAsyncComponent.ts` e identifique como o Vue controla os estados de carregamento, sucesso e erro.

---

# Desafio

Atualize sua **MiniVue** para suportar:

* `defineAsyncComponent()`;
* carregamento sob demanda;
* componente de loading;
* componente de erro;
* timeout;
* mecanismo de retry.

---

# Projeto do capítulo

Ao final deste capítulo sua MiniVue deverá ser capaz de carregar componentes sob demanda, exibir estados de carregamento e erro e integrar esse fluxo ao Renderer, aproximando-se ainda mais da implementação oficial do Vue.

---

# Checklist

* [ ] Sei explicar o que é um Async Component.
* [ ] Entendi como funciona `defineAsyncComponent()`.
* [ ] Sei a diferença entre Lazy Loading e Code Splitting.
* [ ] Entendi o papel do bundler na geração de chunks.
* [ ] Minha MiniVue suporta componentes assíncronos básicos.

---

# Próximo capítulo

## **Capítulo 36 — Diretivas Customizadas: Como Funcionam Internamente e Como Criar as Suas**

No próximo capítulo estudaremos as **Diretivas Customizadas** do Vue 3. Você aprenderá como `v-focus`, `v-click-outside`, `v-intersect` e outras diretivas são implementadas, como o Compiler as transforma, como o Renderer executa seus hooks (`created`, `beforeMount`, `mounted`, `beforeUpdate`, `updated`, `beforeUnmount` e `unmounted`) e como adicionar suporte a diretivas personalizadas na sua MiniVue.
