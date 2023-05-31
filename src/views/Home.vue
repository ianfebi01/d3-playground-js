<template>
    <div class="hello">
        <h1>{{ msg }}</h1>
        <div class="bar-chart border border-red-600"></div>
        <div class="canvas"></div>
    </div>
</template>
  
<script>
import * as d3 from "d3";

export default {
    name: "HelloWorld",
    props: {
        msg: String,
    },
    data() {
        return {
            x: null,
            gdp: [
                { country: "USA", value: 20.5 },
                { country: "China", value: 13.4 },
                { country: "Germany", value: 4.0 },
                { country: "Japan", value: 4.9 },
                { country: "France", value: 2.8 },
            ],
            menu: [
                {
                    name: "veg soup",
                    orders: 200,
                    complete: 180
                },
                {
                    name: "veg curry",
                    orders: 600,
                    complete: 400
                },
                {
                    name: "veg pasta",
                    orders: 300,
                    complete: 230
                },
                {
                    name: "veg burger",
                    orders: 700,
                    complete: 670
                },
                {
                    name: "veg surprise",
                    orders: 900,
                    complete: 850
                },
                {
                    name: "veg gg",
                    orders: 800,
                    complete: 700
                },
            ],
        };
    },
    mounted() {
        this.generateArc();
        // this.draw();
        this.drawD3();
    },
    methods: {
        updateChart(event) {
            var extent = event.selection;
            console.log("extent", extent[0]);
            const [x0, x1] = extent.map(this.y.invert);
            // console.log("invertt extent", extent[0]);
            console.log("invertt x0", x0);
            console.log("invertt x1", x1);

            // console.log(this.y.invert(extent[1]));
        },
        drawD3() {
            // select the svg container first
            const svg = d3
                .select(".bar-chart")
                .append("svg")
                .attr("width", 300)
                .attr("height", 300);

            // create margins & dimensions
            const margin = { top: 20, right: 20, bottom: 100, left: 100 };
            const graphWidth = 300 - margin.left - margin.right;
            const graphHeight = 300 - margin.top - margin.bottom;

            // var bar = svg.selectAll('.bar1')
            //   .data(data, d => d.score1)
            //   .enter().append('g')
            //   .attr('transform', (d, i) => `translate(${(i * 3) * barWidth + 5}, 0)`)
            //   .attr('class', 'bar1')

            // var bar2 = svg.selectAll('.bar2')
            //   .data(data, d => d.score2)
            //   .enter().append('g')
            //   .attr('transform', (d, i) => `translate(${barWidth + (i * barWidth * 3) + 1},0)`)
            //   .attr('class', 'bar2');

            const graph = svg
                .append("g")
                .attr("width", graphWidth)
                .attr("height", graphHeight)
                .attr("transform", `translate(${margin.left}, ${margin.top})`);

            // create axes groups
            const xAxisGroup = graph
                .append("g")
                .attr("transform", `translate(0, ${graphHeight})`);

            const yAxisGroup = graph.append("g");

            // d3.json("menu.json").then((data) => {
            this.y = d3
                .scaleLinear()
                .domain([0, d3.max(this.menu, (d) => d.orders)])
                .range([graphHeight, 0]);

            this.x = d3
                .scaleBand()
                .domain(this.menu.map((item) => item.name))
                .range([0, graphWidth])
                .paddingInner(0.2)
                .paddingOuter(0.2);

            // join the data to circs
            const rects = graph.selectAll("rect").data(this.menu);

            // add attrs to circs already in the DOM
            rects
                .attr("width", this.x.bandwidth)
                .attr("height", (d) => graphHeight - this.y(d.orders))
                .attr("fill", "orange")
                .attr("x", (d) => this.x(d.name))
                .attr("y", (d) => this.y(d.orders));

            // append the enter selection to the DOM
            rects
                .enter()
                .append("rect")
                .attr("width", this.x.bandwidth)
                .attr("height", (d) => graphHeight - this.y(d.orders))
                .attr("fill", "orange")
                .attr("x", (d) => this.x(d.name))
                .attr("y", (d) => this.y(d.orders));

            // console.log(this.x.invert(150));

            // create & call axesit
            const xAxis = d3.axisBottom(this.x);
            const yAxis = d3
                .axisLeft(this.y)
                .ticks(3)
                .tickFormat((d) => d + " orders");

            xAxisGroup.call(xAxis);
            yAxisGroup.call(yAxis);

            xAxisGroup
                .selectAll("text")
                .attr("transform", "rotate(-40)")
                .attr("text-anchor", "end");
            // });

            svg.call(
                d3
                    .brushY() // Add the brush feature using the d3.brush function
                    .extent([
                        [0, 0],
                        [300, 300],
                    ]) // initialise the brush area: start at 0,0 and finishes at width,height: it means I select the whole graph area
                    .on("start end", this.updateChart)
            );
        },

        draw() {
            const svg = d3.select(".bar-chart");

            const y = d3.scaleLinear().domain([0, 1000]).range([0, 500]);
            // join the data to circs
            const rects = svg.selectAll("rect").data(this.menu);

            // add attrs to circs already in the DOM
            rects
                .attr("width", 50)
                .attr("height", (d) => y(d.orders))
                .attr("fill", "orange")
                .attr("x", (d, i) => i * 70);

            // append the enter selection to the DOM
            rects
                .enter()
                .append("rect")
                .attr("width", 50)
                .attr("height", (d) => y(d.orders))
                .attr("fill", "orange")
                .attr("x", (d, i) => i * 70);
        },
        generateArc() {
            const w = 500;
            const h = 500;

            const svg = d3
                .select(".canvas")
                .append("svg")
                .attr("width", w)
                .attr("height", h);

            const sortedGDP = this.gdp.sort((a, b) => (a.value > b.value ? 1 : -1));
            const color = d3.scaleOrdinal(d3.schemeDark2);

            const max_gdp = d3.max(sortedGDP, (o) => o.value);

            const angleScale = d3
                .scaleLinear()
                .domain([0, max_gdp])
                .range([0, 1.5 * Math.PI]);

            const arc = d3
                .arc()
                .innerRadius((d, i) => (i + 1) * 25)
                .outerRadius((d, i) => (i + 2) * 25)
                .startAngle(angleScale(0))
                .endAngle((d) => angleScale(d.value));

            const g = svg.append("g");

            g.selectAll("path")
                .data(sortedGDP)
                .enter()
                .append("path")
                .attr("d", arc)
                .attr("fill", (d, i) => color(i))
                .attr("stroke", "#FFF")
                .attr("stroke-width", "1px")
                .on("mouseenter", function () {
                    d3.select(this).transition().duration(200).attr("opacity", 0.5);
                })
                .on("mouseout", function () {
                    d3.select(this).transition().duration(200).attr("opacity", 1);
                });

            g.selectAll("text")
                .data(this.gdp)
                .enter()
                .append("text")
                .text((d) => `${d.country} -  ${d.value} Trillion`)
                .attr("x", -150)
                .attr("dy", -8)
                .attr("y", (d, i) => -(i + 1) * 25);

            g.attr("transform", "translate(200,300)");
        },
    },
};
</script>
  
  <!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
    margin: 40px 0 0;
}

ul {
    list-style-type: none;
    padding: 0;
}

li {
    display: inline-block;
    margin: 0 10px;
}

a {
    color: #42b983;
}
</style>
  