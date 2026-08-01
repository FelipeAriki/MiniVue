# Capítulo 17 — O Algoritmo de Diff: Como o Vue Atualiza Listas com Máxima Eficiência

> **Objetivo:** compreender em profundidade como o Vue atualiza listas utilizando o algoritmo de Diff. Ao final deste capítulo você entenderá como funcionam `patchKeyedChildren()`, `patchUnkeyedChildren()`, a importância da propriedade `key`, o algoritmo **Longest Increasing Subsequence (LIS)** e como o Vue consegue mover o menor número possível de elementos no DOM.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Explicar como o Vue atualiza listas.
* Entender a diferença entre listas com e sem `key`.
* Compreender `patchKeyedChildren()`.
* Compreender `patchUnkeyedChildren()`.
* Implementar um Diff simplificado.
* Entender o algoritmo LIS.
* Explicar por que o `key` é tão importante.

---

# Pré-requisitos

* Capítulos 01 ao 16.

---

# O problema

Imagine.

Antes.

```vue
<ul>

<li>A</li>

<li>B</li>

<li>C</li>

</ul>
```

Depois.

```vue
<ul>

<li>A</li>

<li>C</li>

<li>D</li>

</ul>
```

O Vue precisa responder.

> O que mudou exatamente?

---

# Primeira solução

Remover tudo.

↓

Criar tudo novamente.

```text
A

B

C

↓

A

C

D
```

Funciona.

Mas desperdiça operações.

---

# O objetivo

Reaproveitar o máximo possível.

Neste exemplo.

```text
A

↓

Mesmo elemento
```

```text
C

↓

Mesmo elemento
```

Somente.

```text
B

↓

Remove
```

E.

```text
D

↓

Cria
```

---

# O algoritmo de Diff

O Vue compara.

```text
Lista antiga

↓

Lista nova
```

Encontrando.

* elementos iguais;
* elementos removidos;
* elementos adicionados;
* elementos movidos.

---

# O papel do key

Imagine.

```vue
<li

:key="produto.id"

v-for="produto in produtos"

>
```

Agora.

Cada elemento possui identidade.

---

# Visualizando

Antes.

```text
1

2

3
```

Depois.

```text
1

3

4
```

O Vue entende.

```text
1

↓

Mesmo
```

```text
2

↓

Remover
```

```text
3

↓

Mover
```

```text
4

↓

Criar
```

---

# Sem key

Agora.

```vue
<li

v-for="produto in produtos"

>
```

Sem chave.

O Vue compara apenas por posição.

---

# Exemplo

Antes.

```text
João

Maria
```

Depois.

```text
Maria

João
```

Sem `key`.

O Vue pensa.

```text
Linha 1 mudou

↓

Atualiza texto
```

```text
Linha 2 mudou

↓

Atualiza texto
```

Mas não percebe que eram as mesmas pessoas.

---

# O problema

Imagine.

```vue
<input>

<input>
```

Usuário digitou.

```text
Felipe
```

Agora a lista muda de ordem.

Sem `key`.

O valor digitado pode aparecer no input errado.

---

# Regra

Sempre utilize `key`.

Principalmente quando houver:

* formulários;
* componentes;
* animações;
* estado interno.

---

# patchUnkeyedChildren()

Quando não existe `key`.

O Vue utiliza um algoritmo simples.

---

# Fluxo

Compara.

```text
Índice 0

↓

Índice 0
```

Depois.

```text
Índice 1

↓

Índice 1
```

Até o menor tamanho.

---

# Exemplo

Antes.

```text
A

B

C
```

Depois.

```text
A

D

C
```

Resultado.

Atualiza apenas.

```text
B

↓

D
```

---

# patchKeyedChildren()

Quando existe `key`.

O algoritmo muda completamente.

Agora o Vue compara identidades.

---

# Primeira otimização

Comparar pelo início.

Imagine.

Antes.

```text
A

B

C

D
```

Depois.

```text
A

B

X

Y
```

O Vue percebe imediatamente.

```text
A

↓

Mesmo
```

```text
B

↓

Mesmo
```

Nem precisa continuar comparando.

---

# Segunda otimização

Comparar pelo final.

Antes.

```text
A

B

C

D
```

Depois.

```text
X

Y

C

D
```

Também economiza trabalho.

---

# Resultado

Depois dessas duas etapas.

Sobra apenas o trecho intermediário.

É nele que o algoritmo pesado trabalha.

---

# Exemplo

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

F

E
```

O Vue elimina.

```text
A

↓

Mesmo
```

```text
E

↓

Mesmo
```

Restando apenas.

```text
B

C

D
```

↓

```text
C

D

F
```

Muito menos trabalho.

---

# Criando um Map

Agora o Vue cria.

```javascript
const keyToNewIndexMap = new Map()
```

---

# Exemplo

Nova lista.

```text
A

C

D

F
```

Map.

```text
A → 0

C → 1

D → 2

F → 3
```

Agora localizar um elemento é praticamente instantâneo.

---

# Comparando

Elemento antigo.

```text
B
```

Existe no Map?

↓

Não.

↓

Remove.

---

Elemento.

```text
C
```

Existe?

↓

Sim.

↓

Atualiza.

---

# Ainda falta um problema

Imagine.

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

D

C
```

Todos continuam existindo.

Mas mudaram de posição.

---

# O objetivo

Mover o menor número possível de elementos.

---

# A solução

O Vue utiliza um algoritmo famoso.

```text
Longest Increasing Subsequence

(LIS)
```

---

# O que é LIS?

Imagine.

Temos a sequência.

```text
2

5

1

3

4
```

A maior subsequência crescente.

É.

```text
2

3

4
```

---

# Mas...

O que isso tem a ver com DOM?

Muito.

---

# Imagine

Posições antigas.

```text
0

2

3

1
```

LIS.

↓

```text
0

2

3
```

Esses elementos já estão na ordem correta.

Não precisam ser movidos.

Somente.

```text
1
```

Será movimentado.

---

# Resultado

O Vue move apenas os elementos que realmente precisam mudar de posição.

---

# Fluxo

```text
Lista antiga

↓

Map

↓

Comparação

↓

LIS

↓

Mover somente o necessário
```

---

# Complexidade

O algoritmo de Diff do Vue possui diversas otimizações.

A parte da LIS é executada em aproximadamente:

```text
O(n log n)
```

Muito mais eficiente do que soluções ingênuas.

---

# Exemplo completo

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

D

B

E

C
```

O Vue identifica.

* A permanece.
* D move.
* B move.
* E permanece.
* C move.

Mas.

Graças ao LIS.

Nem todos os elementos precisam ser removidos e inseridos novamente.

---

# Inserções

Imagine.

Antes.

```text
A

B
```

Depois.

```text
A

B

C
```

O algoritmo percebe.

```text
Fim da lista

↓

Inserir C
```

Sem comparar tudo novamente.

---

# Remoções

Antes.

```text
A

B

C
```

Depois.

```text
A
```

Resultado.

```text
Remover

B

↓

C
```

---

# Movimentações

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

O Vue reaproveita os três elementos.

Apenas altera suas posições no DOM.

---

# Componentes

O Diff também funciona para componentes.

```vue
<Usuario

:key="usuario.id"
/>
```

O Vue reutiliza a instância existente sempre que possível.

Isso preserva:

* estado;
* refs;
* watchers;
* computed;
* ciclo de vida.

---

# O que acontece sem key?

O Vue pode reutilizar o componente errado.

Consequências.

* formulário perde dados;
* animações quebram;
* estado interno muda de item.

---

# Regra prática

Nunca utilize.

```vue
:key="index"
```

Quando a lista puder:

* ordenar;
* filtrar;
* remover;
* inserir.

---

# Quando o índice é aceitável?

Somente quando:

* lista é completamente estática;
* nunca muda de ordem;
* nunca remove;
* nunca insere.

Mesmo assim.

Prefira uma chave real.

---

# Como o React faz?

O React também utiliza `key`.

Mas a implementação do algoritmo de Diff é diferente.

O Vue utiliza otimizações específicas, como:

* comparação pelas extremidades;
* Block Tree;
* Patch Flags;
* LIS.

---

# Performance

Imagine uma tabela com:

```text
10.000 linhas
```

Mover apenas dois itens.

Sem Diff otimizado.

↓

10.000 atualizações.

---

Com o algoritmo do Vue.

↓

Apenas os elementos realmente necessários são movimentados.

---

# Arquitetura

```text
Render

↓

Novo VNode

↓

patchChildren()

↓

patchKeyedChildren()

↓

Map

↓

LIS

↓

DOM
```

---

# Comparando

Nossa implementação.

```text
Compara posição
```

Vue.

```text
Compara início

↓

Compara fim

↓

Map

↓

LIS

↓

Move mínimo possível
```

---

# Curiosidade

O algoritmo `patchKeyedChildren()` do Vue é considerado uma das partes mais sofisticadas do Runtime Core.

Ele combina diversas estratégias em vez de depender apenas da LIS. A LIS é aplicada somente quando realmente há movimentação de elementos. Em muitos casos comuns (adições no início/fim ou remoções simples), o Vue nem chega a executar esse algoritmo, tornando as atualizações ainda mais rápidas.

---

# Resumo

Neste capítulo aprendemos que:

* `key` fornece identidade aos elementos.
* `patchUnkeyedChildren()` compara por posição.
* `patchKeyedChildren()` compara por identidade.
* O Vue otimiza comparando início e fim antes de executar o algoritmo principal.
* Um `Map` acelera a localização dos elementos.
* O algoritmo **Longest Increasing Subsequence (LIS)** reduz a quantidade de movimentações no DOM.
* Componentes também dependem de `key` para preservar seu estado interno.

---

# Exercícios

## Exercício 1

Implemente um Diff simples para listas sem `key`.

---

## Exercício 2

Implemente um `Map` que associe `key` ao índice do novo VNode.

---

## Exercício 3

Explique por que `key="index"` pode causar bugs.

---

## Exercício 4

Pesquise e implemente uma versão simplificada do algoritmo LIS.

---

# Desafio

Atualize sua **MiniVue** para suportar:

* `patchUnkeyedChildren()`;
* `patchKeyedChildren()`;
* comparação pelas extremidades;
* reutilização de elementos;
* movimentação de elementos;
* suporte a `key`.

---

# Projeto do capítulo

Ao final deste capítulo sua biblioteca deverá conseguir:

* atualizar listas sem recriar todos os elementos;
* reutilizar elementos existentes;
* inserir e remover itens corretamente;
* identificar elementos por `key`;
* mover elementos preservando o estado;
* utilizar uma implementação simplificada de LIS.

---

# Checklist

* [ ] Sei explicar por que `key` é importante.
* [ ] Entendi a diferença entre listas com e sem `key`.
* [ ] Sei explicar `patchKeyedChildren()`.
* [ ] Entendi a função do `Map` no algoritmo.
* [ ] Sei explicar o conceito de LIS.
* [ ] Minha MiniVue já consegue atualizar listas de forma eficiente.

---

# Próximo capítulo

## **Capítulo 18 — O Compilador do Vue: Como um Template se Transforma em Código JavaScript**

A partir do próximo capítulo entraremos no **Compiler** do Vue. Você aprenderá como um `<template>` é transformado em uma Render Function, estudará **Parsing**, **AST (Abstract Syntax Tree)**, **Transform Pipeline**, **Code Generation**, **Static Hoisting**, **Patch Flags**, **Block Tree** e entenderá por que grande parte da performance do Vue 3 vem do compilador — e não apenas do Virtual DOM. Este será o início da exploração do terceiro grande pilar da arquitetura do Vue.
