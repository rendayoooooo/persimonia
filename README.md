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

    <!-- 🔥 Firebase SDK（動作を安定させるため安定版に固定） -->
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
            --ai-glow: var(--neon-green);
        }
        body { font-family: 'Mochiy Pop One', 'Zen Maru Gothic', sans-serif; background-color: var(--bg-color); color: #e0e0e0; margin: 0; padding: 20px; transition: background 0.5s; }
        .container { max-width: 1200px; margin: 0 auto; }
        
        .login-card { background: #1a1a2e; border: 2px dashed var(--neon-purple); padding: 15px; border-radius: 12px; margin-bottom: 20px; display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; gap: 10px; box-shadow: 0 0 10px rgba(108,92,231,0.3); }
        .btn-google { font-family: inherit; background: white; color: #333; border: none; padding: 10px 18px; border-radius: 8px; font-weight: bold; cursor: pointer; display: flex; align-items: center; gap: 8px; box-shadow: 0 4px 0 #bbb; }
        .btn-google:active { transform: translateY(4px); box-shadow: none; }
        .user-info { display: flex; align-items: center; gap: 10px; }
        .user-pic { width: 32px; height: 32px; border-radius: 50%; border: 2px solid var(--neon-green); }

        .status-header { display: flex; flex-wrap: wrap; gap: 15px; justify-content: space-between; background: #252525; padding: 15px; border-radius: 12px; border: 2px solid var(--neon-purple); margin-bottom: 25px; }
        .status-item { font-size: 1em; color: #fff; }
        .status-val { color: var(--neon-yellow); font-weight: bold; }
        h1 { color: var(--neon-pink); text-align: center; font-size: 2.2em; text-shadow: 0 0 10px var(--neon-pink); margin-bottom: 25px; }
        h2, h3 { color: var(--neon-green); border-bottom: 3px dashed var(--neon-green); padding-bottom: 5px; }
        
        .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 25px; margin-bottom: 25px; }
        @media (max-width: 900px) { .grid { grid-template-columns: 1fr; } }
        .card { background: var(--card-bg); padding: 25px; border-radius: 16px; box-shadow: 0 8px 16px rgba(0,0,0,0.5); border: 2px solid #2d2d2d; }
        
        .ai-room { background: #151515; border: 3px solid var(--ai-glow); border-radius: 16px; padding: 20px; text-align: center; margin-bottom: 25px; box-shadow: 0 0 15px var(--ai-glow); }
        .ai-avatar { font-size: 3.5em; animation: bounce 1s infinite alternate; display: inline-block; }
        .ai-dialogue { background: #252525; border-radius: 12px; padding: 12px; margin-top: 10px; border: 1px solid #444; color: #fff; font-size: 0.95em; }
        
        .hp-bar-container { background: #333; height: 24px; border-radius: 12px; overflow: hidden; position: relative; border: 1px solid #555; }
        .hp-bar { background: linear-gradient(90deg, #ff7675, #d63031); height: 100%; width: 100%; transition: width 0.4s; }
        .wish-bar { background: linear-gradient(90deg, #6c5ce7, #a29bfe); height: 100%; width: 0%; transition: width 0.4s; }
        .bar-label { position: absolute; width: 100%; text-align: center; top: 2px; font-size: 0.85em; font-weight: bold; color: #fff; text-shadow: 1px 1px 2px #000; }

        .calendar { display: grid; grid-template-columns: repeat(7, 1fr); gap: 5px; background: #1a1a1a; padding: 10px; border-radius: 12px; border: 1px solid #333; }
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
        .stat-card { background: #252525; padding: 10px; border-radius: 12px; text-align: center; border: 2px solid #444; cursor: pointer; }
        .stat-num { font-size: 1.2em; font-weight: bold; color: #fff; margin-top: 3px; }
        .detail-panel { display: none; background: #2d2d2d; border: 2px solid var(--neon-yellow); border-radius: 12px; padding: 12px; margin-bottom: 15px; font-size: 0.85em; max-height: 150px; overflow-y: auto; }
        .detail-panel.active { display: block; }
        
        .btn-action { font-family: inherit; border: none; padding: 10px; border-radius: 10px; cursor: pointer; font-weight: bold; color: white; transition: 0.2s; }
        .btn-pink { background: var(--neon-pink); box-shadow: 0 3px 0 #d63031; }
        .btn-purple { background: var(--neon-purple); box-shadow: 0 3px 0 #5641e4; }
        .btn-green { background: var(--neon-green); box-shadow: 0 3px 0 #009473; width: 100%; }
        .btn-excel { background: #e1b12c; box-shadow: 0 4px 0 #cc8e14; color: #121212; font-size: 1.1em; width: 100%; padding: 12px; border-radius: 12px; margin-top: 20px; text-shadow: 0 0 2px #fff; }
        .btn-excel:hover { background: #fbc531; transform: translateY(-2px); box-shadow: 0 6px 0 #cc8e14; }
        .btn-excel:active { transform: translateY(4px); box-shadow: none; }
        @keyframes bounce { from { transform: translateY(0); } to { transform: translateY(-8px); } }
    </style>
</head>
<body>

<div class="container">
    <h1>⚡ parsimonia 〜家計簿アプリ〜 ⚡</h1>

    <!-- Google 認証エリア -->
    <div class="login-card">
        <div id="authStatusText" style="color:var(--neon-yellow);">🔒 クラウド未接続：ログインすると個人データが自動同期されます</div>
        <div id="userZone">
            <button class="btn-google" onclick="loginWithGoogle()">
                <img src="https://fonts.gstatic.com/s/i/productlogos/googleg/v6/web-24dp/logo_googleg_color_1x_web_24dp.png" alt="G" style="width:16px;">
                Googleアカウントでログイン
            </button>
        </div>
    </div>

    <!-- ステータスヘッダー -->
    <div class="status-header">
        <div class="status-item">PLAYER: <span class="status-val" id="playerName" style="color:#ff7675;">GUEST</span></div>
        <div class="status-item">節約LV: <span class="status-val" id="playerLevel">1</span></div>
        <div class="status-item">EXP: <span class="status-val" id="playerExp">0/100</span></div>
        <div class="status-item">現在の称号: <span class="status-val" id="playerTitle" style="color:#55efc4;">ノーマル市民</span></div>
        <div class="status-item">入力ストリーク: <span class="status-val" id="streakDays">1日連続中！🔥</span></div>
    </div>

    <!-- Gemini API設定 -->
    <div class="card" style="padding:12px; margin-bottom:20px; display:flex; justify-content:space-between; align-items:center;">
        <label style="margin:0;">🔑 Gemini API Key: </label>
        <input type="password" id="apiKeyInput" placeholder="キーを入力すると画像スキャンが解放" style="width:60%; margin:0;" onchange="saveApiKey()">
    </div>

    <!-- AIちゃんのお部屋 -->
    <div class="ai-room" id="aiRoom">
        <div class="ai-avatar" id="aiAvatar">😺</div>
        <div class="ai-dialogue" id="aiMoodSpeech">ハロー！ログインボタンを押すと、Googleの安全なログイン画面に移動するよ！</div>
        
        <div class="grid" style="margin-top: 20px; gap: 15px; margin-bottom: 0;">
            <div class="gauge-zone">
                <label style="color:#fff; font-size:0.8em;">💖 残り予算HPメーター（ゼロでゲームオーバー）</label>
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

    <div class="grid">
        <!-- 左カラム -->
        <div>
            <div class="card" style="margin-bottom: 25px;">
                <h2>① レシートスキャン ＆ 入力フォーム</h2>
                <div style="border: 2px dashed var(--neon-pink); padding: 15px; border-radius: 12px; text-align: center; background: #251818; margin-bottom: 15px;">
                    <label style="color: var(--neon-pink); cursor:pointer;">📸 写真を選択するかカメラを起動
                        <input type="file" accept="image/*" style="display: none;" onchange="handleFileSelect(this)">
                    </label>
                    <div id="scanStatus" style="font-size:0.8em; color:var(--neon-yellow); margin-top:5px;"></div>
                    <button type="button" class="btn-action btn-purple" style="padding:5px 10px; font-size:0.8em; margin-top:5px;" onclick="startAIScan()">✨ AIスキャン開始</button>
                </div>

                <div style="margin-bottom: 15px; display:flex; gap:10px;">
                    <button type="button" class="btn-action btn-purple" style="flex:1; font-size:0.85em;" onclick="injectFixedExpenses()">⚡ 固定費（家賃・スマホ等）を自動入力</button>
                </div>

                <form id="receiptForm">
                    <div class="form-row">
                        <div><label>日時</label><input type="date" id="date" required value="2026-06-13"></div>
                        <div><label>お店</label><input type="text" id="shop" required></div>
                    </div>
                    <div class="form-group"><label>買ったもの</label><input type="text" id="product" required></div>
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
                    <div class="form-group"><label>ひと言メモ</label><input type="text" id="memo"></div>
                    <button type="submit" class="btn-action btn-green">レシートデータをワールドに登録！</button>
                </form>
            </div>
        </div>

        <!-- 右カラム -->
        <div>
            <div class="card">
                <h2>② リアルタイム集計 ＆ ご褒美設定</h2>
                
                <div class="form-row" style="background:#252525; padding:10px; border-radius:8px; margin-bottom:15px;">
                    <div><label>今月の予算</label><input type="number" id="monthlyBudget" value="50000" oninput="updateApp()"></div>
                    <div><label>ご褒美アイテム</label><input type="text" id="wishName" value="欲しかったアウター🧥" oninput="updateApp()"></div>
                    <div><label>アイテムの値段</label><input type="number" id="wishPrice" value="12000" oninput="updateApp()"></div>
                </div>

                <div class="summary-box">
                    <div class="stat-card" onclick="togglePanel('panel-total')">総支出(税込)<div class="stat-num" id="totalAmount" style="color:var(--neon-pink);">¥0</div></div>
                    <div class="stat-card" onclick="togglePanel('panel-save')">浮いた金額<div class="stat-num" id="savedAmount" style="color:var(--neon-purple);">¥0</div></div>
                </div>
                <div id="panel-total" class="detail-panel"><strong>🚨 支出ログ内訳:</strong><div id="panel-total-list"></div></div>
                <div id="panel-save" class="detail-panel"><strong>✨ セーフプール資金詳細:</strong><div id="panel-save-info"></div></div>

                <div class="summary-box">
                    <div class="stat-card">不要かも支出<div class="stat-num" id="wasteAmount" style="color:#ff7675;">¥0</div></div>
                    <div class="stat-card">ヘビロテ店舗<div class="stat-num" id="topShop">-</div></div>
                </div>

                <h3>📅 マンスリー支出カレンダー (2026年6月)</h3>
                <div class="calendar" id="monthlyCalendar"></div>

                <div class="chart-container"><canvas id="categoryChart"></canvas></div>
                <div class="chart-container"><canvas id="weeklyChart"></canvas></div>
                <div class="chart-container"><canvas id="dailyChart"></canvas></div>

                <h3>🤖 AIちゃんからのネオンアドバイス</h3>
                <div class="ai-box" id="aiAdvice">データを入れるとここにアドバイスが届くよ！</div>

                <button type="button" class="btn-action btn-excel" onclick="exportToExcel()">📊 このデータをExcelシートに出力（合計自動算出）</button>
            </div>
        </div>
    </div>

    <div class="card" style="overflow-x: auto;">
        <h2>③ レシートログマスターデータベース</h2>
        <table id="receiptTable">
            <thead>
                <tr><th>日時</th><th>店名</th><th>商品名</th><th>金額(税込)</th><th>カテゴリ</th><th>必要度</th><th>メモ</th><th>アクション</th></tr>
            </thead>
            <tbody></tbody>
        </table>
    </div>
</div>

<script>
    // ⚙️ Reonくん専用の接続キー
    const firebaseConfig = {
        apiKey: "AIzaSyCXPKNZXmAceeC4IwECxy7u5EjAIzlKVmc",
        authDomain: "parsimonia-kakeiboapp.firebaseapp.com",
        projectId: "parsimonia-kakeiboapp",
        storageBucket: "parsimonia-kakeiboapp.firebasestorage.app",
        messagingSenderId: "720989097838",
        appId: "1:720989097838:web:a145ec4c5b751138cf3b7f",
        measurementId: "G-9640DXW5JW"
    };

    // 🚀 【鉄壁化】ボタンを押した瞬間に確実に初期化とジャンプを強制する仕組み
    function getFirebase() {
        if (!firebase.apps.length) {
            firebase.initializeApp(firebaseConfig);
        }
        return firebase;
    }

    const categories = ["食費", "外食", "飲み物", "カフェ", "日用品", "交通", "娯楽", "学校", "服・美容", "サブスク", "医療", "その他"];
    const importanceLevels = ["必要", "やや必要", "不要かも"];
    const neonColors = ["#ff7675", "#fdcb6e", "#00b894", "#0984e3", "#6c5ce7", "#fd79a8", "#e84393", "#ffeaa7", "#fab1a0", "#74b9ff", "#a29bfe", "#55efc4"];

    let playerLV = 1; let playerEXP = 0; let base64Image = "";
    let dataList = [];
    let currentUser = null;

    window.onload = function() {
        if(localStorage.getItem('gemini_api_key')) document.getElementById('apiKeyInput').value = localStorage.getItem('gemini_api_key');
        buildCalendarFramework();
        
        // 起動時にログイン結果の待ち受け状態を作る
        const fb = getFirebase();
        fb.auth().getRedirectResult().then(result => {
            if (result && result.user) {
                console.log("Redirectログイン成功", result.user);
            }
        }).catch(e => console.error("Redirectエラー", e));

        fb.auth().onAuthStateChanged(user => {
            if (user) {
                currentUser = user;
                document.getElementById('authStatusText').innerText = "🟢 クラウド同期中：オンラインの強固なデータベースに保護されています";
                document.getElementById('playerName').innerText = user.displayName.toUpperCase();
                document.getElementById('userZone').innerHTML = `<div class="user-info"><img class="user-pic" src="${user.photoURL}"> <button class="btn-action btn-pink" style="padding:5px 10px; font-size:0.8em;" onclick="logout()">ログアウト</button></div>`;
                loadCloudData();
            } else {
                currentUser = null;
                document.getElementById('authStatusText').innerText = "🔒 ゲストモード：ログインするとクラウド保存が解放されます";
                document.getElementById('playerName').innerText = "GUEST";
                document.getElementById('userZone').innerHTML = `<button class="btn-google" onclick="loginWithGoogle()"><img src='https://fonts.gstatic.com/s/i/productlogos/googleg/v6/web-24dp/logo_googleg_color_1x_web_24dp.png' style='width:16px;'> Googleログイン</button>`;
                dataList = [
                    { id: 1, date: "2026-06-13", shop: "ネオカフェ", product: "ゲスト用サンプルお茶", amountExTax: 500, amountInTax: 550, category: "カフェ", importance: "不要かも", memo: "ログインしてみてね！" }
                ];
                updateApp();
            }
        });
    }

    // 🚀 ボタンを押した瞬間に確実に作動させる最強のジャンプ関数
    function loginWithGoogle() { 
        try {
            const fb = getFirebase();
            const provider = new fb.auth.GoogleAuthProvider(); 
            fb.auth().signInWithRedirect(provider); 
        } catch(e) {
            alert("ログイン処理の起動に失敗しました。時間をおいて再度お試しください。");
            console.error(e);
        }
    }
    
    function logout() { getFirebase().auth().signOut(); }
    function saveApiKey() { localStorage.setItem('gemini_api_key', document.getElementById('apiKeyInput').value); }

    function loadCloudData() {
        const fb = getFirebase();
        fb.firestore().collection("users").doc(currentUser.uid).collection("receipts").orderBy("date", "asc")
        .onSnapshot(snapshot => {
            dataList = [];
            snapshot.forEach(doc => { dataList.push({ id: doc.id, ...doc.data() }); });
            playerEXP = dataList.length * 15;
            updateApp();
        }, e => console.error("Firestoreロードエラー:", e));
    }

    function saveReceipt(newItem) {
        if(currentUser) {
            getFirebase().firestore().collection("users").doc(currentUser.uid).collection("receipts").add(newItem);
        } else {
            dataList.push({ id: Date.now(), ...newItem });
            updateApp();
        }
    }

    function deleteItem(id) {
        if(currentUser && typeof id === "string" && id.length > 5) {
            getFirebase().firestore().collection("users").doc(currentUser.uid).collection("receipts").doc(id).delete();
        } else {
            dataList = dataList.filter(item => item.id != id);
            updateApp();
        }
    }

    function buildCalendarFramework() {
        const cal = document.getElementById('monthlyCalendar'); cal.innerHTML = '';
        ["日","月","火","水","木","金","土"].forEach(h => cal.insertAdjacentHTML('beforeend', `<div class="cal-day-head">${h}</div>`));
        cal.insertAdjacentHTML('beforeend', `<div class="cal-cell" style="visibility:hidden;"></div>`);
        for(let day=1; day<=30; day++) { cal.insertAdjacentHTML('beforeend', `<div class="cal-cell" id="cal-day-${day}"><span class="cal-num">${day}</span><span class="cal-amt" id="cal-amt-${day}"></span></div>`); }
    }

    function injectFixedExpenses() {
        saveReceipt({ date: "2026-06-01", shop: "家賃管理組合", product: "家賃固定費", amountExTax: 40000, amountInTax: 40000, category: "その他", importance: "必要", memo: "自動枠" });
        saveReceipt({ date: "2026-06-01", shop: "サイバーモバイル", product: "スマホ通信代", amountExTax: 2980, amountInTax: 3278, category: "サブスク", importance: "必要", memo: "自動枠" });
    }

    function handleFileSelect(input) { const file = input.files[0]; if (file) { const reader = new FileReader(); reader.onload = function(e) { base64Image = e.target.result.split(',')[1]; document.getElementById('scanStatus').innerText = "📷 スキャンボタンを押してね"; }; reader.readAsDataURL(file); } }
    async function startAIScan() {
        const apiKey = document.getElementById('apiKeyInput').value; if(!apiKey || !base64Image) { alert("キーと画像を入れてね！"); return; }
        document.getElementById('scanStatus').innerText = "⏳ AIちゃんが解析中...";
        const promptText = `JSON形式だけでレシートデータを解析して。フォーマットは前回と同じで。`;
        const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${apiKey}`;
        try {
            const response = await fetch(url, { method: "POST", headers: { "Content-Type": "application/json" }, body: JSON.stringify({ contents: [{ parts: [{ text: promptText }, { inlineData: { mimeType: "image/jpeg", data: base64Image } }] }] }) });
            const result = await response.json(); const cleanJson = result.candidates[0].content.parts[0].text.replace(/```json/g, "").replace(/
```/g, "").trim(); const receiptData = JSON.parse(cleanJson);
            document.getElementById('date').value = receiptData.date; document.getElementById('shop').value = receiptData.shop; document.getElementById('product').value = receiptData.product; document.getElementById('amountInTax').value = receiptData.amountInTax; document.getElementById('category').value = receiptData.category; document.getElementById('importance').value = receiptData.importance;
            document.getElementById('scanStatus').innerText = "✅ 完了！下の登録を押してね！";
        } catch(e) { document.getElementById('scanStatus').innerText = "❌ 解析エラー"; }
    }

    function calcTax() { const ex = document.getElementById('amountExTax').value; if(ex) document.getElementById('amountInTax').value = Math.round(ex*1.1); }
    function calcChange() { const In = document.getElementById('amountInTax').value; const dep = document.getElementById('deposit').value; if(In && dep) document.getElementById('change').value = dep - In >= 0 ? dep - In : 0; }
    function togglePanel(id) { const p = document.getElementById(id); p.classList.toggle('active'); }

    const catSelect = document.getElementById('category'); categories.forEach(c => catSelect.add(new Option(c, c)));
    const impSelect = document.getElementById('importance'); importanceLevels.forEach(i => impSelect.add(new Option(i, i)));
    let myCatChart, myWeeklyChart, myDailyChart;

    let global_savedMoney = 0; let global_waste = 0; let global_maxShop = "-"; let global_maxCat = "-"; let global_catCounts = {};

    function updateApp() {
        const tbody = document.querySelector('#receiptTable tbody'); const totalListDiv = document.getElementById('panel-total-list');
        tbody.innerHTML = ''; totalListDiv.innerHTML = '';
        for(let d=1; d<=30; d++) { const el = document.getElementById(`cal-amt-${d}`); if(el) el.innerText = ''; }

        let total = 0; global_waste = 0; global_catCounts = {}; let shopCounts = {};
        let weeklyCounts = { "1週目":0, "2週目":0, "3週目":0, "4週目":0 };
        let dailyCounts = {}; for(let d=1; d<=30; d++) dailyCounts[d] = 0;

        dataList.forEach(item => {
            total += item.amountInTax;
            let detailTxt = `<div class="detail-item"><span>${item.date} [${item.shop}]</span><strong>¥${item.amountInTax}</strong></div>`;
            totalListDiv.insertAdjacentHTML('beforeend', detailTxt);

            if(item.importance === "不要かも") global_waste += item.amountInTax;
            global_catCounts[item.category] = (global_catCounts[item.category] || 0) + item.amountInTax;
            shopCounts[item.shop] = (shopCounts[item.shop] || 0) + item.amountInTax;

            let dayNum = parseInt(item.date.split("-")[2]);
            if(!isNaN(dayNum) && dayNum <= 30) {
                dailyCounts[dayNum] += item.amountInTax;
                if(dayNum <= 7) weeklyCounts["1週目"] += item.amountInTax;
                else if(dayNum <= 14) weeklyCounts["2週目"] += item.amountInTax;
                else if(dayNum <= 21) weeklyCounts["3週目"] += item.amountInTax;
                else weeklyCounts["4週目"] += item.amountInTax;
            }

            tbody.insertAdjacentHTML('beforeend', `<tr>
                <td>${item.date}</td><td>${item.shop}</td><td>${item.product}</td><td><strong>¥${item.amountInTax}</strong></td>
                <td>${item.category}</td><td><span class="badge ${item.importance==='必要'?'badge-needed':item.importance==='やや必要'?'badge-maybe':'badge-no'}">${item.importance}</span></td>
                <td>${item.memo||''}</td><td><button class="btn-delete" onclick="deleteItem('${item.id}')">削除</button></td>
            </tr>`);
        });

        for(let d=1; d<=30; d++) { if(dailyCounts[d] > 0) document.getElementById(`cal-amt-${d}`).innerText = `¥${dailyCounts[d].toLocaleString()}`; }
        document.getElementById('totalAmount').innerText = `¥${total.toLocaleString()}`;
        document.getElementById('wasteAmount').innerText = `¥${global_waste.toLocaleString()}`;

        const budget = parseInt(document.getElementById('monthlyBudget').value) || 50000;
        let hpPercent = ((budget - total) / budget) * 100; if(hpPercent < 0) hpPercent = 0;
        document.getElementById('hpBar').style.width = `${hpPercent}%`;
        document.getElementById('hpBarLabel').innerText = `HP: ¥${(budget - total).toLocaleString()} / ¥${budget.toLocaleString()} (${hpPercent.toFixed(0)}%)`;

        global_savedMoney = budget - total >= 0 ? budget - total : 0;
        document.getElementById('savedAmount').innerText = `¥${global_savedMoney.toLocaleString()}`;
        document.getElementById('panel-save-info').innerText = `現在のセーフ資金残高は ¥${global_savedMoney.toLocaleString()} です。`;

        const wishName = document.getElementById('wishName').value; const wishPrice = parseInt(document.getElementById('wishPrice').value) || 1;
        let wishPercent = (global_savedMoney / wishPrice) * 100; if(wishPercent > 100) wishPercent = 100;
        document.getElementById('wishBar').style.width = `${wishPercent}%`;
        document.getElementById('wishBarLabel').innerText = `「${wishName}」まであと ¥${(wishPrice - global_savedMoney > 0 ? wishPrice - global_savedMoney : 0).toLocaleString()} (${wishPercent.toFixed(0)}%)`;

        playerLV = Math.floor(playerEXP / 100) + 1;
        document.getElementById('playerLevel').innerText = playerLV;
        document.getElementById('playerExp').innerText = `${playerEXP % 100}/100`;

        let title = "ノーマル市民";
        if(global_waste === 0 && total > 0) title = "👑 自炊の神";
        else if(global_catCounts["カフェ"] === undefined || global_catCounts["カフェ"] < 1000) title = "🛡️ カフェの誘惑に勝てし者";
        if(playerLV >= 3) title = "💎 節約大魔王";
        document.getElementById('playerTitle').innerText = title;

        let wasteRatio = total > 0 ? (global_waste / total) * 100 : 0;
        let aiAvatar = document.getElementById('aiAvatar'); let aiSpeech = document.getElementById('aiMoodSpeech');

        if(!currentUser) {
            aiAvatar.innerText = "🔒"; aiSpeech.innerText = "上のボタンからGoogleログインしてみて！専用のネオン世界が同期されるよ！";
        } else if(hpPercent <= 20) {
            document.documentElement.style.setProperty('--ai-glow', '#d63031'); aiAvatar.innerText = "👻";
            aiSpeech.innerText = "HP残りわずか！！ガチでゲームオーバー5秒前！これ以上の魔力消費（無駄遣い）はダメ絶対！😭";
        } else if(wasteRatio >= 15) {
            document.documentElement.style.setProperty('--ai-glow', '#ff7675'); aiAvatar.innerText = "👿";
            aiSpeech.innerText = `「不要かも」の出費が ${wasteRatio.toFixed(0)}% だよぉ。${wishName}のメーターが進まなくなっちゃう…つら。`;
        } else {
            document.documentElement.style.setProperty('--ai-glow', '#00b894'); aiAvatar.innerText = "🥰";
            aiSpeech.innerText = `現在の無駄遣い率はわずか ${wasteRatio.toFixed(0)}%！クラウド同期も完璧だし、君こそ真のハッカーだね！`;
        }

        global_maxShop = Object.keys(shopCounts).reduce((a, b) => shopCounts[a] > shopCounts[b] ? a : b, "-");
        global_maxCat = Object.keys(global_catCounts).reduce((a, b) => global_catCounts[a] > global_catCounts[b] ? a : b, "-");
        document.getElementById('topShop').innerText = global_maxShop;

        updateCharts(catCounts, weeklyCounts, dailyCounts);
    }

    function updateCharts(catCounts, weeklyCounts, dailyCounts) {
        if (myCatChart) myCatChart.destroy();
        myCatChart = new Chart(document.getElementById('categoryChart'), { type: 'pie', data: { labels: Object.keys(catCounts), datasets: [{ data: Object.values(catCounts), backgroundColor: neonColors }] }, options: { plugins: { legend: { position: 'right', labels: { color: '#fff' } } } } });
        if (myWeeklyChart) myWeeklyChart.destroy();
        myWeeklyChart = new Chart(document.getElementById('weeklyChart'), { type: 'bar', data: { labels: Object.keys(weeklyCounts), datasets: [{ data: Object.values(weeklyCounts), backgroundColor: ["#55efc4", "#a29bfe", "#fd79a8", "#fdcb6e"] }] }, options: { plugins: { legend: { display: false } }, scales: { x: { ticks: { color: '#fff' } }, y: { ticks: { color: '#fff' } } } } });
        if (myDailyChart) myDailyChart.destroy();
        myDailyChart = new Chart(document.getElementById('dailyChart'), { type: 'line', data: { labels: Object.keys(dailyCounts).map(d => `${d}日`), datasets: [{ data: Object.values(dailyCounts), borderColor: '#ff7675', tension: 0.3, fill: false }] }, options: { plugins: { legend: { display: false } }, scales: { x: { ticks: { color: '#fff' } }, y: { ticks: { color: '#fff' } } } } });
    }

    function exportToExcel() {
        if (dataList.length === 0) { alert("データがないよ！"); return; }
        const wb = XLSX.utils.book_new();

        const totalTaxIn = dataList.reduce((sum, item) => sum + item.amountInTax, 0);
        const sheet1Data = dataList.map(item => ({
            "日時": item.date, "店名": item.shop, "商品名": item.product, "金額(税込)": item.amountInTax,
            "カテゴリ": item.category, "必要度": item.importance, "メモ": item.memo || "",
            "節約候補": item.importance === "不要かも" ? "★ムダ遣い注意" : ""
        }));
        
        sheet1Data.push({
            "日時": "【合計】", "店名": "", "商品名": "", "金額(税込)": totalTaxIn,
            "カテゴリ": "", "必要度": "", "メモ": "", "節約候補": ""
        });
        
        const ws1 = XLSX.utils.json_to_sheet(sheet1Data);
        XLSX.utils.book_append_sheet(wb, ws1, "レシート入力");

        const sheet2Data = [
            ["項目", "金額 / 内容"],
            ["今月の支出合計(税込)", totalTaxIn],
            ["不要かもしれない出費の合計", global_waste],
            ["今月浮いたお金（残予算）", global_savedMoney],
            ["", ""],
            ["【カテゴリ別支出内訳】", ""]
        ];
        Object.keys(global_catCounts).forEach(cat => { sheet2Data.push([cat, global_catCounts[cat]]); });
        const ws2 = XLSX.utils.aoa_to_sheet(sheet2Data);
        XLSX.utils.book_append_sheet(wb, ws2, "月別まとめ");

        const aiComment = document.getElementById('aiAdvice').innerText;
        const sheet3Data = [
            ["分析項目", "診断結果"],
            ["今月一番お金を使ったカテゴリ", global_maxCat],
            ["今月一番お金を使った店", global_maxShop],
            ["AIちゃんからのアドバイス", aiComment]
        ];
        const ws3 = XLSX.utils.aoa_to_sheet(sheet3Data);
        XLSX.utils.book_append_sheet(wb, ws3, "節約分析");

        XLSX.writeFile(wb, "parsimonia_kakeibo_data.xlsx");
    }

    document.getElementById('receiptForm').addEventListener('submit', function(e) {
        e.preventDefault();
        const newItem = {
            date: document.getElementById('date').value, shop: document.getElementById('shop').value, product: document.getElementById('product').value,
            amountExTax: parseInt(document.getElementById('amountExTax').value) || 0, amountInTax: parseInt(document.getElementById('amountInTax').value),
            deposit: parseInt(document.getElementById('deposit').value) || 0, change: parseInt(document.getElementById('change').value) || 0,
            category: document.getElementById('category').value, importance: document.getElementById('importance').value, memo: document.getElementById('memo').value
        };
        saveReceipt(newItem);
        document.getElementById('shop').value = ''; document.getElementById('product').value = ''; document.getElementById('amountExTax').value = ''; document.getElementById('amountInTax').value = ''; document.getElementById('deposit').value = ''; document.getElementById('change').value = ''; document.getElementById('memo').value = '';
    });
</script>
</body>
</html>

