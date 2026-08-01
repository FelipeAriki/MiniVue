# Capítulo 46 — Custom Renderer: Criando um Renderer para Canvas, Terminal e Outras Plataformas

> **Objetivo:** compreender profundamente a arquitetura de **Custom Renderers** do Vue 3. Ao final deste capítulo você entenderá como o Runtime Core é completamente independente da plataforma, aprenderá como funciona o `createRenderer()`, verá como o Vue pode renderizar aplicações para Canvas, Terminal, aplicações nativas e ambientes customizados e implementará seu próprio Renderer na MiniVue.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Entender a arquitetura multiplataforma do Vue.
* Explicar o funcionamento do `createRenderer()`.
* Criar um Renderer personalizado.
* Compreender como surgem projetos como NativeScript Vue e renderizadores para Canvas.
* Implementar um Custom Renderer completo na MiniVue.

---

# Pré-requisitos

* Capítulos 01 ao 45.

---

# Introdução

Quando pensamos.

Em Vue.

Normalmente.

Pensamos.

No navegador.

---

HTML.

CSS.

JavaScript.

---

Mas.

Na realidade.

O Vue.

Nunca.

Foi criado.

Apenas.

Para o DOM.

---

Sua arquitetura.

Foi desenhada.

Para.

Renderizar.

Em qualquer.

Plataforma.

---

Inclusive.

Onde.

Nem existe.

HTML.

---

# A ideia principal

Imagine.

Que o Runtime.

Nunca faça.

Isso.

```javascript
document.createElement()
```

---

Nem.

```javascript
element.appendChild()
```

---

Nem.

```javascript
addEventListener()
```

---

Então.

Como.

Ele cria.

A interface?

---

Resposta.

Ele.

Não cria.

---

Quem cria.

É.

O Renderer.

---

# Separação

Visualmente.

```text
Vue Runtime Core

↓

Host Operations

↓

Renderer

↓

Plataforma
```

---

A plataforma.

Pode ser.

---

DOM.

---

Canvas.

---

Terminal.

---

Aplicação nativa.

---

Game Engine.

---

PDF.

---

SVG.

---

Qualquer.

Sistema.

Que consiga.

Receber.

Comandos.

---

# O createRenderer()

Toda.

Essa mágica.

Começa.

Aqui.

```javascript
createRenderer(options)
```

---

O parâmetro.

`options`.

Contém.

As operações.

Da plataforma.

---

Exemplo.

```javascript
createRenderer({

createElement,

insert,

remove,

patchProp,

createText,

setText,

parentNode,

nextSibling

})
```

---

Observe.

O Runtime.

Não conhece.

Nada.

Sobre.

DOM.

---

Ele apenas.

Chama.

Essas funções.

---

# Exemplo

Imagine.

Um Renderer.

Para.

Console.

---

Ao invés.

De criar.

Elementos.

---

Criamos.

Objetos.

---

```javascript
createElement(tag){

return {

tag,

children:[]

}

}
```

---

Depois.

Ao inserir.

---

```javascript
insert(

child,

parent

)
```

---

Apenas.

Adicionamos.

No array.

---

No final.

Podemos.

Imprimir.

A árvore.

---

Resultado.

```text
App

└── Div

    ├── H1

    └── Button
```

---

Nenhum.

HTML.

Foi criado.

---

Mesmo assim.

O Vue.

Funcionou.

---

# Outro exemplo

Canvas.

---

Em vez.

De.

```javascript
createElement("rect")
```

---

Criamos.

```javascript
new Rectangle()
```

---

Depois.

No insert.

---

Desenhamos.

Na tela.

---

```text
Renderer

↓

Canvas API

↓

Pixels
```

---

O Runtime.

Continua.

Exatamente.

O mesmo.

---

# Aplicações nativas

Imagine.

Um componente.

```vue
<Button>

Salvar

</Button>
```

---

No navegador.

Ele vira.

```html
<button>
```

---

Em um Renderer.

Nativo.

Pode virar.

```text
UIButton
```

---

Ou.

```text
Android Button
```

---

Tudo.

Sem alterar.

O Runtime.

---

# Fluxo

```text
Template

↓

Compiler

↓

Render Function

↓

Runtime Core

↓

Custom Renderer

↓

Plataforma
```

---

Apenas.

A última.

Camada.

Muda.

---

Todo.

O restante.

É reutilizado.

---

# Criando um Renderer

Primeiro.

Criamos.

As operações.

---

```javascript
const host={

createElement,

insert,

remove,

patchProp

}
```

---

Depois.

Chamamos.

```javascript
const renderer=

createRenderer(host)
```

---

Agora.

O Runtime.

Passa.

A utilizar.

Nosso.

Host.

---

# Criando elementos

Exemplo.

```javascript
createElement(tag){

return{

type:tag,

children:[]

}

}
```

---

Observe.

Nenhum.

DOM.

---

Apenas.

Objetos.

---

# Inserindo

```javascript
insert(

child,

parent

){

parent.children.push(child)

}
```

---

Agora.

Já temos.

Uma árvore.

---

# Texto

```javascript
createText(text){

return{

type:"text",

text

}

}
```

---

Depois.

```javascript
setText(

node,

value

){

node.text=value

}
```

---

Tudo.

Continua.

Independente.

Do navegador.

---

# Eventos

Imagine.

```vue
<button

@click="salvar"

>
```

---

Nosso Renderer.

Pode armazenar.

Assim.

```javascript
node.events.click=

handler
```

---

Depois.

Outra.

Parte.

Da plataforma.

Executa.

O handler.

---

Não existe.

Obrigação.

De utilizar.

DOM Events.

---

# Patch

Quando.

Um VNode.

Muda.

---

O Runtime.

Executa.

```javascript
patch(

old,

new

)
```

---

Internamente.

Ele chama.

```javascript
patchProp()
```

---

Nosso.

Renderer.

Decide.

Como.

Atualizar.

A plataforma.

---

# Exemplo prático

Imagine.

Uma Engine.

De jogos.

---

Template.

```vue
<Sprite

:x="100"

:y="200"

>
```

---

Renderer.

↓

```javascript
sprite.x=100

sprite.y=200
```

---

Nenhum.

HTML.

Foi criado.

---

Mesmo assim.

Toda.

A reatividade.

Continua.

Funcionando.

---

# Reutilização

Observe.

Tudo.

Que continua.

Igual.

```text
Compiler

✔

Reactivity

✔

Scheduler

✔

Watcher

✔

Computed

✔

Diff Algorithm

✔

Patch Flags

✔
```

---

Apenas.

O Renderer.

É diferente.

---

Essa.

É uma.

Das maiores.

Qualidades.

Da arquitetura.

Do Vue.

---

# Como implementar?

Na MiniVue.

Podemos.

Criar.

Uma plataforma.

Fictícia.

---

```javascript
class Node{

constructor(type){

this.type=type

this.children=[]

}

}
```

---

Depois.

Implementar.

```javascript
createElement(tag){

return new Node(tag)

}
```

---

E.

```javascript
insert(

child,

parent

){

parent.children.push(child)

}
```

---

Quando.

Chamarmos.

O Renderer.

---

Teremos.

Uma árvore.

Completa.

---

Sem utilizar.

O navegador.

---

# Outra ideia

Criar.

Um Renderer.

Para.

Terminal.

---

Cada.

Elemento.

É convertido.

Em texto.

---

```text
+----------------------+

| Login                |

|                      |

| [ Entrar ]           |

+----------------------+
```

---

Toda.

A interface.

É produzida.

Pelo Runtime.

---

# Arquitetura

Visualmente.

```text
Vue Runtime

↓

Renderer

↓

Host Operations

↓

Platform API

↓

Resultado
```

---

A Platform API.

Pode ser.

Qualquer.

Sistema.

---

# Performance

Como.

O Runtime.

É reutilizado.

---

As otimizações.

Do Vue.

Também.

Funcionam.

Nos.

Custom Renderers.

---

Incluindo.

* Scheduler;
* Patch Flags;
* Block Tree;
* Static Hoisting;
* Diff Algorithm.

---

Não é.

Necessário.

Reimplementá-las.

---

# Código-fonte

A implementação principal pode ser estudada em:

```text
packages/runtime-core/src/renderer.ts
```

---

Também vale analisar:

```text
packages/runtime-core/src/apiCreateApp.ts
```

---

```text
packages/runtime-core/src/component.ts
```

---

```text
packages/runtime-core/src/vnode.ts
```

---

O Runtime DOM apenas fornece as Host Operations para o Renderer genérico definido nesses módulos.

---

# Casos reais

Diversos projetos utilizam essa arquitetura.

Entre eles.

* Vue Runtime DOM (navegador).
* Renderizadores experimentais para Canvas.
* Integrações com aplicações nativas.
* Ambientes embarcados e interfaces customizadas.

O ponto central é que **o Runtime Core permanece praticamente o mesmo**, enquanto apenas a camada de integração com a plataforma muda.

---

# Curiosidade

O Renderer do Vue segue um padrão conhecido como **Host Renderer**. Em vez de conhecer detalhes da plataforma, ele depende de um conjunto de operações fornecidas externamente. Essa abordagem reduz o acoplamento e torna o Runtime altamente reutilizável, um dos principais diferenciais arquiteturais do Vue 3.

---

# Resumo

Neste capítulo aprendemos que:

* O Runtime Core é totalmente independente da plataforma.
* `createRenderer()` recebe as operações específicas do ambiente.
* Apenas o Renderer muda entre plataformas.
* Toda a reatividade e o algoritmo de diff são reutilizados.
* É possível criar Renderers para DOM, Canvas, terminal e outros ambientes.

---

# Exercícios

## Exercício 1

Implemente um Renderer que crie uma árvore de objetos em memória.

---

## Exercício 2

Implemente `createElement()`, `insert()` e `remove()` para essa árvore.

---

## Exercício 3

Adicione suporte a nós de texto.

---

## Exercício 4

Implemente um sistema simples de eventos armazenando handlers nos nós.

---

## Exercício 5

Crie um Renderer que imprima a árvore final no terminal em formato hierárquico.

---

# Desafio

Atualize sua **MiniVue** para suportar:

* `createRenderer()`;
* Host Operations personalizadas;
* Renderer desacoplado do DOM;
* árvore de elementos em memória;
* suporte a texto e eventos;
* renderização em uma plataforma fictícia.

---

# Projeto do capítulo

Ao final deste capítulo sua MiniVue deverá possuir um Renderer totalmente desacoplado do navegador, capaz de reutilizar toda a infraestrutura de reatividade, Scheduler e Diff para renderizar interfaces em qualquer plataforma definida por você.

---

# Checklist

* [ ] Entendi como funciona `createRenderer()`.
* [ ] Sei explicar a separação entre Runtime Core e Renderer.
* [ ] Entendi como o Vue suporta múltiplas plataformas.
* [ ] Sei implementar Host Operations personalizadas.
* [ ] Minha MiniVue possui um Custom Renderer funcional.

---

# Próximo capítulo

## **Capítulo 47 — Vue DevTools Internals: Instrumentação, Debugging e Integração com o Runtime**

No próximo capítulo estudaremos como o **Vue DevTools** se comunica com o Runtime. Você aprenderá como componentes, estado reativo, eventos, Pinia, Router e Performance Timeline são instrumentados, como o Runtime expõe informações para inspeção e como implementar um sistema básico de inspeção e depuração na sua MiniVue. Este capítulo mostrará como ferramentas profissionais conseguem visualizar toda a árvore de componentes em tempo real.
