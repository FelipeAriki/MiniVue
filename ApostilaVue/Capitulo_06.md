# Capítulo 06 — Escopo Léxico, Closures e Contexto de Execução

> **Objetivo:** compreender como o JavaScript encontra variáveis, como closures funcionam internamente e por que elas são um dos pilares da Composition API, dos Composables e de praticamente toda a arquitetura do Vue 3.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Entender Escopo Léxico (Lexical Scope).
* Explicar como o JavaScript procura variáveis.
* Compreender Closures profundamente.
* Entender Execution Context.
* Relacionar Closures com Composables.
* Entender por que `setup()` funciona.
* Preparar a base para estudar Reatividade.

---

# Pré-requisitos

* Capítulo 01 ao 05.

---

# O que é Escopo?

Escopo é o conjunto de variáveis que podem ser acessadas em determinado ponto do programa.

Imagine uma casa.

Cada cômodo possui seus próprios objetos.

Quem está na sala consegue enxergar os objetos da sala.

Quem está no quarto enxerga os objetos do quarto.

Mas uma pessoa no quarto não consegue enxergar diretamente um objeto que existe apenas na cozinha.

O JavaScript funciona exatamente assim.

---

# Tipos de Escopo

O JavaScript possui três tipos principais.

## Escopo Global

Variáveis declaradas fora de qualquer função.

```javascript
const framework = "Vue"
```

Pode ser acessada por praticamente todo o programa.

---

## Escopo de Função

```javascript
function exemplo(){

    const nome = "Felipe"

}
```

A variável `nome` só existe dentro dessa função.

---

## Escopo de Bloco

Introduzido com `let` e `const`.

```javascript
if(true){

    const mensagem = "Olá"

}
```

A variável desaparece ao sair do bloco.

---

# Como o JavaScript encontra uma variável?

Imagine:

```javascript
const linguagem = "JavaScript"

function frontend(){

    console.log(linguagem)

}

frontend()
```

O processo é:

```text
frontend()

↓

Existe "linguagem" aqui?

↓

Não

↓

Procure no escopo pai

↓

Encontrou

↓

Utilize esse valor
```

Esse processo é chamado de **Lexical Lookup**.

---

# O que significa "Léxico"?

"Léxico" significa que o escopo é determinado **pela posição do código durante a escrita**, e não pelo local onde a função é chamada.

Observe:

```javascript
const nome = "Vue"

function imprimir(){

    console.log(nome)

}
```

Mesmo que `imprimir()` seja chamada de outro lugar.

Ela continuará enxergando `nome`.

Porque seu escopo foi definido quando foi criada.

---

# Escopo não depende da chamada

Veja.

```javascript
const linguagem = "Vue"

function a(){

    b()

}

function b(){

    console.log(linguagem)

}

a()
```

`b()` não procura variáveis dentro de `a()`.

Ela procura no local onde foi criada.

Esse detalhe é extremamente importante.

---

# O que é uma Closure?

Closure é uma função que consegue acessar variáveis do escopo onde foi criada, mesmo após esse escopo teoricamente ter terminado.

Essa definição costuma parecer abstrata.

Vamos visualizar.

---

# Primeiro exemplo

```javascript
function criarMensagem(){

    const mensagem = "Olá"

    function imprimir(){

        console.log(mensagem)

    }

    return imprimir

}

const fn = criarMensagem()

fn()
```

Resultado:

```text
Olá
```

Mas...

`criarMensagem()` já terminou.

Por que `mensagem` ainda existe?

Porque a Closure manteve uma referência.

---

# Visualizando

Antes do retorno.

```text
criarMensagem()

↓

mensagem = "Olá"

↓

imprimir()
```

Após retornar.

```text
fn()

↓

Closure

↓

mensagem = "Olá"
```

Mesmo que a função original tenha terminado.

A variável continua viva.

---

# A Closure copia a variável?

Não.

Ela mantém uma referência.

Isso significa que o valor pode mudar.

Observe.

```javascript
function contador(){

    let numero = 0

    return function(){

        numero++

        return numero

    }

}

const incrementar = contador()

console.log(incrementar())

console.log(incrementar())

console.log(incrementar())
```

Resultado.

```text
1

2

3
```

A variável nunca foi recriada.

Ela continua existindo dentro da Closure.

---

# Por que isso acontece?

Porque a Engine percebe que existe uma função utilizando aquela variável.

Ela não pode liberar a memória.

Caso contrário.

A função deixaria de funcionar.

---

# Relação com o Garbage Collector

No capítulo anterior vimos:

Objeto sem referências.

↓

Garbage Collector remove.

Agora.

Observe.

```javascript
function criar(){

    const usuario = {

        nome: "Vue"

    }

    return () => usuario

}
```

O objeto não pode ser removido.

A Closure continua apontando para ele.

---

# Closures e Memória

Closures são extremamente úteis.

Mas também podem causar Memory Leaks.

Imagine.

```javascript
function criar(){

    const listaGigante = new Array(1000000)

    return function(){

        console.log(listaGigante.length)

    }

}
```

Mesmo que você utilize apenas `length`.

Todo o array continua ocupando memória.

---

# Closures no Vue

Agora chegamos ao motivo deste capítulo.

Observe um Composable.

```javascript
import { ref } from "vue"

export function useCounter(){

    const contador = ref(0)

    function incrementar(){

        contador.value++

    }

    return {

        contador,

        incrementar

    }

}
```

Por que `incrementar()` consegue acessar `contador`?

Porque ela é uma Closure.

---

# setup()

O mesmo acontece dentro do `setup()`.

```javascript
setup(){

    const nome = ref("Vue")

    function alterar(){

        nome.value = "Vue 3"

    }

    return {

        nome,

        alterar

    }

}
```

Quando o botão chama:

```vue
@click="alterar"
```

A função continua enxergando `nome`.

Mesmo sendo executada muito tempo depois.

---

# Computed

Observe.

```javascript
const contador = ref(0)

const dobro = computed(() => {

    return contador.value * 2

})
```

A função passada para `computed` também é uma Closure.

Ela mantém acesso ao `contador`.

---

# Watch

O mesmo ocorre.

```javascript
watch(contador, () => {

    console.log(contador.value)

})
```

A callback captura variáveis externas.

---

# Event Listeners

```javascript
const nome = "Vue"

button.addEventListener("click", () => {

    console.log(nome)

})
```

O clique pode acontecer minutos depois.

Mesmo assim.

A variável continua acessível.

---

# Closures estão em todos os lugares

Você utiliza Closures praticamente o tempo todo.

Exemplos.

```javascript
map()

filter()

reduce()

setTimeout()

Promises

EventListeners

Watch

Computed

Composables

Setup

Pinia

Vue Router
```

Sem Closures.

Nada disso funcionaria da mesma forma.

---

# Armadilhas

## Capturar objetos enormes

```javascript
const lista = carregarUmMilhaoDeRegistros()

return () => lista
```

A lista permanecerá viva.

---

## Criar Closures desnecessárias

Em loops gigantes.

```javascript
for(let i = 0; i < 1000000; i++){

    botoes.push(() => i)

}
```

Isso cria um milhão de funções.

---

# Performance

Closures possuem custo.

Mas esse custo normalmente é pequeno.

O problema aparece quando:

* capturam objetos gigantes;
* permanecem vivas desnecessariamente;
* são criadas em excesso.

---

# Como o Vue utiliza Closures

Praticamente toda a API pública depende delas.

* `setup()`
* `computed()`
* `watch()`
* `watchEffect()`
* `provide()`
* `inject()`
* Composables
* Pinia
* Vue Router

Se você compreender Closures profundamente.

Grande parte da arquitetura do Vue ficará muito mais simples.

---

# Comparação

## Sem Closure

```javascript
let numero = 0

numero++
```

Variável global.

---

## Com Closure

```javascript
function criar(){

    let numero = 0

    return () => ++numero

}
```

Estado privado.

Muito mais seguro.

---

# Boas práticas

* Prefira Closures para encapsular estado.
* Evite capturar objetos gigantes.
* Libere listeners quando não forem mais necessários.
* Utilize Composables para compartilhar lógica.

---

# Resumo

Neste capítulo aprendemos que:

* Escopo define onde variáveis podem ser acessadas.
* O JavaScript utiliza Escopo Léxico.
* Closures mantêm referências ao escopo onde foram criadas.
* Elas são fundamentais para o Vue 3.
* Composables, `setup()`, `computed()` e `watch()` existem graças às Closures.

---

# Exercícios

### Exercício 1

Explique, com suas palavras, o que é uma Closure.

---

### Exercício 2

Por que o código abaixo continua funcionando?

```javascript
function criar(){

    const nome = "Vue"

    return () => console.log(nome)

}
```

---

### Exercício 3

Implemente uma função `criarContador()` que mantenha um estado privado utilizando Closures.

---

### Exercício 4

Explique por que `setup()` consegue acessar variáveis mesmo após a renderização inicial.

---

# Desafio

Implemente um Composable chamado `useTodoList()` utilizando apenas Closures e `ref()`.

Ele deve expor:

* lista
* adicionar()
* remover()
* limpar()

Explique quais variáveis permanecem encapsuladas e por quê.

---

# Projeto do capítulo

Crie uma pequena biblioteca de utilitários utilizando Closures.

Implemente:

* `useCounter`
* `useToggle`
* `useLocalStorage`
* `useDebounce`
* `useThrottle`

Não utilize Vue inicialmente.

Depois refatore todos para utilizar `ref()` e compare as diferenças.

---

# Checklist

* [ ] Entendi Escopo Léxico.
* [ ] Sei explicar Closures.
* [ ] Entendi Execution Context.
* [ ] Sei por que `setup()` funciona.
* [ ] Consigo explicar como Composables utilizam Closures.
* [ ] Estou preparado para estudar Proxy e Reflect.

---

# Próximo capítulo

## **Capítulo 07 — Proxy e Reflect: A Base da Reatividade do Vue 3**

Neste capítulo começaremos a estudar os internals do Vue. Você entenderá como `Proxy` intercepta operações em objetos, como `Reflect` funciona, por que o Vue abandonou `Object.defineProperty` e como esses conceitos tornam possível a reatividade do Vue 3.
