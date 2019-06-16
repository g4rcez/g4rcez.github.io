---
layout: post
title: "Criando uma biblioteca HTML/CSS/JS - Parte 03"
image: ""
date: 2019-06-16
tags:
    - Javascript
    - HTML5
    - CSS3
    - React
description: "Montando componentes - A teoria de Hooks"
categories:
    - Web
---

Fala Dev (acho que isso vai dar merda de direitos autorais), tranquilo? Bom, nesse post finalmente teremos a criação de um componente React. O primeiro que escolhi foi o `CurrencyInput`. Ainda não consegui parar e disponibilizar o código, então vou fazendo aqui e posto um gist no final, beleza? _Se não estiver, problema seu, to tentando ajudar kkkk_

> Me inspirei fortemente em como o [Nubank](https://nubank.com.br/#/) fez seu CurrencyInput no app, e acho que consegui fazer algo bastante próximo.

Nesse componente, tomei cuidado com o teclado exibido no mobile. Como o CurrencyInput contém caracteres monetários `^[A-Z]{1,3}[0-9$,. ]+$`, não podemos fazer com que ele seja do `type=number` e sim `type=text`. Mas `type=text` exibe um teclado alfanumérico e o nosso caso exige apenas números e vírgula, no caso da moeda ser brasileira. Para resolver esse problema, basta

```javascript
return (
    <input
        {...props} // Assumindo todas as propriedades passadas
        type="text" // tipo de texto
        value={value} // a mágica da máscara vai ser feita aqui
        pattern="^[A-Z]{1,3}[0-9$,. ]+$" // o pattern para o corrigir a exibição
        inputMode="decimal" // o formato do teclado para ser exibido no mobile
        name={props.name} // nome do input
        onChange={change} // nossa função de onChange
    />
);
```

Para criar a máscara, precisamos de algumas funções para tratar nossos valores. Vamos listar:

-   Transformar a máscara monetária em valor inteiro _(nesse caso, vc pode pesquisar pelo [sidekicker](https://github.com/vandalvnl/sidekicker))_
-   Remover os zeros a esquerda _(pois começaremos digitando os centavos e depois os reais)_
-   Adicionar os decimais/centavos
-   Adicionar o formatador da moeda _("R\$ " no nosso caso)_
-   Fazer o padding dos zeros caso esteja sendo digitado os centavos

Como já disse, transformar valores monetários em valor inteiro eu utilizei minha própria lib [sidekicker](https://github.com/vandalvnl/sidekicker) para fazer isso, mas você pode conferir o código:

> Todos os exemplos são em Typescript, pois a lib está sendo escrita em Typescript

```javascript
export const formatBrlToFloat = (currency: string) => {
    const final = currency
        .replace(/\./g, "")
        .replace(/,/g, ".")
        .replace(/[^0-9\.]/g, "");
    return Number.parseFloat(final);
};
```

Agora precisamos do padding para centavos, remover os zeros a esquerda e adicionar os centavos

```javascript
// Removendo o que não for valor numérico
const fromValue = (value = "") => value.replace(/(-(?!\d))|[^0-9|-]/g, "") || "";

// Adicionando o padding de zeros caso seja centavos
const padding = (digits: string) => {
    const minLength = 3;
    const currentLength = digits.length;
    if (currentLength >= minLength) {
        return digits;
    }
    const amountToAdd = minLength - currentLength;
    return `${"0".repeat(amountToAdd)}${digits}`;
};

// Removendo os zeros a esquerda
const removeLeadingZeros = (num: string) => num.replace(/^0+([0-9]+)/, "$1");
```

Com isso, temos tudo pronto para criar nossas conversões finais para a máscara e o método para incializar o input caso não haja nada preenchido, e teremos `R$ 0,00`.

```javascript
// Método mágico que faz a criação da máscara
export const toCurrency = (value: string, separator = ",", prefix = "R$ ") => {
    const valueToMask = padding(fromValue(value));
    return `${prefix}${addDecimals(valueToMask, separator)}`;
};
// Aqui faço a tipagem de possíveis valores vindos do type=text que o próprio HTML tem como regra
type InputTypeText = string | number | string[];
// Conversão segura para inicializar os valores
const safeConvert = (str: InputTypeText = "0") => toCurrency(Number.parseFloat(`${str}`).toFixed(2));
```

Juntando tudo, teremos nosso componente finalizado. Se liga no código final e não deixe de ler os comentários

```javascript
import React, { useState, useEffect } from "react";
import { formatBrlToFloat } from "sidekicker/lib/strings";

type onChangeParameters = React.ChangeEvent<HTMLInputElement> & { target: { rawValue: number } }
export type CurrencyInputType = React.InputHTMLAttributes<HTMLInputElement> & {
	prefix?: string;
	separator?: string;
	onChange?(e: onChangeParameters): any;
};

type InputTypeText = string | number | string[];

const fromValue = (value = "") => value.replace(/(-(?!\d))|[^0-9|-]/g, "") || "";

const padding = (digits: string) => {
	const minLength = 3;
	const currentLength = digits.length;
	if (currentLength >= minLength) {
		return digits;
	}
	const amountToAdd = minLength - currentLength;
	return `${"0".repeat(amountToAdd)}${digits}`;
};

const removeLeadingZeros = (num: string) => num.replace(/^0+([0-9]+)/, "$1");

const addDecimals = (num: string, separator = ",") => {
	const centsStart = num.length - 2;
	const amount = removeLeadingZeros(num.substring(0, centsStart));
	const cents = num.substring(centsStart);
	return amount + separator + cents;
};

export const toCurrency = (value: string, separator = ",", prefix = "R$ ") => {
	const valueToMask = padding(fromValue(value));
	return `${prefix}${addDecimals(valueToMask, separator)}`;
};

const safeConvert = (str: InputTypeText = "0") => toCurrency(Number.parseFloat(`${str}`).toFixed(2));

// o prefixo default e o separador são os da moeda brasileira, logo, você não precisa mudar caso
// esteja usando formatação BRL
export default ({ prefix = "R$ ", separator = ",", name = "", ...props }: CurrencyInputType) => {
    // Hook para ter o estado dentro de um componente de função, assim poderemos manipular
    // de acordo com a necessidade no nosso onChange, mas mantendo o valor do usuário
    const [value, setValue] = useState(safeConvert(props.value));

    // O hook use effect para ser executado sempre que nosso valor (props.value) mudar
    useEffect(() => {
		setValue(safeConvert(props.value));
	}, [props.value]);

    // O nosso evento onChange recebe o real evento do HTML, sem mudanças
	const change = (e: React.ChangeEvent<HTMLInputElement>) => {
        // AQUI É MÁGICA, AMIGO. Onde a cada vez que o valor muda, nós aplicamos a função toCurrency
        // que transforma o valor de texto simples em um valor monetário
        const valueAsCurrency = toCurrency(e.target.value, separator, prefix);
        // Alteramos o valor do estado para a mudança ser aplicada ao input
        setValue(valueAsCurrency);
        // Caso seja fornecido um método de onChange...
		if (props.onChange) {
			e.persist();
			return props.onChange({
				...e,
				target: {
                    ...e.target,
                    // O valor "cru", transformando monetário em um number do JS
                    rawValue: formatBrlToFloat(valueAsCurrency),
                    // O valor com máscara
                    value: valueAsCurrency,
                    // Apenas propagando o nome passado
					name: props.name || ""
				}
			});
		}
	};
    // Aqui não muda nada do começo do post
	return (
		<input
			{...props}
			type="text"
			value={value}
			pattern="^[A-Z]{1,3}[0-9$,. ]+$"
			inputMode="decimal"
			name={props.name}
			onChange={change}
		/>
	);
};
```

E é isso, se vc separar em etapas o que deve fazer, tudo fica mais claro e você irá conseguir desenvolver seu componente. Esse deu um certo trabalho devido aos replaces e tudo mais, porém saiu. Talvez alguns bugs ainda existam que não tive tempo de testar exaustivamente. Mas sinta-se a vontade para comentar aí e me ajudar a resolver ou implementar o seu próprio e falar que o seu é melhor.

Ainda estou organizando o repositório com tudo da série para que você consiga ter algo mais palpável, aguarda que irei postar sobre. 

E é isso aí galera, esse foi o primeiro componente da nossa série, espero que tenham gostado e entendido o processo do desenvolvimento.