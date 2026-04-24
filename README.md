<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<title>وزارة الداخلية</title>

<style>
body{margin:0;font-family:Tahoma;background:#070b14;color:#fff}
header{background:#0f172a;padding:16px;text-align:center;font-size:22px;font-weight:bold}

.container{padding:12px;max-width:1100px;margin:auto}

.card{background:#111827;padding:10px;margin:10px 0;border-radius:10px;border:1px solid #1f2a44}

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
<h3>🔐 دخول</h3>
<input id="pass" placeholder="الباسورد">
<button class="primary" onclick="login()">دخول</button>
</div>

<div id="app" style="display:none">

<div class="card">
<h3>➕ إضافة عسكري</h3>

<input id="name" placeholder="اسم العسكري">
<input id="id" placeholder="Game ID (إجباري)">
<input id="discord" placeholder="Discord (اختياري)">

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
<input id="search" placeholder="اكتب الاسم">
<button onclick="find()">بحث</button>
<div id="result"></div>
</div>

<div id="list"></div>

</div>

</div>

<script>

const PASS="0008";

const ranks=[
"جندي","جندي أول","عريف","وكيل رقيب","رقيب","رقيب أول",
"رئيس رقباء","ملازم","ملازم أول","نقيب","رائد","مقدم",
"عقيد","عميد","لواء","فريق","فريق أول"
];

function login(){
if(pass.value!==PASS)return alert("❌ خطأ");
app.style.display="block";
init();
render();
}

function init(){
rank.innerHTML=ranks.map(r=>`<option>${r}</option>`).join("");
}

function get(){return JSON.parse(localStorage.getItem("data")||"[]")}
function save(d){localStorage.setItem("data",JSON.stringify(d))}

function add(){
let d=get();

if(!name.value || !id.value){
return alert("الاسم + Game ID مطلوب");
}

d.push({
name:name.value,
id:id.value,
discord:discord.value || "لا يوجد",
rank:rank.value,
unit:unit.value,
points:0,
warn:0
});

save(d);
render();
}

/* عرض */
function render(){
let d=get();

list.innerHTML=d.map((p,i)=>`
<div class="card">
<b>${p.name}</b>

<div class="tag">🆔 ${p.id}</div>
<div class="tag">💬 ${p.discord}</div>
<div class="tag">🎖 ${p.rank}</div>
<div class="tag">🚓 ${p.unit}</div>

<div class="tag">⭐ ${p.points}</div>
<div class="tag">⚠ ${p.warn}</div>

<div>
<button class="primary" onclick="point(${i})">⭐ بوينت</button>
<button class="warn" onclick="warn(${i})">⚠ تحذير</button>
<button class="danger" onclick="del(${i})">حذف</button>
</div>

</div>
`).join("");
}

/* بحث */
function find(){
let d=get();
let q=search.value.trim();

let p=d.find(x=>x.name.includes(q));

if(!p)return result.innerHTML="❌ غير موجود";

let i=d.indexOf(p);

result.innerHTML=`
<div class="card">
<b>${p.name}</b>

<div>🆔 ${p.id}</div>
<div>💬 ${p.discord}</div>
<div>🎖 ${p.rank}</div>
<div>🚓 ${p.unit}</div>

<button class="primary" onclick="point(${i})">⭐ بوينت</button>
<button class="warn" onclick="warn(${i})">⚠ تحذير</button>
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
if(!confirm("متأكد؟"))return;
d.splice(i,1);
save(d);
render();
}

</script>

</body>
</html>
