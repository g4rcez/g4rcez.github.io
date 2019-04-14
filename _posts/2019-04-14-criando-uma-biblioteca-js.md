---
layout: post
title: "Criando uma biblioteca HTML/CSS/JS - Parte 00"
image: ""
date: 2019-04-14
tags:
    - Javascript
    - HTML5
    - CSS3
    - React
description: "Entenda os passos necessários para começar"
categories:
    - Web
---

Fala garotos e garotas de programa (os que programam, não isso que você pensou), a partir de hoje, quero
compartilhar a experiência super maneira que estou tendo que é "Criar um framework/biblioteca de componentes"
onde trabalho, com o fim de ter um styleguide padronizado e maduro, de acordo com as práticas da equipe.

Antes de começar, é preciso ter em mente que você irá esbarrar com coisas que provavelmente não conhece, então
para ninguém ficar a deriva no mar de confusão, vou deixar uma série de referências que me ajudaram a entender
muito do que eu precisava fazer.

-   [W3C School](https://www.w3schools.com): Quem nunca esqueceu alguma coisa básica, que atire a primeira pedra
-   [Over Reacted, o blog do Dan Abramov](http://overreacted.io): Como uso react, muitas das novidades de hooks acabei entendedo por lá
-   [MDN](https://developer.mozilla.org/en-US/docs/Web): Preciso nem falar o que tem aqui né?
-   [Stackoverflow](https://stackoverflow.com): Dúvidas no geral, procure aqui

Blz. Agora bora por partes pra entender o porque de precisar criar uma biblioteca sendo que já existem umas 50 milhões:

-   Styleguide do negócio: como as bibliotecas possuem estilo próprio, acabou sendo necessário customizar alguns componentes delas para que pudesse se adequar ao que usamos, e isso acabou dando mais trabalho e gerando inconsistências em alguns casos
-   Mudança contínua e adição de novos elementos
-   Evitar repetição de código

Houveram alguns outros problemas também, mas tecnicamente, isso é o que precisamos saber.

## Problema 0: Cores

Imagina você começar a escrever um milhão de códigos com `style` ou `class` no CSS e depois ter que mudar tudo? Seria uma merda infinita...Pra isso, precisei criar uma forma de guardar todas as cores em um único lugar e que fosse de fácil alteração. Pra isso, fiz o seguinte

1. Criei um arquivo `colors.js` com a estrutura `{"nomeCor":"hexadecimal"}`. _Acabou que além de cores, coloquei estilos de fonte, string para sombrar em algumas caixas, padrão de exibição do texto e ficou algo flexível para alterar tudo, seguindo a especificação de regras do CSS_
2. Em um arquivo `index.js`, fiz a importação deste arquivo e usei uma magia para que **TODO O MEU CSS** pudesse enxergar essas cores e usar nas classes CSS
3. Mágica feita, basta usar. O melhor de tudo é, apenas um lugar vc define todo o tema do seu site, sem quebrar absolutamente nada. _Ficar bonito é diferente de não quebrar_

Importante notar que no passo dois foi feita uma `magia` para que pudesse funcionar. Vamos analisar a magia feita (em Typescript):

```javascript
import config from "colors.json";
/* Conteúdo do colors.json
{
    "main":"#000000",
    "secondary": "#eeeeee",
    "body": "#141414",
    "shadow": "0px 5px 10px 0px rgba(0, 0, 0, 0.35)",
    "text-title": "#7557ad",
    "logo": "https://static-egc.xvideos-cdn.com/v3/img/skins/default/xvideos.com.svg"
}
*/
const root = document.querySelector(":root");
export default function InitColors() {
    Object.keys(config).forEach((x: string) => {
        root.style.setProperty(`--${x}`, `${config[x]}`);
    });
}
```
> Calma aí cara, vem afobado não, vem tranquilo, vem tranquilo

O que foi feito nesse pedaço de código aí? O que é `:root`? `setProperty`? Não fique confuso, vamos lá

-   `:root`: Como diz o mdn, nosso `:root` é a pseudoclasse que representa o nosso HTML - [veja aqui](https://developer.mozilla.org/en-US/docs/Web/CSS/:root). Tudo o que for colocado aqui, será visível para nosso CSS, mas como podemos acessar? Basta utilizar o `var(--variavel)` [como também diz a mdn](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties). Por isso no meu `setProperty` foi usado `(--cor: hexadecimal)`.

-   Encapsulei isso dentro de uma função para que pudesse ter controle e compor meu estilo utilizando funções, assim em alguns trechos eu poderia

-   **DICA**: caso você não tenha lido tudo sobre `Using CSS custom properties`, lembre-se que no caso da sua variável não
    for definida, você pode passar um valor de fallback `var(--css-var, #000000)`.

-   **ATENÇÃO**: você pode preferir usar o hexadecimal de certas cores com apenas 3 dígitos, mas isso pode te prejudicar quando optar por usar [transparência com hexadecimal](https://stackoverflow.com/questions/1751263/hex-colors-numeric-representation-for-transparent) e acabar em erro, pois para isso você precisa de ter obrigatoriamente os 6 dígitos + canal alfa. Para isso, basta extender suas cores com esse método:

```javascript
export default function HexExpand(hex: string) {
    // usei essa regra para não aplicar este método quando eu não utilizar cores em hexa
    // ou já inserir a cor em hexadecimal + canal alfa direto no meu arquivo de cores
    if (hex.length >= 7) {
        return hex;
    }
    return hex
        .slice(hex.startsWith("#") ? 1 : 0)
        .split("")
        .map((x) => x + x)
        .join("");
}
```

## Problema 1: CSS

Ainda que com a solução apresentada no problema 0, eu tive que copiar e colar muitas regras para diferentes cores, e isso as vezes gerava uma série de erros que acabava demorando para identificar, afinal de contas minha folha de estilo estava com quase 6.3k linhas. Para isso, eu tinha algumas opções, usar uma porrada de lib de node para gerar meu CSS, usar LESS para criar mixins e replicar minhas regras ou...escrever um Shell boladão para atualizar minha folha de estilo das cores, e foi o que eu fiz:

```shellscript
# Gera CSS para texto
grep "#" colors.json | tr -d ' ",' | sed 's/:/ { color: /g' | sed 's/^[^[:space:]]\+/.&/1' | sed 's/$/ }/g'
# Gera CSS para texto com o seletor :hover
grep "#" colors.json | tr -d ' ",' | sed 's/:/:hover { color: /g' | sed 's/^[^[:space:]]\+/.&/1' | sed 's/$/ }/g'
# Gera CSS para cor de fundo
grep "#" colors.json | tr -d ' ",' | sed 's/:/ { background-color: /g' | sed 's/^[^[:space:]]\+/.&/1' | sed 's/$/ }/g'
```

Com isso feito, joguei tudo num `script.sh` para gerar meu arquivo `colors.css` e pronto, a cada nova cor adicionada no
meu `colors.json`, eu tinha um novo css de acordo com as cores. Utilizei o `nodemon` para monitorar minhas modificações:

```shellscript
npm i -g nodemon # caso você não tenha nodemon instalado
nodemon -w colors.json --exec "bash script.sh"
```

## Conclusão 0

Essas soluções foram bem efetivas para boa parte dos problemas que tive no começo, mas ao lidar com exigências dinâmicas, era preciso criar diversas classes CSS para ter uma gama maior, mas chegaria em um ponto limite, e também não poderia haver mudanças deste tipo em tempo de execução, então essa primeira abordagem não resolveu meu problema inicial, mas foram medidas bem eficientes que me levaram a definir a estrutura do projeto e descobrir formas dinâmicas de lidar com um problema que são as cores de um site, sem precisar copiar e colar, possibilitando erros.

Espero que isso tenha te ajudado e até a próxima, onde irei abordar um pouco da estrutura de ReactJS para a criação dos componentes e como evitar a replicação de código.
