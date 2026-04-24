<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<title>وزارة الداخلية</title>

<style>
body{margin:0;font-family:Tahoma;background:#070b14;color:#fff}
header{background:#0f172a;padding:16px;text-align:center;font-size:22px}

.container{max-width:1100px;margin:auto;padding:12px}

.card{background:#111827;padding:10px;margin:10px 0;border-radius:10px}

input,select{
width:100%;padding:10px;margin:5px 0;
background:#0a0e14;color:#fff;border:none;border-radius:8px}

button{padding:8px;border:none;border-radius:8px;cursor:pointer}
.primary{background:#2563eb;color:#fff}
.warn{background:#f59e0b}
.danger{background:#ef4444}

.tag{display:inline-block;background:#1f2937;padding:4px 8px;border-radius:6px;margin:3px;font-size:12px}
</style>
</head>

<body>

<header>🏛️ وزارة الداخلية</header>

<div class="container">

<div class="card">
<h3>➕ إضافة عسكري</h3>

<input id="name" placeholder="اسم">
<input id="gid" placeholder="Game ID">
<input id="disc" placeholder="Discord (اختياري)">

<select id="rank"></select>

<select id="unit">
<option>Patrol</option>
<option>Traffic</option>
<option>SWAT</option>
<option>Investigation</option>
</select>

<button class="primary" onclick="add()">إضافة</button>
</div>

<div class="card">
<h3>🔎 بحث</h3>
<input id="search" placeholder="اسم">
<button onclick="find()">بحث</button>
<div id="result"></div>
</div>

<div id="list"></div>

</div>

<script>

const ranks=[
"جندي","جندي أول","عريف","وكيل رقيب","رقيب","رقيب أول",
"رئيس رقباء","ملازم","ملازم أول","نقيب","رائد","مقدم",
"عقيد","عميد","لواء","فريق","فريق أول"
];

const colors={
"جندي":"#6b7280",
"عريف":"#22c55e",
"رقيب":"#f59e0b",
"ملازم":"#3b82f6",
"نقيب":"#a855f7",
"مقدم":"#eab308",
"عقيد":"#ef4444",
"لواء":"#111827",
"فريق أول":"#000"
};

window.onload=()=>{
rank.innerHTML=ranks.map(r=>`<option>${r}</option>`).join("");
render();
};

function get(){return JSON.parse(localStorage.getItem("data")||"[]")}
function save(d){localStorage.setItem("data",JSON.stringify(d))}

/* إضافة */
function add(){

let d=get();

if(!name.value || !gid.value){
alert("الاسم + Game ID");
return;
}

d.push({
name:name.value,
gid:gid.value,
disc:disc.value||"لا يوجد",
rank:rank.value,
unit:unit.value,
points:0,
warn:0
});

save(d);

name.value="";
gid.value="";
disc.value="";

render();
}

/* عرض */
function render(){

let d=get();

d.sort((a,b)=>ranks.indexOf(b.rank)-ranks.indexOf(a.rank));

list.innerHTML=d.map((p,i)=>`
<div class="card">

<b>${p.name}</b>

<div class="tag" style="background:${colors[p.rank]}">${p.rank}</div>
<div class="tag">🚓 ${p.unit}</div>
<div class="tag">🆔 ${p.gid}</div>

<div class="tag">⭐ ${p.points}</div>
<div class="tag">⚠ ${p.warn}</div>

<div>
<button class="primary" onclick="point(${i})">⭐</button>
<button class="warn" onclick="warn(${i})">⚠</button>
<button class="danger" onclick="del(${i})">🗑</button>
</div>

</div>
`).join("");
}

/* بحث */
function find(){

let d=get();
let q=search.value.trim();

let p=d.find(x=>x.name.includes(q));

if(!p){
result.innerHTML="❌ غير موجود";
return;
}

let i=d.indexOf(p);

result.innerHTML=`
<div class="card">
<b>${p.name}</b>
<div>🎖 ${p.rank}</div>
<div>🚓 ${p.unit}</div>

<button class="primary" onclick="point(${i})">⭐</button>
<button class="warn" onclick="warn(${i})">⚠</button>
</div>
`;
}

/* بوينت */
function point(i){
let d=get();
d[i].points++;

if(d[i].points>=3){
let idx=ranks.indexOf(d[i].rank);
if(idx<ranks.length-1)d[i].rank=ranks[idx+1];
d[i].points=0;
}

save(d);
render();
}

/* تحذير */
function warn(i){
let d=get();
d[i].warn++;

if(d[i].warn>=3){
let idx=ranks.indexOf(d[i].rank);
if(idx>0)d[i].rank=ranks[idx-1];
d[i].warn=0;
}

save(d);
render();
}

/* حذف */
function del(i){
let d=get();
if(!confirm("حذف؟"))return;
d.splice(i,1);
save(d);
render();
}

</script>

</body>
</html>
