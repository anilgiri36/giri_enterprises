<html>
<head>
<meta charset="UTF-8">
<title>Giri Enterprises System</title>

<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-auth-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore-compat.js"></script>

<style>
body{margin:0;font-family:Arial;background:#eef2f7}

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
  padding:12px;
  margin:10px;
  width:180px;
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

/* APP */
#app{
  display:none;
  padding:10px;
}
</style>
</head>

<body>

<!-- WELCOME PAGE -->
<div id="welcome">
<h1>Welcome System</h1>

<button onclick="openLogin('admin')">Admin Login</button>
<button onclick="openLogin('user')">User Login</button>
</div>

<!-- LOGIN PAGE -->
<div id="loginBox">
<div class="card">

<h3 id="title">Login</h3>

<input id="email" placeholder="Email"><br><br>
<input id="pass" type="password" placeholder="Password"><br><br>

<button onclick="login()">Login</button>
<button onclick="back()">Back</button>

</div>
</div>

<!-- APP -->
<div id="app">
<h2 id="dashboardTitle"></h2>
<p>Dashboard Loaded Successfully</p>
</div>

<script>

/* FIREBASE INIT */
firebase.initializeApp({
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID"
});

const auth = firebase.auth();
const db = firebase.firestore();

let selectedRole = "";

/* OPEN LOGIN */
function openLogin(type){
  console.log("Login clicked:", type);

  selectedRole = type;

  document.getElementById("welcome").style.display="none";
  document.getElementById("loginBox").style.display="flex";

  document.getElementById("title").innerText =
    type === "admin" ? "Admin Login" : "User Login";
}

/* BACK */
function back(){
  document.getElementById("loginBox").style.display="none";
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
      alert("Wrong role selected");
      await auth.signOut();
      return;
    }

    document.getElementById("loginBox").style.display="none";
    document.getElementById("app").style.display="block";

    document.getElementById("dashboardTitle").innerText =
      role.toUpperCase() + " DASHBOARD";

  } catch(err){
    console.error(err);
    alert(err.message);
  }
}

</script>

</body>
</html>
