---
layout: post
title:      "React - functional vs class components"
date:       2019-03-19 03:47:32 +0000
permalink:  react_-_functional_vs_class_components
---


Functional components vs. class components. What's the difference?

### Functional components

Functional components are just simple javascript functions that return JSX.  These components typically don't have their own state and they receive data via an optional 'props' object as an argument. Here's an example:

```
import React from 'react'

const SomeButton = (props) => {
  return (
    <button>props.buttonText</button>
  )
}

export default SomeButton
```

### Class components

Class components are JavaScript classes that extend React's Component class. This gives you access to some extra features compared to functional components.

One of these features is internal state management via the setState() function. This function is used to update a user-defined 'state' instance object and then re-render the component if necessary.

Another feature class components offer is lifecycle methods. Lifecycle methods are called by React at certain points during a component's life cycle, allowing you to manipulate your component at these points. For example, there is a componentDidMount() lifecycle method that is called when a component first mounts. Or a componentWillUnmount() lifecycle method that is called right before a component unmounts.

Here's an example of a basic class component:

```
import React, { Component } from 'react'

class SomeList extends Component {

	state = {
		showList: true
	}

	handleToggleListClick = () => {
		this.setState({showList: !this.state.showList})
	}

	render () {

		const list = (
			<ul>
				<li>List item 1</li>
				<li>List item 2</li>
				<li>List item 3</li>
			</ul>
		)

		return (
			<div>
				{this.state.showList ? list : null}
				<button onClick={this.handleToggleListClick}>Toggle List</button>
			</div>
		)
	}
}

export default SomeList
```

