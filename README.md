<!DOCTYPE html>
<html lang="te">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>తాటి కల్లు లైవ్ స్టాక్</title>
    <style>
        body {
            font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
            background-color: #f0f2f5;
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
        }
        .app-container {
            width: 100%;
            max-width: 480px;
            background-color: white;
            min-height: 100vh;
            box-shadow: 0 4px 12px rgba(0,0,0,0.1);
        }
        .app-bar {
            background-color: #388e3c; /* Green theme */
            color: white;
            padding: 16px 20px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .app-bar.admin-mode {
            background-color: #448aff; /* Blue theme for Admin */
        }
        .app-title {
            font-size: 20px;
            font-weight: bold;
            cursor: pointer; /* To indicate it's clickable for secret login */
            user-select: none;
        }
        .lang-btn {
            background: transparent;
            border: none;
            color: white;
            font-size: 16px;
            font-weight: bold;
            cursor: pointer;
        }
        .content {
            padding: 30px 20px;
            text-align: center;
        }
        /* Message Card */
        .admin-msg-card {
            background-color: #ffcc80;
            padding: 15px;
            border-radius: 8px;
            margin-bottom: 40px;
            display: none; /* Hidden by default */
            text-align: left;
            font-weight: bold;
            color: #e65100;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        /* Stock Display */
        .stock-label {
            font-size: 22px;
            font-weight: 500;
            color: #333;
        }
        .stock-value {
            font-size: 60px;
            font-weight: bold;
            color: #2e7d32;
            margin: 15px 0 50px 0;
        }
        /* Buttons */
        .btn {
            width: 100%;
            padding: 15px;
            font-size: 18px;
            border-radius: 8px;
            cursor: pointer;
            font-weight: bold;
            border: none;
            color: white;
            display: flex;
            justify-content: center;
            align-items: center;
            gap: 10px;
        }
        .wa-btn {
            background-color: #43a047;
        }
        .update-btn {
            background-color: #448aff;
            margin-top: 20px;
        }
        .close-admin-btn {
            background-color: #d32f2f;
            margin-top: 15px;
        }
        /* Admin Panel */
        .admin-panel {
            display: none; /* Hidden by default */
            text-align: left;
        }
        .input-group {
            margin-bottom: 20px;
        }
        .input-group label {
            display: block;
            margin-bottom: 8px;
            font-weight: bold;
            color: #333;
        }
        .input-group input {
            width: 100%;
            padding: 15px;
            font-size: 16px;
            border: 1px solid #ccc;
            border-radius: 6px;
            box-sizing: border-box;
        }
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
            
            <button class="btn wa-btn" id="waBtn">
                💬 వాట్సాప్ ద్వారా ఆర్డర్ చేయండి
            </button>
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

<script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-app.js";
    import { getDatabase, ref, onValue, update } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-database.js";

    // 🔴 ముఖ్య గమనిక: మీ ఫైర్‌బేస్ ప్రాజెక్ట్ సెట్టింగ్స్ ఇక్కడ కాపీ చేసి పేస్ట్ చేయండి
    const firebaseConfig = {
        apiKey: "AIzaSyBVXwmaYhzdoC79r4E5ND2Gj8BI0W9dDAI",
        authDomain: "natural-wine-d3c21.firebaseapp.com",
        databaseURL: "https://natural-wine-d3c21-default-rtdb.firebaseio.com/",
        projectId: "natural-wine-d3c21",
        storageBucket: "natural-wine-d3c21.firebasestorage.app",
        messagingSenderId: "937219921686",
        appId: "1:937219921686:web:857e8b62942366ebf02237"
    };

    // Initialize Firebase
    const app = initializeApp(firebaseConfig);
    const database = getDatabase(app);
    const shopRef = ref(database, 'shop_data');

    // UI Elements
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

    // State
    let isTelugu = true;
    let isAdminMode = true;
    let currentLiters = 0;

    // Translations Dictionary
    const translations = {
        te: {
            appTitle: "తాజా స్టాక్",
            adminTitle: "అడ్మిన్ డ్యాష్‌బోర్డ్",
            langBtn: "English",
            stockLabel: "ప్రస్తుతం ఉన్న తాటి కల్లు:",
            liters: "లీటర్లు",
            waBtn: "💬 వాట్సాప్ ద్వారా ఆర్డర్ చేయండి",
            waMsg: "హలో, నాకు తాటి కల్లు కావాలి.",
            adminHeader: "స్టాక్ అప్‌డేట్ చేయండి:",
            kalluInputLabel: "తాటి కల్లు (లీటర్లలో)",
            msgInputLabel: "కస్టమర్లకు ప్రకటన (ఉదా: ఫ్రెష్ కల్లు వచ్చింది)",
            updateBtn: "అప్‌డేట్ చేయండి",
            closeBtn: "క్లోజ్ (Close)",
            successMsg: "స్టాక్ అప్‌డేట్ చేయబడింది!"
        },
        en: {
            appTitle: "Live Stock",
            adminTitle: "Admin Dashboard",
            langBtn: "తెలుగు",
            stockLabel: "Available Thati Kallu:",
            liters: "Liters",
            waBtn: "💬 Order via WhatsApp",
            waMsg: "Hello, I want Thati Kallu.",
            adminHeader: "Update Stock:",
            kalluInputLabel: "Thati Kallu (in Liters)",
            msgInputLabel: "Announcement (e.g., Fresh Kallu available)",
            updateBtn: "Update",
            closeBtn: "Close Admin",
            successMsg: "Stock Successfully Updated!"
        }
    };

    // Update UI language
    function updateUI() {
        const t = isTelugu ? translations.te : translations.en;
        
        appTitle.innerText = isAdminMode ? t.adminTitle : t.appTitle;
        langBtn.innerText = t.langBtn;
        
        // Customer View
        stockLabel.innerText = t.stockLabel;
        stockValue.innerText = `${currentLiters} ${t.liters}`;
        waBtn.innerText = t.waBtn;
        
        // Admin View
        adminHeader.innerText = t.adminHeader;
        kalluInputLabel.innerText = t.kalluInputLabel;
        msgInputLabel.innerText = t.msgInputLabel;
        updateBtn.innerText = t.updateBtn;
        closeAdminBtn.innerText = t.closeBtn;
    }

    // Toggle Language
    langBtn.addEventListener('click', () => {
        isTelugu = !isTelugu;
        updateUI();
    });

    // --- Firebase Live Listeners ---
    onValue(shopRef, (snapshot) => {
        const data = snapshot.val();
        if (data) {
            currentLiters = data.thati_kallu || 0;
            const adminMessage = data.admin_message || "";
            
            // Update Stock
            const t = isTelugu ? translations.te : translations.en;
            stockValue.innerText = `${currentLiters} ${t.liters}`;
            
            // Show/Hide Message Card
            if (adminMessage.trim() !== "") {
                adminMsgCard.style.display = "block";
                adminMsgCard.innerHTML = `📢 ${adminMessage}`;
            } else {
                adminMsgCard.style.display = "none";
            }
        }
    });

    // --- WhatsApp Button Logic ---
    waBtn.addEventListener('click', () => {
        const adminNumber = "919989471413"; // 🔴 మీ వాట్సాప్ నెంబర్ ఇక్కడ ఇవ్వండి
        const t = isTelugu ? translations.te : translations.en;
        const message = encodeURIComponent(t.waMsg);
        window.open(`https://wa.me/${adminNumber}?text=${message}`, '_blank');
    });

    // --- Secret Admin Login ---
    // (వెబ్‌లో లాంగ్ ప్రెస్‌కి బదులు డబుల్ క్లిక్ (Double Tap) వాడాము)
    let clickCount = 1;
    appTitle.addEventListener('click', () => {
        if(isAdminMode) return; // ఇప్పటికే అడ్మిన్ లో ఉంటే ఏమీ చేయద్దు
        
        clickCount++;
        if (clickCount === 2) {
            const password = prompt("Admin Login - Enter Password:");
            if (password === "1985") { // 🔴 పాస్‌వర్డ్ ఇక్కడ ఉంది
                isAdminMode = true;
                customerDashboard.style.display = "none";
                adminPanel.style.display = "block";
                appBar.classList.add("admin-mode");
                updateUI();
            } else if (password !== null) {
                alert("Wrong Password!");
            }
            clickCount = 0;
        }
        setTimeout(() => clickCount = 0, 1000); // 1 సెకనులో డబుల్ క్లిక్ చేయాలి
    });

    // Close Admin
    closeAdminBtn.addEventListener('click', () => {
        isAdminMode = false;
        adminPanel.style.display = "none";
        customerDashboard.style.display = "block";
        appBar.classList.remove("admin-mode");
        updateUI();
    });

    // --- Update Stock to Firebase ---
    updateBtn.addEventListener(clickCount === 2) => {
        const newLiters = parseInt(kalluInput.value) || 0;
        const newMsg = msgInput.value || "";

        update(shopRef, {
            thati_kallu: newLiters,
            admin_message: newMsg
        }).then(() => {
            const t = isTelugu ? translations.te : translations.en;
            alert(t.successMsg);
            kalluInput.value = ""; // ఇన్‌పుట్స్ క్లియర్ చేయడం
            msgInput.value = "";
        }).catch((error) => {
            alert("Error: " + error);
        });
    });

    // Initial Load
    updateUI();
</script>

</body>
</html>
