<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>وزارة الداخلية</title>

<style>
body{margin:0;font-family:Tahoma;background:#070b14;color:#fff}
header{background:#0f172a;padding:16px;text-align:center;font-size:22px}
.container{padding:12px;max-width:1100px;margin:auto}
.card{background:#111827;padding:10px;margin:10px;border-radius:10px;border:1px solid #1f2a44}
input,select{width:100%;padding:10px;margin:5px 0;border:none;border-radius:8px;background:#0a0e14;color:#fff}
button{padding:8px;border:none;border-radius:8px;cursor:pointer}
.primary{background:#2563eb;color:#fff}
.warn{background:#f59e0b}
.danger{background:#ef4444}
.tag{display:inline-block;background:#1f2937;padding:4px 8px;border-radius:6px;margin:3px;font-size:12px}
.log{background:#000;padding:5px;margin:3px;border-radius:6px;font-size:12px}
</style>
</head>

<body>

<header>🏛️ وزارة الداخلية</header>

<div class="container">

<div class="card" id="loginBox">
<h3>🔐 دخول</h3>
<input id="pass" placeholder="الباسورد">
<button class="primary" onclick="login()">دخول</button>
</div>

<div id="app" style="display:none">

<div class="card">
<h3>➕ إضافة عسكري</h3>
<input id="name" placeholder="اسم العسكري">
<input id="id" placeholder="Game ID (إجباري)">
<input id="discord" placeholder="Discord ID (اختياري)">
<button class="primary" onclick="add()">إضافة</button>
</div>

<div class="card">
<h3>🔎 بحث عسكري</h3>
<input id="searchName" placeholder="اكتب الاسم (عربي أو إنجليزي)">
<button onclick="find()">بحث</button>
<div id="result"></div>
</div>

<div class="card">
<h3>📊 الإحصائيات</h3>
<div id="stats"></div>
</div>

<div id="list"></div>

<div class="card">
<h3>📜 السجل</h3>
<div id="logs"></div>
</div>

</div>

</div>

<script>

const PASS="0008";

function login(){
if(pass.value!==PASS)return alert("خطأ");
loginBox.style.display="none";
app.style.display="block";
render();
}

function get(){return JSON.parse(localStorage.getItem("data")||"[]")}
function save(d){localStorage.setItem("data",JSON.stringify(d))}

function log(t){
let l=JSON.parse(localStorage.getItem("logs")||"[]");
l.unshift(new Date().toLocaleString()+" - "+t);
localStorage.setItem("logs",JSON.stringify(l));
}

/* إضافة */
function add(){
let d=get();

if(!name.value || !id.value){
return alert("اسم + Game ID مطلوبين");
}

d.push({
name:name.value,
id:id.value,
discord:discord.value || "لا يوجد",
points:0,
warn:0
});

log("إضافة "+name.value);

save(d);
render();
}

/* عرض */
function render(){
let d=get();

/* ترتيب */
list.innerHTML=d.map((p,i)=>`
<div class="card">
<b>${p.name}</b>

<div class="tag">🆔 GameID: ${p.id}</div>
<div class="tag">💬 Discord: ${p.discord}</div>
<div class="tag">⭐ ${p.points}</div>
<div class="tag">⚠ ${p.warn}</div>

<div>
<button class="primary" onclick="point(${i})">⭐ بوينت</button>
<button class="warn" onclick="warn(${i})">⚠ تحذير</button>
<button class="danger" onclick="del(${i})">حذف</button>
</div>

</div>
`).join("");


/* احصائيات */
stats.innerHTML="عدد العسكريين: "+d.length;

/* سجل */
let l=JSON.parse(localStorage.getItem("logs")||"[]");
logs.innerHTML=l.slice(0,20).map(x=>`<div class="log">${x}</div>`).join("");
}

/* بحث يدعم العربي */
function find(){
let d=get();

let q=(searchName.value||"").trim();

let p=d.find(x =>
x.name.includes(q) || q.includes(x.name)
);

if(!p)return result.innerHTML="❌ غير موجود";

let i=d.indexOf(p);

result.innerHTML=`
<div class="card">
<b>${p.name}</b>

<div>🆔 ${p.id}</div>
<div>💬 ${p.discord}</div>

<button class="primary" onclick="point(${i})">⭐ بوينت</button>
<button class="warn" onclick="warn(${i})">⚠ تحذير</button>
</div>
`;
}

/* بوينت */
function point(i){
let d=get();
d[i].points++;

log("بوينت "+d[i].name);

if(d[i].points>=3){
alert("ترقية تلقائية 🔥");
d[i].points=0;
}

save(d);
render();
}

/* تحذير */
function warn(i){
let d=get();
d[i].warn++;

log("تحذير "+d[i].name);

if(d[i].warn>=3){
alert("تنزيل رتبة 🚨");
d[i].warn=0;
}

save(d);
render();
}

/* حذف */
function del(i){
let d=get();
d.splice(i,1);
save(d);
render();
}

</script>

</body>
</html>
