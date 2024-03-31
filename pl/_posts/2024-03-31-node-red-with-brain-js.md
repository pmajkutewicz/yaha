---
lng_pair: nodered_brainjs
title: Integracja brain.js z Node-Red


category: node_red
tags: [ node_red, brainjs, howto ]
img: ":2024-03-31-brainjs-nodered/post.png"
comments_disable: false
date: 2024-03-31 02:00:00 +0200

published: true
---

# Dlaczego?

Dla zabawy i poznania Node-Red. Pomyślałem sobie, że fajnie było by mieć "czarną skrzynkę', która wyrzuci mi wynik dla zadanych parametrów.
Np. na wejściu mam:

- Nie ma mnie w domu
- Wykryto człowieka
- Alarm jest włączony

I dostaję na wyjściu

- tak, wyślij wiadomość

Oczywiście na wejściu będzie więcej parametrów

A więc padło na sieci neuronowe.

# brain.js

Znalazłem sobie taką bibliotekę, dla starszych wersji były też gotowe rozwiązania dla Node-Red. Niestety nie znalazłem nic gotowego do użycia w 2024.
Tak czy inaczej. Kilka stron do czytania i mam gotowy przykład.

## How to install

Lecimy do katalogu `~/.node-red`, to ten w którym mam `lib`, `node_modules` i `settings.js` i odpalamy tam:

```shell
npm i brain.js
```

Następnie edytujemy `settings.js`. Szukamy `functionGlobalContext` i dodajemy `brain.js`, powinno to wyglądać mniej więcej tak:

```js
functionGlobalContext: {
  // os:require('os'),
  brainjs:require('brain.js')
}
```

No i restart.

# Example

Przykład jest banalny:
![Example flow](:2024-03-31-brainjs-nodered/flow.png){:data-align="center"}

## Function node

Oczywiście to nie jest cel, który chciałem osiągnąć, ale dostałem ważną informację, działa.

### Setup

Wczytujemy `brain.js`

![Function setup](:2024-03-31-brainjs-nodered/function_setup.png){:data-align="center"}

### On Start

Tutaj tworzymy sieć i ją uczymy. Zrobimy to raz po starcie.

![Function on start](:2024-03-31-brainjs-nodered/function_onstart.png){:data-align="center"}

### On Message

A teraz tylko odpalamy sieć dla parametrów wejściowych i rezultat wysyłamy dalej.

![Function on message](:2024-03-31-brainjs-nodered/function_onmessage.png){:data-align="center"}

## Exported flow

```json
[
  {
    "id": "4f7b8a1ffd9af2d2",
    "type": "tab",
    "label": "Flow 1",
    "disabled": false,
    "info": "",
    "env": []
  },
  {
    "id": "4403444239c81a4f",
    "type": "inject",
    "z": "4f7b8a1ffd9af2d2",
    "name": "",
    "props": [
      {
        "p": "payload"
      },
      {
        "p": "topic",
        "vt": "str"
      }
    ],
    "repeat": "",
    "crontab": "",
    "once": false,
    "onceDelay": 0.1,
    "topic": "",
    "payload": "",
    "payloadType": "date",
    "x": 120,
    "y": 80,
    "wires": [
      [
        "28b6e4a5c8517672"
      ]
    ]
  },
  {
    "id": "28b6e4a5c8517672",
    "type": "function",
    "z": "4f7b8a1ffd9af2d2",
    "name": "function 1",
    "func": "var net = context.get('net')\n\nmsg.payload = net.run([1, 0]); // [0.987]\n\nreturn msg",
    "outputs": 1,
    "timeout": 0,
    "noerr": 0,
    "initialize": "// Code added here will be run once\n// whenever the node is started.\n\nconst config = {\n    binaryThresh: 0.5,\n    hiddenLayers: [3], // array of ints for the sizes of the hidden layers in the network\n    activation: 'sigmoid', // supported activation types: ['sigmoid', 'relu', 'leaky-relu', 'tanh'],\n    leakyReluAlpha: 0.01, // supported for activation type 'leaky-relu'\n};\n\n\n// create a simple feed forward neural network with backpropagation\nconst net = new brain.NeuralNetwork(config);\n\nnet.train([\n    { input: [0, 0], output: [0] },\n    { input: [0, 1], output: [1] },\n    { input: [1, 0], output: [1] },\n    { input: [1, 1], output: [0] },\n]);\n\ncontext.set('net', net)",
    "finalize": "",
    "libs": [
      {
        "var": "brain",
        "module": "brain.js"
      }
    ],
    "x": 300,
    "y": 80,
    "wires": [
      [
        "3820ad2482c3582c"
      ]
    ]
  },
  {
    "id": "3820ad2482c3582c",
    "type": "debug",
    "z": "4f7b8a1ffd9af2d2",
    "name": "debug 1",
    "active": true,
    "tosidebar": true,
    "console": true,
    "tostatus": false,
    "complete": "true",
    "targetType": "full",
    "statusVal": "",
    "statusType": "auto",
    "x": 490,
    "y": 80,
    "wires": []
  }
]
```

# Sources

Linki które mi pomogły:

- [https://github.com/BrainJS/brain.js](https://github.com/BrainJS/brain.js)
- [https://github.com/BrainJS/brain.js/issues/245](https://github.com/BrainJS/brain.js/issues/245)
- [https://flows.nodered.org/flow/195773d3b493d81c9bf012f64da02ea3](https://flows.nodered.org/flow/195773d3b493d81c9bf012f64da02ea3)



