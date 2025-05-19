<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Retirement Calculator</title>
  <meta name="description" content="Interactive Retirement Calculator by Meeks">
  <meta name="author" content="MaddMeeks">
  <meta name="keywords" content="retirement calculator, 401k, roth IRA, investment, budget forecast">
  <link rel="canonical" href="https://github.com/MaddMeeks/Meeks">
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jsPDF/2.5.1/jspdf.umd.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.0/papaparse.min.js"></script>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap');

    :root {
      --primary: #4CAF50;
      --primary-dark: #388E3C;
      --bg-light: #f8f9fa;
      --bg-dark: #2c3e50;
      --text-light: #333;
      --text-dark: #f5f5f5;
      --input-bg-light: #fff;
      --input-bg-dark: #34495e;
    }

    body {
      font-family: 'Inter', sans-serif;
      background-color: var(--bg-light);
      color: var(--text-light);
      transition: background-color 0.3s, color 0.3s;
    }

    .dark-mode {
      background-color: var(--bg-dark);
      color: var(--text-dark);
    }

    .dark-mode .card,
    .dark-mode input,
    .dark-mode select {
      background-color: var(--input-bg-dark);
      color: var(--text-dark);
    }

    .dark-mode input,
    .dark-mode select {
      border-color: #555;
    }

    .dark-mode .nav-tabs .nav-link.active {
      background-color: #1e2a36;
      color: #fff;
    }

    .highlight-box {
      background: #d1e7dd;
      border: 1px solid #badbcc;
      border-radius: 10px;
      padding: 1rem;
      margin-bottom: 1rem;
    }

    .dark-mode .highlight-box {
      background: #2d4b3f;
      border-color: #1f392e;
    }

    .cta {
      font-size: 1.2rem;
      font-weight: 600;
      color: var(--primary-dark);
    }

     .brand-logo {
      font-size: 2rem;
      font-weight: 700;
      color: var(--primary);
    }

    .subtitle {
      font-size: 1.1rem;
      color: #555;
      font-style: italic;
    }

    .result-card {
      border-left: 5px solid var(--primary-dark);
      background-color: #eafaf1;
      padding: 1rem;
      border-radius: 10px;
      margin-bottom: 1rem;
      box-shadow: 0 1px 4px rgba(0, 0, 0, 0.05);
    }

    .result-card h5 {
      font-weight: 600;
    }

    .dark-mode .result-card {
      background-color: #244130;
      border-color: #3aa06b;
    }

    .dark-mode .subtitle {
      color: #ccc;
	}
  </style>
</head>
<body>
<div class="container py-5">
   <div class="text-center mb-4">
    <div class="brand-logo">Meeks</div>
    <div class="subtitle">Retirement Calculator</div>
    <a href="https://github.com/MaddMeeks/Meeks.git" target="_blank" class="d-block mt-2">View Source on GitHub</a>
    <button class="btn btn-outline-secondary mt-3" onclick="toggleDarkMode()">🌙</button>
  </div>

  <ul class="nav nav-tabs" id="calculatorTabs" role="tablist">
    <li class="nav-item" role="presentation">
      <button class="nav-link active" id="form-tab" data-bs-toggle="tab" data-bs-target="#form" type="button" role="tab">Input Form</button>
    </li>
    <li class="nav-item" role="presentation">
      <button class="nav-link" id="results-tab" data-bs-toggle="tab" data-bs-target="#results" type="button" role="tab">Results</button>
    </li>
  </ul>

  <div class="tab-content mt-4" id="calculatorTabsContent">
    <div class="tab-pane fade show active" id="form" role="tabpanel">
      <!-- Input form content -->
      <form onsubmit="event.preventDefault(); calculate();">
        <div class="row g-3">
          <div class="col-12 col-md-6 col-lg-4">
            <label for="age" class="form-label">Current Age</label>
            <input type="number" id="age" class="form-control" required>
          </div>
          <div class="col-12 col-md-6 col-lg-4">
            <label for="retirementAge" class="form-label">Retirement Age</label>
            <input type="number" id="retirementAge" class="form-control" required>
          </div>
          <div class="col-12 col-md-6 col-lg-4">
            <label for="endAge" class="form-label">Expected Age of Life</label>
            <input type="number" id="endAge" class="form-control" required>
          </div>

          <div class="col-12 col-md-6 col-lg-4">
            <label for="annualSalary" class="form-label">Current Annual Salary</label>
            <input type="number" id="annualSalary" class="form-control" step="any" required>
          </div>
          <div class="col-12 col-md-6 col-lg-4">
            <label for="salaryGrowth" class="form-label">Annual Salary Growth (%) <small>Avg: 3%</small> </label>
            <input type="number" id="salaryGrowth" class="form-control" step="any" required>
          </div>
          <div class="col-12 col-md-6 col-lg-4">
            <label for="inflation" class="form-label">Inflation Rate (%) <small>Avg: 2-3%</small> </label>
            <input type="number" id="inflation" class="form-control" step="any" required>
          </div>
          <div class="col-12 col-md-6 col-lg-4">
            <label for="taxRate" class="form-label">Tax Rate (%) <small>Avg: 15-20%</small> </label>
            <input type="number" id="taxRate" class="form-control" step="any" required>
          </div>
          <div class="col-12 col-md-6 col-lg-4">
            <label for="rateOfReturn" class="form-label">Annual Market Return (%) <small>Avg: 6-10%</small> </label>
            <input type="number" id="rateOfReturn" class="form-control" step="any" required>
          </div>

          <div class="col-12 col-md-6 col-lg-4">
            <label for="current401K" class="form-label">Current 401K Balance</label>
            <input type="number" id="current401K" class="form-control" step="any" required>
          </div>
          <div class="col-12 col-md-6 col-lg-4">
            <label for="totalContribution" class="form-label">401K Contribution (%) <small>including employer contribution</small> </label>
            <input type="number" id="totalContribution" class="form-control" step="any" required>
          </div>

          <div class="col-12 col-md-6 col-lg-4">
            <label for="currentRoth" class="form-label">Current Roth IRA Balance</label>
            <input type="number" id="currentRoth" class="form-control" step="any" required>
          </div>
          <div class="col-12 col-md-6 col-lg-4">
            <label for="rothAnnualContribution" class="form-label">Annual Roth IRA Contribution</label>
            <input type="number" id="rothAnnualContribution" class="form-control" step="any" required>
          </div>

          <div class="col-12 col-md-6 col-lg-4">
            <label for="currentOther" class="form-label">Current Other Investments</label>
            <input type="number" id="currentOther" class="form-control" step="any" required>
          </div>
          <div class="col-12 col-md-6 col-lg-4">
            <label for="otherAnnualContribution" class="form-label">Annual Other Investment Contribution</label>
            <input type="number" id="otherAnnualContribution" class="form-control" step="any" required>
          </div>
        </div>
        <button type="submit" class="btn btn-primary mt-4">Calculate</button>
      </form>

      <button class="btn btn-link mt-3" data-bs-toggle="collapse" data-bs-target="#advancedSettings" aria-expanded="false" aria-controls="advancedSettings">
        Advanced Settings ▼
      </button>
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

    <div class="tab-pane fade" id="results" role="tabpanel">
      <div class="highlight-box">
        <h4 class="cta mb-3">Results Summary</h4>
        <p id="total401k"></p>
        <p id="totalRoth"></p>
        <p id="totalOther"></p>
        
        <!-- Highlighted Results -->
        <div class="result-card">
          <h5>Total Savings at Retirement (After Tax)</h5>
          <p id="grandTotal" class="mb-0 fs-5 fw-bold text-success"></p>
        </div>

        <div class="result-card">
          <h5>Inflation Adjusted Total (Present Value)</h5>
          <p id="adjustedTotal" class="mb-0 fs-5 fw-bold text-success"></p>
        </div>

        <div class="result-card">
          <h5>Estimated Monthly Withdrawal (Present Value)</h5>
          <p id="monthly" class="mb-0 fs-5 fw-bold text-success"></p>
        </div>
      </div>

      <canvas id="breakdownChart" width="400" height="200"></canvas>

      <div class="mt-4 d-flex gap-2">
        <button class="btn btn-outline-primary" onclick="downloadCSV()">Download CSV</button>
        <button class="btn btn-outline-danger" onclick="downloadPDF()">Download PDF</button>
        <button class="btn btn-outline-dark" onclick="downloadJSON()">Download JSON</button>
      </div>
    </div>
  </div>
</div>
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
<script>
let chart;

function toggleDarkMode() {
  const body = document.body;
  body.classList.toggle("dark-mode");
  const btn = document.querySelector('button[onclick="toggleDarkMode()"]');
  btn.textContent = body.classList.contains("dark-mode") ? "☀️" : "🌙";
}


function calculate() {
  // Read inputs and parse as floats
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

  // Calculate years until retirement and years in retirement
  const yearsToRetirement = retirementAge - age;
  const yearsInRetirement = endAge - retirementAge;

  // Initialize accumulators for balances over time
  let balance401K = current401K;
  let balanceRoth = currentRoth;
  let balanceOther = currentOther;

  // Arrays for chart data and yearly balances
  const years = [];
  const totalBalances = [];

  // Calculate 401K balance at retirement with annual contributions and growth
  for(let i = 0; i < yearsToRetirement; i++) {
    // Salary growth each year
    const salaryThisYear = annualSalary * Math.pow(1 + salaryGrowth, i);

    // Contribution this year (percentage of salary)
    const contribution401K = salaryThisYear * totalContribution;

    // Grow balance 401K by rate of return and add contribution
    balance401K = (balance401K + contribution401K) * (1 + rateOfReturn);

    // For chart, store total balance this year (just pre-retirement for now)
    years.push(age + i);
    totalBalances.push(balance401K + balanceRoth + balanceOther);
  }

  // Calculate Roth IRA balance growth until retirement with contributions
  for(let i = 0; i < yearsToRetirement; i++) {
    balanceRoth = (balanceRoth + rothAnnualContribution) * (1 + rateOfReturn);
  }

  // Calculate Other investments balance growth until retirement with contributions
  for(let i = 0; i < yearsToRetirement; i++) {
    balanceOther = (balanceOther + otherAnnualContribution) * (1 + rateOfReturn);
  }

// Apply tax to 401K at retirement
const afterTax401K = balance401K * (1 - taxRate);

// Combine totals at retirement using after-tax 401K
const totalAtRetirement = afterTax401K + balanceRoth + balanceOther;

// Adjusted total factoring in inflation to present value
const adjustedTotal = totalAtRetirement / Math.pow(1 + inflation, yearsToRetirement);

// Monthly withdrawal over retirement years
const monthlyWithdrawalBeforeTax = totalAtRetirement / (yearsInRetirement * 12);
const monthlyWithdrawalAfterTax = monthlyWithdrawalBeforeTax; // Already accounted for tax in 401K


  // Output to results fields
  document.getElementById("total401k").textContent = `401K Balance at Retirement (After Tax): $${afterTax401K.toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 })}`;
  document.getElementById("totalRoth").textContent = `Roth IRA Balance at Retirement: $${balanceRoth.toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 })}`;
  document.getElementById("totalOther").textContent = `Other Investments Balance at Retirement: $${balanceOther.toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 })}`;
  document.getElementById("grandTotal").textContent = `Total Savings at Retirement (After Tax): $${totalAtRetirement.toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 })}`;
  document.getElementById("adjustedTotal").textContent = `Inflation Adjusted Total (Present Value): $${adjustedTotal.toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 })}`;
  document.getElementById("monthly").textContent = `Estimated Monthly Withdrawal (Present Value): $${monthlyWithdrawalAfterTax.toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 })}`;

  // Prepare chart data
  const ctx = document.getElementById('breakdownChart').getContext('2d');
  if (chart) {
    chart.destroy();
  }

  const chartType = document.getElementById('chartType').value;

  chart = new Chart(ctx, {
    type: chartType,
    data: {
      labels: ['401K', 'Roth IRA', 'Other Investments'],
      datasets: [{
        label: 'Investment Breakdown',
        data: [afterTax401K.toFixed(2), balanceRoth.toFixed(2), balanceOther.toFixed(2)],
        backgroundColor: [
          'rgba(75, 192, 192, 0.6)',
          'rgba(255, 159, 64, 0.6)',
          'rgba(153, 102, 255, 0.6)'
        ],
        borderColor: [
          'rgba(75, 192, 192, 1)',
          'rgba(255, 159, 64, 1)',
          'rgba(153, 102, 255, 1)'
        ],
        borderWidth: 1
      }]
    },
    options: {
      responsive: true,
      plugins: {
        legend: { position: 'bottom' },
        title: {
          display: true,
          text: 'Investment Breakdown at Retirement'
        }
      }
    }
  });

  // Switch to Results tab so user can immediately see results
  const resultsTab = new bootstrap.Tab(document.querySelector('#results-tab'));
  resultsTab.show();
}

function downloadCSV() {
  // Gather data
  const data = [
    ['Category', 'Amount'],
    ['401K Balance at Retirement', document.getElementById("total401k").textContent.replace(/[^0-9.]/g, '')],
    ['Roth IRA Balance at Retirement', document.getElementById("totalRoth").textContent.replace(/[^0-9.]/g, '')],
    ['Other Investments Balance at Retirement', document.getElementById("totalOther").textContent.replace(/[^0-9.]/g, '')],
    ['Total Savings at Retirement', document.getElementById("grandTotal").textContent.replace(/[^0-9.]/g, '')],
    ['Inflation Adjusted Total (Present Value)', document.getElementById("adjustedTotal").textContent.replace(/[^0-9.]/g, '')],
    ['Estimated Monthly Withdrawal (After Tax)', document.getElementById("monthly").textContent.replace(/[^0-9.]/g, '')],
  ];

  const csv = Papa.unparse(data);

  const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
  const link = document.createElement("a");
  link.href = URL.createObjectURL(blob);
  link.download = "retirement_results.csv";
  link.click();
}

function downloadPDF() {
  const { jsPDF } = window.jspdf;
  const doc = new jsPDF();

  doc.setFontSize(16);
  doc.text("Retirement Calculator Results", 10, 10);

  let y = 20;
  [
    "total401k",
    "totalRoth",
    "totalOther",
    "grandTotal",
    "adjustedTotal",
    "monthly"
  ].forEach(id => {
    const text = document.getElementById(id).textContent;
    doc.text(text, 10, y);
    y += 10;
  });

  doc.save("retirement_results.pdf");
}


function downloadJSON() {
  const data = {
    "401K Balance at Retirement": document.getElementById("total401k").textContent.replace(/[^0-9.]/g, ''),
    "Roth IRA Balance at Retirement": document.getElementById("totalRoth").textContent.replace(/[^0-9.]/g, ''),
    "Other Investments Balance at Retirement": document.getElementById("totalOther").textContent.replace(/[^0-9.]/g, ''),
    "Total Savings at Retirement": document.getElementById("grandTotal").textContent.replace(/[^0-9.]/g, ''),
    "Inflation Adjusted Total (Present Value)": document.getElementById("adjustedTotal").textContent.replace(/[^0-9.]/g, ''),
    "Estimated Monthly Withdrawal (After Tax)": document.getElementById("monthly").textContent.replace(/[^0-9.]/g, '')
  };

  const jsonStr = JSON.stringify(data, null, 2);
  const blob = new Blob([jsonStr], { type: 'application/json' });
  const link = document.createElement('a');
  link.href = URL.createObjectURL(blob);
  link.download = 'retirement_results.json';
  link.click();
}
</script>
</body>
</html>
