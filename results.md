---
title: Benchmark Results
nav_order: 4
---

# SWAT SV-COMP Benchmark Results
{: .no_toc }

Nightly [benchexec](https://github.com/sosy-lab/benchexec) runs of the latest SWAT against the
SV-COMP Java verification tasks, under the official competition limits (15 min / 15 GB / 4 cores).
{: .fs-5 .fw-300 }

<p style="margin:1.5rem 0;">
  <a id="compare-link" class="btn btn-primary fs-5 mb-4 mb-md-0" target="_blank" rel="noopener">
    Open the latest SWAT&nbsp;<b>dev&nbsp;vs&nbsp;main</b> results table&nbsp;↗</a>
</p>

The table above is the native benchexec viewer comparing the **most recent dev and main** builds
(dev on the left), one tab per property, with per-task verdicts, times, and clickable logs. Each
individual run below is also kept (last 3 per channel).

<h2>Past runs</h2>
<div id="runs-list" style="opacity:.85;">Loading…</div>

<script>
(function () {
  var p = window.location.pathname, m = p.match(/^(.*\/docs\/)/);
  var BASE = m ? m[1] : p.replace(/[^/]*$/, "");
  document.getElementById("compare-link").href = BASE + "assets/results/compare.html";
  fetch(BASE + "assets/results/manifest.json", { cache: "no-store" })
    .then(function (r) { return r.ok ? r.json() : { runs: [] }; })
    .then(function (data) {
      var runs = data.runs || [];
      if (!runs.length) { document.getElementById("runs-list").textContent = "No runs published yet."; return; }
      var byCh = {};
      runs.forEach(function (r) { (byCh[r.channel] = byCh[r.channel] || []).push(r); });
      var html = "";
      Object.keys(byCh).forEach(function (ch) {
        html += "<h3>" + ch + "</h3><ul>";
        byCh[ch].forEach(function (r) {
          html += '<li><a href="' + BASE + r.path + '" target="_blank" rel="noopener">' +
                  r.date + " · " + r.short + "</a> — score <b>" + r.score + "</b>, correct " +
                  r.correct + "/" + r.total + (r.profile === "test" ? " <em>(test)</em>" : "") + "</li>";
        });
        html += "</ul>";
      });
      document.getElementById("runs-list").innerHTML = html;
    })
    .catch(function () { document.getElementById("runs-list").textContent = "Could not load run list."; });
})();
</script>
