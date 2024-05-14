It looks like the issue might be with the initial call to `selectScenario('excellent')`. This call sets the scenario vector to `[1, 0, 0, 0, 0]` for the "excellent" scenario before the `selectScenario` function is fully defined, which could lead to unexpected behavior. Let's remove that initial call and instead set the default scenario vector in the `selectScenario` function itself. Here's the updated code:

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mortality Risk Calculator</title>
</head>

<body>
    <div>
        <label for="scenario-dropdown">Select Scenario:</label>
        <select id="scenario-dropdown">
            <option value="excellent">Excellent</option>
            <option value="verygood">Very Good</option>
            <option value="good">Good</option>
            <option value="fair">Fair</option>
            <option value="poor">Poor</option>
        </select>
        <button id="calculate-risk-button">Calculate Mortality Risk</button>
    </div>
    <!-- <div id="mortality-risk-results"></div> ; the interactive graphics do it all :) -->
    <div style="width: 80vw; margin: 0 auto;">
        <canvas id="mortality-risk-graph" width="400" height="400"></canvas>
    </div>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="assets/js/script.js"></script>
</body>

</html>
```

```javascript
'use strict';

var scenarioVector = [0, 0, 1, 0, 0]; // Default to 'good' scenario

function selectScenario(scenario) {
  switch (scenario) {
    case 'excellent':
      scenarioVector = [1, 0, 0, 0, 0];
      break;
    case 'verygood':
      scenarioVector = [0, 1, 0, 0, 0];
      break;
    case 'good':
      scenarioVector = [0, 0, 1, 0, 0];
      break;
    case 'fair':
      scenarioVector = [0, 0, 0, 1, 0];
      break;
    case 'poor':
      scenarioVector = [0, 0, 0, 0, 1];
      break;
    default:
      scenarioVector = [0, 0, 1, 0, 0]; // Set default to 'good'
  }
  calculateMortalityRisk(scenario); // Pass selected scenario to calculateMortalityRisk
}

function calculateMortalityRisk(scenario) {
  const beta = [0, .29266961, .63127178, 1.0919233, 2.010895]; // Beta coefficients for excellent, very good, good, fair, poor
  const s0 = [.9999999, .96281503, .91558171, .87179276, .82403985]; // Survival probabilities at timepoints 0, 5, 10, 15, 20
  const timePoints = [0, 5, 10, 15, 20];
  const logHR = beta.reduce((acc, curr, index) => acc + (curr * scenarioVector[index]), 0);
  const f0 = s0.map(s => (1 - s) * 100);
  const f1 = f0.map((f, index) => f * Math.exp(logHR));

  // Color schemes for different scenarios
  const colorSchemes = {
    'excellent': 'rgba(0, 191, 255, 1)',    // Bright blue
    'verygood': 'rgba(255, 0, 255, 1)',      // Magenta
    'good': 'rgba(106, 168, 79, 1)',         // Cabbage green
    'fair': 'rgba(255, 255, 0, 1)',          // Lemon yellow
    'poor': 'rgba(128, 0, 128, 1)'           // Purple
  };

  const riskResults = timePoints.map((time, index) => `Risk at ${time} years: ${f1[index].toFixed(2)}%`);

  // Draw graph
  const ctx = document.getElementById('mortality-risk-graph').getContext('2d');
  const chart = new Chart(ctx, {
    type: 'line',
    data: {
      labels: timePoints.map(String),
      datasets: [{
        label: 'Mortality Risk',
        data: f1,
        steppedLine: 'before',
        borderColor: colorSchemes[scenario],
        backgroundColor: colorSchemes[scenario].replace('1)', '0.2)'),
        borderWidth: 3
      }]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      scales: {
        x: {


          title: {
            display: true,
            text: 'Timepoints (years)'
          }
        },
        y: {
          title: {
            display: true,
            text: 'Mortality Risk (%)'
          },
          suggestedMin: 0,
          suggestedMax: 80,
          stepSize: 20,
          fontSize: 14
        }
      }
    }
  });

  document.getElementById("mortality-risk-results").innerText = riskResults.join('\n');
}

// Attach event listener to the dropdown to update the scenarioVector
document.getElementById("scenario-dropdown").addEventListener("change", function() {
  selectScenario(this.value);
});
```

This change should ensure that the "excellent" scenario works correctly.
