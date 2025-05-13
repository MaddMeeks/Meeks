<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Retirement Calculator</title>
  <meta name="description" content="Interactive Retirement Calculator by Meeks" />
  <meta name="author" content="MaddMeeks" />
  <meta name="keywords" content="retirement calculator, 401k, roth IRA, investment, budget forecast" />
  <link rel="canonical" href="https://github.com/MaddMeeks/Meeks.git" />
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap');

    :root {
      --bg-light: white;
      --text-light: #333;
      --bg-dark: #1e1e1e;
      --text-dark: #f1f1f1;
    }

    body {
      font-family: 'Inter', sans-serif;
      padding: 2rem;
      background: linear-gradient(to right, #74ebd5, #ACB6E5);
      color: var(--text-light);
      transition: background 0.3s ease, color 0.3s ease;
    }

    body.dark-mode {
      background: #121212;
      color: var(--text-dark);
    }

    .container {
      max-width: 950px;
      margin: 0 auto;
      background: var(--bg-light);
      color: var(--text-light);
      padding: 2rem;
      border-radius: 15px;
      box-shadow: 0 4px 20px rgba(0,0,0,0.15);
      transition: background 0.3s ease, color 0.3s ease;
    }

    body.dark-mode .container {
      background: var(--bg-dark);
      color: var(--text-dark);
    }

    input, button, label {
      color: inherit;
    }

    input {
      width: 100%;
      padding: 0.5rem;
      margin: 0.3rem 0 1rem;
      border: 1px solid #ccc;
      border-radius: 8px;
      background: inherit;
    }

    label {
      font-weight: 600;
      display: block;
      margin-top: 1rem;
    }

    button {
      margin-top: 1.5rem;
      padding: 0.75rem 2rem;
      font-size: 1rem;
      background-color: #4CAF50;
      color: white;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      transition: background 0.3s ease;
    }

    button:hover {
      background-color: #45a049;
    }

    h1, h3 {
      text-align: center;
    }

    .results {
      margin-top: 2rem;
      background: #ecf9ec;
      padding: 1rem;
      border-radius: 8px;
    }

    body.dark-mode .results {
      background: #2c2c2c;
    }

    canvas {
      margin-top: 2rem;
    }

    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 1rem;
    }

    th, td {
      border: 1px solid #ccc;
      padding: 0.5rem;
      text-align: center;
    }

    .button-row {
      display: flex;
      justify-content: space-around;
      margin-top: 1.5rem;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>Retirement Calculator</h1>
    <p style="text-align:center"><a href="https://github.com/MaddMeeks/Meeks.git" target="_blank">View Source on GitHub</a></p>
    <form id="retirementForm">
      <label>Current Age:<input type="number" id="age" value="30" /></label>
      <label>Retirement Age:<input type="number" id="retirementAge" value="65" /></label>

      <h3>401K</h3>
      <label>Current 401K Amount:<input type="number" id="current401K" value="0" /></label>
      <label>Total Contribution %:<input type="number" id="totalContribution" value="10" /></label>
      <label>401K Rate of Return %:<input type="number" id="rateOfReturn401K" value="7" /></label>

      <h3>Roth IRA</h3>
      <label>Current Roth Amount:<input type="number" id="currentRoth" value="0" /></label>
      <label>Annual Roth Contribution:<input type="number" id="rothAnnualContribution" value="6500" /></label>
      <label>Roth Rate of Return %:<input type="number" id="rateOfReturnRoth" value="7" /></label>

      <h3>Other Investments</h3>
      <label>Current Investment:<input type="number" id="currentOther" value="0" /></label>
      <label>Annual Contribution:<input type="number" id="otherAnnualContribution" value="0" /></label>
      <label>Other Investment Return %:<input type="number" id="rateOfReturnOther" value="7" /></label>

      <h3>Economic Factors</h3>
      <label>Annual Salary:<input type="number" id="annualSalary" value="60000" /></label>
      <label>Salary Growth %:<input type="number" id="salaryGrowth" value="2" /></label>
      <label>Inflation %:<input type="number" id="inflation" value="2" /></label>
      <label>Tax Rate %:<input type="number" id="taxRate" value="25" /></label>

      <div class="button-row">
        <button type="button" onclick="calculate()">Calculate</button>
        <button type="button" onclick="toggleDarkMode()">Toggle Dark Mode</button>
        <button type="button" onclick="downloadCSV()">Export CSV</button>
        <button type="button" onclick="downloadPDF()">Export PDF</button>
      </div>
    </form>

    <div class="results" id="results" style="display:none">
      <h3>Results</h3>
      <p id="total"></p>
      <p id="afterTax"></p>
      <p id="adjusted"></p>
      <p id="monthly"></p>
      <canvas id="breakdownChart" width="400" height="200"></canvas>
      <table id="breakdownTable">
        <thead>
          <tr><th>Year</th><th>401K</th><th>Roth IRA</th><th>Other</th></tr>
        </thead>
        <tbody></tbody>
      </table>
    </div>
  </div>

<script>
function futureValue(current, annual, rate, years) {
  let history = [];
  let value = current;
  for (let i = 0; i < years; i++) {
    value = (value + annual) * (1 + rate);
    history.push(parseFloat(value.toFixed(2)));
  }
  return [value, history];
}

function calculate() {
  const get = id => parseFloat(document.getElementById(id).value);
  const age = get("age"), retirementAge = get("retirementAge"), years = retirementAge - age;

  const annualSalary = get("annualSalary");
  const annual401KContrib = annualSalary * (get("totalContribution") / 100);
  const [future401K, history401K] = futureValue(get("current401K"), annual401KContrib, get("rateOfReturn401K") / 100, years);
  const [futureRoth, historyRoth] = futureValue(get("currentRoth"), get("rothAnnualContribution"), get("rateOfReturnRoth") / 100, years);
  const [futureOther, historyOther] = futureValue(get("currentOther"), get("otherAnnualContribution"), get("rateOfReturnOther") / 100, years);

  const taxRate = get("taxRate") / 100;
  const afterTax401K = future401K * (1 - taxRate);
  const total = afterTax401K + futureRoth + futureOther;
  const adjusted = total / Math.pow(1 + get("inflation") / 100, years);
  const monthly = adjusted / ((85 - retirementAge) * 12);

  document.getElementById("total").textContent = `Total at Retirement: $${total.toFixed(2)}`;
  document.getElementById("afterTax").textContent = `401K After Tax: $${afterTax401K.toFixed(2)}`;
  document.getElementById("adjusted").textContent = `Purchasing Power Today: $${adjusted.toFixed(2)}`;
  document.getElementById("monthly").textContent = `Monthly Allowance Estimate: $${monthly.toFixed(2)}`;

  const ctx = document.getElementById('breakdownChart').getContext('2d');
  new Chart(ctx, {
    type: 'line',
    data: {
      labels: Array.from({length: years}, (_, i) => `Year ${i + 1}`),
      datasets: [
        { label: '401K', data: history401K, borderColor: 'blue', fill: false },
        { label: 'Roth IRA', data: historyRoth, borderColor: 'green', fill: false },
        { label: 'Other', data: historyOther, borderColor: 'orange', fill: false }
      ]
    },
    options: {
      responsive: true,
      plugins: { title: { display: true, text: 'Growth Breakdown Over Time' } }
    }
  });

  const tbody = document.querySelector('#breakdownTable tbody');
  tbody.innerHTML = "";
  for (let i = 0; i < years; i++) {
    tbody.innerHTML += `<tr><td>${i + 1}</td><td>$${history401K[i]}</td><td>$${historyRoth[i]}</td><td>$${historyOther[i]}</td></tr>`;
  }

  document.getElementById("results").style.display = 'block';
}

function toggleDarkMode() {
  document.body.classList.toggle("dark-mode");
}

function downloadCSV() {
  const rows = [["Year", "401K", "Roth IRA", "Other"]];
  document.querySelectorAll("#breakdownTable tbody tr").forEach(tr => {
    rows.push(Array.from(tr.children).map(td => td.textContent));
  });
  const csvContent = rows.map(e => e.join(",")).join("\n");
  const blob = new Blob([csvContent], { type: "text/csv" });
  const link = document.createElement("a");
  link.href = URL.createObjectURL(blob);
  link.download = "retirement_breakdown.csv";
  link.click();
}

function downloadPDF() {
  const { jsPDF } = window.jspdf;
  const doc = new jsPDF();
  doc.text("Retirement Calculator Results", 10, 10);
  const rows = [];
  document.querySelectorAll("#breakdownTable tr").forEach(tr => {
    rows.push(Array.from(tr.children).map(td => td.textContent));
  });
  doc.autoTable({ head: [rows[0]], body: rows.slice(1), startY: 20 });
  doc.save("retirement_report.pdf");
}
</script>
</body>
</html>
