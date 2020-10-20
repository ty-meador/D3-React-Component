


# D3-React-Component ![logo]( https://assets.gitlab-static.net/uploads/-/system/project/avatar/21666192/D3R.png?width=64 )

[![Coverage Status](https://coveralls.io/repos/gitlab/Native-Coder/d3-react-component/badge.svg?branch=master)](https://coveralls.io/gitlab/Native-Coder/d3-react-component?branch=master) [![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/licenses/MIT) [![build status](https://gitlab.com/Native-Coder/d3-react-component/badges/master/pipeline.svg)](https://gitlab.com/Native-Coder/d3-react-component/-/pipelines) [![dependency count: 0](https://badgen.net/bundlephobia/dependency-count/d3-react-component?color=green)](https://www.npmjs.com/package/d3-react-component) [![minzipped size](https://badgen.net/bundlephobia/minzip/d3-react-component)](https://bundlephobia.com/result?p=d3-react-component) [![Gitlab](https://badgen.net/badge/icon/gitlab?icon=gitlab&label)](https://gitlab.com/Native-Coder/d3-react-component) [![npm package](https://badgen.net/badge/icon/npm?icon=npm&label)](https://www.npmjs.com/package/d3-react-component)

D3-React-Component provides a React component wrapper for **any** d3 visualization. The component is called "D3Canvas" and the entire package is only [1.9kb gzipped!](https://bundlephobia.com/result?p=d3-react-component)(Seriously, this README is bigger than the entire build at ~8kb)

Find the full respoitory on [![gitlab](https://badgen.net/badge/icon/gitlab?icon=gitlab&label)]


# Design Philosophies
- [x] Should provide a single component that will work with any d3 visualization
- [x] Should provide a component-compatible parent class that can be extended for simplicity
- [x] Should have a dead-simple API that contains only 3 methods: Create, Update, Destroy
- [x] Should work with d3 loaded in a script tag, just like if you weren't using react. It must not require any other d3/react libraries to work
- [x] Zero dependency

# Installation

### NPM

    npm install d3-react-component
### Manual
1. Copy `Chart`.js and `D3Canvas.js` from the `/src` folder of this repository into the `/src` folder of your react app.
2. profit

# Usage
1. Load d3 in a script tag. (add `<script src="https://d3js.org/d3.v6.min.js"></script>` to your index.html file)
2. Create a class for your chart (for simplicity, extend the provided `Chart` class)
3. Pass your custom class to the D3Canvas component

#### Notes:
	- Class must have a create method
		- Use this method to store chart-specific functions, variables, etc
   	 - Class must have an update method
	   	 - Use this method to render the chart, update scale domains, perform enter/exit/update etc
#### Reasons to extend the provided Chart class (It is highly recommended that you do so)
	 - Margin math is done for you and is accessible from your subclass with `this.margin`
	 - Default margins are setup for you (20px default)
	 - A reference to the window.d3 object is mapped to `this.d3` for your subclass
# Examples

### Step1: Create a class for your chart (for simplicity, extend the provided `Chart` class):
The first step is to create a class for your chart/visualization. The class that you create is required to have 2 methods, with an optional third method. Each of these methods directly maps to a phase in Reacts DOM lifecycle.

 - Create: This method is called after the canvas node has been mounted to the DOM
	 - Use it to create the `SVG` element on the page, create your scales, and init any other d3 functions
 - Update: This method is called any time a new `data` prop is passed to the D3Canvas component
	 - Use it to render your chart, performe enter/update/exit transitions, update your scale domains, etc
 - Destroy (optional method): This method is called before React unmounts the canvas node from the DOM
	 - Use it if you need to clean up after yourself.

Here is a very basic example of what it looks like when you extend the provided `Chart` class. We'll call our chart "MyAwesomeChart" and we'll save it in its own file
#### "MyAwesomeChart.js"
```javascript

import {Chart} from "d3-react-component" // If you used NPM
//import Chart from "./Chart.js" // If you performed a manual install

export default class MyAwesomeChart extends Chart {
	/**
	 * Create and store the necessary functions for your vis
	 * @param canvasNode - The DOM node that will contain your vis. Append your svg element to this!
	 * @param data - The data that was initially passed to the vis
	 * @param props - any properties that you passed to the D3Canvas component. (Except margins, we'll discuss that later...)
	 */
	create( canvasNode, data, props ){
		// Create the SVG element
		this.svg = this.d3.select( canvasNode ) // Access the d3 object with this.d3
		  .append( "svg" )
			.attr( "width", this.totalWidth ) // this.totalWidth provided by Chart.js
			.attr( "height", this.totalHeight ) // this.totalHeight provided by Chart.js
		  .append( "g" )
			.attr(
				"transform",
				`translate( ${this.margin.left},${this.margin.top} )` // This.margin object provided by Chart.js
			);
		// This is where you create and store your scales, axis formats, path generators, etc
	}

	/**
	 * Anytime the D3Canvas component is updated with new data, this
	 * function is called. use it to update/transition your vis
	 * @param data - The new data
	 */
	update( data ){
		// Code to update the chart
		// This is where you update your scale domains, perform enter/exit/update etc
	}
}
```
* Notice that we don't need any margin logic. Our parent class takes care of that for us by following the conventions laid out [here](https://observablehq.com/@d3/margin-convention)
* Notice that you automagically get a reference to the d3 object. Use ```this.d3```. ( again, the parent class takes care of this for us )
* If you chose not to extend the provided `Chart` class, you will have to get a reference to the d3 object yourself (use `window.d3`). You will also have to perform any margin mathematics yourself. see [here](https://observablehq.com/@d3/margin-convention)

### Step 2: Pass your custom class to the D3Canvas component
Next, you'll need to import the D3Canvas component into your React App, and pass the Class that you just wrote to it.
#### "App.js"
```jsx
	import {D3Canvas} from "d3-react-component"
	import MyAwesomeChart from "./MyAwesomeChart.js"
	function App(){
		return(
			<D3Canvas
				id={"some_unique_id"}
				chart={MyAwesomeChart}
				width={1000}
				height={500}
				leftMargin={50}
				rightMargin={20}
				topMargin={35}
				bottomMargin={20}
			/>
		);
	}

```
* Notice that we passed the *class* to the component. We did NOT instantiate it!
* Note: All margins default to 0px
* Note: Both width and height default to 500px

## Custom properties
any custom properties that you pass to the D3Canvas component will be passed to the ```create``` method as the third parameter ( props )

### App.js
```jsx
	import {D3Canvas} from "d3-react-component"
	import MyAwesomeChart from "./MyAwesomeChart.js"
	function App(){
		return(
			<D3Canvas
				id={"some_unique_id"}
				chart={MyAwesomeChart}
				...
				myCustomProperty={{
					foo: "bar"
				}}
			/>
		 );
	}

```

### MyAwesomeChart.js
```javascript
import {Chart} from "d3-react-component"
export default class MyAwesomeChart extends Chart {

	create( canvasNode, data, props ){
		console.log( props )
		// Outputs { myCustomProperty : { foo: "bar" } }
		...
	}

	update( data ){...}
}
```

## Accessing Margins, Width, and Height
### Margins
You can access the margins in your chart class with ```this.margin```. For instance, if you want to get the left margin, just use ```this.margin.left```.
### Width and Height ( minus margins )
You can access the computed width and height with ```this.wdith``` and ```this.height``` respectively. ( these values are minus margins )
### Total Width and Total Height ( width and height including margins )
If you would like to access the *total width* or *total height* you can use ```this.totalWidth``` and ```this.totalHeight``` respectively. ( these values include the margins )



# Refactoring An existing chart

If you have existing D3 charts, they can be quite easily refactored to work with the D3Canvas Component. I have adapted this [chart](https://bl.ocks.org/jchakko/d6f56f8ddc5aaea786273e6cda0424fb) to a [Chart class](https://gitlab.com/Native-Coder/d3-react-component/-/blob/master/examples/LineChart.js) so that you can compare them. (You will find it along with its data file in the "examples" folder of the [repository](https://gitlab.com/Native-Coder/d3-react-component))

To run the example:

1. Start by placing data.csv into the /public folder
1. Next, Place D3Canvas, Chart.js, and LineChart.js in the /src folder
1. Edit App.js as to look as follows

```jsx
import React from 'react';
import './App.css';
import {D3Canvas} from "d3-react-component"
import LineChart from "./LineChart.js"
function App() {
	return (
		<D3Canvas
			id={"LineChart"}
			chart={LineChart}
			data={"data.csv"}
			leftMargin={50}
			width={1000}
			height={500}
		  />
	);
}

export default App;

```
