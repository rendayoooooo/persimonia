<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>parsimonia 〜家計簿アプリ〜</title>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Mochiy+Pop+One&family=Zen+Maru+Gothic:wght@700&display=swap" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>

    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-auth-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore-compat.js"></script>

    <style>
        :root {
            --bg-color: #121212;
            --card-bg: #1e1e1e;
            --neon-green: #00b894;
            --neon-pink: #ff7675;
            --neon-purple: #6c5ce7;
            --neon-yellow: #fdcb6e;
            --neon-blue: #00cec9;
            --ai-glow: var(--neon-blue);
        }
        body { font-family: 'Mochiy Pop One', 'Zen Maru Gothic', sans-serif; background-color: var(--bg-color); color: #e0e0e0; margin: 0; padding: 20px; transition: background 0.5s; }
        .container { max-width: 1200px; margin: 0 auto; }
        
        h1 { color: var(--neon-blue); text-align: center; font-size: 2.2em; text-shadow: 0 0 15px var(--neon-blue); margin-bottom: 25px; }
        h2, h3 { color: var(--neon-blue); border-bottom: 3px dashed var(--neon-blue); padding-bottom: 5px; margin-top: 0; }

        .tab-navigation { display: flex; justify-content: center; gap: 15px; margin-bottom: 25px; }
        .tab-btn { font-family: inherit; font-size: 1.1em; background: #252525; color: #888; border: 2px solid #444; padding: 12px 24px; border-radius: 12px; cursor: pointer; transition: 0.3s; }
        .tab-btn.active { background: var(--neon-blue); color: #121212; border-color: var(--neon-blue); box-shadow: 0 0 15px rgba(0,206,201,0.4); font-weight: bold; }
        
        .app-page { display: none; animation: fadeIn 0.4s ease-in-out forwards; }
        .app-page.active { display: block; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }

        .login-card { background: #1a1a2e; border: 2px dashed var(--neon-purple); padding: 15px; border-radius: 12px; margin-bottom: 20px; display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; gap: 10px; box-shadow: 0 0 10px rgba(108,92,231,0.3); }
        .btn-google { font-family: inherit; background: white; color: #333; border: none; padding: 10px 18px; border-radius: 8px; font-weight: bold; cursor: pointer; display: flex; align-items: center; gap: 8px; box-shadow: 0 4px 0 #bbb; }
        .btn-google:active { transform: translateY(4px); box-shadow: none; }
        .user-info { display: flex; align-items: center; gap: 10px; }
        .user-pic { width: 32px; height: 32px; border-radius: 50%; border: 2px solid var(--neon-green); }

        .status-header { display: flex; flex-wrap: wrap; gap: 15px; justify-content: space-between; background: #252525; padding: 15px; border-radius: 12px; border: 2px solid var(--neon-purple); margin-bottom: 25px; }
        .status-item { font-size: 1em; color: #fff; }
        .status-val { color: var(--neon-yellow); font-weight: bold; }
        
        .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 25px; margin-bottom: 25px; }
        @media (max-width: 900px) { .grid { grid-template-columns: 1fr; } }
        .card { background: var(--card-bg); padding: 25px; border-radius: 16px; box-shadow: 0 8px 16px rgba(0,0,0,0.5); border: 2px solid #2d2d2d; }
        
        .chart-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; }
        @media (max-width: 768px) { .chart-grid { grid-template-columns: 1fr; } }
        .chart-container { background: #1a1a1a; padding: 10px; border-radius: 12px; border: 1px solid #333; margin-top: 10px; position: relative; height: 220px; width: 100%; display: flex; justify-content: center; align-items: center; }
        canvas { max-height: 100% !important; max-width: 100% !important; }

        .ai-room { background: #151515; border: 3px solid var(--ai-glow); border-radius: 16px; padding: 20px; text-align: center; margin-bottom: 25px; box-shadow: 0 0 15px var(--ai-glow); }
        .ai-avatar { font-size: 3.5em; animation: bounce 1s infinite alternate; display: inline-block; }
        .ai-dialogue { background: #252525; border-radius: 12px; padding: 12px; margin-top: 10px; border: 1px solid #444; color: #fff; font-size: 0.95em; text-align: left; }
        
        .hp-bar-container { background: #333; height: 24px; border-radius: 12px; overflow: hidden; position: relative; border: 1px solid #555; }
        .hp-bar { background: linear-gradient(90deg, #ff7675, #d63031); height: 100%; width: 100%; transition: width 0.4s; }
        .wish-bar { background: linear-gradient(90deg, #6c5ce7, #a29bfe); height: 100%; width: 0%; transition: width 0.4s; }
        .bar-label { position: absolute; width: 100%; text-align: center; top: 2px; font-size: 0.85em; font-weight: bold; color: #fff; text-shadow: 1px 1px 2px #000; }

        .calendar { display: grid; grid-template-columns: repeat(7, 1fr); gap: 5px; background: #1a1a1a; padding: 10px; border-radius: 12px; border: 1px solid #333; margin-bottom: 20px; }
        .cal-day-head { text-align: center; font-size: 0.8em; color: var(--neon-purple); font-weight: bold; padding: 5px 0; }
        .cal-cell { background: #252525; min-height: 50px; border-radius: 6px; padding: 4px; font-size: 0.75em; display: flex; flex-direction: column; justify-content: space-between; border: 1px solid #333; }
        .cal-num { font-weight: bold; color: #888; }
        .cal-amt { color: var(--neon-pink); font-weight: bold; text-align: right; }

        .form-group { margin-bottom: 12px; }
        .form-row { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 12px; }
        label { display: block; margin-bottom: 3px; font-size: 0.85em; color: #a29bfe; }
        input, select, textarea { font-family: inherit; width: 100%; padding: 8px; border: 2px solid #444; border-radius: 8px; box-sizing: border-box; background-color: #252525; color: #fff; }
        
        table { width: 100%; border-collapse: collapse; margin-top: 15px; background: var(--card-bg); border-radius: 12px; overflow: hidden; font-size: 0.85em; }
        th, td { border: 1px solid #333; padding: 10px; text-align: left; }
        th { background-color: #2d2d2d; color: var(--neon-yellow); }
        
        .summary-box { display: grid; grid-template-columns: repeat(auto-fit, minmax(130px, 1fr)); gap: 10px; margin-bottom: 15px; }
        .stat-card { background: #252525; padding: 10px; border-radius: 12px; text-align: center; border: 2px solid #444; cursor: pointer; transition: 0.2s; }
        .stat-card:hover { transform: translateY(-2px); filter: brightness(1.2); }
        .stat-num { font-size: 1.2em; font-weight: bold; color: #fff; margin-top: 3px; }
        
        .detail-panel { display: none; background: #2d2d2d; border: 2px solid var(--neon-blue); border-radius: 12px; padding: 12px; margin-bottom: 15px; font-size: 0.85em; max-height: 200px; overflow-y: auto; }
        .detail-panel.active { display: block; }
        .detail-item { display: flex; justify-content: space-between; border-bottom: 1px dashed #444; padding: 5px 0; }
        
        .btn-action { font-family: inherit; border: none; padding: 10px; border-radius: 10px; cursor: pointer; font-weight: bold; color: white; transition: 0.2s; }
        .btn-pink { background: var(--neon-pink); box-shadow: 0 3px 0 #d63031; }
        .btn-purple { background: var(--neon-purple); box-shadow: 0 3px 0 #5641e4; }
        .btn-green { background: var(--neon-green); box-shadow: 0 3px 0 #009473; width: 100%; }
        .btn-excel { background: #e1b12c; box-shadow: 0 4px 0 #cc8e14; color: #121212; font-size: 1.1em; width: 100%; padding: 12px; border-radius: 12px; margin-top: 10px; }
        
        .badge { padding: 4px 8px; border-radius: 6px; font-size: 0.8em; font-weight: bold; }
        .badge-needed { background: #00b894; color: #fff; }
        .badge-maybe { background: #fdcb6e; color: #121212; }
        .badge-no { background: #ff7675; color: #fff; }
        .btn-delete { background: #ff7675; border: none; padding: 4px 8px; border-radius: 6px; color: #fff; cursor: pointer; font-family: inherit; margin-left: 5px; }
        .btn-edit { background: var(--neon-blue); border: none; padding: 4px 8px; border-radius: 6px; color: #121212; font-weight: bold; cursor: pointer; font-family: inherit; }

        .calendar-header-flex { display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; gap: 10px; margin-bottom: 10px; }
        .ai-stock-list { list-style: none; padding: 0; margin: 5px 0 0 0; }
        .ai-stock-item { background: #1e1e1e; padding: 8px 12px; border-left: 4px solid var(--neon-yellow); margin-bottom: 6px; border-radius: 0 6px 6px 0; font-size: 0.9em; display: flex; justify-content: space-between; align-items: center; }
        @keyframes bounce { from { transform: translateY(0); } to { transform: translateY(-8px); } }
    </style>
</head>
<body>

<div class="container">
    <h1>⚡ parsimonia 〜家計簿アプリ〜 ⚡</h1>

    <div class="tab-navigation">
        <button id="btn-tab1" class="tab-btn active" onclick="switchPage(1)">📊 メインステータス面</button>
        <button id="btn-tab2" class="tab-btn" onclick="switchPage(2)">📸 スキャン＆登録面</button>
        <button id="btn-tab3" class="tab-btn" onclick="switchPage(3)">🔑 ログ＆システム面</button>
    </div>

    <div id="page1" class="app-page active">
        <div class="login-card">
            <div id="authStatusText" style="color:var(--neon-yellow);">🔒 クラウド未接続：ログインすると個人データが自動同期されます</div>
            <div id="userZone">
                <button class="btn-google" onclick="loginWithGoogle()">
                    <img src="https://fonts.gstatic.com/s/i/productlogos/googleg/v6/web-24dp/logo_googleg_color_1x_web_24dp.png" alt="G" style="width:16px;">
                    Googleアカウントでログイン
                </button>
            </div>
        </div>

        <div class="status-header">
            <div class="status-item">PLAYER: <span class="status-val" id="playerName" style="color:#ff7675;">GUEST</span></div>
            <div class="status-item">節約LV: <span class="status-val" id="playerLevel">1</span></div>
            <div class="status-item">EXP: <span class="status-val" id="playerExp">0/100</span></div>
            <div class="status-item">現在の称号: <span class="status-val" id="playerTitle" style="color:#55efc4;">ノーマル市民</span></div>
            <div class="status-item">入力ストリーク: <span class="status-val" id="streakDays">1日連続中！🔥</span></div>
        </div>

        <div class="ai-room">
            <div class="ai-avatar" id="aiAvatar">😺</div>
            <div style="font-size:0.85em; color:#aaa; margin-top:5px;">クラウドデータベース防御シールド稼働状況</div>
            
            <div class="grid" style="margin-top: 15px; gap: 15px; margin-bottom: 0;">
                <div class="gauge-zone">
                    <label style="color:#fff; font-size:0.8em;">💖 残り予算HPメーター</label>
                    <div class="hp-bar-container">
                        <div class="hp-bar" id="hpBar"></div>
                        <div class="bar-label" id="hpBarLabel">HP: ¥50,000 / ¥50,000 (100%)</div>
                    </div>
                </div>
                <div class="gauge-zone">
                    <label style="color:#fff; font-size:0.8em;">🎁 ご褒美獲得進捗メーター</label>
                    <div class="hp-bar-container">
                        <div class="wish-bar" id="wishBar"></div>
                        <div class="bar-label" id="wishBarLabel">ご褒美達成率: 0%</div>
                    </div>
                </div>
            </div>
        </div>

        <div class="card" style="border: 2px solid var(--neon-blue); box-shadow: 0 0 15px rgba(0,206,201,0.2); margin-bottom: 25px;">
            <h3>🧠 AI自動ストック＆期限警告ロジスティクス</h3>
            <div class="ai-dialogue">
                <strong style="color:var(--neon-yellow);">⏰ ストック消費・賞味期限緊急度順リスト:</strong>
                <ul class="ai-stock-list" id="aiStockAnalyzerList">
                    <li style="color:#aaa;">データを登録すると、購入日をもとに緊急アラートが最優先で上部に並びます...</li>
                </ul>
            </div>
        </div>

        <div class="summary-box">
            <div class="stat-card" style="border-color: var(--neon-pink);" onclick="togglePanel('panel-total')">📋 選択月の総支出(税込)<div class="stat-num" id="totalAmount" style="color:var(--neon-pink);">¥0</div></div>
            <div class="stat-card" style="border-color: var(--neon-purple);" onclick="togglePanel('panel-save')">✨ 浮いた金額（残予算）<div class="stat-num" id="savedAmount" style="color:var(--neon-purple);">¥0</div></div>
            <div class="stat-card" style="border-color: var(--neon-yellow);" onclick="togglePanel('panel-waste')">🚨 節約候補（不要かも支出）<div class="stat-num" id="wasteAmount" style="color:var(--neon-yellow);">¥0</div></div>
        </div>
        
        <div id="panel-total" class="detail-panel"><strong>🚨 選択月の支出ログ内訳:</strong><div id="panel-total-list"></div></div>
        <div id="panel-save" class="detail-panel"><strong>✨ セーフプール資金残高詳細:</strong><div id="panel-save-info"></div></div>
        <div id="panel-waste" class="detail-panel"><strong>🔍 節約候補（不要かも）に設定されたお買い物内訳:</strong><div id="panel-waste-list"></div></div>

        <div class="card" style="margin-bottom: 25px;">
            <div class="calendar-header-flex">
                <h3 style="border-bottom:none; padding-bottom:0; margin:0;">📅 マンスリー支出カレンダー</h3>
                <div style="display:flex; align-items:center; gap:10px;">
                    <label style="color: var(--neon-blue); font-weight: bold; margin: 0; white-space:nowrap;">🌙 分析月選択:</label>
                    <select id="targetMonthSelect" style="width: auto; padding: 5px 10px; margin: 0;" onchange="updateApp()">
                        <option value="2026-05">2026年 5月</option>
                        <option value="2026-06" selected>2026年 6月</option>
                        <option value="2026-07">2026年 7月</option>
                    </select>
                </div>
            </div>
            <div class="calendar" id="monthlyCalendar"></div>
            <button type="button" class="btn-action btn-excel" onclick="exportToExcel()">📊 このデータをExcelシートに出力（合計自動算出）</button>
        </div>

        <div class="chart-grid">
            <div class="card">
                <h3>🛍️ お店の利用頻度</h3>
                <div class="chart-container"><canvas id="shopChart"></canvas></div>
            </div>
            <div class="card">
                <h3>🛡️ 必要度別の支出割合</h3>
                <div class="chart-container"><canvas id="importanceChart"></canvas></div>
            </div>
            <div class="card">
                <h3>📈 週別の合計支出グラフ</h3>
                <div class="chart-container"><canvas id="weeklyChart"></canvas></div>
            </div>
            <div class="card">
                <h3>📉 日別の合計支出推移</h3>
                <div class="chart-container"><canvas id="dailyChart"></canvas></div>
            </div>
        </div>
    </div>

    <div id="page2" class="app-page">
        <div class="grid">
            <div class="card">
                <h2>📸 ① レシート自動AIスキャン</h2>
                <div style="border: 2px dashed var(--neon-pink); padding: 25px; border-radius: 12px; text-align: center; background: #251818; margin-bottom: 15px;">
                    <label style="color: var(--neon-pink); cursor:pointer; font-size:1.1em;">📸 写真を選択するかカメラを起動
                        <input type="file" accept="image/*" style="display: none;" onchange="handleFileSelect(this)">
                    </label>
                    <div id="scanStatus" style="font-size:0.9em; color:var(--neon-yellow); margin-top:10px;"></div>
                    <button type="button" class="btn-action btn-purple" style="padding:8px 16px; font-size:0.9em; margin-top:10px;" onclick="startAIScan()">✨ AIスキャン開始</button>
                </div>
                <button type="button" class="btn-action btn-purple" style="width:100%; font-size:0.9em; margin-bottom:15px;" onclick="injectFixedExpenses()">⚡ 固定費（家賃・スマホ等）を自動入力</button>
                
                <form id="receiptForm">
                    <div class="form-row">
                        <div><label>日時</label><input type="date" id="date" required value="2026-06-13"></div>
                        <div>
                            <label>お店</label>
                            <input type="text" id="shop" list="shopHistoryDatalist" required placeholder="履歴から自動補完されます">
                            <datalist id="shopHistoryDatalist"></datalist>
                        </div>
                    </div>
                    <div class="form-group"><label>買ったもの</label><input type="text" id="product" required placeholder="例: 牛乳 シャンプー"></div>
                    <div class="form-row">
                        <div><label>金額(税抜)</label><input type="number" id="amountExTax" oninput="calcTax()"></div>
                        <div><label>金額(税込)</label><input type="number" id="amountInTax" required oninput="calcChange()"></div>
                    </div>
                    <div class="form-row">
                        <div><label>お預かり</label><input type="number" id="deposit" oninput="calcChange()"></div>
                        <div><label>おつり</label><input type="number" id="change" readonly></div>
                    </div>
                    <div class="form-row">
                        <div><label>カテゴリ</label><select id="category"></select></div>
                        <div><label>必要度</label><select id="importance"></select></div>
                    </div>
                    <div class="form-group"><label>ひひとことメモ</label><input type="text" id="memo"></div>
                    <button type="submit" class="btn-action btn-green">レシートデータをワールドに登録！</button>
                </form>
            </div>

            <div class="card">
                <h2>💸 ② リアルタイム集計 ＆ ご褒美設定</h2>
                <div class="form-row" style="background:#252525; padding:15px; border-radius:12px; margin-bottom:20px;">
                    <div><label>今月の予算</label><input type="number" id="monthlyBudget" value="50000" oninput="updateApp()"></div>
                    <div><label>ご褒美アイテム</label><input type="text" id="wishName" value="欲しかったアウター🧥" oninput="updateApp()"></div>
                    <div><label>アイテムの値段</label><input type="number" id="wishPrice" value="12000" oninput="updateApp()"></div>
                </div>
                <p style="font-size:0.9em; color:#aaa; line-height:1.6;">
                    予算枠や夢のご褒美アイテムを入力すると、ダッシュボード上のHPインジケーターにダイレクトに反映・計算されます。
                </p>
            </div>
        </div>
    </div>

    <div id="page3" class="app-page">
        <div class="card" style="margin-bottom:25px;">
            <h2>🔑 システムコア設定（Gemini API Key）</h2>
            <div style="display:flex; justify-content:space-between; align-items:center; gap:15px;">
                <label style="margin:0; white-space:nowrap;">Gemini API Key: </label>
                <input type="password" id="apiKeyInput" placeholder="AI Studioのキーを入力すると画像スキャンが解放されます" style="margin:0;" onchange="saveApiKey()">
            </div>
        </div>

        <div class="card" style="overflow-x: auto;">
            <h2>📊 ③ レシートログマスターデータベース（全件編集対応モード）</h2>
            <table id="receiptTable">
                <thead>
                    <tr><th>日時</th><th>店名</th><th>商品名</th><th>金額(税込)</th><th>カテゴリ</th><th>必要度</th><th>メモ</th><th>アクション</th></tr>
                </thead>
                <tbody></tbody>
            </table>
        </div>
    </div>
</div>

<script>
    // ⚙️ Firebase設定
    const firebaseConfig = {
        apiKey: "AIzaSyCXPKNZXmAceeC4IwECxy7u5EjAIzlKVmc",
        authDomain: "parsimonia-kakeiboapp.firebaseapp.com",
        projectId: "parsimonia-kakeiboapp",
        storageBucket: "parsimonia-kakeiboapp.firebasestorage.app",
        messagingSenderId: "720989097838",
        appId: "1:720989097838:web:a145ec4c5b751138cf3b7f",
        measurementId: "G-9640DXW5JW"
    };

    function getFirebase() {
        if (!firebase.apps.length) { firebase.initializeApp(firebaseConfig); }
        return firebase;
    }

    function switchPage(pageNum) {
        document.querySelectorAll('.app-page').forEach(page => page.classList.remove('active'));
        document.querySelectorAll('.tab-btn').forEach(btn => btn.classList.remove('active'));
        
        const targetPage = document.getElementById(`page${pageNum}`);
        const targetBtn = document.getElementById(`btn-tab${pageNum}`);
        if(targetPage) targetPage.classList.add('active');
        if(targetBtn) targetBtn.classList.add('active');
        updateApp();
    }

    const categories = ["食費", "外食", "飲み物", "カフェ", "日用品", "交通", "娯楽", "学校", "服・美容", "サブスク", "医療", "その他"];
    const importanceLevels = ["必要", "やや必要", "不要かも"];
    const neonColors = ["#00cec9", "#ff7675", "#fdcb6e", "#00b894", "#0984e3", "#6c5ce7", "#fd79a8", "#e84393", "#ffeaa7", "#74b9ff", "#a29bfe", "#55efc4"];

    let playerLV = 1; let playerEXP = 0; let base64Image = "";
    let dataList = []; let currentUser = null;
    let myShopChart, myImpChart, myWeeklyChart, myDailyChart;
    let global_savedMoney = 0; let global_waste = 0;

    // 安全にDOMの値を読み書きするためのヘルパー
    function getSafeValue(id, fallback = '') {
        const el = document.getElementById(id);
        return el ? el.value : fallback;
    }

    window.onload = function() {
        if(localStorage.getItem('gemini_api_key')) {
            const apiEl = document.getElementById('apiKeyInput');
            if(apiEl) apiEl.value = localStorage.getItem('gemini_api_key');
        }
        
        const catSelect = document.getElementById('category'); if(catSelect) categories.forEach(c => catSelect.add(new Option(c, c)));
        const impSelect = document.getElementById('importance'); if(impSelect) importanceLevels.forEach(i => impSelect.add(new Option(i, i)));

        buildCalendarFramework();
        
        const fb = getFirebase();
        fb.auth().getRedirectResult().then(result => { if (result && result.user) { console.log("Login Success"); } }).catch(e => console.error(e));

        fb.auth().onAuthStateChanged(user => {
            if (user) {
                currentUser = user;
                const authText = document.getElementById('authStatusText');
                const pName = document.getElementById('playerName');
                const uZone = document.getElementById('userZone');
                if(authText) authText.innerText = "🟢 クラウド同期中：オンラインの強固なデータベースに保護されています";
                if(pName) pName.innerText = user.displayName.toUpperCase();
                if(uZone) uZone.innerHTML = `<div class="user-info"><img class="user-pic" src="${user.photoURL}"> <button class="btn-action btn-pink" style="padding:5px 10px; font-size:0.8em;" onclick="logout()">ログアウト</button></div>`;
                loadCloudData();
            } else {
                currentUser = null;
                const authText = document.getElementById('authStatusText');
                const pName = document.getElementById('playerName');
                const uZone = document.getElementById('userZone');
                if(authText) authText.innerText = "🔒 ゲストモード：ログインするとクラウド保存が解放されます";
                if(pName) pName.innerText = "GUEST";
                if(uZone) uZone.innerHTML = `<button class="btn-google" onclick="loginWithGoogle()"><img src='https://fonts.gstatic.com/s/i/productlogos/googleg/v6/web-24dp/logo_googleg_color_1x_web_24dp.png' style='width:16px;'> Googleログイン</button>`;
                dataList = [
                    { id: 1, date: "2026-06-11", shop: "イオンネオモール", product: "牛乳 卵", amountExTax: 500, amountInTax: 550, category: "食費", importance: "必要", memo: "賞味期限アラート確認用サンプル" },
                    { id: 2, date: "2026-06-13", shop: "スターバックス", product: "贅沢フラペチーノ", amountExTax: 700, amountInTax: 770, category: "カフェ", importance: "不要かも", memo: "無駄遣いタップ確認用サンプル" }
                ];
                updateApp();
            }
        });
    }

    function loginWithGoogle() { try { const fb = getFirebase(); const provider = new fb.auth.GoogleAuthProvider(); fb.auth().signInWithRedirect(provider); } catch(e) { alert("認証エラー"); } }
    function logout() { getFirebase().auth().signOut(); }
    function saveApiKey() { localStorage.setItem('gemini_api_key', getSafeValue('apiKeyInput')); }

    function loadCloudData() {
        getFirebase().firestore().collection("users").doc(currentUser.uid).collection("receipts").orderBy("date", "asc")
        .onSnapshot(snapshot => {
            dataList = [];
            snapshot.forEach(doc => { dataList.push({ id: doc.id, ...doc.data() }); });
            playerEXP = dataList.length * 15;
            updateApp();
        });
    }

    function saveReceipt(newItem) {
        if(currentUser) { getFirebase().firestore().collection("users").doc(currentUser.uid).collection("receipts").add(newItem); } 
        else { dataList.push({ id: Date.now(), ...newItem }); updateApp(); }
    }

    function deleteItem(id) {
        if(currentUser && typeof id === "string" && id.length > 5) { getFirebase().firestore().collection("users").doc(currentUser.uid).collection("receipts").doc(id).delete(); } 
        else { dataList = dataList.filter(item => item.id != id); updateApp(); }
    }

    function updateItem(id, updatedFields) {
        if(currentUser && typeof id === "string" && id.length > 5) {
            getFirebase().firestore().collection("users").doc(currentUser.uid).collection("receipts").doc(id).update(updatedFields);
        } else {
            dataList = dataList.map(item => item.id == id ? { ...item, ...updatedFields } : item);
            updateApp();
        }
    }

    function buildCalendarFramework() {
        const cal = document.getElementById('monthlyCalendar'); if(!cal) return;
        cal.innerHTML = '';
        ["日","月","火","水","木","金","土"].forEach(h => cal.insertAdjacentHTML('beforeend', `<div class="cal-day-head">${h}</div>`));
        cal.insertAdjacentHTML('beforeend', `<div class="cal-cell" style="visibility:hidden;"></div>`);
        for(let day=1; day<=30; day++) { cal.insertAdjacentHTML('beforeend', `<div class="cal-cell" id="cal-day-${day}"><span class="cal-num">${day}</span><span class="cal-amt" id="cal-amt-${day}"></span></div>`); }
    }

    function injectFixedExpenses() {
        saveReceipt({ date: "2026-06-01", shop: "家賃管理組合", product: "家賃固定費", amountExTax: 40000, amountInTax: 40000, category: "その他", importance: "必要", memo: "自動枠" });
        saveReceipt({ date: "2026-06-01", shop: "サイバーモバイル", product: "スマホ通信代", amountExTax: 2980, amountInTax: 3278, category: "サブスク", importance: "必要", memo: "自動枠" });
    }

    function handleFileSelect(input) { const file = input.files[0]; if (file) { const reader = new FileReader(); reader.onload = function(e) { base64Image = e.target.result.split(',')[1]; const ss = document.getElementById('scanStatus'); if(ss) ss.innerText = "📷 スキャン準備完了"; }; reader.readAsDataURL(file); } }
    async function startAIScan() {
        const apiKey = getSafeValue('apiKeyInput'); if(!apiKey || !base64Image) { alert("キーと画像を選んでね！"); return; }
        const ss = document.getElementById('scanStatus'); if(ss) ss.innerText = "⏳ AI解析中...";
        const promptText = `JSON形式だけでレシートデータを解析して。`;
        const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${apiKey}`;
        try {
            const response = await fetch(url, { method: "POST", headers: { "Content-Type": "application/json" }, body: JSON.stringify({ contents: [{ parts: [{ text: promptText }, { inlineData: { mimeType: "image/jpeg", data: base64Image } }] }] }) });
            const result = await response.json(); const cleanJson = result.candidates[0].content.parts[0].text.replace(/```json/g, "").replace(/```/g, "").trim(); const receiptData = JSON.parse(cleanJson);
            
            const elDate = document.getElementById('date'); if(elDate) elDate.value = receiptData.date;
            const elShop = document.getElementById('shop'); if(elShop) elShop.value = receiptData.shop;
            const elProd = document.getElementById('product'); if(elProd) elProd.value = receiptData.product;
            const elInTax = document.getElementById('amountInTax'); if(elInTax) elInTax.value = receiptData.amountInTax;
            const elCat = document.getElementById('category'); if(elCat) elCat.value = receiptData.category;
            const elImp = document.getElementById('importance'); if(elImp) elImp.value = receiptData.importance;
            
            if(ss) ss.innerText = "✅ 解析完了！";
        } catch(e) { if(ss) ss.innerText = "❌ 解析エラー"; }
    }

    function calcTax() { const ex = getSafeValue('amountExTax'); if(ex) { const el = document.getElementById('amountInTax'); if(el) el.value = Math.round(ex*1.1); } }
    function calcChange() { const In = getSafeValue('amountInTax'); const dep = getSafeValue('deposit'); if(In && dep) { const el = document.getElementById('change'); if(el) el.value = dep - In >= 0 ? dep - In : 0; } }
    
    function togglePanel(id) { 
        document.querySelectorAll('.detail-panel').forEach(p => { if(p.id !== id) p.classList.remove('active'); }); 
        const p = document.getElementById(id); if(p) p.classList.toggle('active'); 
    }

    function enterEditMode(id) {
        const row = document.getElementById(`row-${id}`);
        const item = dataList.find(i => i.id == id);
        if(!row || !item) return;

        row.innerHTML = `
            <td><input type="date" id="edit-date-${id}" value="${item.date}" style="padding:4px;"></td>
            <td><input type="text" id="edit-shop-${id}" value="${item.shop}" style="padding:4px;"></td>
            <td><input type="text" id="edit-product-${id}" value="${item.product}" style="padding:4px;"></td>
            <td><input type="number" id="edit-amt-${id}" value="${item.amountInTax}" style="padding:4px; width:80px;"></td>
            <td>
                <select id="edit-cat-${id}" style="padding:4px;">
                    ${categories.map(c => `<option value="${c}" ${item.category===c?'selected':''}>${c}</option>`).join('')}
                </select>
            </td>
            <td>
                <select id="edit-imp-${id}" style="padding:4px;">
                    ${importanceLevels.map(i => `<option value="${i}" ${item.importance===i?'selected':''}>${i}</option>`).join('')}
                </select>
            </td>
            <td><input type="text" id="edit-memo-${id}" value="${item.memo||''}" style="padding:4px;"></td>
            <td>
                <button class="btn-edit" onclick="saveEditField('${id}')">保存</button>
                <button class="btn-delete" onclick="updateApp()">取消</button>
            </td>
        `;
    }

    function saveEditField(id) {
        const updated = {
            date: getSafeValue(`edit-date-${id}`),
            shop: getSafeValue(`edit-shop-${id}`),
            product: getSafeValue(`edit-product-${id}`),
            amountInTax: parseInt(getSafeValue(`edit-amt-${id}`)) || 0,
            category: getSafeValue(`edit-cat-${id}`),
            importance: getSafeValue(`edit-imp-${id}`),
            memo: getSafeValue(`edit-memo-${id}`)
        };
        updateItem(id, updated);
    }

    function updateApp() {
        const selectedMonth = getSafeValue('targetMonthSelect', '2026-06');
        
        const tbody = document.querySelector('#receiptTable tbody'); 
        const totalListDiv = document.getElementById('panel-total-list');
        const wasteListDiv = document.getElementById('panel-waste-list');
        const datalistEl = document.getElementById('shopHistoryDatalist');

        if(tbody) tbody.innerHTML = ''; 
        if(totalListDiv) totalListDiv.innerHTML = '';
        if(wasteListDiv) wasteListDiv.innerHTML = '';
        if(datalistEl) datalistEl.innerHTML = '';

        for(let d=1; d<=30; d++) { const el = document.getElementById(`cal-amt-${d}`); if(el) el.innerText = ''; }

        let total = 0; global_waste = 0; let shopCounts = {}; let impCounts = { "必要": 0, "やや必要": 0, "不要かも": 0 };
        let weeklyCounts = { "1週目":0, "2週目":0, "3週目":0, "4週目":0 };
        let dailyCounts = {}; for(let d=1; d<=30; d++) dailyCounts[d] = 0;

        let detectedStockItems = []; 
        let registeredShops = new Set(); 

        dataList.forEach(item => {
            if(item.shop) registeredShops.add(item.shop);

            if(tbody) {
                tbody.insertAdjacentHTML('beforeend', `<tr id="row-${item.id}">
                    <td>${item.date}</td><td>${item.shop}</td><td>${item.product}</td><td><strong>¥${item.amountInTax}</strong></td>
                    <td>${item.category}</td><td><span class="badge ${item.importance==='必要'?'badge-needed':item.importance==='やや必要'?'badge-maybe':'badge-no'}">${item.importance}</span></td>
                    <td>${item.memo||''}</td>
                    <td>
                        <button class="btn-edit" onclick="enterEditMode('${item.id}')">編集</button>
                        <button class="btn-delete" onclick="deleteItem('${item.id}')">削除</button>
                    </td>
                </tr>`);
            }

            if (item.date.startsWith(selectedMonth)) {
                total += item.amountInTax;

                if(totalListDiv) {
                    totalListDiv.insertAdjacentHTML('beforeend', `<div class="detail-item"><span>${item.date} [${item.shop}] - ${item.product}</span><strong>¥${item.amountInTax}</strong></div>`);
                }

                if(item.importance === "不要かも") {
                    global_waste += item.amountInTax;
                    if(wasteListDiv) {
                        wasteListDiv.insertAdjacentHTML('beforeend', `<div class="detail-item" style="color:var(--neon-yellow);"><span>⚠️ ${item.date} [${item.shop}] ${item.product}</span><strong>¥${item.amountInTax}</strong></div>`);
                    }
                }

                impCounts[item.importance] = (impCounts[item.importance] || 0) + item.amountInTax;
                shopCounts[item.shop] = (shopCounts[item.shop] || 0) + item.amountInTax;

                let dayNum = parseInt(item.date.split("-")[2]);
                if(!isNaN(dayNum) && dayNum <= 30) {
                    dailyCounts[dayNum] += item.amountInTax;
                    if(dayNum <= 7) weeklyCounts["1週目"] += item.amountInTax;
                    else if(dayNum <= 14) weeklyCounts["2週目"] += item.amountInTax;
                    else if(dayNum <= 21) weeklyCounts["3週目"] += item.amountInTax;
                    else weeklyCounts["4週目"] += item.amountInTax;
                }

                const calEl = document.getElementById(`cal-amt-${dayNum}`);
                if(calEl) calEl.innerText = `¥${dailyCounts[dayNum].toLocaleString()}`;

                const buyDate = new Date(item.date);
                const currentSimulatedTime = new Date("2026-06-13"); 
                const daysElapsed = Math.floor((currentSimulatedTime - buyDate) / (1000 * 60 * 60 * 24));

                const lowerProd = item.product.toLowerCase();
                if(lowerProd.includes("牛乳") || lowerProd.includes("ミルク")) {
                    let rem = 5 - daysElapsed;
                    detectedStockItems.push({ name: "🥛 牛乳 / ミルク", info: rem <= 0 ? "🚨 期限切れ！加熱するか廃棄を推奨！" : rem <= 2 ? `🚨 残り${rem}日：2日前を切りました！超お急ぎ消費！` : `📊 残り ${rem} 日 (目安)`, priority: rem <= 2 ? 100 : rem });
                }
                if(lowerProd.includes("卵") || lowerProd.includes("たまご")) {
                    let rem = 14 - daysElapsed;
                    detectedStockItems.push({ name: "🥚 卵パック", info: rem <= 0 ? "🚨 期限超過：しっかり加熱して！" : rem <= 2 ? `🚨 残り${rem}日：2日前です！オムレツやゆで卵に！` : `📊 残り ${rem} 日 (生食目安)`, priority: rem <= 2 ? 90 : rem });
                }
                if(lowerProd.includes("肉") || lowerProd.includes("パック")) {
                    let rem = 2 - daysElapsed;
                    detectedStockItems.push({ name: "🥩 生肉パック", info: rem <= 0 ? "🚨 期限切れ警告！" : rem <= 2 ? `🚨 残り${rem}日：2日前！すぐに冷凍するか調理して！` : `📊 残り ${rem} 日`, priority: rem <= 2 ? 95 : rem });
                }
                if(lowerProd.includes("シャンプー")) {
                    let rem = 30 - daysElapsed;
                    detectedStockItems.push({ name: "🧴 シャンプーボトル", info: rem <= 3 ? `🚨 残り約${rem}日：まもなく底をつきます！` : `📊 残り約 ${rem} 日分 (余裕あり)`, priority: rem <= 3 ? 80 : 10 });
                }
            }
        });

        if(datalistEl) registeredShops.forEach(s => { datalistEl.insertAdjacentHTML('beforeend', `<option value="${s}"></option>`); });

        if(document.getElementById('totalAmount')) document.getElementById('totalAmount').innerText = `¥${total.toLocaleString()}`;
        if(document.getElementById('wasteAmount')) document.getElementById('wasteAmount').innerText = `¥${global_waste.toLocaleString()}`;

        const budget = parseInt(getSafeValue('monthlyBudget', '50000')) || 50000;
        let hpPercent = ((budget - total) / budget) * 100; if(hpPercent < 0) hpPercent = 0;
        if(document.getElementById('hpBar')) document.getElementById('hpBar').style.width = `${hpPercent}%`;
        if(document.getElementById('hpBarLabel')) document.getElementById('hpBarLabel').innerText = `HP: ¥${(budget - total).toLocaleString()} / ¥${budget.toLocaleString()} (${hpPercent.toFixed(0)}%)`;

        global_savedMoney = budget - total >= 0 ? budget - total : 0;
        if(document.getElementById('savedAmount')) document.getElementById('savedAmount').innerText = `¥${global_savedMoney.toLocaleString()}`;
        if(document.getElementById('panel-save-info')) document.getElementById('panel-save-info').innerText = `現在のセーフ資金残高は ¥${global_savedMoney.toLocaleString()} です。`;

        const wishName = getSafeValue('wishName', '夢のご褒美アイテム'); 
        const wishPrice = parseInt(getSafeValue('wishPrice', '12000')) || 1;
        let wishPercent = (global_savedMoney / wishPrice) * 100; if(wishPercent > 100) wishPercent = 100;
        if(document.getElementById('wishBar')) document.getElementById('wishBar').style.width = `${wishPercent}%`;
        if(document.getElementById('wishBarLabel')) document.getElementById('wishBarLabel').innerText = `「${wishName}」まであと ¥${(wishPrice - global_savedMoney > 0 ? wishPrice - global_savedMoney : 0).toLocaleString()} (${wishPercent.toFixed(0)}%)`;

        playerLV = Math.floor(playerEXP / 100) + 1;
        if(document.getElementById('playerLevel')) document.getElementById('playerLevel').innerText = playerLV;
        if(document.getElementById('playerExp')) document.getElementById('playerExp').innerText = `${playerEXP % 100}/100`;

        detectedStockItems.sort((a, b) => b.priority - a.priority);
        const stockListEl = document.getElementById('aiStockAnalyzerList');
        if(stockListEl) {
            if(detectedStockItems.length > 0) {
                stockListEl.innerHTML = "";
                detectedStockItems.forEach(item => {
                    let isDanger = item.info.includes("🚨");
                    stockListEl.insertAdjacentHTML('beforeend', `<li class="ai-stock-item" style="border-color:${isDanger?'#ff7675':'#00cec9'}; background:${isDanger?'#2e1b1b':'#1e1e1e'};">
                        <span>${item.name}</span><strong style="color:${isDanger?'#ff7675':'#fff'};">${item.info}</strong>
                    </li>`);
                });
            } else {
                stockListEl.innerHTML = `<li style="color:#aaa;">現在この月にアラート対象の食材・日用品はありません。クリーンです！✨</li>`;
            }
        }

        renderAllCharts(shopCounts, impCounts, weeklyCounts, dailyCounts);
    }

    function renderAllCharts(shopCounts, impCounts, weeklyCounts, dailyCounts) {
        const ctxShop = document.getElementById('shopChart');
        if(ctxShop) {
            if(myShopChart) myShopChart.destroy();
            myShopChart = new Chart(ctxShop, {
                type: 'pie',
                data: { labels: Object.keys(shopCounts), datasets: [{ data: Object.values(shopCounts), backgroundColor: neonColors }] },
                options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { position: 'right', labels: { color: '#fff', font:{size:10} } } } }
            });
        }

        const ctxImp = document.getElementById('importanceChart');
        if(ctxImp) {
            if(myImpChart) myImpChart.destroy();
            myImpChart = new Chart(ctxImp, {
                type: 'pie',
                data: { labels: Object.keys(impCounts), datasets: [{ data: Object.values(impCounts), backgroundColor: ["#00b894", "#fdcb6e", "#ff7675"] }] },
                options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { position: 'right', labels: { color: '#fff', font:{size:10} } } } }
            });
        }

        const ctxWeekly = document.getElementById('weeklyChart');
        if(ctxWeekly) {
            if(myWeeklyChart) myWeeklyChart.destroy();
            myWeeklyChart = new Chart(ctxWeekly, {
                type: 'bar',
                data: { labels: Object.keys(weeklyCounts), datasets: [{ label: '支出額', data: Object.values(weeklyCounts), backgroundColor: ["#00cec9", "#6c5ce7", "#fd79a8", "#fdcb6e"] }] },
                options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { display: false } }, scales: { x: { ticks: { color: '#fff', font:{size:9} } }, y: { ticks: { color: '#fff', font:{size:9} } } } }
            });
        }

        const ctxDaily = document.getElementById('dailyChart');
        if(ctxDaily) {
            if(myDailyChart) myDailyChart.destroy();
            myDailyChart = new Chart(ctxDaily, {
                type: 'line',
                data: { labels: Object.keys(dailyCounts).map(d => `${d}日`), datasets: [{ data: Object.values(dailyCounts), borderColor: '#00cec9', tension: 0.2, fill: false, pointBackgroundColor: '#ff7675', pointRadius:2 }] },
                options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { display: false } }, scales: { x: { ticks: { color: '#fff', font:{size:8} } }, y: { ticks: { color: '#fff', font:{size:9} } } } }
            });
        }
    }

    function exportToExcel() {
        if (dataList.length === 0) { alert("データがないよ！"); return; }
        const wb = XLSX.utils.book_new();
        const totalTaxIn = dataList.reduce((sum, item) => sum + item.amountInTax, 0);
        const sheet1Data = dataList.map(item => ({ "日時": item.date, "店名": item.shop, "商品名": item.product, "金額(税込)": item.amountInTax, "カテゴリ": item.category, "必要度": item.importance, "メモ": item.memo || "" }));
        sheet1Data.push({ "日時": "【合計】", "店名": "", "商品名": "", "金額(税込)": totalTaxIn, "カテゴリ": "", "必要度": "", "メモ": "" });
        const ws1 = XLSX.utils.json_to_sheet(sheet1Data); XLSX.utils.book_append_sheet(wb, ws1, "データ出力");
        XLSX.writeFile(wb, "parsimonia_hybrid_data.xlsx");
    }

    // DOMContentLoadedを使い、HTML構造が読み終わってから全てのイベントや安全ガードを施す
    document.addEventListener("DOMContentLoaded", function() {
        const form = document.getElementById('receiptForm');
        if(form) {
            form.addEventListener('submit', function(e) {
                e.preventDefault();
                const newItem = {
                    date: getSafeValue('date'), 
                    shop: getSafeValue('shop'), 
                    product: getSafeValue('product'),
                    amountExTax: parseInt(getSafeValue('amountExTax')) || 0, 
                    amountInTax: parseInt(getSafeValue('amountInTax')) || 0,
                    deposit: parseInt(getSafeValue('deposit')) || 0, 
                    change: parseInt(getSafeValue('change')) || 0,
                    category: getSafeValue('category'), 
                    importance: getSafeValue('importance'), 
                    memo: getSafeValue('memo')
                };
                saveReceipt(newItem);
                
                const elShop = document.getElementById('shop'); if(elShop) elShop.value = ''; 
                const elProd = document.getElementById('product'); if(elProd) elProd.value = ''; 
                const elExTax = document.getElementById('amountExTax'); if(elExTax) elExTax.value = ''; 
                const elInTax = document.getElementById('amountInTax'); if(elInTax) elInTax.value = ''; 
                const elDep = document.getElementById('deposit'); if(elDep) elDep.value = ''; 
                const elChange = document.getElementById('change'); if(elChange) elChange.value = ''; 
                const elMemo = document.getElementById('memo'); if(elMemo) elMemo.value = '';
            });
        }
    });
</script>
</body>
</html>
