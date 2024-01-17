---
source: https://www.chartjs.org/docs/latest/getting-started/
tags:
  - javascript
  - draft
  - chartjs
---
The simplest way to include the chartjs library is to link the cdn from [jsdeliver](https://www.jsdelivr.com) in a script tag

> [!note]
> [jsdeliver](https://www.jsdelivr.com) can serve files form any npm package in the public registry 
> ```html
> <script src="https://cdn.jsdelivr.net/npm/packagename"></script>
> ```

```html
<script src="https://cdn.jsdelivr.net/npm/chart.js" type="module"></script>
<script src="./indexjs" type="module"></script>
```

> [!warning]
> For some reason both `script` tags must have `type="module"`

As for the html mark up chartjs requires a `<canvas>` element on which to render the graph. 

```html
<div>
  <canvas id="myChart"></canvas>
</div>
```

> [!tip]
> Place the `canvas` within a container (typically a `div`) as charts responsively scale to fit the dimensions of their container

Including chartjs exposes the `Chart` class which is used to create new charts. The first argument is the `<canvas>` element on which the chart is rendered. The second argument is an options object which describes the chart.

The code below generates a bar graph (specified by `type: "bar"`) of various colors and random heights. 

`data` 
This property is an object describing the data to chart. 

`labels` 
corresponds to the values on the x-axis of the chart; colors in this case.

`datasets`
This is an array with each element being a dataset that is plotted on the chart. Each dataset is an object containing the group of data (in this case bars) that are placed on the graph as well as various options to customize the appearance of the dataset individually.   

Think of the chart as a blank graph with labelled axes. We can plot data from various sources on this blank graph, however this leads to a jumbled mess of multiple data points making it very difficult to distinguish which point comes from which source. Datasets essentially allow the grouping of the points from each source distinguished by a label for each source, plotting data points from each source in a unique color and other unique styles.


`options`
is an object that allows the specification of custom values for various options provides by chartjs

```javascript
function buildArr(size, getItem){
  const rtnArr = [];
  for(let i = 0; i < size; i++){
    rtnArr.push(getItem(i, rtnArr));
  }
  return rtnArr;
}

function randArr(size, min, max){
  return buildArr(size, () => Math.floor(Math.random() * (max - min) + min))
}

new Chart(document.getElementById("myChart"), {
  type: "bar",
  data: {
    labels: [
      "January", "February", "March", 
      "April", "May", "June", 
      "July", "August", "September", 
      "October", "November", "December"
    ],
    datasets: [
      {
        label: "customers",
        data: randArr(12, 10, 1000),
        borderWidth: 1
      },
      {
        label: "subscribers",
        data: randArr(12, 10, 1000),
        borderWidth: 1
      },      
    ]
  },
  options: {
    scales: {
      y: { beginAtZero: true }
    }
  }
})
```

![[Pasted image 20240117203148.png]]

read later
- [jsdeliver](https://www.jsdelivr.com) a cdn for packages on the npm registry 