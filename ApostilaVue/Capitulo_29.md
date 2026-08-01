# Capítulo 29 — Longest Increasing Subsequence (LIS): O Algoritmo que Torna o Diff do Vue 3 Extremamente Eficiente

> **Objetivo:** compreender profundamente um dos algoritmos mais importantes do Runtime do Vue 3. Ao final deste capítulo você entenderá por que o Vue utiliza a **Longest Increasing Subsequence (LIS)**, implementará o algoritmo do zero, acompanhará sua utilização dentro de `patchKeyedChildren()` e entenderá como o Vue minimiza movimentações no DOM.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Entender por que mover elementos é um problema complexo.
* Explicar a Longest Increasing Subsequence (LIS).
* Implementar a LIS do zero.
* Compreender o algoritmo `patchKeyedChildren()`.
* Explicar por que o Vue utiliza LIS.
* Analisar o código-fonte do Vue relacionado ao Diff.

---

# Pré-requisitos

* Capítulos 01 ao 28.

---

# Recapitulando

No capítulo anterior aprendemos.

```text
Compiler

↓

Patch Flags

↓

Renderer

↓

Patch
```

Agora vamos estudar.

A parte mais inteligente.

Do Patch.

---

# O problema

Imagine.

Árvore antiga.

```text
A

B

C

D

E
```

Nova árvore.

```text
D

B

E

A

C
```

Todos os elementos existem.

Mas.

Mudaram de posição.

---

# Primeira solução

Remover tudo.

↓

Criar novamente.

---

Funciona.

Mas é horrível.

---

# Segunda solução

Mover.

Todos.

---

Ainda ruim.

---

# Objetivo

Mover.

O menor número possível.

De elementos.

---

# A ideia

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

# Esse é o segredo

Encontrar.

A maior sequência.

Que já está.

Na posição correta.

---

Essa sequência chama-se.

```text
Longest Increasing Subsequence
```

Ou.

```text
LIS
```

---

# O que é uma subsequência?

Imagine.

```text
1

4

2

3

5
```

Uma subsequência válida.

É.

```text
1

2

3

5
```

---

Outra.

```text
1

4

5
```

Também.

---

Mas.

```text
4

2
```

Não é crescente.

---

# O objetivo

Encontrar.

A maior.

Subsequência crescente.

---

# Exemplo

Sequência.

```text
2

5

1

8

3

9

4
```

LIS.

```text
2

5

8

9
```

Ou.

```text
2

3

4
```

Dependendo do algoritmo utilizado.

---

# O que importa?

O tamanho máximo.

---

# Relação com o Vue

Imagine.

Índices antigos.

```text
4

2

5

1

3
```

Precisamos descobrir.

Quais.

Já estão.

Na ordem correta.

---

Esses.

Não precisam ser movidos.

---

# Como o Vue faz?

Primeiro.

Cria um array.

Chamado.

```javascript
newIndexToOldIndexMap
```

---

Exemplo.

```text
Novo índice

↓

Índice antigo
```

Resultado.

```text
4

2

5

1

3
```

---

Agora.

O Vue roda.

A LIS.

Sobre esse array.

---

Resultado.

```text
2

5
```

Ou.

Outra subsequência válida.

---

Todos os elementos.

Fora.

Da LIS.

Precisam ser movidos.

---

# Visualmente

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

Índices.

```text
0

2

3

1

4
```

LIS.

```text
0

2

3

4
```

Elemento fora.

```text
1
```

↓

Mover apenas.

B.

---

# Implementação simples

Primeiro.

A versão didática.

```javascript
function lis(arr){

    const dp = new Array(arr.length).fill(1)

    let max = 1

    for(let i=0;i<arr.length;i++){

        for(let j=0;j<i;j++){

            if(arr[j]<arr[i]){

                dp[i]=Math.max(

                    dp[i],

                    dp[j]+1

                )

            }

        }

        max=Math.max(max,dp[i])

    }

    return max

}
```

---

# Complexidade

Essa implementação.

É.

```text
O(n²)
```

---

Funciona.

Mas.

Não serve.

Para um framework.

---

# O Vue

Utiliza.

Uma implementação.

Muito mais rápida.

---

Complexidade.

```text
O(n log n)
```

---

# Como?

Utilizando.

Busca binária.

---

# Ideia

Em vez.

De comparar.

Todos.

Os elementos.

Mantemos.

Uma sequência.

Ordenada.

---

Cada novo elemento.

É inserido.

Na posição correta.

Utilizando.

```text
Binary Search
```

---

# Resultado

Muito mais rápido.

Em listas grandes.

---

# O algoritmo

Simplificando.

```javascript
function getSequence(arr){

}
```

Essa função.

Existe.

No código-fonte.

Do Vue.

---

# Onde?

```text
packages/runtime-core/src/renderer.ts
```

---

Procure.

```javascript
function getSequence(...)
```

Ela está.

Praticamente.

No final.

Do arquivo.

---

# O Patch

Agora.

Vamos voltar.

Para.

```javascript
patchKeyedChildren()
```

---

Primeiro.

O Vue.

Compara.

O início.

Das listas.

---

Exemplo.

```text
A

B

C
```

↓

```text
A

B

D
```

A.

↓

Mesmo.

---

B.

↓

Mesmo.

---

Parou.

Em.

```text
C

↓

D
```

---

Depois.

Compara.

O final.

---

Exemplo.

```text
A

B

C

D
```

↓

```text
X

B

C

D
```

Começando.

Pelo fim.

---

D.

↓

Mesmo.

---

C.

↓

Mesmo.

---

B.

↓

Mesmo.

---

Agora.

Só resta.

```text
A

↓

X
```

---

Essa otimização.

Evita.

Percorrer.

Toda a lista.

---

# Região desconhecida

Depois.

Sobra.

Apenas.

O trecho.

Que realmente mudou.

---

Exemplo.

```text
A

[B

C

D]

E
```

↓

```text
A

[D

B

C]

E
```

---

Somente.

A região.

Entre colchetes.

É analisada.

---

# Mapa

Agora.

O Vue cria.

```javascript
Map<key,index>
```

---

Resultado.

```text
B→1

C→2

D→0
```

---

Cada elemento antigo.

Procura.

Sua Key.

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

O Vue.

Percorre.

A lista nova.

---

Elementos inexistentes.

↓

São montados.

---

# Movimentação

Agora.

O Vue sabe.

Quem existe.

---

Quem foi removido.

---

Quem foi criado.

---

Falta apenas.

Mover.

---

É aqui.

Que entra.

A LIS.

---

# Fluxo completo

```text
Patch

↓

Compara início

↓

Compara fim

↓

Map()

↓

newIndexToOldIndexMap

↓

LIS

↓

Move()

↓

DOM
```

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

Map.

```text
A→0

D→1

B→2

E→3

C→4
```

Índices antigos.

```text
0

3

1

4

2
```

LIS.

```text
0

3

4
```

Movimentos.

↓

Somente.

B.

E.

C.

---

# Por que funciona?

Porque.

Todos.

Os elementos.

Da LIS.

Já estão.

Na ordem correta.

---

Mover qualquer um deles.

Seria.

Desnecessário.

---

# Complexidade

Comparação inicial.

```text
O(n)
```

---

Map.

```text
O(n)
```

---

Busca.

```text
O(1)
```

---

LIS.

```text
O(n log n)
```

---

Resultado.

Excelente desempenho.

Mesmo.

Para milhares.

De elementos.

---

# Exemplo real

Imagine.

Uma tabela.

Com.

```text
10.000 linhas
```

Sem LIS.

Uma ordenação.

Poderia.

Mover.

10.000 elementos.

---

Com LIS.

Talvez.

Apenas.

200.

Precisem.

Ser movidos.

---

# Código-fonte

Abra.

```text
packages/runtime-core/src/renderer.ts
```

Procure.

```javascript
patchKeyedChildren()
```

Você encontrará aproximadamente as seguintes etapas:

1. Comparação pelo início.
2. Comparação pelo fim.
3. Montagem de novos nós.
4. Remoção de nós antigos.
5. Construção do `newIndexToOldIndexMap`.
6. Execução da `getSequence()`.
7. Movimentação mínima dos elementos.

Essa é uma excelente leitura para consolidar o entendimento.

---

# Curiosidade

A implementação da `getSequence()` no Vue possui menos de 70 linhas, mas resolve um problema clássico de Ciência da Computação com complexidade `O(n log n)`. É um dos trechos mais elegantes e estudados do Runtime.

---

# Resumo

Neste capítulo aprendemos que:

* Mover elementos é um dos problemas mais caros do Virtual DOM.
* A Longest Increasing Subsequence identifica os elementos que já estão na ordem correta.
* O Vue move apenas os elementos fora da LIS.
* `patchKeyedChildren()` combina comparação por prefixo, sufixo, `Map`, `newIndexToOldIndexMap` e LIS.
* O algoritmo possui excelente desempenho mesmo para listas muito grandes.

---

# Exercícios

## Exercício 1

Implemente uma versão `O(n²)` da LIS.

---

## Exercício 2

Implemente uma versão `O(n log n)` utilizando busca binária.

---

## Exercício 3

Implemente `newIndexToOldIndexMap` na sua MiniVue.

---

## Exercício 4

Adicione movimentação mínima de elementos utilizando a sequência retornada pela LIS.

---

## Exercício 5

Leia a implementação de `getSequence()` no código-fonte do Vue e reescreva-a com comentários explicando cada etapa.

---

# Desafio

Atualize sua **MiniVue Renderer** para:

* implementar `patchKeyedChildren()`;
* calcular a LIS;
* mover apenas os elementos necessários;
* manter a complexidade próxima da utilizada pelo Vue.

---

# Projeto do capítulo

Ao final deste capítulo sua MiniVue deverá conseguir:

* comparar listas com `key`;
* reutilizar nós existentes;
* minimizar movimentações;
* aplicar uma implementação simplificada da estratégia utilizada pelo Vue 3.

---

# Checklist

* [ ] Sei explicar o problema de movimentação de elementos.
* [ ] Entendi o conceito de Longest Increasing Subsequence.
* [ ] Sei por que o Vue utiliza a LIS.
* [ ] Consigo acompanhar o fluxo geral de `patchKeyedChildren()`.
* [ ] Minha MiniVue já realiza movimentações eficientes de elementos.

---

# Próximo capítulo

## **Capítulo 30 — Render Functions e `h()`: Criando Interfaces sem Templates**

No próximo capítulo deixaremos o Template Compiler de lado e construiremos interfaces diretamente com **Render Functions** e a função `h()`. Você aprenderá como componentes como Vue Router, Pinia, bibliotecas de UI e o próprio Runtime do Vue utilizam essa API para criar interfaces dinâmicas, além de entender quando Render Functions são superiores aos templates e como elas se conectam diretamente ao Renderer que construímos nos capítulos anteriores.
