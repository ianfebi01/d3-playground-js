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
  name: "multibar",
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
          complete: 180,
        },
        {
          name: "veg curry",
          orders: 600,
          complete: 400,
        },
        {
          name: "veg pasta",
          orders: 300,
          complete: 230,
        },
        {
          name: "veg burger",
          orders: 700,
          complete: 670,
        },
        {
          name: "veg surprise",
          orders: 900,
          complete: 850,
        },
        {
          name: "veg gg",
          orders: 800,
          complete: 700,
        },
      ],
      dataVal: [],
    };
  },
  computed: {
    data: {
      get() {
        return this.dataVal;
      },
      set(newVal) {
        this.dataVal = newVal;
      },
    },
    maxValue() {
      const allData = [];
      this.data.map((item) => {
        allData.push(parseInt(item.score1));
        allData.push(parseInt(item.score2));
        allData.push(parseInt(item.score3));
      });

      return Math.max(...allData);
    },
  },
  mounted() {
    // this.generateArc();
    // this.draw();
    this.data = d3.csvParse(
      "name,score1,score2,score3,date\njohn,100,80,70,25-Apr-07\njack,50,60,70,26-Apr-07\njill,120,130,140,27-Apr-07\njane,150,140,100,30-Apr-07\nian,190,140,70,1-May-07\nrose,140,170,70,4-May-07\n"
    );
    console.log(this.data);
    this.drawD3();
  },
  methods: {
    drawD3() {
      var margin = {
        top: 50,
        right: 50,
        bottom: 70,
        left: 70,
      };
      var width = 600 - margin.left - margin.right;
      var height = 300 - margin.top - margin.bottom;

      console.log(this.data);
      const parseTime = d3.timeParse("%d-%b-%y");
      const parseTimeYear = d3.timeFormat("%a %d");

      // 2. Set x and y scales
      var yScale = d3
        .scaleLinear()
        .range([height, 0])
        .domain([0, this.maxValue]);

      var xScale = d3
        .scaleBand()
        .domain(this.data.map((d) => d.name))
        .range([0, width])
        .paddingInner(0.2)
        .paddingOuter(0.2);

      const xScaleTime = d3
        .scaleTime()
        .domain(
          d3.extent(this.data, function (d) {
            return parseTime(d.date);
          })
        )
        .range([0, width]);

      //3. Creating the Chart Axes
      // const x = d3
      //     .scaleTime()
      //     .domain(
      //         d3.extent(data, function (d) {
      //             return parseTime(d.date);
      //         })
      //     )
      //     .rangeRound([0, width]);

      // 3. Create svg object
      var svg = d3
        .select(".bar-chart")
        .append("svg")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom)
        .append("g")
        .attr("transform", `translate(${margin.left}, ${margin.top})`);

      // parseInt data
      this.data.forEach(function (d) {
        d.score1 = parseInt(d.score1);
        d.score2 = parseInt(d.score2);
        d.score3 = parseInt(d.score3);
      });

      // Use JS to sort array by score
      // this.data = this.data.sort(function (a, b) {
      //     return a.score1 - b.score2
      // });

      //scale axis based on data
      // xScale.domain(this.data.map(d => d.name));
      // yScale.domain([0, this.maxValue]);

      var barWidth = width / this.data.length / 3;

      var bar = svg
        .selectAll(".bar1")
        .data(this.data, (d) => d.score1)
        .enter()
        .append("g")
        .attr("transform", (d, i) => `translate(${i * 3 * barWidth + 25}, 0)`)
        .attr("class", "bar1");

      var bar2 = svg
        .selectAll(".bar2")
        .data(this.data, (d) => d.score2)
        .enter()
        .append("g")
        .attr(
          "transform",
          (d, i) => `translate(${barWidth + i * barWidth * 3 + 20},0)`
        )
        .attr("class", "bar2");

      //4. Creating a Line
      const line = d3
        .line()
        .x(function (d) {
          return xScaleTime(parseTime(d.date));
        })
        .y(function (d) {
          return yScale(d.score1);
        });

      // var bar3 = svg.selectAll('.bar3')
      //     .data(this.data, d => d.score3)
      //     .enter().append('g')
      //     .attr('transform', (d, i) => `translate(${(barWidth * 2) + (i * barWidth * 3) - 3},0)`)
      //     .attr('class', 'bar3');

      // Append a graph to each bar 'g' setting the width and height
      bar
        .append("rect")
        .attr("y", function (d) {
          return yScale(d.score1);
        })
        .attr("width", barWidth - 5)
        .attr("height", (d) => height - yScale(d.score1))
        .style("fill", "#99466D")
        .attr("rx", "15");

      bar2
        .append("rect")
        .attr("y", (d) => yScale(d.score2))
        .attr("width", barWidth - 5)
        .attr("height", (d) => height - yScale(d.score2))
        .style("fill", "#06909C")
        .attr("rx", "15");

      //6. Appending a path to the Chart
      svg
        .append("path")
        .datum(this.data)
        .attr("fill", "none")
        .attr("stroke", "#FEC998")
        .attr("stroke-width", 10)
        .attr("d", line);

      // bar3.append('rect')
      //     .attr('y', d => yScale(d.score3))
      //     .attr('width', barWidth - 5)
      //     .attr('height', d => height - yScale(d.score3))
      //     .style('fill', 'blue')
      //     .attr('rx', '15')

      // Add text to each bar graph
      // text-anchor middle is the svg text equivalent to transform: translate(-50%, -50%) for regular CSS divs
      // bar.append('text')
      //     .text(d => d.score1)
      //     .attr('text-anchor', 'middle')
      //     .attr('x', barWidth / 2)
      //     .attr('y', d => yScale(d.score1) - 5);

      // bar2.append('text')
      //     .text(d => d.score2)
      //     .attr('text-anchor', 'middle')
      //     .attr('transform', d => `translate(${barWidth / 2},${yScale(d.score2) - 5})`)

      // bar3.append('text')
      //     .text(d => d.score3)
      //     .attr('text-anchor', 'middle')
      //     .attr('transform', d => `translate(${barWidth / 2},${yScale(d.score3) - 5})`)

      // Add X axis at bottom of chart (we must do this at bottom after data has been appended to svg)
      svg
        .append("g")
        .attr("transform", `translate(0, ${height + 10})`)
        .call(d3.axisBottom(xScale).tickSizeOuter(0).tickSizeInner(0))
        .call((g) => g.select(".domain").remove())
        .selectAll("text")
        .attr("color", "#00585F")
        .attr("class", "uppercase font-bold text-[15px]");

      // Add y axis with label
      // svg.append('g')
      //     .call(d3.axisLeft(yScale));

      // Add chart title
      svg
        .append("text")
        .text("Chart: Survey results")
        .style("font-size", "20px")
        .attr(
          "transform",
          `translate(${0 - margin.left}, ${0 - margin.top / 2})`
        );

      //   Add chart x axis label
      // svg.append('text')
      //     .text('Participants')
      //     .attr('text-anchor', 'middle')
      //     .attr('transform', `translate(${width / 2}, ${height + (margin.bottom / 2)})`);

      // Add chart y axis label
      // svg.append('text')
      //     .text('Points scored')
      //     .attr('transform', 'rotate(-90)')
      //     .attr('text-anchor', 'middle')
      //     .attr('x', 0 - (height / 2))
      //     .attr('y', 0 - (margin.left / 2))
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
