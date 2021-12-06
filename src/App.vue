<!--
 * @Author: wangwenchao6
 * @Date: 2021-12-06 13:58:21
 * @LastEditTime: 2021-12-06 18:17:15
 * @LastEditors: wangwenchao6
 * @Description: 
-->
<template>
  <div id="app"></div>
</template>

<script>
import * as d3 from "d3";
export default {
  name: "App",
  data() {
    return {};
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      //画布大小
      var width = 1900;
      var height = 900;
      //在 body 里添加一个 SVG 画布
      var svg = d3
        .select("body")
        .append("svg")
        .attr("width", width)
        .attr("height", height);

      const dataset = {
        nodes: [
          { id: 1 },
          { id: 2 },
          { id: 3 },
          { id: 4 },
          { id: 5 },
          { id: 6 },
        ],
        links: [
          { source: 1, target: 5 },
          { source: 4, target: 5 },
          { source: 4, target: 6 },
          { source: 3, target: 2 },
          { source: 5, target: 2 },
          { source: 1, target: 2 },
          { source: 3, target: 4 },
        ],
      };
      // Initialize the links
      const link = svg
        .append("g")
        .attr("class", "links")
        .selectAll("line")
        .data(dataset.links)
        .enter()
        .append("line")
        .attr("stroke-width", function () {
          return 2;
        });
      // Initialize the nodes
      const node = svg
        .append("g")
        .attr("class", "nodes")
        .selectAll("circle")
        .data(dataset.nodes)
        .enter()
        .append("circle")
        .attr("r", 20)
        .call(
          d3
            .drag() //sets the event listener for the specified typenames and returns the drag behavior.
            .on("start", dragstarted) //start - after a new pointer becomes active (on mousedown or touchstart).
            .on("drag", dragged) //drag - after an active pointer moves (on mousemove or touchmove).
            .on("end", dragended) //end - after an active pointer becomes inactive (on mouseup, touchend or touchcancel).
        );

      // Text to nodes
      const text = svg
        .append("g")
        .attr("class", "text")
        .selectAll("text")
        .data(dataset.nodes)
        .enter()
        .append("text")
        .text((d) => d.id);

      var simulation = d3
        .forceSimulation()
        .force("charge", d3.forceManyBody().strength(-500).distanceMax(500))
        .force(
          "link",
          d3.forceLink().id(function (d) {
            return d.id;
          })
        )
        .force("center", d3.forceCenter(width / 2, height / 2));

      simulation
        .nodes(dataset.nodes) //sets the simulation’s nodes to the specified array of objects, initializing their positions and velocities, and then re-initializes any bound forces;
        .on("tick", ticked);
      simulation.force("link").links(dataset.links);
      // dataset.nodes   .on("tick", ticked)
      function ticked() {
        link
          .attr("x1", (d) => d.source.x)
          .attr("y1", (d) => d.source.y)
          .attr("x2", (d) => d.target.x)
          .attr("y2", (d) => d.target.y);

        node.attr("cx", (d) => d.x).attr("cy", (d) => d.y);

        text
          .attr("x", (d) => d.x - 5) //position of the lower left point of the text
          .attr("y", (d) => d.y + 5); //position of the lower left point of the text
      }
      //When the drag gesture starts, the targeted node is fixed to the pointer
      //The simulation is temporarily “heated” during interaction by setting the target alpha to a non-zero value.
      function dragstarted(event,d) {
        // console.log(currentEvent);
        if (!event.active) simulation.alphaTarget(0.3).restart(); //sets the current target alpha to the specified number in the range [0,1].
        d.fy = d.y; //fx - the node’s fixed x-position. Original is null.
        d.fx = d.x; //fy - the node’s fixed y-position. Original is null.
      }

      //When the drag gesture starts, the targeted node is fixed to the pointer
      function dragged(event,d) {
        d.fx = event.x;
        d.fy = event.y;
      }

      //the targeted node is released when the gesture ends
      function dragended() {
        // if (!event.active) simulation.alphaTarget(0);
        // d.fx = null;
        // d.fy = null;
        // console.log("dataset after dragged is ...", dataset);
      }
    },
    init3() {
      //画布大小
      var width = 400;
      var height = 400;
      //在 body 里添加一个 SVG 画布
      var svg = d3
        .select("body")
        .append("svg")
        .attr("width", width)
        .attr("height", height);

      var dataset = [30, 10, 43, 55, 13];

      var pie = d3.pie();

      var piedata = pie(dataset);

      var outerRadius = 150; //外半径
      var innerRadius = 0; //内半径，为0则中间没有空白

      var arc = d3
        .arc() //弧生成器
        .innerRadius(innerRadius) //设置内半径
        .outerRadius(outerRadius); //设置外半径

      var arcs = svg
        .selectAll("g")
        .data(piedata)
        .enter()
        .append("g")
        .attr("transform", "translate(" + width / 2 + "," + width / 2 + ")");
      var color = d3.schemeCategory10;
      arcs
        .append("path")
        .attr("fill", function (d, i) {
          return color[i];
        })
        .attr("d", function (d) {
          return arc(d); //调用弧生成器，得到路径值
        });

      arcs
        .append("text")
        .attr("transform", function (d) {
          return "translate(" + arc.centroid(d) + ")";
        })
        .attr("text-anchor", "middle")
        .attr("fill", "#fff")
        .text(function (d) {
          return d.data;
        });
    },
    init2() {
      //画布大小
      var width = 400;
      var height = 400;

      //在 body 里添加一个 SVG 画布
      var svg = d3
        .select("body")
        .append("svg")
        .attr("width", width)
        .attr("height", height);

      //画布周边的空白
      var padding = { left: 30, right: 30, top: 20, bottom: 20 };

      //定义一个数组
      var dataset = [10, 20, 30, 40, 33, 24, 12, 5];

      //x轴的比例尺
      var xScale = d3
        .scaleBand()
        .domain(d3.range(dataset.length))
        .rangeRound([0, width - padding.left - padding.right]);

      //y轴的比例尺
      var yScale = d3
        .scaleLinear()
        .domain([0, d3.max(dataset)])
        .range([height - padding.top - padding.bottom, 0]);

      //矩形之间的空白
      var rectPadding = 4;

      //定义x轴
      var xAxis = d3.axisBottom().scale(xScale);
      //定义y轴
      var yAxis = d3.axisLeft().scale(yScale);

      //添加矩形元素
      var rects = svg
        .selectAll(".MyRect")
        .data(dataset)
        .enter()
        .append("rect")
        .attr("class", "MyRect")
        .attr(
          "transform",
          "translate(" + padding.left + "," + padding.top + ")"
        )
        .attr("x", function (d, i) {
          return xScale(i) + rectPadding / 2;
        })
        .attr("width", xScale.bandwidth() - rectPadding)
        .attr("height", 0)
        .attr("y", function () {
          var min = yScale.domain()[0];
          return yScale(min);
        })
        .attr("fill", "steelblue")
        .on("mouseover", function () {
          d3.select(this).transition().attr("fill", "#7ED26D");
        })
        .on("mouseout", function () {
          d3.select(this).transition().attr("fill", "steelblue");
        })
        .transition()
        .duration(1000)
        .attr("y", function (d) {
          return yScale(d);
        })
        .attr("height", function (d) {
          return height - padding.top - padding.bottom - yScale(d);
        });
      //添加文字元素
      var texts = svg
        .selectAll(".MyText")
        .data(dataset)
        .enter()
        .append("text")
        .attr("class", "MyText")
        .attr("transform", `translate(${padding.left},${padding.top})`)
        .attr("text-anchor", "middle")
        .attr("x", function (d, i) {
          return xScale(i) + rectPadding / 2;
        })
        .attr("dx", function () {
          return (xScale.bandwidth() - rectPadding) / 2;
        })
        .attr("y", function () {
          var min = yScale.domain()[0];
          return yScale(min);
        })
        .attr("dy", function () {
          return 0;
        })
        .transition()
        .duration(1000)
        .delay(200)
        .attr("y", function (d) {
          return yScale(d);
        })
        .attr("dy", function () {
          return 20;
        })
        .text(function (d) {
          return d;
        });

      //添加x轴
      svg
        .append("g")
        .attr("class", "axis")
        .attr(
          "transform",
          "translate(" + padding.left + "," + (height - padding.bottom) + ")"
        )
        .call(xAxis);

      //添加y轴
      svg
        .append("g")
        .attr("class", "axis")
        .attr(
          "transform",
          "translate(" + padding.left + "," + padding.top + ")"
        )
        .call(yAxis);

      console.log(rects);
      console.log(texts);
    },
    init1() {
      var width = 300; //画布的宽度
      var height = 300; //画布的高度

      var dataset = [2.5, 2.1, 1.7, 1.3, 0.9];
      var rectHeight = 25; //每个矩形所占的像素高度(包括空白)

      var linear = d3
        .scaleLinear()
        .domain([0, d3.max(dataset)])
        .range([0, 250]);

      const app = d3.select("body").select("#app");

      var svg = app //选择文档中的body元素
        .append("svg") //添加一个svg元素
        .attr("width", width) //设定宽度
        .attr("height", height); //设定高度

      svg
        .selectAll("rect")
        .data(dataset)
        .enter()
        .append("rect")
        .attr("x", 20)
        .attr("y", function (d, i) {
          return i * rectHeight;
        })
        .attr("width", function (d) {
          return linear(d);
        })
        .attr("height", rectHeight - 2)
        .attr("fill", "steelblue");

      var axis = d3
        .axisBottom()
        .scale(linear) //指定比例尺
        .ticks(7); //指定刻度的数量

      svg
        .append("g")
        .attr("class", "axis")
        .attr("transform", "translate(20,130)")
        .call(axis);

      console.log(axis);
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
.axis path,
.axis line {
  fill: none;
  stroke: black;
  shape-rendering: crispEdges;
}

.axis text {
  font-family: sans-serif;
  font-size: 11px;
}

.MyRect {
  /* fill: steelblue; */
}
.MyText {
  fill: #fff;
}

.links line {
  stroke: #000;
}

.nodes circle {
  fill: pink;
  stroke: #000;
}

text {
  pointer-events: none;
}
</style>
