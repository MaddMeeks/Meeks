<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Retirement Calculator</title>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jsPDF/2.5.1/jspdf.umd.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.0/papaparse.min.js"></script>
  <style>
    body { font-family: 'Inter', sans-serif; }
    .brand-logo { font-size: 2rem; font-weight: 700; color: #4CAF50; }
    .subtitle { font-size: 1.1rem; color: #555; font-style: italic; }
    .dark-mode { background-color: #2c3e50; color: #f5f5f5; }
    .dark-mode .card, .dark-mode input, .dark-mode select { background-color: #34495e; color: #f5f5f5; }
    .result-card { border-left: 5px solid #388E3C; background: #eafaf1; padding: 1rem; border-radius: 10px; margin-bottom: 1rem; }
    .dark-mode .result-card { background-color: #244130; border-color: #3aa06b; }
  </style>
</head>
<body>
<div class="container py-5">
  <div class="text-center mb-4">
    <div class="brand-logo">Meeks</div>
    <div class="subtitle">Retirement Calculator</div>
    <button class="btn btn-outline-secondary mt-3" onclick="toggleDarkMode()">🌙</button>
  </div>

  <!-- NAV TABS -->
  <ul class="nav nav-tabs" id="calculatorTabs" role="tablist">
    <li class="nav-item"><button class="nav-link active" data-bs-toggle="tab" data-bs-target="#form">Inputs</button></li>
    <li class="nav-item"><button class="nav-link" data-bs-toggle="tab" data-bs-target="#results">Results</button></li>
    <li class="nav-item"><button class="nav-link" data-bs-toggle="tab" data-bs-target="#charts">Charts</button></li>
    <li class="nav-item"><button class="nav-link" data-bs-toggle="tab" data-bs-target="#export">Export</button></li>
  </ul>

  <!-- TAB CONTENT -->
  <div class="tab-content mt-4">
    <!-- INPUTS -->
    <div class="tab-pane fade show active" id="form">
      <form onsubmit="event.preventDefault(); calculate();">
        <div class="row g-3">
          <div class="col-md-4"><label class="form-label">Current Age</label><input type="number" id="age" class="form-control" required></div>
          <div class="col-md-4"><label class="form-label">Retirement Age</label><input type="number" id="retirementAge" class="form-control" required></div>
          <div class="col-md-4"><label class="form-label">Expected Age of Life</label><input type="number" id="endAge" class="form-control" required></div>
          <div class="col-md-4"><label class="form-label">Current Annual Salary</label><input type="number" id="annualSalary" class="form-control" step="any" required></div>
          <div class="col-md-4"><label class="form-label">Annual Salary Growth (%)</label><input type="number" id="salaryGrowth" class="form-control" step="any" required></div>
          <div class="col-md-4"><label class="form-label">Inflation Rate (%)</label><input type="number" id="inflation" class="form-control" step="any" required></div>
          <div class="col-md-4"><label class="form-label">Tax Rate (%)</label><input type="number" id="taxRate" class="form-control" step="any" required></div>
          <div class="col-md-4"><label class="form-label">Annual Market Return (%)</label><input type="number" id="rateOfReturn" class="form-control" step="any" required></div>
          <div class="col-md-4"><label class="form-label">Current 401K Balance</label><input type="number" id="current401K" class="form-control" step="any" required></div>
          <div class="col-md-4"><label class="form-label">401K Contribution (%)</label><input type="number" id="totalContribution" class="form-control" step="any" required></div>
          <div class="col-md-4"><label class="form-label">Current Roth IRA Balance</label><input type="number" id="currentRoth" class="form-control" step="any" required></div>
          <div class="col-md-4"><label class="form-label">Annual Roth IRA Contribution</label><input type="number" id="rothAnnualContribution" class="form-control" step="any" required></div>
          <div class="col-md-4"><label class="form-label">Current Other Investments</label><input type="number" id="currentOther" class="form-control" step="any" required></div>
          <div class="col-md-4"><label class="form-label">Annual Other Investment Contribution</label><input type="number" id="otherAnnualContribution" class="form-control" step="any" required></div>
        </div>
        <button type="submit" class="btn btn-primary mt-4">Calculate</button>
      </form>

      <button class="btn btn-link mt-3" data-bs-toggle="collapse" data-bs-target="#advancedSettings">Advanced Settings ▼</button>
      <div class="collapse mt-2" id="advancedSettings">
        <div class="card card-body">
          <label for="chartType">Chart Type:</label>
          <select id="chartType" class="form-select">
            <option value="bar">Bar</option>
            <option value="line">Line</option>
            <option value="pie">Pie</option>
            <option value="doughnut">Doughnut</option>
          </select>
        </div>
      </div>
    </div>

    <!-- RESULTS -->
    <div class="tab-pane fade" id="results">
      <div class="result-card"><p id="total401k"></p></div>
      <div class="result-card"><p id="totalRoth"></p></div>
      <div class="result-card"><p id="totalOther"></p></div>
      <div class="result-card fw-bold"><p id="grandTotal"></p></div>
      <div class="result-card fw-bold"><p id="adjustedTotal"></p></div>
      <div class="result-card fw-bold"><p id="monthly"></p></div>
    </div>

    <!-- CHARTS -->
    <div class="tab-pane fade" id="charts">
      <canvas id="breakdownChart" width="400" height="200"></canvas>
    </div>

    <!-- EXPORT -->
    <div class="tab-pane fade" id="export">
      <button class="btn btn-outline-primary" onclick="downloadCSV()">Download CSV</button>
      <button class="btn btn-outline-danger" onclick="downloadPDF()">Download PDF</button>
      <button class="btn btn-outline-dark" onclick="downloadJSON()">Download JSON</button>
    </div>
  </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
<script>
let chart;

function toggleDarkMode() {
  document.body.classList.toggle("dark-mode");
  const btn = document.querySelector('button[onclick="toggleDarkMode()"]');
  btn.textContent = document.body.classList.contains("dark-mode") ? "☀️" : "🌙";
}

function calculate() {
  const age = parseInt(document.getElementById("age").value);
  const retirementAge = parseInt(document.getElementById("retirementAge").value);
  const endAge = parseInt(document.getElementById("endAge").value);
  const annualSalary = parseFloat(document.getElementById("annualSalary").value);
  const salaryGrowth = parseFloat(document.getElementById("salaryGrowth").value) / 100;
  const inflation = parseFloat(document.getElementById("inflation").value) / 100;
  const taxRate = parseFloat(document.getElementById("taxRate").value) / 100;
  const rateOfReturn = parseFloat(document.getElementById("rateOfReturn").value) / 100;
  const current401K = parseFloat(document.getElementById("current401K").value);
  const totalContribution = parseFloat(document.getElementById("totalContribution").value) / 100;
  const currentRoth = parseFloat(document.getElementById("currentRoth").value);
  const rothAnnualContribution = parseFloat(document.getElementById("rothAnnualContribution").value);
  const currentOther = parseFloat(document.getElementById("currentOther").value);
  const otherAnnualContribution = parseFloat(document.getElementById("otherAnnualContribution").value);

  const yearsToRetirement = retirementAge - age;
  const yearsInRetirement = endAge - retirementAge;

  let balance401K = current401K;
  let balanceRoth = currentRoth;
  let balanceOther = currentOther;

  for(let i = 0; i < yearsToRetirement; i++) {
    const salaryThisYear = annualSalary * Math.pow(1 + salaryGrowth, i);
    const contribution401K = salaryThisYear * totalContribution;
    balance401K = (balance401K + contribution401K) * (1 + rateOfReturn);
    balanceRoth = (balanceRoth + rothAnnualContribution) * (1 + rateOfReturn);
    balanceOther = (balanceOther + otherAnnualContribution) * (1 + rateOfReturn);
  }

  const afterTax401K = balance401K * (1 - taxRate);
  const totalAtRetirement = afterTax401K + balanceRoth + balanceOther;
  const adjustedTotal = totalAtRetirement / Math.pow(1 + inflation, yearsToRetirement);
  const realRate = (1 + rateOfReturn) / (1 + inflation) - 1;
  const monthlyWithdrawal = (adjustedTotal * realRate) /
    (1 - Math.pow(1 + realRate, -yearsInRetirement)) / 12;

  document.getElementById("total401k").textContent = `401K Balance (After Tax): $${afterTax401K.toLocaleString()}`;
  document.getElementById("totalRoth").textContent = `Roth IRA Balance: $${balanceRoth.toLocaleString()}`;
  document.getElementById("totalOther").textContent = `Other Investments: $${balanceOther.toLocaleString()}`;
  document.getElementById("grandTotal").textContent = `Total Savings: $${totalAtRetirement.toLocaleString()}`;
  document.getElementById("adjustedTotal").textContent = `Inflation Adjusted Total: $${adjustedTotal.toLocaleString()}`;
  document.getElementById("monthly").textContent = `Estimated Monthly Withdrawal: $${monthlyWithdrawal.toLocaleString()}`;

  const ctx = document.getElementById('breakdownChart').getContext('2d');
  if (chart) chart.destroy();
  chart = new Chart(ctx, {
    type: document.getElementById('chartType').value,
    data: {
      labels: ['401K', 'Roth IRA', 'Other'],
      datasets: [{ label: 'Breakdown', data: [afterTax401K, balanceRoth, balanceOther] }]
    }
  });

  new bootstrap.Tab(document.querySelector('#results')).show();
}

function downloadCSV() {
  const data = [
    ["401K", document.getElementById("total401k").textContent],
    ["Roth", document.getElementById("totalRoth").textContent],
    ["Other", document.getElementById("totalOther").textContent],
    ["Total", document.getElementById("grandTotal").textContent],
    ["Adjusted", document.getElementById("adjustedTotal").textContent],
    ["Monthly", document.getElementById("monthly").textContent]
  ];
  const csv = Papa.unparse(data);
  const blob = new Blob([csv], { type: "text/csv" });
  const link = document.createElement("a");
  link.href = URL.createObjectURL(blob);
  link.download = "retirement_results.csv";
  link.click();
}

function downloadPDF() {
  const { jsPDF } = window.jspdf;
  const doc = new jsPDF();
  doc.setFontSize(16);
  doc.text("Retirement Results", 10, 10);
  let y = 20;
  ["total401k","totalRoth","totalOther","grandTotal","adjustedTotal","monthly"].forEach(id=>{
    doc.text(document.getElementById(id).textContent,10,y);
    y+=10;
  });
  doc.save("retirement_results.pdf");
}

function downloadJSON() {
  const data = {
    "401K": document.getElementById("total401k").textContent,
    "Roth": document.getElementById("totalRoth").textContent,
    "Other": document.getElementById("totalOther").textContent,
    "Total": document.getElementById("grandTotal").textContent,
    "Adjusted": document.getElementById("adjustedTotal").textContent,
    "Monthly": document.getElementById("monthly").textContent
  };
  const blob = new Blob([JSON.stringify(data, null, 2)], { type: "application/json" });
  const link = document.createElement("a");
  link.href = URL.createObjectURL(blob);
  link.download = "retirement_results.json";
  link.click();
}
</script>
</body>
</html>
