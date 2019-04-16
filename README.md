# react-amplitude-hooks

[![npm](https://img.shields.io/npm/v/react-amplitude-hooks.svg?style=flat-square)](https://www.npmjs.com/package/@amplitude/react-amplitude)

A React component library for easy product analytics instrumentation.

## Improvement from the @amplitude/react-amplitude library

This library is designed to be fully API compatible with the @amplitude/react-amplitude library, but taking advantage of the modern React features of hooks and context API.

- Hook support `const { instrument, logEvent } = useAmplitude();`
- TypeScript support

## Example: Instrumenting Tic-Tac-Toe (From Facebook's [Intro to React Tutorial](https://reactjs.org/tutorial/tutorial.html))

This is a hooks re-write of the example for demonstration purposes. (TODO_TODO)

Events logged:
 - start game
 - game won
 - click square
 - jump to move
 - go to game start

 Event properties:
 - scope (array; "square", "history", "game")
 - moves made (number)
 - winner ("X" or "O")
 - current player ("X" or "O")
 

```jsx
import React from "react";
import { render } from "react-dom";
import amplitude from "amplitude-js";
import {
  AmplitudeProvider,
  Amplitude,
  LogOnMount
} from "@amplitude/react-amplitude";

const AMPLITUDE_KEY = "";

function Square(props) {
  return (
    <Amplitude
      eventProperties={inheritedProps => ({
        ...inheritedProps,
        scope: [...inheritedProps.scope, "square"]
      })}
    >
      {({ instrument }) => (
        <button
          className="square"
          onClick={instrument("click square", props.onClick)}
        >
          {props.value}
        </button>
      )}
    </Amplitude>
  );
}

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
      xIsNext: step % 2 === 0
    });
  }

  render() {
    const history = this.state.history;
    const current = history[this.state.stepNumber];
    const winner = calculateWinner(current.squares);

    const moves = history.map((step, move) => {
      const desc = move ? "Go to move #" + move : "Go to game start";
      return (
        <li key={move}>
          <Amplitude
            eventProperties={inheritedProps => ({
              ...inheritedProps,
              scope: [...inheritedProps.scope, "move button"]
            })}
          >
            {({ logEvent }) => (
              <button
                onClick={() => {
                  logEvent(move ? "jump to move" : "go to game start");
                  this.jumpTo(move);
                }}
              >
                {desc}
              </button>
            )}
          </Amplitude>
        </li>
      );
    });

    let status;
    if (winner) {
      status = "Winner: " + winner;
    } else {
      status = "Current player: " + (this.state.xIsNext ? "X" : "O");
    }

    return (
      <AmplitudeProvider
        amplitudeInstance={amplitude.getInstance()}
        apiKey={AMPLITUDE_KEY}
      >
        <Amplitude
          eventProperties={{
            scope: ["game"],
            "moves made": this.state.step,
            "current player": this.state.xIsNext ? "X" : "O",
            winner
          }}
        >
          <LogOnMount eventType="start game" />
          {!!winner && <LogOnMount eventType="game won" />}
          <div className="game">
            <div className="game-board">
              <Board
                squares={current.squares}
                onClick={i => this.handleClick(i)}
              />
            </div>
            <Amplitude
              eventProperties={inheritedProps => ({
                ...inheritedProps,
                scope: [...inheritedProps.scope, "history"]
              })}
            >
              <div className="game-info">
                <div>{status}</div>
                <ol>{moves}</ol>
              </div>
            </Amplitude>
          </div>
        </Amplitude>
      </AmplitudeProvider>
    );
  }
}

// ========================================

render(<Game />, document.getElementById("root"));

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

## Installation

With npm:

```
npm install --save react-amplitude-hooks
```

With yarn:

```
yarn add react-amplitude-hooks
```

react-amplitude-hooks does not come with its own copy of the Amplitude JavaScript SDK. You can either install the Amplitude SDK via npm or with a JavaScript snippet.

## API Documentation

### `useAmplitude(eventProperties?: object, instanceName?: string)`

| Argument        | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| eventProperties | If an object is provided, the object will be merged with event properties higher-up in the component hierarchy and included in all events logged in this component or any components in its subtree.<br/>  If a function is provided, it will be called with a single parameter, `inheritedProperties`, that contains all of the event properties from components higher in the React component hierarchy. The return value from this function will be used for all logged events in this component or other components in its subtree. |
| instanceName    | The Amplitude instance to log events to.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |






### AmplitudeProvider 

Note: You must have an `<AmplitudeProvider>` in your React tree for logging to occur.


| Name              | Description                                                                                                                                                                                                                        |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| amplitudeInstance | A required prop. You should be providing an Amplitude instance returned from the Amplitude JS SDK's [`getInstance`](https://amplitude.zendesk.com/hc/en-us/articles/115002889587-JavaScript-SDK-Reference#amplitudeclient) method. |
| apiKey            | An optional prop that can be used to initialize the Amplitude instance with the provided key.                                                                                                                                      |
| userId            | An optional prop that can be used to attribute all events to a specific user.                                                                                                                                                      |

### Amplitude 

| Name             | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| children         | If can pass a function as the children prop, it will be called with a single object parameter, and the return value of that function will be used for rendering the actual React subtree.                                                                                                                                                                                                                                                                                                                                               |
| eventProperties  | If an object is provided, the object will be merged with event properties higher-up in the component hierarchy and included in all events logged in this component or any components in its subtree.<br/>  If a function is provided, it will be called with a single parameter, `inheritedProperties`, that contains all of the event properties from components higher in the React component hierarchy. The return value from this function will be used for all logged events in this component or other components in its subtree. |
| debounceInterval | If provided, events logged by the component will be debounced by this amount, in milliseconds.                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| userProperties   | An optional object that if provided, will trigger updates to the current user's "user properties."                                                                                                                                                                                                                                                                                                                                                                                                                                      |

#### Examples

The single object parameter has two useful fields: `logEvent` and `instrument`.

`logEvent` is a function that can be used to imperatively log events. All event properties from this component's `eventProperties` prop and any inherited properties will be included in these events.

Example:

```jsx
function Button(props) {
  return (
    <Amplitude>
      {({ logEvent }) =>
        <button
          onClick={() => {
            logEvent('button click');
            props.onClick();
          }}
        >
          {props.children}
        </button>
      }
    </Amplitude>
  )
}
```

`instrument` is a function that can be used to declaratively log events. If you have pre-existing event handlers, just wrap the functions with an `instrument`, and events will fire every time your normal event handlers are executed. `instrument` takes two parameters, the event type and the function to proxy.

`instrument` is also useful if you would like to prevent re-rendering via shouldComponentUpdate optimizations like PureComponent. It memoizes its arguments and returns the same function instance across re-renders. For this reason, it's not recommended to use `instrument` for functions that change on every render (i.e. inlined or "arrow" functions in `render`).

Example:

```jsx
function Button(props) {
  return (
    <Amplitude>
      {({ instrument }) =>
        <button
          onClick={instrument('button click', props.onClick)}
        >
          {props.children}
        </button>
      }
    </Amplitude>
  )
}
```


### LogOnMount 

| Name            | Description                                                                         |
| --------------- | ----------------------------------------------------------------------------------- |
| eventType       | When this component mounts, it will log an event with this value as the event type. |
| eventProperties | These properties will be applied to the event when the component mounts.            |
| instanceName    | The Amplitude instance to log events to.                                            |

### LogOnChange 

| Name            | Description                                                                                         |
| --------------- | --------------------------------------------------------------------------------------------------- |
| value           | Required prop, that when changes (diffing is done with shallow equality comparison), logs an event. |
| eventType       | When the `value` prop changes, it will log an event with this value as the event type.              |
| eventProperties | These properties will be applied to the event when the component mounts.                            |
| instanceName    | The Amplitude instance to log events to.                                                            |

## License

MIT

## Friends & Related Projects
+ [analytics-react](https://github.com/segmentio/analytics-react): Write analytics code once with Segment and collect customer data from any source and send it to over 250+ destinations.
