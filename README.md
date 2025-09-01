<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8"/>
  <meta name="viewport" content="width=device-width, initial-scale=1"/>
  <title>Rework Browser</title>
  <style>
    /* Layout and Theme (unchanged) */
    body { background: #000; color: #aaa; font-family: Arial, sans-serif; margin: 0; padding: 0; }
    .hidden { display: none; }
    #loginSection, #registerSection, #userInfo, #ownerPanel {
      margin: 20px; padding: 20px; border: 1px solid #333; background: #1a1a1a; border-radius: 8px;
    }
    input, textarea {
      width: 100%; padding: 10px; margin: 10px 0; background: #111; color: #eee; border: 1px solid #444; border-radius: 5px;
    }
    button {
      width: 100%; padding: 10px; margin: 5px 0; background: #222; color: #eee; border: none; border-radius: 5px;
      cursor: pointer; transition: background 0.2s;
    }
    button:hover { background: #444; }
    #tabs { display: flex; flex-wrap: wrap; padding: 0 20px; }
    .tab {
      background: #111; color: #ccc; padding: 10px 15px; margin: 5px; border-radius: 5px;
      cursor: pointer; position: relative; user-select: none; display: flex; align-items: center;
    }
    .tab.active { background: #333; font-weight: bold; color: #fff; }
    .tab:hover { background: #222; }
    .closeTab { margin-left: 8px; color: #999; font-weight: bold; cursor: pointer; }
    .closeTab:hover { color: #fff; }
    #urlInput, #goButton, #newTabBtn { margin: 0 20px; padding: 10px; }
    #goButton, #newTabBtn { width: auto; }
    #iframe { width: 100%; height: 500px; margin-top: 20px; background: #000; border: none; }
    #loadingText { text-align: center; color: #eee; display: none; margin-top: 10px; }
    #chatArea {
      margin: 20px; max-height: 200px; overflow-y: auto; border: 1px solid #333;
      background: #1a1a1a; padding: 10px; border-radius: 5px; color: #ccc; font-size: 14px;
    }
    .chatMessage { margin: 5px 0; word-wrap: break-word; }
    #chatContainer { display: flex; align-items: flex-start; margin: 20px; }
    #chatInput { flex: 1; margin-right: 10px; }
    /* Modal Styles (new, keeps layout intact) */
    #modalOverlay {
      position: fixed; top: 0; left: 0; width: 100%; height: 100%;
      background: rgba(0,0,0,0.75); display: none; justify-content: center; align-items: center; z-index: 9999;
    }
    #modalBox {
      background: #1a1a1a; padding: 20px; border-radius: 8px; color: #eee;
      max-width: 600px; max-height: 70vh; overflow-y: auto; box-shadow: 0 0 15px #222;
    }
    #modalHeader { font-size: 1.2em; margin-bottom: 15px; }
    #modalList { list-style: none; padding: 0; margin: 0 0 15px 0; }
    #modalList li {
      padding: 8px 12px; border-bottom: 1px solid #333; cursor: pointer;
    }
    #modalList li:hover { background: #333; }
    #modalActions { text-align: right; }
    #modalActions button {
      margin-left: 10px; padding: 8px 15px; background: #222; border: none; border-radius: 5px;
      cursor: pointer; transition: background 0.2s;
    }
    #modalActions button:hover { background: #444; }
  </style>
</head>
<body>

  <!-- Your existing layout -->
  <div id="loginSection">
    <h2>Login</h2>
    <input type="text" id="loginUsername" placeholder="Username"/>
    <input type="password" id="loginPassword" placeholder="Password"/>
    <button id="loginBtn">Login</button>
    <p>Don't have an account? <a href="#" id="showRegisterLink">Register</a></p>
  </div>

  <div id="registerSection" class="hidden">
    <h2>Register</h2>
    <input type="text" id="registerUsername" placeholder="Username"/>
    <input type="password" id="registerPassword" placeholder="Password"/>
    <button id="registerBtn">Register</button>
    <p>Already have an account? <a href="#" id="showLoginLink">Login</a></p>
  </div>

  <div id="userInfo" class="hidden">
    <h2>Welcome, <span id="currentUser"></span></h2>
    <button id="logoutBtn">Logout</button>
  </div>

  <div id="ownerPanel" class="hidden">
    <h2>Owner Commands</h2>
    <button id="viewHistoryBtn">View Other People's History</button>
    <button id="fetchMessagesBtn">Fetch Message History</button>
    <button id="sendGlobalMessageBtn">Send Global Message</button>
    <button id="updateScriptBtn">Daily Update Script</button>
    <button id="banUserBtn">Ban User</button>
    <button id="customizeTagsBtn">Customize Name Tags</button>
    <button id="backBtn">Back</button>
  </div>

  <div id="tabs"></div>

  <div style="padding: 20px;">
    <input type="url" id="urlInput" placeholder="Enter Google Sites URL..."/>
    <button id="goButton">Go</button>
    <button id="newTabBtn">New Tab</button>
  </div>

  <div id="loadingText">Loading...</div>
  <iframe id="iframe" src=""></iframe>

  <div id="chatContainer">
    <div id="chatArea"></div>
    <textarea id="chatInput" placeholder="Type a message..." rows="2"></textarea>
    <button id="chatSendBtn">Send Chat</button>
  </div>

  <!-- Modal Overlay (new functionality) -->
  <div id="modalOverlay">
    <div id="modalBox">
      <div id="modalHeader"></div>
      <ul id="modalList"></ul>
      <div id="modalActions">
        <button id="modalCancelBtn">Cancel</button>
        <button id="modalConfirmBtn">Confirm</button>
      </div>
    </div>
  </div>

  <script>
    // Owner login credentials
    const OWNER_USERNAME = "xlcmz", OWNER_PASSWORD = "Santi0224#";
    let users = {}, currentUser = null, tabs = [], activeTabId = null, chatMessages = [];

    // Elements
    const loginSection = document.getElementById("loginSection");
    const registerSection = document.getElementById("registerSection");
    const userInfo = document.getElementById("userInfo");
    const ownerPanel = document.getElementById("ownerPanel");
    const currentUserSpan = document.getElementById("currentUser");
    const loginUsername = document.getElementById("loginUsername");
    const loginPassword = document.getElementById("loginPassword");
    const registerUsername = document.getElementById("registerUsername");
    const registerPassword = document.getElementById("registerPassword");
    const loginBtn = document.getElementById("loginBtn");
    const registerBtn = document.getElementById("registerBtn");
    const logoutBtn = document.getElementById("logoutBtn");
    const showRegisterLink = document.getElementById("showRegisterLink");
    const showLoginLink = document.getElementById("showLoginLink");
    const tabsDiv = document.getElementById("tabs");
    const urlInput = document.getElementById("urlInput");
    const goButton = document.getElementById("goButton");
    const newTabBtn = document.getElementById("newTabBtn");
    const iframeEl = document.getElementById("iframe");
    const loadingText = document.getElementById("loadingText");
    const chatArea = document.getElementById("chatArea");
    const chatInput = document.getElementById("chatInput");
    const chatSendBtn = document.getElementById("chatSendBtn");

    // Modal elements
    const modalOverlay = document.getElementById("modalOverlay");
    const modalHeader = document.getElementById("modalHeader");
    const modalList = document.getElementById("modalList");
    const modalCancelBtn = document.getElementById("modalCancelBtn");
    const modalConfirmBtn = document.getElementById("modalConfirmBtn");

    // Panel toggles
    showRegisterLink.addEventListener("click", e => { e.preventDefault(); loginSection.classList.add("hidden"); registerSection.classList.remove("hidden"); });
    showLoginLink.addEventListener("click", e => { e.preventDefault(); registerSection.classList.add("hidden"); loginSection.classList.remove("hidden"); });

    // Register user
    registerBtn.addEventListener("click", () => {
      const u = registerUsername.value.trim(), p = registerPassword.value;
      if (!u||!p) return alert("Enter username & password.");
      if (u.toLowerCase()===OWNER_USERNAME) return alert("Username reserved.");
      if (users[u]) return alert("Username taken.");
      users[u] = p;
      alert("Registered! Please login.");
      registerSection.classList.add("hidden");
      loginSection.classList.remove("hidden");
      registerUsername.value = registerPassword.value = "";
    });

    // Login
    loginBtn.addEventListener("click", () => {
      const u = loginUsername.value.trim(), p = loginPassword.value;
      if (!u||!p) return alert("Enter username & password.");
      if (u===OWNER_USERNAME && p===OWNER_PASSWORD) {
        currentUser = OWNER_USERNAME; alert("Owner logged in!"); showOwnerPanel(); return;
      }
      if (users[u]===p) {
        currentUser = u; alert("Logged in!"); showUserPanel(); return;
      }
      alert("Invalid credentials.");
    });

    function showOwnerPanel() {
      loginSection.classList.add("hidden");
      registerSection.classList.add("hidden");
      userInfo.classList.add("hidden");
      ownerPanel.classList.remove("hidden");
      renderTabs();
    }

    function showUserPanel() {
      loginSection.classList.add("hidden");
      registerSection.classList.add("hidden");
      ownerPanel.classList.add("hidden");
      userInfo.classList.remove("hidden");
      currentUserSpan.textContent = currentUser;
      renderTabs();
    }

    logoutBtn.addEventListener("click", () => {
      currentUser = null;
      userInfo.classList.add("hidden");
      ownerPanel.classList.add("hidden");
      loginSection.classList.remove("hidden");
      tabs = []; activeTabId = null;
      renderTabs();
      iframeEl.src = "";
      chatArea.innerHTML = "";
      chatMessages = [];
    });

    // Tabs
    function genId() { return "t_" + Math.random().toString(36).substr(2,9); }
    function openNewTab(url) {
      const id = genId(); tabs.push({ id, url }); activeTabId = id;
      urlInput.value = url; loadUrl(url); renderTabs();
    }
    function switchTab(id) { const t = tabs.find(x=>x.id===id); if (t) { activeTabId = id; urlInput.value = t.url; loadUrl(t.url); } renderTabs(); }
    function closeTab(id) {
      tabs = tabs.filter(x=>x.id!==id);
      if (activeTabId===id) {
        if (tabs.length) { activeTabId = tabs[tabs.length-1].id; loadUrl(tabs[tabs.length-1].url); }
        else { activeTabId = null; iframeEl.src=""; urlInput.value=""; }
      }
      renderTabs();
    }
    function renderTabs() {
      tabsDiv.innerHTML = "";
      tabs.forEach(t => {
        const div = document.createElement("div");
        div.className = "tab" + (t.id===activeTabId?" active":"");
        div.title = t.url;
        div.textContent = t.url.length>25 ? t.url.substr(0,22)+"..." : t.url;
        const x = document.createElement("span");
        x.textContent = "×"; x.className = "closeTab";
        x.addEventListener("click", e=>{ e.stopPropagation(); closeTab(t.id); });
        div.append(x);
        div.addEventListener("click", ()=>switchTab(t.id));
        tabsDiv.append(div);
      });
    }

    // Load Google Site only
    function isGoogleSites(u) {
      try{ const X = new URL(u); return X.hostname==="sites.google.com" && (X.pathname.startsWith("/view")||X.pathname.startsWith("/site")||X.pathname.startsWith("/sites")); }
      catch{return false;}
    }

    function loadUrl(input) {
      let u = input.trim();
      if (!u) return alert("Enter a URL.");
      if (!isGoogleSites(u)) return alert("Enter valid Google Sites URL.");
      if (!u.startsWith("http")) u = "https://"+u;
      urlInput.value = u; loadingText.style.display = "block"; iframeEl.src = u;
      if (activeTabId) tabs.find(x=>x.id===activeTabId).url = u;
      renderTabs();
    }

    iframeEl.addEventListener("load", ()=>loadingText.style.display="none");

    goButton.addEventListener("click", ()=>activeTabId? loadUrl(urlInput.value) : openNewTab(urlInput.value));
    urlInput.addEventListener("keydown", e=>{ if(e.key==="Enter") goButton.click(); });
    newTabBtn.addEventListener("click", () => openNewTab("https://sites.google.com/view/"));

    // Chat
    chatSendBtn.addEventListener("click", sendChat);
    chatInput.addEventListener("keydown", e=>{ if(e.key==="Enter"&&!e.shiftKey){ e.preventDefault(); sendChat(); }});
    function sendChat() {
      if (!currentUser) return alert("Log in to chat.");
      const m = chatInput.value.trim(); if (!m) return;
      const msg = { u: currentUser, m, t: new Date().toLocaleTimeString() };
      chatMessages.push(msg);
      const div = document.createElement("div"); div.className="chatMessage";
      div.textContent = `[${msg.t}] ${msg.u}: ${msg.m}`;
      chatArea.append(div);
      chatInput.value = ""; chatArea.scrollTop = chatArea.scrollHeight;
    }

    // Modal
    let sel = null;
    function openModal(title, items, onConfirm) {
      modalHeader.textContent = title;
      modalList.innerHTML = "";
      sel = null;
      items.forEach(it=>{
        const li = document.createElement("li");
        li.textContent = it;
        li.addEventListener("click", ()=>{
          Array.from(modalList.children).forEach(c=> c.style.background="");
          li.style.background="#333";
          sel=it;
        });
        modalList.append(li);
      });
      modalConfirmBtn.disabled = !items.length;
      modalOverlay.style.display="flex";
      modalConfirmBtn.onclick = ()=>{ if(onConfirm) onConfirm(sel); closeModal(); };
      modalCancelBtn.onclick = closeModal;
    }
    function closeModal() { modalOverlay.style.display="none"; sel = null; }

    // Owner commands
    viewHistoryBtn.onclick = ()=>openModal("View Other History", ["alice: site1","bob: site2"], s=>alert("Selected: "+s));
    fetchMessagesBtn.onclick = ()=>openModal("Fetch Message History", ["alice: hi","bob: hello"], s=>alert("Selected: "+s));
    sendGlobalMessageBtn.onclick = ()=>{ const m = prompt("Global msg:"); if(m) alert("Global: "+m); };
    updateScriptBtn.onclick = ()=>alert("Running update script...");
    banUserBtn.onclick = ()=>openModal("Ban User", ["alice","bob","carol"], s=>alert("Banned: "+s));
    customizeTagsBtn.onclick = ()=>openModal("Customize Tags", ["Mod","VIP"], s=>alert("Tag: "+s));
    backBtn.onclick = ()=>{ ownerPanel.classList.add("hidden"); loginSection.classList.remove("hidden"); };

    // Start on login
    loginSection.classList.remove("hidden");
    registerSection.classList.add("hidden");
    userInfo.classList.add("hidden");
    ownerPanel.classList.add("hidden");
  </script>
</body>
</html>
