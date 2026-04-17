<html>
<head>
<meta charset="UTF-8">
<title>Role Based POS System</title>

<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-auth-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore-compat.js"></script>

<style>
body{margin:0;font-family:Arial;background:#f2f6ff}

/* WELCOME */
#welcome{
  height:100vh;
  display:flex;
  justify-content:center;
  align-items:center;
  flex-direction:column;
  background:#2c3e50;
  color:white;
}

button{
  padding:10px;
  margin:10px;
  width:200px;
  cursor:pointer;
}

/* LOGIN */
#loginBox{
  display:none;
  height:100vh;
  justify-content:center;
  align-items:center;
  flex-direction:column;
  background:#34495e;
  color:white;
}

.card{
  background:white;
  color:black;
  padding:20px;
  border-radius:10px;
  width:280px;
}

/* DASHBOARD */
#app{
  display:none;
  padding:10px;
}

.section{
  background:white;
  margin:10px;
  padding:15px;
  border-radius:10px;
}

input,select{
  padding:8px;
  margin:5px;
  width:90%;
}

.btn{
  background:#2980b9;
  color:white;
  border:none;
}
.delete{background:red}
.update{background:orange}
</style>
</head>

<body>

<!-- WELCOME -->
<div id="welcome">
<h1>Welcome System</h1>
<button onclick="openLogin('admin')">Admin Login</button>
<button onclick="openLogin('user')">User Login</button>
</div>

<!-- LOGIN -->
<div id="loginBox">
<div class="card">
<h3 id="title"></h3>

<input id="email" placeholder="Email">
<input id="pass" type="password" placeholder="Password">

<button onclick="login()">Login</button>
<button onclick="back()">Back</button>
</div>
</div>

<!-- DASHBOARD -->
<div id="app">

<!-- ADMIN PANEL -->
<div id="adminPanel" class="section">
<h2>Admin Panel</h2>

<input id="newEmail" placeholder="User Email">
<input id="newPass" placeholder="Password">

<select id="role">
<option value="user">User</option>
<option value="admin">Admin</option>
</select>

<button class="btn" onclick="createUser()">Create User</button>

<h3>Users</h3>
<ul id="userList"></ul>
</div>

<!-- USER AREA -->
<div class="section">
<h2>User Area</h2>
<p>Limited access dashboard</p>
</div>

</div>

<script>

/* FIREBASE CONFIG */
const firebaseConfig = {
  apiKey: "AIzaSyBaTFSvArWkwktLcf71pm-lGLc7TmV56dU",
  authDomain: "giri-enterprises-pos.firebaseapp.com",
  databaseURL: "https://giri-enterprises-pos-default-rtdb.firebaseio.com",
  projectId: "giri-enterprises-pos",
  storageBucket: "giri-enterprises-pos.firebasestorage.app",
  messagingSenderId: "287416000311",
  appId: "1:287416000311:web:353a28d193e6739aa384b2",
  measurementId: "G-NX7N5JDKP6"
};

// Initialize Firebase
const app = initializeApp(firebaseConfig);
const analytics = getAnalytics(app);

let selectedRole = "";

/* NAVIGATION */
function openLogin(type){
  selectedRole = type;
  welcome.style.display="none";
  loginBox.style.display="flex";
  title.innerText = type.toUpperCase()+" LOGIN";
}

function back(){
  loginBox.style.display="none";
  welcome.style.display="flex";
}

/* LOGIN */
async function login(){
  try{

    let res = await auth.signInWithEmailAndPassword(email.value, pass.value);
    let uid = res.user.uid;

    let doc = await db.collection("users").doc(uid).get();

    if(!doc.exists){
      alert("User not found in database");
      return;
    }

    let role = doc.data().role;

    if(role !== selectedRole){
      alert("Wrong role selected");
      await auth.signOut();
      return;
    }

    loginBox.style.display="none";
    app.style.display="block";

    if(role !== "admin"){
      adminPanel.style.display="none";
    } else {
      loadUsers();
    }

  }catch(e){
    alert(e.message);
  }
}

/* CREATE USER (ADMIN ONLY) */
async function createUser(){

  let u = await auth.createUserWithEmailAndPassword(newEmail.value, newPass.value);

  await db.collection("users").doc(u.user.uid).set({
    email: newEmail.value,
    role: role.value
  });

  alert("User Created");
  loadUsers();
}

/* LOAD USERS */
async function loadUsers(){
  let snap = await db.collection("users").get();

  userList.innerHTML="";

  snap.forEach(d=>{
    userList.innerHTML += `
      <li>
        ${d.data().email} (${d.data().role})
      </li>`;
  });
}

</script>

</body>
</html>
