<!DOCTYPE html>

<html lang="zh-CN">

<head>

    <meta charset="UTF-8">

    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>全球时区联动对照 - 清爽版</title>

    <style>

        :root {

            --primary: #2563eb;

            --bj-color: #f59e0b;

            --bg: #f1f5f9;

            --card: #ffffff;

            --text-main: #1e293b;

            --text-muted: #64748b;

        }



        body {

            font-family: system-ui, -apple-system, sans-serif;

            background-color: var(--bg);

            margin: 0;

            padding: 20px;

            display: flex;

            flex-direction: column;

            align-items: center;

        }



        .container { width: 100%; max-width: 800px; }



        .header-controls {

            position: sticky;

            top: 0;

            background: var(--bg);

            padding: 10px 0 20px;

            z-index: 100;

        }



        .search-container { position: relative; display: flex; gap: 10px; }

        .search-bar { flex: 1; position: relative; }

        input[type="text"] {

            width: 100%; padding: 12px; border: 2px solid #ddd; border-radius: 8px; box-sizing: border-box; font-size: 16px;

        }



        .suggestions {

            position: absolute; top: 100%; left: 0; right: 0;

            background: white; border-radius: 8px; box-shadow: 0 10px 15px rgba(0,0,0,0.1);

            max-height: 400px; overflow-y: auto; z-index: 1000; display: none;

        }

        .suggestion-item { padding: 12px 15px; cursor: pointer; border-bottom: 1px solid #f1f5f9; display: flex; justify-content: space-between; }

        .suggestion-item:hover { background: #f8fafc; }



        .btn-live { background: #10b981; color: white; border: none; padding: 0 15px; border-radius: 8px; cursor: pointer; font-weight: bold; }



        .time-list { display: flex; flex-direction: column; gap: 12px; margin-top: 15px; }



        .card {

            background: var(--card);

            padding: 20px 24px;

            border-radius: 14px;

            box-shadow: 0 4px 6px -1px rgba(0,0,0,0.05);

            display: grid;

            grid-template-columns: 280px 1fr 140px;

            align-items: center;

            gap: 15px;

            border-left: 6px solid #cbd5e1;

        }



        /* 北京时间特殊样式 */

        .card.beijing { 

            border-left-color: var(--bj-color); 

            background: #fffbeb; 

            margin-bottom: 20px; 

            border-top: 1px solid #fde68a;

            border-bottom: 1px solid #fde68a;

            border-right: 1px solid #fde68a;

        }



        /* 核心名称显示区 */

        .city-box { display: flex; flex-direction: column; gap: 4px; }

        .name-line { display: flex; align-items: baseline; gap: 10px; }

        .cn-name { font-size: 1.3rem; font-weight: 800; color: var(--text-main); }

        .en-name { font-size: 0.95rem; color: var(--text-muted); font-weight: 500; }

        

        .zero-point-hint {

            display: inline-block;

            margin-top: 4px;

            padding: 2px 8px;

            background: rgba(0,0,0,0.04);

            color: #475569;

            font-size: 0.8rem;

            border-radius: 6px;

            width: fit-content;

        }



        /* 调节滑动条 */

        .slider-zone { display: flex; align-items: center; gap: 10px; }

        input[type="range"] { flex: 1; cursor: pointer; height: 8px; }



        /* 时间显示 */

        .time-display { text-align: right; }

        .time-val { font-family: 'Consolas', monospace; font-size: 1.7rem; font-weight: 800; color: var(--primary); }

        .date-val { font-size: 0.85rem; color: var(--text-muted); }



        .remove-btn { color: #ef4444; cursor: pointer; font-size: 11px; text-decoration: underline; margin-top: 5px; width: fit-content; }



        @media (max-width: 650px) {

            .card { grid-template-columns: 1fr; gap: 12px; }

            .time-display { text-align: left; border-top: 1px solid rgba(0,0,0,0.05); padding-top: 10px; display: flex; justify-content: space-between; align-items: center; }

        }

    </style>

</head>

<body>



<div class="container">

    <div class="header-controls">

        <div class="search-container">

            <div class="search-bar">

                <input type="text" id="citySearch" placeholder="搜索全球城市或时区 (如: 纽约, London, 12)..." oninput="handleSearch(this.value)">

                <div id="suggestions" class="suggestions"></div>

            </div>

            <button id="liveBtn" class="btn-live" style="opacity:0.3" onclick="toggleLive()">恢复实时</button>

        </div>

    </div>



    <div id="timeList" class="time-list"></div>

</div>



<script>

    let isLive = true;

    let baseTimestamp = Date.now();



    const cityDB = [

        { cn: '纽约', en: 'New York', id: 'America/New_York' },

        { cn: '伦敦', en: 'London', id: 'Europe/London' },

        { cn: '东京', en: 'Tokyo', id: 'Asia/Tokyo' },

        { cn: '巴黎', en: 'Paris', id: 'Europe/Paris' },

        { cn: '悉尼', en: 'Sydney', id: 'Australia/Sydney' },

        { cn: '洛杉矶', en: 'Los Angeles', id: 'America/Los_Angeles' },

        { cn: '迪拜', en: 'Dubai', id: 'Asia/Dubai' },

        { cn: '新加坡', en: 'Singapore', id: 'Asia/Singapore' },

        { cn: '曼谷', en: 'Bangkok', id: 'Asia/Bangkok' },

        { cn: '胡志明市', en: 'Ho Chi Minh', id: 'Asia/Ho_Chi_Minh' },

        { cn: '柏林', en: 'Berlin', id: 'Europe/Berlin' },

        { cn: '首尔', en: 'Seoul', id: 'Asia/Seoul' }

    ];



    let currentZones = [];



    // 初始化 UTC 列表

    function initDefault() {

        const sorted = [];

        for (let i = 12; i >= -12; i--) {

            if (i === 8) continue;

            const id = i === 0 ? 'UTC' : `Etc/GMT${i > 0 ? '-' : '+'}${Math.abs(i)}`;

            const sign = i >= 0 ? '+' : '';

            sorted.push({ cn: `UTC ${sign}${i}`, en: 'Timezone', id: id, isSystem: true });

        }

        currentZones = sorted;

    }



    function handleSearch(query) {

        const box = document.getElementById('suggestions');

        if (!query) { box.style.display = 'none'; return; }

        const res = cityDB.filter(c => c.cn.includes(query) || c.en.toLowerCase().includes(query.toLowerCase()));

        if (res.length > 0) {

            box.innerHTML = res.map(c => `

                <div class="suggestion-item" onclick="addCustom('${c.id}', '${c.cn}', '${c.en}')">

                    <span><strong>${c.cn}</strong> <small>${c.en}</small></span>

                    <span style="color:#94a3b8; font-size:12px;">${c.id}</span>

                </div>

            `).join('');

            box.style.display = 'block';

        } else {

            box.style.display = 'none';

        }

    }



    function addCustom(id, cn, en) {

        if (!currentZones.find(z => z.id === id)) {

            currentZones.unshift({ cn, en, id, isSystem: false });

        }

        document.getElementById('citySearch').value = '';

        document.getElementById('suggestions').style.display = 'none';

        render();

    }



    function removeZone(index) {

        currentZones.splice(index, 1);

        render();

    }



    function getBeijingTimeAtLocalZero(zoneId) {

        const now = new Date();

        const tStr = now.toLocaleString('en-US', { timeZone: zoneId });

        const uStr = now.toLocaleString('en-US', { timeZone: 'UTC' });

        const offset = (new Date(tStr) - new Date(uStr)) / 60000;

        const diff = 480 - offset;

        const total = (1440 + diff) % 1440;

        return `${Math.floor(total/60).toString().padStart(2,'0')}:${Math.floor(total%60).toString().padStart(2,'0')}`;

    }



    function updateAll(id, hVal) {

        isLive = false;

        document.getElementById('liveBtn').style.opacity = "1";

        const temp = new Date(baseTimestamp);

        const curH = parseInt(temp.toLocaleString("en-US", {timeZone: id, hour12: false}).split(', ')[1].split(':')[0]);

        baseTimestamp += (hVal - curH) * 3600000;

        render();

    }



    function toggleLive() {

        isLive = true; baseTimestamp = Date.now();

        document.getElementById('liveBtn').style.opacity = "0.3";

        render();

    }



    function createCardHTML(zone, displayDate, isBeijing = false, index = -1) {

        const formatter = new Intl.DateTimeFormat('en-GB', {

            timeZone: zone.id, hour: '2-digit', minute: '2-digit', second: '2-digit', hour12: false

        });

        const parts = formatter.formatToParts(displayDate);

        const h = parts.find(p => p.type === 'hour').value;

        const m = parts.find(p => p.type === 'minute').value;

        const s = parts.find(p => p.type === 'second').value;

        

        const dateStr = new Intl.DateTimeFormat('zh-CN', {

            timeZone: zone.id, month: 'short', day: 'numeric', weekday: 'short'

        }).format(displayDate);



        return `

            <div class="card ${isBeijing ? 'beijing' : ''}">

                <div class="city-box">

                    <div class="name-line">

                        <span class="cn-name">${zone.cn}</span>

                        <span class="en-name">${zone.en}</span>

                    </div>

                    <div class="zero-point-hint">0点 = 北京 <b>${getBeijingTimeAtLocalZero(zone.id)}</b></div>

                    ${!isBeijing && !zone.isSystem ? `<span class="remove-btn" onclick="removeZone(${index})">删除</span>` : ''}

                </div>

                <div class="slider-zone">

                    <input type="range" min="0" max="23" value="${parseInt(h)}" oninput="updateAll('${zone.id}', this.value)">

                    <span style="width:30px; font-weight:bold; color:var(--primary); font-size:1.1rem;">${h}h</span>

                </div>

                <div class="time-display">

                    <div class="time-val">${h}:${m}:${s}</div>

                    <div class="date-val">${dateStr}</div>

                </div>

            </div>

        `;

    }



    function render() {

        const list = document.getElementById('timeList');

        const d = new Date(isLive ? Date.now() : baseTimestamp);

        let html = createCardHTML({ cn: '北京 (基准)', en: 'Beijing', id: 'Asia/Shanghai' }, d, true);

        html += currentZones.map((z, i) => createCardHTML(z, d, false, i)).join('');

        list.innerHTML = html;

    }



    initDefault();

    setInterval(() => { if(isLive) render(); }, 1000);

    render();

    document.addEventListener('click', (e) => {

        if (!e.target.closest('.search-container')) document.getElementById('suggestions').style.display = 'none';

    });

</script>



</body>

</html>
