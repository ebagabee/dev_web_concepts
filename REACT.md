# React

- UI declarativa baseada em estado.
- React te obrigado a ser explicito sobre o que muda.

## 1.1 O Modelo mental: UI = f(state)

A ideia central é que a UI é uma função pura do estado. Voce nao manipula o DOM; voce descreve como a tela deveria ser para um dado estado e o React calculado a diferenca e aplica no DOM real.

- Em React, voce chama `setCount(novoValor)`. Isso agenda um re-render: a funcao do componente roda de novo inteira, produz uma nova arvore de elementos, e o React reconcilia.

_Como Explicar_: "O Componente React eh uma funcao que recebe estado e devolve a descricao da tela. Mudou o estado, a funcao roda de novo. O React compara o resultado novo com o antigo (Virtual DOM) e so toca no DOM o que realmente mudou."

## 1.2 JSX

JSX eh acucar sintatica que vira React.createElement(...). Nao eh HTML - Eh JavaScript

```js
const el = <h1 className="title">Ola {nome}</h1>;
// Vira: React.createElement('h1', {className: 'title'}, 'Ola ', nome)
```
