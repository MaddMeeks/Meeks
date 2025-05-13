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
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap');

        :root {
            --bg-color: white;
            --text-color: #333;
            --container-bg: white;
            --result-bg: #ecf9ec;
        }

        body.dark {
            --bg-color: #1e1e1e;
            --text-color: #eee;
            --container-bg: #2c2c2c;
            --result-bg: #333;
        }

        body {
            font-family: 'Inter', sans-serif;
            padding: 2rem;
            background-color: var(--bg-color);
            color: var(--text-color);
            transition: background 0.3s, color 0.3s;
        }

        .container {
            max-width: 950px;
            margin: 0 auto;
            background: var(--container-bg);
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
            color: var(--text-color);
        }

        .results {
            margin-top: 2rem;
            background: var(--result-bg);
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

        .dark-toggle {
            position: absolute;
            top: 20px;
            right: 30px;
        }

        .dark-toggle button {
            background-color: #444;
            color: #fff;
            border: none;
            padding: 0.5rem 1rem;
            border-radius: 8px;
        }

        .note {
            font-size: 0.8rem;
            color: #555;
            margin-top: -10px;
        }
    </style>
</head>
<body>
<div class="dark-toggle">
    <button onclick="toggleDarkMode()">Toggle Dark Mode</button>
</div>

<div class="container">
    <h1>Retirement Calculator</h1>
    <p style="text-align:center"><a href="https://github.com/MaddMeeks/Meeks.git" target="_blank">View Source on GitHub</a></p>
    <form id="retirementForm">
        <label>Current Age:<input type="number" id="age" value="30"></label>
        <label>Retirement Age:<input type="number" id="retirementAge" value="65"></label>
        <label>Expected Age To Live:<input type="number" id="expectedAge" value="95"></label>

        <h3>401K</h3>
        <label>Current 401K Amount:<input type="number" id="current401K" value="0"></label>
        <label>Total Contribution % (include employer contributions):<input type="number" id="totalContribution" value="10"></label>
        <label>401K Rate of Return %:<input type="number" id="rateOfReturn401K" value="7"></label>

        <h3>Roth IRA</h3>
        <label>Current Roth Amount:<input type="number" id="currentRoth" value="0"></label>
        <label>Annual Roth Contribution:<input type="number" id="rothAnnualContribution" value="6500"></label>
        <label>Roth Rate of Return %:<input type="number" id="rateOfReturnRoth" value="7"></label>

        <h3>Other Investments</h3>
        <label>Current Investment:<input type="number" id="currentOther" value="0"></label>
        <label>Annual Contribution:<input type="number" id="otherAnnualContribution" value="0"></label>
        <label>Other Investment Return %:<input type="number" id="rateOfReturnOther" value="7"></label>

        <h3>Economic Factors</h3>
        <label>Annual Salary:<input type="number" id="annualSalary" value="60000"></label>
        <label>Salary Growth %:<input type="number" id="salaryGrowth" value="2"></label>
        <label>Inflation %:<input type="number" id="inflation" value="2"></label>
        <label>Tax Rate %:<input type="number" id="taxRate" value="25"></label>

        <button type="button" onclick="calculate()">Calculate</button>
    </form>

    <div class="results" id="results" style="display:none">
        <h3>Results</h3>
        <p id="total401k"></p>
        <p id="totalRoth"></p>
        <p id="totalOther"></p>
        <p id="grandTotal"></p>
        <p id="adjustedTotal"></p>
        <p id="monthlyAllowance"></p>
        <p class="note">Accounted for inflation increase each year.</p>
        <canvas id="breakdownChart" width="400" height="200"></canvas>
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

function formatCurrency(num) {
    return num.toLocaleString('en-US', { style: 'currency', currency: 'USD' });
}

function calculate() {
    const get = id => parseFloat(document.getElementById(id).value);
    const age = get("age");
    const retirementAge = get("retirementAge");
    const expectedAge = get("expectedAge");
    const years = expectedAge - retirementAge;

    const salaryGrowth = get("salaryGrowth") / 100;
    const inflation = get("inflation") / 100;
    const taxRate = get("taxRate") / 100;

    let salary = get("annualSalary");
    const totalContribution = get("totalContribution") / 100;
    const rateOfReturn401K = get("rateOfReturn401K") / 100;
    const current401K = get("current401K");

    const [future401K, history401K] = futureValueWithGrowth(
        current401K,
        rateOfReturn401K,
        years,
        year => salary * Math.pow(1 + salaryGrowth, year) * totalContribution
    );

    const afterTax401K = future401K * (1 - taxRate);
    const adjusted401K = afterTax401K / Math.pow(1 + inflation, years);

    const [futureRoth, historyRoth] = futureValueWithGrowth(
        get("currentRoth"),
        get("rateOfReturnRoth") / 100,
        years,
        () => get("rothAnnualContribution")
    );
    const adjustedRoth = futureRoth / Math.pow(1 + inflation, years);

    const [futureOther, historyOther] = futureValueWithGrowth(
        get("currentOther"),
        get("rateOfReturnOther") / 100,
        years,
        () => get("otherAnnualContribution")
    );
    const adjustedOther = futureOther / Math.pow(1 + inflation, years);

    const total = afterTax401K + futureRoth + futureOther;
    const adjusted = adjusted401K + adjustedRoth + adjustedOther;
    const monthlyAllowance = adjusted / ((expectedAge - retirementAge) * 12);

    document.getElementById("total401k").textContent = `401K After Tax: ${formatCurrency(afterTax401K)} (Adj: ${formatCurrency(adjusted401K)})`;
    document.getElementById("totalRoth").textContent = `Roth IRA: ${formatCurrency(futureRoth)} (Adj: ${formatCurrency(adjustedRoth)})`;
    document.getElementById("totalOther").textContent = `Other Investments: ${formatCurrency(futureOther)} (Adj: ${formatCurrency(adjustedOther)})`;
    document.getElementById("grandTotal").textContent = `Total at Retirement: ${formatCurrency(total)}`;
    document.getElementById("adjustedTotal").textContent = `Purchasing Power Today: ${formatCurrency(adjusted)}`;
    document.getElementById("monthlyAllowance").textContent = `Monthly Allowance Estimate: ${formatCurrency(monthlyAllowance)}`;

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

function toggleDarkMode() {
    document.body.classList.toggle("dark");
}
</script>
</body>
</html>
