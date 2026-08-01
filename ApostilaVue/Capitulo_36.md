# Capítulo 36 — Diretivas Customizadas: Como Funcionam Internamente e Como Criar as Suas

> **Objetivo:** compreender profundamente como funcionam as **Diretivas Customizadas** do Vue 3. Ao final deste capítulo você entenderá como o Compiler transforma diretivas, como o Renderer executa seus hooks, como criar diretivas profissionais e como implementar suporte a diretivas na sua MiniVue.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Entender o propósito das diretivas customizadas.
* Saber quando utilizá-las.
* Explicar como o Vue executa os hooks das diretivas.
* Criar diretivas reutilizáveis.
* Compreender a integração entre Compiler e Renderer.
* Implementar um sistema de diretivas na MiniVue.

---

# Pré-requisitos

* Capítulos 01 ao 35.

---

# Recapitulando

Até agora.

Aprendemos.

Como criar.

Componentes.

---

Mas.

Nem todo comportamento.

Precisa.

Ser um componente.

---

Às vezes.

Queremos apenas.

Adicionar.

Um comportamento.

A um elemento.

---

Exemplo.

```vue
<input

v-focus

/>
```

---

Ou.

```vue
<div

v-click-outside

/>
```

---

Ou ainda.

```vue
<img

v-lazy

/>
```

---

Esses.

São exemplos.

De diretivas.

---

# O que é uma diretiva?

Uma diretiva.

É um objeto.

Que controla.

O comportamento.

De um elemento.

Do DOM.

---

Ela.

Não cria.

Interface.

---

Ela modifica.

O comportamento.

Da interface.

---

# Componentes × Diretivas

Componentes.

Criam.

Estrutura.

---

Diretivas.

Controlam.

Comportamentos.

---

Exemplo.

```vue
<Modal/>
```

↓

Componente.

---

```vue
<input

v-focus

/>
```

↓

Diretiva.

---

# Quando criar uma diretiva?

Quando.

Precisamos.

Manipular.

O DOM.

Diretamente.

---

Exemplos.

* foco automático;
* detectar clique externo;
* observar interseção;
* arrastar elementos;
* atalhos de teclado;
* máscaras;
* animações.

---

# Primeiro exemplo

```javascript
const vFocus={

mounted(el){

el.focus()

}

}
```

---

Registro.

Global.

```javascript
app.directive(

"focus",

vFocus

)
```

---

Uso.

```vue
<input

v-focus

/>
```

---

Resultado.

O input.

Recebe foco.

Automaticamente.

---

# Estrutura

Uma diretiva.

É.

Um objeto.

---

```javascript
const directive={

...

}
```

---

Com hooks.

---

# Hooks

O Vue 3.

Possui.

Sete hooks.

---

```text
created
```

---

```text
beforeMount
```

---

```text
mounted
```

---

```text
beforeUpdate
```

---

```text
updated
```

---

```text
beforeUnmount
```

---

```text
unmounted
```

---

# created()

Executa.

Antes.

De qualquer.

Manipulação.

Do DOM.

---

Ideal.

Para.

Preparação.

De estado.

---

# beforeMount()

O elemento.

Já existe.

---

Mas ainda.

Não foi.

Inserido.

No DOM.

---

# mounted()

O elemento.

Já está.

No DOM.

---

É aqui.

Que normalmente.

Manipulamos.

O elemento.

---

Exemplo.

```javascript
mounted(el){

el.focus()

}
```

---

# beforeUpdate()

Executa.

Antes.

Do Patch.

---

Permite.

Preparar.

Alterações.

---

# updated()

Executa.

Depois.

Da atualização.

---

Muito útil.

Quando.

O valor.

Da diretiva.

Mudou.

---

# beforeUnmount()

Executa.

Antes.

Da remoção.

Do elemento.

---

Ideal.

Para remover.

Listeners.

---

# unmounted()

Executa.

Depois.

Da remoção.

---

Excelente.

Para limpeza.

---

# Exemplo completo

```javascript
const vLog={

mounted(){

console.log(

"Montou"

)

},

updated(){

console.log(

"Atualizou"

)

},

unmounted(){

console.log(

"Removeu"

)

}

}
```

---

# Argumentos

Cada hook.

Recebe.

Diversos parâmetros.

---

```javascript
mounted(

el,

binding,

vnode,

prevVNode

)
```

---

# el

Representa.

O elemento.

Real.

Do DOM.

---

```javascript
el.focus()
```

---

# binding

Contém.

Informações.

Da diretiva.

---

Exemplo.

```vue
<input

v-focus="ativo"

/>
```

---

Dentro.

Da diretiva.

```javascript
binding.value
```

↓

```javascript
ativo
```

---

Também.

Existe.

```javascript
binding.oldValue
```

---

Muito útil.

Em.

```text
updated()
```

---

# argument

Imagine.

```vue
<div

v-color:red

/>
```

---

Resultado.

```javascript
binding.arg
```

↓

```text
red
```

---

# Modifiers

Também existem.

Os modifiers.

---

Exemplo.

```vue
<input

v-focus.lazy

/>
```

---

Resultado.

```javascript
binding.modifiers
```

↓

```javascript
{

lazy:true

}
```

---

# Exemplo

```javascript
mounted(

el,

binding

){

if(

binding.modifiers.lazy

){

...

}

}
```

---

# v-click-outside

Uma diretiva.

Muito famosa.

---

Implementação.

Simplificada.

```javascript
const clickOutside={

mounted(el,binding){

el.__click__=(event)=>{

if(

!el.contains(

event.target

)

){

binding.value()

}

}

document.addEventListener(

"click",

el.__click__

)

},

unmounted(el){

document.removeEventListener(

"click",

el.__click__

)

}

}
```

---

Observe.

Criamos.

Um listener.

No mounted.

---

Removemos.

No unmounted.

---

Caso contrário.

Teríamos.

Memory Leak.

---

# Outro exemplo

v-color.

```javascript
mounted(

el,

binding

){

el.style.color=

binding.value

}
```

---

Uso.

```vue
<p

v-color="'red'"

/>
```

---

# Diretiva dinâmica

```vue
<p

v-color="cor"

/>
```

---

Quando.

```javascript
cor.value
```

Muda.

---

O hook.

```javascript
updated()
```

Executa.

---

```javascript
updated(

el,

binding

){

el.style.color=

binding.value

}
```

---

# Como o Compiler trata?

Imagine.

```vue
<div

v-focus

/>
```

---

O Compiler.

Não ignora.

A diretiva.

---

Ele gera.

Informações.

No VNode.

---

Simplificando.

```javascript
createVNode(

"div",

{

directives:[

...

]

}

)
```

---

O Renderer.

Encontra.

Essas diretivas.

---

Executa.

Os hooks.

Correspondentes.

---

# Fluxo

```text
Template

↓

Compiler

↓

VNode

↓

Diretivas

↓

Renderer

↓

Hooks

↓

DOM
```

---

# Atualização

Quando.

O componente.

Renderiza novamente.

---

O Renderer.

Compara.

As diretivas.

---

Se necessário.

Executa.

```text
beforeUpdate
```

↓

```text
updated
```

---

# Remoção

Quando.

O elemento.

Sai.

Do DOM.

---

O Renderer.

Executa.

```text
beforeUnmount
```

↓

```text
unmounted
```

---

Muito parecido.

Com.

Lifecycle.

Dos componentes.

---

# MiniVue

Podemos criar.

Uma estrutura.

Assim.

```javascript
function invokeDirective(

hook,

directive,

el

){

directive[hook]?.(

el

)

}
```

---

Na montagem.

```javascript
invokeDirective(

"mounted",

directive,

element

)
```

---

Na atualização.

```javascript
invokeDirective(

"updated",

directive,

element

)
```

---

Na remoção.

```javascript
invokeDirective(

"unmounted",

directive,

element

)
```

---

# Performance

As diretivas.

Executam.

Somente.

Nos elementos.

Que as possuem.

---

Elas.

Não percorrem.

Toda.

A árvore.

---

Mesmo assim.

Evite.

Manipulações.

Pesadas.

Nos hooks.

---

# Casos reais

As bibliotecas.

Utilizam.

Diretivas.

Para.

* drag and drop;
* resize;
* tooltip;
* lazy loading;
* máscara;
* observadores;
* atalhos;
* clipboard;
* autofocus.

---

# Quando NÃO usar

Não utilize.

Diretivas.

Quando.

O comportamento.

Pode ser.

Implementado.

Como componente.

Ou composable.

---

Diretivas.

Devem.

Manipular.

Principalmente.

O DOM.

---

# Código-fonte

Grande parte da lógica relacionada às diretivas está distribuída entre:

```text
packages/runtime-core/src/directives.ts
```

e

```text
packages/runtime-core/src/renderer.ts
```

O arquivo `directives.ts` contém funções responsáveis por invocar os hooks das diretivas durante as diferentes fases do ciclo de vida do elemento.

---

# Curiosidade

No Vue 2 existiam hooks como `bind` e `inserted`. No Vue 3 eles foram reformulados para se alinhar ao ciclo de vida dos componentes, surgindo hooks como `beforeMount`, `mounted`, `beforeUpdate` e `updated`. Essa mudança tornou o comportamento das diretivas muito mais consistente com o restante do framework.

---

# Resumo

Neste capítulo aprendemos que:

* Diretivas adicionam comportamento a elementos do DOM.
* Elas possuem um ciclo de vida próprio.
* O Compiler registra as diretivas no VNode.
* O Renderer executa os hooks apropriados.
* Diretivas são ideais para manipulação direta do DOM.
* A MiniVue pode implementar um sistema simples de execução de hooks.

---

# Exercícios

## Exercício 1

Implemente uma diretiva `v-focus`.

---

## Exercício 2

Crie uma diretiva `v-color` que altere dinamicamente a cor de um elemento.

---

## Exercício 3

Implemente `v-click-outside`.

---

## Exercício 4

Adicione suporte a `binding.value`, `binding.arg` e `binding.modifiers` na sua MiniVue.

---

## Exercício 5

Leia `packages/runtime-core/src/directives.ts` e identifique como o Vue invoca os hooks das diretivas durante montagem, atualização e desmontagem.

---

# Desafio

Atualize sua **MiniVue** para suportar:

* registro global de diretivas;
* hooks completos;
* `binding.value`;
* `binding.oldValue`;
* argumentos;
* modifiers;
* atualização automática durante o Patch.

---

# Projeto do capítulo

Ao final deste capítulo sua MiniVue deverá ser capaz de registrar diretivas personalizadas, armazená-las nos VNodes e executar corretamente seus hooks ao longo de todo o ciclo de vida do elemento.

---

# Checklist

* [ ] Sei explicar o propósito das diretivas customizadas.
* [ ] Entendi os sete hooks das diretivas.
* [ ] Sei utilizar `binding.value`, `arg` e `modifiers`.
* [ ] Consigo implementar diretivas reutilizáveis.
* [ ] Minha MiniVue suporta diretivas personalizadas.

---

# Próximo capítulo

## **Capítulo 37 — Plugins no Vue 3: Arquitetura, Injeção Global e Extensibilidade do Framework**

No próximo capítulo estudaremos como funcionam os **Plugins** do Vue 3. Você aprenderá como bibliotecas como **Pinia**, **Vue Router**, **Vue I18n** e **PrimeVue** se integram ao framework, como funciona `app.use()`, o mecanismo de instalação de plugins, `provide/inject` global, `app.config.globalProperties` e como implementar um sistema de plugins completo na sua MiniVue.
