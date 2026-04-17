<html>
<head>
<meta charset="UTF-8">
<title>Giri Enterprises POS</title>

<!-- Firebase COMPAT SDK (IMPORTANT) -->
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-auth-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore-compat.js"></script>

<style>
body{margin:0;font-family:Arial;background:#eef2f7;}
.topbar{background:#2c3e50;color:#fff;text-align:center;padding:10px;}
.centerBox{display:flex;justify-content:center;align-items:center;height:100vh;flex-direction:column;background:#34495e;color:#fff;}
.card{background:#fff;color:#000;padding:20px;border-radius:10px;width:300px;}
.container{display:none;padding:10px;}
.box{background:#fff;margin:10px;padding:15px;border-radius:10px;}
input,select,button{padding:8px;margin:5px;width:90%;}
button{background:#2980b9;color:#fff;border:none;cursor:pointer;}
table{width:100%;border-collapse:collapse;}
td,th{border:1px solid #ccc;padding:6px;text-align:center;}
</style>
</head>

<body>

<div class="topbar">
🏪 Giri Enterprises | Latur, Maharashtra
</div>

<!-- WELCOME -->
<div id="welcome" class="centerBox">
<h2>Welcome</h2>
<button onclick="showLogin('admin')">Admin Login</button>
<button onclick="showLogin('employee')">Employee Login</button>
</div>

<!-- LOGIN -->
<div id="login" class="centerBox" style="display:none;">
<div class="card">
<h3 id="loginTitle"></h3>
<input id="email" placeholder="Email">
<input id="pass" type="password" placeholder="Password">
<button onclick="login()">Login</button>
<button onclick="back()">Back</button>
</div>
</div>

<!-- APP -->
<div id="app" class="container">

<div id="adminBox" class="box">
<h3>Admin Panel</h3>

<input id="newEmail" placeholder="New Email">
<input id="newPass" placeholder="New Password">
<select id="role">
<option value="employee">Employee</option>
<option value="admin">Admin</option>
</select>

<button onclick="createUser()">Create User</button>

<h4>Users</h4>
<ul id="userList"></ul>
</div>

<div class="box">
<h3>Billing</h3>

<input id="item" placeholder="Item">
<input id="price" placeholder="Price">
<input id="qty" placeholder="Qty">

<button onclick="addItem()">Add</button>

<table id="billTable">
<tr><th>Item</th><th>Price</th><th>Qty</th><th>Total</th></tr>
</table>

<h3>Total: ₹<span id="total">0</span></h3>

<button onclick="saveBill()">Save</button>
<button onclick="printBill()">Print</button>
</div>

</div>

<script>
/* FIREBASE INIT */
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

/* NAV */
function showLogin(type){
  selectedRole = type;
  document.getElementById("welcome").style.display="none";
  document.getElementById("login").style.display="flex";
  document.getElementById("loginTitle").innerText = type.toUpperCase()+" LOGIN";
}

function back(){
  document.getElementById("login").style.display="none";
  document.getElementById("welcome").style.display="flex";
}

/* LOGIN FIXED */
async function login(){
  try{
    let email = document.getElementById("email").value;
    let pass = document.getElementById("pass").value;

    let res = await auth.signInWithEmailAndPassword(email, pass);

    let uid = res.user.uid;

    let doc = await db.collection("users").doc(uid).get();

    if(!doc.exists){
      alert("User not found in Firestore");
      await auth.signOut();
      return;
    }

    let role = doc.data().role;

    if(role !== selectedRole){
      alert("Wrong role selected");
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

  } catch(e){
    console.error(e);
    alert("Login Error: " + e.message);
  }
}

/* CREATE USER */
async function createUser(){
  let email = newEmail.value;
  let pass = newPass.value;
  let role = document.getElementById("role").value;

  let user = await auth.createUserWithEmailAndPassword(email, pass);

  await db.collection("users").doc(user.user.uid).set({
    email: email,
    role: role
  });

  alert("User Created");
  loadUsers();
}

/* LOAD USERS */
async function loadUsers(){
  let snap = await db.collection("users").get();
  let list = document.getElementById("userList");
  list.innerHTML="";

  snap.forEach(d=>{
    list.innerHTML += `<li>${d.data().email} (${d.data().role})</li>`;
  });
}

/* BILL */
function addItem(){
  let t = price.value * qty.value;

  let row = billTable.insertRow();
  row.innerHTML = `<td>${item.value}</td><td>${price.value}</td><td>${qty.value}</td><td>${t}</td>`;

  total += t;
  totalEl.innerText = total;
}

/* SAVE */
async function saveBill(){
  await db.collection("bills").add({
    total,
    time: new Date().toISOString()
  });

  alert("Saved");
}

/* PRINT */
function printBill(){
  let w = window.open();
  w.document.write(`<h2>Giri Enterprises</h2>${billTable.outerHTML}<h3>Total: ₹${total}</h3>`);
  w.print();
}
</script>

</body>
</html>
