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

# 1) Não deixe a cobertura de código cair

O ponto-chave é não deixar o projeto deteriorar. Se ele está no começo, esta é uma chance de ouro de começar com mais rigor e vê-lo resultar em um projeto mais estável, legível e bem testado. Tão ou mais importante do que ter uma boa cobertura percentual de testes é impedir que esse número recue. Não tenha medo de exigir 100% de cobertura de testes no código que está para ser incorporado na master ou em uma branch. Se essa exigência for colocada desde o pontapé inicial do projeto, veja que maravilha: você terá 100% de cobertura de testes unitários 

![](https://miro.medium.com/max/1024/1*ijNN8c4Gbyk5dYlOX3YCKQ.png)

<p class="figcaption">A pirâmide de testes. A base, que queremos alargar, é composta pelos testes unitários. 
<br />Crédito da imagem: <a href="https://medium.com/android-dev-br/explorando-a-pir%C3%A2mide-de-testes-no-android-parte-1-18ea135808df">Phellipe Silva / Android Dev BR</a></p>

- Por quê?
    - Aumentar a cobertura de código quando a base estiver grande é mais penoso
    - A exigência de cobertura implica em subir código que vai efetivamente ser usado, evitando desperdícios
    - Mesmo que o time não pratique os testes primeiro (TDD), escrever um grande volume de testes depois de fazer o código é muito sofrido. O build, muito provavelmente, vai quebrar. Isso condiciona as pessoas a fazer commits menores e, por conta disso, dividir as tarefas em partes menores. 
    - Tarefas mais atômicas proporcionam uma melhor visibilidade do andamento do projeto e oportunidades de paralelismo entre os membros do time
    - Times muito paralelos tendem a perceber mais rapidamente os incômodos do código, como classees que comecem a fazer mais coisas do que deveriam. Muitas pessoas mexendo em uma mesma classe normalmente indica que ela tem muitas dependências e/ou responsabilidades, o que torna a evolução mais difícil e os conflitos de merge mais frequentes
    
- Como?
    - Sonar + Jenkins
    - Integração contínua como cultura e uma ideia a ser comprada pelo time. Apenas o ferramental não basta.
    - Plugins de gamificação - CCTray + áudios no commit bem-sucedido, na quebra, na tentativa fracassada de consertar e na bem-sucedida

# 2) Mantenha as classes com uma única responsabilidade

![Foto em que se vê um enorme kaiju atrás de prédios de três andares. O monstro é três vezes mais alto do que os prédios](https://img1.looper.com/img/gallery/the-best-kaiju-movies-youve-never-seen/intro-1516897231.jpg "Vai um suco de kaiju?")

<p class="figcaption">Uma classe <em>kaiju</em>, aquela que todo mundo tem medo de mexer</p>

- Por quê?
    - Faz parte do nosso trabalho dividir problemas grandes em partes menores. Com as classes não poderia ser diferente.
    - Porque objetos foram pensados para serem pequenas células, não panaceias (no fim das contas, é esse o objetivo quando vemos autores falarem sobre alta coesão e baixo acoplamento)
    - Fazer testes para classes do tamanho de um kaiju dão um trabalho enorme
    - Muitas entradas geram uma explosão combinatória de estados e resultados possíveis nas operações dos objetos, tornando-os mais suscetíveis a bugs e aumentando o custo das correções
    - Menos relacionamentos e responsabilidades = menos linhas
    - Compaixão com seu eu daqui a 6 meses, quando você nem se lembrar mais o que esse código faz

- Como?
    - Veja o código de fora para dentro, pensando NO QUE ele faz, e não COMO, e responda a pergunta: o que essa classe deveria fazer?
    - Não tenha vergonha de criar novas classes
    - Sonar (métricas no código entrante), Sonar (métricas no código já existente), Lint (commit hook), Jenkins (quality gate baseado em acoplamento, número de linhas e de funções)

- Resultado esperado:
    - Código enxuto, mais bem modelado e fácil de navegar. 
    - Força a mente a pensar em sub-abstrações. Ficamos melhores em definir "o que é", em vez de "como". Aprimoramento da modelagem de objetos. 
    - Código mais enxuto
    - Código mais óbvio
    - Código mais dissertativo
    

### Para ler mais
- [Livro: Refatoração para padrões, de Joshua Kerievsky](https://amzn.to/2obFz9C)
- [Livro: A arte do código legível, de Boswell e Foucher](https://amzn.to/2oTeiZY)
- [Construindo uma esteira de entrega contínua com Jenkins](https://dzone.com/articles/building-a-continuous-delivery-pipeline-using-jenk)
- [SonarScanner para Jenkins](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-jenkins/)


Bônus:

<iframe width="560" height="315" src="https://www.youtube.com/embed/0p_1QSUsbsM" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

