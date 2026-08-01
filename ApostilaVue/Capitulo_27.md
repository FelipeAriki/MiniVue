# Capítulo 27 — Virtual DOM e Diffing: O Algoritmo de Patch Completo do Vue 3

> **Objetivo:** compreender profundamente como o Vue compara duas árvores de VNodes e atualiza o DOM com o menor número possível de operações. Ao final deste capítulo você entenderá o algoritmo de Diff do Vue 3, o papel das `keys`, como funciona o algoritmo da **Longest Increasing Subsequence (LIS)** e por que o Vue é um dos frameworks mais rápidos da atualidade.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Explicar o Virtual DOM.
* Entender como funciona o algoritmo de Diff.
* Explicar a importância das `key`.
* Implementar o Patch de filhos.
* Entender a Longest Increasing Subsequence (LIS).
* Explicar como o Vue minimiza operações no DOM.
* Implementar uma versão simplificada do algoritmo de Patch.

---

# Pré-requisitos

* Capítulos 01 ao 26.

---

# Recapitulando

No capítulo anterior aprendemos.

```text
Render Function

↓

VNode

↓

Renderer

↓

DOM
```

Mas ainda existe uma pergunta.

Como o Vue sabe exatamente o que mudou?

---

# A resposta

Através do.

```text
Virtual DOM
```

---

# O que é o Virtual DOM?

É uma representação em memória da interface.

Por exemplo.

```html
<ul>

    <li>A</li>

    <li>B</li>

</ul>
```

Não é manipulado diretamente.

Primeiro é transformado em.

```javascript
{

type:"ul",

children:[

{

type:"li",

children:"A"

},

{

type:"li",

children:"B"

}

]

}
```

Esse objeto é uma.

```text
VNode
```

---

# O ciclo

```text
Estado

↓

Render()

↓

VNode

↓

Patch()

↓

DOM
```

Sempre que o estado muda.

Uma nova árvore de VNodes é criada.

---

# O objetivo

Imagine.

Árvore antiga.

```text
A

B

C
```

Nova árvore.

```text
A

B

D
```

Não faz sentido reconstruir tudo.

Precisamos descobrir.

Que apenas.

```text
C

↓

D
```

Mudou.

---

# O Diff

É exatamente esse algoritmo.

Comparar.

```text
VNode Antiga

↓

VNode Nova
```

Encontrando.

A menor quantidade possível.

De alterações.

---

# Primeira comparação

Antes de qualquer coisa.

O Vue pergunta.

```javascript
oldVNode.type

===

newVNode.type
```

---

# Se forem diferentes

```javascript
<div>

↓

<span>
```

Não existe reaproveitamento.

O Vue remove.

```text
DIV
```

E cria.

```text
SPAN
```

---

# Se forem iguais

Agora podemos atualizar.

---

# Exemplo

Antes.

```html
<div class="red">

Felipe

</div>
```

Depois.

```html
<div class="blue">

Lucas

</div>
```

O elemento é o mesmo.

Apenas.

* props
* texto

Precisam mudar.

---

# Patch Element

Fluxo.

```text
Elemento

↓

Props

↓

Children
```

---

# Primeiro

Atualizamos.

Props.

---

# Depois

Atualizamos.

Filhos.

---

# Os quatro casos

Quando falamos de filhos.

Existem apenas quatro possibilidades.

---

## Caso 1

Texto.

↓

Texto.

---

Antes.

```text
Felipe
```

Depois.

```text
Lucas
```

Executamos.

```javascript
setElementText()
```

---

## Caso 2

Texto.

↓

Array.

Antes.

```html
<div>

Olá

</div>
```

Depois.

```html
<div>

<p>A</p>

<p>B</p>

</div>
```

Precisamos.

Remover.

O texto.

E montar.

Os filhos.

---

## Caso 3

Array.

↓

Texto.

Antes.

```html
<div>

<p>A</p>

<p>B</p>

</div>
```

Depois.

```html
<div>

Fim

</div>
```

Removemos.

Todos os filhos.

Depois.

Inserimos.

O texto.

---

## Caso 4

Array.

↓

Array.

Esse é.

O caso difícil.

---

# Comparando Arrays

Imagine.

Antes.

```text
A

B

C
```

Depois.

```text
A

B

C
```

Nada muda.

---

Agora.

Antes.

```text
A

B

C
```

Depois.

```text
A

B

C

D
```

Basta inserir.

```text
D
```

---

Agora.

Antes.

```text
A

B

C
```

Depois.

```text
B

C
```

Precisamos remover.

```text
A
```

---

Até aqui.

É simples.

---

# O problema real

Imagine.

Antes.

```text
A

B

C

D

E
```

Depois.

```text
D

B

E

A

C
```

Agora.

Tudo mudou de posição.

---

# Sem algoritmo

Poderíamos fazer.

```text
Remove tudo

↓

Cria tudo novamente
```

Funciona.

Mas é extremamente lento.

---

# O objetivo

Mover.

Apenas.

O necessário.

---

# Keys

Agora entra.

Um conceito fundamental.

```vue
<li

v-for="item in lista"

:key="item.id"

>
```

---

# O que é uma Key?

É a identidade.

Do elemento.

---

Sem Key.

O Vue apenas compara.

Posições.

---

Com Key.

O Vue compara.

Identidades.

---

# Exemplo

Sem Key.

Antes.

```text
A

B

C
```

Depois.

```text
C

A

B
```

O Vue pode acreditar.

Que tudo mudou.

---

Com Key.

```text
1

2

3
```

↓

```text
3

1

2
```

O Vue sabe.

Que são os mesmos elementos.

Apenas.

Mudaram de posição.

---

# Por isso

Nunca use.

```vue
:key="index"
```

Quando a ordem puder mudar.

---

# Mapeamento

Primeiro.

O Vue cria.

```javascript
Map()
```

---

Exemplo.

```text
Novo Array

↓

Key

↓

Índice
```

---

Resultado.

```javascript
{

10:0,

20:1,

30:2

}
```

Agora.

Encontrar um elemento.

É.

```text
O(1)
```

---

# Comparando

Cada elemento antigo.

Procura.

Sua Key.

No novo Array.

---

Se encontrou.

↓

Atualiza.

---

Se não encontrou.

↓

Remove.

---

# Inserções

Depois.

O Vue percorre.

O novo Array.

---

Se encontrou.

Elemento novo.

↓

Monta.

---

# Ainda existe um problema

Mover elementos.

---

# Exemplo

Antes.

```text
A

B

C

D
```

Depois.

```text
B

A

C

D
```

O que mover?

---

# Primeira ideia

Mover.

Tudo.

---

Mas.

Isso ainda gera.

Operações desnecessárias.

---

# Longest Increasing Subsequence

Agora.

Entramos.

Na parte mais sofisticada.

Do algoritmo.

---

# A ideia

Encontrar.

A maior sequência.

Que já está.

Na ordem correta.

---

Exemplo.

Antes.

```text
A

B

C

D

E
```

Depois.

```text
A

C

D

B

E
```

Observe.

```text
A

C

D

E
```

Já estão.

Na ordem correta.

---

Então.

Só precisamos mover.

```text
B
```

---

# Outro exemplo

Índices.

```text
2

3

5

7

9
```

Essa sequência.

Já está crescente.

---

Não precisa mover.

Nenhum desses elementos.

---

# Resultado

Movemos.

Somente.

Os elementos.

Que não pertencem.

À LIS.

---

# Complexidade

Sem LIS.

Poderíamos mover.

Todos.

---

Com LIS.

Movemos.

O mínimo possível.

---

# Complexidade

A implementação do Vue.

Executa aproximadamente.

```text
O(n log n)
```

Para encontrar.

A LIS.

---

# Fluxo completo

```text
VNode Antiga

↓

Patch

↓

Patch Children

↓

Diff

↓

Keys

↓

Map()

↓

LIS

↓

Mover

↓

DOM
```

---

# Exemplo Visual

Antes.

```text
A

B

C

D

E
```

Depois.

```text
A

C

D

B

E
```

LIS.

```text
A

C

D

E
```

Movimento.

```text
B
```

Apenas.

---

# Sem Keys

O Vue utiliza.

Uma estratégia.

Mais simples.

---

Resultado.

Mais operações.

Mais Render.

---

# Por isso

As Keys.

São obrigatórias.

Em listas dinâmicas.

---

# Patch Recursivo

Lembre.

Cada filho.

Também é um VNode.

Então.

```javascript
patch(

oldChild,

newChild

)
```

É chamado.

Recursivamente.

---

# Toda a árvore

É comparada.

```text
App

↓

Header

↓

Menu

↓

Item
```

Cada nó.

Passa.

Pelo algoritmo.

---

# O DOM

Só é atualizado.

Quando realmente necessário.

---

# Comparando

Antes.

```text
VNode

↓

DOM
```

Agora.

```text
VNode Antiga

↓

VNode Nova

↓

Diff

↓

Patch

↓

DOM
```

---

# Arquivos reais

Grande parte desse algoritmo está em.

```text
packages/runtime-core/src/renderer.ts
```

Funções importantes.

```text
patch()

patchChildren()

patchKeyedChildren()

patchUnkeyedChildren()

move()

unmount()
```

---

# O algoritmo

O método.

```text
patchKeyedChildren()
```

Possui centenas de linhas.

E concentra.

Grande parte.

Da inteligência.

Do Runtime.

---

# Comparando

Nossa MiniVue.

```text
Render

↓

DOM
```

Vue.

```text
Render

↓

VNode

↓

Patch

↓

Diff

↓

LIS

↓

DOM
```

---

# Curiosidade

A utilização da **Longest Increasing Subsequence (LIS)** não foi inventada pelo Vue. Trata-se de um algoritmo clássico da Ciência da Computação aplicado de forma bastante engenhosa ao problema de movimentação de nós no DOM. A ideia é identificar quais elementos já estão na ordem correta e mover apenas os demais, reduzindo significativamente o número de operações de inserção e remoção.

---

# Resumo

Neste capítulo aprendemos que:

* O Virtual DOM representa a interface em memória.
* O algoritmo de Diff compara duas árvores de VNodes.
* O Patch atualiza apenas o necessário.
* `key` representa a identidade de um elemento.
* O Vue utiliza `Map` para localizar elementos rapidamente.
* A Longest Increasing Subsequence reduz drasticamente a quantidade de movimentações no DOM.
* O método `patchKeyedChildren()` é um dos algoritmos mais importantes do Runtime.

---

# Exercícios

## Exercício 1

Implemente um algoritmo simples de comparação entre duas listas de VNodes.

---

## Exercício 2

Implemente suporte a `key` na sua MiniVue.

---

## Exercício 3

Implemente `patchChildren()` para os quatro casos (texto→texto, texto→array, array→texto e array→array).

---

## Exercício 4

Pesquise o algoritmo da Longest Increasing Subsequence e implemente uma versão simplificada.

---

## Exercício 5

Abra `packages/runtime-core/src/renderer.ts` e acompanhe a execução de `patchKeyedChildren()` identificando onde ocorre a criação do `Map`, a busca por chaves e a etapa de movimentação dos elementos.

---

# Desafio

Atualize sua **MiniVue Renderer** para suportar:

* `key` em listas;
* `patchChildren()`;
* comparação entre listas;
* movimentação de elementos;
* uma implementação simplificada da LIS.

---

# Projeto do capítulo

Ao final deste capítulo sua biblioteca deverá conseguir:

* comparar duas árvores de VNodes;
* reutilizar elementos existentes;
* mover elementos sem recriá-los;
* minimizar alterações no DOM utilizando um algoritmo inspirado no Vue.

---

# Checklist

* [ ] Sei explicar o que é o Virtual DOM.
* [ ] Entendi como funciona o algoritmo de Diff.
* [ ] Sei por que `key` é importante.
* [ ] Entendi o papel do `Map` em `patchKeyedChildren()`.
* [ ] Sei explicar a ideia da Longest Increasing Subsequence.
* [ ] Consigo acompanhar a lógica geral de `patchKeyedChildren()`.
* [ ] Minha MiniVue já possui um algoritmo básico de Diff.

---

# Próximo capítulo

## **Capítulo 28 — Otimizações do Compiler: Patch Flags, Shape Flags, Block Tree e Renderização Otimizada**

Agora que você entende como o Renderer compara duas árvores de VNodes, veremos como o **Compiler** trabalha junto com ele para tornar esse processo muito mais rápido. Você aprenderá o funcionamento de **Patch Flags**, **Shape Flags**, **Block Tree**, **Static Hoisting**, **Tree Flattening** e outras otimizações que fazem o Vue evitar comparações desnecessárias durante o processo de atualização. Este capítulo conecta definitivamente o **Compiler** ao **Renderer**, revelando uma das maiores vantagens arquiteturais do Vue 3.
