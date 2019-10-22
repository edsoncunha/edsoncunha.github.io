---
layout: post
title:  Dois truques mágicos para manter projetos de software saudáveis
image: https://m.media-amazon.com/images/M/MV5BNTEyNzk4MTYtMjFkMC00YjZlLWFhMjItNTRmMzc1OGMyMzMyXkEyXkFqcGdeQXVyNjMxNDE2ODU@._V1_.jpg
tags: engenharia de software, testes
---

Com apenas duas restrições automatizáveis, podemos conseguir desdobramentos muito interessantes para nos ajudar a manter a saúde de uma base de código.

> A arte de programar consiste em organizar e dominar a complexidade
> 
> (Edsger Dijkstra)

# 1) Não deixe a cobertura de testes cair

O ponto-chave é não deixar o projeto deteriorar. Se ele está no começo, esta é uma chance de ouro de começar com mais rigor e vê-lo resultar em um projeto mais estável, legível e bem testado. Tão ou mais importante do que ter uma boa cobertura percentual de testes é impedir que esse número recue. Não tenha medo de exigir 100% de cobertura de testes no código que está para ser incorporado na master ou em uma branch. Se essa exigência for colocada desde o pontapé inicial do projeto, veja que maravilha: você terá 100% de cobertura de testes unitários 

![A pirâmide de testes de Mike Cohn](https://martinfowler.com/articles/practical-test-pyramid/testPyramid.png)

<p class="figcaption">A pirâmide de testes, de Mike Cohn. A base, que queremos alargar, é composta pelos testes unitários. 
<br />Crédito da imagem: <a href="https://martinfowler.com/articles/practical-test-pyramid.html">Martin Fowler</a></p>

- Por quê?
    - Aumentar a cobertura de código quando a base estiver grande é mais penoso
    - Testes não nos livram integralmente de problemas, mas a ausência deles é muito pior
    - Mesmo que o time não pratique os testes primeiro (TDD), escrever um grande volume de testes depois de fazer o código é muito sofrido. O build, muito provavelmente, vai quebrar. Isso condiciona as pessoas a fazer commits menores e, por conta disso, dividir as tarefas em partes menores. 
    - Tarefas mais atômicas proporcionam uma melhor visibilidade do andamento do projeto e oportunidades de paralelismo entre os membros do time
    - Times muito paralelos tendem a perceber mais rapidamente os incômodos do código, como classes que comecem a fazer mais coisas do que deveriam. Muitas pessoas mexendo em uma mesma classe normalmente indica que ela tem muitas dependências e/ou responsabilidades, o que torna a evolução mais difícil e os conflitos de merge mais frequentes
    
- Como?
    - Faça o Jenkins construir o projeto a cada commit
    - Gere relatórios de teste xUnit
    - Use um plugin como o [Cobertura](https://github.com/jenkinsci/cobertura-plugin), que falha o build caso as métricas estejam abaixo do limite. O pulo do gato com esse plugin é permitir que ele atualize o limite a cada build: se a cobertura cresceu de 40% para 44%, então essa passa a ser a cobertura mínima para o que o build passe.
    - Se você usa Sonar, terá o mesmo resultado ao exigir 100% de cobertura no código que está entrando. Nos _quality gates_, procure as métricas que contém "new code" na descrição e associe-as à análise do seu projeto.

Os resultados que vi foram bem interessantes: 
- Códigos desnecessários não são cobertos pelos testes, o que faz build quebrar. As pessoas pararam de projetar canhões para matar moscas.
- As tarefas passaram a ser mais subdivididas
- O planejamento e a organização do time ficaram mais apurados
- As entregas ficaram distribuídas mais homogeneamente ao longo do tempo

# 2) Mantenha as classes com uma única responsabilidade

![Foto em que se vê um enorme kaiju atrás de prédios de três andares. O monstro é três vezes mais alto do que os prédios](https://img1.looper.com/img/gallery/the-best-kaiju-movies-youve-never-seen/intro-1516897231.jpg "Vai um suco de kaiju?")

<p class="figcaption">Uma classe <em>kaiju</em>, aquela que todo mundo tem medo de mexer</p>

<!-- Essa prática é o S do SOLID. Falo dela separadamente porque tenho um pequeno probleminha: não gosto de decoreba. Existem princípios fundamentais na programação que têm seus vários desdobramentos rebatizados.
Entender as questões fundamentais, as raízes, é mais efetivo do que decorar. -->

O que desejamos é escrever código que pareça óbvio. Se alguém vir seu trabalho e não demorar a falar um "ah, beleza", pode ficar feliz, você está no caminho certo. 

Em programação dinâmica, existe um conceito chamado _Princípio da Otimalidade_ ([Richard Bellman, 1957](https://en.wikipedia.org/wiki/Bellman_equation#Bellman's_Principle_of_Optimality)):

> Para um dado estado do sistema, a política ótima para os estados remanescentes é independente da política de decisão adotada em estados anteriores.

 Em outras palavras, uma solução é a melhor possível - a solução _ótima_ - se cada uma de suas subdivisões é também a melhor possível. Por exemplo: um pacote TCP chegará a seu destino pela melhor rota se, a cada passo, ele escolher o trecho mais rápido possível entre os roteadores que ele "enxerga" a partir daquele ponto.

Aplicando o conceito ao nossos projetos, o que temos é que ele será bacana, enxuto e fácil de compreender se cada uma de suas partes também o forem. Se não abrirmos mão de construir classes enxutas e coesas, assim também serão os agrupamentos de mais alto nível, como pacotes, módulos, bibliotecas etc.


- Outras motivações:
    - Faz parte do nosso trabalho dividir problemas grandes em partes menores. Dividir uma classe _kaiju_ em vários calangos especialistas segue o mesmo raciocínio.
    - Testar classes grandes dá um trabalho enorme, sobretudo se você escrever os testes depois do código real. 
    - Quando as classes se relacionam com muitas outras e/ou seus métodos têm muitos parâmetros, temos aí uma explosão combinatória de entradas, saídas e estados. Isso torna o código muito mais vulnerável a bugs, já que é difícil mapear todas as possibilidades.
    - Compaixão com seu eu daqui a 6 meses, quando você nem se lembrar mais o que esse código faz.

- Como conseguir?
    - Veja o código de fora para dentro, pensando NO QUE ele faz, e não COMO, e responda a pergunta: o que essa classe deveria fazer?
    - Se sua resposta tiver as palavras E ou OU, é porque existe mais de uma responsabilidade. Descreva essa responsabilidade por meio de uma classe (testes e nomes explicativos são ótimas maneiras de comunicar a intenção de uma classe).
    - Use o Sonar para analisar o código. Ele consegue entender se sua classe está virando um canivete suíço por meio da quantidade de linhas e classes com as quais ela se relaciona.
    - Amarre o Sonar com o Jenkins. Isto é, dispare a análise do Sonar no build e configure-o para falhar caso o _quality gate_ não tenha sido atendido.
    - O Sonar dá a possibilidade de analisar o código que está _entrando_ separadamente do código antigo. Além de ser super conveniente para quem trabalha com _branches_ e _pull requests_, essa medida não paralisa o time caso as análises sejam ativadas em uma base de código já existente.

## Conclusão

As práticas citadas neste texto só funcionam se forem abraçadas pelas pessoas e cercadas pelas ferramentas. É preciso que comprem as ideias para que as ferramentas não sejam abandonadas. Estas, por sua vez, resolvem um problema humano: nós eventualmente nos distraímos.
    

### Para ler mais
- [Livro: Refatoração para padrões, de Joshua Kerievsky](https://amzn.to/2obFz9C)
- [Livro: A arte do código legível, de Boswell e Foucher](https://amzn.to/2oTeiZY)
- [Construindo uma esteira de entrega contínua com Jenkins](https://dzone.com/articles/building-a-continuous-delivery-pipeline-using-jenk)
- [SonarScanner para Jenkins](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-jenkins/)


Bônus:

<iframe width="560" height="315" src="https://www.youtube.com/embed/0p_1QSUsbsM" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

