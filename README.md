# ACC
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Attendance Calculator with Login</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body {
      font-family: system-ui, sans-serif;
      background: #f5f7fa;
      margin: 0;
      padding: 20px;
      text-align: center;
    }
    h1 { margin-bottom: 8px; }
    input, select { padding: 6px; margin: 6px; border: 1px solid #ccc; border-radius: 6px; }
    button { padding: 8px 14px; border: none; border-radius: 6px; background: #007bff; color: white; cursor: pointer; margin: 5px; }
    button:hover { background: #0056b3; }
    .card { background: white; padding: 16px; border-radius: 10px; margin: 20px auto; max-width: 900px; box-shadow: 0 4px 10px rgba(0,0,0,.1); }
    .calendar { display: grid; grid-template-columns: repeat(auto-fill, minmax(90px, 1fr)); gap: 6px; margin-top: 15px; }
    .day { padding: 12px; border-radius: 6px; background: #eee; cursor: pointer; }
    .present { background: #22c55e; color: white; }
    .absent { background: #ef4444; color: white; }
    .holiday { background: #9ca3af; color: white; }
    .hidden { display: none; }
    p.error { color: red; }

    /* 🔐 Center login/signup screens */
    #loginScreen, #signupScreen {
      display: none;
      align-items: center;
      justify-content: center;
      height: 100vh;
    }
    #loginScreen.active, #signupScreen.active {
      display: flex;
    }
    #loginScreen .card, #signupScreen .card {
      background: white;
      padding: 30px;
      border-radius: 12px;
      box-shadow: 0 4px 12px rgba(0,0,0,.1);
      width: 320px;
      text-align: center;
    }
    #loginScreen .card input, #signupScreen .card input {
      width: 90%;
      padding: 10px;
      margin: 10px auto;
      border: 1px solid #ccc;
      border-radius: 6px;
      display: block;
    }
    #loginScreen .card button, #signupScreen .card button {
      width: 100%;
      padding: 12px;
      margin-top: 10px;
      border: none;
      border-radius: 6px;
      background: #007bff;
      color: white;
      cursor: pointer;
      font-size: 16px;
    }
    #loginScreen .card button:hover, #signupScreen .card button:hover { background: #0056b3; }
  </style>
</head>
<body>

  <!-- 🔐 LOGIN SCREEN -->
  <div id="loginScreen">
    <div class="card">
      <h2>🔐 Login</h2>
      <input type="text" id="loginUser" placeholder="Username (email)">
      <input type="password" id="loginPass" placeholder="Password">
      <button onclick="login()">Sign In</button>
      <p id="loginError" class="error"></p>
      <p>Don't have an account? <a href="#" onclick="showSignup()">Sign Up</a></p>
    </div>
  </div>

  <!-- 📝 SIGNUP SCREEN -->
  <div id="signupScreen">
    <div class="card">
      <h2>📝 Sign Up</h2>
      <input type="text" id="signupUser" placeholder="Username (email)">
      <input type="password" id="signupPass" placeholder="Password">
      <button onclick="signup()">Register</button>
      <p id="signupError" class="error"></p>
      <p>Already have an account? <a href="#" onclick="showLogin()">Login</a></p>
    </div>
  </div>

  <!-- 📊 ATTENDANCE CALCULATOR -->
  <div id="calculatorScreen" class="hidden">
    <h1>📊 Attendance Calculator (L/T/P/S)</h1>
    <p>Track attendance separately for Lectures, Tutorials, Practicals, and Seminars.</p>
    <button onclick="logout()">🚪 Logout</button>
    <a href="wc.html">
      <button id="toggleWeatherBtn" onclick="toggleWeather()">🌦 Show Weather & Calendar</button>
    </a>
    <a href="calc.html" >
      <button class="link button">🧮 QuantumCalc</button>
    </a>
    <!-- Controls -->
    <div class="card">
      <label>Subject:
        <select id="subject">
          <option value="Lecture">Lecture (L)</option>
          <option value="Tutorial">Tutorial (T)</option>
          <option value="Practical">Practical (P)</option>
          <option value="Seminar">Seminar (S)</option>
        </select>
      </label>
      <br>
      <label>Requirement %:
        <select id="req">
          <option value="75">75%</option>
          <option value="65">65%</option>
          <option value="80">80%</option>
        </select>
      </label>
      <br>
      <label>Present Classes: <input type="number" id="present"></label>
      <label>Total Classes: <input type="number" id="total"></label>
      <br>
      <button onclick="generateAttendance()">Generate</button>
      <p id="summary"></p>
    </div>

    <!-- Tools -->
    <div class="card">
      <button onclick="saveAttendance()">💾 Save</button>
      <button onclick="loadAttendance()">📂 Load</button>
      <button onclick="exportCSV()">⬇️ Export CSV</button>
      <button onclick="predictNext()">🔮 Next Class Predictor</button>
      <p id="prediction"></p>
    </div>

    <!-- Calendar -->
    <div class="card">
      <h3>🗓 Attendance Calendar</h3>
      <div id="calendar" class="calendar"></div>
    </div>

    <!-- Charts -->
    <div class="card">
      <h3>📊 Charts (per subject)</h3>
      <canvas id="pieChart"></canvas>
      <canvas id="weekChart"></canvas>
    </div>
  </div>

<script>
/* ------------------ LOGIN SYSTEM ------------------ */
function showSignup() {
  document.getElementById("loginScreen").classList.remove("active");
  document.getElementById("signupScreen").classList.add("active");
}
function showLogin() {
  document.getElementById("signupScreen").classList.remove("active");
  document.getElementById("loginScreen").classList.add("active");
}

function signup() {
  const user = document.getElementById("signupUser").value;
  const pass = document.getElementById("signupPass").value;
  if (!user || !pass) {
    document.getElementById("signupError").textContent = "⚠️ Please fill all fields";
    return;
  }
  let users = JSON.parse(localStorage.getItem("users")) || {};
  if (users[user]) {
    document.getElementById("signupError").textContent = "⚠️ User already exists!";
    return;
  }
  users[user] = pass;
  localStorage.setItem("users", JSON.stringify(users));
  alert("✅ Account created. Please login.");
  showLogin();
}

function login() {
  const user = document.getElementById("loginUser").value;
  const pass = document.getElementById("loginPass").value;
  let users = JSON.parse(localStorage.getItem("users")) || {};
  if (users[user] && users[user] === pass) {
    localStorage.setItem("loggedInUser", user);
    document.getElementById("loginScreen").classList.remove("active");
    document.getElementById("signupScreen").classList.remove("active");
    document.getElementById("calculatorScreen").classList.remove("hidden");
  } else {
    document.getElementById("loginError").textContent = "❌ Invalid username or password!";
  }
}

function logout() {
  localStorage.removeItem("loggedInUser");
  document.getElementById("calculatorScreen").classList.add("hidden");
  document.getElementById("loginScreen").classList.add("active");
}

window.onload = function() {
  if (localStorage.getItem("loggedInUser")) {
    document.getElementById("calculatorScreen").classList.remove("hidden");
  } else {
    document.getElementById("loginScreen").classList.add("active");
  }
}

/* ------------------ ATTENDANCE CALCULATOR ------------------ */
let dataStore = { Lecture:{}, Tutorial:{}, Practical:{}, Seminar:{} };
let total = { Lecture:0, Tutorial:0, Practical:0, Seminar:0 };
let present = { Lecture:0, Tutorial:0, Practical:0, Seminar:0 };
let required = 75;

function currentSubject() { return document.getElementById("subject").value; }

function generateAttendance() {
  const subj = currentSubject();
  present[subj] = Number(document.getElementById("present").value);
  total[subj] = Number(document.getElementById("total").value);
  required = Number(document.getElementById("req").value);

  if (total[subj] <= 0 || present[subj] < 0 || present[subj] > total[subj]) {
    alert("Enter valid numbers");
    return;
  }
  dataStore[subj] = {};
  for (let i = 1; i <= total[subj]; i++) {
    dataStore[subj][i] = (i <= present[subj]) ? "present" : "absent";
  }
  updateSummary(); renderCalendar(); drawCharts();
}

function updateSummary() {
  const subj = currentSubject();
  const attended = Object.values(dataStore[subj]).filter(v => v === "present").length;
  const totalClasses = Object.values(dataStore[subj]).filter(v => v !== "holiday").length;
  const percent = totalClasses ? (attended / totalClasses) * 100 : 0;
  const need = Math.max(0, Math.ceil((required/100)*totalClasses) - attended);
  const summary = document.getElementById("summary");
  summary.innerHTML =
    `(${subj}) Present: <b>${attended}</b> / ${totalClasses} (${percent.toFixed(2)}%) <br>` +
    (percent >= required ? `✅ Requirement met (${required}%)` :
    `⚠️ Need ${need} more classes to reach ${required}%`);
  summary.style.color = percent >= required ? "green" : "red";
}

function renderCalendar() {
  const subj = currentSubject();
  const cal = document.getElementById("calendar");
  cal.innerHTML = "";
  for (let i = 1; i <= total[subj]; i++) {
    const cell = document.createElement("div");
    cell.className = "day " + dataStore[subj][i];
    cell.textContent = "Day " + i;
    cell.onclick = () => {
      dataStore[subj][i] = dataStore[subj][i] === "holiday" ? "present" :
                          dataStore[subj][i] === "present" ? "absent" : "holiday";
      renderCalendar(); updateSummary(); drawCharts();
    };
    cal.appendChild(cell);
  }
}

function saveAttendance() {
  localStorage.setItem("attendanceData", JSON.stringify({dataStore,total,present,required}));
  alert("Attendance saved ✅");
}
function loadAttendance() {
  const data = JSON.parse(localStorage.getItem("attendanceData"));
  if (data) {
    dataStore = data.dataStore; total = data.total; present = data.present; required = data.required;
    renderCalendar(); updateSummary(); drawCharts();
  } else { alert("No saved data ❌"); }
}
function exportCSV() {
  let csv = "Subject,Day,Status\n";
  for (const subj in dataStore) {
    for (let i=1; i<=total[subj]; i++) {
      csv += `${subj},Day ${i},${dataStore[subj][i]}\n`;
    }
  }
  const blob = new Blob([csv], { type: "text/csv" });
  const link = document.createElement("a");
  link.href = URL.createObjectURL(blob);
  link.download = "attendance.csv"; link.click();
}
function predictNext() {
  const subj = currentSubject();
  const attended = Object.values(dataStore[subj]).filter(v => v==="present").length;
  const totalClasses = Object.values(dataStore[subj]).filter(v => v!=="holiday").length;

  const attendNext = ((attended+1)/(totalClasses+1))*100;
  const skipNext = (attended/(totalClasses+1))*100;

  document.getElementById("prediction").innerHTML =
    `(${subj})<br>` +
    `If you attend next: <b>${attendNext.toFixed(2)}%</b> ` +
    (attendNext >= required ? "✅ Requirement met" : "⚠️ Below requirement") + "<br>" +
    `If you skip next: <b>${skipNext.toFixed(2)}%</b> ` +
    (skipNext >= required ? "✅ Requirement met" : "⚠️ Below requirement");
}

let pieChart, weekChart;
function drawCharts() {
  const subj = currentSubject();
  const ctx1 = document.getElementById("pieChart").getContext("2d");
  if (pieChart) pieChart.destroy();
  const p = Object.values(dataStore[subj]).filter(v => v==="present").length;
  const a = Object.values(dataStore[subj]).filter(v => v==="absent").length;
  const h = Object.values(dataStore[subj]).filter(v => v==="holiday").length;
  pieChart = new Chart(ctx1, {
    type: "pie",
    data: { labels: ["Present","Absent","Holiday"],
      datasets: [{ data:[p,a,h], backgroundColor:["#22c55e","#ef4444","#9ca3af"] }] }
  });

  const ctx2 = document.getElementById("weekChart").getContext("2d");
  if (weekChart) weekChart.destroy();
  const weeks = Math.ceil(total[subj]/7);
  let weekData = [];
  for (let w=0; w<weeks; w++) {
    let start = w*7+1, end = Math.min(total[subj],(w+1)*7);
    let weekDays = 0, weekPres = 0;
    for (let d=start; d<=end; d++) {
      if (dataStore[subj][d] !== "holiday") { weekDays++; if (dataStore[subj][d]==="present") weekPres++; }
    }
    weekData.push(weekDays? (weekPres/weekDays*100) : 0);
  }
  weekChart = new Chart(ctx2, {
    type: "line",
    data: { labels: weekData.map((_,i)=>"Week "+(i+1)),
      datasets: [{ label:`${subj} Weekly %`, data:weekData,
        borderColor:"#007bff", backgroundColor:"rgba(0,123,255,0.2)",
        fill:true, tension:0.4 }] }
  });
}
</script>
</body>
</html>
<!-- 🌦 Weather + Calendar Section -->
<div class="card" style="max-width:600px; margin:auto; text-align:center;">
  <h3 style="margin-bottom:15px;">🌦 Weather Forecast & 🗓 Calendar</h3>

  <!-- Weather -->
  <div style="margin-bottom:20px;">
    <input type="text" id="city" placeholder="Enter city" 
           style="padding:8px; border:1px solid #ccc; border-radius:6px; width:60%; max-width:250px;">
    <button onclick="getWeather()" 
            style="padding:8px 14px; border:none; border-radius:6px; background:#007bff; color:white; margin-left:6px; cursor:pointer;">
      Check
    </button>
    <p id="weather" style="margin-top:12px; font-size:1rem; font-weight:500;"></p>
  </div>

  <hr style="margin:20px 0;">

  <!-- Calendar -->
  <div>
    <h4 id="monthYear" style="margin-bottom:10px; font-size:1.2rem; color:#333;"></h4>
    <div id="calendarGrid" 
         style="display:grid; grid-template-columns: repeat(7, 1fr); gap:8px; justify-items:center;">
    </div>
  </div>
</div>

<script>
/* 🌤 Weather */
async function getWeather() {
  const city = document.getElementById("city").value.trim();
  if (!city) { alert("Enter city name"); return; }
  try {
    const res = await fetch(`https://wttr.in/${city}?format=%C+%t`);
    const txt = await res.text();
    document.getElementById("weather").innerText = "📍 " + city + " → " + txt;
  } catch {
    document.getElementById("weather").innerText = "❌ Error fetching weather";
  }
}

/* 🗓 Calendar (clean design) */
function renderCalendar() {
  const today = new Date();
  const month = today.getMonth();
  const year = today.getFullYear();
  const monthNames = ["January","February","March","April","May","June",
                      "July","August","September","October","November","December"];
  
  document.getElementById("monthYear").innerText = `${monthNames[month]} ${year}`;
  const calendarGrid = document.getElementById("calendarGrid");
  calendarGrid.innerHTML = "";

  // Days of week header
  const daysOfWeek = ["Sun","Mon","Tue","Wed","Thu","Fri","Sat"];
  daysOfWeek.forEach(d => {
    const cell = document.createElement("div");
    cell.innerHTML = `<b>${d}</b>`;
    cell.style.color = "#007bff";
    cell.style.fontSize = "0.9rem";
    calendarGrid.appendChild(cell);
  });

  // First day of month
  const firstDay = new Date(year, month, 1).getDay();
  const daysInMonth = new Date(year, month+1, 0).getDate();

  // Empty slots before month starts
  for (let i=0; i<firstDay; i++) {
    const emptyCell = document.createElement("div");
    calendarGrid.appendChild(emptyCell);
  }

  // Days of month
  for (let d=1; d<=daysInMonth; d++) {
    const cell = document.createElement("div");
    cell.textContent = d;
    cell.style.width = "40px";
    cell.style.height = "40px";
    cell.style.display = "flex";
    cell.style.alignItems = "center";
    cell.style.justifyContent = "center";
    cell.style.borderRadius = "50%";
    cell.style.cursor = "default";

    // Highlight today
    if (d === today.getDate()) {
      cell.style.background = "#007bff";
      cell.style.color = "white";
      cell.style.fontWeight = "bold";
      cell.style.boxShadow = "0 2px 6px rgba(0,0,0,0.2)";
    } else {
      cell.style.background = "#f8f9fa";
      cell.style.color = "#333";
    }

    calendarGrid.appendChild(cell);
  }
}

// Load calendar when page opens
window.addEventListener("load", renderCalendar);
</script>
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <title>QuantumCalc</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #3498db; /* Blue background */
      display: flex;
      justify-content: center;
      align-items: flex-start;
      min-height: 100vh;
      margin: 0;
      padding: 40px;
      gap: 30px;
    }

    .calculator {
      background: white;
      padding: 20px;
      border-radius: 12px;
      box-shadow: 0 4px 12px rgba(0,0,0,.2);
      width: 300px;
      text-align: center;
    }

    .calculator h1 {
      margin-bottom: 15px;
      font-size: 24px;
      color: #2c3e50;
    }

    #display {
      width: 100%;
      height: 40px;
      font-size: 18px;
      text-align: right;
      padding-right: 10px;
      margin-bottom: 10px;
      border: 1px solid #ccc;
      border-radius: 6px;
    }

    .buttons {
      display: grid;
      grid-template-columns: repeat(4, 1fr);
      gap: 10px;
    }

    button {
      padding: 15px;
      font-size: 18px;
      border: none;
      border-radius: 8px;
      background: #2980b9;
      color: white;
      cursor: pointer;
    }

    button:hover {
      background: #1c5980;
    }

    .equal {
      grid-column: span 2;
      background: #27ae60;
    }

    .equal:hover {
      background: #1e8449;
    }

    .clear {
      background: #c0392b;
    }

    .clear:hover {
      background: #922b21;
    }

    .del {
      background: #f39c12;
    }

    .del:hover {
      background: #d68910;
    }

    .history-container {
      background: white;
      border-radius: 12px;
      padding: 20px;
      box-shadow: 0 4px 12px rgba(0,0,0,.2);
      width: 250px;
      max-height: 400px;
      overflow-y: auto;
    }

    .history-container h2 {
      font-size: 20px;
      margin-bottom: 15px;
      color: #2c3e50;
      text-align: center;
    }

    .history-item {
      font-size: 14px;
      margin-bottom: 8px;
      cursor: default;
      border-bottom: 1px solid #ccc;
      padding-bottom: 4px;
    }
  </style>
</head>
<body>

  <div class="calculator">
    <h1>QuantumCalc</h1>
    <input type="text" id="display" disabled>
    <div class="buttons">
      <button onclick="append('7')">7</button>
      <button onclick="append('8')">8</button>
      <button onclick="append('9')">9</button>
      <button onclick="append('/')">÷</button>

      <button onclick="append('4')">4</button>
      <button onclick="append('5')">5</button>
      <button onclick="append('6')">6</button>
      <button onclick="append('*')">×</button>

      <button onclick="append('1')">1</button>
      <button onclick="append('2')">2</button>
      <button onclick="append('3')">3</button>
      <button onclick="append('-')">−</button>

      <button onclick="append('0')">0</button>
      <button onclick="append('%')">%</button>
      <button onclick="append('.')">.</button>
      <button onclick="append('+')">+</button>

      <button class="clear" onclick="clearDisplay()">AC</button>
      <button class="del" onclick="deleteLast()">DEL</button>
      <button class="equal" onclick="calculate()">=</button>
    </div>
  </div>

  <div class="history-container">
    <h2>Recent Calculations</h2>
    <div id="history"></div>
  </div>

  <script>
    const display = document.getElementById("display");
    const historyDiv = document.getElementById("history");
    let history = [];

    function append(value) {
      display.value += value;
    }

    function clearDisplay() {
      display.value = "";
    }

    function deleteLast() {
      display.value = display.value.slice(0, -1);
    }

    function calculate() {
      try {
        const result = eval(display.value);
        history.unshift(`${display.value} = ${result}`);
        if(history.length > 10) history.pop(); // Keep last 10
        updateHistory();
        display.value = result;
      } catch {
        display.value = "Error";
      }
    }

    function updateHistory() {
      historyDiv.innerHTML = history.map(item => `<div class="history-item">${item}</div>`).join('');
    }
  </script>

</body>
</html>
