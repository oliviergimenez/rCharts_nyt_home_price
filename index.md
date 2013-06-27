---
title: rCharts NYT Interactive Home Price
author: Timely Portfolio  (all credit to NYTimes)
github: {user: timelyportfolio, repo: rCharts_nyt_home_price, branch: "gh-pages"}
framework: bootstrap
mode: selfcontained
widgets: "nyt_home"
highlighter: prettify
hitheme: twitter-bootstrap
lead : >
  NYT's Home Price Visualization with rCharts
---

# Disclaimer and Attribution

**I claim absolutely no credit for this visualization, which I consider one of the most instructive I have ever seen.  If anybody believes this to be not fair use, I will take it down immediately.  I am implicitly assuming approval for this due to the interview http://datastori.es/data-stories-22-nyt-graphics-and-d3-with-mike-bostock-and-shan-carter/.** 

---
# Another Favorite from NYT
  


---
# in rCharts

<h1>Housing’s Rise and Fall in 20 Cities</h1>
<div id="interactiveFreeFormMain">
<link rel="stylesheet" type="text/css" href="http://graphics8.nytimes.com/newsgraphics/2013/05/28/case-shiller/2a8c44bdaf0703a2361add917782a6dd88c7d81e/base.css">
<!--[if lte IE 8]>
  <img src="http://graphics8.nytimes.com/newsgraphics/2013/05/28/case-shiller/2a8c44bdaf0703a2361add917782a6dd88c7d81e/fallback.png" alt="chart" width="970">
<![endif]-->
<!--[if gt IE 8]><!-->
<div class="g-shell">
<div class="g-tooltip">
<span class="g-playername-tooltip"></span>
<span class="g-dltype-tooltip"></span>
<span class="g-salary-tooltip"></span>
</div>
<h5 class="g-intro-sentence">
If you bought
<!--<span class="g-selector g-price-selector"></span> -->
an average home in
<span class="g-selector g-city-selector"></span>
around
<span class="g-selector g-month-selector"></span>
<span class="g-selector g-date-selector"></span>
it would be worth
<span class="g-answer">24 percent less</span> today.
</h5>
<div class="g-chart"></div>
<div class="g-table-container">
<div class="g-table-intro">
<h4>Behind the data </h4>
<p class="g-table-readin">The Standard & Poor&rsquo;s Case-Shiller Home Price Index for 20 major metropolitan areas is one of the most closely watched gauges of the housing market. The figures for April were released June 25. Figures shown here are not seasonally adjusted or adjusted for inflation.</p>
</div>
<table class="g-table">
<tr>
<th class="g-proper-city">City</th>
<th class="g-yoy-chg" colspan="2">Year-over-year change</th>
<th class="g-peak-month-td" colspan="2">Since peak</th>
<th class="g-change-since-selected" colspan="2"> <span class="g-date-compare">Since Jan. 2000</span></th>
</tr>
</table>
</div>
</div>

```r
require(rCharts)

p1 <- rCharts$new()
p1$field("LIB", list(url = paste0(getwd(), "/libraries/widgets/nyt_home"), name = ""))
p1$field("templates", modifyList(p1$templates, list(script = "libraries/widgets/nyt_home/layouts/nyt_home.html")))
cat(noquote(p1$html()))
```

<script src="http://graphics8.nytimes.com/newsgraphics/2013/05/28/case-shiller/2a8c44bdaf0703a2361add917782a6dd88c7d81e/lib.js"></script>

<script>
(function() {

  var nytMonths = {"Jan": "Jan.",
                   "Feb": "Feb.",
                   "Mar": "March",
                   "Apr": "April",
                   "May": "May",
                   "Jun": "June",
                   "Jul": "July",
                   "Aug": "Aug.",
                   "Sep": "Sept",
                   "Oct": "Oct.",
                   "Nov": "Nov.",
                   "Dec": "Dec."
                    }

  d3.selection.prototype.moveToFront = function() {
    return this.each(function(){
      this.parentNode.appendChild(this);
    });
  };

  var margin = {top: 20, right: 10, bottom: 1, left: 10},
      width = 950 - margin.left - margin.right,
      height = 500 - margin.top - margin.bottom;

  var formatYear = d3.time.format("'%y"),
      format = d3.time.format("%m/%d/%y"),
      bisectDate = d3.bisector(function(d) { return d.date; }).left,
      formatToMonth = d3.time.format("%B %Y"),
      formatPercent = d3.format(".1%"),
      formatVal = d3.format(".1f"),
      formatMonAbb = d3.time.format("%b. %Y"),
      formatMonAbb2 = d3.time.format("%b"),
      formatMonthOnly = d3.time.format("%b"),
      formatYearOnly = d3.time.format("%Y"),
      formatPercentChange = d3.format("+%"),
      formatChangeInWords = function (val) {
        var change =  100 - formatNumberWords(val);
        var str = change > 0 ? " percent less" : " percent more";
        if (change < 1 && change > -1 ) {
          return "about the same";
        };
        return (Math.abs(change) + str);
      },
      formatNumberWords = d3.format("0f"),
      formatPercentChange2 = function(d) { return d/100},
      formatFullYear = d3.time.format("%Y"),
      defaultCity = "SPCS20R",
      currentCity,
      hoverCity,
      currentDateIndex,
      defaultDateIndex = 39, //TODO Calculate it!
      currentPriceTier = "val";

  var formatAxes = function(d) {
    var adj = d - 100;
    if (adj == 0) { return "Starting price"; }
    if (adj < 0) { return Math.abs(adj) + "% less"; }
    if (adj > 0) { return Math.abs(adj) + "% more"; }
  };

  var x = d3.time.scale()
      .range([0, width]);

  var y = d3.scale.log()
      .range([height, 0]);

  var barScale = d3.scale.linear()
      .range([0,75])
      .domain([0,75])

  var line = d3.svg.line()
      .x(function(d) { return x(d.date); })
      .y(function(d) { return y(d.adjustedVal); })
      .defined(function(d) { return !isNaN(d.adjustedVal); })

  var slider = d3.select(".g-chart").append("div")
      .attr("class", "g-slider")
      .style("width", width + "px")
      .style("margin-left", margin.left + "px");

  slider.append("div")
      .attr("class", "g-slider-tray")

  var sliderFill = slider.append("div")
      .attr("class", "g-slider-fill")

  var sliderHandle = slider.append("div")
      .attr("class", "g-slider-handle");

    sliderHandle.append("img")
      .attr("class", "g-slider-handle-icon")
      .attr("src", "http://graphics8.nytimes.com/newsgraphics/2013/05/28/case-shiller/2a8c44bdaf0703a2361add917782a6dd88c7d81e/handle@2x.png")

  var svg = d3.select(".g-chart").append("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom)
      .attr("class", "g-svg")
    .append("g")
      .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

  var xAxis = d3.svg.axis()
      .scale(x)
      .orient("top")
      .ticks(d3.time.years)
      .tickSize(4)
      .tickPadding(2)
      .tickFormat(formatYear);

  var yAxis = d3.svg.axis()
      .scale(y)
      .orient("left")
      .ticks(7)
      .tickValues( [ 35, 50, 75, 100, 150, 200, 250] )
      .tickSize(-width - margin.left - margin.right)
      .tickFormat(formatAxes);

  queue()
      .defer(d3.csv, "http://graphics8.nytimes.com/newsgraphics/2013/05/28/case-shiller/2a8c44bdaf0703a2361add917782a6dd88c7d81e/lookup2.csv")
      .defer(d3.csv, "http://graphics8.nytimes.com/newsgraphics/2013/05/28/case-shiller/2a8c44bdaf0703a2361add917782a6dd88c7d81e/case-shiller-tiered2.csv")

      .await(ready)

  function ready(err, lookup, data) {

    data.forEach(function(d) {
      d.date = new Date(+d.year, +d.month -1, 1 )
      d.val = +d.val;
      d.high = +d.high;
      d.low = +d.low;
      d.mid = +d.mid;
    });

    cityNameByKey = {};

    lookup.forEach(function(d) {
      cityNameByKey[d.code] = d.city;
    });

    x.domain(d3.extent(data, function(d) { return d.date; }));
    y.domain([35, 300]);

    nested = d3.nest()
        .key(function(d) { return d.citycode; })
        .entries(data);

    nested.forEach(function(city) {
      city.updatedMonth = city.values[nested[1].values.length-1].date;
      city.mostRecentValue = city.values[nested[1].values.length-1].val;
      city.maxVal = d3.max(city.values, function(d) { return d.val;});
      city.peakidx = city.values.map(function(d) { return d.val}).indexOf(city.maxVal);
      city.peakMonth = city.values.map(function(d) { return d.date})[city.peakidx];
      city.yearAgoDate = city.values[nested[1].values.length-13].date;
      city.yearAgoVal = city.values[nested[1].values.length-13].val;
      city.yearOnYearChange = (city.mostRecentValue - city.yearAgoVal) / city.yearAgoVal;
      city.changeFromPeak = (city.mostRecentValue - city.maxVal)/(city.maxVal);
      city.changeSinceBeginning = (city.mostRecentValue - city.values[0].val)/city.values[0].val;
      city.proper = cityNameByKey[city.key];
    });

    nested.sort(function(a,b) {
      return b.yearOnYearChange - a.yearOnYearChange;
    });

    var indexFromMouseX = d3.scale.linear()
        .domain([0, width])
        .rangeRound([1, nested[1].values.length - 1])
        .clamp(true)

    svg.append("rect")
        .attr("class", "background")
        .attr("width", width)
        .attr("height", height);

    svg.append("g")
        .attr("class", "x axis")
        .attr("transform", "translate(0," + 5 + ")")
        .call(xAxis)
      .selectAll("g")
        .classed("minor", function(d, i) { return d.getFullYear() % 1; })
        .filter(".minor")
      .select("line")
        .attr("y2", 2);

        d3.selectAll(".x.axis text").attr("dy", 20)

    // lines
    var linecontainer = svg.append("g")
        .classed("g-linecontainer", true);

    var screeny = svg.append("rect")
        .classed("g-screen", true)
        .attr("width", 0)
        .attr("height", height)
        .attr("x", 0 - margin.left);

    var yAxisMarker = svg.append("g")
        .attr("class", "y axis")
        .attr("transform", "translate(" + (0) + ",0)")
        .call(yAxis)
      .selectAll("g")
      .classed("minor", true)

        // .classed("minor", function(d,i) { return i !== 0; })
        .classed("g-baseline", function(d) { return d == 100 })
        // .classed("g-first", function(d,i) { return i == 0; })
      .select("text")
        .attr("x", 30)
        .attr("y", -5)
        .attr("dy", null);

    var lines = linecontainer.selectAll("path")
        .data(nested)
      .enter().append("path")
        .attr("class", (function(d) { return ( "g-city-line " + d.key)}) )
        .classed("g-highlight-line", function(d,i) { return d.key == currentCity});

    var hoverbar = svg.append("rect")
        .classed("g-hover-bar", true)
        .attr("width", 2)
        .attr("height", height)
        .attr("x", 0 - margin.left);

    var focus = svg.append("g")
        .attr("class", "focus")
        .attr("transform", "translate(" + x(nested[0].values[0].date) + ", " + y(100) + ")")

    focus.append("circle")
        .attr("r", 5)

    focus.append("text")
        .attr("x", 9)
        .attr("dy", ".35em");

    var currentMonth = slider.append("div")
        .classed("g-current-month", true);

    var mostRecentMonth = slider.append("div")
        .classed("g-most-recent-month", true)
        .html("...to <b>" + "April 2013" + "</b>");

    var endpoint = svg.append("g")
        .attr("class", "g-endpoint")
        .attr("transform", "translate(" + x(nested[0].values[nested[0].values.length-1].date) + ", " + y(100) + ")")

    endpoint.append("circle")
        .attr("r", 5);

    endpoint.append("text")
        .classed("g-end-label g-selected-city-endpoint-label", true)
        .attr("y", -45)
        .attr("x", - 0)
        .text("U.S. 20-city index")

    endpoint.append("text")
        .classed("g-end-label g-selected-city-endpoint-value", true)
        .attr("y", -20)
        .attr("x", - 0)
        .text("+12%")

    //
    //  Init Table
    //
    function initTable() {
      var rows = d3.select(".g-table").selectAll(".table-row")
          .data(nested)
        .enter()
          .append("tr")
          .attr("class", function(d) { return "g-table-row " + d.key + "-row"; })
          .classed("g-selected-row", function(d) { return d.key == "SPCSIND20"});

      var cityNames = rows.append("td")
          .text(function(d) { return d.proper; })
          .classed("g-proper-city", true);

      var yearlyChange = rows.append("td")
          .classed("g-num-td", true)
          .text(function(d) { return formatPercentChange(d.yearOnYearChange); });

      var yearlyChangeBarTd = rows.append("td")
          .classed("g-bar-td", true);

      var yearChangeBarContainer = yearlyChangeBarTd.append("div")
          .classed("g-bar-container", true);

      var changeBar = yearChangeBarContainer.append("div")
          .classed("g-yoy-bar", true)
             .style("width", function(d) {
                var chg = 100 * d.yearOnYearChange;
                return Math.abs(barScale(chg)).toString() + "px";
            })
            .style("left", function(d) {
               var chg = 100 * d.yearOnYearChange;
              if (chg<0) {
                var nudge = Math.abs(barScale(chg));
                return (barScale.domain()[1] - nudge).toString() + "px";
              }
              else {
                return barScale.domain()[1].toString() + "px";
              }
           });

      var zeroMarker3 = yearChangeBarContainer.append("div")
          .classed("g-zeromarker", true)
          .style("left", barScale.domain()[1] + "px");

      var peakMonthTd = rows.append("td")
          .classed("g-num-td", true)
          .text(function(d) {
            return formatPercentChange(d.changeFromPeak);
          });

      var peakMonthBarTd = rows.append("td")
          .classed("g-bar-td", true);

      var peakMonthBarContainer = peakMonthBarTd.append("div")
          .classed("g-bar-container", true);

      var changeBar = peakMonthBarContainer.append("div")
          .classed("g-yoy-bar", true)
          .style("width", function(d) {
             var chg = 100 * d.changeFromPeak;
             return Math.abs(barScale(chg)).toString() + "px";
          })
          .style("left", function(d) {
             var chg = 100 * d.changeFromPeak;
            if (chg < 0) {
              var nudge = Math.abs(barScale(chg));
              return (barScale.domain()[1] - nudge).toString() + "px";
            }
            else {
              return barScale.domain()[1].toString() + "px";
            }
          });

      var zeroMarker2 = peakMonthBarContainer.append("div")
          .classed("g-zeromarker", true)
          .style("left", barScale.domain()[1] + "px");

      var changeSinceSelectedTd = rows.append("td")
          .classed("g-change-since-selected-td g-num-td", true);

      var changeSelectedBarTD = rows.append("td")
          .classed("g-live-change-bar-td", true);

      var liveBarContainer = changeSelectedBarTD.append("div")
          .classed("g-live-bar-container", true);

      var liveChangeBar = liveBarContainer.append("div")
          .classed("g-live-bar", true);

      var zeroMarker = liveBarContainer.append("div")
          .classed("g-zeromarker", true)
          .style("left", barScale.domain()[1] + "px");
    }

    initTable();

    //
    // Init Dropdowns
    //
    var dropDowns = d3.map();
    dropDowns.set("price", {
      class:".g-price-selector",
      data:[["an average priced", "val"], ["a low priced", "low"], ["a medium priced", "mid"], ["a high priced", "high"]],
      change: selectPrice

    });
    dropDowns.set("city", {
      class:".g-city-selector",
      data:nested.map(function(d) {

        if(d.key == "SPCS20R") {
          return ["a major city", d.key ];
        }
        else {
          return [cityNameByKey[d.key], d.key];

        }

      }).sort(),
      change: selectCity
    });
    dropDowns.set("date", {
      class:".g-date-selector",
      data:nested[0].values.map(function(d, i) { return [formatMonAbb(d.date), i]; }),
      change: selectDateWithIndex
    });

    dropDowns.forEach(function(key, d) {
      d.select = d3.select(d.class)
          .datum(d.data)
        .append("select")
          .on("change", function() { d.change(this.value); });

      d.select.selectAll("option")
          .data(function(d) { return d; })
        .enter().append("option")
          .text(function(d) { return d[0]; })
          .attr("value", function(d) { return d[1]; })
    });

    // TODO make this find the 10-year index instead of hardcode 41 and animate it
    // selectDateWithIndex(1);
    selectCity(defaultCity);
    d3.transition()
        .duration(1000)
        .ease("out")
        .tween("intro", function() {
          return function(t) {
              selectDateWithIndex(Math.round(t * defaultDateIndex));
            };
        })

    //
    // Mouse Events
    //
    svg.on("mousemove", hover);
    svg.on("mouseout", function() {
      unHighlightCity();

      d3.select(".g-tooltip")
          .classed("g-hovering-tooltip", false)
    })

    slider.call(d3.behavior.drag()
        .on("dragstart", dragstarted)
        .on("drag", dragged));

    svg.on("click", function(d) {
      selectCity(hoverCity);
    });

    d3.selectAll(".g-table-row").on("click", function(d) {
      selectCity(d.key);
    }).on("mouseover", function(d) {
      highlightCity(d.key)
    }).on("mouseout", function(d) {
      unHighlightCity(d.key)
    });

    function selectPrice(priceKey) {
      if (currentPriceTier !== priceKey) {
        dropDowns.get("price").select.node().value = priceKey;
        currentPriceTier = priceKey;
        adjustData();
      }
    }

    function selectDateWithIndex(dateIndex) {
      if (currentDateIndex !== dateIndex) {
        currentDateIndex = dateIndex;
        dropDowns.get("date").select.node().value = currentDateIndex;
        adjustData();
      }
    }

    function selectCity(cityKey) {
      cityKey = cityKey ? cityKey : defaultCity;
      if (currentCity !== cityKey) {
        if (isNaN(nested.filter(function(d) { return d.key == cityKey; })[0].values[0][currentPriceTier])) selectPrice("val");
        dropDowns.get("price").select.property("disabled", isNaN(nested.filter(function(d) { return d.key == cityKey; })[0].values[0].low));
        dropDowns.get("city").select.node().value = cityKey;

        currentCity = cityKey;
        d3.selectAll(".g-city-line").classed("g-highlight-line", function(d) { return d.key == cityKey; });
        d3.selectAll(".g-table-row").classed("g-selected-row", function(d) { return d.key == cityKey; });
        d3.select("." + cityKey).moveToFront();
        d3.select(".g-selected-city-endpoint-label").text(cityNameByKey[cityKey])

        updateLabels();
      }
    }

    function updateLabels() {
      var obj = nested.filter(function(d) { return d.key == currentCity})[0]
      var lastVal = obj.values[obj.values.length-1].adjustedVal;
      d3.select(".g-answer").text(formatChangeInWords(lastVal));
      d3.select(".g-selected-city-endpoint-value").text(formatChangeInWords(lastVal))
      d3.select(".g-endpoint")
          .attr("transform", "translate(" + x(obj.values[obj.values.length-1].date) + ","+ y(lastVal)+")" );
    }

    function highlightCity(cityKey) {
      if (hoverCity !== cityKey) {

        hoverCity = cityKey;
        d3.selectAll(".g-city-line").classed("g-hover-line", function(d) { return d.key == cityKey; });
        d3.selectAll(".g-table-row").classed("g-hover-row", function(d) { return d.key == cityKey; });
      }
    }

    function unHighlightCity() {
      if (hoverCity !== null) {
        hoverCity = null;
        d3.selectAll(".g-city-line").classed("g-hover-line", false);
        d3.selectAll(".g-table-row").classed("g-hover-row", false);

        d3.select(".g-tooltip")
            .classed("g-hovering-tooltip", false)
      }
    }

    function dragstarted() {
      unHighlightCity();
      dragged();
      d3.event.sourceEvent.preventDefault();
    }

    function dragged() {
      selectDateWithIndex(indexFromMouseX(d3.mouse(svg.node())[0]));
    }

    function hover() {
      var m = d3.mouse(svg.node()),
          index = indexFromMouseX(m[0]),
          pct = y.invert(m[1]);

      var closestDifference = Infinity,
          closestIndex = -1,
          currentDifference;

      nested.forEach(function(d, i) {
        currentDifference = Math.abs(pct - d.values[index].adjustedVal);
        if (currentDifference < closestDifference) {
          closestDifference = currentDifference;
          closestIndex = i;
        }
      });

      if (closestDifference < 30) {
        highlightCity(nested[closestIndex].key);

        var m = d3.mouse(svg.node());
        d3.select(".g-tooltip")
            .classed("g-hovering-tooltip", true)
            .style("left", m[0] + "px")
            .style("top", m[1] + 70 + "px")
            .text(cityNameByKey[nested[closestIndex].key]);

        svg.style("cursor", "pointer");
      } else {
        svg.style("cursor", "inherit");

        unHighlightCity();
      }
    }

    function adjustData() {
      nested.forEach(function(d) {
        d.values.forEach(function(j) {
          j.adjustedVal = 100 * (j[currentPriceTier] / d.values[currentDateIndex][currentPriceTier]);
        })
      });

      lines.attr("d", function(d) { return line(d.values); } );

      var thisDate = nested[0].values[currentDateIndex].date,
          xPos = x(thisDate)

      var newpos = xPos < 100 ? 60 : (xPos - 40);
      // when xpos is less than 50, freeze it at 100
      d3.selectAll(".y.axis .tick.minor text")
        .attr("transform", "translate(" + newpos + ",0)")

      focus.attr("transform", "translate(" + (xPos - 2) + "," + y(100) + ")");

      d3.select(".g-date-compare")
          .text(function(d) { return ("Since " + nytMonths[formatMonthOnly(thisDate)] + " " + formatYearOnly(thisDate)   )})

      d3.select(".g-screen").attr("width", xPos + 10 );

      if (xPos) {};

      currentMonth
          .style("left", Math.min(Math.max(xPos - 160, 0), width - 320) + "px")
          .html(function(d) {  return "Price changes from <b>" + nytMonths[formatMonAbb2(thisDate)] + " " +  formatYearOnly(thisDate) +  "</b>..." });

      sliderFill.style("width",  (width - x(thisDate)) + "px" );

      sliderHandle.style("left", xPos + "px")

      d3.select(".g-hover-bar").attr("x", xPos - 2 );

      d3.selectAll(".g-change-since-selected-td")
          .text(function(d) {
            var current = d.values[d.values.length-1].val
            var compareVal = d.values[currentDateIndex].val;
            var chg = (current - compareVal ) / compareVal;
            return formatPercentChange(chg);
          });

      var thisCityObj = nested.filter(function(d) { return d.key == currentCity})[0];

      updateLabels();

      d3.selectAll(".g-live-bar")
          .style("width", function(d) {
              var current = d.values[d.values.length-1].val
              var compareVal = d.values[currentDateIndex].val;
              var chg = 100 * (current - compareVal ) / compareVal;
              return Math.abs(barScale(chg)).toString() + "px";
          })
          .style("left", function(d) {
            var current = d.values[d.values.length-1].val
            var compareVal = d.values[currentDateIndex].val;
            var chg = 100 * (current - compareVal ) / compareVal;

            if (chg < 0) {
              var nudge = Math.abs(barScale(chg));
              return (barScale.domain()[1] - nudge).toString() + "px";
              // what fraction of the width in pixels is this?
              var nudge = starterBarWidth * Math.abs(chg)/100;
              return (starterBarWidth - nudge).toString() + "px";
            }
            else {
              return barScale.domain()[1].toString() + "px";
            }
         });
    }
  }

})()</script>
