# Lern-Tracker-
<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Lern-Profi Tracker</title>
  <style>
    /* Absolute Farbvorgaben, damit der Dark-Mode nichts unlesbar macht */
    body {
      background-color: #f1f5f9 !important;
      color: #0f172a !important;
      font-family: system-ui, -apple-system, sans-serif;
      max-width: 500px;
      margin: 0 auto;
      padding: 15px;
    }

    .card {
      background-color: #ffffff !important;
      color: #0f172a !important;
      padding: 20px;
      border-radius: 16px;
      box-shadow: 0 4px 12px rgba(0,0,0,0.08);
      margin-bottom: 15px;
    }

    h1 { color: #4338ca !important; margin: 0 0 10px 0; font-size: 1.5rem; text-align: center; }
    h2 { color: #1e293b !important; margin: 0 0 12px 0; font-size: 1.1rem; }
    p { color: #475569 !important; margin: 0 0 12px 0; font-size: 0.9rem; }

    /* Dashboard */
    .dashboard {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 10px;
    }
    .stat-box {
      background-color: #f8fafc !important;
      border: 1px solid #e2e8f0;
      padding: 10px;
      border-radius: 10px;
      text-align: center;
    }
    .stat-val { font-size: 1.3rem; font-weight: bold; color: #4338ca !important; }
    .stat-lbl { font-size: 0.75rem; color: #64748b !important; font-weight: 600; }

    /* Eingabefelder */
    .input-group {
      display: flex;
      gap: 8px;
      margin-bottom: 12px;
    }
    input[type="text"], input[type="number"] {
      flex: 1;
      padding: 10px;
      border: 2px solid #cbd5e1;
      border-radius: 8px;
      font-size: 0.95rem;
      background-color: #ffffff !important;
      color: #0f172a !important;
    }

    /* Buttons */
    button {
      background-color: #4f46e5 !important;
      color: #ffffff !important;
      border: none;
      padding: 10px 14px;
      border-radius: 8px;
      cursor: pointer;
      font-weight: bold;
    }

    .btn-sm { padding: 6px 10px; font-size: 0.8rem; }
    .btn-success { background-color: #16a34a !important; }
    .btn-danger { background-color: #dc2626 !important; }
    .btn-accent { background-color: #d97706 !important; width: 100%; padding: 10px; }

    /* Listen */
    ul { list-style: none; padding: 0; margin: 0; }
    li {
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 10px;
      background-color: #f8fafc !important;
      border: 1px solid #e2e8f0;
      border-radius: 8px;
      margin-bottom: 6px;
      color: #0f172a !important;
    }
    li.done { text-decoration: line-through; opacity: 0.5; }

    /* Ladebalken */
    .progress-bg {
      background-color: #e2e8f0 !important;
      height: 12px;
      border-radius: 6px;
      overflow: hidden;
      margin-bottom: 10px;
    }
    .progress-fill {
      background-color: #d97706 !important;
      height: 100%;
      width: 0%;
      transition: width 0.3s;
    }
  </style>
</head>
<body>

  <div class="card">
    <h1>🚀 Lern-Profi Tracker</h1>
    <div class="dashboard">
      <div class="stat-box">
        <div class="stat-val" id="star-count">⭐ 0</div>
        <div class="stat-lbl">Sterne</div>
      </div>
      <div class="stat-box">
        <div class="stat-val" id="streak-count">🔥 0</div>
        <div class="stat-lbl">Tage-Serie</div>
      </div>
    </div>
  </div>

  <div class="card">
    <h2>📚 Aufgaben heute</h2>
    <div class="input-group">
      <input type="text" id="task-input" placeholder="z. B. 15 Min. Mathe">
      <button onclick="addTask()">+</button>
    </div>
    <ul id="task-list"></ul>
  </div>

  <div class="card">
    <h2>🎁 Belohnung</h2>
    <div class="progress-bg">
      <div class="progress-fill" id="reward-progress"></div>
    </div>

    <div class="input-group">
      <input type="text" id="reward-title" placeholder="Belohnung (z.B. Kino)">
      <input type="number" id="reward-cost" placeholder="Sterne" style="max-width: 80px;">
      <button onclick="setReward()">Setzen</button>
    </div>

    <div id="reward-display" style="text-align: center; font-weight: bold; margin-top: 10px;"></div>
  </div>

  <script>
    let state = JSON.parse(localStorage.getItem('app_state_v2')) || {
      stars: 0,
      streak: 0,
      lastActiveDate: null,
      tasks: [],
      reward: { title: "Noch kein Ziel", cost: 5 }
    };

    function saveState() {
      localStorage.setItem('app_state_v2', JSON.stringify(state));
      render();
    }

    function addTask() {
      const input = document.getElementById('task-input');
      if (!input.value.trim()) return;
      state.tasks.push({ text: input.value.trim(), done: false });
      input.value = '';
      saveState();
    }

    function completeTask(index) {
      if (state.tasks[index].done) return;
      
      const today = new Date().toDateString();
      if (state.lastActiveDate !== today) {
        state.streak += 1;
        state.lastActiveDate = today;
      }

      state.tasks[index].done = true;
      state.stars += 1;
      saveState();
    }

    function deleteTask(index) {
      state.tasks.splice(index, 1);
      saveState();
    }

    function setReward() {
      const title = document.getElementById('reward-title').value.trim();
      const cost = parseInt(document.getElementById('reward-cost').value);
      if (title && cost > 0) {
        state.reward = { title, cost };
        document.getElementById('reward-title').value = '';
        document.getElementById('reward-cost').value = '';
        saveState();
      }
    }

    function claimReward() {
      if (state.stars >= state.reward.cost) {
        state.stars -= state.reward.cost;
        alert(`🎉 Super! Du hast "${state.reward.title}" freigeschaltet!`);
        saveState();
      }
    }

    function render() {
      document.getElementById('star-count').innerText = `⭐ ${state.stars}`;
      document.getElementById('streak-count').innerText = `🔥 ${state.streak}`;

      const list = document.getElementById('task-list');
      list.innerHTML = '';
      state.tasks.forEach((task, index) => {
        const li = document.createElement('li');
        if (task.done) li.classList.add('done');
        li.innerHTML = `
          <span>${task.text}</span>
          <div>
            ${!task.done ? `<button class="btn-sm btn-success" onclick="completeTask(${index})">Erledigt</button>` : '✅'}
            <button class="btn-sm btn-danger" onclick="deleteTask(${index})">✕</button>
          </div>
        `;
        list.appendChild(li);
      });

      const rewardDisplay = document.getElementById('reward-display');
      const progressFill = document.getElementById('reward-progress');
      
      const percent = Math.min(100, Math.round((state.stars / state.reward.cost) * 100)) || 0;
      progressFill.style.width = `${percent}%`;

      if (state.stars >= state.reward.cost && state.reward.cost > 0) {
        rewardDisplay.innerHTML = `
           Ziel erreicht: <b>${state.reward.title}</b><br><br>
          <button class="btn-accent" onclick="claimReward()">Belohnung einlösen!</button>
        `;
      } else {
        rewardDisplay.innerHTML = `Ziel: <b>${state.reward.title}</b> (${state.stars}/${state.reward.cost} ⭐)`;
      }
    }

    render();
  </script>
</body>
</html>
