---
lng_pair: nodered_brainjs
title: Integrating brain.js with Node-Red

category: node_red
tags: [ node_red, brainjs, howto ]
img: ":2024-03-31-brainjs-nodered/post.png"
comments_disable: false
#date: 2024-03-31 02:00:00 +0200

#published: true
---

# You ask me why?

For fun and pleasure. I'm testing few things and one part of it is solution for "black box" that will decide what to do
based on incoming parameters.
For example, my inputs will be:

- I'm out of home
- Person detected
- Alarm enabled

Output:

- send message to me

Of course I'm planning to heve few more input params.

So why not Neural network?

# brain.js

Founded that lib, and looks very promising, but there is no out of box node to use in Node-Red. Anyway, thanks to a few
sites and post, I have working flow:

## How to install

First go to `~/.node-red` dir, the one with `lib`, `node_modules` and `settings.js` and execute:

```shell
npm i brain.js
```

After installation edit `settings.js`. Look for `functionGlobalContext` and edit (as follows):

```js
functionGlobalContext: {
  // os:require('os'),
  brainjs:require('brain.js')
}
```

After that - restart Node-Red.

# Example

Example flow is trivial:
![Example flow](:2024-03-31-brainjs-nodered/flow.png){:data-align="center"}

## Function node

This is basic example and still doesn't do what I want, but is perfectly fine as proof of concept.

### Setup

I need to load `brain.js` lib

![Function setup](:2024-03-31-brainjs-nodered/function_setup.png){:data-align="center"}

### On Start

Here I can create Neural network and train it. No need to do it on every message.

![Function on start](:2024-03-31-brainjs-nodered/function_onstart.png){:data-align="center"}

### On Message

And here is trivial part, parse parameters to network, and pass output to next node.

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

Links to pages, that helped me:

- [https://github.com/BrainJS/brain.js](https://github.com/BrainJS/brain.js)
- [https://github.com/BrainJS/brain.js/issues/245](https://github.com/BrainJS/brain.js/issues/245)
- [https://flows.nodered.org/flow/195773d3b493d81c9bf012f64da02ea3](https://flows.nodered.org/flow/195773d3b493d81c9bf012f64da02ea3)
