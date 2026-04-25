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

/* login */
#loginBox{
position:fixed;top:0;left:0;width:100%;height:100%;
background:#000a;display:flex;justify-content:center;align-items:center;z-index:9999;
}
.loginCard{background:#111827;padding:20px;border-radius:12px;width:300px;text-align:center;}
</style>
</head>

<body>

<!-- LOGIN -->
<div id="loginBox">
<div class="loginCard">
<h3>🔐 دخول</h3>
<input type="password" id="pass" placeholder="0008">
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

<!-- الجيش -->
<div id="army" class="section active">

<div class="card">
<h3>➕ إضافة عسكري</h3>
<input id="name" placeholder="اسم">
<input id="gid" placeholder="ID">
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

<!-- النقاط -->
<div id="points" class="section"></div>

<!-- التحذيرات -->
<div id="warns" class="section"></div>

<!-- الملاحظات -->
<div id="notes" class="section">

<div class="card">
<h3>➕ إضافة ملاحظة</h3>
<select id="noteSelect"></select>
<input id="noteText">
<button class="primary" onclick="addNote()">إضافة</button>
</div>

<div id="notesList"></div>

</div>

</div>

<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-database-compat.js"></script>

<script>

/* login */
function login(){
if(pass.value==="0008"){
loginBox.style.display="none";
}else alert("خطأ");
}

const firebaseConfig = {
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

window.onload=()=>{
rank.innerHTML=ranks.map(r=>`<option>${r}</option>`).join("");
load();
};

function show(id,btn){
document.querySelectorAll(".section").forEach(s=>s.classList.remove("active"));
document.querySelectorAll(".nav button").forEach(b=>b.classList.remove("active"));
document.getElementById(id).classList.add("active");
btn.classList.add("active");
}

/* تحميل */
function load(){
db.ref("players").on("value",snap=>{
let arr=Object.entries(snap.val()||{}).map(([id,v])=>({id,...v}));
arr.sort((a,b)=>ranks.indexOf(a.rank)-ranks.indexOf(b.rank));
renderArmy(arr);
renderPoints(arr);
renderWarns(arr);
renderNotes(arr);
fillSelect(arr);
});
}

/* إضافة عسكري */
function add(){
db.ref("players").push({
name:name.value||"بدون اسم",
gid:gid.value||"بدون ID",
rank:rank.value,
unit:unit.value,
points:0,
warn:0,
notes:[]
});
name.value="";gid.value="";
}

/* تعديل شامل */
function editPlayer(id){
db.ref("players/"+id).once("value",s=>{
let p=s.val();
p.name=prompt("اسم",p.name)||p.name;
p.gid=prompt("ID",p.gid)||p.gid;
p.rank=prompt("رتبة",p.rank)||p.rank;
p.unit=prompt("قطاع",p.unit)||p.unit;
db.ref("players/"+id).set(p);
});
}

/* حذف عسكري */
function deletePlayer(id){
if(confirm("حذف؟")) db.ref("players/"+id).remove();
}

/* نقاط */
function addPoint(id){
db.ref("players/"+id).once("value",s=>{
let p=s.val();
p.points++;
if(p.points>=3){p.points=0;let i=ranks.indexOf(p.rank);if(i>0)p.rank=ranks[i-1];}
db.ref("players/"+id).set(p);
});
}
function removePoint(id){
db.ref("players/"+id).once("value",s=>{
let p=s.val();
if(p.points>0)p.points--;
db.ref("players/"+id).set(p);
});
}

/* تحذيرات */
function addWarn(id){
db.ref("players/"+id).once("value",s=>{
let p=s.val();
p.warn++;
if(p.warn>=3){p.warn=0;let i=ranks.indexOf(p.rank);if(i<ranks.length-1)p.rank=ranks[i+1];}
db.ref("players/"+id).set(p);
});
}
function removeWarn(id){
db.ref("players/"+id).once("value",s=>{
let p=s.val();
if(p.warn>0)p.warn--;
db.ref("players/"+id).set(p);
});
}

/* عرض العسكريين */
function renderArmy(d){
armyList.innerHTML=d.map(p=>`
<div class="card">
<b>${p.name}</b><br>
<span class="smallTag">🆔 ${p.gid}</span>
<span class="smallTag">${p.rank}</span>
<span class="smallTag">${p.unit}</span>
<br>
<button onclick="addPoint('${p.id}')">⭐</button>
<button onclick="addWarn('${p.id}')">⚠</button>
<button onclick="editPlayer('${p.id}')">✏</button>
<button onclick="deletePlayer('${p.id}')">🗑</button>
</div>`).join("");
}

/* نقاط */
function renderPoints(d){
let f=d.filter(p=>p.points>0);
points.innerHTML=f.length?f.map(p=>`
<div class="card">
${p.name} ⭐ ${p.points}
<button onclick="addPoint('${p.id}')">➕</button>
<button onclick="removePoint('${p.id}')">➖</button>
</div>`).join(""):"<div class='card'>لا يوجد</div>";
}

/* تحذيرات */
function renderWarns(d){
let f=d.filter(p=>p.warn>0);
warns.innerHTML=f.length?f.map(p=>`
<div class="card">
${p.name} ⚠ ${p.warn}
<button onclick="addWarn('${p.id}')">➕</button>
<button onclick="removeWarn('${p.id}')">➖</button>
</div>`).join(""):"<div class='card'>لا يوجد</div>";
}

/* ملاحظات (FIX كامل) */
function renderNotes(d){
notesList.innerHTML=d.filter(p=>p.notes&&p.notes.length>0).map(p=>`
<div class="card">
<b>${p.name}</b><br>
${p.notes.map((n,i)=>`
📝 ${n.text}
<button onclick="editNote('${p.id}',${i})">✏</button>
<button onclick="deleteNote('${p.id}',${i})">🗑</button><br>
`).join("")}
</div>`).join("");
}

/* إضافة ملاحظة */
function addNote(){
let id=noteSelect.value;
let text=noteText.value.trim();
if(!id||!text)return;

db.ref("players/"+id).once("value",s=>{
let p=s.val();
if(!p.notes)p.notes=[];
p.notes.push({text});
db.ref("players/"+id).set(p);
noteText.value="";
});
}

/* تعديل ملاحظة */
function editNote(id,i){
db.ref("players/"+id).once("value",s=>{
let p=s.val();
let t=prompt("تعديل",p.notes[i].text);
if(t){p.notes[i].text=t;db.ref("players/"+id).set(p);}
});
}

/* حذف ملاحظة */
function deleteNote(id,i){
db.ref("players/"+id).once("value",s=>{
let p=s.val();
p.notes.splice(i,1);
db.ref("players/"+id).set(p);
});
}

/* اختيار */
function fillSelect(d){
noteSelect.innerHTML=d.map(p=>`<option value="${p.id}">${p.name}</option>`).join("");
}

</script>

</body>
</html>
