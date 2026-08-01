# Capítulo 07 — Proxy e Reflect: A Base da Reatividade do Vue 3

> **Objetivo:** compreender profundamente como `Proxy` e `Reflect` funcionam, por que foram adicionados ao JavaScript e como eles tornaram possível o sistema de reatividade do Vue 3.

---

# Objetivos

Ao final deste capítulo você será capaz de:

* Explicar o que é um Proxy.
* Entender todos os principais traps do Proxy.
* Compreender o papel do Reflect.
* Entender por que o Vue 2 utilizava `Object.defineProperty()`.
* Explicar por que o Vue 3 migrou para `Proxy`.
* Criar pequenos sistemas reativos utilizando apenas JavaScript.

---

# Pré-requisitos

* Capítulos 01 ao 06.

---

# Introdução

Até agora entendemos:

* Como o navegador funciona.
* Como o JavaScript executa código.
* Como o Event Loop agenda tarefas.
* Como Closures mantêm estado.

Mas ainda falta responder uma pergunta:

> **Como o Vue percebe que uma variável mudou?**

Observe:

```javascript
const contador = ref(0)

contador.value++
```

Em nenhum momento avisamos ao Vue que o valor mudou.

Mesmo assim:

* a tela atualiza;
* os componentes renderizam novamente;
* os computed são recalculados;
* os watchers executam.

Como?

A resposta começa com **Proxy**.

---

# Antes do Vue 3

O Vue 2 utilizava:

```javascript
Object.defineProperty()
```

Exemplo:

```javascript
const obj = {}

Object.defineProperty(obj, "nome", {

    get(){

    },

    set(valor){

    }

})
```

Sempre que alguém fazia:

```javascript
obj.nome = "Felipe"
```

O método `set()` era chamado.

Assim o Vue descobria a alteração.

---

# O problema do Vue 2

Essa solução possuía várias limitações.

Por exemplo:

Adicionar propriedades.

```javascript
usuario.idade = 24
```

O Vue 2 não detectava automaticamente.

Também havia problemas com:

* Arrays
* delete
* Map
* Set

Além de um custo elevado para tornar cada propriedade reativa.

---

# Surge o Proxy

Em 2015 (ES6).

O JavaScript ganhou:

```javascript
Proxy
```

O Proxy permite interceptar praticamente qualquer operação realizada sobre um objeto.

Imagine um segurança.

Antes de alguém entrar em uma sala.

Ele verifica.

"Quem é?"

"Pode entrar?"

O Proxy faz exatamente isso.

Toda operação passa por ele.

---

# Criando o primeiro Proxy

```javascript
const usuario = {

    nome: "Felipe"

}

const proxy = new Proxy(usuario, {})
```

Até aqui.

Nada mudou.

O objeto continua funcionando normalmente.

---

# Interceptando leituras

Podemos interceptar qualquer acesso.

```javascript
const proxy = new Proxy(usuario, {

    get(target, property){

        console.log(property)

        return target[property]

    }

})
```

Agora.

```javascript
console.log(proxy.nome)
```

Resultado.

```text
nome

Felipe
```

O Proxy percebeu a leitura.

---

# Entendendo os parâmetros

```javascript
get(target, property, receiver)
```

## target

Objeto original.

```javascript
usuario
```

---

## property

Qual propriedade foi acessada.

```javascript
nome
```

---

## receiver

Quem iniciou a operação.

Em aplicações comuns.

Normalmente é o próprio Proxy.

---

# Interceptando escrita

Agora.

```javascript
const proxy = new Proxy(usuario, {

    set(target, property, value){

        console.log(property)

        console.log(value)

        target[property] = value

        return true

    }

})
```

Quando fazemos.

```javascript
proxy.nome = "Vue"
```

Resultado.

```text
nome

Vue
```

O Proxy descobriu imediatamente.

---

# O return true

Todo trap `set()` deve retornar:

```javascript
true
```

Caso contrário.

A Engine entende que a operação falhou.

---

# O Proxy não altera o objeto

O objeto continua existindo.

```javascript
usuario
```

O Proxy apenas fica na frente dele.

Imagine.

```text
Usuário

↓

Proxy

↓

Objeto
```

Toda operação passa pelo Proxy.

---

# Todos os traps

O Proxy possui dezenas de interceptadores.

Os mais utilizados.

```javascript
get()

set()

deleteProperty()

has()

ownKeys()

apply()

construct()

defineProperty()

getPrototypeOf()

setPrototypeOf()
```

O Vue utiliza vários deles.

---

# Trap get()

Executa quando alguém faz.

```javascript
obj.nome
```

---

# Trap set()

Executa quando alguém faz.

```javascript
obj.nome = "Vue"
```

---

# Trap deleteProperty()

Executa quando alguém faz.

```javascript
delete obj.nome
```

---

# Trap has()

Executa quando alguém faz.

```javascript
"nome" in obj
```

---

# Trap ownKeys()

Executa quando fazemos.

```javascript
Object.keys(obj)
```

Ou.

```javascript
for...in
```

---

# Trap apply()

Intercepta chamadas de função.

```javascript
funcao()
```

---

# Trap construct()

Intercepta.

```javascript
new Classe()
```

---

# O Reflect

Agora surge outra dúvida.

Observe.

```javascript
get(target, property){

    return target[property]

}
```

Funciona.

Mas.

Existe uma forma melhor.

---

# O Reflect

Reflect é uma API criada junto com o Proxy.

Ela permite executar o comportamento padrão do JavaScript.

Exemplo.

```javascript
Reflect.get(target, property)
```

É equivalente.

```javascript
target[property]
```

Mas muito mais seguro.

---

# Reescrevendo

```javascript
get(target, property){

    console.log(property)

    return Reflect.get(target, property)

}
```

Esse é exatamente o padrão utilizado pelo Vue.

---

# Reflect.set()

```javascript
set(target, property, value){

    return Reflect.set(

        target,

        property,

        value

    )

}
```

---

# Por que usar Reflect?

Porque ele respeita:

* Herança
* Getters
* Setters
* Prototype Chain
* Receiver

Em vez de implementar toda essa lógica manualmente.

---

# Criando um sistema reativo simples

Agora vamos construir algo parecido com o Vue.

```javascript
const state = {

    contador: 0

}
```

Criando o Proxy.

```javascript
const reactive = new Proxy(state, {

    get(target, property){

        console.log("Lendo")

        return Reflect.get(target, property)

    },

    set(target, property, value){

        console.log("Atualizando")

        Reflect.set(target, property, value)

        return true

    }

})
```

Agora.

```javascript
reactive.contador
```

Resultado.

```text
Lendo
```

Depois.

```javascript
reactive.contador++
```

Resultado.

```text
Lendo

Atualizando
```

Perceba.

O Proxy descobriu exatamente o momento da leitura e da escrita.

É justamente isso que o Vue precisa.

---

# Mas ainda falta algo...

Imagine.

```javascript
console.log(state.contador)
```

O Proxy não será chamado.

Porque acessamos o objeto original.

Somente isto ativa o Proxy.

```javascript
reactive.contador
```

Por isso o Vue sempre devolve o Proxy.

Nunca o objeto original.

---

# Como o Vue utiliza Proxy?

De forma extremamente simplificada.

```javascript
function reactive(obj){

    return new Proxy(obj,{

        get(){

            // track()

        },

        set(){

            // trigger()

        }

    })

}
```

Essas duas funções.

```text
track()

trigger()
```

São o coração da reatividade.

Nos próximos capítulos construiremos ambas.

---

# Por que o Proxy foi revolucionário?

Ele resolveu praticamente todos os problemas do Vue 2.

Agora o framework consegue detectar automaticamente:

* novas propriedades;
* remoção de propriedades;
* arrays;
* índices;
* comprimento do array;
* objetos aninhados;
* Maps;
* Sets.

Tudo utilizando a mesma API.

---

# Comparação

## Vue 2

```text
Object.defineProperty
```

Problemas:

* adicionar propriedades;
* remover propriedades;
* arrays;
* desempenho.

---

## Vue 3

```text
Proxy
```

Vantagens.

* API completa.
* Muito mais flexível.
* Melhor desempenho.
* Mais simples de manter.

---

# Armadilhas

## Proxy não é o objeto

```javascript
proxy !== objeto
```

Sempre.

---

## Proxy não funciona em primitivas

Isto gera erro.

```javascript
new Proxy(10,{})
```

Somente objetos podem ser utilizados.

---

## Proxy não intercepta referências antigas

```javascript
const original = {}

const proxy = new Proxy(original,{})
```

Se alguém continuar utilizando:

```javascript
original
```

Nada será interceptado.

---

# Performance

Criar um Proxy possui custo.

Mas.

Depois de criado.

As operações são extremamente rápidas.

O Vue ainda utiliza cache interno para evitar criar múltiplos Proxies para o mesmo objeto.

Mais adiante veremos como isso funciona utilizando `WeakMap`.

---

# Resumo

Neste capítulo aprendemos que:

* Proxy intercepta operações em objetos.
* Reflect executa o comportamento padrão do JavaScript.
* O Vue 3 utiliza Proxy como base da reatividade.
* Leituras serão utilizadas para registrar dependências.
* Escritas serão utilizadas para disparar atualizações.

---

# Exercícios

## Exercício 1

Crie um Proxy que registre todas as leituras de propriedades.

---

## Exercício 2

Crie um Proxy que impeça valores negativos.

Exemplo:

```javascript
produto.estoque = -10
```

Deve lançar um erro.

---

## Exercício 3

Implemente um Proxy que registre todas as propriedades removidas utilizando `deleteProperty`.

---

## Exercício 4

Pesquise todos os traps existentes na especificação ECMAScript e explique a finalidade de cada um.

---

# Desafio

Implemente uma função:

```javascript
function reactive(obj)
```

Que:

* registre leituras;
* registre escritas;
* utilize Reflect;
* funcione com objetos aninhados.

Ainda não utilize `track()` nem `trigger()`.

---

# Projeto do capítulo

Desenvolva uma pequena biblioteca chamada **Mini Reactive**.

Nesta primeira versão ela deverá:

* criar Proxies;
* interceptar leituras;
* interceptar escritas;
* registrar logs das operações;
* suportar objetos aninhados.

Essa biblioteca evoluirá nos próximos capítulos até se tornar uma implementação simplificada do sistema reativo do Vue.

---

# Checklist

* [ ] Sei explicar o que é Proxy.
* [ ] Entendi o papel do Reflect.
* [ ] Sei diferenciar Vue 2 e Vue 3 quanto à reatividade.
* [ ] Consigo criar um Proxy personalizado.
* [ ] Entendi por que o Vue utiliza `get()` e `set()`.

---

# Próximo capítulo

## **Capítulo 08 — Construindo o Sistema de Reatividade do Zero (Parte 1): `track()` e `trigger()`**

Neste capítulo começaremos a implementar o núcleo do sistema reativo do Vue. Você entenderá como o framework registra dependências, identifica quais efeitos precisam ser atualizados e como nasce o mecanismo que alimenta `ref`, `reactive`, `computed` e `watch`. Esse será o início da construção do nosso próprio runtime reativo.
