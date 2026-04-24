<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>وزارة الداخلية - Velora RP</title>

<style>
body{
margin:0;
font-family:Tahoma;
color:#fff;
background: linear-gradient(-45deg,#050814,#0b1220,#0a0e1a,#050816);
background-size:400% 400%;
animation:bg 10s ease infinite;
}

@keyframes bg{
0%{background-position:0% 50%}
50%{background-position:100% 50%}
100%{background-position:0% 50%}
}

header{
background:#0f172a;
padding:16px;
text-align:center;
font-size:22px;
font-weight:bold;
}

/* 🔥 القوائم فوق */
.nav{
display:flex;
gap:6px;
justify-content:center;
background:#0b1220;
padding:10px;
flex-wrap:wrap;
}

.nav button{
padding:8px;
border:none;
border-radius:8px;
background:#1f2937;
color:#fff;
cursor:pointer;
}

.nav button.active{background:#2563eb}

.container{max-width:1100px;margin:auto;padding:12px}

.card{
background:#111827cc;
padding:10px;
margin:10px 0;
border-radius:10px;
backdrop-filter: blur(5px);
}

input,select{
width:100%;
padding:10px;
margin:5px 0;
background:#0a0e14;
color:#fff;
border:none;
border-radius:8px;
}

button{padding:6px 8px;border:none;border-radius:6px;cursor:pointer}
.primary{background:#2563eb;color:#fff}
.warn{background:#f59e0b}
.danger{background:#ef4444}

.tag{
display:inline-block;
background:#1f2937;
padding:4px 8px;
border-radius:6px;
margin:3px;
font-size:12px;
}

.section{display:none}
.section.active{display:block}
</style>
</head>

<body>

<header>🏛️ وزارة الداخلية - Velora RP</header>

<!-- إضافة -->
<div class="container">

<div class="card">
<h3>➕ إضافة عسكري</h3>

<input id="name" placeholder="اسم">
<input id="gid" placeholder="Game ID">

<select id="rank"></select>

<select id="unit">
<option>قوات طوارئ</option>
<option>أمن عام</option>
<option>المرور</option>
<option>التحقيقات</option>
</select>

<button class="primary" onclick="add()">إضافة</button>
</div>

<!-- ملاحظات -->
<div class="card">
<h3>📝 إضافة ملاحظة</h3>
<select id="noteSelect"></select>
<input id="noteText" placeholder="الملاحظة">
<button class="primary" onclick="addNote()">إضافة</button>
</div>

</div>

<!-- القوائم فوق -->
<div class="nav">
<button class="active" onclick="show('army',this)">👮 العسكريين</button>
<button onclick="show('points',this)">⭐ النقاط</button>
<button onclick="show('warns',this)">⚠ التحذيرات</button>
<button onclick="show('notes',this)">📝 الملاحظات</button>
</div>

<div class="container">

<div id="army" class="section active"></div>
<div id="points" class="section"></div>
<div id="warns" class="section"></div>
<div id="notes" class="section"></div>

</div>

<!-- Firebase -->
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-database-compat.js"></script>

<script>

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
"رقيب أول","رقيب","وكيل رقيب","عريف","جندي أول","جندي"
];

window.onload=()=>{
rank.innerHTML=ranks.map(r=>`<option>${r}</option>`).join("");
load();
};

/* تبويب */
function show(id,btn){
document.querySelectorAll(".section").forEach(s=>s.classList.remove("active"));
document.querySelectorAll(".nav button").forEach(b=>b.classList.remove("active"));
document.getElementById(id).classList.add("active");
btn.classList.add("active");
}

/* تحميل */
function load(){
db.ref("players").on("value",snap=>{
let data=snap.val()||{};
let arr=Object.entries(data).map(([id,v])=>({id,...v}));
render(arr);
fillSelect(arr);
});
}

/* ➕ إضافة (إصلاح الاسم) */
function add(){

let n=name.value.trim();
let g=gid.value.trim();

db.ref("players").push({
name: n || "بدون اسم",
gid: g || "بدون ID",
rank:rank.value,
unit:unit.value,
points:0,
warn:0,
notes:[]
});

name.value="";
gid.value="";
}

/* ⭐ */
function addPoint(id){
db.ref("players/"+id).once("value",snap=>{
let p=snap.val();
p.points++;

if(p.points>=3){
p.points=0;
let i=ranks.indexOf(p.rank);
if(i>0)p.rank=ranks[i-1];
}

db.ref("players/"+id).set(p);
});
}

/* ⚠ */
function addWarn(id){
db.ref("players/"+id).once("value",snap=>{
let p=snap.val();
p.warn++;

if(p.warn>=3){
p.warn=0;
let i=ranks.indexOf(p.rank);
if(i<ranks.length-1)p.rank=ranks[i+1];
}

db.ref("players/"+id).set(p);
});
}

/* 🗑 حذف ملاحظة */
function deleteNote(pid,index){
db.ref("players/"+pid).once("value",snap=>{
let p=snap.val();
p.notes.splice(index,1);
db.ref("players/"+pid).set(p);
});
}

/* 📝 إضافة ملاحظة */
function addNote(){

let id=noteSelect.value;
let text=noteText.value;

if(!id||!text)return;

db.ref("players/"+id).once("value",snap=>{
let p=snap.val();
if(!p.notes)p.notes=[];

p.notes.push({
text:text,
time:new Date().toLocaleString("ar")
});

db.ref("players/"+id).set(p);
noteText.value="";
});
}

/* تعبئة */
function fillSelect(arr){
noteSelect.innerHTML=arr.map(p=>
`<option value="${p.id}">${p.name}</option>`
).join("");
}

/* عرض */
function render(d){

army.innerHTML=d.map(p=>`
<div class="card">
<b>${p.name}</b><br>

<div class="tag">🎖 ${p.rank}</div>
<div class="tag">🚓 ${p.unit}</div>

<div class="tag">⭐ ${p.points}</div>
<div class="tag">⚠ ${p.warn}</div>

<button class="primary" onclick="addPoint('${p.id}')">⭐</button>
<button class="warn" onclick="addWarn('${p.id}')">⚠</button>

</div>
`).join("");

points.innerHTML=d.map(p=>`
<div class="card">${p.name} ⭐ ${p.points}</div>
`).join("");

warns.innerHTML=d.map(p=>`
<div class="card">${p.name} ⚠ ${p.warn}</div>
`).join("");

notes.innerHTML=d.map(p=>`
<div class="card">
<b>${p.name}</b><br>

${(p.notes||[]).map((n,i)=>
`📝 ${n.text} - ${n.time}
<button class="danger" onclick="deleteNote('${p.id}',${i})">حذف</button><br>`
).join("")}

</div>
`).join("");

}

</script>

</body>
</html>
