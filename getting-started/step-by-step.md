---
source: https://www.chartjs.org/docs/latest/getting-started/usage.html
tags:
  - chartjs
  - javascript
  - draft
---
There are multiple ways to include chartjs. The simplest is via a cdn as covered in the previous note. The preferred method downloading the package from npm importing it using the [[import-export]] syntax. This has the benefits of being able to leverage tree shaking, typescript support etc. but with the draw back of requiring a bundler.

```
npm install chart.js
```

```javascript
import Chart from "chart.js/auto"
```

> [!note]
> importing `"chart.js/auto"` imports all chartjs components.

By default chartjs charts are responsive in that they fit themselves to the width of the container the are wrapped in. Wrapping the `<canvas>` element in a container allows the container to be styled in order to adjust the dimensions of the chart

```html
<div style="max-height: 90vh; display: grid; place-items: center;" >
  <canvas id="myChart"></canvas>
</div>
```

the code below creates a basic bar chart using simulated data in the form of an array of objects. The basic syntax is similar to that described in the previous note. Rather then generating a random array for data points map is used to create the data point array from the simulated data

```javascript
import Chart from "chart.js/auto"

(async function() {
  const data = [
    { year: 2010, count: 10 },
    { year: 2011, count: 20 },
    { year: 2012, count: 15 },
    { year: 2013, count: 25 },
    { year: 2014, count: 22 },
    { year: 2015, count: 30 },
    { year: 2016, count: 28 },
  ];

  new Chart(
    document.getElementById('acquisitions'),
    {
      type: 'bar',
      data: {
        labels: data.map(row => row.year),
        datasets: [
          {
            label: 'Acquisitions by year',
            data: data.map(row => row.count)
          }
        ]
      }
    }
  );
})();
```

![[Pasted image 20240117135423.png]]

> [!note]
> All examples seem to wrap the chart creation logic in an `async` function


***basic customizations***

`options.animation: boolean` 
enables of disables chart animations

`options.plugins.lengend.display: boolean` 
enables of disables the legend of the graph, this corresponds to the `label` property in each `dataset`.

`options.plugins.tooltip.enable: boolean`  toggles the tooltip that displays when a data point is hovered over

***bubble charts***
Bubble charts allow the display of 3 dimensions of data on a 2 dimensional plane. The extra dimension is displayed as the radius of the point.  

the `data` property of each data set object is an array of objects with `{ x, y, r }` properties which corresponds to the information for each point.

The code below generates a bubble chart of random data points.

```javascript
function buildArr(size, getItem){
  const rtnArr = [];
  for(let i = 0; i < size; i++){
    rtnArr.push(getItem(i, rtnArr));
  }
  return rtnArr;
}

function randInt(min, max){
  return Math.floor(Math.random() * (max - min) + min)
}

new Chart(document.getElementById("myChart"), {
  type: "bubble",
  data: {
    datasets: [
      {
        label: "dataset one",
        data: buildArr(300, () => ({
          x: randInt(0, 100),
          y: randInt(0, 100),
          r: randInt(2, 5)
        }))
      },
    ],
  },
  options: {
    scales: {
      x: { beginAtZero: true },
      y: { beginAtZero: true }
    }
  }
})
```

![[Pasted image 20240117184846.png]]

***more customizations***

***aspect ratio***
The aspect ratio of the chart can be specified in the `options.aspectRatio` option in the configuration object.

```javascript
new Chart(ctx, {
  options: { aspectRatio: 1 }
})
```

> [!info]
> By default chartjs charts have an aspect ratio of `1`  for all radial charts or 2 for other types

***axes range***
the range of the access can be specified with `options.scales.(x|y).(max|min)`. 

> [!info]
> By default the range of an axis is determine by the minimum and maximum value of the corresponding data point values.

> [!tip]
> `options.scales.(x|y).beginAtZero: boolean` is an alias for `options.scales.(x|y).min: 0`

```javascript
new Chart(ctx, {
  options: {
    scales: {
      x: { min: 0, max: 100 },
      y: { min: 0, max: 100 }
    }
  }
})
```

***custom ticks***
the ticks on the chart axes can be customized by specifying a callback function for  `options.scales.(x|y).ticks.callback`. This callback is passed the tick value and should return a string of the desired tick marker.

```javascript
new Chart(ctx, {
  options: {
    scales: {
      x: {
        ticks: { callback: x => x + "m" }
      },
      y: {
        ticks: { callback: y => y + "m" }
      },
    }
  }
})
```


below is the resulting chart from applying the discussed customizations to the bubble chart from earlier 

![[Pasted image 20240117192241.png]]


***sorting the data***
The code below demonstrates the power of multiple datasets as discussed in the period note. In short it groups the dataset into 3 based on a specified criteria (accomplished using `Array.prototype.filter`). Each group is then passed as the `data` of a dataset object in the `data.datasets` array. The groups are labeled using `label`.  

```javascript
const data = [
  ...buildArr(950, () => ({
    x: randInt(5, 95),
    y: randInt(5, 95),
    r: randInt(2, 5)
  })),
  ...buildArr(50, item => {
    const v = randInt(5, 95);
    return { x: v, y: v, r: randInt(2, 5) }
  })
] 

config.datasets = [
  {
    label: "x > y",
    data: data.filter(p => p.x < p.y)
  },   
  {
    label: "x = y",
    data: data.filter(p => p.x === p.y)
  },   
  {
    label: "x < y",
    data: data.filter(p => p.x > p.y)
  },                  
]
```

![[Pasted image 20240117194313.png]]


***plugins***
Plugins allow for very powerful chart customization.  A lot of chartjs functionality is provided through in-built plugins too. Chartjs also allows for the creation of custom plugins. There are various community plugins that can be found [here](https://github.com/chartjs/awesome#plugins).

Plugins are objects that implement certain methods specified in the chartjs [plugin API](https://www.chartjs.org/docs/latest/developers/plugins.html). These methods are called at specific times in the chart rendering pipeline and passed (among other things) the chart object and the plugin options (specified in the config object). The chart object contains chart related data, such as the dimension or the canvas context of the chart. With direct access to the canvas context the possibilities are endless.

below is an example of a custom plugin the draws a border around a chart.

```javascript
  const chartAreaBorder = {
    id: 'chartAreaBorder',

    beforeDraw(chart, args, options) {
      const { ctx, chartArea: { left, top, width, height } } = chart;

      ctx.save();
      ctx.strokeStyle = options.borderColor;
      ctx.lineWidth = options.borderWidth;
      ctx.setLineDash(options.borderDash || []);
      ctx.lineDashOffset = options.borderDashOffset;
      ctx.strokeRect(left, top, width, height);
      ctx.restore();
    }
  };

// pass the plugin to chartjs
config.plugins = [ chartAreaBorder ]; 

config.options.plugins = {
  // specify the options passed into the plugin's methods
  // plugin.id: optionsObject
  chartAreaBorder: {
    borderColor: 'red',
    borderWidth: 2,
    borderDash: [ 5, 5 ],
    borderDashOffset: 2,
  }
}
```


read later
- [chartjs plugins](https://github.com/chartjs/awesome#plugins)
- [roughjs](https://roughjs.com/) library for creating hand-drawn-like graphics on js canvas. 