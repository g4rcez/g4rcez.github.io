---
layout: post
title:  "Fireshell CTF"
image: ''
date:   2019-01-28
tags:
- CTF
- Security
description: 'Resolvendo Alphabet, a Crypto do FireshellCTF'
categories:
- CTF
---

E aí galera, nesse fim de semana (`2019-01-26` e `2019-01-27`) rolou um CTF da Fireshell, um time BR.
O evento teve 24h de duração, mas não joguei todo o evento. Uma das challs que resolvi foi a `Alphabet`,
uma de crypto bem divertida de se resolver.

Para começar, primeiro vamos a dica:

> If you know your keyboard, you know the flag

E o [download deste arquivo](https://ctf.fireshellsecurity.team/files/60b518cdbf44d01154733be7c95cf543/submit_the_flag_that_is_here.7z), que é um txt com diversos hashs md5 e sha256 de todas, ou quase todas, as
letras de um teclado. Tendo isso, podemos ver o problema como

```
hashMD5 || hashSha256 => Chave
Tecla no teclado => Valor
```

Entenda como um hashmap, ou um objeto JS, como foi o que pensei. Tendo isso, foi simples pensar no código para resolver,
aqui a solução abaixo:

{% highlight javascript %}
const md5 = require('md5')
const hash = require('hash.js')
const alphabet = {}
function getArray(first, sec) {
    var a = [], i = first.charCodeAt(0), j = sec.charCodeAt(0);
    for (; i <= j; ++i) {
        a.push(String.fromCharCode(i));
    }
    return a;
}
const arr = [...getArray('a', 'z'), "#", "_", "-", 
    "1", "2", "3", "4", "5", "6", "7", "8","9", "0", 
    "+", "!", "@", "$", "%"
]
arr.forEach(value => { 
    const key = md5(value); 
    alphabet[key] = value 
})
arr.forEach(value => { 
    const key = hash.sha256().update(value).digest('hex'); 
    alphabet[key] = value 
})
arr.map(x => x.toUpperCase()).forEach(value => { const key = md5(value); alphabet[key] = value })
arr.map(x => x.toUpperCase()).forEach(value => { 
    const key = hash.sha256().update(value).digest('hex');
    alphabet[key] = value 
})
const fs = require('fs');
const path = require('path');
const filePath = path.join(__dirname, 'submit_the_flag_that_is_here.txt');
fs.readFile(filePath, { encoding: 'utf-8' }, function (err, data) {
    if (!err) {
        let flag = ''
        data.replace(/ /g, "\n").split("\n").forEach(line => {
            const str = alphabet[line]
            if (str) {
                flag += str
            }
        })
        console.log(flag)
    } else {
        console.log(err);
    }
});
{% endhighlight %}

Esse código gerou um output gigante, então foi meio foda achar. Pra me facilitar, tendo o padrão de flag em mente, 
fiz este comando:

```bash
node solve.js | sed 's/.*F#//g'
```

E o output começou direto na flag, foi só pegar as palavras, jogar no padrão e pontuar