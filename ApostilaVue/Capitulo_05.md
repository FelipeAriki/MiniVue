# Capítulo 05 — Heap, Garbage Collector e Gerenciamento de Memória

> **Objetivo:** entender como o JavaScript gerencia memória, onde objetos são armazenados, como funciona o Garbage Collector e como evitar vazamentos de memória em aplicações Vue.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Diferenciar Stack e Heap.
* Entender onde primitivas e objetos são armazenados.
* Explicar referências em JavaScript.
* Compreender o funcionamento do Garbage Collector.
* Identificar Memory Leaks.
* Aplicar boas práticas de gerenciamento de memória em Vue.

---

# Pré-requisitos

* Capítulo 01 — Como o JavaScript e o Navegador Funcionam
* Capítulo 02 — Event Loop
* Capítulo 03 — Microtasks e Macrotasks
* Capítulo 04 — Call Stack

---

# Revisando a Stack

No capítulo anterior vimos que a Stack guarda:

* Funções em execução.
* Variáveis locais.
* Parâmetros.
* Endereço de retorno.

Mas...

Existe um problema.

Imagine:

```javascript
const usuario = {
    nome: "Felipe",
    idade: 24
}
```

Onde esse objeto é armazenado?

Na Stack?

Não.

---

# A Heap

A Heap é uma região de memória destinada ao armazenamento de dados complexos.

Ela guarda:

* Objetos
* Arrays
* Maps
* Sets
* Funções
* Classes
* Dates
* RegExp

Enquanto isso, a Stack guarda apenas referências para esses objetos.

---

# Visualizando

```javascript
const usuario = {
    nome: "Felipe"
}
```

Internamente podemos imaginar:

```text
Stack

usuario
 │
 ▼
0x001
```

Na Heap:

```text
Heap

0x001

┌───────────────┐
│ nome: Felipe  │
└───────────────┘
```

A variável **não contém o objeto**.

Ela contém apenas um endereço.

---

# Tipos Primitivos

Primitivos normalmente são armazenados diretamente.

Exemplo:

```javascript
let idade = 24

let ativo = true

let nome = "Vue"
```

Esses valores são pequenos.

Não precisam ficar na Heap.

---

# Objetos

Objetos sempre são manipulados por referência.

```javascript
const a = {
    nome: "Vue"
}

const b = a
```

Agora:

```text
Stack

a ───────┐
         │
b ───────┘
         │
         ▼
Heap

{ nome: "Vue" }
```

Existe apenas um objeto.

---

# Alterando referências

```javascript
b.nome = "React"
```

Resultado:

```javascript
console.log(a.nome)
```

```text
React
```

Porque ambos apontam para o mesmo objeto.

---

# Cópia de objetos

Muitos iniciantes imaginam:

```javascript
const copia = usuario
```

Isso NÃO cria uma cópia.

Cria apenas outra referência.

---

# Cópia verdadeira

```javascript
const copia = {

    ...usuario

}
```

Agora teremos dois objetos diferentes.

---

# Heap cresce constantemente

Imagine:

```javascript
for(let i = 0; i < 1000; i++){

    const objeto = {

        numero: i

    }

}
```

Mil objetos foram criados.

Quando podem ser removidos?

É aqui que entra o Garbage Collector.

---

# O que é Garbage Collector?

O Garbage Collector (GC) é responsável por liberar memória automaticamente.

Você não precisa fazer:

```cpp
free()

delete()
```

Como em C ou C++.

O JavaScript faz isso automaticamente.

---

# Quando um objeto pode ser removido?

Quando ele deixa de ser acessível.

Exemplo:

```javascript
function criar(){

    const usuario = {

        nome: "Vue"

    }

}
```

Após o término da função.

Não existe mais nenhuma referência para o objeto.

Ele torna-se elegível para coleta.

---

# Conceito de Reachability

O Garbage Collector utiliza o conceito de **objetos alcançáveis**.

Pergunta:

Existe algum caminho para chegar nesse objeto?

Se:

Sim.

↓

Mantém.

Se:

Não.

↓

Remove.

---

# Exemplo

```javascript
let usuario = {

    nome: "Vue"

}
```

Existe uma referência.

Objeto permanece.

Agora:

```javascript
usuario = null
```

Resultado:

```text
Stack

usuario = null
```

O objeto ficou sem referências.

O GC poderá removê-lo.

---

# O GC remove imediatamente?

Não.

Ele decide quando executar.

Isso depende da Engine.

Não existe momento exato.

---

# Algoritmo Mark and Sweep

A maioria das Engines utiliza uma estratégia semelhante.

Etapa 1

Marca objetos alcançáveis.

```text
Raízes

↓

window

↓

document

↓

variáveis globais

↓

objetos utilizados
```

Etapa 2

Tudo que não foi marcado é removido.

---

# Memory Leak

Memory Leak significa:

Objetos que continuam ocupando memória mesmo não sendo mais necessários.

---

# Exemplo clássico

```javascript
const lista = []

setInterval(()=>{

    lista.push({

        data: Date.now()

    })

},100)
```

A lista nunca para de crescer.

A memória aumenta continuamente.

---

# Outro exemplo

```javascript
window.addEventListener("resize", atualizar)
```

Se nunca removermos:

```javascript
window.removeEventListener(...)
```

O navegador continuará mantendo referências.

Mesmo que o componente Vue tenha sido destruído.

---

# Closures

Closures também podem manter objetos vivos.

Exemplo.

```javascript
function criar(){

    const usuario = {

        nome: "Vue"

    }

    return () => usuario

}
```

Mesmo após `criar()` terminar.

O objeto continua existindo.

Porque a Closure mantém referência.

---

# Heap Snapshot

O Chrome DevTools permite visualizar objetos na memória.

Passos.

Memory

↓

Heap Snapshot

↓

Take Snapshot

Você conseguirá visualizar:

* Objetos vivos.
* Referências.
* Quantidade de memória.
* Vazamentos.

---

# Relação com Vue

Imagine:

```javascript
watch(() => {

})
```

Se o watcher nunca for destruído.

Ele continuará vivo.

Mantendo:

* estados;
* componentes;
* objetos.

Isso gera Memory Leak.

---

# Eventos

Outro erro comum.

```javascript
onMounted(()=>{

    window.addEventListener("scroll", atualizar)

})
```

Sem:

```javascript
onUnmounted(()=>{

    window.removeEventListener("scroll", atualizar)

})
```

O componente é destruído.

Mas o listener continua.

---

# Timers

Mesmo problema.

```javascript
const id = setInterval(...)
```

Sem:

```javascript
clearInterval(id)
```

O timer continuará executando.

---

# WebSockets

Imagine.

```javascript
const socket = new WebSocket(...)
```

Quando o componente sair da tela.

Precisamos:

```javascript
socket.close()
```

Caso contrário.

A conexão permanece aberta.

---

# KeepAlive

Mais adiante estudaremos KeepAlive.

Um componente mantido em cache:

* continua existindo;
* continua ocupando memória.

Isso precisa ser levado em consideração.

---

# Computed

Computed não gera Memory Leak sozinho.

Mas um computed contendo referências para estruturas gigantes pode manter objetos vivos por mais tempo que o necessário.

---

# DevTools

Ferramentas importantes.

Chrome DevTools

↓

Memory

↓

Performance

↓

Allocation Timeline

↓

Heap Snapshot

Aprender a utilizar essas ferramentas é essencial para aplicações grandes.

---

# Boas práticas

* Remova EventListeners.
* Limpe Timers.
* Feche WebSockets.
* Utilize `onUnmounted`.
* Evite guardar objetos enormes desnecessariamente.
* Evite estados globais sem necessidade.

---

# Anti-patterns

Guardar tudo em um array global.

Nunca limpar caches.

Criar milhares de Watchers.

Nunca cancelar requisições.

Nunca remover listeners.

---

# Relação com Vue

Quando estudarmos:

* `watch()`
* `watchEffect()`
* `effectScope()`
* `KeepAlive`
* `Suspense`

Você verá que todos dependem de um bom gerenciamento de memória.

---

# Resumo

Aprendemos que:

* Stack guarda referências e contexto de execução.
* Heap guarda objetos.
* Objetos vivem enquanto possuem referências.
* Garbage Collector remove objetos inacessíveis.
* Memory Leaks normalmente acontecem por referências esquecidas.
* Vue oferece mecanismos para evitar vazamentos, mas cabe ao desenvolvedor utilizá-los corretamente.

---

# Exercícios

## Exercício 1

Explique a diferença entre Stack e Heap.

---

## Exercício 2

Explique por que:

```javascript
const a = { nome: "Vue" }

const b = a
```

não cria uma cópia.

---

## Exercício 3

Pesquise o algoritmo **Mark and Sweep** utilizado pela V8.

---

## Exercício 4

Abra o Chrome DevTools.

Crie objetos.

Faça um Heap Snapshot.

Identifique quantos objetos permanecem vivos.

---

# Desafio

Crie uma aplicação que:

* adiciona listeners;
* cria timers;
* cria objetos grandes.

Depois corrija todos os possíveis Memory Leaks.

---

# Projeto do capítulo

Desenvolva um painel de monitoramento que:

* cria e remove componentes dinamicamente;
* adiciona EventListeners;
* inicia Timers;
* abre WebSockets simulados.

Utilize o Chrome DevTools para verificar se toda a memória é liberada após destruir os componentes.

---

# Checklist

* [ ] Sei diferenciar Stack e Heap.
* [ ] Entendi referências.
* [ ] Sei explicar Garbage Collector.
* [ ] Entendi Mark and Sweep.
* [ ] Sei identificar Memory Leaks.
* [ ] Sei evitar vazamentos de memória em aplicações Vue.

---

# Próximo capítulo

## **Capítulo 06 — Escopo Léxico, Closures e Contexto de Execução**

Você entenderá como o JavaScript resolve variáveis, como closures funcionam internamente e por que elas são fundamentais para a Composition API, composables, `setup()`, `watch()`, `computed()` e praticamente toda a arquitetura do Vue 3.
