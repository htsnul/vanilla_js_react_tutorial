# Vannila JSでReactのチュートリアルを再現

## 概要

[Reactチュートリアル](https://reactjs.org/tutorial/tutorial.html)
のマルバツゲームを、Web標準のみ外部ライブラリなしのVanilla JSで再現する。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/136875/23da50b7-d941-4a01-7e38-8655d62eacab.png)

実際に動いているものはこちら。
https://htsnul.github.io/vanilla_js_react_tutorial/

1つずつ見ていこう。

## Square

Reactでの、

```css
.square {
  background: #fff;
  border: 1px solid #999;
  float: left;
  font-size: 24px;
  font-weight: bold;
  line-height: 34px;
  height: 34px;
  margin-right: -1px;
  margin-top: -1px;
  padding: 0;
  text-align: center;
  width: 34px;
}
```

と、

```jsx
function Square(props) {
  return (
    <button className="square" onClick={props.onClick}>
      {props.value}
    </button>
  );
}
```

を、Vanilla JSでは、

```js
class SquareComponent extends HTMLElement {
  constructor() {
    super();
    this.root = this.attachShadow({ mode: "closed" });
    this.root.innerHTML = `
      <style>
        button {
          background: #fff;
          border: 1px solid #999;
          float: left;
          font-size: 24px;
          font-weight: bold;
          line-height: 34px;
          height: 34px;
          margin-right: -1px;
          margin-top: -1px;
          padding: 0;
          text-align: center;
          width: 34px;
        }
      </style>
      <button></button>
    `;
  }

  setProps(props) {
    this.props = props;
    this.update();
  }

  update() {
    this.root.querySelector("button").innerHTML = this.props.value || "";
    this.root.querySelector("button").onclick = this.props.onClick;
  }
}

customElements.define("square-component", SquareComponent);
```

とした。

基本的に要素をWeb Componentsを使って実装していく。

Web Componentsについては、
[Web Components | MDN](https://developer.mozilla.org/ja/docs/Web/Web_Components)
が分かりやすい。

最初の比較がReactの関数コンポーネントとの比較になってしまうので、
Vanilla JSではちょっと長くなってしまうように感じられるが、
後に出てくるクラスコンポーネントとの比較ではそこまで長くならない。

コンストラクタで雛形を作成し、
`setProps` （後には `setState` も）が呼ばれたら、`update` によって、
手動での差分更新を行う。

Shadow DOMにより、スタイルのスコープ化ができている。
そのため、これぐらいの規模のコンポーネントであれば、
CSSクラス名を使わずとも気にせずタグでスタイルが指定できる。

今後もだいたい同じ流れでVanilla JSにしていく。

`props` について、Web Componentの属性値で受け渡す手もあるのだが、
Web Componentの属性値は文字列しか扱えないため、
コールバックなどを扱うことを考えると、`setProps` など、
別関数に分けて別途呼び出すのが良いように思えた。

また、カスタム要素の名前を`square-component` と `component` を付加しているが、
これは、カスタム要素は、名前にハイフンが含まれる必要があるためである。

## Board

Reactでの、

```css
.board-row:after {
  clear: both;
  content: "";
  display: table;
}
```

と、

```jsx
class Board extends React.Component {
  renderSquare(i) {
    return (
      <Square
        value={this.props.squares[i]}
        onClick={() => this.props.onClick(i)}
      />
    );
  }

  render() {
    return (
      <div>
        <div className="board-row">
          {this.renderSquare(0)}
          {this.renderSquare(1)}
          {this.renderSquare(2)}
        </div>
        <div className="board-row">
          {this.renderSquare(3)}
          {this.renderSquare(4)}
          {this.renderSquare(5)}
        </div>
        <div className="board-row">
          {this.renderSquare(6)}
          {this.renderSquare(7)}
          {this.renderSquare(8)}
        </div>
      </div>
    );
  }
}
```

を、Vanilla JSでは、

```js
class BoardComponent extends HTMLElement {
  constructor() {
    super();
    this.root = this.attachShadow({ mode: "closed" });
    this.root.innerHTML = `
      <style>
        .board-row:after {
          clear: both;
          content: "";
          display: table;
        }
      </style>
      <div>
        <div class="board-row">
          <square-component></square-component>
          <square-component></square-component>
          <square-component></square-component>
        </div>
        <div class="board-row">
          <square-component></square-component>
          <square-component></square-component>
          <square-component></square-component>
        </div>
        <div class="board-row">
          <square-component></square-component>
          <square-component></square-component>
          <square-component></square-component>
        </div>
      </div>
    `;
  }

  setProps(props) {
    this.props = props;
    this.update();
  }

  update() {
    this.root.querySelectorAll("square-component").forEach((elm, i) => {
      elm.setProps({
        value: this.props.squares[i],
        onClick: () => this.props.onClick(i)
      });
    });
  }
}

customElements.define("board-component", BoardComponent);

```

とした。

今回はクラスコンポーネントなので比較しやすい。

Reactでは `render` で行っている処理を、
Vanilla JSではコンストラクタでの雛形作成と、`update` での手動での差分更新に分離している。

## Game

Reactでの、

```css
.game {
  display: flex;
  flex-direction: row;
}

.game-info {
  margin-left: 20px;
}
```

と、

```jsx
class Game extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      history: [
        {
          squares: Array(9).fill(null)
        }
      ],
      stepNumber: 0,
      xIsNext: true
    };
  }

  handleClick(i) {
    const history = this.state.history.slice(0, this.state.stepNumber + 1);
    const current = history[history.length - 1];
    const squares = current.squares.slice();
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    squares[i] = this.state.xIsNext ? "X" : "O";
    this.setState({
      history: history.concat([
        {
          squares: squares
        }
      ]),
      stepNumber: history.length,
      xIsNext: !this.state.xIsNext
    });
  }

  jumpTo(step) {
    this.setState({
      stepNumber: step,
      xIsNext: (step % 2) === 0
    });
  }

  render() {
    const history = this.state.history;
    const current = history[this.state.stepNumber];
    const winner = calculateWinner(current.squares);

    const moves = history.map((step, move) => {
      const desc = move ?
        'Go to move #' + move :
        'Go to game start';
      return (
        <li key={move}>
          <button onClick={() => this.jumpTo(move)}>{desc}</button>
        </li>
      );
    });

    let status;
    if (winner) {
      status = "Winner: " + winner;
    } else {
      status = "Next player: " + (this.state.xIsNext ? "X" : "O");
    }

    return (
      <div className="game">
        <div className="game-board">
          <Board
            squares={current.squares}
            onClick={i => this.handleClick(i)}
          />
        </div>
        <div className="game-info">
          <div>{status}</div>
          <ol>{moves}</ol>
        </div>
      </div>
    );
  }
}
```

を、Vanilla JSでは、

```js
class GameComponent extends HTMLElement {
  constructor() {
    super();
    this.state = {
      history: [
        {
          squares: Array(9).fill(null)
        }
      ],
      stepNumber: 0,
      xIsNext: true
    };
    this.root = this.attachShadow({ mode: "closed" });
    this.root.innerHTML = `
      <style>
        :host > div {
          display: flex;
          flex-direction: row;
        }

        .game-info {
          margin-left: 20px;
        }
      </style>
      <div>
        <div>
          <board-component></board-component>
        </div>
        <div class="game-info">
          <div></div>
          <ol></ol>
        </div>
      </div>
    `;
    this.update();
  }

  setState(state) {
    this.state = { ...this.state, ...state };
    this.update();
  }

  handleClick(i) {
    const history = this.state.history.slice(0, this.state.stepNumber + 1);
    const current = history[history.length - 1];
    const squares = current.squares.slice();
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    squares[i] = this.state.xIsNext ? "X" : "O";
    this.setState({
      history: history.concat([
        {
          squares: squares
        }
      ]),
      stepNumber: history.length,
      xIsNext: !this.state.xIsNext
    });
  }

  jumpTo(step) {
    this.setState({
      stepNumber: step,
      xIsNext: (step % 2) === 0
    });
  }

  update() {
    const history = this.state.history;
    const current = history[this.state.stepNumber];
    const winner = calculateWinner(current.squares);

    const moves = history.map((step, move) => {
      const desc = move ?
        'Go to move #' + move :
        'Go to game start';
      const fragment = document.createRange().createContextualFragment(`
        <li>
          <button>${desc}</button>
        </li>
      `);
      fragment.querySelector("button").onclick = () => {
        this.jumpTo(move);
      }
      return fragment;
    });

    let status;
    if (winner) {
      status = "Winner: " + winner;
    } else {
      status = "Next player: " + (this.state.xIsNext ? "X" : "O");
    }
    this.root.querySelector("board-component").setProps({
      squares: current.squares,
      onClick: (i) => this.handleClick(i)
    });
    this.root.querySelector(".game-info > div").innerHTML = status;
    this.root.querySelector(".game-info > ol").replaceChildren(...moves);
  }
}

customElements.define("game-component", GameComponent);
```

とした。

`handleClick` や `jumpTo` は差がない。
コンストラクタや、`render` の大部分も同じである。
これらも、雛形を作って、手動での差分更新をするという部分が変更点になっている。

`moves`については、`DocumentFragment` を生成して配列にして、
それを、`replaceChildren` で一気に入れ替えしている。

毎回DOMを上書きしてしまっているのでReactに比べると効率が悪くなってしまっているはずだ。
このあたりは、手動での差分更新を最小にするコストとの記述の簡潔さとのバランスになってきそうだ。

## その他部分

その他の部分は、

```js
function calculateWinner(squares) {
  // ...
}
```
があるが、これも完全に同じで変更なしで動く。

## 全体像

Web ComponentsによりCSSは各コンポーネントで指定できるので、
HTMLとJSのみで動作する。

index.htmlは、

```html
<!DOCTYPE html>
<script type="module" src="index.js"></script>
<game-component></game-component>
```

とだけ記載しておけばよい。

JSはここまでのを全体を載せると、

```js
class SquareComponent extends HTMLElement {
  constructor() {
    super();
    this.root = this.attachShadow({ mode: "closed" });
    this.root.innerHTML = `
      <style>
        button {
          background: #fff;
          border: 1px solid #999;
          float: left;
          font-size: 24px;
          font-weight: bold;
          line-height: 34px;
          height: 34px;
          margin-right: -1px;
          margin-top: -1px;
          padding: 0;
          text-align: center;
          width: 34px;
        }
      </style>
      <button></button>
    `;
  }

  setProps(props) {
    this.props = props;
    this.update();
  }

  update() {
    this.root.querySelector("button").innerHTML = this.props.value || "";
    this.root.querySelector("button").onclick = this.props.onClick;
  }
}

customElements.define("square-component", SquareComponent);

class BoardComponent extends HTMLElement {
  constructor() {
    super();
    this.root = this.attachShadow({ mode: "closed" });
    this.root.innerHTML = `
      <style>
        .board-row:after {
          clear: both;
          content: "";
          display: table;
        }
      </style>
      <div>
        <div class="board-row">
          <square-component></square-component>
          <square-component></square-component>
          <square-component></square-component>
        </div>
        <div class="board-row">
          <square-component></square-component>
          <square-component></square-component>
          <square-component></square-component>
        </div>
        <div class="board-row">
          <square-component></square-component>
          <square-component></square-component>
          <square-component></square-component>
        </div>
      </div>
    `;
  }

  setProps(props) {
    this.props = props;
    this.update();
  }

  update() {
    this.root.querySelectorAll("square-component").forEach((elm, i) => {
      elm.setProps({
        value: this.props.squares[i],
        onClick: () => this.props.onClick(i)
      });
    });
  }
}

customElements.define("board-component", BoardComponent);

class GameComponent extends HTMLElement {
  constructor() {
    super();
    this.state = {
      history: [
        {
          squares: Array(9).fill(null)
        }
      ],
      stepNumber: 0,
      xIsNext: true
    };
    this.root = this.attachShadow({ mode: "closed" });
    this.root.innerHTML = `
      <style>
        :host > div {
          display: flex;
          flex-direction: row;
        }

        .game-info {
          margin-left: 20px;
        }
      </style>
      <div>
        <div>
          <board-component></board-component>
        </div>
        <div class="game-info">
          <div></div>
          <ol></ol>
        </div>
      </div>
    `;
    this.update();
  }

  setState(state) {
    this.state = { ...this.state, ...state };
    this.update();
  }

  handleClick(i) {
    const history = this.state.history.slice(0, this.state.stepNumber + 1);
    const current = history[history.length - 1];
    const squares = current.squares.slice();
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    squares[i] = this.state.xIsNext ? "X" : "O";
    this.setState({
      history: history.concat([
        {
          squares: squares
        }
      ]),
      stepNumber: history.length,
      xIsNext: !this.state.xIsNext
    });
  }

  jumpTo(step) {
    this.setState({
      stepNumber: step,
      xIsNext: (step % 2) === 0
    });
  }

  update() {
    const history = this.state.history;
    const current = history[this.state.stepNumber];
    const winner = calculateWinner(current.squares);

    const moves = history.map((step, move) => {
      const desc = move ?
        'Go to move #' + move :
        'Go to game start';
      const fragment = document.createRange().createContextualFragment(`
        <li>
          <button>${desc}</button>
        </li>
      `);
      fragment.querySelector("button").onclick = () => {
        this.jumpTo(move);
      }
      return fragment;
    });

    let status;
    if (winner) {
      status = "Winner: " + winner;
    } else {
      status = "Next player: " + (this.state.xIsNext ? "X" : "O");
    }
    this.root.querySelector("board-component").setProps({
      squares: current.squares,
      onClick: (i) => this.handleClick(i)
    });
    this.root.querySelector(".game-info > div").innerHTML = status;
    this.root.querySelector(".game-info > ol").replaceChildren(...moves);
  }
}

customElements.define("game-component", GameComponent);

function calculateWinner(squares) {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6]
  ];
  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }
  return null;
}
```

となる。

実際に動いているものはここで確認可能だ。
https://htsnul.github.io/vanilla_js_react_tutorial/


## まとめ

Reactの見どころはDOMの差分適用が大きいが、
それ以外にも、コンポーネント化や、単方向データフローなどもある。

今回は、Vanilla JSで手動での差分更新は行っているが、
コンポーネント化や、単方向データフローは実現できている。
本チュートリアルの見どころのタイムトラベルなども実現できている。

手動での差分更新が許容できるならば、このような作り方もあり得るかもしれない。

