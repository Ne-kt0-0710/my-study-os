<!doctype html>
<html lang="ru">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>StudyOS Pro Ultimate</title>
    <script src="https://cdn.jsdelivr.net"></script>
    <style>
        :root { --bg:#0b0c10; --card:#11131a; --muted:#a9b0c3; --text:#eef1ff; --accent:#6aa6ff; --danger:#ff5c7a; --ok:#39d98a; --radius:16px; }
        body.light { --bg:#f0f2f5; --card:#ffffff; --text:#1a1a1a; --muted:#666; --accent:#2563eb; }
        
        body { margin:0; font-family: system-ui, sans-serif; background: var(--bg); color: var(--text); transition: 0.3s; }
        .wrap { max-width: 1200px; margin: 0 auto; padding: 20px; }
        
        header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 25px; flex-wrap: wrap; gap: 15px; }
        .timer-card { background: var(--card); padding: 10px 20px; border-radius: var(--radius); border: 1px solid rgba(128,128,128,0.1); text-align: center; min-width: 150px; }
        #timer_val { font-size: 22px; font-weight: bold; color: var(--accent); }

        .grid { display: grid; grid-template-columns: 1.2fr 0.8fr; gap: 20px; }
        @media (max-width: 900px) { .grid { grid-template-columns: 1fr; } }

        .card { background: var(--card); border-radius: var(--radius); border: 1px solid rgba(128,128,128,0.1); padding: 20px; margin-bottom: 20px; box-shadow: 0 8px 32px rgba(0,0,0,0.2); }
        h2 { margin-top: 0; font-size: 14px; text-transform: uppercase; letter-spacing: 1px; color: var(--accent); }

        .tabs { display: flex; gap: 5px; margin-bottom: 15px; }
        .tabs button { flex: 1; padding: 8px; font-size: 12px; cursor: pointer; border-radius: 8px; border: 1px solid rgba(128,128,128,0.2); background: rgba(128,128,128,0.05); color: inherit; }
        .tabs button.active { background: var(--accent); color: white; border-color: var(--accent); }

        .day-row { background: rgba(128,128,128,0.05); padding: 15px; border-radius: 12px; margin-bottom: 10px; position: relative; }
        .day-row b { color: var(--accent); display: block; margin-bottom: 5px; }
        .del-btn { position: absolute; right: 10px; top: 10px; background: none; border: none; color: var(--danger); cursor: pointer; }

        .inline { display: flex; gap: 10px; margin-bottom: 15px; }
        input, select, button { padding: 10px; border-radius: 10px; border: 1px solid rgba(128,128,128,0.2); background: rgba(128,128,128,0.05); color: inherit; outline: none; }
        button { cursor: pointer; transition: 0.2s; }
        button:hover { opacity: 0.8; }

        .todo-item { display: flex; align-items: center; gap: 10px; padding: 8px 0; border-bottom: 1px solid rgba(128,128,128,0.1); }
        .done { text-decoration: line-through; opacity: 0.5; }
        
        .status-bad { border: 1px solid var(--danger) !important; background: rgba(255,92,122,0.05) !important; }
        .badge { padding: 4px 8px; border-radius: 6px; font-size: 11px; background: rgba(128,128,128,0.1); }
    </style>
</head>
<body>
    <audio id="bell" src="https://assets.mixkit.co"></audio>

    <div class="wrap">
        <header>
            <div>
                <h1 style="margin:0">StudyOS Pro <span onclick="toggleTheme()" style="cursor:pointer" id="t-icon">🌙</span></h1>
                <div id="date-now" class="muted" style="margin-top:5px"></div>
            </div>
            <div class="timer-card">
                <div id="t-status" style="font-size:10px; text-transform:uppercase">До урока</div>
                <div id="timer_val">00:00</div>
            </div>
        </header>

        <div class="grid">
            <div class="col">
                <!-- РАСПИСАНИЕ -->
                <div class="card">
                    <h2>📅 Конструктор расписания</h2>
                    <div class="inline">
                        <select id="day-sel"><option>Пн</option><option>Вт</option><option>Ср</option><option>Чт</option><option>Пт</option></select>
                        <input type="text" id="les-input" style="flex:2" placeholder="Название предмета...">
                        <button onclick="addLesson()" style="background:var(--accent); color:white">➕</button>
                    </div>
                    <div id="sched-cont"></div>
                </div>

                <!-- ДНЕВНИК -->
                <div class="card">
                    <h2>📔 Дневник оценок</h2>
                    <div class="tabs" id="term-tabs">
                        <button onclick="setTerm(1)" class="active">1 Четв.</button>
                        <button onclick="setTerm(2)">2 Четв.</button>
                        <button onclick="setTerm(3)">3 Четв.</button>
                        <button onclick="setTerm(4)">4 Четв.</button>
                    </div>
                    <div class="inline">
                        <input type="text" id="sub-input" placeholder="Предмет">
                        <select id="grd-input"><option>5</option><option>4</option><option>3</option><option>2</option></select>
                        <button onclick="addGrade()">Записать</button>
                    </div>
                    <div id="grade-list"></div>
                </div>
            </div>

            <div class="col">
                <!-- АНАЛИТИКА -->
                <div class="card" id="ana-card">
                    <h2>📈 Аналитика и Прогноз</h2>
                    <div id="advice" style="font-size:12px; margin-bottom:10px; font-weight:bold"></div>
                    <canvas id="myChart" height="200"></canvas>
                </div>

                <!-- ДОМАШКА -->
                <div class="card">
                    <h2>✅ Список дел</h2>
                    <div class="inline">
                        <input type="text" id="todo-input" style="flex:1" placeholder="Что нужно сделать?">
                        <button onclick="addTodo()">➕</button>
                    </div>
                    <div id="todo-cont"></div>
                </div>
            </div>
        </div>
    </div>

    <script>
        let term = 1;
        let grades = JSON.parse(localStorage.getItem('st_grades')) || [];
        let schedule = JSON.parse(localStorage.getItem('st_sched')) || {"Пн":[],"Вт":[],"Ср":[],"Чт":[],"Пт":[]};
        let todos = JSON.parse(localStorage.getItem('st_todo')) || [];
        let chart;

        // Тайминг уроков (в минутах от 00:00)
        const lessonStarts = [480, 575, 670, 775, 870, 965]; // 8:00, 9:35...

        function updateSystem() {
            const now = new Date();
            const mins = now.getHours() * 60 + now.getMinutes();
            
            // Календарь
            document.getElementById('date-now').innerText = now.toLocaleDateString('ru-RU', {weekday:'long', day:'numeric', month:'long'});

            // Умный Таймер
            const next = lessonStarts.find(t => t > mins);
            if(next) {
                let diff = next - mins;
                document.getElementById('timer_val').innerText = `${diff} мин`;
                if(diff <= 2) {
                    document.getElementById('t-status').innerText = "🚨 СКОРО УРОК";
                    if(now.getSeconds() === 0) document.getElementById('bell').play();
                } else {
                    document.getElementById('t-status').innerText = "До урока осталось";
                }
            }
        }

        function setTerm(t) {
            term = t;
            const btns = document.querySelectorAll('#term-tabs button');
            btns.forEach((b, i) => b.className = (i + 1 === t) ? 'active' : '');
            renderAll();
        }

        function addGrade() {
            const name = document.getElementById('sub-input').value;
            const val = parseInt(document.getElementById('grd-input').value);
            if(name) {
                grades.push({name, val, term});
                document.getElementById('sub-input').value = '';
                renderAll();
            }
        }

        function addLesson() {
            const day = document.getElementById('day-sel').value;
            const val = document.getElementById('les-input').value;
            if(val) {
                schedule[day].push(val);
                document.getElementById('les-input').value = '';
                renderAll();
            }
        }

        function delLes(day, i) { schedule[day].splice(i, 1); renderAll(); }

        function addTodo() {
            const val = document.getElementById('todo-input').value;
            if(val) { todos.push({text: val, done: false}); document.getElementById('todo-input').value=''; renderAll(); }
        }

        function toggleTodo(i) { todos[i].done = !todos[i].done; renderAll(); }

        function toggleTheme() {
            document.body.classList.toggle('light');
            document.getElementById('t-icon').innerText = document.body.classList.contains('light') ? '☀️' : '🌙';
        }

        function renderAll() {
            // Расписание
            const sCont = document.getElementById('sched-cont');
            sCont.innerHTML = Object.entries(schedule).map(([day, list]) => `
                <div class="day-row">
                    <b>${day}</b> ${list.map((l, i) => `<span>${l} <button onclick="delLes('${day}',${i})" style="color:red; border:none; background:none; cursor:pointer">×</button></span>`).join(' • ') || 'Нет уроков'}
                </div>
            `).join('');

            // Дневник
            const gList = document.getElementById('grade-list');
            const filtered = grades.filter(g => g.term === term);
            gList.innerHTML = filtered.map(g => `<div class="todo-item"><span>${g.name}</span><b style="margin-left:auto">${g.val}</b></div>`).join('');

            // Домашка
            document.getElementById('todo-cont').innerHTML = todos.map((t, i) => `
                <div class="todo-item ${t.done?'done':''}">
                    <input type="checkbox" ${t.done?'checked':''} onclick="toggleTodo(${i})"> ${t.text}
                </div>
            `).join('');

            updateChart(filtered);
            localStorage.setItem('st_grades', JSON.stringify(grades));
            localStorage.setItem('st_sched', JSON.stringify(schedule));
            localStorage.setItem('st_todo', JSON.stringify(todos));
        }

        function updateChart(data) {
            const stats = {};
            data.forEach(g => {
                if(!stats[g.name]) stats[g.name] = {s:0, c:0};
                stats[g.name].s += g.val; stats[g.name].c++;
            });

            const labels = Object.keys(stats);
            const avgs = labels.map(l => (stats[l].s / stats[l].c).toFixed(2));
            const globalAvg = avgs.length ? (avgs.reduce((a,b)=>a+parseFloat(b),0)/avgs.length) : 0;

            // Калькулятор
            const advice = document.getElementById('advice');
            if(globalAvg > 0 && globalAvg < 4.5) {
                let need = Math.ceil((4.5 * (avgs.length + 3) - (globalAvg * avgs.length)) / 5);
                advice.innerText = `💡 Чтобы выйти на "5", нужно еще примерно ${need>0?need:1} пятерок.`;
            } else { advice.innerText = "✨ Твои баллы в порядке!"; }

            // Индикация тревоги
            document.getElementById('ana-card').className = (globalAvg > 0 && globalAvg < 3.5) ? 'card status-bad' : 'card';

            const ctx = document.getElementById('myChart').getContext('2d');
            if(chart) chart.destroy();
            chart = new Chart(ctx, {
                type: 'bar',
                data: { labels, datasets: [{label: 'Средний балл', data: avgs, backgroundColor: '#6aa6ff'}]},
                options: { scales: { y: { beginAtZero:true, max:5 } } }
            });
        }

        setInterval(updateSystem, 1000);
        renderAll();
        updateSystem();
    </script>
</body>
</html>
