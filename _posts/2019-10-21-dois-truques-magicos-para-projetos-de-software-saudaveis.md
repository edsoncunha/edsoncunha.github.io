---
layout: post
title:  Dois truques mágicos para manter uma base de código saudável
image: https://m.media-amazon.com/images/M/MV5BNTEyNzk4MTYtMjFkMC00YjZlLWFhMjItNTRmMzc1OGMyMzMyXkEyXkFqcGdeQXVyNjMxNDE2ODU@._V1_.jpg
tags: engenharia de software, testes, qualidade
---

Com apenas duas restrições automatizáveis, podemos conseguir desdobramentos muito interessantes para nos ajudar a manter a saúde de uma base de código.

> A arte de programar consiste em organizar e dominar a complexidade
> 
> (Edsger Dijkstra)

# 1) Não deixe a cobertura de testes cair

O ponto-chave é não deixar o projeto deteriorar. Se ele está no começo, é uma chance de ouro para começar com mais rigor e vê-lo crescer estável, legível e bem testado. Tão ou mais importante do que ter uma boa cobertura percentual de testes é impedir que ela recue. Não tenha medo de exigir 100% de cobertura no código que está para ser incorporado na _master_ ou em uma _branch_. Se essa exigência for feita desde o início do projeto, veja que maravilha: você terá 100% de cobertura de testes unitários.

_Aliás, não se limite a exigir uma cobertura total. Use esse momento para preparar toda a esteira de entrega, do commit ao ambiente de produção. Farei um novo texto sobre isso em breve._

![A pirâmide de testes de Mike Cohn](https://martinfowler.com/articles/practical-test-pyramid/testPyramid.png)

<p class="figcaption">A pirâmide de testes, de Mike Cohn. A base, que queremos alargar, é composta pelos testes unitários. 
<br />Crédito da imagem: <a href="https://martinfowler.com/articles/practical-test-pyramid.html">Martin Fowler</a></p>

## Motivações

- Aumentar a cobertura de código quando a base estiver grande é mais penoso
- Testes não nos livram integralmente de . Contudo, em qual cenário é mais provável topar com mais problemas: uma base de código com testes ou sem?
- Mesmo que o time não pratique os testes primeiro ([TDD](https://pt.wikipedia.org/wiki/Test-driven_development)), escrever um grande volume de testes depois de fazer o código é muito sofrido. O build, muito provavelmente, vai quebrar. Isso condiciona as pessoas a fazer commits menores e, por conta disso, dividir as tarefas em partes menores.
- Tarefas mais atômicas proporcionam uma melhor visibilidade do andamento do projeto e oportunidades de paralelismo entre os membros do time
- Times muito paralelos, por sua vez, tendem a perceber mais rapidamente os incômodos do código, como classes que comecem a fazer mais coisas do que deveriam. Muitas pessoas mexendo em uma mesma classe normalmente é um sinal de que ela tem muitas dependências e/ou responsabilidades. Modificá-la fica cada vez mais difícil.
    
## Como chegar lá

- Tenha um processo automatizado de build (rodando no Jenkins, Drone, Travis ou que preferir).
- Faça o build disparar a cada commit.
- Utilize um plugin para calcular como ficará a cobertura de testes com o código que está sendo enviado no commit.
- Use um plugin (como o [Cobertura](https://github.com/jenkinsci/cobertura-plugin), no caso da dupla Java + Jenkins) que falhe o build caso as métricas estejam abaixo do limite.
- O pulo do gato é atualizar os limites a cada build: se a cobertura cresceu de 40% para 44%, então essa passa a ser a cobertura mínima para o que o build passe (o Cobertura faz isso). Se, num commit seguinte, o percentual tiver caído, o build _precisa_ falhar. Esse é o coração do negócio.
- Alternativamente, use o [SonarQube](https://www.sonarqube.org/). Ele permite gerar métricas analisando apenas o código que está entrando no commit. Ele é um analisador genérico e com suporte a várias linguagens, por isso o recomendo sem cerimônia. Em seus _quality gates_, procure a métrica "Coverage on new code", sapeque 100% como exigência e associe-a ao _quality profile_ do seu projeto. Por fim, vincule o sucesso do build ao _quality gate_.

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

O que desejamos é escrever código que pareça óbvio. Se alguém vir seu trabalho e não demorar a falar um "ah, beleza!", pode ficar feliz: você está no caminho certo.

Em programação dinâmica, existe um conceito chamado _Princípio da Otimalidade_ ([Richard Bellman, 1957](https://en.wikipedia.org/wiki/Bellman_equation#Bellman's_Principle_of_Optimality)):

> Para um dado estado do sistema, a política ótima para os estados remanescentes é independente da política de decisão adotada em estados anteriores.

 Em outras palavras, uma solução é a melhor possível - a solução _ótima_ - se cada uma de suas subdivisões é também a melhor possível. Por exemplo: um pacote TCP chegará a seu destino pela melhor rota se, a cada passo, o roteador que tem o pacote  escolher como roteador seguinte aquele que proporcionar o menor tempo de transferência dos dados.

Aplicando esse conceito aos nossos projetos, o que temos é que eles serão bacanas, enxutos e fáceis de compreender se cada uma de suas partes também o forem. A classe é nossa unidade fundamental de trabalho. Se não abrirmos mão de construir classes enxutas e coesas, assim também serão os agrupamentos de mais alto nível, como pacotes, módulos, bibliotecas etc.


## Motivações
- Faz parte do nosso trabalho dividir problemas grandes em partes menores. Dividir uma classe _kaiju_ em vários calangos especialistas segue o mesmo raciocínio.
- Testar classes grandes dá um trabalho enorme, sobretudo se você escrever os testes depois do código real.
- Quando as classes se relacionam com muitas outras e/ou seus métodos têm muitos parâmetros, temos aí uma explosão combinatória de entradas, saídas e estados. Isso torna o código muito mais vulnerável a bugs, já que é difícil mapear todas as possibilidades.
- Compaixão com seu eu daqui a 6 meses, quando você nem se lembrar mais o que esse código faz.

## Como chegar lá
- Veja o código de fora para dentro, pensando NO QUE ele faz, e não COMO, e responda a pergunta: o que essa classe deveria fazer?
- Se sua resposta tiver as palavras E ou OU, é porque existe mais de uma responsabilidade. Descreva essa responsabilidade por meio de uma nova classe (testes e nomes explicativos são ótimas maneiras de comunicar as intenções). Responda a pergunta para a nova classe, e assim recursivamente.
- Use ferramentas de análise estática, como Lint e Sonar, para analisar o código. A partir de características como da quantidade de linhas, número de métodos e quantidade de relacionamentos com outras classes, essas ferramentas conseguem identificar se sua classe está virando um canivete suíço.
- Não se limite a meramente obter essas métricas. Use-as como pontos decisivos em seu processo de build. Se as métricas estiverem desfavoráveis em relação aos limites definidos por você, o build _precisa_ falhar.
- Não paralise os trabalhos do seu time. Se você ativar essa quebra de build em um projeto em andamento, configure os limites dos indicadores de qualidade de acordo com o estado atual que foi medido no projeto. Se hoje ele tem 4 kaijus e 15 megazords, assim seja.
- Se sua ferramenta de análise tiver a capacidade de inspecionar apenas o código que está entrando com o commit (como o SonarQube), aí sim, você pode ser radical: megazords recém saídos de fábrica não podem entrar e ponto final.

## Conclusões

- O rigor com o código que está entrando na base é interessante para permitir que o passivo técnico aumente. Se o projeto for iniciado já com essas barreiras, as chances de ele evoluir de maneira saudável são muito boas.
- As medidas também valem para projetos em andamento. Se eles já têm seu passivo técnico, paciência, mas ele ao menos não vai aumentar.
- O início da adoção pode ser doloroso e gerar ruídos. Contudo, com o tempo, o time se adapta e divide o trabalho em partes menores para que os ajustes sejam mais fáceis caso o código seja reprovado no processo de build.
- Por melhores que eventualmente sejam as ferramentas, as práticas só funcionam se forem abraçadas pelas pessoas. Não é apenas o ferramental que nos ajuda, mas o entendimento de que nós, como humanos, eventualmente nos distraímos e erramos. As ferramentas estão lá para nos apontar os problemas quando deixarmos o cachimbo cair.
    

### Para ler mais

- Livro: [Refatoração para padrões, de Joshua Kerievsky](https://amzn.to/2obFz9C)
- Livro: [A arte do código legível, de Boswell e Foucher](https://amzn.to/2oTeiZY)
- Artigo: [Construindo uma esteira de entrega contínua com Jenkins](https://dzone.com/articles/building-a-continuous-delivery-pipeline-using-jenk)
- Documentação: [SonarScanner para Jenkins](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-jenkins/)


Bônus:

<iframe width="560" height="315" src="https://www.youtube.com/embed/0p_1QSUsbsM" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

