/***********************
 * Savings Coach JS
 ***********************/
var recommendedAccount;

const questions = [
  {
    question: "How do you handle unexpected expenses (like medical bills or car repairs)?",
    options: [
      { answer: "I have an emergency fund for such situations.", weight: 4 },
      { answer: "I try to cover it from my regular savings.", weight: 2 },
      { answer: "I use extra income or borrow if needed.", weight: 1 },
      { answer: "I don’t usually plan for these, so I figure it out when it happens.", weight: 0 }
    ],
    name: "q1"
  },
  {
    question: "When you receive extra income (e.g., bonuses, gifts), what do you do with it?",
    options: [
      { answer: "I save it all.", weight: 4 },
      { answer: "I save a portion and spend the rest.", weight: 2 },
      { answer: "I invest it in high-return opportunities.", weight: 1 },
      { answer: "I spend most of it and save if I remember to.", weight: 0 }
    ],
    name: "q2"
  },
  {
    question: "How do you prefer to manage your savings?",
    options: [
      { answer: "I prefer safe, low-risk options like fixed deposits and savings accounts.", weight: 4 },
      { answer: "I like to have a mix of savings and small investments.", weight: 2 },
      { answer: "I look for investment opportunities that could maximize returns.", weight: 1 },
      { answer: "I don’t have a fixed strategy—I save whenever possible.", weight: 0 }
    ],
    name: "q3"
  },
  {
    question: "How do you feel about taking financial risks?",
    options: [
      { answer: "I avoid risks and prefer stability.", weight: 4 },
      { answer: "I take calculated risks when I see a good opportunity.", weight: 2 },
      { answer: "I’m comfortable taking risks if the potential gains are high.", weight: 1 },
      { answer: "I don’t think much about risks—I go with what feels right in the moment.", weight: 0 }
    ],
    name: "q4"
  },
  {
    question: "Do you set specific savings goals?",
    options: [
      { answer: "Yes, I always plan and track my savings goals.", weight: 4 },
      { answer: "I set goals but don’t track them closely.", weight: 2 },
      { answer: "I save randomly when I have extra money.", weight: 1 },
      { answer: "I don’t really set savings goals.", weight: 0 }
    ],
    name: "q5"
  }
];

let currentQuestionIndex = 0;
let scores = {
  cautiousSaver: 0,
  balancedSaver: 0,
  riskTaker: 0,
  opportunist: 0
};

document.addEventListener("DOMContentLoaded", showQuestion);

function showQuestion() {
  const questionContainer = document.getElementById("question-container");
  const currentQuestion = questions[currentQuestionIndex];
  questionContainer.innerHTML = `
    <div class="question">
      <label>${currentQuestion.question}</label>
      ${currentQuestion.options.map(option => `
        <input type="radio" name="${currentQuestion.name}" value="${option.answer}" onchange="nextQuestion()"> ${option.answer}<br>
      `).join("")}
    </div>
  `;
}

function nextQuestion() {
  const currentQuestion = questions[currentQuestionIndex];
  const selectedOption = document.querySelector(`input[name="${currentQuestion.name}"]:checked`);
  if (!selectedOption) return;
  const selectedAnswer = selectedOption.value;
  const selectedWeight = currentQuestion.options.find(option => option.answer === selectedAnswer).weight;

  if (selectedAnswer.includes("emergency fund") || selectedAnswer.includes("safe, low-risk")) {
    scores.cautiousSaver += selectedWeight;
  } else if (selectedAnswer.includes("mix of savings") || selectedAnswer.includes("track my savings")) {
    scores.balancedSaver += selectedWeight;
  } else if (selectedAnswer.includes("calculated risks") || selectedAnswer.includes("investment opportunities")) {
    scores.riskTaker += selectedWeight;
  } else {
    scores.opportunist += selectedWeight;
  }
  currentQuestionIndex++;
  if (currentQuestionIndex < questions.length) {
    showQuestion();
  } else {
    showResults();
  }
}

function showResults() {
  document.getElementById("results").style.display = "block";
  document.getElementById("quiz-form").style.display = "none";
  let personality = determinePersonality(scores);
  document.getElementById("personality-type").innerText = personality.type;
  document.getElementById("strengths").innerText = "Strengths: " + personality.strengths;
  document.getElementById("areas-to-improve").innerText = "Areas for Improvement: " + personality.areasToImprove;
  document.getElementById("coaching-message").innerText = personality.coachingMessage;
}

function determinePersonality(scores) {
  let highestCategory = Object.keys(scores).reduce((a, b) => scores[a] > scores[b] ? a : b);
  return {
    cautiousSaver: {
      type: "The Cautious Saver",
      strengths: "Great at saving and avoiding unnecessary risks.",
      areasToImprove: "Consider exploring low-risk investments.",
      coachingMessage: "Set a realistic savings goal and track progress."
    },
    balancedSaver: {
      type: "The Balanced Saver",
      strengths: "Good balance between saving and investing.",
      areasToImprove: "Make savings strategy more specific and proactive.",
      coachingMessage: "Improve your savings strategy and stay on track!"
    },
    riskTaker: {
      type: "The Risk-Taker",
      strengths: "Comfortable with investments and growth.",
      areasToImprove: "Manage risk with some cautious savings.",
      coachingMessage: "Diversify your savings to protect against risks."
    },
    opportunist: {
      type: "The Opportunist",
      strengths: "Good at taking advantage of opportunities.",
      areasToImprove: "Build a more structured savings plan.",
      coachingMessage: "Focus on long-term financial stability."
    }
  }[highestCategory];
}

function goToDashboard() {
  document.querySelector(".quiz-section").style.display = "none";
  document.querySelector("#dashboard").style.display = "block";
  
  let personalityType = document.getElementById("personality-type").innerText;
  let goalText = "";
  if (personalityType === "The Cautious Saver") {
    goalText = "Your Goal: Save TZS 500 in 3 months";
  } else if (personalityType === "The Balanced Saver") {
    goalText = "Your Goal: Save TZS 750 in 4 months";
  } else if (personalityType === "The Risk-Taker") {
    goalText = "Your Goal: Save TZS 1000 in 5 months";
  } else if (personalityType === "The Opportunist") {
    goalText = "Your Goal: Save TZS 1250 in 6 months";
  } else {
    goalText = "Your Goal: Save TZS 500 in 3 months";
  }
  document.getElementById("tracker-goal").innerText = goalText;
  document.getElementById("dashboard-message").innerText = "Great job! You're on the right track to achieve your financial goals!";
  
  // Determine the recommended product based on personality type
  const product = productMapping[personalityType] || productMapping["The Cautious Saver"];
  // Set the global recommendedAccount variable
  recommendedAccount = product.name;
  
  // Update the recommendation display and the Learn More link
  displayRecommendation(personalityType);
  updateLearnMoreLink();
  
  let recommendationNote = `Based on your profile as a ${personalityType}, we recommend the ${product.name} for optimal savings!`;
  document.getElementById("tracker-recommendation-note").innerText = recommendationNote;
}

function startSaving() {
  document.getElementById("tracker").style.display = "none";
  document.getElementById("recommendation").style.display = "block";
}

function openAccount() {
  alert("Redirecting to account opening page...");
}

function learnMore() {
  alert("Learn more about how this account can help maximize your savings. More details coming soon!");
}

/**********************
 * Recommendation Mapping
 **********************/
const productMapping = {
  "The Cautious Saver": {
    name: "GT Target Account",
    description: "GT Target is a high interest bearing account that encourages financial discipline through savings. Designed to help you save toward a specific target like a wedding, holiday, or higher education.",
    features: "Zero account opening balance, Mandatory Standing Order of TZS 50,000, High interest, Minimum savings period of six months, 1% bonus interest for a one-year commitment.",
    cta: "Open Account"
  },
  "The Balanced Saver": {
    name: "Savings Account",
    description: "Earn attractive interest on your daily balance while enjoying 24/7 banking and a personalized TZ Mastercard for cash withdrawals and online payments.",
    features: "24/7 Banking services, Instant Personalized MasterCard, No monthly maintenance charges, Competitive interest rate.",
    cta: "Open Account"
  },
  "The Risk-Taker": {
    name: "Fixed Deposit",
    description: "Our Fixed/Tenured Deposits offer the best long-term savings options. Deposit a specific amount at an agreed interest rate and tenure.",
    features: "Guaranteed capital and return, Tenure: 30-365 days, Option to reinvest or withdraw at maturity.",
    cta: "Open Account"
  },
  "The Opportunist": {
    name: "Jibakishie Account",
    description: "A spend-to-save account that lets you automatically save a percentage of every spend, without feeling the pinch.",
    features: "Save 1% to 5% of every spending, Earn competitive interest on accumulated funds, Easy subscription via mobile or internet banking.",
    cta: "Open Account"
  }
};

function displayRecommendation(personalityType) {
  const product = productMapping[personalityType] || productMapping["The Cautious Saver"];
  document.getElementById("product-details").innerHTML = `
    <h4>${product.name}</h4>
    <p>${product.description}</p>
    <p><strong>Features:</strong> ${product.features}</p>
  `;
  document.getElementById("open-account-button").innerText = "Start Growing Your Savings – Open an Account Today!";
}

/**********************
 * Savings Tracker JS
 **********************/
function getNudge(monthlyIncome, monthlyExpenses, savingsGoal, timeframe, percentageSave) {
  if (percentageSave > 30) {
    return "You're saving aggressively! Consider a high-yield account to maximize your returns.";
  } else if (percentageSave < 10) {
    return "It looks like saving is tough. Try tracking expenses closely and reducing unnecessary spending.";
  } else if (savingsGoal > (monthlyIncome * timeframe * 0.5) && timeframe < 6) {
    return "Your goal is ambitious! Consider cutting non-essentials or increasing your income sources.";
  } else if (percentageSave >= 15 && percentageSave <= 25) {
    return "You're on track! Sticking to this plan will get you to your goal smoothly.";
  } else if (percentageSave >= 10 && percentageSave <= 30 && timeframe > 12) {
    return "Great discipline! A longer timeline means you can invest safely for better returns.";
  } else {
    return "Keep up the good work and adjust your plan as needed!";
  }
}

document.addEventListener("DOMContentLoaded", () => {
  const calculateButton = document.getElementById("calculate-button");
  const futureBalanceElement = document.getElementById("future-balance");
  const monthlyIncomeInput = document.getElementById("monthly-income");
  const monthlyExpensesInput = document.getElementById("monthly-expenses");
  const savingsGoalInput = document.getElementById("savings-goal");
  const timeframeInput = document.getElementById("timeframe");
  const percentageSaveInput = document.getElementById("percentage-save");
  const interestRateInput = document.getElementById("interest-rate");
  const savingsChartCtx = document.getElementById("savings-chart").getContext("2d");
  const explanationElement = document.getElementById("explanation");
  const suggestionsElement = document.getElementById("suggestions");
  const chartDescriptionElement = document.getElementById("chart-description");
  const personalizedNudgeElement = document.getElementById("personalized-nudge");

  const exampleData = {
    income: 1000000,
    expenses: 500000,
    goal: 2000000,
    timeframe: 12,
    percentageSave: 20
  };

  function calculateFutureBalance(income, expenses, goal, timeframe, percentageSave) {
    const remainingIncome = income - expenses;
    const monthlySavings = remainingIncome * (percentageSave / 100);
    return monthlySavings * timeframe;
  }

  function updateChart(chart, data) {
    chart.data.labels = Array.from({ length: data.timeframe }, (_, i) => i + 1);
    chart.data.datasets[0].data = Array.from({ length: data.timeframe }, (_, i) =>
      (data.income - data.expenses) * (data.percentageSave / 100) * (i + 1)
    );
    chart.update();
  }

  function provideExplanationAndSuggestions(monthlySavings) {
    if (monthlySavings < 0) {
      explanationElement.textContent = "Negative growth detected: Your monthly expenses exceed your savings, leading to cumulative debt over time.";
      suggestionsElement.innerHTML = `
        <h3>Suggestions for Improvement:</h3>
        <ul>
          <li>Reduce Expenses: Consider areas where you can cut down on your monthly expenses.</li>
          <li>Increase Income: Look for opportunities to increase your income or the percentage of income saved.</li>
          <li>Adjust Savings Goal: You might want to adjust your savings goal or extend the timeframe to make it more achievable.</li>
        </ul>
      `;
    } else {
      explanationElement.textContent = "";
      suggestionsElement.innerHTML = "";
    }
  }

  function updateChartDescription(data) {
    chartDescriptionElement.innerHTML = `
      <h3>Chart Description:</h3>
      <p>The chart shows an estimate of how much your savings could grow over time based on your inputs. Changes in your monthly income, expenses, savings goal, timeframe, and interest rate can affect the outcome.</p>
      <p>Results do not predict future performance and do not account for economic or market factors.</p>
    `;
  }

  const savingsChart = new Chart(savingsChartCtx, {
    type: "bar",
    data: {
      labels: Array.from({ length: exampleData.timeframe }, (_, i) => i + 1),
      datasets: [
        {
          label: "Savings Growth",
          backgroundColor: "rgba(128, 128, 128, 0.2)",
          borderColor: "rgba(128, 128, 128, 1)",
          borderWidth: 1,
          data: Array.from({ length: exampleData.timeframe }, (_, i) =>
            (exampleData.income - exampleData.expenses) * (exampleData.percentageSave / 100) * (i + 1)
          )
        }
      ]
    },
    options: {
      scales: {
        y: {
          beginAtZero: true
        }
      }
    }
  });

  calculateButton.addEventListener("click", (e) => {
    e.preventDefault();
    const monthlyIncome = parseFloat(monthlyIncomeInput.value);
    const monthlyExpenses = parseFloat(monthlyExpensesInput.value);
    const savingsGoal = parseFloat(savingsGoalInput.value);
    const timeframe = parseFloat(timeframeInput.value);
    const percentageSave = parseFloat(percentageSaveInput.value);
    const interestRate = parseFloat(interestRateInput.value) || 0;

    const futureBalance = calculateFutureBalance(monthlyIncome, monthlyExpenses, savingsGoal, timeframe, percentageSave);
    futureBalanceElement.textContent = `TZS ${futureBalance.toLocaleString()}`;

    updateChart(savingsChart, {
      income: monthlyIncome,
      expenses: monthlyExpenses,
      goal: savingsGoal,
      timeframe: timeframe,
      percentageSave: percentageSave
    });

    provideExplanationAndSuggestions((monthlyIncome - monthlyExpenses) * (percentageSave / 100));
    updateChartDescription({
      income: monthlyIncome,
      expenses: monthlyExpenses,
      goal: savingsGoal,
      timeframe: timeframe,
      interestRate: interestRate
    });

    const nudgeMessage = getNudge(monthlyIncome, monthlyExpenses, savingsGoal, timeframe, percentageSave);
    personalizedNudgeElement.innerText = nudgeMessage;
  });

  futureBalanceElement.textContent = `TZS ${calculateFutureBalance(exampleData.income, exampleData.expenses, exampleData.goal, exampleData.timeframe, exampleData.percentageSave).toLocaleString()}`;
  provideExplanationAndSuggestions((exampleData.income - exampleData.expenses) * (exampleData.percentageSave / 100));
  updateChartDescription({
    income: exampleData.income,
    expenses: exampleData.expenses,
    goal: exampleData.goal,
    timeframe: exampleData.timeframe,
    interestRate: 0
  });
  personalizedNudgeElement.innerText = getNudge(exampleData.income, exampleData.expenses, exampleData.goal, exampleData.timeframe, exampleData.percentageSave);
});

// Global function to update the Learn More link based on the recommended account
function updateLearnMoreLink() {
  var learnMoreLink = document.getElementById("learn-more-link");
  if (!recommendedAccount) {
    console.error("Recommended account is not set.");
    learnMoreLink.href = "#";
    return;
  }
  var normalizedAccount = recommendedAccount.trim().toLowerCase();
  console.log("Normalized Account:", normalizedAccount);

  if (normalizedAccount === "savings account") {
    learnMoreLink.href = "https://www.gtbank.co.tz/personal-banking/accounts/savings-investment-accounts/savings-account";
  } else if (normalizedAccount === "gt target account") {
    learnMoreLink.href = "https://www.gtbank.co.tz/personal-banking/accounts/savings-investment-accounts/jibakishie-account-2";
  } else if (normalizedAccount === "jibakishie account") {
    learnMoreLink.href = "https://www.gtbank.co.tz/personal-banking/accounts/savings-investment-accounts/jibakishie-account";
  } else if (normalizedAccount === "fixed deposit") {
    learnMoreLink.href = "https://www.gtbank.co.tz/business-banking/accounts/fixed-deposit";
  } else {
    learnMoreLink.href = "#";
  }

  console.log("Updated Learn More Link:", learnMoreLink.href);
}
