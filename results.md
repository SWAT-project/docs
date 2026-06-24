---
title: Benchmark Results
nav_order: 4
---

# SWAT SV-COMP Benchmark Results
{: .no_toc }

Nightly benchexec runs of SWAT against the SV-COMP Java verification tasks, under the
official competition limits (15 min / 15 GB / 4 cores). Pick a channel and a run.
{: .fs-5 .fw-300 }

<div id="swat-results">
  <div style="display:flex; gap:1.5rem; flex-wrap:wrap; align-items:flex-end; margin:1rem 0;">
    <label>Channel<br><select id="ch-sel" style="min-width:10rem; padding:.3rem;"></select></label>
    <label>Run (date · commit)<br><select id="run-sel" style="min-width:22rem; padding:.3rem;"></select></label>
    <span id="run-meta" style="font-size:.9em; opacity:.8;"></span>
  </div>
  <p id="status" style="opacity:.7;">Loading runs…</p>
  <iframe id="tbl" title="benchexec results table" style="width:100%; height:80vh; border:1px solid #444; border-radius:6px; background:#fff;"></iframe>
</div>

<script>
(function () {
  // Resolve the site base (…/docs/) robustly regardless of the page's permalink.
  var p = window.location.pathname;
  var m = p.match(/^(.*\/docs\/)/);
  var BASE = m ? m[1] : p.replace(/[^/]*$/, "");
  var ASSETS = BASE + "assets/results/";
  var chSel = document.getElementById("ch-sel"),
      runSel = document.getElementById("run-sel"),
      meta = document.getElementById("run-meta"),
      status = document.getElementById("status"),
      tbl = document.getElementById("tbl");
  var RUNS = [];

  function byChannel(ch) { return RUNS.filter(function (r) { return r.channel === ch; }); }

  function loadRun(r) {
    if (!r) { tbl.removeAttribute("src"); meta.textContent = ""; return; }
    tbl.src = BASE + r.path;
    meta.textContent = (r.profile ? "[" + r.profile + "] " : "") + (r.rundefs || "");
  }

  function fillRuns(ch) {
    runSel.innerHTML = "";
    byChannel(ch).forEach(function (r, i) {
      var o = document.createElement("option");
      o.value = i;
      o.textContent = r.date + " · " + (r.short || (r.commit || "").slice(0, 7));
      runSel.appendChild(o);
    });
    runSel.selectedIndex = 0;
    loadRun(byChannel(ch)[0]);
  }

  fetch(ASSETS + "manifest.json", { cache: "no-store" })
    .then(function (r) { if (!r.ok) throw new Error(r.status); return r.json(); })
    .then(function (data) {
      RUNS = data.runs || [];
      if (!RUNS.length) { status.textContent = "No runs published yet."; return; }
      status.textContent = "";
      var channels = [];
      RUNS.forEach(function (r) { if (channels.indexOf(r.channel) < 0) channels.push(r.channel); });
      channels.forEach(function (c) {
        var o = document.createElement("option"); o.value = c; o.textContent = c; chSel.appendChild(o);
      });
      chSel.value = channels[0];
      fillRuns(channels[0]);
      chSel.addEventListener("change", function () { fillRuns(chSel.value); });
      runSel.addEventListener("change", function () { loadRun(byChannel(chSel.value)[runSel.value]); });
    })
    .catch(function (e) { status.textContent = "Could not load results manifest (" + e + ")."; });
})();
</script>
