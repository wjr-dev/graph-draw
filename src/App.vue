<!--
 * @Author: wangwenchao6
 * @Date: 2021-12-06 13:58:21
 * @LastEditTime: 2021-12-07 18:28:42
 * @LastEditors: wangwenchao6
 * @Description: 
-->
<template>
  <div id="app">
    <button @click="reLayout">重新布局</button>
  </div>
</template>

<script>
import * as d3 from "d3";
export default {
  name: "App",
  data() {
    return {};
  },
  mounted() {
    const data = localStorage.getItem("data");
    if (data) {
      this.init(JSON.parse(data));
    } else {
      fetch("./drawgraph_test.json")
        .then((response) => response.json())
        .then((res) => {
          const graph = {
            nodes: res.st.map((item) => {
              if (item.pathName === "bus-1200000") {
                return {
                  id: item.mRID,
                  category: "fixed",
                  group: Math.round(Math.random() * 8),
                  x: 766,
                  y: 473,
                  pathName: item.pathName,
                };
              } else {
                return {
                  id: item.mRID,
                  category: "unfixed",
                  group: Math.round(Math.random() * 8),
                  x: null,
                  y: null,
                  pathName: item.pathName,
                };
              }
            }),
            links: res.line.map((item) => {
              return {
                ...item,
                source: item.StartSt,
                target: item.EndSt,
              };
            }),
          };
          this.init(graph);
        });
    }
  },
  methods: {
    reLayout(){
      localStorage.removeItem('data')
    },
    init(dataset) {
      //画布大小
      var width = 3000;
      var height = 3000;
      //在 body 里添加一个 SVG 画布
      var svg = d3
        .select("body")
        .append("svg")
        .attr("width", width)
        .attr("height", height);

      const nodesCopy = JSON.parse(JSON.stringify(dataset.nodes));
      const linksCopy = JSON.parse(JSON.stringify(dataset.links));
      // 箭头源
      svg
        .append("defs")
        .append("marker")
        .attr("id", "arrowhead")
        .attr("viewBox", "-0 -3 10 10") //the bound of the SVG viewport for the current SVG fragment. defines a coordinate system 10 wide and 10 high starting on (0,-5)
        .attr("refX", 35) // x coordinate for the reference point of the marker. If circle is bigger, this need to be bigger.
        .attr("refY", 0)
        .attr("orient", "auto")
        .attr("markerWidth", 10)
        .attr("markerHeight", 10)
        .attr("xoverflow", "visible")
        .append("svg:path")
        .attr("d", "M 0,-3 L 10 ,0 L 0,3")
        .attr("fill", "#333")
        .style("stroke", "none");
      // 关联线
      const link = svg
        .append("g")
        .attr("class", "links")
        .selectAll("line")
        .data(dataset.links)
        .enter()
        .append("line")
        .attr("marker-end", "url(#arrowhead)");

      // 箭头
      const edgepaths = svg
        .selectAll(".edgepath") //make path go along with the link provide position for link labels
        .data(dataset.links)
        .enter()
        .append("path")
        .attr("class", "edgepath")
        .attr("fill-opacity", 0)
        .attr("stroke-opacity", 0)
        .attr("id", function (d, i) {
          return "edgepath" + i;
        })
        .style("pointer-events", "none");
      // 关联文字
      // const edgelabels = svg
      //   .selectAll(".edgelabel")
      //   .data(dataset.links)
      //   .enter()
      //   .append("text")
      //   .style("pointer-events", "none")
      //   .attr("class", "edgelabel")
      //   .attr("id", function (d, i) {
      //     return "edgelabel" + i;
      //   });

      // edgelabels
      //   .append("textPath") //To render text along the shape of a <path>, enclose the text in a <textPath> element that has an href attribute with a reference to the <path> element.
      //   .attr("xlink:href", function (d, i) {
      //     return "#edgepath" + i;
      //   })
      //   .style("pointer-events", "none")
      //   .attr("startOffset", "50%")
      //   .text((d) => d.pathName);

      const node = svg
        .append("g")
        .attr("class", "nodes")
        .selectAll("circle")
        .data(dataset.nodes)
        .enter()
        .append("circle")
        .attr("r", 25)
        .attr("fill", (d) => d3.schemeDark2[d.group])
        .call(
          d3
            .drag()
            .on("start", dragstarted)
            .on("drag", dragged)
            .on("end", dragended)
        );
      // .on("dblclick", (e, d) => {

      // });

      const text = svg
        .append("g")
        .attr("class", "text")
        .selectAll("text")
        .data(dataset.nodes)
        .enter()
        .append("text")
        .text((d) => d.pathName);

      var simulation = d3
        .forceSimulation()
        .force("charge", d3.forceManyBody().strength(-500))
        .force(
          "link",
          d3.forceLink().id(function (d) {
            return d.id;
          })
        )
        .force("center", d3.forceCenter(width / 2, height / 2));

      simulation.nodes(dataset.nodes).on("tick", ticked);
      simulation.force("link").links(dataset.links).distance(200);
      function ticked() {
        node
          .attr("cx", (d) => {
            const copy = nodesCopy.find((item) => item.id === d.id);
            if (d.category === "fixed" && copy.x) {
              d.x = copy.x;
            }
            return d.x;
          })
          .attr("cy", (d) => {
            const copy = nodesCopy.find((item) => item.id === d.id);
            if (d.category === "fixed" && copy.y) {
              d.y = copy.y;
            }
            return d.y;
          });

        text.attr("x", (d) => d.x).attr("y", (d) => d.y);

        link
          .attr("x1", (d) => d.source.x)
          .attr("y1", (d) => d.source.y)
          .attr("x2", (d) => d.target.x)
          .attr("y2", (d) => d.target.y);

        edgepaths.attr(
          "d",
          (d) => `M ${d.source.x} ${d.source.y} L ${d.target.x} ${d.target.y}`
        );
      }

      function dragstarted(event, d) {
        const index = nodesCopy.findIndex((item) => item.id === d.id);
        nodesCopy[index].x = null;
        nodesCopy[index].y = null;
        if (!event.active) simulation.alphaTarget(0.3).restart();
      }

      function dragged(event, d) {
        d.fx = Math.round(event.x);
        d.fy = Math.round(event.y);
      }

      //the targeted node is released when the gesture ends
      function dragended(event, d) {
        d.category = "fixed";
        const graph = {
          nodes: dataset.nodes.map((item) => {
            return {
              id: item.id,
              category: item.category,
              group: item.group,
              x: item.x,
              y: item.y,
              pathName: item.pathName,
            };
          }),
          links: linksCopy,
        };
        localStorage.setItem("data", JSON.stringify(graph));
      }
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  /* text-align: center; */
  color: #2c3e50;
}

.links line {
  stroke: #000;
}

.nodes circle {
  /* fill: pink; */
  stroke: #000;
}

.nodes rect {
  fill: steelblue;
  stroke: #000;
}
.text {
  fill: #fff;
  font-size: 12px;
  text-anchor: middle;
  dominant-baseline: middle;
}
.edgelabel {
  fill: tomato;
  font-size: 12px;
  text-anchor: middle;
  dominant-baseline: text-after-edge;
}
text {
  pointer-events: none;
}
</style>
