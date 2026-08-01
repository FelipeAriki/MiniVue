# 🚀 MiniVue - Construindo um Framework Inspirado no Vue.js do Zero

> Um guia completo para entender **como o Vue funciona internamente**, construindo passo a passo uma implementação inspirada na arquitetura do Vue 3.

---

# 📖 Sobre o projeto

O **MiniVue** é um projeto educacional criado para ensinar, na prática, a arquitetura de um framework frontend moderno.

Ao longo de **60 capítulos**, o leitor constrói uma implementação própria inspirada no **Vue.js 3**, compreendendo desde os conceitos fundamentais de reatividade até recursos avançados como:

* Compiler
* Runtime
* Virtual DOM
* Scheduler
* Componentes
* SSR
* Streaming
* Hydration
* Router
* Store
* DevTools
* CLI
* Testes
* Performance
* Open Source

O objetivo **não é copiar o código do Vue**, mas compreender as ideias, algoritmos e decisões arquiteturais que tornam o framework um dos mais modernos do ecossistema JavaScript.

---

# 🎯 Objetivos

Ao concluir este material você será capaz de:

* Entender profundamente como o Vue funciona internamente.
* Criar seu próprio sistema de reatividade.
* Construir um Virtual DOM.
* Implementar um Renderer.
* Desenvolver um Compiler.
* Criar um Scheduler.
* Implementar Componentes.
* Construir um Router.
* Desenvolver uma Store inspirada no Pinia.
* Entender Server Side Rendering.
* Implementar Hydration.
* Ler o código-fonte oficial do Vue com muito mais facilidade.
* Desenvolver frameworks e bibliotecas frontend.

---

# 📚 Público-alvo

Este material é indicado para:

* Desenvolvedores Vue.js
* Desenvolvedores Front-end
* Desenvolvedores Full Stack
* Engenheiros de Software
* Estudantes de Arquitetura de Software
* Pessoas interessadas em Compiladores
* Pessoas interessadas em Frameworks JavaScript

---

# 📋 Pré-requisitos

É recomendado possuir conhecimento básico de:

* HTML
* CSS
* JavaScript ES6+
* DOM
* Módulos ES
* npm

Conhecimentos em Vue ajudam, mas **não são obrigatórios**.

---

# 🛠 Tecnologias estudadas

Durante o curso diversos conceitos e tecnologias são abordados:

* JavaScript
* TypeScript (conceitos)
* Proxy
* Reflect
* WeakMap
* Map
* Set
* Virtual DOM
* AST
* Parser
* Compiler
* Code Generator
* Scheduler
* SSR
* Hydration
* Streaming SSR
* Monorepo
* CI/CD
* GitHub Actions
* npm
* DevTools

---

# 📖 Estrutura do curso

O material está dividido em **60 capítulos**, organizados em uma sequência progressiva.

## Módulo 1 — Fundamentos

* Introdução
* Arquitetura do Vue
* Como funciona um Framework
* Introdução à Reactivity

---

## Módulo 2 — Sistema de Reactividade

Neste módulo você implementará:

* ref()
* reactive()
* readonly()
* shallowReactive()
* computed()
* watch()
* watchEffect()
* Effect
* Dependency Tracking
* Scheduler

---

## Módulo 3 — Runtime

Construção do Runtime:

* createApp()
* App Context
* Component Instance
* Lifecycle
* Render Pipeline

---

## Módulo 4 — Virtual DOM

Implementação completa de:

* VNode
* h()
* Renderer
* Patch
* Diff
* Fragment
* Text Nodes
* Keys

---

## Módulo 5 — Componentes

Construção de:

* Props
* Slots
* Emits
* Provide / Inject
* Async Components
* Teleport
* KeepAlive
* Suspense
* Transition

---

## Módulo 6 — Compiler

Construção completa do Compiler:

* Lexer
* Parser
* AST
* Transform
* Code Generator
* Render Function

---

## Módulo 7 — Renderer Avançado

Implementação de:

* Patch Flags
* Block Tree
* Static Hoisting
* Diff Otimizado
* Longest Increasing Subsequence (LIS)

---

## Módulo 8 — Ecossistema

Construção de:

* Mini Router
* Mini Store
* Plugins
* DevTools
* CLI

---

## Módulo 9 — Server Side Rendering

Implementação de:

* renderToString()
* Hydration
* Streaming
* Partial Hydration
* Islands Architecture

---

## Módulo 10 — Projeto Final

Construção completa da MiniVue incluindo:

* Testes
* Benchmark
* Performance
* Publicação Open Source
* Comparação com o Vue Oficial

---

# 📂 Organização sugerida do repositório

```text
MiniVue/

├── chapters/
│   ├── 01/
│   ├── 02/
│   ├── ...
│   └── 60/
│
├── examples/
│
├── source/
│
├── diagrams/
│
├── assets/
│
├── LICENSE
│
└── README.md
```

Caso você implemente a MiniVue durante os estudos:

```text
MiniVue/

packages/

reactivity/

runtime-core/

runtime-dom/

compiler-core/

compiler-dom/

router/

store/

shared/

examples/

tests/

docs/
```

---

# 📑 Conteúdo abordado

Durante os 60 capítulos são explorados:

## Reactivity

* ref
* reactive
* readonly
* computed
* watch
* EffectScope
* Scheduler

---

## Runtime

* Component Instance
* Lifecycle
* Setup
* Render Function
* App Context

---

## Renderer

* Virtual DOM
* Patch
* Diff
* Fragment
* Teleport
* Suspense
* Transition
* KeepAlive

---

## Compiler

* Parser
* AST
* Transform
* Codegen
* Runtime Compiler

---

## Performance

* Patch Flags
* Static Hoisting
* Block Tree
* Scheduler
* LIS
* Benchmarks

---

## SSR

* renderToString
* Hydration
* Streaming
* Initial State

---

## Ferramentas

* Router
* Store
* DevTools
* Plugins
* CLI

---

# 🎓 O que você aprenderá

Ao finalizar este projeto você terá aprendido:

* Como funciona a Reactivity do Vue.
* Como construir um Compiler.
* Como implementar um Virtual DOM.
* Como funciona um Renderer moderno.
* Como implementar um Scheduler.
* Como construir um Router.
* Como desenvolver uma Store.
* Como implementar SSR.
* Como criar um framework inspirado no Vue.

Mais importante:

Você entenderá **por que** cada uma dessas peças existe e como elas trabalham juntas.

---

# 📚 Referências utilizadas

Este material foi inspirado em conceitos presentes em:

* Vue.js
* Vue Router
* Pinia
* Vite
* ECMAScript
* DOM Specification
* Compiler Theory
* Design Patterns
* Estruturas de Dados
* Algoritmos

Todo o conteúdo foi produzido com finalidade **educacional**, utilizando arquitetura inspirada no Vue, sem reproduzir seu código-fonte.

---

# ⚠ Aviso

Este projeto **não é uma implementação oficial do Vue.js**.

A MiniVue foi desenvolvida exclusivamente para fins de estudo, visando facilitar a compreensão dos conceitos e da arquitetura utilizados por frameworks modernos.

---

# 🤝 Como contribuir

Contribuições são bem-vindas!

Você pode contribuir de diversas formas:

* Corrigindo erros de digitação.
* Melhorando exemplos.
* Corrigindo diagramas.
* Abrindo Issues.
* Enviando Pull Requests.
* Sugerindo novos conteúdos.

Caso encontre algum problema:

1. Abra uma Issue.
2. Explique o problema.
3. Caso possível, envie uma sugestão de melhoria.

---

# ⭐ Roadmap

Algumas ideias futuras para evolução deste material:

* [ ] Atualização para novas versões do Vue.
* [ ] Apêndice sobre Vite.
* [ ] Apêndice sobre Nuxt.
* [ ] Estudo completo do Compiler SFC.
* [ ] Vapor Mode.
* [ ] Fine-grained Reactivity.
* [ ] Server Components.
* [ ] Estudos sobre Signals.
* [ ] Renderer para Canvas.
* [ ] Renderer para Terminal.

---

# 📄 Licença

Este material pode ser disponibilizado sob a licença **MIT**, caso deseje permitir reutilização com atribuição.

---

# 🙏 Agradecimentos

Agradeço a todos os mantenedores do ecossistema Vue.js pelo excelente trabalho realizado ao longo dos anos.

Este projeto existe graças ao conhecimento compartilhado pela comunidade Open Source.

---

# 📌 Considerações finais

A melhor forma de aprender um framework não é apenas utilizá-lo, mas compreender sua arquitetura.

A MiniVue foi criada exatamente com esse propósito: permitir que qualquer desenvolvedor entenda como um framework moderno é construído, desde os primeiros conceitos até funcionalidades avançadas.

Espero que este material contribua para sua evolução como desenvolvedor e desperte o interesse pela engenharia de software, compiladores, arquitetura de frameworks e projetos Open Source.

Se este repositório foi útil para você, considere deixar uma ⭐ no GitHub. Isso ajuda outras pessoas a encontrarem o projeto e incentiva sua evolução.

---

## Bons estudos e boa construção da sua MiniVue! 🚀
