<script>
var margin = {
    top: 50,
    right: 50,
    bottom: 70,
    left: 70
};
var width = 600 - margin.left - margin.right;
var height = 500 - margin.top - margin.bottom;

var color = d3.scaleOrdinal(d3.schemeCategory20b);

// 2. Set x and y scales
var xScale = d3.scaleBand().range([0, width]);
var yScale = d3.scaleLinear().range([height, 0]);

// 3. Create svg object
var svg = d3.select('body').append('svg')
    .attr('width', width + margin.left + margin.right)
    .attr('height', height + margin.top + margin.bottom)
    .append('g')
    .attr('transform', `translate(${margin.left}, ${margin.top})`)

// LOAD data - we are using csvParse because CodePen cannot load external files, normally we would fetch external file using .csv
var data = d3.csvParse('name,score1,score2,score3\njohn,100,80,70\njack,50,60,70\njill,120,130,140\njane,150,140,100\n');
// 4. Fetch and handle data
// d3.csv(DATA, function(data){
// handle error
// if(err) throw err;

// parseInt data
data.forEach(function (d) {
    d.score1 = parseInt(d.score1);
    d.score2 = parseInt(d.score2);
    d.score3 = parseInt(d.score3);
});
console.log(data)

// Use JS to sort array by score
data = data.sort(function (a, b) {
    return a.score1 - b.score1
});

//scale axis based on data
xScale.domain(data.map(d => d.name));
yScale.domain([0, 200]);

// Create new bar groups appending data and setting starting y index position (we use enter() join to create new 'g' for data point if non-existent)
var barWidth = (width / data.length) / 3; // bar height equidistant across graph height

var bar = svg.selectAll('.bar1')
    .data(data, d => d.score1)
    .enter().append('g')
    .attr('transform', (d, i) => `translate(${(i * 3) * barWidth + 5}, 0)`)
    .attr('class', 'bar1')

var bar2 = svg.selectAll('.bar2')
    .data(data, d => d.score2)
    .enter().append('g')
    .attr('transform', (d, i) => `translate(${barWidth + (i * barWidth * 3) + 1},0)`)
    .attr('class', 'bar2');

var bar3 = svg.selectAll('.bar3')
    .data(data, d => d.score3)
    .enter().append('g')
    .attr('transform', (d, i) => `translate(${(barWidth * 2) + (i * barWidth * 3) - 3},0)`)
    .attr('class', 'bar3');


// Append a graph to each bar 'g' setting the width and height
bar.append('rect')
    .attr('y', function (d) {
        return yScale(d.score1)
    })
    .attr('width', barWidth - 5)
    .attr('height', d => height - yScale(d.score1))
    .style('fill', color('score1'));

bar2.append('rect')
    .attr('y', d => yScale(d.score2))
    .attr('width', barWidth - 5)
    .attr('height', d => height - yScale(d.score2))
    .style('fill', color('score2'));

bar3.append('rect')
    .attr('y', d => yScale(d.score3))
    .attr('width', barWidth - 5)
    .attr('height', d => height - yScale(d.score3))
    .style('fill', color('score3'));

// Add text to each bar graph
// text-anchor middle is the svg text equivalent to transform: translate(-50%, -50%) for regular CSS divs
bar.append('text')
    .text(d => d.score1)
    .attr('text-anchor', 'middle')
    .attr('x', barWidth / 2)
    .attr('y', d => yScale(d.score1) - 5);

bar2.append('text')
    .text(d => d.score2)
    .attr('text-anchor', 'middle')
    .attr('transform', d => `translate(${barWidth / 2},${yScale(d.score2) - 5})`)

bar3.append('text')
    .text(d => d.score3)
    .attr('text-anchor', 'middle')
    .attr('transform', d => `translate(${barWidth / 2},${yScale(d.score3) - 5})`)


// Add X axis at bottom of chart (we must do this at bottom after data has been appended to svg)
svg.append('g')
    .attr('transform', `translate(0, ${height})`)
    .call(d3.axisBottom(xScale).tickSize(0));

// Add y axis with label
svg.append('g')
    .call(d3.axisLeft(yScale));

// Add chart title
svg.append('text')
    .text('Chart: Survey results')
    .style('font-size', '20px')
    .attr('transform', `translate(${0 - margin.left}, ${0 - (margin.top / 2)})`);

//   Add chart x axis label
svg.append('text')
    .text('Participants')
    .attr('text-anchor', 'middle')
    .attr('transform', `translate(${width / 2}, ${height + (margin.bottom / 2)})`);

// Add chart y axis label 
svg.append('text')
    .text('Points scored')
    .attr('transform', 'rotate(-90)')
    .attr('text-anchor', 'middle')
    .attr('x', 0 - (height / 2))
    .attr('y', 0 - (margin.left / 2))

// ==== Add legend
var legend = svg.selectAll('.legend')
    .data(color.domain())
    .enter()
    .append('g')
    .attr('class', 'legend')

var legendSpacing = 80;

legend.append('text')
    .text(d => d)
    .attr('transform', (d, i) => `translate(${i * legendSpacing}, ${height + (margin.bottom)})`)

legend.append('rect')
    .attr('width', legendSpacing / 2)
    .attr('height', 5)
    .style('fill', color)
    .attr('transform', (d, i) => `translate(${i * legendSpacing}, ${height + (margin.bottom - 25)})`)
</script>