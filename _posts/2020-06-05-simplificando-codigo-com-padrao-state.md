---
layout: post
title:  Organizando o código com máquinas de estados
image: /images/2019/12/02/antes-depois-maquinas-de-estados.jpg
tags: código, padrões
---

É nosso dever, enquanto profissionais que fazem software, deixar nossos códigos em ordem. Às vezes, porém, sentimos que vai ficando cada vez mais difícil modificar determinadas partes do projeto.

## O problema

Com o tempo, fica cada vez mais difícil resolver bugs e acrescentar requisitos. Não poder excluir um pedido depois que ele já foi enviado, não poder excluir um usuário se ele já fez uma compra, verificar os estados de vários flags, colocar ifs com base nesses flags em várias partes do código, e por aí vai.

![Poste com muitos, muitos gatos](/images/2019/12/02/poste.jpg)
<p class="figcaption">O código, depois de algumas entregas...</p>

**De modo geral, os sintomas são estes**:

- Aparecem atributos remetendo a estados e datas. Exemplos: `data_criacao`, `data_alteracao`, `data_exclusao`, `criado`, `inativo`, `cancelado`.
- Existem variações importantes de comportamento associadas a datas e/ou flags. Se o estado é `criado`, algumas ações são possíveis, se `excluido`, as ações possíveis são outras, e assim por diante.
- Fica cada vez mais sofrido acrescentar novas ações e corrigir problemas conforme o tempo passa.

## Um Estudo de Caso

Imagine que você tem que fazer um site de reclamações sobre empresas, o Resmungue Aqui :)

Os requisitos básicos são os seguintes:

- Uma reclamação pode ser editada quantas vezes a pessoa quiser, até que decida submetê-la.
- Uma vez submetida, ela entra em estágio de avaliação.
- O site torna públicas apenas as reclamações aprovadas.
- Reclamações aprovadas podem ser respondidas por representantes de empresas.

Você implementa e o código está bonito, pequeno, tudo certo.
Depois de um tempo, chegam mais modificações:

- Reclamações em avaliação não podem ser editadas por ninguém.
- Reclamações em avaliação podem ser aprovadas ou reprovadas.

Mais um tempo se passa e você tem que implementar o seguinte:

- Reclamações reprovadas podem ser editadas para nova análise.
- Reclamações já aprovadas, se editadas, voltam à fase de análise.

Consegue imaginar a dificuldade crescente em lidar com esse código e todas as variações (atuais e futuras!) das reclamações do Resmungue Aqui?!

## Solução

Uma das maneiras mais eficientes de lidar com a complexidade é fazer desenhos.
Podemos então fazer um mapeamento entre os estados e as transições possíveis entre eles. Para isso construimos uma *máquina de estados*, que nada mais é do que um  *grafo direcionado*.

![Estados das reclamações](/images/2019/12/02/reclamacao-estados.png)

Os vértices são os estados e as arestas são as ações que levam de um estado a outro. Se uma reclamação é submetida, ela passa do estado de `Rascunho` para o estado de `Em avaliação` por meio da ação `submeter()`, que conecta os dois estados.

A partir desse modelo, fica bem mais fácil compreender quais ações são possíveis conforme o estado de uma reclamação. Ele nos ajuda a gerenciar a absorver melhor a complexidade, além de podermos usá-lo para conversar sobre os requisitos do projeto.

### Como implementar a máquina?

Agora, fica mais fácil compreender que cada estado (vértice do grafo) pode ser envelopado em uma classe específica. Pela sua concisão, todos os exemplos de código do texto serão dados em Kotlin:

```kotlin
class Rascunho: EstadoReclamacao

class EmAvaliacao: EstadoReclamacao

class Reprovado: EstadoReclamacao

class Aprovado: EstadoReclamacao
```

#### O pulo do gato

As ações na reclamação variam conforme o estado da reclamação. Portanto, cada estado trata a ação à sua maneira:

```kotlin
abstract class EstadoReclamacao {
    fun editar(reclamacao: Reclamacao)

    fun submeter(reclamacao: Reclamacao)

    fun aprovar(reclamacao: Reclamacao)

    fun reprovar(reclamacao: Reclamacao)  

    fun acaoNaoSuportada() = throw NotSupportedException()
}

class Rascunho: EstadoReclamacao {
    fun submeter(reclamacao: Reclamacao) {
        // seu código
    }

    fun editar() {
        // seu código
    }

    fun aprovar(reclamacao: Reclamacao) = acaoNaoSuportada()

    fun reprovar(reclamacao: Reclamacao) = acaoNaoSuportada()
}

class EmAvaliacao: EstadoReclamacao {
    fun submeter(reclamacao: Reclamacao) = acaoNaoSuportada()

    fun editar() = acaoNaoSuportada()

    fun aprovar(reclamacao: Reclamacao) {
        // seu código
    }

    fun reprovar(reclamacao: Reclamacao) {
        // seu código
    }
}

// etc
```

Por fim, escrevemos o código para que o objeto `reclamacao` *repasse* as ações `submeter()`, `editar()`, `aprovar()` e `reprovar()` ao seu estado corrente.

```kotlin
class Reclamacao(val conteudo: String, internal var estado: EstadoReclamacao) {
    fun editar() = estado.editar(this)

    fun submeter() = estado.submeter(this)

    fun aprovar() = estado.aprovar(this)

    fun reprovar() = estado.reprovar(this)
}
```

#### E as mudanças de estados?

Até agora, movemos com sucesso as responsabilidades que dependem de cada estado. Mas como _mudar_ de um estado a outro? O que fizemos até aqui foi _delegar_ o comportamento específico a cada estado.

Existem pelo menos três maneiras de fazer as transições:

- a) uma classe gerenciadora, que tenha a visão do todo e gerencie as comutações. Alguns frameworks, como o Spring, funcionam desta maneira. Por um lado, essa abordagem facilita no entendimento global de quais são as possibilidades daquela máquina de estados. Por outro, ela tende a favorecer classes inchadas, o que não gosto.
- b) cada estado indica o estado seguinte, conforme a ação que é disparada dentro dele. Prefiro esta abordagem. Com ela, os testes unitários ficam bem mais expressivos e elegantes.
- c) um comutador que receba tanto os estados quanto as ações como objetos. É a opção (a) com esteroides, e fica bem bacana.

Por brevidade, seguirei a abordagem (b). Nela, portanto, se o estado vigente é `EmAvaliacao` e ele processar a ação `aprovar()`, é o próprio código da classe `EmAvaliacao` que atualizará o estado da nossa `reclamacao` para `Aprovado`:

```kotlin
class EmAvaliacao: EstadoReclamacao {
    // ...

    fun aprovar(reclamacao: Reclamacao) {
        reclamacao.estado = Aprovado()
    }

    // ...
}
```

**Falar é de graça**
Você pode conferir o [código-fonte no Github](https://github.com/edsoncunha/tutorials/tree/master/pt-br/maquina-de-estados/kotlin).

O que achou? Se quiser discutir implementações específicas, envolvendo linguagens e frameworks de mercado, deixe um comentário ou mande uma mensagem.
