---
title: Declarative Data Visualizations in React
date: "2018-04-10"
---

Most of the time when we're designing data visualizations, we want them to be easy to use and reuse. When we design a particular visualization, we don't want to worry about _how_ that visualization is being created, we really just want to be able to plug data into it and get a nice visual out. For example, we'll design a program that takes a set of data, we'll _declare_ what shape we want it (scatterplot, boxplot, etc.) and it will figure out the transformations and render operations that need to take place for that chart to show up. While we're using our API, we won't need to worry about rendering details like: "What should be the distance between points?" or "What's the height of the data relative to the total plot?" We'll just tell the program the requirements, and it will figure out how to fulfill them.

<!-- TOC -->

## Primer on SVG

SVG is a markup language, much like HMTL, for representing vector graphics. We can use SVG markup inside of HTML by wrapping it in `<svg>` tags. Inside of the `<svg>` tag, we can place a bunch of elements, like circles, rectanges, and [more](https://developer.mozilla.org/en-US/docs/Web/SVG/Element#SVG_elements_A_to_Z). We'll stick to the `<circle />`, `<rect />`, `<path />` and `<g>...</g>` elements for this article.

### SVG Coordinate system

SVG uses a <span id="left-handed">left-handed coordinate system</span>. That is to say, the Y axis points down instead of up:

![Left handed Coordinate System](/left-handed-coordinate-system.svg)

This is a [confusing](https://math.stackexchange.com/a/2395007) [coordinate system](https://www.w3.org/TR/SVG/coords.html#InitialCoordinateSystem) rising from computer graphics, where the origin (0, 0) is in the top left of the screen.

Here's an example of a circle plotted at the origin:

```html
<!-- 10 x 5 grid, starting at 0, 0 -->
<svg viewBox="0 0 10 5">
  <circle cx="0" cy="0" r="1" fill="red"/>
</svg>
```

<!-- 10 x 5 grid, starting at 0, 0 -->

<svg viewBox="0 0 10 5" class="checkerboard">
  <circle cx="0" cy="0" r="1" fill="red"/>
</svg>

And here's an example of a circle plotted at (2, 4):

```html
<!-- 10 x 5 grid, starting at 0, 0 -->
<svg viewBox="0 0 10 5">
  <circle cx="2" cy="4" r="1" fill="red" />
</svg>
```

<!-- 10 x 5 grid, starting at 0, 0 -->

<svg viewBox="0 0 10 5" class="checkerboard">
  <circle cx="2" cy="4" r="1" fill="red" />
</svg>

This is important, because if we want to graph data, we'll need to invert the Y axis for any data. The x-axis should basically act as we expect.

### Viewbox: Relative Size and Boundries

Often underappreciated, the SVG `viewBox` attribute sets the stage for us to place our elements. By specifying four numbers, `min-x`, `min-y`, `width`, `height`, we specify the relative size of our canvas. All of our elements' heights, widths, and positions, will be relative to this canvas. So this SVG has a box taking up the left half:

```html
  <!-- 100 x 70 grid, starting at 0, 0 -->
  <svg viewBox="0 0 100 70">
    <rect height="50" width="50" fill="red"/>
  </svg>
```

<!-- 100 x 70 grid, starting at 0, 0 -->

<svg viewBox="0 0 100 70" class="checkerboard">
  <rect height="50" width="50" fill="red"/>
</svg>

And this SVG, the _same_ rectangle takes up a fraction of the space, because the viewBox is much 10 times as large.

```html
  <!-- 1000 x 700 grid, starting at 0, 0 -->
  <svg viewBox="0 0 1000 700">
    <rect height="50" width="50" fill="red"/>
  </svg>
```

<!-- 1000 x 700 grid, starting at 0, 0 -->

<svg viewBox="0 0 1000 700" class="checkerboard">
  <rect height="50" width="50" fill="red"/>
</svg>

There are more [in-depth](https://www.sarasoueidan.com/blog/svg-coordinate-systems/) sources you can consult if you want to further understand the principals at play here. The most important bit to understand is the relative size and shape of the grid we're using. I'll make note of the relative grid for the next few examples.

### Manually Plotting a Series

Let's try to plot this series: [(0, 5); (2, 8); (4, 7); (5, 8); (6, 9)]. As a first attempt a plotting it, let's to the naïve thing and just stick it into an SVG as-is:

```html
<!-- 10 x 10 grid, starting at 0, 0 -->
<svg viewBox="0 0 10 10">
  <circle cx="0" cy="5" r="0.25" fill="#000" />
  <circle cx="2" cy="8" r="0.25" fill="#000" />
  <circle cx="4" cy="7" r="0.25" fill="#000" />
  <circle cx="5" cy="8" r="0.25" fill="#000" />
  <circle cx="6" cy="9" r="0.25" fill="#000" />
</svg>
```

<svg viewBox="0 0 10 10" class="checkerboard">
  <circle cx="0" cy="5" r="0.25" fill="#000" />
  <circle cx="2" cy="8" r="0.25" fill="#000" />
  <circle cx="4" cy="7" r="0.25" fill="#000" />
  <circle cx="5" cy="8" r="0.25" fill="#000" />
  <circle cx="6" cy="9" r="0.25" fill="#000" />
</svg>

First of all, our data is upside down, so we should probably fix that. The easiest way to transform the data before plotting. Let's flip the data upside down (-y) and add an offset (-y + 10) to get it back onto our viewBox:

| x   | y   | -y  | -y + 10 |
| --- | --- | --- | ------- |
| 0   | 5   | -5  | 5       |
| 2   | 8   | -8  | 2       |
| 4   | 7   | -7  | 3       |
| 5   | 8   | -8  | 2       |
| 6   | 9   | -9  | 1       |

Plot of _x_ and _-y + 10_:

```html
<!-- 10 x 10 grid, starting at 0, 0 -->
<svg viewBox="0 0 10 10">
  <circle cx="0" cy="5" r="0.25" fill="#000" />
  <circle cx="2" cy="2" r="0.25" fill="#000" />
  <circle cx="4" cy="3" r="0.25" fill="#000" />
  <circle cx="5" cy="2" r="0.25" fill="#000" />
  <circle cx="6" cy="1" r="0.25" fill="#000" />
</svg>
```

<svg viewBox="0 0 10 10" class="checkerboard">
  <circle cx="0" cy="5" r="0.25" fill="#000" />
  <circle cx="2" cy="2" r="0.25" fill="#000" />
  <circle cx="4" cy="3" r="0.25" fill="#000" />
  <circle cx="5" cy="2" r="0.25" fill="#000" />
  <circle cx="6" cy="1" r="0.25" fill="#000" />
</svg>

Okay, better, it actually looks like our data, but it's a little off the page. Let's shift the x coordinates a bit(x'), but keep the same output:

| x'  | -y + 10 |
| --- | ------- |
| 2   | 5       |
| 4   | 2       |
| 6   | 3       |
| 7   | 2       |
| 8   | 1       |

Plot of _x'_ and _-y + 10_:

```html
<!-- 10 x 10 grid, starting at 0, 0 -->
<svg viewBox="0 0 10 10">
  <circle cx="2" cy="5" r="0.25" fill="#000" />
  <circle cx="4" cy="2" r="0.25" fill="#000" />
  <circle cx="6" cy="3" r="0.25" fill="#000" />
  <circle cx="7" cy="2" r="0.25" fill="#000" />
  <circle cx="8" cy="1" r="0.25" fill="#000" />
</svg>
```

<svg viewBox="0 0 10 10" class="checkerboard">
  <circle cx="2" cy="5" r="0.25" fill="#000" />
  <circle cx="4" cy="2" r="0.25" fill="#000" />
  <circle cx="6" cy="3" r="0.25" fill="#000" />
  <circle cx="7" cy="2" r="0.25" fill="#000" />
  <circle cx="8" cy="1" r="0.25" fill="#000" />
</svg>

Close enough. We've now got a basic plot of our data within an SVG and we've handled some of the most common transformations we will need to perform on that data. For the rest of the this post we'll perform our transforms with functions instead of manually. Now let's start integrating our charts with React.

## Scatterplot in React

Now that we understand the general principals of plotting our data, let's plot the same data using React. We'll store the data in an array and map it to the corresponding `<circle>` elements:

<div class="full-width-codesandbox">
<iframe src="https://codesandbox.io/embed/github/jameskraus/sandbox-svg-react-boxplot/tree/naive/?codemirror=1&view=split" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>
</div>

Now, let's write a few helper functions to perform the data transformation to
get the data to appear on the SVG. Remember, we need to invert the data
(y → -y), move it down by 10 units (-y → -y + 10), and shift the inputs by a
few units to get the plot centered (x → x + 2).

<div class="full-width-codesandbox">
<iframe src="https://codesandbox.io/embed/github/jameskraus/sandbox-svg-react-boxplot/tree/transformed/?codemirror=1&highlights=13,14,16" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>
</div>

Alright, now we've got the data rendering to the SVG. It's a little annoying to write the transformation functions to adjust the [domain and range](http://www.purplemath.com/modules/fcns2.htm) of our data series.

Our custom transformation functions (invertOutput and shiftInput) worked fine for this data set, but if the domain or range of the data set ever changes, we'll run into problems with our data appearing offscreen. So, how do we define a set of transformations that will work for any data set? First, we'll want to know the min/max of our dataset. We could calculate this with [Math.min](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/min) and [Math.max](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/max), but D3 contains a function specifically for calculating both: [d3.extent()](https://github.com/d3/d3-array#extent). Let's go ahead and use that:

<div class="full-width-codesandbox">
<iframe src="https://codesandbox.io/embed/github/jameskraus/d3-extent-example/tree/master/?module=%2Fsrc%2Findex.js&codemirror=1&highlights=11,12" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>
</div>

D3 has some [excellent scaling functions](https://github.com/d3/d3-scale) available for transforming the domain/range of various inputs. The scales map values from input _domains_ to output _ranges_. We'll use the minimum and maximum values from d3.extent() as the input domain for our scaling function. Since our plot is a 10x10 grid, we'll set our output range to [2, 8] so our values appear in the middle of the graph (with 2 units of padding):

<div class="full-width-codesandbox">
<iframe src="https://codesandbox.io/embed/github/jameskraus/sandbox-svg-react-boxplot/tree/with-d3-scale/?codemirror=1&highlights=18,19,20,21,22,24,25,26,27" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>
 </div>

Voilà! A graph that can take in arbitrary sets and render them to the page. Go ahead and tinker with the `data` values and you should see the graph update to match the new data.

## Boxplots

We'll be using the `<rect>` element to render the boxplot. The `rect` element takes four primary attributes: `x`, `y`, `height`, and `width`. First of all, we'll create some dummy data to plot:

```js
const data = [1, 2, 3, 10, 4, 5, 6]
```

We can work backwords to create our ideal component. Ideally, the `x`, `y`, and `height` attributes need to be calculated for each box, but the width of the box is probably the same for each bar, so we'll render something like this:

```js
<svg viewBox="0 0 10 10">
  {boxes.map(({ xPos, yPos, height }) => (
    <rect x={xPos} y={yPos} width={widthPerElement} height={height} r="0.25" />
  ))}
</svg>
```

The height of the bar should be given by a scale function that takes the raw value and fits it inside our graph. We'll generate the x position (xPos) from a hypothetical `scalePosition` function that takes in the index of the current item and returns a good place to put it on the graph. Until we preview the graph, we'll just keep the y position (yPos) constant and see how the graph turns out.

```js
const boxes = data.map((value, i) => {
  const xPos = scalePosition(i)
  const height = scaleValue(value)
  const yPos = 0 // we'll adjust this later
  return {
    xPos,
    yPos,
    height,
  }
})
```

Implementing the scale function for values is a matter of using D3's [scaleLinear API](https://github.com/d3/d3-scale#linear-scales) again:

```js
// Scale input domain from 0 to the max value in the data set
// to an input range of 0, 8 (for some padding)
const scaleValue = d3
  .scaleLinear()
  .domain([0, Math.max(...data)])
  .range([0, 8])
```

For scaling the position, we'll need to have an input domain [0, n-1] for n data points. Since we draw rectangles from the top-left corner, the output range of our scale will be the coordinates of the top-left corners of the first and last rectangles.

```js
// Takes the index of the item in our data array and
// returns the position of the top-left point
const scalePosition = d3
  .scaleLinear()
  .domain([0, data.length - 1])
  // Uses widthPerElement to offset the top-left point
  // by however wide our box happens to be. So as long
  // as the last box has width = widthPerElement, then
  // it will just touch the maxXPos
  .range([minXPos, maxXPos - widthPerElement])
```

Finally, we'll set all the constants we used:

```js
// One unit less than the min/max coordinates on our graph
const minXPos = 1
const maxXPos = 9

const totalWidth = maxXPos - minXPos
const widthPerElement = totalWidth / (data.length * 2 - 1)
```

And throwing it all together, here's our boxplot:

<div class="full-width-codesandbox">
<iframe src="https://codesandbox.io/embed/github/jameskraus/sandbox-svg-react-boxplot/tree/first-box-plot/?codemirror=1" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>
</div>

Since SVG's y-axis points down, all of our boxes are pointing down as well. We can turn them the right-side-up by offsetting their yPos by a fixed amount minus their height. Let's use a height of `9` since then they'll take up about 90% of the svg:

<div class="full-width-codesandbox">
<iframe src="https://codesandbox.io/embed/github/jameskraus/sandbox-svg-react-boxplot/tree/box-plot-right-side-up/?codemirror=1" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>
</div>

There we have it! A box plot which automatically scales to our input data _and_ it's right side up. Now that we've got the general hang of things, let's take a quick detour into path elements.

## Path Elements

Paths allow us to specify arbitrary shapes, and while we can get a _lot_ done with basic shapes like `<rect />` and `<circle />`, paths give us the most flexibility. Paths accept the path description attribute `d`, which contains a list of [commands](https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute/d) to move our position, draw a line, draw a curve, draw an arc, or close our path.

### Manually graphing with a path

Let's graph some points in the SVG coordinate system. Since these are already in the [left-handed coordinate system](#left-handed), we don't have to transform the data (_whew_). We'll also plot the (x,&nbsp;y) pairs with circles to make them stand out a bit:

| x   | y   |
| --- | --- |
| 10  | 18  |
| 20  | 6   |
| 30  | 6   |

```html
<!-- 40 x 20 grid -->
<svg viewBox="0 0 40 20" style="height: 400px; width: 400px">
  <!-- M: Start at (10, 18) ; L: draw line to (20, 6) ; L: draw line to (30, 6) -->
  <path d="M 10 18 L 20 6 L 30 6" stroke="black" fill="transparent" stroke-width=".5" />

  <!-- The points of y -->
  <circle cx="10" cy="18" r=".5" fill="#a44" />
  <circle cx="20" cy="6" r=".5" fill="#4a4" />
  <circle cx="30" cy="6" r=".5" fill="#44a" />
</svg>
```

<!-- 40 x 20 grid -->

<svg viewBox="0 0 40 20" style="height: 400px; width: 400px" class="checkerboard">
  <!-- Start at (10, 18) ; draw line to (20, 6) ; draw line to (30, 6) -->
  <path d="M 10 18 L 20 6 L 30 6" stroke="black" fill="transparent" stroke-width=".5" />

  <!-- The points of y -->

  <circle cx="10" cy="18" r=".5" fill="#a44" />
  <circle cx="20" cy="6" r=".5" fill="#4a4" />
  <circle cx="30" cy="6" r=".5" fill="#44a" />
</svg>

### More Advanced Paths by Leveraging D3

Let's try graphing the same data from our [boxplots](#boxplots), but as a pie chart instead. The end result will be 7 paths, one for each data point, with shapes corresponding to their values. There are two main D3 features we'll want to leverage to generate the pie charts: [`d3.pie()`](https://github.com/d3/d3-shape/blob/master/README.md#pies) and [`d3.arc()`](https://github.com/d3/d3-shape/blob/master/README.md#arcs). First, `d3.pie()` takes our data and generates the general characteristics of each arc of the pie chart (start angle, end angle, etc.) while `d3.arc()` takes that arc data and generates the paths' `d` attribute for each [sector](https://en.wikipedia.org/wiki/Circular_sector) of the chart. Let's see it in action:

<div class="full-width-codesandbox">
<iframe src="https://codesandbox.io/embed/github/jameskraus/sandbox-d3-path/tree/pie-chart/?codemirror=1&view=split" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>
</div>

### What good is React without state management?

Using React to render our charts, with its declarative style, makes rendering stateful charts so much easier. Let's slam the bar chart and pie chart together to showcase sharing state. When the user hovers over an item, we'll add a `stroke` around the edge of the corresponding bar and sector:

<div class="full-width-codesandbox">
<iframe src="https://codesandbox.io/embed/github/jameskraus/sandbox-d3-path/tree/linked-charts/?codemirror=1&view=split" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>
</div>

## Wrap-Up

You don't need React for declarative data visualization, but it does make it easier. HTML and SVG are already declarative languages: you don't tell the computer the steps for rending elements, you just tell it what you want rendered and the browser does all the hard work (scaling, coloring, alignment, etc.). JSX builds on top of this declarative foundation and lets us specify components (with props) to render and React worries about when and how to render them. The most difficult part of rendering SVGs with React is calculating the coordinates and shapes that will make up our visualization. Fortunately, there are already great libraries like D3 to handle the most common cases of data manipulation and shape calculation.

You can explore the D3 documentation for [more shapes](https://github.com/d3/d3-shape/blob/master/README.md) you might want to use in your visualizations. If you're looking for inspiration, check out the [D3 gallery](https://github.com/d3/d3/wiki/Gallery) which contains many high-quality data visualization examples. Until next time, thanks for reading!

<style type="text/css">
.checkerboard {
  max-height: 400px;
  max-width: 400px;
  background-color: #ddd;
  background-image:
  linear-gradient(45deg, #fff 25%, transparent 25%),
  linear-gradient(-45deg, #fff 25%, transparent 25%),
  linear-gradient(45deg, transparent 75%, #fff 75%),
  linear-gradient(-45deg, transparent 75%, #fff 75%);
  background-size: 20px 20px;
  background-position: 0 0, 0 10px, 10px -10px, -10px 0px;
}
</style>
