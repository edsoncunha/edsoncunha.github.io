---
layout: post
title:  Desbagunçando o código com o padrão State
image: https://live.staticflickr.com/7209/6960939955_df60b8523b_c.jpg
tags: código, padrões
---

No começo, o projeto caminha bem. Com o tempo, vão surgindo situações que temos que contornar: pode fazer tal ação quando o objeto tá na situação X, não pode quando tá na situação Y. Seguimos em frente, até que pisca o alerta do TFM (**t**á **f**oda de **m**exer).

![Poste com muitos, muitos gatos](https://i.pinimg.com/236x/bf/15/49/bf1549764d0235980afb18542db8459a.jpg)
<p class="figcaption">Mais um dia normal debugando código</p>

**Sintomas:**

- Aparecem atributos remetendo a status e datas. Exemplos: `data_criacao`, `data_alteracao`, `data_exclusao`, `criado`, `inativo`, `cancelado`.
- O comportamento da entidade é bastante sensível a esses atributos.
- Fica cada vez mais difícil dar ao objeto o comportamento desejado, que tipicamente varia de acordo com seu status.

## Exemplo

Imagine que você tem que fazer um site de opiniões sobre produtos.

Os requisitos básicos são os seguintes:

1. Uma opinião pode ser editada quantas vezes a pessoa quiser, até que decida submetê-la.
2. Uma vez submetida, ela entra em estágio de avaliação.
3. Opiniões em avaliação não podem ser editadas por ninguém.
4. Opiniões em avaliação podem ser aprovadas ou reprovadas.
5. Opiniões reprovadas podem ser editadas para nova análise.
6. Opiniões já aprovadas, se editadas, voltam à fase de análise.
7. O site torna públicas apenas as opiniões aprovadas.

As mudanças de status a partir das ações ficariam assim, portanto:

![Estados das opiniões](/images/2019/12/02/reclamacao-estados.png)

**Como seria o código?**

Façamos um trecho em Kotlin:

<script src="https://gist.github.com/edsoncunha/d9949df8f8b5ab78ee3376727a0fd1cd.js"></script>

Com apenas duas funções implementadas, o código já é um pouco difícil de ler. Em cada ação (função), primeiro temos que checar se ela é válida, a partir do status atual da opinião. Se a ação for válida, executamos o código que quisermos, para em seguida mudar o status da opinião.

Não é difícil imaginar que depois de algum tempo e várias indas e vindas de requisitos e pessoas no projeto, essa classe vai virar um pesadelo.

Nas entrelinhas, é como se cada status determinasse um subtipo de `Opiniao`, cada qual com seu comportamento. Poderíamos resolver o problema por herança, então? Não, porque a `Opiniao` não pode modificar sua subclasse em tempo de execução em linguagens como Kotlin, Java e C#. Como desfazer esse nó?

## O padrão State

Pela definição dos autores:

> Permite a um objeto alterar seu comportamento quando seu estado interno muda. O objeto parecerá ter mudado sua classe.

É exatamente o que precisamos para que a `Opiniao` tenha um comportamento específico para cada status.

Como não dá para que uma classe troque seu próprio subtipo em tempo de execução, o truque é fazer *parecer*, ao observador externo, que o subtipo mudou.

Fazemos isso por composição. Ou seja, construímos uma subclasse *para cada estado* da `Opiniao` e delegamos a execução das ações que queremos para ela.

A seguir, veremos como fazer isso.


### A implementação canônica

Passos:

1. Definir um tipo abstrato para o estado. Vamos chamá-lo de `OpiniaoState`.
2. Definir, nesse tipo ancestral, as ações cujas consequências variam conforme cada estado: `editar()`, `submeter()`, `aprovar()` e `reprovar()`.
3. Criar uma subclasse para cada estado. No nosso exemplo, elas são `RascunhoState`, `EmAvaliacaoState`, `ReprovadoState`, `AprovadoState`.
4. Guardar uma instância de `OpiniaoState` em uma variável da `Opiniao`.
5. Delegar a essa instância de `OpiniaoState` a execução dos métodos `Opiniao.editar()', 'Opiniao.submeter()`, e assim por diante.

Código:

```
interface EstadoOpiniao {
    fun editar(opiniao: Opiniao)

    fun submeter(opiniao: Opiniao)

    fun aprovar(opiniao: Opiniao)

    fun reprovar(opiniao: Opiniao)
}
```


```
class Opiniao(val conteudo: String, internal var estado: EstadoOpiniao) {
    fun editar() = estado.editar(this)

    fun submeter() = estado.submeter(this)

    fun aprovar() = estado.aprovar(this)

    fun reprovar() = estado.reprovar(this)
}
```

```
class Aprovado : EstadoOpiniao {
    override fun editar(opiniao: Opiniao) {
        opiniao.estado = EmAvaliacao()
    }
}
```

**Vantagens**

* Curta
* Direta

**Desvantagens**
* Rebuscada se o cenário for simples
* Limitada se a quantidade de ações for muito grande

## Implementação com transição de comportamento na classe ancestral

### Vantagens
Te dá o histórico de brinde
Deixa as transições de estado mais explícitas
### Desvantagens
Viola a separação entre comando e consulta.

(Ressaltar que é antes de sair de um estado e ir para outro que está o comportamento específico. Portanto, em muitos casos, colocar o comportamento no OnEnter não vai resolver. Só funcionaria se você colocasse na assinatura da função o estado anterior, mas aí você tomaria decisões com base em estados, o que é justamente o que a máquina de estados está tentando resolver)

## Versão com muitas ações, usando classes

### Vantagens
Boa quando há muitas ações
Elegância e praticidade para enviar ações com parâmetros

### Desvantagens
Ofusca a complexidade do grafo (é preciso ficar de olho)
Implica em switches que usam instanceof, o que pode ser amenizado por facilidades da linguagem utilizada. Com o smart cast do Kotlin, é moleza.

## Implementações do mundo real com Spring

### State Machine do spring

### Vantagens
Injeção de dependência de maneira transparente
Boa legibilidade na configuração das transições

### Desvantagens
Ruim de testar
Legibilidade ruim para grafos grandes

## Versão final, persistente

### Vantagens
Testes unitários fáceis e legíveis
Boa legibilidade das classes de cada estado
Persistência transparente com um valor de enum para cada State

### Desvantagens
Não fica encapsulado na entidade
Busca de estado tem que ser por fora, para evitar referência circular na injeção de dependência
Lazy gera efeitos colaterais dentro dos instanceof das actions


-> código do projeto completo
 

