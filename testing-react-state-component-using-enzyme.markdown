---
title: Testing React state component using enzyme
date: 2020-02-21 10:56:00 Z
permalink: "/testing-react-state-component-using-enzyme"
layout: post
---

# Introduction
React components can be functional or stateful. Unit testing the state of the React component is very important for testing the business logic. Enzyme provides us with methods to access the state of the component. Let us see how to test a react component state using enzyme

# Sample React component

Please find a test component below

```
import React, { Component } from "react";
class Test extends Component {
  constructor() {
    super();
    this.state = {
      index: 0
    };
    this.incrementIndex = this.incrementIndex.bind(this);
  }
  incrementIndex() {
    this.setState({
      index: this.state.index + 1
    });
  }
  render() {
    return (
      <div>
        <span id="index" data-testid="normal-index">
          {this.state.index}
        </span>
        <button onClick={this.incrementIndex}>Increment Index</button>
      </div>
    );
  }
}
export default Normal;
```

# Explanation

In the above component, we maintain the state of component "index". On click of the button, we update the state variable "index". We display this state variable in the UI.

# Testing using enzyme

Enzyme provides following methods to access the state 

Access the state of the component

```
const wrapper = shallow(<Test/>);
wrapper.state('index') // get state variable index
wrapper.state().index // get state variable index
```

# Test case

```
import React from "react";
import { shallow } from "enzyme";
import _ from "lodash";
import Test from "../Test";
describe("Normal Tests", () => {
  it("should initialize index to 0", () => {
    // check for initial state
    const app = shallow(<Test />);
    expect(app.state("index")).toEqual(0);
  });
  it("should call increment index on click of button", () => {
    const app = shallow(<Test />);
    const spy = jest.spyOn(app.instance(), "incrementIndex");
    app.instance().forceUpdate();
    // force click and check for update
    app.find("button").simulate("click");
    expect(spy).toHaveBeenCalled();
  });
  it("should increment index - function test", () => {
    const app = shallow(<Test />);
    expect(app.state("index")).toEqual(0);
    app.instance().incrementIndex();
    expect(app.state("index")).toEqual(1);
  });
  it("should increment index - button test", () => {
    const app = shallow(<Test />);
    expect(app.state("index")).toEqual(0);
    app.find("button").simulate("click");
    expect(app.state("index")).toEqual(1);
  });
  it("it should display incremented index on click of button", () => {
    const app = shallow(<Test />);
    app.find("button").simulate("click");
    expect(app.find("span#index").text()).toEqual("1");
  });
});
```

