# Capítulo 11 — `ref()`: Implementando o Primeiro Primitive Reativo do Vue

> **Objetivo:** implementar o `ref()` do zero e compreender exatamente como o Vue torna valores primitivos reativos. Ao final deste capítulo você entenderá por que existe `.value`, como funciona a classe `RefImpl` e como `track()` e `trigger()` trabalham juntos para tornar números, strings e booleans totalmente reativos.

---

# Objetivos

Ao concluir este capítulo você será capaz de:

* Explicar por que `ref()` existe.
* Entender por que primitivas precisam de um wrapper.
* Implementar um `ref()` simplificado.
* Entender a classe `RefImpl`.
* Compreender `.value`.
* Explicar a diferença entre `ref()` e `reactive()`.

---

# Pré-requisitos

* Capítulos 01 ao 10.

---

# O problema

Até agora implementamos:

```javascript
const usuario = reactive({

    nome: "Felipe"

})
```

Funciona perfeitamente.

Mas...

Como tornar isto reativo?

```javascript
let contador = 0
```

Não podemos fazer:

```javascript
new Proxy(0,{})
```

Resultado.

```text
TypeError

Cannot create proxy with a non-object
```

O Proxy funciona apenas com objetos.

---

# A solução do Vue

O Vue cria um objeto.

Dentro dele.

Guarda o valor primitivo.

Imagine.

```javascript
ref(10)
```

Na verdade.

O Vue transforma isso em algo parecido com:

```javascript
{

    value:10

}
```

Agora.

Esse objeto pode ser utilizado pelo Proxy.

Ou até implementar manualmente `get()` e `set()`.

---

# Por que `.value`?

Muitos iniciantes perguntam.

> Por que não usar simplesmente o número?

Porque.

```javascript
10
```

Não pode ser interceptado.

Já isto.

```javascript
{

    value:10

}
```

Pode.

---

# Nossa primeira implementação

```javascript
function ref(valor){

    return {

        value: valor

    }

}
```

Agora.

```javascript
const contador = ref(0)
```

Resultado.

```javascript
contador.value
```

Até aqui.

Ainda não existe reatividade.

---

# Tornando reativo

Precisamos interceptar:

Leitura.

```javascript
contador.value
```

E escrita.

```javascript
contador.value++
```

---

# Utilizando getters

```javascript
function ref(valor){

    return {

        get value(){

            return valor

        },

        set value(novo){

            valor = novo

        }

    }

}
```

Agora conseguimos controlar os acessos.

---

# Integrando ao track()

Quando alguém faz.

```javascript
contador.value
```

Precisamos registrar dependências.

```javascript
get value(){

    track(...)

    return valor

}
```

---

# Integrando ao trigger()

Quando alguém altera.

```javascript
contador.value = 20
```

Executamos.

```javascript
set value(novo){

    valor = novo

    trigger(...)

}
```

Agora temos reatividade.

---

# Testando

```javascript
const contador = ref(0)

effect(()=>{

    console.log(

        contador.value

    )

})
```

Resultado.

```text
0
```

Depois.

```javascript
contador.value++
```

Resultado.

```text
1
```

Sem chamarmos nada manualmente.

---

# Um detalhe importante

Observe.

```javascript
contador.value = 10

contador.value = 10
```

Nosso código executaria.

```text
trigger()

trigger()
```

Mesmo que nada tenha mudado.

---

# Como o Vue resolve?

Antes de atualizar.

Ele compara.

```javascript
if(

    valor !== novo

){

    valor = novo

    trigger()

}
```

Assim.

Atualizações desnecessárias são evitadas.

---

# Comparando corretamente

Na implementação oficial.

O Vue utiliza.

```javascript
Object.is()
```

Em vez de.

```javascript
===
```

Por quê?

Porque.

```javascript
NaN === NaN
```

É.

```text
false
```

Enquanto.

```javascript
Object.is(

NaN,

NaN

)
```

Retorna.

```text
true
```

Também trata corretamente:

```javascript
+0

-0
```

---

# Criando RefImpl

O Vue utiliza uma classe.

```javascript
class RefImpl{

}
```

Vamos simplificar.

```javascript
class RefImpl{

    constructor(valor){

        this._value = valor

    }

}
```

---

# Getter

```javascript
get value(){

    trackRefValue(this)

    return this._value

}
```

---

# Setter

```javascript
set value(novo){

    if(

        Object.is(

            novo,

            this._value

        )

    ){

        return

    }

    this._value = novo

    triggerRefValue(this)

}
```

Perceba.

Agora não chamamos mais `track()` diretamente.

---

# Por quê?

Porque.

Um `ref` não utiliza:

```text
WeakMap

↓

Objeto

↓

Map

↓

Propriedade
```

Existe apenas.

```text
Ref

↓

Set

↓

Effects
```

É muito mais simples.

---

# Estrutura interna

Visualmente.

```text
Ref

↓

dep

↓

Set

↓

Effect
```

Cada Ref possui seu próprio Set.

---

# Implementando

```javascript
class RefImpl{

    constructor(valor){

        this.dep =

            new Set()

    }

}
```

---

# trackRefValue()

```javascript
function trackRefValue(ref){

    if(

        activeEffect

    ){

        ref.dep.add(

            activeEffect

        )

    }

}
```

---

# triggerRefValue()

```javascript
function triggerRefValue(ref){

    ref.dep.forEach(effect=>{

        effect.run()

    })

}
```

Perceba.

Muito mais simples.

---

# Fluxo completo

```text
contador.value

↓

getter

↓

trackRefValue()

↓

dep

↓

Set

↓

contador.value++

↓

setter

↓

triggerRefValue()

↓

Effects
```

---

# ref() agora

Nossa função.

```javascript
function ref(valor){

    return new RefImpl(valor)

}
```

Muito parecida com a implementação oficial.

---

# Por que o Vue utiliza `_value`?

Porque.

O usuário acessa.

```javascript
contador.value
```

Internamente.

O Vue armazena.

```javascript
contador._value
```

Separando.

API pública.

↓

Implementação interna.

---

# Objetos dentro de ref()

Observe.

```javascript
const usuario = ref({

    nome:"Felipe"

})
```

Pergunta.

O objeto interno será reativo?

Sim.

O Vue faz.

```javascript
reactive(obj)
```

Automaticamente.

---

# Simplificando

Internamente.

```javascript
if(

typeof valor

===

"object"

){

    reactive(valor)

}
```

Assim.

```javascript
usuario.value.nome
```

Também será reativo.

---

# shallowRef()

Existe outra API.

```javascript
shallowRef()
```

Diferença.

```javascript
ref({

})
```

Transforma o objeto interno em reativo.

Enquanto.

```javascript
shallowRef({

})
```

Mantém o objeto exatamente como foi recebido.

Apenas o `.value` é observado.

---

# Comparação

## ref

```javascript
const usuario = ref({

nome:"Vue"

})
```

Mudança.

```javascript
usuario.value.nome = "React"
```

É detectada.

---

## shallowRef

```javascript
const usuario = shallowRef({

nome:"Vue"

})
```

Agora.

```javascript
usuario.value.nome = "React"
```

Não dispara atualização.

Somente.

```javascript
usuario.value = {}
```

---

# isRef()

O Vue também possui.

```javascript
isRef(valor)
```

Internamente.

Cada Ref possui uma marca.

```javascript
__v_isRef
```

Assim.

```javascript
isRef(contador)
```

Retorna.

```text
true
```

---

# unref()

Outra função útil.

```javascript
unref(contador)
```

Equivale a.

```javascript
contador.value
```

Se não for Ref.

Retorna o próprio valor.

---

# toValue()

Nas versões recentes do Vue existe.

```javascript
toValue()
```

Ela resolve automaticamente:

* Ref
* Getter
* Valor comum

Muito utilizada em composables.

---

# Relação com Template

Pergunta.

Por que no template escrevemos.

```vue
{{ contador }}
```

E não.

```vue
{{ contador.value }}
```

Porque o compilador faz isso automaticamente.

Mais adiante estudaremos esse processo.

---

# Performance

Cada Ref possui seu próprio conjunto de dependências.

Isso torna a atualização extremamente eficiente.

Apenas quem depende daquele Ref será executado.

---

# Comparando ref e reactive

| ref                          | reactive            |
| ---------------------------- | ------------------- |
| Primitivos                   | Objetos             |
| Possui `.value`              | Não                 |
| Um único Set de dependências | WeakMap → Map → Set |
| Wrapper                      | Proxy               |

---

# Resumo

Neste capítulo aprendemos que:

* `Proxy` não funciona com primitivas.
* `ref()` cria um wrapper.
* `.value` existe para permitir interceptação.
* Cada Ref possui seu próprio conjunto de dependências.
* O Vue implementa `RefImpl`.
* `trackRefValue()` e `triggerRefValue()` são versões especializadas de `track()` e `trigger()`.

---

# Exercícios

## Exercício 1

Implemente uma versão simplificada de `ref()` utilizando getters e setters.

---

## Exercício 2

Adicione `trackRefValue()` e `triggerRefValue()` à sua implementação.

---

## Exercício 3

Explique por que `Proxy` não pode ser utilizado diretamente com números.

---

## Exercício 4

Implemente `isRef()` e `unref()`.

---

# Desafio

Evolua sua biblioteca **MiniVue Reactive** para suportar:

* `ref()`;
* `RefImpl`;
* `trackRefValue()`;
* `triggerRefValue()`;
* `Object.is()`;
* `isRef()`;
* `unref()`.

---

# Projeto do capítulo

Implemente completamente o sistema de `ref()` da sua biblioteca.

Ao final ela deverá suportar:

* valores primitivos;
* objetos (convertidos automaticamente para `reactive()`);
* `shallowRef()`;
* `isRef()`;
* `unref()`;
* comparação utilizando `Object.is()`.

---

# Checklist

* [ ] Entendi por que `ref()` existe.
* [ ] Sei explicar a função da propriedade `.value`.
* [ ] Consigo implementar um `RefImpl`.
* [ ] Entendi `trackRefValue()` e `triggerRefValue()`.
* [ ] Sei a diferença entre `ref()` e `reactive()`.
* [ ] Minha biblioteca já suporta valores primitivos reativos.

---

# Próximo capítulo

## **Capítulo 12 — `reactive()`: Implementando Objetos Reativos Completos**

Agora construiremos uma implementação muito próxima da utilizada pelo Vue para `reactive()`. Você aprenderá sobre **Proxy Cache**, `WeakMap`, Proxies profundos (*Deep Reactive*), `readonly()`, `shallowReactive()`, `markRaw()`, `toRaw()` e os mecanismos utilizados pelo Vue para garantir identidade, desempenho e evitar a criação de múltiplos Proxies para o mesmo objeto. Esse capítulo marca a transição da nossa biblioteca para uma implementação extremamente próxima do runtime oficial do Vue 3.
