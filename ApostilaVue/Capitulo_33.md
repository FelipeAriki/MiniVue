# Capítulo 33 — KeepAlive: Cache Inteligente de Componentes e Gerenciamento de Estado

> **Objetivo:** compreender profundamente como funciona o componente **KeepAlive** no Vue 3. Ao final deste capítulo você entenderá como o Vue mantém componentes em memória sem desmontá-los, como funciona o cache interno, o algoritmo **LRU (Least Recently Used)**, os ciclos de vida `activated` e `deactivated` e implementará uma versão simplificada na MiniVue.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Explicar o propósito do KeepAlive.
* Entender quando utilizá-lo.
* Compreender seu funcionamento interno.
* Explicar o algoritmo LRU utilizado pelo Vue.
* Entender os ciclos `activated` e `deactivated`.
* Implementar um sistema de cache de componentes.

---

# Pré-requisitos

* Capítulos 01 ao 32.

---

# O problema

Imagine uma aplicação.

```text
App

↓

Dashboard

↓

Usuários

↓

Detalhes
```

O usuário navega.

```text
Usuários

↓

Detalhes

↓

Usuários
```

---

Sem KeepAlive.

O componente.

É destruído.

Toda vez.

---

Fluxo.

```text
Mount

↓

Unmount

↓

Mount

↓

Unmount
```

---

# Consequências

Todo estado.

É perdido.

---

Exemplo.

```javascript
const filtro = ref("")
```

O usuário digitou.

```text
Felipe
```

---

Saiu da tela.

---

Voltou.

---

Resultado.

```text
""
```

Tudo foi perdido.

---

Outro exemplo.

Imagine.

Uma tabela.

Com:

* filtros;
* paginação;
* ordenação;
* scroll.

---

Sem KeepAlive.

Tudo reinicia.

---

# A solução

Existe.

Um componente.

Especial.

```vue
<KeepAlive>

<RouterView/>

</KeepAlive>
```

---

Agora.

Ao trocar.

De página.

O componente.

Não é destruído.

---

Ele apenas.

É desativado.

---

# Fluxo

Sem KeepAlive.

```text
Mount

↓

Unmount
```

---

Com KeepAlive.

```text
Mount

↓

Deactivate

↓

Activate

↓

Deactivate
```

Observe.

Nunca ocorre.

Unmount.

---

# Resultado

Todo estado.

Permanece.

---

Incluindo.

```text
Refs

Reactive

Computed

Watchers

DOM

Scroll

Inputs
```

---

# Exemplo

```vue
<KeepAlive>

<Componente/>

</KeepAlive>
```

---

Primeira renderização.

```text
Cache

↓

Vazio
```

---

O Vue.

Monta.

O componente.

---

Depois.

Armazena.

No cache.

---

# Segunda renderização

O componente.

Já existe.

No cache.

---

Então.

O Vue.

Não cria.

Outro.

---

Ele reutiliza.

A instância.

---

# Visualmente

```text
Primeira vez

↓

Criar instância

↓

Guardar no cache

↓

Renderizar
```

---

Depois.

```text
Cache

↓

Encontrou

↓

Reutilizar

↓

Renderizar
```

---

# O cache

Internamente.

O Vue utiliza.

Estruturas.

Muito eficientes.

---

Principalmente.

```javascript
Map()
```

---

E.

```javascript
Set()
```

---

Simplificando.

```javascript
cache = new Map()
```

---

Cada componente.

Recebe.

Uma chave.

---

Exemplo.

```text
Home

↓

Instância
```

---

```text
Dashboard

↓

Instância
```

---

```text
Usuários

↓

Instância
```

---

# Quando navegar

O Vue pergunta.

```javascript
cache.has(key)
```

---

Se.

```text
Sim
```

↓

Reutiliza.

---

Se.

```text
Não
```

↓

Cria.

Nova instância.

---

# include

Podemos limitar.

Quais componentes.

Serão mantidos.

---

```vue
<KeepAlive

include="Home"

>

<RouterView/>

</KeepAlive>
```

---

Também.

```vue
include="Home,Dashboard"
```

---

Ou.

```vue
:include="['Home','Dashboard']"
```

---

# exclude

Também existe.

```vue
<KeepAlive

exclude="Login"

/>
```

---

Todos.

Serão armazenados.

Exceto.

Login.

---

# max

Agora.

Um recurso.

Muito importante.

---

Imagine.

Uma aplicação.

Com.

100 páginas.

---

Guardar.

Todas.

Em memória.

Pode consumir.

Muitos recursos.

---

Existe.

A propriedade.

```vue
<KeepAlive

:max="10"

/>
```

---

Agora.

Somente.

10 componentes.

Permanecem.

No cache.

---

# O problema

O que acontece.

Quando chega.

O décimo primeiro?

---

Qual componente.

Será removido?

---

# LRU Cache

O Vue utiliza.

Um algoritmo clássico.

Chamado.

```text
Least Recently Used
```

---

Ou.

```text
LRU
```

---

# Como funciona?

Sempre que.

Um componente.

É utilizado.

---

Ele vai.

Para o final.

Da fila.

---

Visualmente.

```text
A

B

C
```

---

Usuário abriu.

```text
A
```

Agora.

```text
B

C

A
```

---

Depois.

Abriu.

```text
C
```

Resultado.

```text
B

A

C
```

---

Observe.

Quem está.

Há mais tempo.

Sem uso.

É.

```text
B
```

---

Quando.

O cache.

Encher.

---

B.

Será removido.

---

# Fluxo

```text
Cache cheio

↓

Adicionar novo

↓

Remover

↓

Menos utilizado
```

---

# Como implementar?

Simplificando.

```javascript
cache = new Map()

keys = new Set()
```

---

Sempre que.

Um componente.

É utilizado.

---

```javascript
keys.delete(key)

keys.add(key)
```

---

Assim.

Ele passa.

Para o final.

---

Quando.

O limite.

É atingido.

---

```javascript
const oldest =

keys.values()

.next().value
```

---

Removemos.

```javascript
cache.delete(oldest)

keys.delete(oldest)
```

---

Muito elegante.

---

# Lifecycle

Agora.

Outro detalhe.

Muito importante.

---

KeepAlive.

Não dispara.

```javascript
onUnmounted()
```

---

Porque.

O componente.

Não foi.

Destruído.

---

Em vez disso.

Temos.

```javascript
onActivated()
```

---

E.

```javascript
onDeactivated()
```

---

# Exemplo

```javascript
onActivated(()=>{

console.log("Voltou")

})
```

---

```javascript
onDeactivated(()=>{

console.log("Saiu")

})
```

---

Esses Hooks.

Existem apenas.

Para componentes.

Mantidos.

Pelo KeepAlive.

---

# Fluxo

Primeira vez.

```text
BeforeMount

↓

Mounted
```

---

Depois.

```text
Deactivated

↓

Activated

↓

Deactivated

↓

Activated
```

---

Nunca ocorre.

```text
Mounted

novamente
```

---

# Router

O uso mais comum.

É.

```vue
<KeepAlive>

<RouterView/>

</KeepAlive>
```

---

Assim.

Cada página.

Permanece.

Em memória.

---

Muito utilizado.

Em:

* ERPs;
* CRMs;
* Dashboards;
* Sistemas administrativos.

---

# Quando NÃO utilizar

Não utilize.

Para.

Componentes.

Muito pesados.

Que dificilmente.

Serão reutilizados.

---

Nem.

Quando.

O estado.

Pode ser.

Facilmente.

Recarregado.

---

# Casos reais

Excelente para:

* formulários longos;
* assistentes (wizard);
* abas;
* dashboards;
* páginas de consulta;
* tabelas grandes.

---

Ruim para:

* componentes temporários;
* telas de autenticação;
* páginas muito simples;
* componentes com alto consumo de memória.

---

# Funcionamento interno

O Compiler.

Produz.

Um VNode.

Especial.

---

O Renderer.

Reconhece.

```text
KEEP_ALIVE
```

---

Depois.

Encaminha.

Para.

Uma implementação.

Especial.

---

Fluxo.

```text
VNode

↓

KeepAlive

↓

Cache

↓

Renderer
```

---

# MiniVue

Uma implementação.

Muito simples.

Pode começar.

Assim.

```javascript
const cache = new Map()

function renderComponent(key){

    if(cache.has(key)){

        return cache.get(key)

    }

    const instance = createComponent()

    cache.set(key, instance)

    return instance

}
```

---

Depois.

Adicione.

Suporte.

Ao LRU.

---

# Arquivos reais

Grande parte da implementação está em.

```text
packages/runtime-core/src/components/KeepAlive.ts
```

Vale estudar.

As funções.

```text
setup()

activate()

deactivate()

pruneCache()

pruneCacheEntry()
```

Também observe.

Como o Renderer.

Interage.

Com.

```text
renderer.ts
```

Durante a ativação.

E desativação.

---

# Curiosidade

O KeepAlive **não impede a reatividade**. Enquanto um componente está desativado, seus estados reativos continuam existindo na memória. Isso significa que, se um estado compartilhado (como um store do Pinia) mudar enquanto o componente estiver desativado, ao ser reativado ele já refletirá os novos dados sem precisar ser recriado.

---

# Resumo

Neste capítulo aprendemos que:

* KeepAlive mantém componentes montados em memória.
* O componente não é destruído, apenas desativado.
* O Vue utiliza um cache baseado em `Map`.
* O limite de cache é controlado pela propriedade `max`.
* O algoritmo LRU remove o componente menos recentemente utilizado.
* `onActivated()` e `onDeactivated()` substituem parte do ciclo de montagem e desmontagem.

---

# Exercícios

## Exercício 1

Utilize `KeepAlive` com `RouterView` e observe a preservação do estado.

---

## Exercício 2

Implemente um cache simples utilizando `Map`.

---

## Exercício 3

Adicione suporte às propriedades `include` e `exclude`.

---

## Exercício 4

Implemente uma política LRU utilizando `Map` e `Set`.

---

## Exercício 5

Leia `packages/runtime-core/src/components/KeepAlive.ts` e identifique como o Vue move componentes entre os estados de ativado e desativado.

---

# Desafio

Atualize sua **MiniVue** para suportar:

* cache de componentes;
* ativação e desativação;
* `include`;
* `exclude`;
* `max`;
* remoção utilizando LRU.

---

# Projeto do capítulo

Ao final deste capítulo sua MiniVue deverá ser capaz de manter componentes em memória, reutilizar instâncias existentes e gerenciar um cache inteligente semelhante ao utilizado pelo Vue oficial.

---

# Checklist

* [ ] Sei explicar o propósito do KeepAlive.
* [ ] Entendi como funciona o cache interno.
* [ ] Sei explicar o algoritmo LRU.
* [ ] Entendi `onActivated()` e `onDeactivated()`.
* [ ] Minha MiniVue possui um sistema básico de cache de componentes.

---

# Próximo capítulo

## **Capítulo 34 — Suspense: Renderização Assíncrona e Coordenação de Componentes**

No próximo capítulo estudaremos o **Suspense**, um dos recursos mais sofisticados do Vue 3. Você aprenderá como o framework coordena componentes assíncronos, controla estados de carregamento, gerencia múltiplas dependências assíncronas e integra esse comportamento ao Renderer e ao Scheduler, além de implementar uma versão simplificada desse mecanismo na MiniVue.
