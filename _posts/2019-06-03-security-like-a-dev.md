---
layout: post
title:  "Security Like a Dev"
image: ''
date:   2019-06-03
tags:
- Security
- Development
description: 'Pensar em segurança como um programador ou programar como um pentester?'
categories:
- Security
---

Começando com o pé direito + alívio cómico

![o mundo não é mais o mesmo](https://2.bp.blogspot.com/-D8B7BIra_tY/WcbUhaWlinI/AAAAAAAAA5Q/Dj0LPvqDCu8AttI99XRUBOLD8j3I9oKIwCLcBGAs/s1600/logan.JPG)
***Você quando vai jogar uma chall de CTF e não acha PHP***

---

Eu posso concordar, por que em menos de 2 anos trabalhando numa startup eu pude ver diversas das tecnologias
que usávamos sendo modificadas agressivamente, algumas sendo deixadas de lado e sendo substituídas por outras, *como foi o caso de Ruby com NodeJS*.

Mas o que isso tem a ver com segurança? Antes de responder, quero dizer que este artigo é uma tentativa de mudar a mentalidade da comunidade de segurança atual, e se você não concorda, você pode me xingar muito no *Twitter* `(e se dar mal por que não tenho twitter pra ler suas ofensas)` ou caso concorde, venha me dar um alô no Telegram. Mas para todos os dois casos, você irá começar a ver segurança e desenvolvimento de uma forma totalmente diferentes

## As falhas comuns

Todo mundo que estuda segurança web conhece o Top10 OWASP, e também sabe que essas falhas apesar de comuns, podem ser facilmente corrigidas, ou melhor, elas podem **NÃO EXISTIR**.

> Sim meu caro amigo, elas podem não existirem, ou serem tão dificultadas a ponto de um atacante inexperiente ou impaciente não reconhecer essa falha no seu sistema


Mas como fazer isso? Simples, siga o conselho da maioria dos desenvolvedores conscientes e faça o favor **DE NÃO REINVENTAR A RODA**. Não que você seja um péssimo programador ou que você não deve tentar inovar, mas existem equipes muito bem pagas, pesquisadores com bastante incentivo...todos eles trabalhando/estudando por diversas horas, afim de melhorar uma determinada tecnologia. Não só isso, eles fazem testes exaustivos, e eu tenho certeza que você não deve testar sua aplicação, ou não testa ela da maneira que deveria

> Relaxa, a maioria não faz. E antes de me xingar, eu também não faço como deveria. Isso não é ser incompetente, é ser vida louca em produção **xD**

*SQLi* e *XSS*. Vamos refletir sobre essas duas falhas em comum. Pense no seguinte ambiente. Uma aplicação de uma grande empresa que leva a sério a qualidade de seu produto, escreve os testes da forma correta, têm programadores que são conscientes quanto a segurança e ainda possui um time de *security* para testar a aplicação depois de feito o *deploy*. Qual a chance de conseguir achar uma falha de SQLi e XSS? Antes de responder, considere os fatos

- A empresa usa uma linguagem consolidada. Vamos adotar C# pois além de performática (dada as proporções para uma linguagem humanamente entendível e de fácil manutenção) e com um mega suporte da *Microsoft*.

- Além da linguagem consolidada, eles usam frameworks especializados em cada parte crucial do sistema para garantir que não haja nenhuma falha exposta tão facilmente

- A equipe de programadores /testadores testam o sistema, incluindo testes unitários, testes funcionais, testes de integração, testes End2End...

- A equipe de *security* realiza os testes no sistema, reportando quaisquer falhas para a equipe de desenvolvimento corrigir qualquer possível falha/bug que cause dano ao sistema

Mais uma vez vale lembrar, não estou falando que existe 100% de segurança nesse sistema e que ninguém será capaz de explorar tal sistema. Mas pense bem...dado esse cenário, qual a possibilidade de haver uma falha comum como SQLi ou XSS? Ela é baixa, bem baixa...

> Vandal, você ainda não expôs seu ponto de vista diferentão sobre segurança e desenvolvimento. Ta me enrolando, p*%$a?

## Mindset newMindset = new Mindset("DevSecOps")

Mais informações sobre o [DevSecOps](https://www.redhat.com/en/topics/devops/what-is-devsecops).

Claro que frameworks possuem falhas, mas como corrigir essa falha no seu sistema? Basta atualizar a versão, atualizar trechos do seu código de acordo com a nova API do framework e pronto, você terá uma possível falha corrigida somente editando um arquivo XML ou JSON. Isso é uma coisa que seu editor de texto com plugins pode te alertar. Ouuuuuu...você pode escrever um pequeno bot em *Python* (sim, Python, bots são em Python, não quebre a tradição só pq Python é feio kkk) que revise as dependências do seu projeto e atualize-as para que você não se preocupe com isso, afinal de contas...Programar é um processo constante de criação de preguiça, tudo o que é manual, você automatiza.

Claro que só isso não vai garantir a segurança. Afinal de contas, você pode estar usando uma API da forma insegura e ferrar com todo o bom trabalho. Nesse caso, podemos apelar para o CodeReview, seja manual ou automatizado com bots do GitHub. É uma boa técnica que pode te auxiliar, mas se você quiser garantia em tempo de desenvolvimento, aconselho bastante que você conheça mais sobre [Linters](https://en.wikipedia.org/wiki/Lint_(software)), com os editores poderosos hoje em dia é moleza configurar um linter para avisar se você está usando uma versão descontinuada de uma API ou se tal método é inseguro

> Com a ajuda de linters, só faz cagada quem quiser forçar algo completamente absurdo.

Claro que os linters não são uma resposta definitiva para o nosso problema. Afinal de contas, programamos grandes trechos de códigos e fazemos apenas um teste superficial.

## Testes, testes, testes...Não testei

É super normal na correria de empresa pra entrega de **PROJETOS** uma equipe deixar de fazer os testes e alegar *"O cliente testa e se der merda a gente corrige"*. Atire a primeira placa mãe aquele que nunca viveu isso.

Com todo o ferramental, ainda podemos fazer cagadas, e é nos testes que nós iremos pegar isso. Vou tentar criar um pequeno cenário ideal de como ocorre o *deploy* de uma aplicação

1. Escrita do código
2. Sincronização em um controle de versão (Git, use sempre Git, esqueça SVN ou Mercurial )
3. Servidor de build capta as mudanças, seja por tag ou commit na master, e irá clonar seu projeto e buildar/compilar/transpilar ou seja lá o que seu `build` faz
4. Caso o build não retorne erros, em poucos minutos a sua nova versão estará no ar

Aaah, que lindo...entrega contínua. Seria mais lindo se o nosso servidor de build falasse 

`"Cara, deu merda na linha N, corrige lá por que eu não vou assumir esse BO"`.

E o melhor de tudo é que ele fala, basta você incluir os testes no seu projeto e ele irá executar todos os testes antes de fazer o build. A melhor parte vem agora...**E SE A NÓS ESCREVÊSSEMOS OS TESTES DE SOFTWARE JUNTO DO TIME DE SECURITY? EU PODERIA EVITAR SUBIR UMA APLICAÇÃO COM SQLI, XSS, CSRF E MAIS OUTRAS FALHAS?**. A resposta é clara:

# SIM

Sim, você pode, e a partir desse momento, você deverá fazer isso. Qual a dificuldade de escrever um teste que irá testar o método de login com a seguinte query:

```SQL
--- Aqui ta tudo ok
SELECT * FROM users WHERE login = %s AND %s LIMIT 1

--- Aqui vai dar ruim
SELECT * FROM users WHERE login = %s AND %s OR 1 = 1 ---LIMIT 1
```

Claro que esse exemplo é bobo, mas escrevi até aqui para que você pudesse considerar a seguinte ideia:

`Nós podemos testar as falhas da nossa aplicação antes mesmo dela existir, então por que não adotar tal prática?`

*Talk is cheap, show me the code*. Irei fazer isso mais a frente, mas nesse artigo, espero ter conseguido fazer com que você pense nessa ideia e melhore o seu workflow de desenvolvimento. São esses pequenos mindsets que irão começar a garantir a segurança do seu sistema e noites melhores dormidas.

É isso garotas e garotos de programa, até a próxima.

## Referências

Em alguns trechos, parti do princípio que o leitor saberia o linguajar técnico, mas se esse não for o caso, seguem algumas referências

- [Qual a diferença entre API, Biblioteca e Framework](https://pt.stackoverflow.com/questions/17501/qual-%C3%A9-a-diferen%C3%A7a-de-api-biblioteca-e-framework)
- [DevSecOps](https://www.redhat.com/en/topics/devops/what-is-devsecops)
- [C# e a dotnet core](https://docs.microsoft.com/en-us/dotnet/core/get-started)
- [Awesome infosec](https://github.com/onlurking/awesome-infosec)
- [XML e JSON](https://www.w3schools.com/js/js_json_xml.asp)