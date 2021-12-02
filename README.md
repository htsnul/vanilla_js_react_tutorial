# Vannila JSでReactのチュートリアルを再現

## 概要

[Reactチュートリアル](https://reactjs.org/tutorial/tutorial.html)
のマルバツゲームをVanilla JSで再現する。

1つずつ見ていこう。

## Square

Reactでの、

```
function Square(props) {
  return (
    <button className="square" onClick={props.onClick}>
      {props.value}
    </button>
  );
}
```

を、Vanilla JSでは、

```
class SquareComponent extends HTMLElement {
  constructor() {
    super();
    this.root = this.attachShadow({ mode: "closed" });
    this.root.innerHTML = `
      <button class="square">
      </button>
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

基本的に要素をWeb Componentを使って実装していく。

Web Componentについては、
[Web Components | MDN](https://developer.mozilla.org/ja/docs/Web/Web_Components)
が分かりやすい。

最初の比較がReactの関数コンポーネントとの比較になってしまうので、
Vanilla JSではちょっと長くなってしまうように感じられるが、
後に出てくるクラスコンポーネントとの比較ではそこまで長くならない。

コンストラクタで雛形を作成し、
`setProps` （後には `setState` も）が呼ばれたら、`update` によって、
手動での差分更新を行う。

今後もだいたい同じ流れでVanilla JSにしていく。

propsについて、Web Componentの属性値で受け渡す手もあるのだが、
Web Componentの属性値は文字列しか扱えないため、
コールバックなどを扱うことを考えると、`setProps` など、
別関数に分けて別途呼び出すのが良いように思えた。

また、カスタム要素の名前を`square-component` と `component` を付加しているが、
これは、カスタム要素は、名前にハイフンが含まれる必要があるためである。

## Board

Reactでの、

```
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

```
class BoardComponent extends HTMLElement {
  constructor() {
    super();
    this.root = this.attachShadow({ mode: "closed" });
    this.root.innerHTML = `
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
```

とした。

今回はクラスコンポーネントなので比較しやすい。

Reactでは `render` で行っている処理を、
Vanilla JSではコンストラクタでの雛形作成と、`update` での手動での差分更新に分離している。

## Game

Reactでの、

```
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

```
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
      <div class="game">
        <div class="game-board">
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
    this.root.querySelector(".game-info > ol").innerHTML = "";
    moves.forEach((move) => {
      this.root.querySelector(".game-info > ol").appendChild(move);
    });
  }
}

customElements.define("game-component", GameComponent);
```

とした。

`handleClick` や `jumpTo` は差がない。
コンストラクタや、`render` の大部分も同じである。
これらも、雛形を作って、手動での差分更新をするという部分が変更点になっている。

`moves`については、毎回DOMを上書きしてしまっているのでReactに比べると効率が悪くなってしまっているはずだ。
このあたりは、手動での差分更新を最小にするコストとの記述の簡潔さとのバランスになってきそうだ。

## その他部分

その他の部分は、

```
function calculateWinner(squares) {
  ...
}
```
があるが、これも完全に同じで変更なしで動く。

また、HTMLには基本的には、

```
<game-component></game-component>
```

とだけ記載しておけばよい。

また、実はCSSについては、Web Componentは、
[グローバルCSSが適用されない](https://stackoverflow.com/questions/35694328/how-to-use-global-css-styles-in-shadow-dom)
という動作がある。

そのため、手元で再現する際は、各カスタム要素での雛形代入部分で、

```
    this.root.innerHTML = `
      <style>
        @import url("index.css")
      </style>
```

というのを追加した。
ただし、通常はむしろカスタム要素内にCSSスコープが閉じているのはメリットになるはずだ。
なので、今回の説明部分ではあえて除外している。

## まとめ

ReactはDOMの差分適用というメリットが大きいが、
それ以外にも、単方向データフローなどのメリットもある。

今回は、Vanilla JSで手動での差分更新を行いつつも、
Reactの単方向データフローは維持している。
その結果、タイムトラベルなども実現できている。

手動での差分更新が許容できるならば、このような作り方もあり得るかもしれない。

