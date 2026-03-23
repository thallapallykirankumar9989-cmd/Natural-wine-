<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>తాటి కల్లు లైవ్ స్టాక్</title>
    <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-database.js"></script>

    <style>
        body { font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif; background-color: #f0f2f5; margin: 0; padding: 0; display: flex; justify-content: center; }
        .app-container { width: 100%; max-width: 480px; background-color: white; min-height: 100vh; box-shadow: 0 4px 12px rgba(0,0,0,0.1); }
        .app-bar { background-color: #388e3c; color: white; padding: 16px 20px; display: flex; justify-content: space-between; align-items: center; }
        .app-bar.admin-mode { background-color: #448aff; }
        .app-title { font-size: 20px; font-weight: bold; cursor: pointer; user-select: none; }
        .lang-btn { background: transparent; border: none; color: white; font-size: 16px; font-weight: bold; cursor: pointer; }
        .content { padding: 30px 20px; text-align: center; }
        .admin-msg-card { background-color: #ffcc80; padding: 15px; border-radius: 8px; margin-bottom: 40px; display: none; text-align: left; font-weight: bold; color: #e65100; }
        .stock-label { font-size: 22px; font-weight: 500; color: #333; }
        .stock-value { font-size: 60px; font-weight: bold; color: #2e7d32; margin: 15px 0 50px 0; }
        .btn { width: 100%; padding: 15px; font-size: 18px; border-radius: 8px; cursor: pointer; font-weight: bold; border: none; color: white; display: flex; justify-content: center; align-items: center; gap: 10px; }
        .wa-btn { background-color: #43a047; }
        .update-btn { background-color: #448aff; margin-top: 20px; }
        .close-admin-btn { background-color: #d32f2f; margin-top: 15px; }
        .admin-panel { display: none; text-align: left; }
        .input-group { margin-bottom: 20px; }
        .input-group label { display: block; margin-bottom: 8px; font-weight: bold; color: #333; }
        .input-group input { width: 100%; padding: 15px; font-size: 16px; border: 1px solid #ccc; border-radius: 6px; box-sizing: border-box; }
    </style>
</head>
<body>

<div class="app-container">
    <div class="app-bar" id="appBar">
        <div class="app-title" id="appTitle">తాజా స్టాక్</div>
        <button class="lang-btn" id="langBtn">English</button>
    </div>

    <div class="content">
        <div id="customerDashboard">
            <div class="admin-msg-card" id="adminMsgCard"></div>
            <div class="stock-label" id="stockLabel">ప్రస్తుతం ఉన్న తాటి కల్లు:</div>
            <div class="stock-value" id="stockValue">0 లీటర్లు</div>
            <button class="btn wa-btn" id="waBtn">💬 వాట్సాప్ ద్వారా ఆర్డర్ చేయండి</button>
        </div>

        <div class="admin-panel" id="adminPanel">
            <h3 id="adminHeader">స్టాక్ అప్‌డేట్ చేయండి:</h3>
            <div class="input-group">
                <label id="kalluInputLabel">తాటి కల్లు (లీటర్లలో)</label>
                <input type="number" id="kalluInput" placeholder="ఉదా: 50">
            </div>
            <div class="input-group">
                <label id="msgInputLabel">కస్టమర్లకు ప్రకటన (ఉదా: ఫ్రెష్ కల్లు వచ్చింది)</label>
                <input type="text" id="msgInput" placeholder="Message...">
            </div>
            <button class="btn update-btn" id="updateBtn">అప్‌డేట్ చేయండి</button>
            <button class="btn close-admin-btn" id="closeAdminBtn">క్లోజ్ (Close Admin)</button>
        </div>
    </div>
</div>

<script>
    const appTitle = document.getElementById('appTitle');
    const langBtn = document.getElementById('langBtn');
    const appBar = document.getElementById('appBar');
    const customerDashboard = document.getElementById('customerDashboard');
    const adminMsgCard = document.getElementById('adminMsgCard');
    const stockLabel = document.getElementById('stockLabel');
    const stockValue = document.getElementById('stockValue');
    const waBtn = document.getElementById('waBtn');
    const adminPanel = document.getElementById('adminPanel');
    const adminHeader = document.getElementById('adminHeader');
    const kalluInputLabel = document.getElementById('kalluInputLabel');
    const msgInputLabel = document.getElementById('msgInputLabel');
    const kalluInput = document.getElementById('kalluInput');
    const msgInput = document.getElementById('msgInput');
    const updateBtn = document.getElementById('updateBtn');
    const closeAdminBtn = document.getElementById('closeAdminBtn');

    let isTelugu = true;
    let isAdminMode = false;
    let currentLiters = 0;

    const translations = {
        te: {
            appTitle: "తాజా స్టాక్", adminTitle: "అడ్మిన్ డ్యాష్‌బోర్డ్", langBtn: "English",
            stockLabel: "ప్రస్తుతం ఉన్న తాటి కల్లు:", liters: "లీటర్లు",
            waBtn: "💬 వాట్సాప్ ద్వారా ఆర్డర్ చేయండి", waMsg: "హలో, నాకు తాటి కల్లు కావాలి.",
            adminHeader: "స్టాక్ అప్‌డేట్ చేయండి:", kalluInputLabel: "తాటి కల్లు (లీటర్లలో)",
            msgInputLabel: "కస్టమర్లకు ప్రకటన", updateBtn: "అప్‌డేట్ చేయండి", closeBtn: "క్లోజ్ (Close)"
        },
        en: {
            appTitle: "Live Stock", adminTitle: "Admin Dashboard", langBtn: "తెలుగు",
            stockLabel: "Available Thati Kallu:", liters: "Liters",
            waBtn: "💬 Order via WhatsApp", waMsg: "Hello, I want Thati Kallu.",
            adminHeader: "Update Stock:", kalluInputLabel: "Thati Kallu (in Liters)",
            msgInputLabel: "Announcement", updateBtn: "Update", closeBtn: "Close Admin"
        }
    };

    function updateUI() {
        const t = isTelugu ? translations.te : translations.en;
        appTitle.innerText = isAdminMode ? t.adminTitle : t.appTitle;
        langBtn.innerText = t.langBtn;
        stockLabel.innerText = t.stockLabel;
        stockValue.innerText = `${currentLiters} ${t.liters}`;
        waBtn.innerText = t.waBtn;
        adminHeader.innerText = t.adminHeader;
        kalluInputLabel.innerText = t.kalluInputLabel;
        msgInputLabel.innerText = t.msgInputLabel;
        updateBtn.innerText = t.updateBtn;
        closeAdminBtn.innerText = t.closeBtn;
    }

    langBtn.addEventListener('click', () => { isTelugu = !isTelugu; updateUI(); });

    waBtn.addEventListener('click', () => {
        const adminNumber = "919989471413"; // వాట్సాప్ నెంబర్ మార్చుకోండి
        const t = isTelugu ? translations.te : translations.en;
        window.open(`https://wa.me/${adminNumber}?text=${encodeURIComponent(t.waMsg)}`, '_blank');
    });

    let clickCount = 0;
    appTitle.addEventListener('click', () => {
        if(isAdminMode) return;
        clickCount++;
        if (clickCount === 2) {
            if (prompt("Admin Password:") === "1985") {
                isAdminMode = true;
                customerDashboard.style.display = "none";
                adminPanel.style.display = "block";
                appBar.classList.add("admin-mode");
                updateUI();
            } else {
                alert("Wrong Password!");
            }
            clickCount = 0;
        }
        setTimeout(() => clickCount = 0, 1000);
    });

    closeAdminBtn.addEventListener('click', () => {
        isAdminMode = false;
        adminPanel.style.display = "none";
        customerDashboard.style.display = "block";
        appBar.classList.remove("admin-mode");
        updateUI();
    });

    updateUI();

    // 🔴 కింద ఉన్న బాక్సులో మీ ఫైర్‌బేస్ కోడ్ కాపీ చేసి పేస్ట్ చేయండి!
    const firebaseConfig = {
        apiKey: "AIzaSyBVXwmaYhzdoC79r4E5ND2Gj8BI0W9dDAIq",
        authDomain: "natural-wine-d3c21.firebaseapp.com",
        databaseURL: "https://natural-wine-d3c21-default-rtdb.firebaseio.com",
        projectId: "natural-wine-d3c21",
        storageBucket: "natural-wine-d3c21.firebasestorage.app",
        messagingSenderId: "937219921686",
        appId: "1:937219921686:web:857e8b62942366ebf02237"
    };

    let shopRef = null;

    try {
        if(firebaseConfig.apiKey !== "YOUR_API_KEY") {
            firebase.initializeApp(firebaseConfig);
            shopRef = firebase.database().ref('shop_data');

            shopRef.on('value', (snapshot) => {
                const data = snapshot.val();
                if (data) {
                    currentLiters = data.thati_kallu || 0;
                    const adminMessage = data.admin_message || "";
                    const t = isTelugu ? translations.te : translations.en;
                    stockValue.innerText = `${currentLiters} ${t.liters}`;
                    
                    if (adminMessage.trim() !== "") {
                        adminMsgCard.style.display = "block";
                        adminMsgCard.innerHTML = `📢 ${adminMessage}`;
                    } else {
                        adminMsgCard.style.display = "none";
                    }
                }
            });
        }
    } catch(err) {
        console.error("Firebase Initialization Error:", err);
    }

    // అప్‌డేట్ బటన్ నొక్కినప్పుడు
    updateBtn.addEventListener('click', () => {
        // తప్పు 1: అసలు ఫైర్‌బేస్ కోడ్ పెట్టకపోతే
        if(firebaseConfig.apiKey === "YOUR_API_KEY") {
            alert("తప్పు 1: దయచేసి ముందుగా ఫైర్‌బేస్ (Firebase) కోడ్‌ను HTML ఫైల్‌లో పేస్ట్ చేయండి. అప్పుడే ఈ బటన్ పనిచేస్తుంది!");
            return;
        }

        // తప్పు 2: కోడ్ పెట్టారు కానీ కనెక్షన్ ఫెయిల్ అయితే
        if(!shopRef) {
             alert("తప్పు 2: ఫైర్‌బేస్ కనెక్షన్ ఫెయిల్ అయ్యింది. మీ కోడ్‌లో ఏమైనా అక్షరాలు మిస్ అయ్యాయేమో మళ్ళీ చెక్ చేయండి.");
             return;
        }

        const newLiters = parseInt(kalluInput.value) || 0;
        const newMsg = msgInput.value || "";
        
        shopRef.update({
            thati_kallu: newLiters,
            admin_message: newMsg
        }).then(() => {
            alert(isTelugu ? "స్టాక్ అప్‌డేట్ చేయబడింది!" : "Stock Updated!");
            kalluInput.value = ""; msgInput.value = "";
        }).catch(err => {
            // తప్పు 3: ఫైర్‌బేస్ డేటాబేస్ రూల్స్ 'true' చేయకపోతే
            alert("తప్పు 3 (ముఖ్యమైనది!): ఫైర్‌బేస్ డేటాబేస్ రూల్స్ 'true' లో ఉన్నాయో లేదో చెక్ చేయండి! \n\nఎర్రర్ వివరాలు: " + err.message);
        });
    });
</script>

</body>
</html>
