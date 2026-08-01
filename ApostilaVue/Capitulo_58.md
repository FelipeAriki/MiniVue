# Capítulo 58 — Projeto Final IX: Publicando sua MiniVue como um Framework Open Source

> **Objetivo:** transformar a MiniVue em um projeto Open Source de nível profissional, organizando um monorepo, configurando versionamento semântico, documentação, automação de releases, publicação no npm, integração com GitHub Actions e criando uma estrutura preparada para receber contribuições da comunidade.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Organizar um framework em formato de monorepo.
* Aplicar Semantic Versioning (SemVer).
* Automatizar builds e releases.
* Publicar pacotes no npm.
* Configurar documentação profissional.
* Preparar seu projeto para contribuições Open Source.

---

# Pré-requisitos

* Capítulos 01 ao 57.

---

# Introdução

Nossa MiniVue.

Agora.

Possui.

---

Compiler.

---

Runtime.

---

Renderer.

---

Reactivity.

---

Router.

---

Store.

---

SSR.

---

DevTools.

---

CLI.

---

Testes.

---

Mas.

Ainda.

Está.

No computador.

Do desenvolvedor.

---

O próximo.

Passo.

É.

Compartilhar.

---

Com.

O mundo.

---

# O ciclo de vida

Um framework.

Não termina.

Quando.

É escrito.

---

Ele.

Precisa.

Ser.

---

Publicado.

---

Versionado.

---

Documentado.

---

Mantido.

---

Evoluído.

---

# Open Source

Um projeto.

Open Source.

Não é.

Apenas.

Código.

---

Também.

É.

Comunidade.

---

Documentação.

---

Automação.

---

Governança.

---

# Estrutura do projeto

Nossa MiniVue.

Pode seguir.

Uma organização.

Semelhante.

Ao Vue.

```text id="9m7v2d"
mini-vue/

packages/

scripts/

examples/

playground/

docs/

.github/
```

---

Cada pasta.

Possui.

Uma função.

Específica.

---

# O monorepo

Em vez.

De vários.

Repositórios.

---

Utilizamos.

Um único.

Monorepo.

---

Visualmente.

```text id="wb9r5u"
Repository

├── compiler

├── runtime

├── reactivity

├── router

├── store

├── cli

└── shared
```

---

Todos.

Compartilham.

Dependências.

---

Ferramentas.

---

Configurações.

---

# Workspace

Os pacotes.

São.

Gerenciados.

Como.

Workspaces.

---

Fluxo.

```text id="3tcb4e"
Root

↓

Packages

↓

Shared Dependencies
```

---

Isso.

Simplifica.

O desenvolvimento.

---

# Versionamento

Todo.

Framework.

Precisa.

Ter.

Versões.

---

Utilizamos.

Semantic Versioning.

---

Formato.

```text id="crqgmh"
MAJOR.MINOR.PATCH
```

---

Exemplo.

```text id="gln4s9"
1.4.2
```

---

# Significado

MAJOR.

Quebra.

Compatibilidade.

---

MINOR.

Nova.

Funcionalidade.

---

PATCH.

Correção.

De bugs.

---

Assim.

Os usuários.

Sabem.

O impacto.

Da atualização.

---

# Build

Antes.

De publicar.

Precisamos.

Gerar.

Os bundles.

---

Fluxo.

```text id="0m4a7w"
Source

↓

Compiler

↓

Bundle

↓

dist/
```

---

Podemos.

Gerar.

---

ESM.

---

CommonJS.

---

UMD.

---

Type Definitions.

---

# Package.json

Cada.

Pacote.

Possui.

Seu.

Próprio.

Manifesto.

---

Exemplo.

```text id="d7p3kx"
name

version

exports

types

license

repository
```

---

Essas.

Informações.

São.

Publicadas.

No npm.

---

# Publicação

Depois.

Do Build.

Executamos.

```bash id="yrw4ps"
npm publish
```

---

O pacote.

Passa.

A estar.

Disponível.

Para.

Toda.

A comunidade.

---

# Changelog

Toda.

Versão.

Precisa.

Explicar.

As mudanças.

---

Exemplo.

```text id="yebm3j"
v1.3.0

• Novo Scheduler

• Melhorias no Diff

• Correções no SSR
```

---

Isso.

Facilita.

A atualização.

---

# Releases

No GitHub.

Criamos.

Uma.

Release.

---

Ela.

Contém.

---

Versão.

---

Notas.

---

Arquivos.

---

Links.

---

# Automação

Todo.

Release.

Pode.

Ser.

Automático.

---

Fluxo.

```text id="4ykq5k"
Merge

↓

Tests

↓

Build

↓

Publish

↓

Release
```

---

Sem.

Intervenção.

Manual.

---

# GitHub Actions

Criamos.

Um workflow.

---

```text id="r3u9na"
Push

↓

CI

↓

Build

↓

Publish
```

---

Caso.

Tudo.

Passe.

---

Uma nova.

Versão.

É publicada.

---

# Documentação

Um framework.

Precisa.

De documentação.

Completa.

---

Estrutura.

```text id="3ef2kd"
Guide

API

Examples

Tutorials

Migration

FAQ
```

---

Quanto.

Melhor.

A documentação.

---

Maior.

A adoção.

---

# Playground

Também.

Criamos.

Um ambiente.

Para.

Testes.

---

Fluxo.

```text id="w8f2mb"
Código

↓

Playground

↓

Resultado
```

---

O usuário.

Pode.

Experimentar.

Sem.

Criar.

Um projeto.

---

# Exemplos

Outro.

Recurso.

Muito.

Importante.

---

Criamos.

Aplicações.

Demonstrando.

---

Router.

---

Store.

---

SSR.

---

Transitions.

---

Reactivity.

---

Isso.

Facilita.

O aprendizado.

---

# Contribuições

Um projeto.

Open Source.

Recebe.

Pull Requests.

---

Precisamos.

Definir.

Regras.

---

```text id="q0c6yb"
Fork

↓

Branch

↓

Commit

↓

Pull Request

↓

Review

↓

Merge
```

---

# Guia do contribuidor

Criamos.

Um arquivo.

```text id="vh0qz8"
CONTRIBUTING.md
```

---

Explicando.

---

Como.

Executar.

O projeto.

---

Como.

Enviar.

PRs.

---

Como.

Escrever.

Testes.

---

Como.

Reportar.

Bugs.

---

# Código de conduta

Também.

Criamos.

```text id="9z2vua"
CODE_OF_CONDUCT.md
```

---

Garantindo.

Um ambiente.

Respeitoso.

Para.

Todos.

---

# Licença

Precisamos.

Escolher.

Uma licença.

---

Exemplos.

```text id="vdfb5g"
MIT

Apache 2.0

GPL
```

---

A licença.

Define.

Como.

O código.

Pode.

Ser utilizado.

---

# Issues

Usuários.

Podem.

Abrir.

---

Bugs.

---

Sugestões.

---

Melhorias.

---

Discussões.

---

Tudo.

Fica.

Centralizado.

---

# Templates

Criamos.

Modelos.

Para.

---

Bug Report.

---

Feature Request.

---

Pull Request.

---

Isso.

Padroniza.

As contribuições.

---

# Roadmap

Também.

Mantemos.

Um plano.

De evolução.

---

Exemplo.

```text id="7ik7bo"
v2.0

Concurrent Rendering

Signals

Compiler Optimizations
```

---

Os usuários.

Sabem.

O futuro.

Do projeto.

---

# Organização final

Nossa MiniVue.

Agora.

Possui.

```text id="3pjlwm"
Compiler

Runtime

Renderer

SSR

Router

Store

CLI

DevTools

Documentation

CI

Release
```

---

Está.

Pronta.

Para.

Ser utilizada.

Por.

Outros.

Desenvolvedores.

---

# Código-fonte

Vale estudar a estrutura organizacional dos seguintes projetos:

```text id="0jjzrk"
vuejs/core
```

---

```text id="9ngg7k"
vuejs/router
```

---

```text id="a2d5kg"
vuejs/pinia
```

---

```text id="v36v4z"
vuejs/devtools
```

---

```text id="xh1yn9"
vuejs/create-vue
```

---

Esses repositórios mostram como a equipe do Vue organiza monorepos, documentação, releases, automação e manutenção de longo prazo.

---

# Curiosidade

O Vue utiliza automação para grande parte do processo de publicação. Commits seguem convenções específicas, os changelogs podem ser gerados automaticamente e os pacotes são publicados de forma coordenada, garantindo que todas as bibliotecas do ecossistema permaneçam compatíveis entre si.

---

# Resumo

Neste capítulo aprendemos que:

* Um framework profissional precisa de um ecossistema de publicação.
* Monorepos simplificam o gerenciamento de múltiplos pacotes.
* Semantic Versioning comunica claramente o impacto das mudanças.
* CI/CD automatiza builds, testes e releases.
* Documentação e exemplos são essenciais para adoção.
* Um projeto Open Source depende de processos claros para colaboração.

---

# Exercícios

## Exercício 1

Organize a MiniVue em um monorepo contendo todos os pacotes desenvolvidos ao longo do curso.

---

## Exercício 2

Configure uma pipeline de GitHub Actions para executar testes e publicar automaticamente uma nova versão.

---

## Exercício 3

Escreva um `CONTRIBUTING.md` explicando como contribuir com a MiniVue.

---

## Exercício 4

Implemente um playground onde usuários possam testar componentes da MiniVue diretamente no navegador.

---

## Exercício 5

Crie um changelog seguindo o padrão Semantic Versioning para três versões fictícias da MiniVue.

---

# Desafio

Transforme sua **MiniVue** em um projeto Open Source completo implementando:

* monorepo;
* publicação no npm;
* Semantic Versioning;
* GitHub Actions;
* documentação;
* playground;
* guias de contribuição;
* releases automatizadas.

O objetivo é deixar o framework pronto para ser utilizado, mantido e evoluído por uma comunidade de desenvolvedores.

---

# Projeto do capítulo

Ao final deste capítulo sua MiniVue deverá estar preparada para distribuição pública, com uma infraestrutura completa de documentação, automação, versionamento e colaboração, seguindo práticas utilizadas pelos principais projetos Open Source do ecossistema JavaScript.

---

# Checklist

* [ ] Organizei a MiniVue em um monorepo.
* [ ] Configurei Semantic Versioning.
* [ ] Automatizei testes, builds e releases.
* [ ] Preparei documentação e playground.
* [ ] Estruturei o projeto para receber contribuições externas.

---

# Próximo capítulo

## **Capítulo 59 — Projeto Final X: Comparando sua MiniVue com o Vue 3 Oficial (Arquitetura, Performance, Trade-offs e Limitações)**

No próximo capítulo faremos uma análise completa da MiniVue em comparação com o Vue 3 oficial. Estudaremos as diferenças arquiteturais, decisões de design, otimizações de performance, funcionalidades avançadas ainda não implementadas e os principais trade-offs entre uma implementação educacional e um framework utilizado em produção.
