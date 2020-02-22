---
title: Testing async code in React using enzyme
date: 2020-02-21 14:35:00 Z
categories:
- enzyme
tags:
- enzyme
---

## Introduction

Javascript is synchronous. The scripts are run in order. However, we have async code using promises, observables, or timeout/interval functions. 

Usually, we have async code in React apps when we make an API call, or we use javascript APIs like `setTimeout` or `setInterval`.

## Types of async code

As mentioned above, the following are usually found async code in React apps.

### Http call using [axios library](https://github.com/axios/axios)
```
// Make a request for a user with a given ID
axios.get('/user?ID=12345')
  .then(function (response) {
    // handle success
    console.log(response);
  })
  .catch(function (error) {
    // handle error
    console.log(error);
  })
  .then(function () {
    // always executed
  });
```

```
setTimeout(() => {
  console.log('settimeout');
}, 1000);
```

## Testing the timeout and interval functions

Please find the code below for timeout and interval functions.

```
import React, { Component } from "react";
class Timers extends Component {
  constructor() {
    super();
    this.state = {
      index: 0
    };
    this.setTimeoutFn = this.setTimeoutFn.bind(this);
    this.setIntervalFn = this.setIntervalFn.bind(this);
  }
  setTimeoutFn() {
    setTimeout(() => {
      this.setState({
        index: 1
      });
    }, 1000);
  }
  setIntervalFn() {
    setInterval(() => {
      this.setState({
        index: this.state.index + 1
      });
    }, 1000);
  }
  render() {
    return <div></div>;
  }
}
export default Timers;
```

Unit test cases for the above timer code

```
import React from "react";
import { shallow } from "enzyme";
import Timers from "../Timers";
describe("settimeout fn", () => {
  it("should increment index by 1 after 1 second (using advanceTimersByTime)", () => {
    const component = shallow(<Timers />);
    jest.useFakeTimers();
    expect(component.state("index")).toEqual(0);
    component.instance().setTimeoutFn();
    jest.advanceTimersByTime(1000);
    expect(component.state("index")).toEqual(1);
    jest.useRealTimers();
  });
  it("should increment index by 1 after 1 second (using runAllTimers)", () => {
    const component = shallow(<Timers />);
    jest.useFakeTimers();
    expect(component.state("index")).toEqual(0);
    component.instance().setTimeoutFn();
    jest.runAllTimers(); // you can also use run.runOnlyPendingTimers()
    expect(component.state("index")).toEqual(1);
    jest.useRealTimers();
  });
});
describe("setinterval fn", () => {
  it("should increment index by 1 after 1 second (using advanceTimersByTime)", () => {
    const component = shallow(<Timers />);
    jest.useFakeTimers();
    expect(component.state("index")).toEqual(0);
    component.instance().setIntervalFn();
    jest.advanceTimersByTime(1000);
    expect(component.state("index")).toEqual(1);
    jest.useRealTimers();
  });
  it("should increment index by 1 after 1 second (using runOnlyPendingTimers)", () => {
    const component = shallow(<Timers />);
    jest.useFakeTimers();
    expect(component.state("index")).toEqual(0);
    component.instance().setIntervalFn();
    jest.runOnlyPendingTimers(); // using runAllTimers will be error as setInterval never
    // finishes. So it will be infinite loop.
    expect(component.state("index")).toEqual(1);
    jest.useRealTimers();
  });
});
```

## Testing the Axios and fetch function

Please find the below React component using axios and fetch to make the http call.

```
import React, { Component } from "react";
import { map } from "lodash";
import axios from "axios";
class AsyncTests extends Component {
  constructor() {
    super();
    this.asyncFunction = this.asyncFunction.bind(this);
    this.axiosFn = this.axiosFn.bind(this);
    this.state = {
      data: []
    };
  }
  axiosFn() {
    axios({
      url: "http://google.com/somedata.json"
    })
      .then(response => {
        this.setState({
          data: response.data
        });
      })
      .catch(e => {
        console.log("error", e);
      });
  }
  asyncFunction() {
    fetch("http://google.com/somedata.json")
      .then(data => {
        this.setState({
          data: data
        });
      })
      .catch(error => {
        this.setState({
          data: error
        });
      });
  }
  render() {
    const { data } = this.state;
    return (
      <div className="App">
        <List data={data} />
        <button id="async" onClick={this.asyncFunction}>
          Async Function
        </button>
      </div>
    );
  }
}
export const List = props => {
  const { data } = props;
  return map(data, d => <span className="index">{d}</span>);
};
export default AsyncTests;
```

Please find the tests below for this component

```
import React from "react";
import ReactDOM from "react-dom";
import { shallow } from "enzyme";
import waitUntil from "async-wait-until";
import MockAdapter from "axios-mock-adapter";
import axios from "axios";
import _ from "lodash";
import AsyncTests from "../AsyncTests";
import { List } from "../AsyncTests";
describe("AsyncTests", () => {
  it("renders without crashing", () => {
    const div = document.createElement("div");
    ReactDOM.render(<AsyncTests />, div);
    ReactDOM.unmountComponentAtNode(div);
  });
  it("should handle async function - success", async done => {
    global.fetch = jest
      .fn()
      .mockImplementation(() => Promise.resolve({ name: "abc" }));
    const app = shallow(<AsyncTests />);
    app.setState({
      data: []
    });
    app.instance().asyncFunction();
    await waitUntil(() => {
      return !_.isEmpty(app.state("data"));
    });
    expect(app.state("data")).toEqual({ name: "abc" });
    done();
  });
  it("should handle async function - error", async done => {
    global.fetch = jest
      .fn()
      .mockImplementation(() =>
        Promise.reject({ error: "There was some error" })
      );
    const app = shallow(<AsyncTests />);
    app.setState({
      data: []
    });
    app.instance().asyncFunction();
    await waitUntil(() => {
      return !_.isEmpty(app.state("data"));
    });
    expect(app.state("data")).toEqual({ error: "There was some error" });
    done();
  });
  it("should handle async function - success wo async await", done => {
    global.fetch = jest
      .fn()
      .mockImplementation(() => Promise.resolve({ name: "abc" }));
    const app = shallow(<AsyncTests />);
    app.setState({
      data: []
    });
    app.instance().asyncFunction();
    waitUntil(() => {
      return !_.isEmpty(app.state("data"));
    }).then(() => {
      expect(app.state("data")).toEqual({ name: "abc" });
      done();
    });
  });
  it("should handle axios function - success", async done => {
    const mock = new MockAdapter(axios);
    mock.onGet(/.*/g).reply(200, {
      name: "abc"
    });
    const app = shallow(<AsyncTests />);
    app.setState({
      data: []
    });
    app.instance().axiosFn();
    await waitUntil(() => {
      return !_.isEmpty(app.state("data"));
    });
    expect(app.state("data")).toEqual({ name: "abc" });
    done();
  });
  it("should render the list", async done => {
    const mock = new MockAdapter(axios);
    mock.onGet(/.*/g).reply(200, [1, 2, 3]);
    const app = shallow(<AsyncTests />);
    app.setState({
      data: []
    });
    app.instance().axiosFn();
    await waitUntil(() => {
      return !_.isEmpty(app.state("data"));
    });
    expect(app.state("data")).toEqual([1, 2, 3]);
    app.instance().forceUpdate();
    expect(app.find(List)).toHaveLength(1);
    done();
  });
});
```