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

        body {
            font-family: 'Inter', sans-serif;
            padding: 2rem;
            background: linear-gradient(to right, #74ebd5, #ACB6E5);
            color: #333;
            animation: fadeIn 1s ease-in;
        }
        .container {
            max-width: 950px;
            margin: 0 auto;
            background: white;
            padding: 2rem;
            border-radius: 15px;
            box-shadow: 0 4px 20px rgba(0,0,0,0.15);
            animation: slideIn 1s ease-out;
            transition: transform 0.5s ease, box-shadow 0.5s ease;
        }
        .container:hover {
            transform: scale(1.01);
            box-shadow: 0 6px 30px rgba(0,0,0,0.2);
        }
        input {
            width: 100%;
            padding: 0.5rem;
            margin: 0.3rem 0 1rem;
            border: 1px solid #ccc;
            border-radius: 8px;
            transition: box-shadow 0.3s ease;
        }
        input:focus {
            box-shadow: 0 0 5px rgba(0, 123, 255, 0.5);
            outline: none;
        }
        label {
            font-weight: 600;
            display: block;
            margin-top: 1rem;
            animation: fadeInUp 0.6s ease;
        }
        button {
            margin-top: 1.5rem;
            padding: 0.75rem 2rem;
            font-size: 1rem;
            background: linear-gradient(135deg, #00b894, #55efc4);
            color: white;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            transition: transform 0.2s ease, background 0.3s ease;
        }
        button:hover {
            background: linear-gradient(135deg, #00cec9, #81ecec);
            transform: scale(1.05);
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
            animation: fadeIn 1s ease-in;
        }
        canvas {
            margin-top: 2rem;
            animation: fadeIn 1s ease-in;
        }
        a {
            color: #007BFF;
        }
        @keyframes fadeIn {
            from { opacity: 0; }
            to { opacity: 1; }
        }
        @keyframes slideIn {
            from { transform: translateY(30px); opacity: 0; }
            to { transform: translateY(0); opacity: 1; }
        }
        @keyframes fadeInUp {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }
    </style>
</head>
<body>
<div class="container">
    <h1>Retirement Calculator</h1>
    <p style="text-align:center"><a href="https://github.com/MaddMeeks/Meeks.git" target="_blank">View Source on GitHub</a></p>
    <form id="retirementForm">
        <label>Current Age:<input type="number" id="age" value="30"></label>
        <label>Retirement Age:<input type="number" id="retirementAge" value="65"></label>

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
        <label>Tax Rate % (applies only to 401K):<input type="number" id="taxRate" value="25"></label>

        <label>Life Expectancy:<input type="number" id="lifeExpectancy" value="85"></label>

        <button type="button" onclick="calculate()">Calculate</button>
    </form>

    <div class="results" id="results" style="display:none">
        <h3>Results</h3>
        <p id="total"></p>
        <p id="afterTax"></p>
        <p id="adjusted"></p>
        <p id="monthly"></p>
        <canvas id="breakdownChart" width="400" height="200"></canvas>
    </div>
</div>

<script>
function futureValue(current, annual, rate, years) {
    let history = [];
    let value = current;
    for (let i = 0; i < years; i++) {
        value = (value + annual) * (1 + rate);
        history.push(value);
    }
    return [value, history];
}

function calculate() {
    const get = id => parseFloat(document.getElementById(id).value);

    const age = get("age");
    const retirementAge = get("retirementAge");
    const lifeExpectancy = get("lifeExpectancy");
    const years = retirementAge - age;

    const annualSalary = get("annualSalary");
    const totalContribution = get("totalContribution");
    const rateOfReturn401K = get("rateOfReturn401K") / 100;
    const current401K = get("current401K");
    const annual401KContrib = annualSalary * (totalContribution / 100);

    const [future401K, history401K] = futureValue(current401K, annual401KContrib, rateOfReturn401K, years);
    const [futureRoth, historyRoth] = futureValue(get("currentRoth"), get("rothAnnualContribution"), get("rateOfReturnRoth") / 100, years);
    const [futureOther, historyOther] = futureValue(get("currentOther"), get("otherAnnualContribution"), get("rateOfReturnOther") / 100, years);

    const taxRate = get("taxRate") / 100;
    const afterTax401K = future401K * (1 - taxRate);
    const total = afterTax401K + futureRoth + futureOther;

    const adjusted = total / Math.pow(1 + get("inflation") / 100, years);
    const monthly = adjusted / ((lifeExpectancy - retirementAge) * 12);

    document.getElementById("total").textContent = `Total at Retirement: $${total.toFixed(2)}`;
    document.getElementById("afterTax").textContent = `401K After Tax: $${afterTax401K.toFixed(2)}`;
    document.getElementById("adjusted").textContent = `Purchasing Power Today: $${adjusted.toFixed(2)}`;
    document.getElementById("monthly").textContent = `Monthly Allowance Estimate: $${monthly.toFixed(2)}`;

    const ctx = document.getElementById('breakdownChart').getContext('2d');
    if (window.breakdownChart) {
        window.breakdownChart.destroy();
    }
    window.breakdownChart = new Chart(ctx, {
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
</script>
</body>
</html>
