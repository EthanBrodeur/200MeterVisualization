<head>
  <meta charset="utf-8">
  <title>200 Meter Visualization</title>
  <script src="https://d3js.org/d3.v4.js"></script>
  <style>
  body {
    padding-top: 2vh;

  }
  svg {
    display:block;
    margin:auto;
  }
  p {
    font-family: serif;
    font-size: 20px;
    padding-left: 4vw;
    padding-right: 4vw;
  }
  div.tooltip {   
    position: absolute;         
    text-align: center;         
    width: 220px;                    
    height: 70px;                   
    padding: 2px;               
    font: 16px sans-serif;      
    background: white;    
    border: 30px;   
    stroke: black;     
    border-radius: 8px;         
    pointer-events: none;           
  }
  .x-axis-title {
    font-size: 20px;
    font-weight: bold;
  }
  .y-axis-title {
    font-size: 20px;
    font-weight: bold;

  }
  .axis {
    font: 20px sans-serif; 
  }

  label {
    font-weight: bold;
    font-size: 30px;
    float: left;
    margin-left: 3vw;
  }
  .chart-title{
    text-decoration: underline;
  }
</style>
</head>
<body>
  <label><input type="checkbox">Flip Axes</label>
  <script type="text/javascript">
    let path = "./clean200meterdata.csv";
    var margin = {top: 60, right: 130, bottom: 100, left: 90},
    width = 1000,
    height = 950;

    var canvas = d3.select("body")
    .append("svg")
    .attr("width", width)
    .attr("height", height);

    var tooltip = d3.select("body").append("div")   
    .attr("class", "tooltip")               
    .style("opacity", 0);
    canvas.append("text")
    .attr("class", "additional")
    .attr("x", width/2)
    .attr("y", height*.98)
    .text("")
    .attr("font-size", "18px")
    .attr("text-anchor", "middle")

    canvas.append("text")
    .attr("class", "chart-title")
    .attr("x", width/2)
    .attr("y", 25)
    .text("World's Best 200 Meter Runners Who Excelled Before Age 21")
    .attr("font-size", "35px")
    .attr("text-anchor", "middle");

    drawLegend();
    // Axis Titles. Code inspired by https://bl.ocks.org/d3noob/23e42c8f67210ac6c678db2cd07a747e
    canvas.append("text")             
    .attr("transform",
      "translate(" + (width/2) + " ," + 
      (height*.95) + ")")
    .attr("class", "x-axis-title")
    .style('text-anchor', 'middle')
    .text("Best Time Before Turning 21");
    canvas.append("text")
    .attr("transform", "rotate(-90)")
    .attr("y", width*.01)
    .attr("class", "y-axis-title")
    .attr("x",0 - (height / 2))
    .attr("dy", "1em")
    .style("text-anchor", "middle")
    .text("Post 21 Best");      

    d3.csv(path, function(data){
      data.forEach(function(d){
        d.Time = +d.Time;
        d.OldTime = +d.OldTime;
      });

      let fastestYoung = d3.min(data, function(d) { return d.Time});
      let slowestYoung = d3.max(data, function(d) { return d.Time});
      let fastestOld = d3.min(data, function(d) { return d.OldTime});
      let slowestOld = d3.max(data, function(d) { return d.OldTime});

      var x = d3.scaleLinear()
      .domain([fastestYoung,slowestYoung])
      .range([width-margin.right, margin.left]);
      var y = d3.scaleLinear()
      .domain([fastestOld, slowestOld])
      .range([margin.top, height-margin.bottom]);

      var xaxis = d3.axisBottom(x);
      canvas.append("g")
      .attr("transform", "translate("+ 0 +","+ (height-margin.bottom) +")")
      .attr("class", "axis")
      .call(xaxis);

      var yaxis = d3.axisLeft(y);
      canvas.append("g")
      .attr("transform", "translate("+ margin.left +","+ 0 +")")
      .attr("class", "axis")
      .call(yaxis);

      canvas.selectAll("circle")
      .data(data)
      .enter()
      .append("circle")
      .attr("class", "data")
      .attr("cx", function(d) {
        return (x(d.Time));
      })
      .attr("cy", function(d) {
        return (y(d.OldTime));
      })
      .attr("fill", function(d) {return (color(d))})
      .attr("r", 6)
      .on("mouseover", function (d) {drawToolTip(d)})
      .on("mouseout", function (d) {clearToolTip(d)});

      d3.selectAll("axis>.tick>text")
      .style("font-size","20px");

      d3.selectAll("input")
      .on("change", change);  

      var timeoutLength = setTimeout(function() {
        d3.select("input").property("checked", false).each(change);
      }, 500);

      function change() {
        clearTimeout(timeoutLength);

        canvas.selectAll("g.axis").remove();
        if (this.checked){
          // set up new scales (flipped)
          var newX = d3.scaleLinear()
          .domain([fastestOld, slowestOld])
          .range([width-margin.right, margin.left]);
          var newY = d3.scaleLinear()
          .domain([fastestYoung,slowestYoung])
          .range([margin.top, height-margin.bottom]);

          var newxaxis = d3.axisBottom(newX);
          canvas.append("g")
          .attr("transform", "translate("+ 0 +","+ (height-margin.bottom) +")")
          .attr("class", "axis")
          .call(newxaxis);
          var newyaxis = d3.axisLeft(newY);
          canvas.append("g")
          .attr("transform", "translate("+ margin.left +","+ 0 +")")
          .attr("class", "axis")
          .call(newyaxis);

          var transition = canvas.transition().duration(150),
          delay = function(d,i) { return i * 50; };

          transition.selectAll("circle.data")
          .delay(delay)
          .attr("cx", function(d) { return newX(d.OldTime); })
          .attr("cy", function(d) { return newY(d.Time); });

          canvas.select("text.x-axis-title")
          .text("Post 21 Best");
          canvas.select("text.y-axis-title")
          .text("Best Time Before Turning 21");

        }
        else {
          newX = x;
          newY = y;
          newxaxis = xaxis;
          newyaxis = yaxis;
          canvas.append("g")
          .attr("transform", "translate("+ 0 +","+ (height-margin.bottom) +")")
          .attr("class", "axis")
          .call(xaxis);

          canvas.append("g")
          .attr("transform", "translate("+ margin.left +","+ 0 +")")
          .attr("class", "axis")
          .call(yaxis);

          var transition = canvas.transition().duration(150),
          delay = function(d, i) { return i * 50; };

          transition.selectAll("circle.data")
          .delay(delay)
          .attr("cx", function(d) { return newX(d.Time); })
          .attr("cy", function(d) { return newY(d.OldTime); });

          canvas.select("text.x-axis-title")
          .text("Best Time Before Turning 21");
          canvas.select("text.y-axis-title")
          .text("Post 21 Best");
        }
      }

    })

function drawLegend() {
  canvas.append("text")
  .attr("class", "legend")
  .attr("x", width*.97)
  .attr("y", height*.7)
  .text("Legend")
  .attr("font-size", "18px")
  .attr("font-weight", "bold")
  .attr("text-anchor", "end");
  canvas.append("text")
  .attr("class", "legend")
  .attr("x", width*.97)
  .attr("y", height*.74)
  .text("Improved after 21")
  .attr("font-size", "18px")
  .attr("text-anchor", "end");
  canvas.append("text")
  .attr("class", "legend")
  .attr("x", width*.97)
  .attr("y", height*.76)
  .text("Peaked before 21")
  .attr("font-size", "18px")
  .attr("text-anchor", "end");
  canvas.append("text")
  .attr("class", "legend")
  .attr("x", width*.97)
  .attr("y", height*.78)
  .text("Only Appear Before 21")
  .attr("font-size", "18px")
  .attr("text-anchor", "end");
  canvas.append("text")
  .attr("class", "legend")
  .attr("x", width*.97)
  .attr("y", height*.80)
  .text("Usain Bolt")
  .attr("font-size", "18px")
  .attr("text-anchor", "end");
  canvas.append("text")
  .attr("class", "legend")
  .attr("x", width*.97)
  .attr("y", height*.82)
  .text("Clarence Munyai")
  .attr("font-size", "18px")
  .attr("text-anchor", "end");

  canvas.append("circle")
  .attr("class", "legend-dot")
  .attr("cx", width*.987)
  .attr("cy", height*.735)
  .attr("r", 6)
  .attr("fill", "blue");
    canvas.append("circle")
  .attr("class", "legend-dot")
  .attr("cx", width*.987)
  .attr("cy", height*.755)
  .attr("r", 6)
  .attr("fill", "black");
    canvas.append("circle")
  .attr("class", "legend-dot")
  .attr("cx", width*.987)
  .attr("cy", height*.775)
  .attr("r", 6)
  .attr("fill", "yellow");
    canvas.append("circle")
  .attr("class", "legend-dot")
  .attr("cx", width*.987)
  .attr("cy", height*.795)
  .attr("r", 6)
  .attr("fill", "green");
    canvas.append("circle")
  .attr("class", "legend-dot")
  .attr("cx", width*.987)
  .attr("cy", height*.815)
  .attr("r", 6)
  .attr("fill", "red");

  canvas.append("rect")
  .attr("width", width*.22)
  .attr("height", height*.145)
  .attr("x", width*.78)
  .attr("y",height*.681)
  .attr("fill", "none")
  .attr("stroke", "black")
  .attr("opacity", .8);
}

function color(d) {
  if (d.Athlete == " Clarence Munyai"){
    return "red";
  }
  else if (d.Athlete == " Usain Bolt") {
    return "green";
  }
  else if (d.OldName == "None") {
        // athlete only has performance before age 21
        return "yellow";
      }

      else if (d.OldTime < d.Time) {
        return "blue";
      }
      else {
        return "black";
      }
    }

    function drawToolTip(d) {
      var string = "";
      if (d.OldName == "None"){
        string = d.Athlete + " ran a " + d.Time + " on " + d.Date + ". To date, he never bettered this performance.";
      }
      else{
        string = d.Athlete + " ran a " + d.Time + " before turning 21. He later ran " + d.OldTime + ", number " + d.OldRank + " in history.";
      }

      tooltip.html(string)
      .style("left", (d3.event.pageX) + "px")     
      .style("top", (d3.event.pageY ) + "px")
      .style("opacity", .8)
      .style("border", "2px solid black");
      if (d.OldName != "None"){
        let addString = "Pre 21 Best: " + d.Time + ", " + d.Rank + " all time, in " + d.Location + " at " + d.AgeYears + " years of age. \n Later, at age " + d.OldAgeYears + ", ran " + d.OldTime + " in " + d.OldLocation + ".";
        canvas.select("text.additional")
        .text(addString);  
      }
      else {
        let addString = "Pre 21 Best: " + d.Time + ", " + d.Rank + " all time, in " + d.Location + " at " + d.AgeYears + " years of age.";
        canvas.select("text.additional")
        .text(addString);        
      }
    }
    function clearToolTip(d) {
      tooltip.style("opacity", 0);    
      canvas.select("text.additional")
      .text("");
    }

  </script>

</body>
