# paps-fantasy
Fantasy supercross and motocross 

paps-fantasy/
│
├── index.html
├── style.css
├── data.js
├── scoring.js
└── app.js

<!DOCTYPE html>
<html>
<head>
  <title>PAPS Fantasy Commissioner</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>

<header>
  <h1>PAPS Fantasy – Commissioner Dashboard</h1>
</header>

function showTab(tabId) {
  document.querySelectorAll(".tab").forEach(tab => {
    tab.classList.add("hidden");
  });

  const activeTab = document.getElementById(tabId);
  if (activeTab) {
    activeTab.classList.remove("hidden");
  }

  function render(activeTab) {
  if (activeTab === "setup") renderSetup();
  if (activeTab === "teams") renderTeams();
  if (activeTab === "lineups") renderLineups();
  if (activeTab === "results") renderResults();
  if (activeTab === "wildcards") renderWildcards();
  if (activeTab === "standings") renderStandings();
}
function save() {
  localStorage.setItem("papsFantasy", JSON.stringify(league));
}

function load() {
  const data = localStorage.getItem("papsFantasy");
  if (data) Object.assign(league, JSON.parse(data));
}

load();


function riderScore(r) {
  let score = r.finish;

  if (r.heatWin) score -= 1;
  if (r.lcq) score += 2;
  if (!r.madeMain) score += 27;
  if (r.tripleWin) score -= 1;

  return score;
}

function teamWeekScore(team, week) {
  let total = 0;

  week.results.forEach(r => {
    if (team.activeRiders.includes(r.name)) {
      total += riderScore(r);
    }
  });

  if (!week.wildcardsSubmitted) total += 60; // 2 wildcards × 30
  if (week.smxPick === "correct") total -= 5;
  if (week.smxPick === "wrong") total += 3;

  return total;
}


function showTab(tab) {
  document.querySelectorAll(".tab").forEach(t => t.classList.add("hidden"));
  document.getElementById(tab).classList.remove("hidden");
  render();
}

/* ---------- LEAGUE SETUP ---------- */
function renderSetup() {
  const el = document.getElementById("setup");
  el.innerHTML = `
    <h2>League Setup</h2>
    <button onclick="addTeam()">Add Team</button>
    <button onclick="nextWeek()">Advance to Week ${league.currentWeek + 1}</button>
  `;
}

function addTeam() {
  const name = prompt("Team Name?");
  league.teams.push({
    name,
    riders: { "450": null, "250E": null, "250W": null },
    injured: [],
    totalScore: 0,
    weeklyLowWins: 0
  });
  save();
  render();
}

/* ---------- TEAMS ---------- */
function renderTeams() {
  const el = document.getElementById("teams");
  el.innerHTML = "<h2>Teams</h2>";

  league.teams.forEach(t => {
    el.innerHTML += `
      <div class="card">
        <strong>${t.name}</strong><br>
        450: ${t.riders["450"] || "—"}<br>
        250E: ${t.riders["250E"] || "—"}<br>
        250W: ${t.riders["250W"] || "—"}<br>
        IR: ${t.injured.join(", ") || "None"}
      </div>
    `;
  });
}

/* ---------- WEEKLY LINEUPS ---------- */
function renderLineups() {
  const el = document.getElementById("lineups");
  el.innerHTML = `<h2>Week ${league.currentWeek} Lineups (Locked)</h2>`;
}

/* ---------- RESULTS ENTRY ---------- */
function renderResults() {
  const el = document.getElementById("results");
  el.innerHTML = `
    <h2>Enter Race Results – Week ${league.currentWeek}</h2>
    <button onclick="addResult()">Add Rider Result</button>
  `;
}

function addResult() {
  if (!league.weeks[league.currentWeek]) {
    league.weeks[league.currentWeek] = { results: [] };
  }

  const r = {
    name: prompt("Rider Name"),
    finish: Number(prompt("Finish Position")),
    heatWin: confirm("Heat Win?"),
    lcq: confirm("LCQ?"),
    madeMain: confirm("Made Main?"),
    tripleWin: confirm("Triple Crown Moto Win?")
  };

  league.weeks[league.currentWeek].results.push(r);
  save();
}

/* ---------- WILDCARDS ---------- */
function renderWildcards() {
  const el = document.getElementById("wildcards");
  el.innerHTML = `<h2>Wildcards & SMX Next</h2>`;
}

/* ---------- STANDINGS ---------- */
function renderStandings() {
  const el = document.getElementById("standings");
  el.innerHTML = "<h2>Standings</h2>";

  league.teams.forEach(team => {
    team.totalScore = 0;

    Object.keys(league.weeks).forEach(w => {
      team.totalScore += teamWeekScore(team, league.weeks[w]);
    });
  });

  league.teams
    .sort((a, b) => a.totalScore - b.totalScore)
    .forEach(t => {
      el.innerHTML += `
        <div class="card">
          <strong>${t.name}</strong><br>
          Total: ${t.totalScore}
        </div>
      `;
    });
}

function nextWeek() {
  league.currentWeek++;
  save();
  render();
}

/* ---------- RENDER ---------- */
function render() {
  renderSetup();
  renderTeams();
  renderLineups();
  renderResults();
  renderWildcards();
  renderStandings();
}

render();

