<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>وزارة الداخلية - Velora RP</title>

<style>
*{box-sizing:border-box}
body{margin:0;font-family:Tahoma,Arial;background:#0b0f17;color:#fff}
header{background:#0f172a;padding:16px;text-align:center;border-bottom:2px solid #2563eb}
.small{font-size:12px;opacity:.7}
.nav{display:flex;gap:8px;flex-wrap:wrap;justify-content:center;background:#0b1220;padding:10px}
.nav button{padding:10px;border:none;border-radius:8px;background:#1f2937;color:#fff;cursor:pointer}
.nav button.active{background:#2563eb}
.container{max-width:1000px;margin:auto;padding:12px}
.card{background:#111827;border:1px solid #1f2a44;border-radius:10px;padding:12px;margin:10px 0}
input,textarea,select{width:100%;padding:10px;margin:6px 0;border:none;border-radius:8px;background:#0a0e14;color:#fff}
button.primary{background:#2563eb;color:#fff;border:none;border-radius:8px;padding:10px;cursor:pointer}
button.danger{background:#ef4444}
button.warn{background:#f59e0b}
button.gray{background:#374151}
.row{display:flex;gap:8px;flex-wrap:wrap}
.tag{display:inline-block;padding:4px 8px;border-radius:6px;background:#1f2937;margin:2px;font-size:12px}
.hidden{display:none}
.list-item{border:1px solid #1f2a44;border-radius:8px;padding:10px;margin:8px 0}
</style>
</head>

<body>

<header>
🏛️ وزارة الداخلية - Velora RP
<div class="small">نظام إدارة اللاعبين</div>
</header>

<div class="nav">
<button onclick="showPage('players')" class="active">👤 اللاعبين</button>
<button onclick="showPage('warnings')">⚠ التحذيرات</button>
<button onclick="showPage('infractions')">🚨 الإنذارات</button>
<button onclick="showPage('notes')">📝 الملاحظات</button>
<button onclick="logout()">🚪 خروج</button>
</div>

<div class="container">

<div id="loginBox" class="card">
<h3>🔐 دخول المسؤول</h3>
<input id="pass" placeholder="الباسورد">
<button class="primary" onclick="login()">دخول</button>
<p id="loginMsg"></p>
</div>

<div id="app" class="hidden">

<div class="card">
<h3>➕ إضافة لاعب</h3>
<input id="p_name" placeholder="الاسم">
<input id="p_id" placeholder="Game ID">
<input id="p_discord" placeholder="Discord">
<select id="p_rank"></select>
<button class="primary" onclick="addPlayer()">إضافة</button>
</div>

<div class="card">
<input id="search" placeholder="🔎 بحث..." oninput="render()">
</div>

<div id="playersPage"></div>
<div id="warningsPage" class="hidden"></div>
<div id="infractionsPage" class="hidden"></div>
<div id="notesPage" class="hidden"></div>

</div>
</div>

<script>

const ADMIN_PASS="0008";

const ranks=[
"جندي","جندي أول","عريف","وكيل رقيب","رقيب","رقيب أول",
"رئيس رقباء","ملازم","ملازم أول","نقيب","رائد","مقدم",
"عقيد","عميد","لواء","فريق","فريق أول"
];

function login(){
if(pass.value!==ADMIN_PASS){loginMsg.innerText="❌ خطأ";return;}
localStorage.setItem("logged","1");
loginBox.classList.add("hidden");
app.classList.remove("hidden");
initRanks();
render();
}

window.onload=()=>{
if(localStorage.getItem("logged")){
loginBox.classList.add("hidden");
app.classList.remove("hidden");
initRanks();
render();
}else{
initRanks();
}
};

function initRanks(){
p_rank.innerHTML=ranks.map(r=>`<option>${r}</option>`).join("");
}

function logout(){
localStorage.removeItem("logged");
location.reload();
}

function showPage(p){
["players","warnings","infractions","notes"].forEach(x=>{
document.getElementById(x+"Page").classList.add("hidden");
});
document.getElementById(p+"Page").classList.remove("hidden");
}

function getData(){
return JSON.parse(localStorage.getItem("players")||"[]");
}

function saveData(d){
localStorage.setItem("players",JSON.stringify(d));
}

function addPlayer(){
let d=getData();
if(!p_name.value||!p_id.value)return alert("عبي البيانات");

d.push({
name:p_name.value,
id:p_id.value,
discord:p_discord.value,
rank:p_rank.value,
warnings:[],
infractions:[],
notes:[]
});

saveData(d);
p_name.value="";p_id.value="";p_discord.value="";
render();
}

function render(){
let data=getData();
let q=(search.value||"").toLowerCase();

data.sort((a,b)=>ranks.indexOf(b.rank)-ranks.indexOf(a.rank));

playersPage.innerHTML=data.filter(p=>p.name.toLowerCase().includes(q)).map((p,i)=>`
<div class="list-item">
<b>${p.name}</b>
<div class="tag">${p.id}</div>
<div class="tag">${p.discord}</div>
<div class="tag">🎖 ${p.rank}</div>
<div>⚠ ${p.warnings.length} | 🚨 ${p.infractions.length}</div>

<div class="row">
<button class="warn" onclick="addWarning(${i})">تحذير</button>
<button class="danger" onclick="addInfraction(${i})">إنذار</button>
<button class="gray" onclick="addNote(${i})">ملاحظة</button>
<button class="danger" onclick="removePlayer(${i})">حذف</button>
</div>
</div>
`).join("");

warningsPage.innerHTML=data.map(p=>`
<div class="list-item"><b>${p.name}</b>
${p.warnings.map(w=>`<div class="tag">⚠ ${w}</div>`).join("")}
</div>`).join("");

infractionsPage.innerHTML=data.map(p=>`
<div class="list-item"><b>${p.name}</b>
${p.infractions.map(w=>`<div class="tag">🚨 ${w}</div>`).join("")}
</div>`).join("");

notesPage.innerHTML=data.map(p=>`
<div class="list-item"><b>${p.name}</b>
${p.notes.map(n=>`<div class="tag">📝 ${n}</div>`).join("")}
</div>`).join("");
}

function addWarning(i){
let t=prompt("سبب التحذير");if(!t)return;
let d=getData();d[i].warnings.push(t);saveData(d);render();
}

function addInfraction(i){
let t=prompt("سبب الإنذار");if(!t)return;
let d=getData();d[i].infractions.push(t);saveData(d);render();
}

function addNote(i){
let t=prompt("ملاحظة");if(!t)return;
let d=getData();d[i].notes.push(t);saveData(d);render();
}

function removePlayer(i){
let d=getData();d.splice(i,1);saveData(d);render();
}

</script>

</body>
</html>
