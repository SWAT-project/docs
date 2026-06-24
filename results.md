---
title: Benchmark Results
nav_order: 4
---

# SWAT SV-COMP Benchmark Results
{: .no_toc }

Nightly [benchexec](https://github.com/sosy-lab/benchexec) runs of the latest SWAT against the
SV-COMP Java verification tasks, under the official competition limits (15 min / 15 GB / 4 cores).
The chart shows the SV-COMP **score** of each full run per channel (**dev** and **main**); only the
last 7 runs per channel are kept. Click a run below to open its full results table (with per-task logs).
{: .fs-5 .fw-300 }

<div id="score-plot" style="width:100%; height:460px; margin:1rem 0;"></div>
<p id="rstatus" style="opacity:.7;">Loading score history…</p>
<div id="runs-list" style="margin-top:1rem;"></div>

<script src="https://cdn.plot.ly/plotly-2.35.2.min.js" charset="utf-8"></script>
<script>
(function () {
  var p = window.location.pathname, m = p.match(/^(.*\/docs\/)/);
  var BASE = m ? m[1] : p.replace(/[^/]*$/, "");
  var ASSETS = BASE + "assets/results/";
  var COLOR = { dev: "#4c9be8", main: "#2ecc71" };
  var statusEl = document.getElementById("rstatus");

  fetch(ASSETS + "manifest.json", { cache: "no-store" })
    .then(function (r) { if (!r.ok) throw new Error(r.status); return r.json(); })
    .then(function (data) {
      var runs = (data.runs || []);
      if (!runs.length) { statusEl.textContent = "No runs published yet."; return; }
      statusEl.textContent = "";

      var channels = [];
      runs.forEach(function (r) { if (channels.indexOf(r.channel) < 0) channels.push(r.channel); });

      var traces = channels.map(function (ch) {
        var rs = runs.filter(function (r) { return r.channel === ch; });
        return {
          name: ch, mode: "lines+markers", type: "scatter",
          x: rs.map(function (r) { return r.generated || r.date; }),
          y: rs.map(function (r) { return r.score; }),
          line: { color: COLOR[ch] || undefined, width: 2 },
          marker: { size: 8 },
          text: rs.map(function (r) {
            return r.short + " · correct " + r.correct + "/" + r.total +
                   " (✗true " + r.incorrect_true + ", ? " + r.unknown + ")" +
                   (r.profile === "test" ? " [test]" : "");
          }),
          hovertemplate: "<b>%{x|%Y-%m-%d}</b><br>score %{y}<br>%{text}<extra>" + ch + "</extra>"
        };
      });

      var fg = "#bdc1c6", grid = "rgba(128,128,128,0.25)";
      Plotly.newPlot("score-plot", traces, {
        margin: { t: 20, r: 20, b: 50, l: 60 },
        paper_bgcolor: "rgba(0,0,0,0)", plot_bgcolor: "rgba(0,0,0,0)",
        font: { color: fg },
        xaxis: { title: "run", type: "date", gridcolor: grid, zeroline: false },
        yaxis: { title: "SV-COMP score", gridcolor: grid, zerolinecolor: grid },
        legend: { orientation: "h", y: 1.12 },
        hovermode: "closest"
      }, { responsive: true, displayModeBar: false });

      // drill-down list (newest first) per channel
      var html = "";
      channels.forEach(function (ch) {
        var rs = runs.filter(function (r) { return r.channel === ch; }).slice().reverse();
        html += "<h3>" + ch + "</h3><ul>";
        rs.forEach(function (r) {
          html += '<li><a href="' + BASE + r.path + '" target="_blank" rel="noopener">' +
                  r.date + " · " + r.short + "</a> — score <b>" + r.score + "</b>, correct " +
                  r.correct + "/" + r.total + (r.profile === "test" ? " <em>(test)</em>" : "") + "</li>";
        });
        html += "</ul>";
      });
      document.getElementById("runs-list").innerHTML = html;
    })
    .catch(function (e) { statusEl.textContent = "Could not load results manifest (" + e + ")."; });
})();
</script>
