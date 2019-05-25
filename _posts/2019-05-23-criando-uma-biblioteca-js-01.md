---
layout: post
title: "Criando uma biblioteca HTML/CSS/JS - Parte 01"
image: ""
date: 2019-05-23
tags:
    - Javascript
    - HTML5
    - CSS3
    - React
description: "Uma solução multitenant"
categories:
    - Web
---

Demorou, mas finalmente comecei a escrever. O problema de hoje não envolve muito do framework, mas sim um pouco
do ecossistema que ele irá habitar, então resolvi transcrever a solução. O que vou apresentar aqui será utilizando [ExpressJS](https://expressjs.com/), porém no meu caso foi utilizando [F#](https://fsharp.org/), como o foco é Javascript, então vou fazer uma solução bem de boa pra ficar fácil entender (e meio gambiarra para evitar milhões de arquivos no post).

## Problema 2: Multitenant

Primeiro, pra quem não sabe o que é o conceito de _Multitenant_, pode ler esse [post do Stackoverflow](https://pt.stackoverflow.com/questions/179757/o-que-%C3%A9-multi-tenancy) ou [esse vídeo](https://www.youtube.com/watch?v=z9xVPZijiOY) que também está nas referências.

Agora sim o problema. Temos o mesmo frontend para diversas marcas, porém, apenas cores, textos e endpoints irão mudar, a lógica é sempre a mesma. As cores foram resolvidas já, porém precisamos alterar a forma com que buscamos, os textos ainda precisam ser alterados, endpoints basta trocar a URL, mas como mudar isso tudo se ao usar [CRA](https://github.com/facebook/create-react-app) ele gera um build estático?

-   Solução 1: Colocar tudo lá dentro e decidir o que usar no load da aplicação (péssimo)
-   Solução 2: Fazer uma requisição para algum arquivo que irá dizer o que o site deverá usar como configuração e distribuir tais dados para a árvore do React (elegante, porém pode ficar poluída)
-   Solução 3: Incluir uma variável na window através de um outro arquivo Javascript, fora da nossa aplicação React (pode parecer gambiarra, talvez seja, mas foi a solução que adotei)

Como já era necessário um servidor (no nosso caso, um server F#) para trabalhar com o problema dos Tenants, apenas fiz uma modificação para que pudesse adotar a **Solução 3**.

**_O exemplo de código mostrado abaixo não deve ser reproduzido em produção, fiz isso apenas para reduzir a complexidade de visualização_**

```javascript
import express from "express";
const app = express();
app.set("view engine", "pug").set("views", path.join(__dirname, "app"));
app.get("/", function(req, res) {
    const [tenant] = req.subdomains;
    // Um função que traga o arquivo de configuração de acordo com o Tenant
    const configs = __FUNCAO_QUE_ACESSA_UM_BUCKET_S3_COM_AS_CONFIGS_DO_TENANT(tenant);
    res.render("index", {
        configFile: configs.file,
        js: `${configs.js}/${configs.version}`,
        css: `${configs.css}/${configs.version}`
    });
});
```

Agora, no pug file (perdoe qualquer erro, nunca usei pug na vida kkk):

```
html
    head
        script(src=#{configFile})
        link(rel="stylesheet", href=#{css})
    body
        noscript Aí não da irmão
        div(id="root")
        script(src#{js})
```

Essa é a lógica presente no backend, a const `configs` trás todo o JSON referente ao tenant, com versões, nomes de arquivo para inclusão e afins. Ainda não consegui me convencer de que essa solução de fazer um append em `window` com a minha configuração foi a melhor, mas afinal de contas, todos os frameworks JS acabam fazendo isso também, por que eu não posso?

O arquivo importado de configuração é nada mais nada menos do que:

```javascript
window.__APP_MULTITENANT_CONFIG__ = {
    // um json com todas as cores e URLs, textos e blablabla
};
```

Lembra da função usada para configurar cores? Agora ela fica assim:

```javascript
// apenas converte hexas de #000 para #000000, assim garanto poder usar o padrão
// hexa + alpha em alguns casos.
import HexExpand from "@/utils/HexExpand";
const { colors, texts, endpoints, logos } = window.__APP_MULTITENANT_CONFIG__;
Object.keys(colors).reduce((acc, el) => ({ ...acc, [el]: HexExpand(colors[el]) }), {});
export const logo = logos.main;
export const logo2 = logos.secondary;
export default colors;
```

E showzera, matamos o problema de cores e multitenant (parte dele) em nossa aplicação, tudo continua funcionando e ao acessar `tenant.awesomesite.io`, receberemos tudo de `tenant`, e ao acessar `foo.awesomesite.io`, teremos tudo de `foo`. O melhor de tudo é que a aplicação não irá saber quais dados ela terá, o backend que dará a garantia de que os dados de `tenant` e `foo` fiquem isolados.

## Problema 3: Textos e i18n

**_i18n_** === Internationalization

Os textos viraram um problema por conta de marketing. "Usa essa frase assim pra dar mais efeito", ouvindo isso várias vezes, acaba virando um problema pra você, que precisa desviar parte das tarefas para alterar um pequeno texto e perde-se todo o foco. E quando precisa de tradução? Aí amigo, f\*\*\*u...

Para me ver livre, decidi adotar a solução que o [i18n](https://www.npmjs.com/package/i18n) nos oferece. Tudo ia bem, tudo tranquilo...até começar a ver diversas função espalhadas, alguns componentes não recebiam as props decentemente e não exibia texto, não traduzia...várias merdas. Talvez eu tenha errado ao usar, mas também me incomodava o fato de que somente `texto` e `data` necessitavam de tradução/conversão, então decidi criar minha própria biblioteca pra esse projeto. Segue o código:

```javascript
// texts.js # Meu componente que resolve os valores a serem traduzidos
import React, { Fragment, useMemo } from "react";
import Translates from "@/locales";
import { convertToDate, convertToDateTime, convertToHour } from "@/converter";
const resolve = (path = "", schema = {}, separator = ".") => {
    return path.split(separator).reduce((prev, curr) => prev && prev[curr], schema);
};
export default function Texts({ isDate = "", isDateTime = "", isHour = "", children }) {
    if (!!isDate) {
        return <Fragment>{convertToDate(children)}</Fragment>;
    } else if (!!isDateTime) {
        return <Fragment>{convertToDateTime(children)}</Fragment>;
    } else if (!!isHour) {
        return <Fragment>{convertToHour(children)}</Fragment>;
    }
    const context = Translates();
    const text = useMemo(() => resolve(children, context), [context]);
    return <Fragment>{text}</Fragment>;
}
```

O arquivo Translates é uma função que identifica o idioma do browser e retorna o objeto que está em `window.__APP_MULTITENANT_CONFIG__`. Esse objeto possui os textos, e para acessar, você precisa passar a string com o path até a propriedade que você quer, e é isso que a função `resolve` faz. Se liga no uso desse componente:

```javascript
<Texts>path.to.text.in.config</Texts>
```

Como são diversos textos e nem todos sabem os caminhos, criei um "mapa", que é nada mais nada menos do que um objeto com objetos que possum propriedades da string para acessar os paths de tradução, algo como:

```
export default {
    Company: {
        Report: {
            Title: "Company.Report.Title"
        }
    },
    Clients: {
        Navbar: {
            Home: "Clients.Navbar.Home"
        }
    }
}
```

E claro, tem uma mini documentação de como usar esse componente e uma explicação de porque (faça isso sempre e quando for sair de férias da empresa, ninguém vai te ligar perguntando como usar tal método).

## Conclusão 1

Bom, ficou um pouco maior do que eu gostaria, mas acho que consegui explicar como resolver o problema de multitenant. Não foi tão fácil fazer isso como pareceu no texto, a solução pode não ser a perfeita, mas é a melhor solução que tive para o meu problema. Com essa solução, os designers e "marketeiros" poderão criar temas e modificar textos sem atrapalhar meu desenvolvimento, e eu não vou precisar ser notificado de que um novo tema foi criado, basta eles saberem o que é JSON e seguir o arquivo de exemplo que criei para eles.

Espero que tenham gostado, e no próximo capítulo pretendo mostrar como farei para criar a workspace de trabalho para componentes e em paralelo, todas as configurações do eject do CRA.

## Referências

-   [Multitenant](https://www.youtube.com/watch?v=z9xVPZijiOY)
