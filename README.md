<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>وزارة الداخلية</title>

<style>
body{margin:0;font-family:Tahoma;background:#070b14;color:#fff}

header{
background:#0f172a;
padding:16px;
text-align:center;
font-size:22px;
font-weight:bold;
}

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

.card{background:#111827;padding:10px;margin:10px 0;border-radius:10px}

input,select{
width:100%;
padding:10px;
margin:5px 0;
background:#0a0e14;
color:#fff;
border:none;
border-radius:8px;
}

button{padding:8px;border:none;border-radius:8px;cursor:pointer}
.primary{background:#2563eb;color:#fff}
.warn{background:#f59e0b}
.danger{background:#ef4444}

.tag{display:inline-block;background:#1f2937;padding:4px 8px;border-radius:6px;margin:3px;font-size:12px}

.section{display:none}
.section.active{display:block}

/* Modal */
.modal{
display:none;
position:fixed;
top:0;left:0;
width:100%;height:100%;
background:rgba(0,0,0,0.7);
justify-content:center;
align-items:center;
}

.modal-content{
background:#111827;
padding:15px;
width:90%;
max-width:400px;
border-radius:10px;
}
</style>
</head>

<body>

<header>🏛️ وزارة الداخلية</header>

<div class="nav">
<button class="active" onclick="show('army',this)">👮 العسكريين</button>
<button onclick="show('points',this)">⭐ البوينتات</button>
<button onclick="show('warns',this)">⚠ التحذيرات</button>
<button onclick="show('notes',this)">📝 الملاحظات</button>
</div>

<div class="container">

<!-- إضافة -->
<div class="card">
<h3>➕ إضافة عسكري</h3>

<input id="name" placeholder="اسم">
<input id="gid" placeholder="Game ID">
<input id="disc" placeholder="Discord">

<select id="rank"></select>

<select id="unit">
<option>قوات طوارئ</option>
<option>أمن عام</option>
<option>المرور</option>
<option>التحقيقات</option>
</select>

<button class="primary" onclick="add()">إضافة</button>
</div>

<!-- بحث -->
<div class="card">
<h3>🔎 بحث</h3>
<input id="search" placeholder="اسم">
<button onclick="find()">بحث</button>
<div id="result"></div>
</div>

<!-- الأقسام -->
<div id="army" class="section active"></div>
<div id="points" class="section"></div>
<div id="warns" class="section"></div>
<div id="notes" class="section"></div>

</div>

<!-- Modal -->
<div class="modal" id="modal">
<div class="modal-content">

<h3>✏ تعديل عسكري</h3>

<input id="edit_name">
<input id="edit_gid">
<input id="edit_disc">

<select id="edit_rank"></select>

<select id="edit_unit">
<option>قوات طوارئ</option>
<option>أمن عام</option>
<option>المرور</option>
<option>التحقيقات</option>
</select>

<button class="primary" onclick="saveEdit()">حفظ</button>
<button class="danger" onclick="closeModal()">إغلاق</button>

</div>
</div>

<script>

const ranks=[
"فريق أول","فريق","لواء","عميد","عقيد","مقدم",
"رائد","نقيب","ملازم أول","ملازم",
"رئيس رقباء","رقيب أول","رقيب","وكيل رقيب","عريف","جندي أول","جندي"
];

let editIndex=null;

/* تخزين */
function get(){return JSON.parse(localStorage.getItem("data")||"[]")}
function save(d){localStorage.setItem("data",JSON.stringify(d))}

/* init */
window.onload=()=>{
rank.innerHTML=ranks.map(r=>`<option>${r}</option>`).join("");
edit_rank.innerHTML=ranks.map(r=>`<option>${r}</option>`).join("");
render();
};

/* تبويب */
function show(id,btn){
document.querySelectorAll(".section").forEach(s=>s.classList.remove("active"));
document.querySelectorAll(".nav button").forEach(b=>b.classList.remove("active"));
document.getElementById(id).classList.add("active");
btn.classList.add("active");
}

/* ترقية */
function promote(p){
let i=ranks.indexOf(p.rank);
if(i>0)p.rank=ranks[i-1];
p.points=0;
}

/* تنزيل */
function demote(p){
let i=ranks.indexOf(p.rank);
if(i<ranks.length-1)p.rank=ranks[i+1];
p.warn=0;
}

/* إضافة */
function add(){

let d=get();

if(!name.value || !gid.value){
alert("لازم الاسم + Game ID");
return;
}

d.push({
name:name.value.trim(),
gid:gid.value.trim(),
disc:disc.value||"لا يوجد",
rank:rank.value,
unit:unit.value,
points:0,
warn:0,
notes:[]
});

save(d);

name.value="";
gid.value="";
disc.value="";

render();
}

/* ⭐ */
function addPoint(i){
let d=get();
d[i].points++;
if(d[i].points>=3)promote(d[i]);
save(d);
render();
}

/* ⚠ */
function addWarn(i){
let d=get();
d[i].warn++;
if(d[i].warn>=3)demote(d[i]);
save(d);
render();
}

/* تعديل */
function edit(i){
let d=get();
let p=d[i];

editIndex=i;

edit_name.value=p.name;
edit_gid.value=p.gid;
edit_disc.value=p.disc;
edit_rank.value=p.rank;
edit_unit.value=p.unit;

modal.style.display="flex";
}

/* حفظ التعديل */
function saveEdit(){
let d=get();

d[editIndex].name=edit_name.value;
d[editIndex].gid=edit_gid.value;
d[editIndex].disc=edit_disc.value;
d[editIndex].rank=edit_rank.value;
d[editIndex].unit=edit_unit.value; // 🔥 يونت مضبوط

save(d);
closeModal();
render();
}

/* إغلاق */
function closeModal(){
modal.style.display="none";
}

/* عرض */
function render(){

let d=get();

d.sort((a,b)=>ranks.indexOf(a.rank)-ranks.indexOf(b.rank));

army.innerHTML=d.map((p,i)=>`
<div class="card">

<b>${p.name}</b><br>

<div class="tag">🎖 ${p.rank}</div>
<div class="tag">🚓 ${p.unit}</div>
<div class="tag">🆔 ${p.gid}</div>
<div class="tag">💬 ${p.disc}</div>

<div class="tag">⭐ ${p.points}</div>
<div class="tag">⚠ ${p.warn}</div>

<div style="margin-top:5px">
<button class="primary" onclick="addPoint(${i})">⭐</button>
<button class="warn" onclick="addWarn(${i})">⚠</button>
<button class="primary" onclick="edit(${i})">✏</button>
</div>

</div>
`).join("");

points.innerHTML=d.map(p=>`
<div class="card">${p.name} — ⭐ ${p.points}</div>
`).join("");

warns.innerHTML=d.map(p=>`
<div class="card">${p.name} — ⚠ ${p.warn}</div>
`).join("");

notes.innerHTML=d.map(p=>`
<div class="card">${p.name} — 📝 ${p.notes.length}</div>
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

result.innerHTML=`
<div class="card">
<b>${p.name}</b><br>
🎖 ${p.rank}<br>
🚓 ${p.unit}<br>
🆔 ${p.gid}
</div>
`;
}

</script>

</body>
</html>
