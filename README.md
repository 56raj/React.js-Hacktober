# React.js Concepts

## What happens when you call setState?

The first thing React will do when setState is called is merge the object you passed into setState into the current state of the component. This will kick off a process called reconciliation. The end goal of reconciliation is to, in the most efficient way possible, update the UI based on this new state. To do this, React will construct a new tree of React elements (which you can think of as an object representation of your UI). Once it has this tree, in order to figure out how the UI should change in response to the new state, React will diff this new tree against the previous element tree. By doing this, React will then know the exact changes which occurred, and by knowing exactly what changes occurred, will able to minimize its footprint on the UI by only making updates where absolutely necessary.

## What’s the difference between an Element and a Component in React?

Simply put, a React element describes what you want to see on the screen. Not so simply put, a React element is an object representation of some UI.

A React component is a function or a class which optionally accepts input and returns a React element (typically via JSX which gets transpiled to a createElement invocation).

## What is the difference between a controlled component and an uncontrolled component?

A large part of React is this idea of having components control and manage their own state. What happens when we throw native HTML form elements (input, select, textarea, etc) into the mix? Should we have React be the “single source of truth” like we’re used to doing with React or should we allow that form data to live in the DOM like we’re used to typically doing with HTML form elements? These two questions are at the heart of controlled vs uncontrolled components.

A controlled component is a component where React is in control and is the single source of truth for the form data. As you can see below, username doesn’t live in the DOM but instead lives in our component state. Whenever we want to update username, we call setState as we’re used to.

```jsx
class ControlledForm extends Component {
  state = {
    username: ""
  };
  updateUsername = e => {
    this.setState({
      username: e.target.value
    });
  };
  handleSubmit = () => {};
  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <input
          type="text"
          value={this.state.username}
          onChange={this.updateUsername}
        />
        <button type="submit">Submit</button>
      </form>
    );
  }
}
```

An uncontrolled component is where your form data is handled by the DOM, instead of inside your React component.

You use refs to accomplish this.

```jsx
class UnControlledForm extends Component {
  handleSubmit = () => {
    console.log("Input Value: ", this.input.value);
  };
  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <input type="text" ref={input => (this.input = input)} />
        <button type="submit">Submit</button>
      </form>
    );
  }
}
```

Though uncontrolled components are typically easier to implement since you just grab the value from the DOM using refs, it’s typically recommended that you favor controlled components over uncontrolled components. The main reasons for this are that controlled components support instant field validation, allow you to conditionally disable/enable buttons, enforce input formats, and are more “the React way”.

## In which lifecycle event do you make AJAX requests and why?

AJAX requests should go in the componentDidMount lifecycle event.

There are a few reasons for this,

Fiber, the new React reconciliation algorithm, has the ability to start and stop rendering as needed for performance benefits. One of the trade-offs of this is that componentWillMount, the other lifecycle event where it might make sense to make an AJAX request, will be “non-deterministic”. What this means is that React may start calling componentWillMount at various times whenever it feels like it needs to. This would obviously be a bad formula for AJAX requests.

You can’t guarantee the AJAX request won’t resolve before the component mounts. If it did, that would mean that you’d be trying to setState on an unmounted component, which not only won’t work, but React will yell at you for. Doing AJAX in componentDidMount will guarantee that there’s a component to update.

## What does shouldComponentUpdate do and why is it important?

Above we talked about reconciliation and what React does when setState is called. What shouldComponentUpdate does is it’s a lifecycle method that allows us to opt out of this reconciliation process for certain components (and their child components). Why would we ever want to do this? As mentioned above, “The end goal of reconciliation is to, in the most efficient way possible, update the UI based on new state”. If we know that a certain section of our UI isn’t going to change, there’s no reason to have React go through the trouble of trying to figure out if it should. By returning false from shouldComponentUpdate, React will assume that the current component, and all its child components, will stay the same as they currently are.
