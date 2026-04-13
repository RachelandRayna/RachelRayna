<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>in tune | Music Portal</title>
    <style>
        :root {
            --bg: #FFFFFF;
            --purple-light: #F8F4FF;
            --purple-main: #7B2CBF;
            --gold: #D4AF37;
            --text-main: #2D2D2D;
            --text-sub: #6B7280;
            --card-shadow: 0 4px 20px rgba(123, 44, 191, 0.08);
        }

        * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
        body { font-family: -apple-system, sans-serif; background: var(--bg); margin: 0; display: flex; flex-direction: column; height: 100vh; }
        
        /* Header */
        header { padding: 40px 20px 20px; text-align: center; border-bottom: 1px solid var(--purple-light); }
        header h1 { font-size: 38px; margin: 0; color: var(--purple-main); font-weight: 800; }
        header p { margin: 0; color: var(--gold); text-transform: uppercase; font-weight: 700; font-size: 11px; letter-spacing: 3px; }

        main { flex: 1; overflow-y: auto; padding: 0 20px 100px; }
        .view { display: none; animation: fadeIn 0.3s ease; }
        .view.active { display: block; }

        /* Calendar Styling */
        .calendar-container { margin-top: 20px; }
        .cal-item { display: flex; margin-bottom: 20px; background: var(--purple-light); border-radius: 15px; overflow: hidden; }
        .cal-date { background: var(--purple-main); color: white; min-width: 70px; display: flex; flex-direction: column; align-items: center; justify-content: center; font-weight: 800; }
        .cal-date .month { font-size: 10px; text-transform: uppercase; opacity: 0.8; }
        .cal-date .day { font-size: 24px; }
        .cal-info { padding: 15px; flex: 1; }
        .cal-info h3 { margin: 0; font-size: 16px; color: var(--purple-main); }
        .cal-info p { margin: 5px 0 0; font-size: 13px; color: var(--text-sub); }

        /* Schedule Styling */
        .day-title { font-size: 22px; font-weight: 800; color: var(--purple-main); margin: 30px 0 15px; display: flex; align-items: center; }
        .day-title::after { content: ""; flex: 1; height: 2px; background: var(--purple-light); margin-left: 15px; }
        .time-label { font-size: 11px; font-weight: 800; color: var(--gold); text-transform: uppercase; margin-bottom: 8px; display: block; }
        .ensemble-card { background: var(--purple-light); border-left: 4px solid var(--gold); padding: 16px; border-radius: 12px; margin-bottom: 10px; display: flex; justify-content: space-between; align-items: center; box-shadow: var(--card-shadow); }
        .time-pill { font-size: 11px; font-weight: 700; background: white; padding: 4px 10px; border-radius: 20px; color: var(--purple-main); border: 1px solid var(--purple-light); }
        .empty-state { padding: 5px; color: var(--text-sub); font-style: italic; font-size: 12px; }

        /* Nav */
        .nav-bar { position: fixed; bottom: 0; left: 0; right: 0; height: 85px; background: white; border-top: 1px solid var(--purple-light); display: flex; justify-content: space-around; padding-top: 10px; }
        .nav-btn { background: none; border: none; color: var(--text-sub); font-size: 12px; display: flex; flex-direction: column; align-items: center; cursor: pointer; }
        .nav-btn.active { color: var(--purple-main); font-weight: 700; }

        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
    </style>
</head>
<body>

<header>
    <h1>in tune</h1>
    <p>Central Music Portal</p>
</header>

<main>
    <div id="sched-view" class="view active">
        <div class="day-section">
            <div class="day-title" id="today-name">Today</div>
            <div id="today-list"></div>
        </div>
        <div class="day-section">
            <div class="day-title" id="tomorrow-name">Tomorrow</div>
            <div id="tomorrow-list"></div>
        </div>
    </div>

    <div id="event-view" class="view">
        <div class="day-title">Monthly Calendar</div>
        <div class="calendar-container" id="calendar-list">
            <div class="cal-item">
                <div class="cal-date"><span class="month">Apr</span><span class="day">18</span></div>
                <div class="cal-info"><h3>Symphony Night</h3><p>Auditorium @ 7:00 PM</p></div>
             </div>
        </div>
    </div>
</main>

<nav class="nav-bar">
    <button class="nav-btn active" onclick="switchTab('sched-view', this)">Schedule</button>
    <button class="nav-btn" onclick="switchTab('event-view', this)">Events</button>
</nav>

<script>
    // PASTE YOUR GOOGLE SHEET CSV LINK HERE
    const SHEET_URL = "REPLACE_THIS_WITH_YOUR_PUBLISHED_CSV_LINK";

    async function fetchSchedule() {
        if (SHEET_URL === "REPLACE_THIS_WITH_YOUR_PUBLISHED_CSV_LINK") {
            console.log("Using built-in schedule. Connect Google Sheets for auto-updates!");
            renderSchedule(defaultSchedule);
            return;
        }

        try {
            const response = await fetch(SHEET_URL);
            const data = await response.text();
            const rows = data.split('\n').slice(1); // Remove header
            
            const dynamicSchedule = { 0:[], 1:[], 2:[], 3:[], 4:[], 5:[], 6:[] };
            
            rows.forEach(row => {
                const [day, cat, name, time, room] = row.split(',');
                if(day) dynamicSchedule[day.trim()].push({ cat: cat.trim(), name: name.trim(), time: time.trim(), room: room.trim() });
            });
            renderSchedule(dynamicSchedule);
        } catch (e) {
            console.error("Error loading sheet:", e);
        }
    }

    const defaultSchedule = {
        1: [{cat:'before', name:"Symphony Orchestra", time:"7:00-8:00 AM", room:"Music Room"}],
        2: [{cat:'before', name:"Junior Orchestra", time:"7:00-8:00 AM", room:"Music Room"}, {cat:'before', name:"Senior Band", time:"7-8 AM", room:"Stage"}, {cat:'after', name:"Jazz Band", time:"3:30 PM", room:"Room 101"}],
        3: [{cat:'before', name:"Senior Chamber Orchestra", time:"7:00-8:00 AM", room:"Music Room"}],
        4: [{cat:'before', name:"Intermediate Orchestra", time:"7:00-8:00 AM", room:"Music Room"}, {cat:'before', name:"Junior Band", time:"7-8 AM", room:"Stage"}, {cat:'lunch', name:"Junior Chamber", time:"Lunch", room:"101"}],
        5: [{cat:'before', name:"Senior Band", time:"7:00-8:00 AM", room:"Stage"}],
        0:[], 6:[]
    };

    function renderDay(data, containerId) {
        const container = document.getElementById(containerId);
        if (!data || data.length === 0) {
            container.innerHTML = '<div class="empty-state">No rehearsals scheduled</div>';
            return;
        }
        
        container.innerHTML = ['before', 'lunch', 'after'].map(cat => {
            const items = data.filter(i => i.cat === cat);
            if (items.length === 0) return '';
            return `<span class="time-label">${cat}</span>` + items.map(item => `
                <div class="ensemble-card">
                    <div><h3>${item.name}</h3><p>${item.room}</p></div>
                    <span class="time-pill">${item.time}</span>
                </div>
            `).join('');
        }).join('');
    }

    function renderSchedule(sched) {
        const today = new Date().getDay();
        const tomorrow = (today + 1) % 7;
        const dayNames = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];
        
        document.getElementById('today-name').innerText = `Today (${dayNames[today]})`;
        document.getElementById('tomorrow-name').innerText = `Tomorrow (${dayNames[tomorrow]})`;

        renderDay(sched[today], 'today-list');
        renderDay(sched[tomorrow], 'tomorrow-list');
    }

    function switchTab(id, btn) {
        document.querySelectorAll('.view').forEach(v => v.classList.remove('active'));
        document.getElementById(id).classList.add('active');
        document.querySelectorAll('.nav-btn').forEach(b => b.classList.remove('active'));
        btn.classList.add('active');
    }

    fetchSchedule();
</script>
</body>
</html>
