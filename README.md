<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Retirement Calculator</title>
  <meta name="description" content="Interactive Retirement Calculator by Meeks">
  <meta name="author" content="MaddMeeks">
  <meta name="keywords" content="retirement calculator, 401k, roth IRA, investment, budget forecast">
  <link rel="canonical" href="https://github.com/MaddMeeks/Meeks.git">
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jsPDF/2.5.1/jspdf.umd.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.0/papaparse.min.js"></script>
  <style>
  @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap');

  :root {
    --primary: #4CAF50;
    --primary-dark: #388E3C;
    --bg-light: #f0f4f8;
    --bg-dark: #2c3e50;
    --text-light: #333;
    --text-dark: #f5f5f5;
    --input-bg-light: #fff;
    --input-bg-dark: #34495e;
    --shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
  }

  * {
    box-sizing: border-box;
  }

  body {
    font-family: 'Inter', sans-serif;
    background: var(--bg-light);
    color: var(--text-light);
    margin: 0;
    padding: 2rem;
    transition: background-color 0.3s, color 0.3s;
  }

  .container {
    max-width: 950px;
    margin: auto;
    background: white;
    padding: 2.5rem;
    border-radius: 16px;
    box-shadow: var(--shadow);
  }

  h1, h3 {
    text-align: center;
    color: #2c3e50;
    margin-bottom: 1rem;
  }

  label {
    font-weight: 600;
    display: block;
    margin-top: 1.2rem;
  }

  input, select {
    width: 100%;
    padding: 0.6rem 0.75rem;
    margin-top: 0.3rem;
    border: 1px solid #ccc;
    border-radius: 10px;
    background-color: var(--input-bg-light);
    font-size: 1rem;
    transition: all 0.2s ease;
  }

  input:focus, select:focus {
    outline: none;
    border-color: var(--primary);
    box-shadow: 0 0 0 3px rgba(76, 175, 80, 0.2);
  }

  button {
    margin-top: 1.5rem;
    padding: 0.8rem 2rem;
    font-size: 1rem;
    font-weight: 600;
    background-color: var(--primary);
    color: white;
    border: none;
    border-radius: 10px;
    cursor: pointer;
    transition: background-color 0.3s ease, transform 0.2s ease;
  }

  button:hover {
    background-color: var(--primary-dark);
    transform: translateY(-1px);
  }

  .results {
    margin-top: 2rem;
    background: #eafaf1;
    padding: 1.5rem;
    border-radius: 12px;
    box-shadow: inset 0 0 8px rgba(0, 0, 0, 0.05);
    font-size: 1.05rem;
  }

  canvas {
    margin-top: 2rem;
    background: #fff;
    border-radius: 8px;
  }

  a {
    color: #007BFF;
    text-decoration: none;
  }

  a:hover {
    text-decoration: underline;
  }

  .dark-mode {
    background-color: var(--bg-dark);
    color: var(--text-dark);
  }

  .dark-mode .container {
    background-color: #1e2a36;
    color: var(--text-dark);
  }

  .dark-mode input,
  .dark-mode select {
    background-color: var(--input-bg-dark);
    color: var(--text-dark);
    border: 1px solid #555;
  }

  .dark-mode input:focus,
  .dark-mode select:focus {
    box-shadow: 0 0 0 3px rgba(255, 255, 255, 0.1);
  }

  .dark-mode .results {
    background: #34495e;
  }

  .dark-mode button {
    background-color: #5cb85c;
  }

  .dark-mode h1, .dark-mode h3 {
    color: var(--text-dark);
  }
	</style>
</head>
<body>
<div class="container">
  <h1>Retirement Calculator</h1>
  <p style="text-align:center"><a href="https://github.com/MaddMeeks/Meeks.git" target="_blank">View Source on GitHub</a></p>
  <button onclick="toggleDarkMode()">Toggle Dark Mode</button>
  <form id="retirementForm">
    <label>Current Age:<input type="number" id="age" value="30" min="0" required></label>
    <label>Retirement Age:<input type="number" id="retirementAge" value="65" min="0" required></label>
    <label>End of Retirement Age (e.g. 85):<input type="number" id="endAge" value="85" min="0" required></label>

    <h3>401K</h3>
    <label>Current 401K Amount:<input type="number" id="current401K" value="0" min="0" required></label>
    <label>Total Contribution % (include employer contributions):<input type="number" id="totalContribution" value="10" min="0" required></label>

    <h3>Roth IRA</h3>
    <label>Current Roth Amount:<input type="number" id="currentRoth" value="0" min="0" required></label>
    <label>Annual Roth Contribution:<input type="number" id="rothAnnualContribution" value="6500" min="0" required></label>

    <h3>Other Investments</h3>
    <label>Current Investment:<input type="number" id="currentOther" value="0" min="0" required></label>
    <label>Annual Contribution:<input type="number" id="otherAnnualContribution" value="0" min="0" required></label>

    <h3>Economic Factors</h3>
    <label>Annual Salary:<input type="number" id="annualSalary" value="60000" min="0" required></label>
    <label>Salary Growth %:<input type="number" id="salaryGrowth" value="2" min="0" required></label>
    <label>Inflation %:<input type="number" id="inflation" value="2" min="0" required></label>
    <label>Tax Rate %:<input type="number" id="taxRate" value="25" min="0" required></label>
    <label>Rate of Return % (For 401K, Roth IRA, Other Investments):<input type="number" id="rateOfReturn" value="7" min="0" required></label>

    <label>Chart Type:
      <select id="chartType">
        <option value="line">Line</option>
        <option value="bar">Bar</option>
        <option value="area">Area</option>
      </select>
    </label>

    <button type="button" onclick="calculate()">Calculate</button>
  </form>

  <div class="results" id="results" style="display:none">
    <h3>Results</h3>
    <p id="total401k"></p>
    <p id="totalRoth"></p>
    <p id="totalOther"></p>
    <p id="grandTotal"></p>
    <p id="adjustedTotal"></p>
    <p id="monthly"></p>
    <canvas id="breakdownChart" width="400" height="200"></canvas>
    <button onclick="downloadCSV()">Download CSV</button>
    <button onclick="downloadPDF()">Download PDF</button>
    <button onclick="downloadJSON()">Download JSON</button>
  </div>
</div>

<script>
let chart;

function futureValueWithGrowth(current, rate, years, contributionFunc) {
  let history = [];
  let value = current;
  for (let i = 0; i < years; i++) {
    let contribution = contributionFunc(i);
    value = (value + contribution) * (1 + rate);
    history.push(value);
  }
  return [value, history];
}

let afterTax401K, futureRoth, futureOther, total, adjusted, monthly;

function calculate() {
  const get = id => parseFloat(document.getElementById(id).value);
  const age = get("age");
  const retirementAge = get("retirementAge");
  const endAge = get("endAge");
  const years = retirementAge - age;

  if (years <= 0 || endAge <= retirementAge) {
    alert("Check age inputs. Retirement age must be greater than current age and less than end age.");
    return;
  }

  const salaryGrowth = get("salaryGrowth") / 100;
  const inflation = get("inflation") / 100;
  const taxRate = get("taxRate") / 100;
  const rateOfReturn = get("rateOfReturn") / 100;

  let salary = get("annualSalary");
  const totalContribution = get("totalContribution") / 100;
  const current401K = get("current401K");

  const [future401K, history401K] = futureValueWithGrowth(
    current401K, rateOfReturn, years,
    year => salary * Math.pow(1 + salaryGrowth, year) * totalContribution
  );

  afterTax401K = future401K * (1 - taxRate);
  const adjusted401K = afterTax401K / Math.pow(1 + inflation, years);

  [futureRoth, historyRoth] = futureValueWithGrowth(get("currentRoth"), rateOfReturn, years, () => get("rothAnnualContribution"));
  const adjustedRoth = futureRoth / Math.pow(1 + inflation, years);

  [futureOther, historyOther] = futureValueWithGrowth(get("currentOther"), rateOfReturn, years, () => get("otherAnnualContribution"));
  const adjustedOther = futureOther / Math.pow(1 + inflation, years);

  total = afterTax401K + futureRoth + futureOther;
  adjusted = adjusted401K + adjustedRoth + adjustedOther;
  monthly = adjusted / ((endAge - retirementAge) * 12);

  document.getElementById("total401k").textContent = `401K After Tax: $${formatNumber(afterTax401K)} (Adj: $${formatNumber(adjusted401K)})`;
  document.getElementById("totalRoth").textContent = `Roth IRA: $${formatNumber(futureRoth)} (Adj: $${formatNumber(adjustedRoth)})`;
  document.getElementById("totalOther").textContent = `Other Investments: $${formatNumber(futureOther)} (Adj: $${formatNumber(adjustedOther)})`;
  document.getElementById("grandTotal").textContent = `Total at Retirement: $${formatNumber(total)}`;
  document.getElementById("adjustedTotal").textContent = `Purchasing Power Today: $${formatNumber(adjusted)}`;
  document.getElementById("monthly").textContent = `Monthly Allowance Estimate: $${formatNumber(monthly)}`;

  const ctx = document.getElementById('breakdownChart').getContext('2d');
  if (chart) chart.destroy();

  const chartType = document.getElementById("chartType").value;
  const type = chartType === "area" ? "line" : chartType;

  chart = new Chart(ctx, {
    type: type,
    data: {
      labels: Array.from({length: years}, (_, i) => `Year ${i + 1}`),
      datasets: [
        {
          label: '401K',
          data: history401K,
          borderColor: 'blue',
          backgroundColor: chartType === "area" ? 'rgba(0,0,255,0.1)' : 'transparent',
          fill: chartType === "area"
        },
        {
          label: 'Roth IRA',
          data: historyRoth,
          borderColor: 'green',
          backgroundColor: chartType === "area" ? 'rgba(0,128,0,0.1)' : 'transparent',
          fill: chartType === "area"
        },
        {
          label: 'Other',
          data: historyOther,
          borderColor: 'orange',
          backgroundColor: chartType === "area" ? 'rgba(255,165,0,0.1)' : 'transparent',
          fill: chartType === "area"
        }
      ]
    },
    options: {
      responsive: true,
      plugins: {
        title: { display: true, text: 'Growth Breakdown Over Time' }
      }
    }
  });

  document.getElementById("results").style.display = 'block';
}

function formatNumber(num) {
  return num.toLocaleString(undefined, {maximumFractionDigits: 2});
}

function toggleDarkMode() {
  document.body.classList.toggle("dark-mode");
  localStorage.setItem("darkMode", document.body.classList.contains("dark-mode"));
}

function downloadCSV() {
  const data = [
    ["Account", "Future Value", "Adjusted Value", "Monthly Allowance"],
    ["401K", formatNumber(afterTax401K), "", formatNumber(monthly)],
    ["Roth IRA", formatNumber(futureRoth), "", formatNumber(monthly)],
    ["Other Investments", formatNumber(futureOther), "", formatNumber(monthly)],
    ["Grand Total", formatNumber(total), formatNumber(adjusted), ""]
  ];
  const csvString = Papa.unparse(data);
  const blob = new Blob([csvString], { type: "text/csv" });
  const a = document.createElement("a");
  a.href = URL.createObjectURL(blob);
  a.download = "retirement_results.csv";
  a.click();
}

function downloadPDF() {
  const { jsPDF } = window.jspdf;
  const doc = new jsPDF();
  doc.text("Retirement Calculator Results", 10, 10);
  doc.text(`401K After Tax: $${formatNumber(afterTax401K)}`, 10, 20);
  doc.text(`Roth IRA: $${formatNumber(futureRoth)}`, 10, 30);
  doc.text(`Other Investments: $${formatNumber(futureOther)}`, 10, 40);
  doc.text(`Grand Total: $${formatNumber(total)}`, 10, 50);
  doc.text(`Adjusted Total: $${formatNumber(adjusted)}`, 10, 60);
  doc.text(`Monthly Allowance Estimate: $${formatNumber(monthly)}`, 10, 70);
  doc.save("retirement_results.pdf");
}

function downloadJSON() {
  const data = {
    afterTax401K, futureRoth, futureOther, total, adjusted, monthly
  };
  const blob = new Blob([JSON.stringify(data, null, 2)], { type: "application/json" });
  const a = document.createElement("a");
  a.href = URL.createObjectURL(blob);
  a.download = "retirement_results.json";
  a.click();
}

window.onload = () => {
  if (localStorage.getItem("darkMode") === "true") {
    document.body.classList.add("dark-mode");
  }
};
</script>
</body>
</html>
