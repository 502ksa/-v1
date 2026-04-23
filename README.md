# وزارة الداخلية
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>وزارة الداخلية - Velora RP</title>

<style>

body{
margin:0;
font-family:Tahoma;
background:#0b0f17;
color:white;
}

header{
background:#111827;
padding:18px;
text-align:center;
border-bottom:2px solid #2563eb;
}

.nav{
display:flex;
gap:10px;
justify-content:center;
padding:10px;
background:#0f172a;
}

.nav button{
flex:1;
padding:10px;
border:none;
border-radius:8px;
background:#1f2937;
color:white;
cursor:pointer;
}

.nav button.active{
background:#2563eb;
}

.container{
max-width:900px;
margin:auto;
padding:15px;
}

.card{
background:#161b22;
padding:15px;
margin:10px 0;
border-radius:10px;
border:1px solid #222c3a;
}

input,textarea{
width:100%;
padding:10px;
margin:6px 0;
border:none;
border-radius:8px;
background:#0a0e14;
color:white;
}

button{
padding:10px;
margin-top:6px;
border:none;
border-radius:8px;
background:#2563eb;
color:white;
cursor:pointer;
}

.hidden{display:none;}

.tag{
display:inline-block;
padding:4px 8px;
background:#1f2937;
border-radius:6px;
margin:2px;
font-size:12px;
}

.good{color:#22c55e;}
.bad{color:#ef4444;}
.warn{color:#fbbf24;}

</style>
</head>

<body>

<header>
🏛️ وزارة الداخلية - Velora RP
</header>

<!-- NAV -->
<div class="nav">
<button onclick="show('apps')" class="active">📄 الطلبات</button>
<button onclick="show('warns')">⚠ التحذيرات</button>
<button onclick="show('notes')">📝 الملاحظات</button>
</div>

<div class="container">

<!-- تسجيل -->
<div class="card">
<h3>🔐 دخول المسؤول</h3>
<button onclick="login()">دخول</button>
</div>

<!-- تقديم -->
<div class="card">
<h3>📄 تقديم</h3>

<input id="name" placeholder="الاسم">
<input id="age" placeholder="العمر">
<input id="discord" placeholder="Discord">
<input id="game" placeholder="Game ID">
<textarea id="exp" placeholder="الخبرات"></textarea>

<button onclick="send()">إرسال</button>
<p id="msg"></p>
</div>

<!-- الطلبات -->
<div id="appsPage"></div>

<!-- التحذيرات -->
<div id="warnPage" class="hidden"></div>

<!-- الملاحظات -->
<div id="notePage" class="hidden"></div>

</div>

<script>

let logged=false;
let page="apps";

/* تنقل */
function show(p){

page=p;

document.getElementById("appsPage").classList.add("hidden");
document.getElementById("warnPage").classList.add("hidden");
document.getElementById("notePage").classList.add("hidden");

if(p=="apps") document.getElementById("appsPage").classList.remove("hidden");
if(p=="warns") document.getElementById("warnPage").classList.remove("hidden");
if(p=="notes") document.getElementById("notePage").classList.remove("hidden");

document.querySelectorAll(".nav button").forEach(b=>b.classList.remove("active"));
event.target.classList.add("active");

render();

}

/* دخول */
function login(){

let pass=prompt("0008");

if(pass!="0008"){
alert("خطأ");
return;
}

logged=true;
render();

}

/* إرسال */
function send(){

let data={
name:name.value,
age:age.value,
discord:discord.value,
game:game.value,
exp:exp.value,
status:"pending",
warnings:0,
notes:[]
};

if(!data.name || !data.age || !data.discord || !data.game || !data.exp){
alert("عبي كل البيانات");
return;
}

let arr=JSON.parse(localStorage.getItem("apps")||"[]");
arr.push(data);
localStorage.setItem("apps",JSON.stringify(arr));

msg.innerText="✔ تم الإرسال";

render();

}

/* عرض */
function render(){

let arr=JSON.parse(localStorage.getItem("apps")||"[]");

/* الطلبات */
let appsHtml="";

arr.forEach((a,i)=>{
appsHtml+=`
<div class="card">
<h3>${a.name}</h3>
<p>${a.game}</p>
<p>${a.discord}</p>

<p class="${a.status=='accepted'?'good':a.status=='rejected'?'bad':'warn'}">
${a.status}
</p>

<button onclick="accept(${i})">قبول</button>
<button onclick="reject(${i})">رفض</button>
<button onclick="warn(${i})">تحذير</button>
<button onclick="note(${i})">ملاحظة</button>
</div>
`;
});

appsPage.innerHTML=appsHtml;

/* التحذيرات */
let warnHtml="";
arr.forEach(a=>{
warnHtml+=`
<div class="card">
<h3>${a.name}</h3>
<p>⚠ التحذيرات: ${a.warnings}</p>
</div>
`;
});
warnPage.innerHTML=warnHtml;

/* الملاحظات */
let noteHtml="";
arr.forEach(a=>{
noteHtml+=`
<div class="card">
<h3>${a.name}</h3>
${a.notes.map(n=>`<div class="tag">📝 ${n}</div>`).join("")}
</div>
`;
});
notePage.innerHTML=noteHtml;

}

/* وظائف */
function accept(i){
let arr=JSON.parse(localStorage.getItem("apps"));
arr[i].status="accepted";
localStorage.setItem("apps",JSON.stringify(arr));
render();
}

function reject(i){
let arr=JSON.parse(localStorage.getItem("apps"));
arr[i].status="rejected";
localStorage.setItem("apps",JSON.stringify(arr));
render();
}

function warn(i){
let arr=JSON.parse(localStorage.getItem("apps"));
arr[i].warnings++;
localStorage.setItem("apps",JSON.stringify(arr));
render();
}

function note(i){
let t=prompt("اكتب ملاحظة");
if(!t) return;

let arr=JSON.parse(localStorage.getItem("apps"));
arr[i].notes.push(t);

localStorage.setItem("apps",JSON.stringify(arr));
render();
}

</script>

</body>
</html>
