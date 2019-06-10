---
layout: post
title: "Criando uma biblioteca HTML/CSS/JS - Parte 02"
image: ""
date: 2019-05-23
tags:
    - Javascript
    - HTML5
    - CSS3
    - React
description: "Montando componentes - A teoria de Hooks"
categories:
    - Web
---

Nessa parte, já sabemos como será feito o estilo dos nossos componentes devido aos capítulos anteriores. A folha de estilos utilizada será da biblioteca [tachyons](http://tachyons.io/), com umas modificações nas cores, para adaptar ao padrão.

Os componentes apresentados serão feitos utilizando a nova API do React para controle de estado, os [`React Hooks`](https://reactjs.org/docs/hooks-reference.html). Mas nessa parte de hoje, apenas irei demonstrar como utilizar os hooks para criar os componentes na parte 03.

> Optei por utilizar hooks pela facilidade de trabalhar com reação a mudanças de estado e pela flexilibidade e reaproveitamento de código que eles nos proporcionam

## [useEffect](https://reactjs.org/docs/hooks-effect.html)

Como diz a documentação `Similar to componentDidMount and componentDidUpdate`. O `useEffect` é executado em toda renderização, e de acordo com suas dependências, ele executa a função passada. Mas pera, dependência? Função? Calma bb, se liga no exemplo:

```javascript
import React, { useEffect } from "react";

export default (props) => {
    useEffect(
        () => {
            document.title = `Página de ${props.name}`;
        }, // Essa era a função de argumento, que será executada toda vez que props.name for atualizado para um novo valor
        [props.name] // Essa é a dependência do useEffect
    );
    return <h1>Usuário: {props.name}</h1>;
};
```

Reforçando o papel do useEffect como o `componentDidMount` e `componentDidUpdate`, podemos utilizar o useEffects para facilitar nossa vida ao lidar com campos em formulários, por exemplo. Nesse exemplo irei usar o `useState` e a explicação sobre virá logo a seguir.

```javascript
import React, { useEffect, useState } from "react";

export default (props) => {
    const [text, setText] = useState("Fuba"); // foobar é pra gringos, BR de verdade usa o fuba
    const [error, setError] = useState(false);
    useEffect(() => {
        setError(text === "foobar");
    }, [text]);
    return (
        <form>
            <input value={text} onChange={(e) => setText(e.target.value)} />
            {error && <small style={{ color: "red" }}>Use fuba e não foobar</small>}
        </form>
    );
};
```

Nesse pequeno exemplo, você já tem uma forma de validar se o valor do seu input segue sua regra de negócio toda vez que ele mudar. Isso acaba facilitando suas interações com o usuário com menos código escrito como era usando `class components` e de forma mais elegante. **NOTA: NÃO DEFENDO O PONTO DE QUE HOOKS SEJAM UMA BALA DE PRATA.**

Antes ainda de passar para o useState, gostaria de apresentar um hook para identificar o tamanho da tela. Nele há uma particularidade que não citei ainda, que é a ação quando o componente desmontar. Para que o useEffect saiba como se comportar quando o componente desmontar, você precisará retornar uma função para que seja executada, limpando seus eventListeners, cancelando requests, limpando timeouts (ou _Timóteo_ como gosto de falar) e intervals.

```javascript
import { useEffect, useState } from "react";
export function useWidth() {
    const [width, setWidth] = useState(window.innerWidth);
    useEffect(() => {
        const resize = () => setWidth(window.innerWidth); // crio uma função de resize para adicionar ao meu event listener
        window.addEventListener("resize", resize); // passo a função criada como referência para o event listener
        return () => window.removeEventListener("resize", resize); // passo a mesma referência de função para que o event listener saiba o que remover. Lembre-se sempre de passar a mesma função de referência
    }, []); // um array vazio de dependências garante que o useEffect só será executado uma vez, que será quando o componente montar. Não passar as dependências para o useEffect irá causar execução sempre que houver renderização
    return width;
}
```

## [useState](https://reactjs.org/docs/hooks-state.html)

O hook useState é quem garante estado (sério mesmo cara?) ao nosso componente de função. Você poderá se sentir tentado a colocar cada campo de estado dentro de uma variável, mais ou menos assim:

```javascript
const [fu, setFu] = useState("fu"); // Não faça isso
const [ba, setBa] = useState("ba"); // Não faça isso
```

O correto para isso seria:

```javascript
const [fuba, setFuba] = useState({ fu: "fu", ba: "ba" }); // Não faça isso
```

Um exemplo bem mais concreto e [clássico da documentação do React](https://reactjs.org/docs/forms.html) para forms, porém utilizando Hooks:

```javascript
import React, { useState } from "react";
export default () => {
    const [state, setState] = useState({ isGoing: false, numberOfGuests: 2 });
    const onChange = (e) => {
        const { name, type } = e.target;
        const value = type === "checkbox" ? e.target.checked : e.target.value;
        // Ao passar uma função, vc recebe como argumento o estado para poder operar, 
        // nesse caso, estou retornando todo o estado e adicionando os novos valores
        setState(prev => { ...prev, [name]: value })
    };
    return (
        <form>
            <label>
                Is going:
                <input name="isGoing" type="checkbox" checked={isGoing} onChange={onChange} />
            </label>
            <br />
            <label>
                Number of guests:
                <input name="numberOfGuests" type="number" value={numberOfGuests} onChange={onChange} />
            </label>
        </form>
    );
};
```

Não tem muito mistério, não é mesmo? Ainda gostaria de apresentar o `useReducer` para casos mais complexos de manipulação de estado, o `useContext` para a ContextAPI e o `useRef`. Mas esse já ficou um pouco maior do que eu queria, então vou encerrar por aqui. Espero que todos tenham entendido e é isso aí << EOF