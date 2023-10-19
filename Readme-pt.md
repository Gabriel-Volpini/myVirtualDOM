
# Minha implementação de uma virtual DOM

## Descrição
A Virtual DOM é uma técnica fundamental usada em bibliotecas e frameworks JavaScript, como [React](https://react.dev/) e [Vue.js](https://vuejs.org/), para otimizar a atualização do **D**ocument **O**bject **M**odel (**DOM**) em aplicações web. Ela funciona criando uma representação em memória do **DOM** real, permitindo comparações eficientes para identificar mudanças e, em seguida, atualiza apenas as partes necessárias do **DOM**, resultando em um melhor desempenho e uma experiência de desenvolvimento mais simplificada, onde as atualizações da interface do usuário são declarativas e baseadas em dados.

## Sumário
- [Visão Geral](#vis%C3%A3o-geral)
- [Pré-requisitos](#pr%C3%A9-requisitos)
- [Instalação](#instala%C3%A7%C3%A3o)
- [Mão na massa](#mão-na-massa)

## Visão Geral
Neste projeto, você encontrará uma implementação básica de uma **Virtual DOM**. Esta implementação é uma forma simplificada do funcionamento dos conceitos por trás da Virtual **DOM** mas serve para pode entender melhor projetos mais complexos.

## Pré-requisitos
Para o desenvolvimento deste projeto foram utilizado os seguintes recursos:
- [NodeJs v20](https://nodejs.org/en)
- [Babel](https://babeljs.io/)

## Mão na massa
  ### Representando a virtual DOM
  Primeiramente precisamos armazenar nossa árvore de alguma forma na memória, e podemos fazer isso usando apenas `JS puro`.

  Suponha que temos a seguinte árvore:
  ```html
  <ul class=”myList”>
    <li>First</li>
    <li>Second</li>
</ul>
```
Podemos representa-la usando somente objetos em JavaScript:
```js
{
  type: ‘ul’, props: { ‘class’: ‘myList’ }, children: [
    { type: ‘li’, props: {}, children: [‘First’] },
    { type: ‘li’, props: {}, children: [‘Second’] }
  ]
}
```
Com isso conseguimos definir um formato padrão para representar um elemento.
```js 
{ type: ‘…’, props: { … }, children: [ … ] }
```
Para a representação dos conteúdos textuais, podemos usar strings

Porém para a escrita de árvores maiores e mais complexas fica trabalhoso escrever dessa forma, para isso podemos escrever a seguinte função para facilitar a geração da estrutura:
```js
function h(type, props, …children) {
  return { type, props, children };
}
```
E com isso podemos escrever nossa árvore da seguinte maneira:
```js
h(‘ul’, { ‘class’: ‘list’ },
  h(‘li’, {}, ‘item 1’),
  h(‘li’, {}, ‘item 2’),
);
```
A visualização ficou bem mais fácil, mas ainda é possível tornar o processo ainda bem mais simples utilizando a sintaxe [JSX](https://pt-br.legacy.reactjs.org/docs/introducing-jsx.html)
De acordo com a [documentação oficial do Babel JSX](https://babeljs.io/docs/babel-plugin-transform-react-jsx/) o que acontece é que ele faz a transpilação desse código:

```jsx
<ul className=”list”>  
	<li>First</li>  
	<li>Second</li>  
</ul>
```
Para o seguinte:
```js
React.createElement(‘ul’, { className: ‘list’ },  
	React.createElement(‘li’, {}, ‘item 1’),  
	React.createElement(‘li’, {}, ‘item 2’),  
);
```
Funcina exatamente da mesma forma que a nossa implementação `h(...)`, porém usando o `React.createElement(...)`, para poder fazer o babel utilizar a nossa função no lugar da função do React, podemos utilizar o chamado *jsx pragma*. Basta adicionar um comentário no inicio do arquivo da seguinte forma:
```jsx
/** @jsx h */

<ul className=”list”>  
  <li>First</li>  
  <li>Second</li>  
</ul>
```
Basicamente o que acontece é que duranteo o momento de transpilação, o que acontece é que avisamos o babel para utilizar a nossa função `h(...)` no lugar da `React.createElement()`

Então sumarizando, podemos escrever nossa **DOM** da seguinte maneira:
```jsx
/** @jsx h */

const myTree = (  
  <ul className=”list”>  
    <li>First</li>  
    <li>Second</li>  
  </ul>  
);
```
E o resultado gerado pelo Babel será:
```js
const myTree = (  
	h(‘ul’, { className: ‘list’ },  
		h(‘li’, {}, ‘First’),  
		h(‘li’, {}, ‘Second’),  
	);  
);
```
E quando a nossa função `h` for executada será retornando apenas `JS objects`, para a nossa representação da **virtual DOM**
```js
const myTree = (  
	{ type: ‘ul’, props: { className: ‘list’ }, children: [  
		{ type: ‘li’, props: {}, children: [‘First’] },  
		{ type: ‘li’, props: {}, children: [‘Second’] }  
	] }  
);
```
Clique [aqui](https://github.com/Gabriel-Volpini/myVirtualDOM/blob/main/examples/1/index.js) para ver um arquivo de exemplo.

### Aplicando nossa representação na DOM

Com a representação da **árvore virtual** pronta, precisamos criar uma **árvore DOM** real que reflita a nossa **árvore virtual**, ja que não podemos simplesmente colocar a nossa no lugar da real.

Para isso vamos definir algumas convenções e nomeclaturas:
 - Todas as variáveis ​​contendo nodes da DOM real vão começar com `$` - Exemplo *$parent* 
 - A representação da DOM virtual será chamada de *node*
 - Assim como no *React* só poderemos ter um node raiz, e todos os outros serão filhos deste

Com isso, vamos criar a nossa função *createElement(...)* que receberá um **node virtual** e retornará um **node real** .

```js
function createElement(node) {  
	return document.createElement(node.type);  
}
```
Como podemos receber tanto elementos normais quanto elementos de texto, precisamos fazer :a seguinte validação:
```js
function createElement(node) {  
	if (typeof node === ‘string’) {  
		return document.createTextNode(node);  
	}  
	return document.createElement(node.type);  
}
```
Agor precisamos levar em consideração os elementos filhos, para cada um deles também precisaremos criar outros nodes com a nossa função *createElement(...)* , e podemos fazer isso de forma recursiva chamando o *appendChild()* para cada filho da seguinte forma:

```js
function createElement(node) {  
	if (typeof node === ‘string’) {  
		return document.createTextNode(node);  
	}  
	const $el = document.createElement(node.type);  
	node.children  
		.map(createElement)  
		.forEach($el.appendChild.bind($el));  
		
	return $el;  
}
```
Clique [aqui](https://github.com/Gabriel-Volpini/myVirtualDOM/blob/main/examples/2/index.js) para um arquivo de exemplo.

###Lidando com mudanças
