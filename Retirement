<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
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

        body {
            font-family: 'Inter', sans-serif;
            padding: 2rem;
            background: linear-gradient(to right, #74ebd5, #ACB6E5);
            color: #333;
            transition: background-color 0.3s, color 0.3s;
        }
        .container {
            max-width: 950px;
            margin: 0 auto;
            background: white;
            padding: 2rem;
            border-radius: 15px;
            box-shadow: 0 4px 20px rgba(0,0,0,0.15);
        }
        input {
            width: 100%;
            padding: 0.5rem;
            margin: 0.3rem 0 1rem;
            border: 1px solid #ccc;
            border-radius: 8px;
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
            color: #2c3e50;
        }
        .results {
            margin-top: 2rem;
            background: #ecf9ec;
            padding: 1rem;
            border-radius: 8px;
            box-shadow: inset 0 0 10px rgba(0,0,0,0.05);
        }
        canvas {
            margin-top: 2rem;
        }
        a {
            color: #007BFF;
        }
        .dark-mode {
            background-color: #2c3e50;
            color: white;
        }
        .dark-mode input, .dark-mode button {
            background-color: #34495e;
            color: white;
        }
        .dark-mode .results {
            background: #34495e;
        }
        .dark-mode h1, .dark-mode h3 {
            color: white;
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
    </div>
</div>

<script>
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

function calculate() {
    const get = id => parseFloat(document.getElementById(id).value);
    const age = get("age");
    const retirementAge = get("retirementAge");
    const years = retirementAge - age;

    if (years <= 0) {
        alert("Retirement age must be greater than current age.");
        return;
    }

    const salaryGrowth = get("salaryGrowth") / 100;
    const inflation = get("inflation") / 100;
    const taxRate = get("taxRate") / 100;
    const rateOfReturn = get("rateOfReturn") / 100;

    // 401K with growing salary
    let salary = get("annualSalary");
    const totalContribution = get("totalContribution") / 100;
    const current401K = get("current401K");

    const [future401K, history401K] = futureValueWithGrowth(
        current401K,
        rateOfReturn,
        years,
        year => salary * Math.pow(1 + salaryGrowth, year) * totalContribution
    );

    const afterTax401K = future401K * (1 - taxRate);
    const adjusted401K = afterTax401K / Math.pow(1 + inflation, years);

    // Roth
    const [futureRoth, historyRoth] = futureValueWithGrowth(
        get("currentRoth"),
        rateOfReturn,
        years,
        () => get("rothAnnualContribution")
    );
    const adjustedRoth = futureRoth / Math.pow(1 + inflation, years);

    // Other Investments
    const [futureOther, historyOther] = futureValueWithGrowth(
        get("currentOther"),
        rateOfReturn,
        years,
        () => get("otherAnnualContribution")
    );
    const adjustedOther = futureOther / Math.pow(1 + inflation, years);

    // Results
    const total = afterTax401K + futureRoth + futureOther;
    const adjusted = adjusted401K + adjustedRoth + adjustedOther;
    const monthly = adjusted / ((85 - retirementAge) * 12);

    document.getElementById("total401k").textContent = `401K After Tax: $${formatNumber(afterTax401K)} (Adj: $${formatNumber(adjusted401K)})`;
    document.getElementById("totalRoth").textContent = `Roth IRA: $${formatNumber(futureRoth)} (Adj: $${formatNumber(adjustedRoth)})`;
    document.getElementById("totalOther").textContent = `Other Investments: $${formatNumber(futureOther)} (Adj: $${formatNumber(adjustedOther)})`;
    document.getElementById("grandTotal").textContent = `Total at Retirement: $${formatNumber(total)}`;
    document.getElementById("adjustedTotal").textContent = `Purchasing Power Today: $${formatNumber(adjusted)}`;
    document.getElementById("monthly").textContent = `Monthly Allowance Estimate: $${formatNumber(monthly)}`;

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
            plugins: {
                title: { display: true, text: 'Growth Breakdown Over Time' }
            }
        }
    });

    document.getElementById("results").style.display = 'block';
}

function formatNumber(num) {
    return num.toLocaleString();
}

function toggleDarkMode() {
    document.body.classList.toggle("dark-mode");
}

function downloadCSV() {
    const data = [
        ["Account", "Future Value", "Adjusted Value", "Monthly Allowance"],
        ["401K", formatNumber(afterTax401K), formatNumber(adjusted401K), formatNumber(monthly)],
        ["Roth IRA", formatNumber(futureRoth), formatNumber(adjustedRoth), formatNumber(monthly)],
        ["Other Investments", formatNumber(futureOther), formatNumber(adjustedOther), formatNumber(monthly)],
        ["Grand Total", formatNumber(total), formatNumber(adjusted), ""]
    ];
    const csv = Papa.parse(data, { header: true, dynamicTyping: true });
    const csvString = Papa.unparse(csv);
    const blob = new Blob([csvString], { type: "text/csv" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
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
</script>
</body>
</html>
