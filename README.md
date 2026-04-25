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
</style>
</head>

<body>

<!-- LOGIN -->
<div id="loginBox" style="
position:fixed;inset:0;background:#000c;
display:flex;justify-content:center;align-items:center;z-index:99999;
">

<div style="background:#111827;padding:20px;border-radius:12px;width:300px;text-align:center;">

<h3>🔐 تسجيل دخول</h3>

<input id="loginName" placeholder="الاسم">
<input id="loginPass" type="password" placeholder="الباسورد">

<button onclick="login()" style="width:100%;padding:10px;background:#2563eb;color:#fff;border:none;border-radius:8px;">دخول</button>

<p id="loginError" style="color:red"></p>

</div>
</div>

<header>🏛️ وزارة الداخلية</header>

<div class="nav">
<button class="active" onclick="show('army',this)">👮 العسكريين</button>
<button onclick="show('points',this)">⭐ النقاط</button>
<button onclick="show('warns',this)">⚠ التحذيرات</button>
<button onclick="show('notes',this)">📝 الملاحظات</button>
<button onclick="show('logs',this)">📜 السجل</button>
</div>

<div class="container">

<!-- ARMY -->
<div id="army" class="section active">

<div class="card" id="addBox">
<h3>➕ إضافة عسكري</h3>
<input id="name" placeholder="اسم">
<input id="gid" placeholder="ID">

<select id="rank"></select>

<select id="unit">
<option>قوات طوارئ</option>
<option>أمن عام</option>
<option>المرور</option>
<option>SWAT</option>
</select>

<button class="primary" onclick="add()">إضافة</button>
</div>

<div id="armyList"></div>
</div>

<div id="points" class="section"></div>
<div id="warns" class="section"></div>
<div id="notes" class="section"></div>
<div id="logs" class="section"></div>

</div>

<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-database-compat.js"></script>

<script>

let currentUser=null;
let logs=[];

/* USERS */
const users=[
{name:"admin",pass:"0008",role:"admin"},
{name:"officer",pass:"0808",role:"officer"},
{name:"viewer",pass:"1",role:"viewer"}
];

/* LOGIN + SAVE NAME */
function login(){
let n=document.getElementById("loginName").value.trim();
let p=document.getElementById("loginPass").value.trim();

let u=users.find(x=>x.name===n && x.pass===p);
if(!u){document.getElementById("loginError").innerText="خطأ";return;}

currentUser=u;
localStorage.setItem("lastUser",n);

document.getElementById("loginBox").style.display="none";
applyRole();
}

/* AUTO NAME */
window.onload=()=>{
let last=localStorage.getItem("lastUser");
if(last)document.getElementById("loginName").value=last;
load();
}

/* ROLE CONTROL */
function applyRole(){

if(currentUser.role==="viewer"){
document.getElementById("addBox").style.display="none";
}

}

/* FIREBASE */
const firebaseConfig={
apiKey:"AIzaSyBw9V3yMuiUa5MgurcxW2V1RCImC6eHcGQ",
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

document.getElementById("rank").innerHTML=ranks.map(r=>`<option>${r}</option>`).join("");

function show(id,btn){
document.querySelectorAll(".section").forEach(s=>s.classList.remove("active"));
document.querySelectorAll(".nav button").forEach(b=>b.classList.remove("active"));
document.getElementById(id).classList.add("active");
btn.classList.add("active");
}

function log(action,name){
logs.push(`${currentUser.name} → ${action} (${name})`);
renderLogs();
}

/* LOAD */
function load(){
db.ref("players").on("value",s=>{
let arr=Object.entries(s.val()||{}).map(([id,v])=>({id,...v}));

renderArmy(arr);
renderPoints(arr);
renderWarns(arr);
renderLogs();
});
}

/* ADD */
function add(){
db.ref("players").push({
name:name.value||"بدون اسم",
gid:gid.value||"بدون ID",
rank:rank.value,
unit:unit.value,
points:0,
warn:0
});

log("إضافة عسكري",name.value);
}

/* PERMISSIONS */
function canEdit(){return currentUser.role==="admin";}
function canOfficer(){return currentUser.role==="admin"||currentUser.role==="officer";}

/* POINTS */
function addPoint(id){
if(!canOfficer())return;

db.ref("players/"+id).once("value",s=>{
let p=s.val();
p.points++;
db.ref("players/"+id).set(p);
log("نقطة",p.name);
});
}

/* WARNS */
function addWarn(id){
if(!canOfficer())return;

db.ref("players/"+id).once("value",s=>{
let p=s.val();
p.warn++;
db.ref("players/"+id).set(p);
log("تحذير",p.name);
});
}

/* DELETE ONLY ADMIN */
function deletePlayer(id){
if(!canEdit())return;
db.ref("players/"+id).remove();
}

/* RENDER ARMY */
function renderArmy(d){
armyList.innerHTML=d.map(p=>`
<div class="card">
<b>${p.name}</b><br>
<span class="smallTag">${p.rank}</span>
<span class="smallTag">${p.unit}</span>
<br><br>

${canOfficer()?`
<button onclick="addPoint('${p.id}')">⭐</button>
<button onclick="addWarn('${p.id}')">⚠</button>
`:``}

${canEdit()?`
<button onclick="deletePlayer('${p.id}')">🗑</button>
`:``}

</div>
`).join("");
}

/* LOGS */
function renderLogs(){
document.getElementById("logs").innerHTML=
logs.map(l=>`<div class="card">${l}</div>`).join("");
}

</script>

</body>
</html>
