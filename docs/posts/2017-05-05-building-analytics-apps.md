---
layout: post
title: "Building Analytics Apps in HTML5 and JavaScript"
author: Shawn Hermans
date: 2017-05-05 12:00:00 -0600
categories: [html5, javascript, d3, plotly]
---
Originally posted as an answer to [What technologies are used to build the sophisticated and advanced client-side dashboards and applications through HTML5 & JavaScript?](https://www.quora.com/What-technologies-are-used-to-build-the-sophisticated-and-advanced-client-side-dashboards-and-applications-through-HTML5-JavaScript/answer/Shawn-Hermans?srid=hLq3)

I can give you a quick overview of some of the technologies I have used to build client-side dashboards and applications.

First, I recommend using [Scalable Vector Graphics](https://www.w3schools.com/html/html5_svg.asp) (SVG) for charts, graphs, and other interactive visualizations. SVG works on all modern browsers, supports interactive JavaScript, and scales well to displays of all sizes.

While you can create your own code to manipulate SVG visualizations, I do not recommend it. If you are looking to create customized SVG visualizations, you should probably use [D3.js - Data-Driven Documents](https://d3js.org/). D3.js is extremely powerful and flexible and is the underlying technology for many plotting and graphing libraries.

D3.js is wonderful but does not provide easy to use, out of the box charting and graphing capabilities. I have tried out a few different JavaScript graphing libraries and finally settled on [Plot.ly](https://plot.ly/). They provide open source libraries for Python, R, MATLAB, and JavaScript. They also provide a commercial offering for hosted services. Out of all of the solutions I tried, it was the easiest to use out of the box.

This covers the charting library but does not provide the overall application platform. There are many options to choose from. [Keen IO](http://keen.github.io/dashboards/) provides a collection of templates that use Bootstrap. Plot.ly also provides a provides a solution for creating [interactive dashboards](https://plot.ly/dashboards/). If you need something more complicated, you can always use a web application framework like Angular, Polymer, or React.

Lastly, there are also options if you are not a strong JavaScript developer. If you are a Python developer, [Bokeh](http://bokeh.pydata.org/en/latest/) uses D3.js and provides a way to create interactive visualization. Similarly, [Shiny](https://shiny.rstudio.com/) is an option if you are an R developer.
