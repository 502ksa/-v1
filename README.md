<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>وزارة الداخلية - Velora RP</title>

<style>
body{margin:0;font-family:Tahoma;color:#fff;background:#0a0f1e;}
header{background:#0f172a;padding:16px;text-align:center;font-size:22px;font-weight:bold;}
.nav{display:flex;gap:6px;justify-content:center;background:#0b1220;padding:10px;flex-wrap:wrap;position:sticky;top:0;}
.nav button{padding:8px;border:none;border-radius:8px;background:#1f2937;color:#fff;cursor:pointer;}
.nav button.active{background:#2563eb}
.container{max-width:1100px;margin:auto;padding:12px}
.card{background:#111827;padding:10px;margin:10px 0;border-radius:10px}
input,select{width:100%;padding:10px;margin:5px 0;background:#0a0e14;color:#fff;border:none;border-radius:8px}
button{padding:6px 8px;border:none;border-radius:6px;cursor:pointer}
.primary{background:#2563eb}
.warn{background:#f59e0b}
.danger{background:#ef4444}
.section{display:none}
.section.active{display:block}
.smallTag{display:inline-block;background:#1f2937;padding:3px 6px;border-radius:6px;font-size:12px;margin:2px}

/* شاشة تسجيل الدخول */
#loginBox{
position:fixed;
top:0;left:0;
width:100%;height:100%;
background:#000a;
display:flex;
justify-content:center;
align-items:center;
z-index:9999;
}
.loginCard{
background:#111827;
padding:20px;
border-radius:12px;
width:300px;
text-align:center;
}
</style>
</head>

<body>

<!-- تسجيل دخول -->
<div id="loginBox">
<div class="loginCard">
<h3>🔐 تسجيل دخول</h3>
<input type="password" id="pass" placeholder="ادخل الباسورد">
<button class="primary" onclick="login()">دخول</button>
</div>
</div>

<header>🏛️ وزارة الداخلية</header>

<div class="nav">
<button class="active" onclick="show('army',this)">👮 العسكريين</button>
<button onclick="show('points',this)">⭐ النقاط</button>
<button onclick="show('warns',this)">⚠ التحذيرات</button>
<button onclick="show('notes',this)">📝 الملاحظات</button>
</div>

<div class="container">

<div id="army" class="section active">

<div class="card">
<h3>➕ إضافة عسكري</h3>
<input id="name" placeholder="اسم">
<input id="gid" placeholder="Game ID">
<select id="rank"></select>
<select id="unit">
<option>أمن عام</option>
<option>قوات الطوارئ</option>
<option>SWAT</option>
</select>
<button class="primary" onclick="add()">إضافة</button>
</div>

<div id="armyList"></div>
</div>

<div id="points" class="section"></div>
<div id="warns" class="section"></div>

<div id="notes" class="section">
<div class="card">
<h3>➕ إضافة ملاحظة</h3>
<select id="noteSelect"></select>
<input id="noteText" placeholder="اكتب الملاحظة">
<button class="primary" onclick="addNote()">إضافة</button>
</div>
<div id="notesList"></div>
</div>

</div>

<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-database-compat.js"></script>

<script>

/* تسجيل دخول */
function login(){
let p=document.getElementById("pass").value;
if(p==="0008"){
document.getElementById("loginBox").style.display="none";
}else{
alert("الباسورد غلط");
}
}

const firebaseConfig = {
apiKey:"AIzaSy...",
authDomain:"velora-rp.firebaseapp.com",
databaseURL:"https://velora-rp-default-rtdb.europe-west1.firebasedatabase.app",
projectId:"velora-rp",
storageBucket:"velora-rp.firebasestorage.app",
messagingSenderId:"681356163114",
appId:"1:681356163114:web:c40b8d7fed229ce8e7eb53"
};

firebase.initializeApp(firebaseConfig);
const db=firebase.database();

const ranks=[
"فريق أول","فريق","لواء","عميد","عقيد","مقدم",
"رائد","نقيب","ملازم أول","ملازم",
"رئيس رقباء","رقيب أول","رقيب","وكيل رقيب","عريف","جندي أول","جندي"
];

window.onload=()=>{
document.getElementById("rank").innerHTML=ranks.map(r=>`<option>${r}</option>`).join("");
load();
};

/* باقي الكود مثل ما هو بدون تغيير */

function show(id,btn){
document.querySelectorAll(".section").forEach(s=>s.classList.remove("active"));
document.querySelectorAll(".nav button").forEach(b=>b.classList.remove("active"));
document.getElementById(id).classList.add("active");
btn.classList.add("active");
}

function load(){
db.ref("players").on("value",snap=>{
let data=snap.val()||{};
let arr=Object.entries(data).map(([id,v])=>({id,...v}));

arr.sort((a,b)=>ranks.indexOf(a.rank)-ranks.indexOf(b.rank));

renderArmy(arr);
renderPoints(arr);
renderWarns(arr);
renderNotes(arr);
fillSelect(arr);
});
}

/* باقي الدوال نفسها بدون تغيير (نفس النسخة السابقة) */

</script>

</body>
</html>
