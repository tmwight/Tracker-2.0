<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Daily Fuel, Fitness & Weight Tracker</title>
  <!-- ZXing Barcode Library & Chart.js -->
  <script src="https://unpkg.com/@zxing/library@latest"></script>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    :root {
      --bg: #0f172a;
      --card-bg: #1e293b;
      --card-border: #334155;
      --text-main: #f8fafc;
      --text-muted: #94a3b8;
      --accent-primary: #38bdf8;
      --exercise-accent: #34d399;
      --weight-accent: #a855f7;
      --protein-color: #f43f5e;
      --carbs-color: #38bdf8;
      --fat-color: #fbbf24;
      --danger: #ef4444;
      --success: #10b981;
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
    }

    body {
      background-color: var(--bg);
      color: var(--text-main);
      min-height: 100vh;
      padding: 24px 16px;
      display: flex;
      justify-content: center;
    }

    .container {
      width: 100%;
      max-width: 720px;
      display: flex;
      flex-direction: column;
      gap: 20px;
    }

    header {
      display: flex;
      justify-content: space-between;
      align-items: center;
    }

    h1 {
      font-size: 1.4rem;
      font-weight: 700;
      color: #fff;
    }

    .date-badge {
      font-size: 0.85rem;
      color: var(--text-muted);
      background: var(--card-bg);
      padding: 6px 12px;
      border-radius: 9999px;
      border: 1px solid var(--card-border);
    }

    .card {
      background: var(--card-bg);
      border: 1px solid var(--card-border);
      border-radius: 14px;
      padding: 20px;
      box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.2);
    }

    .dashboard-header {
      display: flex;
      justify-content: space-between;
      align-items: baseline;
      margin-bottom: 12px;
    }

    .calories-highlight {
      font-size: 2.25rem;
      font-weight: 800;
    }

    .calories-target {
      font-size: 1rem;
      color: var(--text-muted);
      font-weight: normal;
    }

    .calorie-summary-row {
      display: flex;
      flex-wrap: wrap;
      gap: 10px;
      margin: 8px 0 16px 0;
      font-size: 0.85rem;
      color: var(--text-muted);
    }

    .summary-chip {
      background: #0f172a;
      padding: 4px 10px;
      border-radius: 6px;
      border: 1px solid var(--card-border);
    }

    .progress-bar-container {
      background: #334155;
      height: 12px;
      border-radius: 9999px;
      overflow: hidden;
      margin-bottom: 20px;
    }

    .progress-bar {
      height: 100%;
      background: var(--accent-primary);
      width: 0%;
      transition: width 0.3s ease, background-color 0.3s ease;
    }

    .macros-grid {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 12px;
    }

    .macro-box {
      background: #0f172a;
      padding: 12px;
      border-radius: 10px;
      border: 1px solid var(--card-border);
    }

    .macro-title {
      font-size: 0.8rem;
      font-weight: 600;
      text-transform: uppercase;
      letter-spacing: 0.5px;
      margin-bottom: 4px;
    }

    .macro-protein { color: var(--protein-color); }
    .macro-carbs { color: var(--carbs-color); }
    .macro-fat { color: var(--fat-color); }

    .macro-value {
      font-size: 1.15rem;
      font-weight: 700;
      margin-bottom: 6px;
    }

    .macro-sub {
      font-size: 0.75rem;
      color: var(--text-muted);
    }

    .macro-bar-bg {
      height: 4px;
      background: #334155;
      border-radius: 9999px;
      overflow: hidden;
      margin-top: 6px;
    }

    .macro-bar-fill {
      height: 100%;
      width: 0%;
      transition: width 0.3s ease;
    }

    .section-title {
      font-size: 1.1rem;
      font-weight: 600;
      margin-bottom: 14px;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }

    .tabs {
      display: flex;
      gap: 8px;
      margin-bottom: 16px;
    }

    .tab-btn {
      flex: 1;
      background: #0f172a;
      color: var(--text-muted);
      border: 1px solid var(--card-border);
      padding: 10px 8px;
      border-radius: 8px;
      font-weight: 600;
      font-size: 0.85rem;
      cursor: pointer;
      transition: all 0.2s;
      text-align: center;
    }

    .tab-btn.active {
      background: var(--card-border);
      color: #fff;
      border-color: var(--accent-primary);
    }

    .search-row {
      display: flex;
      gap: 8px;
      margin-bottom: 12px;
    }

    .search-wrapper {
      position: relative;
      flex: 1;
    }

    .search-input {
      width: 100%;
      padding: 12px 14px;
      background: #0f172a;
      border: 1px solid var(--accent-primary);
      border-radius: 10px;
      color: var(--text-main);
      font-size: 0.95rem;
      outline: none;
    }

    .search-input-exercise {
      border-color: var(--exercise-accent);
    }

    .btn-scanner {
      background: #334155;
      color: var(--text-main);
      border: 1px solid var(--accent-primary);
      padding: 0 14px;
      border-radius: 10px;
      font-weight: 600;
      font-size: 0.85rem;
      cursor: pointer;
      display: flex;
      align-items: center;
      gap: 6px;
      white-space: nowrap;
      transition: background 0.2s;
    }

    .search-results {
      position: absolute;
      top: 105%;
      left: 0;
      right: 0;
      background: #1e293b;
      border: 1px solid var(--card-border);
      border-radius: 10px;
      max-height: 250px;
      overflow-y: auto;
      z-index: 50;
      box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.4);
      display: none;
    }

    .search-result-item {
      padding: 10px 14px;
      cursor: pointer;
      display: flex;
      justify-content: space-between;
      align-items: center;
      border-bottom: 1px solid #334155;
    }

    .search-result-item:last-child {
      border-bottom: none;
    }

    .search-result-item:hover {
      background: #334155;
    }

    .divider {
      text-align: center;
      position: relative;
      margin: 16px 0;
      color: var(--text-muted);
      font-size: 0.75rem;
      text-transform: uppercase;
      letter-spacing: 1px;
    }

    .divider::before, .divider::after {
      content: "";
      position: absolute;
      top: 50%;
      width: 40%;
      height: 1px;
      background: var(--card-border);
    }

    .divider::before { left: 0; }
    .divider::after { right: 0; }

    .form-grid {
      display: grid;
      grid-template-columns: 2fr 1fr 1fr 1fr 1fr auto;
      gap: 8px;
      align-items: end;
    }

    .exercise-form-grid {
      display: grid;
      grid-template-columns: 2fr 1fr 1fr auto;
      gap: 8px;
      align-items: end;
    }

    .weight-form-grid {
      display: grid;
      grid-template-columns: 1fr 1.5fr 1fr auto;
      gap: 8px;
      align-items: end;
    }

    @media (max-width: 600px) {
      .form-grid, .exercise-form-grid, .weight-form-grid {
        grid-template-columns: 1fr 1fr;
      }
      .form-grid .btn-submit, .exercise-form-grid .btn-submit, .weight-form-grid .btn-submit {
        grid-column: span 2;
      }
    }

    .input-group {
      display: flex;
      flex-direction: column;
      gap: 4px;
    }

    .input-group label {
      font-size: 0.75rem;
      color: var(--text-muted);
      font-weight: 500;
    }

    input {
      background: #0f172a;
      border: 1px solid var(--card-border);
      color: var(--text-main);
      padding: 9px 10px;
      border-radius: 8px;
      font-size: 0.9rem;
      outline: none;
      width: 100%;
    }

    input:focus {
      border-color: var(--accent-primary);
    }

    button.btn-submit {
      background: var(--accent-primary);
      color: #0f172a;
      border: none;
      font-weight: 700;
      padding: 10px 16px;
      border-radius: 8px;
      cursor: pointer;
      transition: opacity 0.2s;
      height: 38px;
    }

    button.btn-exercise {
      background: var(--exercise-accent);
      color: #0f172a;
    }

    button.btn-weight {
      background: var(--weight-accent);
      color: #fff;
    }

    .chart-container {
      position: relative;
      height: 260px;
      width: 100%;
      margin-top: 14px;
    }

    .weight-stat-row {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 10px;
      margin-bottom: 16px;
    }

    .weight-stat-box {
      background: #0f172a;
      border: 1px solid var(--card-border);
      border-radius: 10px;
      padding: 10px;
      text-align: center;
    }

    .weight-stat-box .title {
      font-size: 0.75rem;
      color: var(--text-muted);
      margin-bottom: 4px;
    }

    .weight-stat-box .val {
      font-size: 1.15rem;
      font-weight: 700;
      color: var(--weight-accent);
    }

    .log-list {
      display: flex;
      flex-direction: column;
      gap: 8px;
    }

    .empty-state {
      text-align: center;
      color: var(--text-muted);
      padding: 24px;
      font-size: 0.9rem;
    }

    .log-item {
      display: flex;
      justify-content: space-between;
      align-items: center;
      background: #0f172a;
      border: 1px solid var(--card-border);
      padding: 12px 14px;
      border-radius: 8px;
    }

    .log-item.exercise-item {
      border-left: 3px solid var(--exercise-accent);
    }

    .log-item.weight-item {
      border-left: 3px solid var(--weight-accent);
    }

    .log-item-details h4 {
      font-size: 0.95rem;
      font-weight: 600;
      margin-bottom: 2px;
    }

    .log-item-macros {
      font-size: 0.78rem;
      color: var(--text-muted);
    }

    .log-item-right {
      display: flex;
      align-items: center;
      gap: 12px;
    }

    .log-item-cals {
      font-weight: 700;
      font-size: 1rem;
    }

    .cals-burned {
      color: var(--exercise-accent);
    }

    .btn-delete {
      background: transparent;
      color: var(--text-muted);
      border: none;
      font-size: 1.1rem;
      cursor: pointer;
      padding: 4px;
      border-radius: 4px;
      line-height: 1;
    }

    .btn-delete:hover {
      color: var(--danger);
      background: rgba(239, 68, 68, 0.1);
    }

    .btn-clear {
      background: transparent;
      color: var(--text-muted);
      border: 1px solid var(--card-border);
      padding: 4px 10px;
      border-radius: 6px;
      font-size: 0.75rem;
      cursor: pointer;
    }

    .btn-clear:hover {
      color: var(--danger);
      border-color: var(--danger);
    }
  </style>
</head>
<body>

  <div class="container">
    <header>
      <h1>Daily Fuel & Weight Tracker</h1>
      <div class="date-badge" id="currentDate">Today</div>
    </header>

    <!-- Dashboard -->
    <div class="card">
      <div class="dashboard-header">
        <div>
          <div class="calories-highlight">
            <span id="calsNet">0</span>
            <span class="calories-target">Net / <span id="calsTarget">2000</span> kcal</span>
          </div>
          <div style="font-size: 0.85rem; color: var(--text-muted); margin-top: 4px;">
            <span id="calsRemaining">2000</span> kcal remaining
          </div>
        </div>
      </div>

      <div class="calorie-summary-row">
        <span class="summary-chip">🍽️ Food: <b id="totalFoodCals">0</b> kcal</span>
        <span class="summary-chip">🔥 Burned: <b id="totalExerciseCals" style="color:var(--exercise-accent);">0</b> kcal</span>
        <span class="summary-chip">⚖️ Current: <b id="currentWeightDisplay" style="color:var(--weight-accent);">310.0</b> lbs</span>
      </div>

      <div class="progress-bar-container">
        <div class="progress-bar" id="calsBar"></div>
      </div>

      <div class="macros-grid">
        <div class="macro-box">
          <div class="macro-title macro-protein">Protein</div>
          <div class="macro-value"><span id="proteinConsumed">0</span>g</div>
          <div class="macro-sub">Target: <span id="proteinTarget">190</span>g</div>
          <div class="macro-bar-bg">
            <div class="macro-bar-fill" id="proteinBar" style="background: var(--protein-color);"></div>
          </div>
        </div>

        <div class="macro-box">
          <div class="macro-title macro-carbs">Carbs</div>
          <div class="macro-value"><span id="carbsConsumed">0</span>g</div>
          <div class="macro-sub">Target: <span id="carbsTarget">165</span>g</div>
          <div class="macro-bar-bg">
            <div class="macro-bar-fill" id="carbsBar" style="background: var(--carbs-color);"></div>
          </div>
        </div>

        <div class="macro-box">
          <div class="macro-title macro-fat">Fat</div>
          <div class="macro-value"><span id="fatConsumed">0</span>g</div>
          <div class="macro-sub">Target: <span id="fatTarget">65</span>g</div>
          <div class="macro-bar-bg">
            <div class="macro-bar-fill" id="fatBar" style="background: var(--fat-color);"></div>
          </div>
        </div>
      </div>
    </div>

    <!-- Logging Section with Tabs -->
    <div class="card">
      <div class="tabs">
        <button class="tab-btn active" id="tabFoodBtn" onclick="switchTab('food')">🍽️ Food</button>
        <button class="tab-btn" id="tabExerciseBtn" onclick="switchTab('exercise')">🏃 Exercise</button>
        <button class="tab-btn" id="tabWeightBtn" onclick="switchTab('weight')">⚖️ Weight Progress</button>
      </div>

      <!-- Food Tab Content -->
      <div id="foodTab">
        <div class="search-row">
          <div class="search-wrapper">
            <input type="text" id="foodSearchInput" class="search-input" placeholder="🔍 Search foods (Fairlife, Greek yogurt, Chicken, Rice)..." autocomplete="off">
            <div class="search-results" id="searchResults"></div>
          </div>

          <input type="file" id="barcodeFileInput" accept="image/*" capture="environment" style="display: none;" onchange="handleImageScan(event)">
          <button class="btn-scanner" onclick="document.getElementById('barcodeFileInput').click()">📷 Barcode</button>
        </div>

        <div class="divider">Or Custom Food Entry</div>

        <form id="foodForm" onsubmit="handleLogFood(event)">
          <div class="form-grid">
            <div class="input-group">
              <label for="foodName">Food Item</label>
              <input type="text" id="foodName" placeholder="e.g., Smoked Pulled Pork" required>
            </div>
            <div class="input-group">
              <label for="foodCals">Calories</label>
              <input type="number" id="foodCals" placeholder="kcal" min="0" required>
            </div>
            <div class="input-group">
              <label for="foodProtein">Protein (g)</label>
              <input type="number" id="foodProtein" placeholder="0" min="0" step="0.1" value="0">
            </div>
            <div class="input-group">
              <label for="foodCarbs">Carbs (g)</label>
              <input type="number" id="foodCarbs" placeholder="0" min="0" step="0.1" value="0">
            </div>
            <div class="input-group">
              <label for="foodFat">Fat (g)</label>
              <input type="number" id="foodFat" placeholder="0" min="0" step="0.1" value="0">
            </div>
            <button type="submit" class="btn-submit">Add</button>
          </div>
        </form>
      </div>

      <!-- Exercise Tab Content -->
      <div id="exerciseTab" style="display: none;">
        <div class="search-wrapper" style="margin-bottom: 14px;">
          <input type="text" id="exerciseSearchInput" class="search-input search-input-exercise" placeholder="🔍 Search exercise (Kayaking, Walking, Swimming, Elliptical, Cycling)..." autocomplete="off">
          <div class="search-results" id="exerciseSearchResults"></div>
        </div>

        <div class="divider">Or Custom Workout Entry</div>

        <form id="exerciseForm" onsubmit="handleLogExercise(event)">
          <div class="exercise-form-grid">
            <div class="input-group">
              <label for="exerciseName">Workout / Activity</label>
              <input type="text" id="exerciseName" placeholder="e.g., Ruck Walk / Yard Work" required>
            </div>
            <div class="input-group">
              <label for="exerciseDuration">Duration (mins)</label>
              <input type="number" id="exerciseDuration" placeholder="e.g., 30" min="1">
            </div>
            <div class="input-group">
              <label for="exerciseCals">Calories Burned</label>
              <input type="number" id="exerciseCals" placeholder="kcal" min="1" required>
            </div>
            <button type="submit" class="btn-submit btn-exercise">Add</button>
          </div>
        </form>
      </div>

      <!-- Weight & Graph Tab Content -->
      <div id="weightTab" style="display: none;">
        <div class="weight-stat-row">
          <div class="weight-stat-box">
            <div class="title">Starting Weight</div>
            <div class="val" id="statStartWeight">310.0 lbs</div>
          </div>
          <div class="weight-stat-box">
            <div class="title">Current Weight</div>
            <div class="val" id="statCurrentWeight">310.0 lbs</div>
          </div>
          <div class="weight-stat-box">
            <div class="title">Total Change</div>
            <div class="val" id="statTotalLost" style="color:var(--success);">0.0 lbs</div>
          </div>
        </div>

        <form id="weightForm" onsubmit="handleLogWeight(event)">
          <div class="weight-form-grid">
            <div class="input-group">
              <label for="weightInput">Weight (lbs)</label>
              <input type="number" id="weightInput" step="0.1" placeholder="e.g. 308.4" required>
            </div>
            <div class="input-group">
              <label for="weightDate">Date</label>
              <input type="date" id="weightDate" required>
            </div>
            <div class="input-group">
              <label for="weightNote">Note (optional)</label>
              <input type="text" id="weightNote" placeholder="e.g., Morning">
            </div>
            <button type="submit" class="btn-submit btn-weight">Log Weight</button>
          </div>
        </form>

        <div class="chart-container">
          <canvas id="weightChart"></canvas>
        </div>
      </div>
    </div>

    <!-- Daily Logged Items List -->
    <div class="card">
      <div class="section-title">
        <span>Today's Log</span>
        <button class="btn-clear" onclick="clearDayLog()">Reset Day</button>
      </div>
      <div class="log-list" id="logList"></div>
    </div>
  </div>

  <script>
    const TARGETS = {
      calories: 2000,
      protein: 190,
      carbs: 165,
      fat: 65,
      weightKg: 140
    };

    const EXERCISE_DATABASE = [
      { name: "Kayaking (Moderate effort)", met: 5.0, category: "Water Sports", defaultMins: 45 },
      { name: "Kayaking (Vigorous / Fast pace)", met: 7.0, category: "Water Sports", defaultMins: 45 },
      { name: "Stand-Up Paddleboarding (SUP)", met: 6.0, category: "Water Sports", defaultMins: 45 },
      { name: "Walking (Brisk, 3.5 mph)", met: 4.3, category: "Cardio", defaultMins: 45 },
      { name: "Walking (Casual, 2.5 mph)", met: 3.0, category: "Cardio", defaultMins: 30 },
      { name: "Walking on Incline / Rucking", met: 6.5, category: "Cardio", defaultMins: 30 },
      { name: "Weight / Resistance Training (Moderate)", met: 4.5, category: "Strength", defaultMins: 45 },
      { name: "Weight Training (Heavy / Intense)", met: 6.0, category: "Strength", defaultMins: 45 },
      { name: "Stationary Bike (Moderate effort)", met: 5.5, category: "Cardio", defaultMins: 30 },
      { name: "Stationary Bike (Vigorous effort)", met: 8.0, category: "Cardio", defaultMins: 30 },
      { name: "Outdoor Cycling (12-14 mph)", met: 8.0, category: "Cardio", defaultMins: 45 },
      { name: "Elliptical Trainer (Moderate)", met: 5.0, category: "Cardio", defaultMins: 30 },
      { name: "Rowing Machine (Moderate)", met: 6.0, category: "Cardio", defaultMins: 30 },
      { name: "Swimming (Freestyle, Moderate)", met: 6.0, category: "Water Sports", defaultMins: 30 },
      { name: "Stair Climber Machine", met: 7.5, category: "Cardio", defaultMins: 20 },
      { name: "HIIT / Circuit Training", met: 8.0, category: "Conditioning", defaultMins: 25 },
      { name: "Yard Work / Lawn Mowing", met: 5.0, category: "General Activity", defaultMins: 45 }
    ];

    const LOCAL_FOODS = [
      { name: "Chicken Breast (Cooked, 8oz)", brand: "Staple", cals: 360, protein: 70, carbs: 0, fat: 8 },
      { name: "Chicken Breast (Cooked, 6oz)", brand: "Staple", cals: 270, protein: 53, carbs: 0, fat: 6 },
      { name: "93/7 Lean Ground Beef (8oz)", brand: "Staple", cals: 340, protein: 48, carbs: 0, fat: 16 },
      { name: "Sirloin Steak (Trimmed, 8oz)", brand: "Staple", cals: 410, protein: 52, carbs: 0, fat: 20 },
      { name: "Salmon Fillet (6oz)", brand: "Staple", cals: 350, protein: 38, carbs: 0, fat: 22 },
      { name: "Eggs (Large, 3 whole)", brand: "Staple", cals: 215, protein: 18, carbs: 1, fat: 15 },
      { name: "Egg Whites (1 Cup / 240g)", brand: "Staple", cals: 125, protein: 26, carbs: 2, fat: 0 },
      { name: "Whey Protein (1 Scoop)", brand: "Generic", cals: 120, protein: 24, carbs: 3, fat: 1.5 },
      { name: "Greek Yogurt 0% (1 Cup)", brand: "Generic", cals: 130, protein: 22, carbs: 9, fat: 0 },
      { name: "White Rice (Cooked, 1 Cup)", brand: "Staple", cals: 205, protein: 4, carbs: 45, fat: 0.5 },
      { name: "Oatmeal (1/2 cup dry)", brand: "Staple", cals: 150, protein: 5, carbs: 27, fat: 3 },
      { name: "Avocado (Half / 100g)", brand: "Staple", cals: 160, protein: 2, carbs: 9, fat: 15 },
      { name: "Olive Oil (1 tbsp)", brand: "Staple", cals: 120, protein: 0, carbs: 0, fat: 14 }
    ];

    let entries = JSON.parse(localStorage.getItem('tracker_entries_combined')) || [];
    let weightHistory = JSON.parse(localStorage.getItem('tracker_weight_history')) || [
      { date: new Date().toISOString().split('T')[0], weight: 310.0, note: "Starting baseline" }
    ];

    let debounceTimer = null;
    let weightChart = null;

    function switchTab(tab) {
      document.getElementById('foodTab').style.display = tab === 'food' ? 'block' : 'none';
      document.getElementById('exerciseTab').style.display = tab === 'exercise' ? 'block' : 'none';
      document.getElementById('weightTab').style.display = tab === 'weight' ? 'block' : 'none';

      document.getElementById('tabFoodBtn').className = tab === 'food' ? 'tab-btn active' : 'tab-btn';
      document.getElementById('tabExerciseBtn').className = tab === 'exercise' ? 'tab-btn active' : 'tab-btn';
      document.getElementById('tabWeightBtn').className = tab === 'weight' ? 'tab-btn active' : 'tab-btn';

      if (tab === 'weight') {
        renderWeightChart();
      }
    }

    // --- Weight Logging & Graph Functions ---
    function handleLogWeight(e) {
      e.preventDefault();
      const weight = parseFloat(document.getElementById('weightInput').value);
      const date = document.getElementById('weightDate').value;
      const note = document.getElementById('weightNote').value.trim();

      if (!weight || !date) return;

      // Update or insert entry
      const existingIdx = weightHistory.findIndex(w => w.date === date);
      if (existingIdx >= 0) {
        weightHistory[existingIdx] = { date, weight, note };
      } else {
        weightHistory.push({ date, weight, note });
      }

      weightHistory.sort((a, b) => new Date(a.date) - new Date(b.date));
      localStorage.setItem('tracker_weight_history', JSON.stringify(weightHistory));
      
      // Also log as daily activity item
      entries.push({
        id: Date.now(),
        type: 'weight',
        name: `Weigh-in: ${weight} lbs ${note ? '(' + note + ')' : ''}`,
        weight,
        time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })
      });
      saveEntries();

      document.getElementById('weightInput').value = '';
      renderWeightChart();
      render();
    }

    function renderWeightChart() {
      if (weightHistory.length === 0) return;

      const labels = weightHistory.map(w => w.date);
      const dataPoints = weightHistory.map(w => w.weight);

      const startWeight = weightHistory[0].weight;
      const currentWeight = weightHistory[weightHistory.length - 1].weight;
      const diff = Math.round((currentWeight - startWeight) * 10) / 10;

      document.getElementById('statStartWeight').textContent = `${startWeight.toFixed(1)} lbs`;
      document.getElementById('statCurrentWeight').textContent = `${currentWeight.toFixed(1)} lbs`;
      document.getElementById('currentWeightDisplay').textContent = `${currentWeight.toFixed(1)}`;
      
      const lostEl = document.getElementById('statTotalLost');
      lostEl.textContent = `${diff <= 0 ? '' : '+'}${diff.toFixed(1)} lbs`;
      lostEl.style.color = diff <= 0 ? 'var(--success)' : 'var(--danger)';

      const ctx = document.getElementById('weightChart').getContext('2d');
      if (weightChart) {
        weightChart.destroy();
      }

      weightChart = new Chart(ctx, {
        type: 'line',
        data: {
          labels: labels,
          datasets: [{
            label: 'Weight (lbs)',
            data: dataPoints,
            borderColor: '#a855f7',
            backgroundColor: 'rgba(168, 85, 247, 0.15)',
            borderWidth: 3,
            pointBackgroundColor: '#a855f7',
            pointRadius: 5,
            pointHoverRadius: 7,
            tension: 0.25,
            fill: true
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          plugins: {
            legend: { display: false },
            tooltip: {
              callbacks: {
                label: (context) => ` ${context.parsed.y} lbs`
              }
            }
          },
          scales: {
            x: {
              grid: { color: '#334155' },
              ticks: { color: '#94a3b8', maxTicksLimit: 6 }
            },
            y: {
              grid: { color: '#334155' },
              ticks: { color: '#94a3b8' }
            }
          }
        }
      });
    }

    // --- Exercise Search Logic ---
    const exerciseSearchInput = document.getElementById('exerciseSearchInput');
    const exerciseSearchResults = document.getElementById('exerciseSearchResults');

    exerciseSearchInput.addEventListener('input', (e) => {
      const q = e.target.value.toLowerCase().trim();
      if (!q) {
        exerciseSearchResults.style.display = 'none';
        return;
      }

      const matches = EXERCISE_DATABASE.filter(ex => 
        ex.name.toLowerCase().includes(q) || ex.category.toLowerCase().includes(q)
      );

      if (matches.length === 0) {
        exerciseSearchResults.innerHTML = '<div style="padding:12px; color:var(--text-muted); font-size:0.85rem;">No exercises found. Add custom workout below.</div>';
      } else {
        exerciseSearchResults.innerHTML = matches.map(ex => {
          const cals = Math.round((ex.defaultMins / 60) * ex.met * TARGETS.weightKg);
          const safeName = ex.name.replace(/'/g, "\\'");
          return `
            <div class="search-result-item" onclick="selectExercise('${safeName}', ${ex.defaultMins}, ${cals})">
              <div>
                <div style="font-size:0.9rem; font-weight:500;">${ex.name} (${ex.defaultMins} min)</div>
                <div style="font-size:0.75rem; color:var(--exercise-accent);">${ex.category} · MET ${ex.met}</div>
              </div>
              <div style="font-weight:700; color:var(--exercise-accent); font-size:0.95rem;">-${cals} kcal</div>
            </div>
          `;
        }).join('');
      }
      exerciseSearchResults.style.display = 'block';
    });

    function selectExercise(name, mins, cals) {
      logExercise(`${name} (${mins} min)`, cals);
      exerciseSearchInput.value = '';
      exerciseSearchResults.style.display = 'none';
    }

    // --- Food Search Logic ---
    const searchInput = document.getElementById('foodSearchInput');
    const searchResults = document.getElementById('searchResults');

    searchInput.addEventListener('input', (e) => {
      const query = e.target.value.trim();
      clearTimeout(debounceTimer);

      if (!query) {
        searchResults.style.display = 'none';
        return;
      }

      searchResults.innerHTML = '<div style="padding:12px; color:var(--text-muted); font-size:0.85rem;">Searching database...</div>';
      searchResults.style.display = 'block';

      debounceTimer = setTimeout(() => {
        searchFoodsOnline(query);
      }, 350);
    });

    async function searchFoodsOnline(query) {
      let combinedResults = [];
      const localMatches = LOCAL_FOODS.filter(f => f.name.toLowerCase().includes(query.toLowerCase()));
      combinedResults.push(...localMatches);

      try {
        const url = `https://world.openfoodfacts.org/cgi/search.pl?search_terms=${encodeURIComponent(query)}&search_simple=1&action=process&json=1&page_size=8`;
        const res = await fetch(url);
        const data = await res.json();

        if (data && data.products) {
          data.products.forEach(p => {
            const nutriments = p.nutriments || {};
            const cals = Math.round(nutriments['energy-kcal_serving'] || nutriments['energy-kcal_100g'] || (nutriments['energy_100g'] ? nutriments['energy_100g'] / 4.184 : 0));
            const protein = Math.round((nutriments.proteins_serving || nutriments.proteins_100g || 0) * 10) / 10;
            const carbs = Math.round((nutriments.carbohydrates_serving || nutriments.carbohydrates_100g || 0) * 10) / 10;
            const fat = Math.round((nutriments.fat_serving || nutriments.fat_100g || 0) * 10) / 10;
            const brand = p.brands || '';
            const serving = p.serving_size ? ` (${p.serving_size})` : '';

            if (p.product_name && cals > 0) {
              combinedResults.push({
                name: `${p.product_name}${serving}`,
                brand: brand ? brand : 'Web Result',
                cals,
                protein,
                carbs,
                fat
              });
            }
          });
        }
      } catch (err) {
        console.log('Lookup error', err);
      }

      renderSearchResults(combinedResults);
    }

    function renderSearchResults(results) {
      if (results.length === 0) {
        searchResults.innerHTML = '<div style="padding:12px; color:var(--text-muted); font-size:0.85rem;">No matches found. Use custom entry below.</div>';
        return;
      }

      searchResults.innerHTML = results.slice(0, 10).map(f => {
        const safeName = f.name.replace(/'/g, "\\'");
        return `
          <div class="search-result-item" onclick="selectSearchFood('${safeName}', ${f.cals}, ${f.protein}, ${f.carbs}, ${f.fat})">
            <div>
              <div style="font-size:0.9rem; font-weight:500;">${f.name}</div>
              <div style="font-size:0.75rem; color:var(--accent-primary);">${f.brand}</div>
              <div style="font-size:0.75rem; color:var(--text-muted);">${f.protein}g P · ${f.carbs}g C · ${f.fat}g F</div>
            </div>
            <div style="font-weight:700; color:var(--accent-primary); font-size:0.95rem;">+ ${f.cals} kcal</div>
          </div>
        `;
      }).join('');
      searchResults.style.display = 'block';
    }

    async function handleImageScan(e) {
      const file = e.target.files[0];
      if (!file) return;

      const codeReader = new ZXing.BrowserMultiFormatReader();
      const imgUrl = URL.createObjectURL(file);
      const img = new Image();

      img.onload = async () => {
        try {
          const result = await codeReader.decodeFromImageElement(img);
          if (result && result.text) {
            await lookupBarcode(result.text);
          }
        } catch (err) {
          alert('Could not detect a clear barcode. Ensure it is in focus and well-lit.');
        } finally {
          URL.revokeObjectURL(imgUrl);
          document.getElementById('barcodeFileInput').value = '';
        }
      };
      img.src = imgUrl;
    }

    async function lookupBarcode(barcode) {
      try {
        const url = `https://world.openfoodfacts.org/api/v0/product/${barcode}.json`;
        const res = await fetch(url);
        const data = await res.json();

        if (data.status === 1 && data.product) {
          const p = data.product;
          const nutriments = p.nutriments || {};
          const cals = Math.round(nutriments['energy-kcal_serving'] || nutriments['energy-kcal_100g'] || (nutriments['energy_100g'] ? nutriments['energy_100g'] / 4.184 : 0));
          const protein = Math.round((nutriments.proteins_serving || nutriments.proteins_100g || 0) * 10) / 10;
          const carbs = Math.round((nutriments.carbohydrates_serving || nutriments.carbohydrates_100g || 0) * 10) / 10;
          const fat = Math.round((nutriments.fat_serving || nutriments.fat_100g || 0) * 10) / 10;
          const name = p.product_name || `Scanned Item (${barcode})`;

          if (cals > 0) {
            logFoodItem(name, cals, protein, carbs, fat);
            alert(`Logged: ${name} (${cals} kcal, ${protein}g Protein)`);
          } else {
            alert(`Found "${name}", but calories were missing. Enter manually below.`);
            document.getElementById('foodName').value = name;
          }
        } else {
          alert(`Barcode not found. Please enter manually.`);
        }
      } catch (err) {
        alert('Network error retrieving barcode data.');
      }
    }

    document.addEventListener('click', (e) => {
      if (!searchInput.contains(e.target) && !searchResults.contains(e.target)) {
        searchResults.style.display = 'none';
      }
      if (!exerciseSearchInput.contains(e.target) && !exerciseSearchResults.contains(e.target)) {
        exerciseSearchResults.style.display = 'none';
      }
    });

    function selectSearchFood(name, cals, protein, carbs, fat) {
      logFoodItem(name, cals, protein, carbs, fat);
      searchInput.value = '';
      searchResults.style.display = 'none';
    }

    function logFoodItem(name, calories, protein, carbs, fat) {
      entries.push({
        id: Date.now(),
        type: 'food',
        name,
        calories,
        protein,
        carbs,
        fat,
        time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })
      });
      saveEntries();
    }

    function logExercise(name, caloriesBurned) {
      entries.push({
        id: Date.now(),
        type: 'exercise',
        name,
        calories: caloriesBurned,
        time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })
      });
      saveEntries();
    }

    function handleLogFood(e) {
      e.preventDefault();
      const name = document.getElementById('foodName').value.trim();
      const cals = parseFloat(document.getElementById('foodCals').value) || 0;
      const protein = parseFloat(document.getElementById('foodProtein').value) || 0;
      const carbs = parseFloat(document.getElementById('foodCarbs').value) || 0;
      const fat = parseFloat(document.getElementById('foodFat').value) || 0;

      if (!name) return;

      logFoodItem(name, cals, protein, carbs, fat);
      document.getElementById('foodForm').reset();
    }

    function handleLogExercise(e) {
      e.preventDefault();
      const name = document.getElementById('exerciseName').value.trim();
      const duration = document.getElementById('exerciseDuration').value;
      const cals = parseFloat(document.getElementById('exerciseCals').value) || 0;

      if (!name || cals <= 0) return;

      const title = duration ? `${name} (${duration} min)` : name;
      logExercise(title, cals);
      document.getElementById('exerciseForm').reset();
    }

    function deleteEntry(id) {
      entries = entries.filter(item => item.id !== id);
      saveEntries();
    }

    function clearDayLog() {
      if (confirm('Are you sure you want to clear all logged food & workouts for today?')) {
        entries = [];
        saveEntries();
      }
    }

    function saveEntries() {
      localStorage.setItem('tracker_entries_combined', JSON.stringify(entries));
      render();
    }

    function render() {
      let foodCals = 0, exerciseCals = 0;
      let totalProtein = 0, totalCarbs = 0, totalFat = 0;

      entries.forEach(e => {
        if (e.type === 'food') {
          foodCals += e.calories;
          totalProtein += e.protein;
          totalCarbs += e.carbs;
          totalFat += e.fat;
        } else if (e.type === 'exercise') {
          exerciseCals += e.calories;
        }
      });

      const netCals = foodCals - exerciseCals;
      const remainingCals = TARGETS.calories - netCals;

      document.getElementById('calsNet').textContent = Math.round(netCals);
      document.getElementById('calsTarget').textContent = TARGETS.calories;
      document.getElementById('totalFoodCals').textContent = Math.round(foodCals);
      document.getElementById('totalExerciseCals').textContent = Math.round(exerciseCals);
      
      const remainingEl = document.getElementById('calsRemaining');
      remainingEl.textContent = Math.round(remainingCals);
      remainingEl.parentElement.style.color = remainingCals < 0 ? 'var(--danger)' : 'var(--text-muted)';

      const calsPct = Math.max(0, Math.min((netCals / TARGETS.calories) * 100, 100));
      const calsBar = document.getElementById('calsBar');
      calsBar.style.width = calsPct + '%';
      calsBar.style.backgroundColor = netCals > TARGETS.calories ? 'var(--danger)' : 'var(--accent-primary)';

      document.getElementById('proteinConsumed').textContent = Math.round(totalProtein);
      document.getElementById('proteinTarget').textContent = TARGETS.protein;
      document.getElementById('proteinBar').style.width = Math.min((totalProtein / TARGETS.protein) * 100, 100) + '%';

      document.getElementById('carbsConsumed').textContent = Math.round(totalCarbs);
      document.getElementById('carbsTarget').textContent = TARGETS.carbs;
      document.getElementById('carbsBar').style.width = Math.min((totalCarbs / TARGETS.carbs) * 100, 100) + '%';

      document.getElementById('fatConsumed').textContent = Math.round(totalFat);
      document.getElementById('fatTarget').textContent = TARGETS.fat;
      document.getElementById('fatBar').style.width = Math.min((totalFat / TARGETS.fat) * 100, 100) + '%';

      const listEl = document.getElementById('logList');
      if (entries.length === 0) {
        listEl.innerHTML = '<div class="empty-state">No food or workouts logged today.</div>';
      } else {
        listEl.innerHTML = entries.slice().reverse().map(e => {
          if (e.type === 'food') {
            return `
              <div class="log-item">
                <div class="log-item-details">
                  <h4>${e.name}</h4>
                  <div class="log-item-macros">${e.protein}g P · ${e.carbs}g C · ${e.fat}g F &nbsp;|&nbsp; ${e.time}</div>
                </div>
                <div class="log-item-right">
                  <span class="log-item-cals">+${e.calories} kcal</span>
                  <button class="btn-delete" title="Delete" onclick="deleteEntry(${e.id})">✕</button>
                </div>
              </div>
            `;
          } else if (e.type === 'exercise') {
            return `
              <div class="log-item exercise-item">
                <div class="log-item-details">
                  <h4>🏃 ${e.name}</h4>
                  <div class="log-item-macros">Exercise Burn &nbsp;|&nbsp; ${e.time}</div>
                </div>
                <div class="log-item-right">
                  <span class="log-item-cals cals-burned">-${e.calories} kcal</span>
                  <button class="btn-delete" title="Delete" onclick="deleteEntry(${e.id})">✕</button>
                </div>
              </div>
            `;
          } else if (e.type === 'weight') {
            return `
              <div class="log-item weight-item">
                <div class="log-item-details">
                  <h4>⚖️ ${e.name}</h4>
                  <div class="log-item-macros">Weigh-in &nbsp;|&nbsp; ${e.time}</div>
                </div>
                <div class="log-item-right">
                  <button class="btn-delete" title="Delete" onclick="deleteEntry(${e.id})">✕</button>
                </div>
              </div>
            `;
          }
        }).join('');
      }
    }

    // Set defaults
    document.getElementById('currentDate').textContent = new Date().toLocaleDateString(undefined, { 
      weekday: 'short', 
      month: 'short', 
      day: 'numeric' 
    });
    document.getElementById('weightDate').value = new Date().toISOString().split('T')[0];
    
    render();
    renderWeightChart();
  </script>
</body>
</html>
