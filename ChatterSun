<!DOCTYPE html>
<html lang="uk">
<head>
  <meta charset="UTF-8" />
  <title>ChatterSun — мій месенджер</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />

  <!-- Firebase -->
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-auth-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-storage-compat.js"></script>

  <style>
    body { margin:0; font-family:Arial; background:#f2f2f2; }
    body.dark { background:#121212; color:#ddd; }
    header { background:#0a84ff; color:white; padding:15px; text-align:center; }
    #app { max-width:420px; margin:auto; background:white; min-height:100vh; }
    #app.dark { background:#1e1e1e; }
    .screen { display:none; padding:15px; }
    input, button { width:100%; padding:10px; margin:6px 0; border-radius:8px; border:1px solid #ccc; }
    button { background:#0a84ff; color:white; border:none }
    .chat { height:350px; overflow-y:auto; border:1px solid #ddd; padding:10px; margin-bottom:10px }
    .msg { margin:5px 0 }
    .me { text-align:right; color:#0a84ff }
    body.dark .me { color:#4fc3f7; }
    img.avatar { width:60px; height:60px; border-radius:50%; display:block; margin:auto }
    .contact { padding:10px; border-bottom:1px solid #ddd; cursor:pointer }
    body.dark .contact { border-bottom:1px solid #555; }
  </style>
</head>
<body>

<header>
  <h2>MyChat</h2>
  <small>Мій власний месенджер</small>
</header>

<div id="app">

  <!-- AUTH -->
  <div class="screen" id="auth">
    <h3>Вхід / Реєстрація</h3>
    <input id="email" placeholder="Email" />
    <input id="password" type="password" placeholder="Пароль" />
    <button onclick="register()">Зареєструватись</button>
    <button onclick="login()">Увійти</button>
  </div>

  <!-- PROFILE -->
  <div class="screen" id="profile">
    <h3>Мій профіль</h3>
    <img id="avatarImg" class="avatar" />
    <input type="file" id="avatar" />
    <input id="name" placeholder="Імʼя" />
    <button onclick="saveProfile()">Зберегти</button>
    <button onclick="openContacts()">Контакти</button>
    <button onclick="toggleTheme()">🌙 Темна тема</button>
    <button onclick="openAdmin()">🧑‍💼 Адмін-панель</button>
  </div>

  <!-- CONTACTS -->
  <div class="screen" id="contacts">
    <h3>Контакти</h3>
    <div id="contactList"></div>
    <button onclick="logout()">Вийти</button>
  </div>

  <!-- CHAT -->
  <div class="screen" id="chat">
    <h3 id="chatWith"></h3>
    <div class="chat" id="messages"></div>
    <input id="msg" placeholder="Повідомлення" />
    <input type="file" id="fileUpload" />
    <button onclick="sendMsg()">Надіслати текст</button>
    <button onclick="sendFile()">Надіслати файл / фото / відео</button>
    <button onclick="recordVoice()">🎤 Голосове</button>
    <button onclick="backContacts()">Назад</button>
    <button onclick="startCall()">📞 Дзвінок</button>
  </div>

  <!-- ADMIN -->
  <div class="screen" id="admin">
    <h3>Адмін-панель</h3>
    <div id="usersList"></div>
    <button onclick="backProfile()">Назад</button>
  </div>

</div>

<script>
  const firebaseConfig = { apiKey:"YOUR_API_KEY", authDomain:"YOUR_PROJECT.firebaseapp.com", projectId:"YOUR_PROJECT", storageBucket:"YOUR_PROJECT.appspot.com" };
  firebase.initializeApp(firebaseConfig);
  const auth = firebase.auth();
  const db = firebase.firestore();
  const storage = firebase.storage();

  let currentChat = null;
  let darkMode = false;

  function show(id){ document.querySelectorAll('.screen').forEach(s=>s.style.display='none'); document.getElementById(id).style.display='block'; }
  show('auth');

  function register(){ auth.createUserWithEmailAndPassword(email.value,password.value).catch(alert); }
  function login(){ auth.signInWithEmailAndPassword(email.value,password.value).catch(alert); }
  function logout(){ auth.signOut(); show('auth'); }

  auth.onAuthStateChanged(async u=>{
    if(!u) return;
    const doc = await db.collection('users').doc(u.uid).get();
    if(!doc.exists) show('profile'); else loadContacts();
  });

  function saveProfile(){
    const file = avatar.files[0];
    const nameVal = name.value;
    if(file){
      const ref = storage.ref('avatars/'+auth.currentUser.uid);
      ref.put(file).then(()=>ref.getDownloadURL().then(url=>{
        db.collection('users').doc(auth.currentUser.uid).set({name:nameVal, avatar:url});
        loadContacts();
      }));
    } else {
      db.collection('users').doc(auth.currentUser.uid).set({name:nameVal});
      loadContacts();
    }
  }

  function loadContacts(){
    show('contacts');
    contactList.innerHTML='';
    db.collection('users').get().then(snap=>{
      snap.forEach(d=>{
        if(d.id!==auth.currentUser.uid){
          const div=document.createElement('div');
          div.className='contact';
          div.textContent=d.data().name;
          div.onclick=()=>openChat(d.id,d.data().name);
          contactList.appendChild(div);
        }
      });
    });
  }

  function openChat(uid,name){
    currentChat = [auth.currentUser.uid, uid].sort().join('_');
    chatWith.textContent=name;
    show('chat');
    db.collection('chats').doc(currentChat).collection('msgs').orderBy('t')
      .onSnapshot(s=>{
        messages.innerHTML='';
        s.forEach(d=>{
          const m=d.data();
          const div=document.createElement('div');
          div.className='msg '+(m.u===auth.currentUser.uid?'me':'');
          if(m.type==='text') div.textContent=m.text;
          if(m.type==='file') div.innerHTML=`<a href="${m.url}" target="_blank">📁 ${m.name}</a>`;
          if(m.type==='voice') div.innerHTML=`<audio controls src="${m.url}"></audio>`;
          messages.appendChild(div);
        });
      });
  }

  function sendMsg(){ db.collection('chats').doc(currentChat).collection('msgs').add({u:auth.currentUser.uid,text:msg.value,type:'text',t:Date.now()}); msg.value=''; }

  function sendFile(){
    const f = fileUpload.files[0];
    if(!f) return;
    const ref = storage.ref('files/'+Date.now()+'_'+f.name);
    ref.put(f).then(()=>ref.getDownloadURL().then(url=>{
      db.collection('chats').doc(currentChat).collection('msgs').add({u:auth.currentUser.uid,type:'file',url:url,name:f.name,t:Date.now()});
    }));
  }

  let mediaRecorder;
  function recordVoice(){
    navigator.mediaDevices.getUserMedia({ audio: true }).then(stream => {
      mediaRecorder = new MediaRecorder(stream);
      let chunks = [];
      mediaRecorder.ondataavailable = e => chunks.push(e.data);
      mediaRecorder.onstop = () => {
        const blob = new Blob(chunks,{type:'audio/webm'});
        const ref = storage.ref('voices/'+Date.now()+'.webm');
        ref.put(blob).then(()=>ref.getDownloadURL().then(url=>{
          db.collection('chats').doc(currentChat).collection('msgs').add({u:auth.currentUser.uid,type:'voice',url:url,t:Date.now()});
        }));
      };
      mediaRecorder.start();
      setTimeout(()=>mediaRecorder.stop(),3000);
    });
  }

  function startCall(){ alert('WebRTC дзвінки можна додати окремо, як у ЕТАП 3'); }

  function backContacts(){ show('contacts'); }
  function toggleTheme(){ darkMode=!darkMode; document.body.classList.toggle('dark',darkMode); document.getElementById('app').classList.toggle('dark',darkMode); }
  function openAdmin(){
    show('admin'); usersList.innerHTML='';
    db.collection('users').get().then(snap=>{ snap.forEach(d=>{
      const div=document.createElement('div'); div.textContent=d.data().name; usersList.appendChild(div);
    });});
  }
  function backProfile(){ show('profile'); }
</script>

</body>
</html>
