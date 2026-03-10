<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Smart Aquarium Cloud Control</title>
  <link href="https://fonts.googleapis.com/css2?family=Kanit:wght@300;400;500&display=swap" rel="stylesheet">
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
  
  <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
  <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-database.js"></script>

  <style>
    :root { 
      --primary-color: #ff4a26; 
      --bg-color: #f0f4f8; 
      --card-bg: #ffffff; 
      --success: #28a745; 
      --danger: #dc3545; 
      --shadow-soft: 0 8px 24px rgba(149, 157, 165, 0.1);
      --shadow-hover: 0 15px 35px rgba(149, 157, 165, 0.2);
    }

    body { 
      font-family: 'Kanit', sans-serif; 
      background: linear-gradient(135deg, #e0f2fe 0%, #f0f4f8 100%); 
      margin: 0; padding: 0; color: #333; min-height: 100vh; 
    }

    /* Header & Clock */
    .header { 
      background: linear-gradient(90deg, #a44f00, #f78843, #ffffff); 
      color: white; padding: 20px 30px; font-size: 24px; font-weight: 500; 
      display: flex; justify-content: space-between; align-items: center; 
      box-shadow: 0 4px 12px rgba(0,0,0,0.1); 
    }
    .clock-display { 
      font-family: 'Courier New', monospace; font-weight: bold; font-size: 22px; 
      background: rgba(0,0,0,0.2); padding: 5px 15px; border-radius: 10px; 
    }

    /* Tabs */
    .tab-container { display: flex; justify-content: center; margin: 20px 0; }
    .tab { 
      display: flex; background: white; border-radius: 30px; padding: 5px; 
      box-shadow: var(--shadow-soft); width: 90%; max-width: 400px; 
    }
    .tab button { 
      flex: 1; padding: 12px; border: none; background: none; font-size: 16px; 
      cursor: pointer; border-radius: 25px; transition: 0.3s; color: #777; 
    }
    .tab button.active { background: var(--primary-color); color: white; font-weight: 500; }
    .tab button:hover:not(.active) { background: #fff5f2; color: var(--primary-color); }

    .container { padding: 10px 20px 40px; max-width: 1000px; margin: auto; }
    .dashboard-row { display: flex; flex-wrap: wrap; gap: 20px; justify-content: center; }

    /* --- เอฟเฟกต์ลอยขึ้นเมื่อชี้ (Card Hover) --- */
    .card { 
      flex: 1; min-width: 300px; background: var(--card-bg); padding: 25px; 
      border-radius: 20px; box-shadow: var(--shadow-soft); 
      transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275); 
    }
    .card:hover {
      transform: translateY(-10px);
      box-shadow: var(--shadow-hover);
    }

    .status-title { 
      font-size: 18px; color: #555; margin-bottom: 20px; display: flex; 
      align-items: center; gap: 10px; border-bottom: 2px solid #f0f4f8; padding-bottom: 10px; 
    }
    .status-title i { transition: transform 0.3s; }
    .card:hover .status-title i { transform: rotate(15deg) scale(1.2); color: var(--primary-color); }

    .status-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; }
    .status-item { 
      background: #f8fafc; padding: 20px 15px; border-radius: 15px; 
      text-align: center; border: 1px solid #edf2f7; transition: 0.3s;
    }
    .status-item:hover { background: #fff; border-color: var(--primary-color); transform: scale(1.05); }
    .status-item.full-width { grid-column: span 2; }
    .status-val { display: block; font-size: 18px; font-weight: 500; margin-top: 8px; }

    /* --- ปุ่มกด (Button Styles & Interactive) --- */
    .control-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; }
    .btn { 
      border: none; padding: 15px 10px; border-radius: 12px; color: white; 
      font-size: 14px; cursor: pointer; transition: all 0.2s; 
      display: flex; flex-direction: column; align-items: center; gap: 8px; 
      box-shadow: 0 4px 0 rgba(0,0,0,0.1);
    }
    .btn:hover { filter: brightness(1.1); transform: translateY(-3px); box-shadow: 0 6px 12px rgba(0,0,0,0.15); }
    .btn:active { transform: translateY(2px); box-shadow: none; } /* เอฟเฟกต์ตอนกดปุ่ม */

    .btn-light { background: #6366f1; } 
    .btn-oxy { background: #06b6d4; } 
    .btn-off { background: #94a3b8; }
    .btn-feed { 
      background: linear-gradient(45deg, #f59e0b, #ef4444); 
      grid-column: span 2; flex-direction: row; justify-content: center; font-size: 16px; 
    }

    /* Settings Form */
    .input-group { margin-bottom: 15px; }
    label { display: block; margin-bottom: 5px; font-weight: 500; }
    input[type="time"] { 
      width: 100%; padding: 12px; border: 2px solid #edf2f7; border-radius: 10px; 
      font-family: inherit; box-sizing: border-box; transition: 0.3s;
    }
    input[type="time"]:focus { border-color: var(--primary-color); outline: none; box-shadow: 0 0 8px rgba(255,74,38,0.2); }
    
    .btn-save { 
      background: var(--primary-color); width: 100%; padding: 15px; border-radius: 12px; 
      color: white; font-weight: bold; border: none; cursor: pointer; margin-top: 10px; 
      transition: 0.3s;
    }
    .btn-save:hover { background: #e03a1a; transform: translateY(-2px); box-shadow: 0 5px 15px rgba(255,74,38,0.3); }

    .on-text { color: var(--success); font-weight: bold; } 
    .off-text { color: var(--danger); }
    .feed-text { color: #f59e0b; font-weight: bold; }
  </style>
</head>
<body>

  <div class="header">
    <div><i class="fas fa-fish"></i> Smart Aquarium Cloud</div>
    <div class="clock-display" id="clock">00:00:00</div>
  </div>
  
  <div class="tab-container">
    <div class="tab">
      <button id="btn-status" class="tablinks active" onclick="openTab(event, 'Status')">แดชบอร์ด</button>
      <button id="btn-settings" class="tablinks" onclick="openTab(event, 'Settings')">ตั้งค่า</button>
    </div>
  </div>

  <div id="Status" class="tabcontent">
    <div class="container dashboard-row">
      <div class="card">
        <div class="status-title"><i class="fas fa-microchip"></i> สถานะอุปกรณ์</div>
        <div class="status-grid">
          <div class="status-item"><span>ไฟส่องสว่าง</span><span id="s-light" class="status-val">...</span></div>
          <div class="status-item"><span>ออกซิเจน</span><span id="s-oxy" class="status-val">...</span></div>
          <div class="status-item full-width"><span>ระบบให้อาหาร</span><span id="s-feed" class="status-val">รอคำสั่ง...</span></div>
        </div>
      </div>
      <div class="card">
        <div class="status-title"><i class="fas fa-gamepad"></i> แผงควบคุม</div>
        <div class="control-grid">
          <button class="btn btn-light" onclick="toggle('light', 'on')"><i class="fas fa-lightbulb"></i>เปิดไฟ</button>
          <button class="btn btn-off" onclick="toggle('light', 'off')"><i class="fas fa-power-off"></i>ปิดไฟ</button>
          <button class="btn btn-oxy" onclick="toggle('oxygen', 'on')"><i class="fas fa-wind"></i>เปิดอ๊อก</button>
          <button class="btn btn-off" onclick="toggle('oxygen', 'off')"><i class="fas fa-power-off"></i>ปิดอ๊อก</button>
          <button class="btn btn-feed" onclick="feed()"><i class="fas fa-utensils"></i> ให้อาหารทันที</button>
        </div>
      </div>
    </div>
  </div>

  <div id="Settings" class="tabcontent" style="display:none;">
    <div class="container">
      <div class="card" style="max-width: 500px; margin: auto;">
        <div class="status-title"><i class="fas fa-clock"></i> ตั้งเวลาทำงาน (Cloud)</div>
        <form id="timerForm" onsubmit="saveSettings(event)">
          <div class="input-group">
            <label>เวลาเปิดไฟ:</label>
            <input type="time" name="lightOn" id="in-lightOn">
          </div>
          <div class="input-group">
            <label>เวลาปิดไฟ:</label>
            <input type="time" name="lightOff" id="in-lightOff">
          </div>
          <div class="input-group">
            <label>เวลาให้อาหาร:</label>
            <input type="time" name="feedTime" id="in-feedTime">
          </div>
          <button type="submit" class="btn-save">บันทึกข้อมูลลง Cloud</button>
        </form>
      </div>
    </div>
  </div>

  <script>
    // Firebase Config (คงเดิมตามที่คุณให้มา)
    const firebaseConfig = {
      apiKey: "AIzaSyDw0Y6-Fb2d2yDCsNPmg2HfwU5KGJ6HDv0",
      authDomain: "ton12-3fdec.firebaseapp.com",
      databaseURL: "https://ton12-3fdec-default-rtdb.asia-southeast1.firebasedatabase.app",
      projectId: "ton12-3fdec",
      storageBucket: "ton12-3fdec.firebasestorage.app",
      messagingSenderId: "605181275408",
      appId: "1:605181275408:web:62be0df8ff6af16fa34995",
      measurementId: "G-5Y3NH3SR9F"
    };

    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();

    function openTab(evt, tabName) {
      var i, tabcontent, tablinks;
      tabcontent = document.getElementsByClassName("tabcontent");
      for (i = 0; i < tabcontent.length; i++) { tabcontent[i].style.display = "none"; }
      tablinks = document.getElementsByClassName("tablinks");
      for (i = 0; i < tablinks.length; i++) { tablinks[i].className = tablinks[i].className.replace(" active", ""); }
      document.getElementById(tabName).style.display = "block";
      evt.currentTarget.className += " active";
    }

    function toggle(device, state) {
      const val = (state === 'on');
      if(device === 'light') db.ref('/control/light_manual').set(val);
      if(device === 'oxygen') db.ref('/control/oxy_manual').set(val);
    }

    function feed() {
      if(confirm("ยืนยันการให้อาหารทันที?")) {
        db.ref('/control/feed_now').set(1);
      }
    }

    function saveSettings(e) {
      e.preventDefault();
      const formData = new FormData(document.getElementById('timerForm'));
      db.ref('/settings').update({
        lightOn: formData.get('lightOn'),
        lightOff: formData.get('lightOff'),
        feedTime: formData.get('feedTime')
      }).then(() => alert("บันทึกสำเร็จ!"));
    }

    db.ref().on('value', (snapshot) => {
      const data = snapshot.val();
      if (!data) return;

      document.getElementById("s-light").innerHTML = data.control?.light_manual ? '<span class="on-text">● เปิด</span>' : '<span class="off-text">○ ปิด</span>';
      document.getElementById("s-oxy").innerHTML = data.control?.oxy_manual ? '<span class="on-text">● เปิด</span>' : '<span class="off-text">○ ปิด</span>';
      
      const feedStatus = data.control?.feed_now;
      if (feedStatus === 1) {
        document.getElementById("s-feed").innerHTML = '<span class="feed-text"><i class="fas fa-spinner fa-spin"></i> กำลังให้อาหาร...</span>';
      } else {
        document.getElementById("s-feed").innerHTML = '<span class="off-text">รอคำสั่ง / ตามเวลา</span>';
      }

      if (data.settings) {
        document.getElementById("in-lightOn").value = data.settings.lightOn || "";
        document.getElementById("in-lightOff").value = data.settings.lightOff || "";
        document.getElementById("in-feedTime").value = data.settings.feedTime || "";
      }
    });

    setInterval(() => {
      document.getElementById("clock").innerText = new Date().toLocaleTimeString('th-TH');
    }, 1000);
  </script>
</body>
</html>
