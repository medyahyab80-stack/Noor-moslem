<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>نور الهداية | التحديث الشامل</title>
    
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <link href="https://fonts.googleapis.com/css2?family=Amiri:wght@400;700&family=Cairo:wght@300;400;700;900&display=swap" rel="stylesheet">
    
    <style>
        :root {
            --gold: #d4af37;
            --emerald: #064e3b;
            --dark: #022c22;
            --glass: rgba(255, 255, 255, 0.1);
        }

        * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; outline: none; }
        body {
            font-family: 'Cairo', sans-serif;
            background: var(--dark);
            color: white; margin: 0; 
            height: 100vh; display: flex; flex-direction: column;
            overflow: hidden;
        }

        /* الطبقات المنبثقة */
        .full-overlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: var(--dark); z-index: 2000; display: none; flex-direction: column;
        }

        /* واجهة المستخدم */
        header { padding: 25px 20px; text-align: center; background: linear-gradient(to bottom, var(--emerald), var(--dark)); border-bottom: 1px solid var(--gold); }
        .main-content { flex: 1; overflow-y: auto; padding: 20px; padding-bottom: 100px; }
        
        /* البطاقات */
        .card { background: var(--glass); border-radius: 20px; padding: 20px; margin-bottom: 15px; border: 1px solid rgba(212,175,55,0.2); transition: 0.3s; }
        .card:active { transform: scale(0.98); background: rgba(255,255,255,0.15); }

        /* البحث */
        .search-box { display: flex; background: white; border-radius: 50px; padding: 5px 15px; margin-bottom: 20px; }
        .search-box input { border: none; flex: 1; padding: 10px; font-family: 'Cairo'; font-size: 1rem; }
        .search-box button { background: var(--gold); border: none; border-radius: 50%; width: 40px; height: 40px; color: white; cursor: pointer; }

        /* التنقل السفلي */
        nav {
            position: fixed; bottom: 0; width: 100%; background: rgba(2, 44, 34, 0.95);
            display: flex; justify-content: space-around; padding: 12px 0;
            border-top: 1px solid var(--gold); backdrop-filter: blur(15px);
        }
        .nav-item { color: #888; text-align: center; font-size: 0.75rem; flex: 1; cursor: pointer; }
        .nav-item.active { color: var(--gold); }
        .nav-item i { font-size: 1.5rem; display: block; margin-bottom: 4px; }

        /* نتائج البحث */
        .search-result-item { background: rgba(255,255,255,0.05); padding: 15px; border-radius: 10px; margin-bottom: 10px; border-right: 3px solid var(--gold); }
        .search-result-item small { color: var(--gold); }

        /* مشغل الصوت */
        .player-bar { position: fixed; bottom: 80px; width: 94%; left: 3%; background: var(--gold); color: black; padding: 12px 20px; border-radius: 15px; display: none; align-items: center; justify-content: space-between; font-weight: bold; }
    </style>
</head>
<body>

<div id="quran-reader" class="full-overlay">
    <div style="padding:15px; display:flex; justify-content:space-between; align-items:center; background:var(--emerald)">
        <button onclick="closeReader()" style="background:none; border:none; color:white; font-size:1.5rem;"><i class="fas fa-times"></i></button>
        <span id="page-title">المصحف الإلكتروني</span>
        <span></span>
    </div>
    <div style="flex:1; overflow:auto; background:#f4ece1; display:flex; justify-content:center; align-items:center;" onclick="flipPage()">
        <img id="mushaf-img" src="https://archive.org/download/quran-images-v2/001.png" style="max-width:100%; box-shadow:0 0 20px rgba(0,0,0,0.2)">
    </div>
</div>

<header>
    <div id="hijri-date" style="color:var(--gold); font-weight:bold;">-- / -- / ----</div>
    <div id="clock" style="font-size:3rem; font-weight:900; margin:10px 0;">00:00</div>
    <div id="location-text"><small><i class="fas fa-location-arrow"></i> جاري تحديد الموقع...</small></div>
</header>

<div class="main-content">
    <section id="home-sec">
        <h3 style="color:var(--gold)"><i class="fas fa-star-and-crescent"></i> مواقيت الصلاة</h3>
        <div id="prayer-times-container"></div>
    </section>

    <section id="quran-sec" style="display:none">
        <div class="search-box">
            <input type="text" id="quran-search-input" placeholder="ابحث عن آية أو كلمة بالقرآن...">
            <button onclick="searchQuran()"><i class="fas fa-search"></i></button>
        </div>
        
        <div id="search-results-area" style="display:none; margin-bottom:20px;">
            <h4 style="color:var(--gold)">نتائج البحث:</h4>
            <div id="search-results-list"></div>
            <hr>
        </div>

        <div class="grid" style="display:grid; grid-template-columns:1fr 1fr; gap:10px;">
            <div class="card" onclick="openMushaf(1)"><i class="fas fa-book-open"></i><br>قراءة المصحف</div>
            <div class="card" onclick="loadSurahsList()"><i class="fas fa-headphones"></i><br>استماع صوتي</div>
        </div>
        <div id="surahs-container" style="margin-top:15px"></div>
    </section>

    <section id="qibla-sec" style="display:none; text-align:center">
        <h3 style="color:var(--gold)">بوصلة القبلة</h3>
        <div id="compass" style="width:250px; height:250px; border:5px solid var(--gold); border-radius:50%; margin:40px auto; position:relative; transition:0.5s;">
            <div style="position:absolute; top:-40px; left:42%; font-size:2.5rem;">🕋</div>
            <div style="width:4px; height:100px; background:var(--gold); position:absolute; bottom:50%; left:50%; transform-origin:bottom;"></div>
        </div>
        <button class="card" style="width:100%" onclick="requestNotificationPermission()">تفعيل تنبيهات الأذان 🔔</button>
    </section>
</div>

<div class="player-bar" id="player-bar">
    <span id="track-name">اسم السورة</span>
    <button onclick="togglePlay()" style="background:none; border:none; font-size:1.5rem;"><i class="fas fa-pause"></i></button>
</div>

<nav>
    <div class="nav-item active" onclick="navTo('home-sec', this)"><i class="fas fa-home"></i>الرئيسية</div>
    <div class="nav-item" onclick="navTo('quran-sec', this)"><i class="fas fa-quran"></i>القرآن</div>
    <div class="nav-item" onclick="navTo('qibla-sec', this)"><i class="fas fa-compass"></i>القبلة</div>
</nav>

<script>
    let currentAudio = new Audio();
    let azanAudio = new Audio('https://www.islamcan.com/common/azan/azan3.mp3');
    let prayerTimes = {};
    let mushafPage = 1;

    // 1. التنقل بين الصفحات
    function navTo(id, el) {
        document.querySelectorAll('section').forEach(s => s.style.display = 'none');
        document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active'));
        document.getElementById(id).style.display = 'block';
        el.classList.add('active');
    }

    // 2. محرك البحث في القرآن
    async function searchQuran() {
        const query = document.getElementById('quran-search-input').value;
        if(!query) return;
        
        const res = await fetch(`https://api.alquran.cloud/v1/search/${query}/all/ar`);
        const data = await res.json();
        const resultsArea = document.getElementById('search-results-list');
        document.getElementById('search-results-area').style.display = 'block';
        
        let html = '';
        if(data.data.results.length === 0) html = '<p>لا توجد نتائج</p>';
        data.data.results.slice(0, 15).forEach(item => {
            html += `<div class="search-result-item">
                        <p style="font-family:'Amiri'; font-size:1.2rem;">${item.text}</p>
                        <small>سورة ${item.surah.name} - آية ${item.numberInSurah}</small>
                     </div>`;
        });
        resultsArea.innerHTML = html;
    }

    // 3. الأذان والتنبيهات
    function requestNotificationPermission() {
        Notification.requestPermission().then(p => {
            if(p === 'granted') alert('تم تفعيل التنبيهات بنجاح!');
        });
    }

    function checkAzan() {
        const now = new Date();
        const currentTime = now.getHours().toString().padStart(2,'0') + ":" + now.getMinutes().toString().padStart(2,'0');
        
        for(let p in prayerTimes) {
            if(currentTime === prayerTimes[p]) {
                if(azanAudio.paused) {
                    azanAudio.play();
                    new Notification("حان الآن موعد أذان " + p);
                }
            }
        }
    }
    setInterval(checkAzan, 60000);

    // 4. جلب البيانات والموقع
    navigator.geolocation.getCurrentPosition(pos => {
        const lat = pos.coords.latitude;
        const lng = pos.coords.longitude;
        fetch(`https://api.aladhan.com/v1/timings?latitude=${lat}&longitude=${lng}&method=4`)
        .then(r => r.json()).then(data => {
            const t = data.data.timings;
            prayerTimes = { "الفجر": t.Fajr, "الظهر": t.Dhuhr, "العصر": t.Asr, "المغرب": t.Maghrib, "العشاء": t.Isha };
            document.getElementById('hijri-date').innerText = data.data.date.hijri.day + " " + data.data.date.hijri.month.ar + " " + data.data.date.hijri.year;
            document.getElementById('location-text').innerHTML = `<small>موقعك: ${data.data.meta.timezone}</small>`;
            
            let html = '';
            for(let p in prayerTimes) {
                html += `<div class="card" style="display:flex; justify-content:space-between">
                            <span>${p}</span><span style="color:var(--gold); font-weight:bold">${prayerTimes[p]}</span>
                         </div>`;
            }
            document.getElementById('prayer-times-container').innerHTML = html;
        });
    });

    // 5. المصحف الصوري
    function openMushaf(p) { 
        mushafPage = p; 
        document.getElementById('quran-reader').style.display = 'flex'; 
        updateMushafImg();
    }
    function closeReader() { document.getElementById('quran-reader').style.display = 'none'; }
    function flipPage() { mushafPage++; updateMushafImg(); }
    function updateMushafImg() {
        const pStr = String(mushafPage).padStart(3, '0');
        document.getElementById('mushaf-img').src = `https://archive.org/download/quran-images-v2/${pStr}.png`;
    }

    // الساعة
    setInterval(() => {
        document.getElementById('clock').innerText = new Date().toLocaleTimeString('ar-EG', {hour:'2-digit', minute:'2-digit'});
    }, 1000);

    function loadSurahsList() {
        fetch('https://api.alquran.cloud/v1/surah').then(r => r.json()).then(data => {
            let h = '';
            data.data.forEach(s => {
                h += `<div class="card" onclick="playS(${s.number}, '${s.name}')" style="margin:5px 0"> سورة ${s.name} <i class="fas fa-play" style="float:left"></i></div>`;
            });
            document.getElementById('surahs-container').innerHTML = h;
        });
    }

    function playS(n, name) {
        currentAudio.src = `https://cdn.islamic.network/quran/audio-surah/128/ar.alafasy/${n}.mp3`;
        currentAudio.play();
        document.getElementById('player-bar').style.display = 'flex';
        document.getElementById('track-name').innerText = "سورة " + name;
    }

    function togglePlay() {
        if(currentAudio.paused) currentAudio.play(); else currentAudio.pause();
    }
</script>
</body>
</html># Noor-moslem
تطبيق نور المسلم
