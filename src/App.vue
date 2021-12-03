<template>
  <div id="app">
    
  </div>
</template>

<script>
import * as d3 from 'd3/dist/d3'
export default {
  name: 'App',
  data(){

  },
  mounted(){
    // this.init()
  },
  methods: {
    init() {
      var svg = d3.select("svg"), g,
        width = +svg.attr("width"),
        height = +svg.attr("height"),
        node, link,
        brushMode, brush, zoomLayer; //intercation canvas: Brush + zoom

      var color = d3.scaleOrdinal(d3.schemeCategory20);

      var simulation = d3.forceSimulation()
        .force("link", d3.forceLink().id(function (d) { return d.id; }))
        .force("charge", d3.forceManyBody())
        .force("center", d3.forceCenter(width / 2, height / 2));

      // add shift event
      d3.select("body")
        .on("keydown", keydown)
        .on("keyup", keyup)

      // brushable network: http://jsfiddle.net/pkerpedjiev/29majy5c/2/
      brush = d3.brush()
        .extent([[0, 0], [width, height]])
        .on("start", function () {
          node.each(function (d) { d.previouslyPicked = brushMode && d.picked; });
        })
        .on("brush", function () {
          if (!d3.event.selection) return;
          var extent = d3.event.selection,
            zoomProp = d3.zoomTransform(zoomLayer.node());
          node.classed("picked", function (d) { return d.picked = d.previouslyPicked ^ ((extent[0][0] - zoomProp.x) / zoomProp.k <= d.x && d.x < (extent[1][0] - zoomProp.x) / zoomProp.k && (extent[0][1] - zoomProp.y) / zoomProp.k <= d.y && d.y < (extent[1][1] - zoomProp.y) / zoomProp.k); });
        })
        .on("end", function () {
          if (!d3.event.selection) return;
          d3.select(this).call(d3.event.target.move, null);
        })

      let zoom = d3.zoom()
        .scaleExtent([1 / 2, 4])
        .on("zoom", zoomed);

      svg.append("g")
        .attr("id", "brush-layer")
        .attr("width", width)
        .attr("height", height)
        .style("fill", "none")
        .datum(function () { return { picked: false, previouslyPicked: false }; })
        .call(brush)
        .on("click", function () {
          node.classed("picked", false);
          node.each(function (d) { d.picked = d.previouslyPicked = false; })
        });


      zoomLayer = svg.append("rect")
        .attr("id", "zoom-layer")
        .attr("width", width)
        .attr("height", height)
        .style("fill", "none")
        .attr("pointer-events", "all")
        .call(zoom)

      g = svg.append("g");

      d3.json("/json/drawgraph_test.json", function (error, graph) {
        if (error) throw error;

        link = g.append("g")
          .attr("class", "links")
          .selectAll("line")
          .data(graph.links)
          .enter().append("line")
          .attr("stroke-width", function (d) { return Math.sqrt(d.value); });

        node = g.append("g")
          .attr("class", "nodes")
          .selectAll("circle")
          .data(graph.nodes)
          .enter().append("circle")
          .attr("r", 5)
          .attr("fill", function (d) { return color(d.group); })
          .on("click", function () {
            var thisNode = d3.select(this);
            if (brushMode) {
              node.each(function (d) { d.previouslyPicked = brushMode && d.picked; });
              node.classed("picked", function (d) {
                return d.picked = d.previouslyPicked ^ (thisNode.datum().id === d.id);
              });
            }
          })
          .call(d3.drag()
            .on("start", dragstarted)
            .on("drag", dragged)
            .on("end", dragended));


        node.append("title")
          .text(function (d) { return d.id; });

        simulation
          .nodes(graph.nodes)
          .on("tick", ticked);

        simulation.force("link")
          .links(graph.links);

        function ticked() {
          link
            .attr("x1", function (d) { return d.source.x; })
            .attr("y1", function (d) { return d.source.y; })
            .attr("x2", function (d) { return d.target.x; })
            .attr("y2", function (d) { return d.target.y; });

          node
            .attr("cx", function (d) { return d.x; })
            .attr("cy", function (d) { return d.y; });
        }
      });

      function dragstarted(d) {
        if (!d3.event.active) simulation.alphaTarget(0.3).restart();
        d.fx = d.x;
        d.fy = d.y;
      }

      function dragged(d) {
        d.fx = d3.event.x;
        d.fy = d3.event.y;
      }

      function dragended(d) {
        if (!d3.event.active) simulation.alphaTarget(0);
        d.fx = null;
        d.fy = null;
      }

      function keydown() {
        brushMode = d3.event.metaKey;
        const cleanNodes = d3.event.keyCode === 68;
        if (brushMode) zoomLayer.attr("pointer-events", "none");
        if (cleanNodes) cleanSelected();
      }

      function keyup() {
        brushMode = d3.event.metaKey;
        if (!brushMode) zoomLayer.attr("pointer-events", "all");
      }

      function zoomed() {
        g.attr("transform", d3.event.transform);
      }

      function cleanSelected() {
        node.classed("picked", false);
        node.each(function (d) { d.picked = d.previouslyPicked = false; })
      }
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
#zoom-layer:active {
  cursor: move;
}

.links line {
  stroke: #999;
  stroke-opacity: 0.6;
}

.nodes circle {
  stroke: #fff;
  stroke-width: 1.5px;
}

.nodes circle.picked {
  stroke: #202020;
  stroke-width: 3px;
  stroke-opacity: 0.6;
}
</style>
