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

Pontos que caem em entrevista:

- `className` em vez de `class`, `htmlFor` em vez de `for` (Sao palavras reservadas em JS).
- {} injeta qualquer expressao JS. Nao aceita `if`/`for` (sao statements), por isso se usa operador ternario e `.map()`.
- Todo componente precisa retornar um unico no raiz, ou usar Fragment `<>...</>`.

## 1.3 Props e Estado

- Props: dados que descem do pai. Imutaveis dentro do filho.
- State: dados internos do componente que, quando mudam, disparam re-rder.

Dados em React fluem so para baixo (one-way). Para o filho avisar o pai, o pai passa uma funcao como prop e o filho a chama

```ts
function Pai() {
  const [nome, setNome] = useState('');
  return <Filho onMudar={setNome} />; // passa o "emit" como prop
}
function Filho({ onMudar }) {
  return <input onChange={e => onMudar(e.target.value)} />;
}
```

## 1.4 Hooks: O Coracao do React Moderno

Hooks sao funcoes que "engancham" o componente no sistema do estado/ciclo de vida do React. Duas regras inviolaveis (e por que existem):

1. So chame hooks no topo do componente, nunca dentro de `if`, loop ou funcao aninhada.

2. So chame hooks dentro do componente React ou de outros hooks.

O porque: React identifica cada hook pela ordem de chamada, nao por nome. Se voce condicionar um useState, a ordem muda entre renders e o React passa o estado errado para o hook errado.

*useState*

```ts
const [valor, setValor] = useState(inicial);
```

- O argumento eh o valor inicial (usado so no primeiro render).
- `setValor` aceita um valor ou uma funcao updater `setValor(prev => prev + 1)`. Use a forma de funcao quando o novo estado depende do anterior, evita bugs com closures antigas e com updates em lote (batching).

*useEffect*

Roda depois que o DOM foi pintado. Serve para efeitos colaterais: fetch, subscriptions, timers, sincronziacao com sistemas externos.

```ts
useEffect(() => {
  const id = setInverval(tick, 1000);
  return () => clearInterval(id); // cleanup
}, [dependencias]);
```

Tres comportamentos conforme o array de dependencias:

- Sem array: roda depois de todo render.
- Array vazio []: roda uma vez (apos o mount). Cleanup no unmount.
- Array com deps [a, b]: roda quando alguma dep muda (comparacao por referencia/`Object.is`).

A funcao de cleanup roda antes do proximo efeito e no unmount. Eh o que evita memory leaks (listeners pendurados, timers, requests).

- Comparando com Vue: `useEffect` cobre o territorio de `mounted` + `watch` + `unmounted` ao mesmo tempo. Mas cuidado `useEffect` nao eh "watcher reativo automatico", voce declara manualmente as dependencias

- Armadilha classica: Esquecer uma dependencia (`a` eh usado dentro mas nao esta no array) gera stale closure, o efeito enxerga um valor congelado. O ESLint `exhaustive-deps` existe para pegar isso.

*useMemo e useCallback*

Ambos memoizam para evitar tabalho/re-render desnecessario.

- `useMemo(() => calcPesado(a), [a])`, memoriza um valor computado.
- `useCallback(fn, [a])`, memoriza uma funcao (eh useMemo(() => fn, deps)) por baixo.

Por que `useCallback` importa: em JS, () => {} cria uma funcao nova a cada render. Se voce passa essa funcao para um filho itmizado com React.memo, ele re-renderiza mesmo sem necessidade, porque a prop "mudou" (referencia nova). `useCallback` mantem a mesma referencia.
  Nao saia memoizando tudo. Memoizacao tem cuso (memoria + comparacao). Use quando ha calculo pesado ou quando a referencia estavel eh necessaria para evitar re-renders em cascata. Saber quando nao usar impressiona mais que decorar a sintaxe.


*useRef*

- Uma "caixa" mutavel (ref.current) que persite entre render sem disparar re-render.

Dois usos:

1. Acessar nos do DOM `<input ref={inputRef} />` -> `inputRef.current.focus()`
2. Guardar valor mutavel que nao deve causar render (ex: id de timer, valor anterior).

  State eh para dados que, ao mudar devem repintar a tela. Ref eh para dados que precisam sobreviver entre renders mas nao tem efeito visual.

*useContext*

Evita prop drilling (passar props por muitos niveis).

```ts
const Tema = createContext('claro');
// provedor: <Tema.Provider value="escuro">...<Tema.Provider/>
// consumidor: const tema = useContext(Tema);
```

*useReducer*

Para estado complexo com varias transicoes. Eh o `Redux` em miniatura, local ao componente.

```tsx
const [state, dispatch] = useReducer(reducer, estadoInicial);
dispatch({ type: 'incrementar' });
```

prefira a `useState` quando as proximas atualizacoes dependem da logica/estado anterior complexos, ou quando ha muitos sub-valores relacionados.

*Custom hooks*

Funcao que comeca com use e combina outros hooks para reutilizar logica (nao UI). Eh o substituto moderno dos mixins/composables.

```tsx
function useLargura() {
  const [w, setW] = useState(window.innerWidth);

  useEffect(() => {
    const h = () => setW(window.innerWidth);
    window.addEventListener('resize', h);
    return () => window.removeEventListener('resize', h);
  }, []);

  return w;
}
```