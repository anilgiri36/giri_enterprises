<html>
<head>
<meta charset="UTF-8">
<title>Giri Enterprises POS</title>

<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-auth.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js"></script>

<style>
body{
  margin:0;
  font-family:Arial;
  background:#eef2f7;
}

/* TOP BAR */
.topbar{
  background:linear-gradient(90deg,#2c3e50,#34495e);
  color:white;
  text-align:center;
  padding:12px;
  font-weight:bold;
}

/* CENTER SCREEN */
.centerBox{
  display:flex;
  justify-content:center;
  align-items:center;
  height:100vh;
  flex-direction:column;
  background:linear-gradient(135deg,#2c3e50,#34495e);
  color:white;
  text-align:center;
}

/* LOGIN CARD */
.card{
  background:white;
  color:black;
  padding:20px;
  border-radius:12px;
  width:300px;
  box-shadow:0 5px 15px rgba(0,0,0,0.3);
}

/* APP */
.container{display:none;padding:10px}

.box{
  background:white;
  margin:10px;
  padding:15px;
  border-radius:10px;
  box-shadow:0 2px 10px rgba(0,0,0,0.1);
}

input,select,button{
  padding:8px;
  margin:5px;
  width:90%;
}

button{
  background:#2980b9;
  color:white;
  border:none;
  cursor:pointer;
  border-radius:5px;
}

table{
  width:100%;
  border-collapse:collapse;
}

td,th{
  border:1px solid #ccc;
  padding:6px;
  text-align:center;
}
</style>
</head>

<body>

<!-- TOP ADDRESS -->
<div class="topbar">
🏪 Giri Enterprises | 📍 Tq. & Dist: Latur, Maharashtra - 413512
</div>

<!-- WELCOME -->
<div id="welcome" class="centerBox">
<h1>🏪 Giri Enterprises POS</h1>
<p>Select Login Type</p>

<button onclick="showLogin('admin')">👑 Admin Login</button>
<button onclick="showLogin('employee')">👨‍💼 Employee Login</button>
</div>

<!-- LOGIN -->
<div id="login" class="centerBox" style="display:none;">
<div class="card">

<h2 id="loginTitle">Login</h2>

<input id="email" placeholder="Email"><br>

<input id="pass" type="password" placeholder="Password">

<button onclick="togglePass()">👁 Show/Hide</button>

<button onclick="login()">Login</button>
<button onclick="back()">⬅ Back</button>

</div>
</div>

<!-- APP -->
<div id="app" class="container">

<!-- ADMIN -->
<div id="adminBox" class="box">
<h3>👑 Admin Panel</h3>

<input id="newEmail" placeholder="Email">
<input id="newPass" type="password" placeholder="Password">

<select id="role">
<option value="employee">Employee</option>
<option value="admin">Admin</option>
</select>

<button onclick="createUser()">Add User</button>

<h4>Users</h4>
<ul id="userList"></ul>
</div>

<!-- BILLING -->
<div class="box">
<h3>🛒 Billing System</h3>

<input id="item" placeholder="Item">
<input id="price" placeholder="Price">
<input id="qty" placeholder="Qty">

<button onclick="addItem()">Add Item</button>

<table id="billTable">
<tr>
<th>Item</th><th>Price</th><th>Qty</th><th>Total</th>
</tr>
</table>

<h3>Total: ₹<span id="total">0</span></h3>

<button onclick="saveBill()">Save Bill</button>
<button onclick="printBill()">Print</button>
</div>

</div>

<!-- MARQUEE -->
<marquee style="background:#2c3e50;color:white;padding:8px;font-weight:bold;">
© Created by Anil Giri | Giri Enterprises POS System | All Rights Reserved
</marquee>

<script>
/* FIREBASE CONFIG */
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_AUTH_DOMAIN",
  projectId: "YOUR_PROJECT_ID"
};

firebase.initializeApp(firebaseConfig);

const auth = firebase.auth();
const db = firebase.firestore();

let total = 0;
let role = "";
let selectedRole = "";

/* WELCOME */
function showLogin(type){
  selectedRole = type;
  document.getElementById("welcome").style.display="none";
  document.getElementById("login").style.display="flex";

  document.getElementById("loginTitle").innerText =
    type === "admin" ? "👑 Admin Login" : "👨‍💼 Employee Login";
}

/* BACK */
function back(){
  document.getElementById("login").style.display="none";
  document.getElementById("welcome").style.display="flex";
}

/* SHOW PASS */
function togglePass(){
  let p = document.getElementById("pass");
  p.type = (p.type==="password")?"text":"password";
}

/* LOGIN */
async function login(){

  let res = await auth.signInWithEmailAndPassword(email.value, pass.value);

  let uid = res.user.uid;
  let doc = await db.collection("users").doc(uid).get();

  role = doc.data().role;

  if(role !== selectedRole){
    alert("❌ Not allowed as " + selectedRole);
    await auth.signOut();
    return;
  }

  document.getElementById("login").style.display="none";
  document.getElementById("app").style.display="block";

  if(role !== "admin"){
    document.getElementById("adminBox").style.display="none";
  } else {
    loadUsers();
  }
}

/* CREATE USER */
async function createUser(){

  let snap = await db.collection("users").get();
  let emp=0, adminExists=false;

  snap.forEach(d=>{
    if(d.data().role==="employee") emp++;
    if(d.data().role==="admin") adminExists=true;
  });

  if(role.value==="employee" && emp>=20){
    alert("Max 20 employees");
    return;
  }

  if(role.value==="admin" && adminExists){
    alert("Only 1 admin allowed");
    return;
  }

  let u = await auth.createUserWithEmailAndPassword(newEmail.value,newPass.value);

  await db.collection("users").doc(u.user.uid).set({
    email:newEmail.value,
    role:role.value
  });

  loadUsers();
}

/* LOAD USERS */
async function loadUsers(){
  let snap = await db.collection("users").get();
  userList.innerHTML="";

  snap.forEach(d=>{
    userList.innerHTML += `<li>${d.data().email} (${d.data().role})</li>`;
  });
}

/* BILL */
function addItem(){
  let t = price.value * qty.value;

  let row = billTable.insertRow();
  row.innerHTML = `
    <td>${item.value}</td>
    <td>${price.value}</td>
    <td>${qty.value}</td>
    <td>${t}</td>
  `;

  total += t;
  totalEl.innerText = total;
}

/* SAVE BILL */
async function saveBill(){
  await db.collection("bills").add({
    total,
    createdAt: new Date().toISOString()
  });

  alert("Saved ✔");
}

/* PRINT */
function printBill(){
  let w = window.open();
  w.document.write(`
    <h2>Giri Enterprises</h2>
    <p>📍 Latur, Maharashtra - 413512</p>
    <hr>
    ${billTable.outerHTML}
    <h3>Total: ₹${total}</h3>
  `);
  w.print();
}
</script>

</body>
</html>
