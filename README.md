<html>
<head>
<meta charset="UTF-8">
<title>Giri Enterprises POS</title>

<!-- FIREBASE v8 (IMPORTANT) -->
<script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-auth.js"></script>
<script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-firestore.js"></script>

<style>
body{margin:0;font-family:Arial;background:#eef2f7;}
.topbar{background:#2c3e50;color:white;text-align:center;padding:10px;}
.centerBox{display:flex;justify-content:center;align-items:center;height:100vh;flex-direction:column;background:#34495e;color:white;}
.card{background:white;color:black;padding:20px;border-radius:10px;width:300px;}
.container{display:none;padding:10px;}
.box{background:white;margin:10px;padding:15px;border-radius:10px;}
input,select,button{padding:8px;margin:5px;width:90%;}
button{background:#2980b9;color:white;border:none;cursor:pointer;}
table{width:100%;border-collapse:collapse;}
td,th{border:1px solid #ccc;padding:6px;text-align:center;}
</style>
</head>

<body>

<div class="topbar">
🏪 Giri Enterprises | 📍 Tq. & Dist: Latur, Maharashtra - 413512
</div>

<!-- WELCOME -->
<div id="welcome" class="centerBox">
<h2>Welcome to Giri Enterprises POS</h2>
<button onclick="showLogin('admin')">👑 Admin Login</button>
<button onclick="showLogin('employee')">👨‍💼 Employee Login</button>
</div>

<!-- LOGIN -->
<div id="login" class="centerBox" style="display:none;">
<div class="card">
<h3 id="loginTitle"></h3>
<input id="email" placeholder="Email">
<input id="pass" type="password" placeholder="Password">
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
<input id="newPass" placeholder="Password">

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
<tr><th>Item</th><th>Price</th><th>Qty</th><th>Total</th></tr>
</table>

<h3>Total: ₹<span id="total">0</span></h3>

<button onclick="saveBill()">Save Bill</button>
<button onclick="printBill()">Print</button>
</div>

</div>

<marquee style="background:#2c3e50;color:white;padding:8px;">
© Created by Anil Giri
</marquee>

<script>

/* FIREBASE CONFIG (FIXED) */
firebase.initializeApp({
  apiKey: "AIzaSyBaTFSvArWkwktLcf71pm-lGLc7TmV56dU",
  authDomain: "giri-enterprises-pos.firebaseapp.com",
  projectId: "giri-enterprises-pos",
  storageBucket: "giri-enterprises-pos.firebasestorage.app",
  messagingSenderId: "287416000311",
  appId: "1:287416000311:web:353a28d193e6739aa384b2"
});

const auth = firebase.auth();
const db = firebase.firestore();

let total = 0;
let selectedRole = "";

/* NAVIGATION */
function showLogin(type){
  selectedRole = type;
  document.getElementById("welcome").style.display="none";
  document.getElementById("login").style.display="flex";
  document.getElementById("loginTitle").innerText =
    type === "admin" ? "👑 Admin Login" : "👨‍💼 Employee Login";
}

function back(){
  document.getElementById("login").style.display="none";
  document.getElementById("welcome").style.display="flex";
}

/* LOGIN */
async function login(){
  try{
    let email = document.getElementById("email").value;
    let pass = document.getElementById("pass").value;

    let res = await auth.signInWithEmailAndPassword(email, pass);
    let uid = res.user.uid;

    let doc = await db.collection("users").doc(uid).get();

    if(!doc.exists){
      alert("User not found in database");
      return;
    }

    let role = doc.data().role;

    if(role !== selectedRole){
      alert("❌ Wrong role selected");
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

  }catch(e){
    alert(e.message);
  }
}

/* CREATE USER */
async function createUser(){
  let email = document.getElementById("newEmail").value;
  let pass = document.getElementById("newPass").value;
  let role = document.getElementById("role").value;

  let u = await auth.createUserWithEmailAndPassword(email, pass);

  await db.collection("users").doc(u.user.uid).set({
    email: email,
    role: role
  });

  alert("User Created ✔");
  loadUsers();
}

/* LOAD USERS */
async function loadUsers(){
  let snap = await db.collection("users").get();
  let list = document.getElementById("userList");
  list.innerHTML = "";

  snap.forEach(d=>{
    list.innerHTML += `<li>${d.data().email} (${d.data().role})</li>`;
  });
}

/* BILLING */
function addItem(){
  let item = document.getElementById("item").value;
  let price = Number(document.getElementById("price").value);
  let qty = Number(document.getElementById("qty").value);

  let t = price * qty;

  let row = document.getElementById("billTable").insertRow();
  row.innerHTML = `<td>${item}</td><td>${price}</td><td>${qty}</td><td>${t}</td>`;

  total += t;
  document.getElementById("total").innerText = total;
}

/* SAVE BILL */
async function saveBill(){
  await db.collection("bills").add({
    total: total,
    date: new Date().toISOString()
  });

  alert("Bill Saved ✔");
}

/* PRINT */
function printBill(){
  let w = window.open();
  w.document.write(`
    <h2>Giri Enterprises</h2>
    <p>Latur, Maharashtra - 413512</p>
    <hr>
    ${document.getElementById("billTable").outerHTML}
    <h3>Total: ₹${total}</h3>
  `);
  w.print();
}

</script>

</body>
</html>
