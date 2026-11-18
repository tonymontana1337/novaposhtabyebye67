<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Findly</title>

<!-- Подключаем Firebase SDK v10 (Modular через CDN) -->
<script type="module">
  import { initializeApp } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-app.js";
  import { getAnalytics } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-analytics.js";
  import { getAuth, createUserWithEmailAndPassword, signInWithEmailAndPassword, signOut, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-auth.js";
  import { getFirestore, doc, getDoc, setDoc } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-firestore.js";

  // Ваши реальные ключи
  const firebaseConfig = {
    apiKey: "AIzaSyAMyIRbvTpWv0KBec5oA7lsKl-NvCRLdyg",
    authDomain: "findly-6698f.firebaseapp.com",
    projectId: "findly-6698f",
    storageBucket: "findly-6698f.firebasestorage.app",
    messagingSenderId: "689407896121",
    appId: "1:689407896121:web:8239bb2174b0cd0c4f7be8",
    measurementId: "G-J8DYXZDWYL"
  };

  // Инициализация
  const app = initializeApp(firebaseConfig);
  const analytics = getAnalytics(app);
  const auth = getAuth(app);
  const db = getFirestore(app);

  // Делаем их доступными глобально для нашего скрипта чата
  window.firebaseAuth = auth;
  window.firebaseDb = db;
  window.authFns = { createUserWithEmailAndPassword, signInWithEmailAndPassword, signOut, onAuthStateChanged };
  window.dbFns = { doc, getDoc, setDoc };
</script>

<style>
/* ==== ТЕМА ======================================================== */
:root{
  --bg:#ffffff;
  --text:#1d1d1f;
  --muted:#86868b;
  --line:#e8e8ed;
  --soft:#f5f5f7;
  --accent-1:#7c5cff;
  --accent-2:#22d3ee;
  --accent-3:#ff7ab8;
  --radius:22px;
  --font:-apple-system,BlinkMacSystemFont,"SF Pro Display","Inter","Segoe UI",Roboto,Helvetica,Arial,sans-serif;
}

*{box-sizing:border-box;margin:0;padding:0}
html,body{height:100%}
body{
  font-family:var(--font);
  background:var(--bg);
  color:var(--text);
  -webkit-font-smoothing:antialiased;
  overflow:hidden;
}
button{font-family:inherit;cursor:pointer;border:none;background:none;color:inherit}

.glow{
  position:fixed;inset:0;pointer-events:none;z-index:0;
  background:
    radial-gradient(40vmax 40vmax at 20% 15%, rgba(124,92,255,.10), transparent 60%),
    radial-gradient(35vmax 35vmax at 85% 25%, rgba(34,211,238,.10), transparent 60%),
    radial-gradient(40vmax 40vmax at 55% 95%, rgba(255,122,184,.10), transparent 60%);
}

.screen{
  position:fixed;inset:0;z-index:1;
  display:flex;flex-direction:column;
  transition:opacity .6s ease, transform .6s ease, visibility .6s;
}
.screen.hidden{opacity:0;visibility:hidden;pointer-events:none;transform:scale(.98)}

#landing{align-items:center;justify-content:center;text-align:center;padding:24px}
.logo{
  font-size:clamp(56px,11vw,128px);
  font-weight:600;
  letter-spacing:-0.045em;
  line-height:1;
}
.logo span{
  background:linear-gradient(135deg,var(--accent-1),var(--accent-2),var(--accent-3));
  -webkit-background-clip:text;background-clip:text;color:transparent;
}
.tagline{
  margin-top:20px;color:var(--muted);
  font-size:clamp(15px,2vw,21px);font-weight:400;letter-spacing:-0.01em;
}

.cta{
  position:absolute;left:50%;bottom:max(24px,env(safe-area-inset-bottom));
  transform:translateX(-50%);
  padding:16px 40px;border-radius:999px;
  background:var(--text);color:#fff;
  font-size:17px;font-weight:500;letter-spacing:-0.01em;
  transition:transform .25s ease, box-shadow .25s ease;
  box-shadow:0 8px 30px rgba(0,0,0,.12);
}
.cta:hover{transform:translateX(-50%) translateY(-2px);box-shadow:0 14px 40px rgba(0,0,0,.18)}
.cta:active{transform:translateX(-50%) scale(.97)}

#chat{align-items:center}
.chat-top{
  width:100%;padding:18px 24px;display:flex;align-items:center;justify-content:space-between;
  background:rgba(255,255,255,.7);backdrop-filter:blur(20px);-webkit-backdrop-filter:blur(20px);
  border-bottom:1px solid var(--line);
}
.chat-top .brand{font-size:20px;font-weight:600;letter-spacing:-0.03em;cursor:pointer}
.chat-top .right{display:flex;align-items:center;gap:12px}
.counter{color:var(--muted);font-size:13px}
.pill{
  padding:8px 16px;border-radius:999px;border:1px solid var(--line);background:#fff;
  font-size:14px;font-weight:500;max-width:140px;overflow:hidden;text-overflow:ellipsis;white-space:nowrap;
}
.pill:hover{background:var(--soft)}

.messages{
  flex:1;width:100%;max-width:760px;overflow-y:auto;
  padding:32px 20px 16px;display:flex;flex-direction:column;gap:20px;
}

.hero{
  margin:auto;text-align:center;padding-bottom:4vh;
  font-size:clamp(30px,6vw,52px);font-weight:600;letter-spacing:-0.04em;line-height:1.1;
}

.msg{max-width:88%;font-size:16px;line-height:1.5;animation:rise .4s ease both}
.msg.user{align-self:flex-end;background:var(--text);color:#fff;padding:12px 18px;border-radius:20px 20px 6px 20px}
.msg.ai{align-self:flex-start}
@keyframes rise{from{opacity:0;transform:translateY(8px)}to{opacity:1;transform:none}}

.typing{display:inline-flex;gap:5px;padding:6px 0}
.typing i{width:7px;height:7px;border-radius:50%;background:var(--muted);animation:blink 1.2s infinite}
.typing i:nth-child(2){animation-delay:.2s}
.typing i:nth-child(3){animation-delay:.4s}
@keyframes blink{0%,80%,100%{opacity:.25}40%{opacity:1}}

.cards{display:grid;gap:12px;margin-top:12px;width:min(100%,700px)}
.card{
  display:flex;gap:16px;align-items:center;
  padding:16px;border:1px solid var(--line);border-radius:var(--radius);
  background:#fff;text-decoration:none;color:inherit;
  transition:transform .25s ease, box-shadow .25s ease;
}
.card:hover{transform:translateY(-2px);box-shadow:0 12px 32px rgba(0,0,0,.08)}
.thumb{
  flex:none;width:72px;height:72px;border-radius:16px;
  background:linear-gradient(135deg,var(--soft),#fff);
  display:grid;place-items:center;font-size:28px;
}
.card .info{flex:1;min-width:0}
.card .title{font-weight:600;font-size:16px;letter-spacing:-0.01em;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.card .meta{color:var(--muted);font-size:13px;margin-top:3px}
.card .why{font-size:14px;margin-top:6px;line-height:1.4}
.card .side{text-align:right;flex:none}
.card .price{font-weight:600;font-size:18px}
.badge{
  display:inline-block;margin-top:6px;padding:3px 10px;border-radius:999px;
  font-size:12px;font-weight:600;color:#fff;
  background:linear-gradient(135deg,var(--accent-1),var(--accent-2));
}

.composer{width:100%;max-width:760px;padding:12px 20px max(24px,env(safe-area-inset-bottom))}
.composer form{
  position:relative;
  display:flex;align-items:center;gap:8px;
  padding:8px 8px 8px 22px;border-radius:999px;
  background:#fff;border:1px solid var(--line);
  box-shadow:0 8px 30px rgba(0,0,0,.06);
}
.composer input{
  flex:1;border:none;outline:none;background:none;
  font-family:inherit;font-size:16px;color:var(--text);min-width:0;
}
.composer input::placeholder{color:var(--muted)}
.send{width:44px;height:44px;border-radius:50%;background:var(--text);color:#fff;font-size:18px;flex:none}
.send:disabled{opacity:.3;cursor:default}

.ph{
  position:absolute;left:22px;right:64px;top:50%;margin-top:-12px;line-height:24px;
  pointer-events:none;color:var(--muted);font-size:16px;
  white-space:nowrap;overflow:hidden;text-overflow:ellipsis;
}
.ph.in{animation:phIn .5s ease both}
.ph.leave{animation:phOut .3s ease both}
.ph.hide{display:none}
@keyframes phIn{from{opacity:0;transform:translateY(8px)}to{opacity:1;transform:none}}
@keyframes phOut{from{opacity:1;transform:none}to{opacity:0;transform:translateY(-8px)}}

.overlay{
  position:fixed;inset:0;z-index:10;display:grid;place-items:center;padding:20px;
  background:rgba(255,255,255,.55);backdrop-filter:blur(18px);-webkit-backdrop-filter:blur(18px);
  opacity:0;visibility:hidden;transition:opacity .3s, visibility .3s;
}
.overlay.show{opacity:1;visibility:visible}
.modal{
  position:relative;width:min(100%,400px);padding:32px 28px 26px;text-align:center;
  border-radius:28px;background:#fff;border:1px solid var(--line);box-shadow:0 30px 80px rgba(0,0,0,.14);
  transform:translateY(12px) scale(.98);transition:transform .35s ease;
}
.overlay.show .modal{transform:none}
.modal h2{font-size:26px;font-weight:600;letter-spacing:-0.03em}
.modal .sub{color:var(--muted);font-size:15px;margin-top:8px;line-height:1.45}
.modal .x{position:absolute;top:12px;right:16px;font-size:26px;color:var(--muted)}
.field{
  width:100%;margin-top:12px;padding:15px 18px;border-radius:14px;
  border:1px solid var(--line);background:var(--soft);font-family:inherit;font-size:16px;outline:none;
}
.field:focus{border-color:var(--accent-1);background:#fff}
.btn{width:100%;margin-top:16px;padding:15px;border-radius:999px;background:var(--text);color:#fff;font-size:16px;font-weight:500}
.btn.grad{background:linear-gradient(135deg,var(--accent-1),var(--accent-2))}
.btn:disabled{opacity:.5}
.link{margin-top:14px;color:var(--muted);font-size:14px}
.link:hover{color:var(--text)}
.err{color:#d6336c;font-size:13px;margin-top:10px;min-height:16px}
.plan-price{font-size:46px;font-weight:600;letter-spacing:-0.04em;margin-top:18px}
.plan-price small{font-size:16px;color:var(--muted);font-weight:400;letter-spacing:0}
.perks{list-style:none;margin-top:12px;text-align:left;display:inline-block;font-size:15px;line-height:1.9}
.perks li::before{content:"✓";margin-right:10px;color:var(--accent-1);font-weight:700}
.demo{color:var(--muted);font-size:12px;margin-top:12px}

@media (max-width:560px){
  .card{flex-wrap:wrap}
  .card .side{width:100%;text-align:left;display:flex;justify-content:space-between;align-items:center}
  .thumb{width:56px;height:56px;font-size:22px}
}
@media (prefers-reduced-motion:reduce){*{animation:none!important;transition:none!important}}
</style>
</head>

<body>
<div class="glow"></div>

<section id="landing" class="screen">
  <h1 class="logo">Findly<span>.</span></h1>
  <p class="tagline">Tell us what you need. And we will find it.</p>
  <button class="cta" id="startBtn">Try</button>
</section>

<section id="chat" class="screen hidden">
  <header class="chat-top">
    <div class="brand" id="brandBtn" title="Home">Findly.</div>
    <div class="right">
      <span class="counter" id="counter"></span>
      <button class="pill" id="accountBtn">Sign in</button>
    </div>
  </header>

  <div class="messages" id="messages"><div class="hero" id="hero">What are you looking for?</div></div>

  <div class="composer">
    <form id="form" autocomplete="off">
      <input id="input" type="text" aria-label="What are you looking for?" />
      <span class="ph" id="ph"></span>
      <button class="send" id="sendBtn" type="submit" aria-label="Send">↑</button>
    </form>
  </div>
</section>

<div class="overlay" id="authModal">
  <div class="modal">
    <button class="x" data-close="authModal" aria-label="Close">×</button>
    <h2 id="authTitle">Create your account</h2>
    <p class="sub" id="authNote"></p>
    <form id="authForm" novalidate>
      <input class="field" id="authEmail" type="email" placeholder="Email" autocomplete="email">
      <input class="field" id="authPass" type="password" placeholder="Password (8+ characters)" autocomplete="new-password">
      <p class="err" id="authError"></p>
      <button class="btn" id="authSubmit" type="submit">Create account</button>
    </form>
    <button class="link" id="authSwitch">Already have an account? Sign in</button>
  </div>
</div>

<div class="overlay" id="payModal">
  <div class="modal">
    <button class="x" data-close="payModal" aria-label="Close">×</button>
    <h2>Keep finding</h2>
    <p class="sub">You’ve used all your free searches.</p>
    <div class="plan-price"><span id="planPrice"></span> <small>/ month</small></div>
    <ul class="perks">
      <li>Unlimited searches</li>
      <li>Best-value ranking</li>
      <li>New features first</li>
    </ul>
    <button class="btn grad" id="subscribeBtn">Subscribe</button>
    <button class="link" data-close="payModal">Not now</button>
    <p class="demo">Demo mode: payment is not connected yet.</p>
  </div>
</div>

<script>
/* ==== НАСТРОЙКИ САЙТА =============================================== */
const CONFIG = {
  fakeDelayMs: 1400,
  currency: "€",
  placeholders: [
    "Winter jacket",
    "I need boots",
    "find jeans",
    "umm t-shirt"
  ],
  placeholderEveryMs: 3000,
  freeGuest: 3,
  freeWithAccount: 5,
  price: "$2.99",
};

/* ==== ПОИСК ========================================================= */
async function askFindly(query){
  await new Promise(r => setTimeout(r, CONFIG.fakeDelayMs));
  return {
    reply: `Here are the best picks for “${query}”. Sorted by value, not just by price.`,
    items: [
      { title:"Example item #1", price:48, source:"Vinted", score:94, why:"Great condition, top-rated seller, priced 30% below market.", url:"#", emoji:"✨" },
      { title:"Example item #2", price:72, source:"eBay",   score:89, why:"Best build quality in this range, many positive reviews.",    url:"#", emoji:"🏷️" },
      { title:"Example item #3", price:35, source:"OLX",    score:84, why:"Cheapest good option, pickup nearby.",                       url:"#", emoji:"📦" }
    ]
  };
}

const $ = id => document.getElementById(id);
const landing = $("landing"), chat = $("chat");
const messages = $("messages"), form = $("form"), input = $("input"), sendBtn = $("sendBtn");

function showChat(){
  landing.classList.add("hidden");
  chat.classList.remove("hidden");
  setTimeout(() => input.focus(), 500);
}
function showLanding(){
  chat.classList.add("hidden");
  landing.classList.remove("hidden");
}

function esc(s){
  return String(s).replace(/[&<>"']/g, c => ({"&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#39;"}[c]));
}
function scrollDown(){ messages.scrollTop = messages.scrollHeight; }

function addUserMsg(text){
  const el = document.createElement("div");
  el.className = "msg user";
  el.textContent = text;
  messages.appendChild(el);
  scrollDown();
}

function addThinking(){
  const el = document.createElement("div");
  el.className = "msg ai";
  el.innerHTML = '<div class="typing"><i></i><i></i><i></i></div>';
  messages.appendChild(el);
  scrollDown();
  return el;
}

function cardHTML(item){
  return `
    <a class="card" href="${esc(item.url || "#")}" target="_blank" rel="noopener">
      <div class="thumb">${esc(item.emoji || "🛍️")}</div>
      <div class="info">
        <div class="title">${esc(item.title)}</div>
        <div class="meta">${esc(item.source)}</div>
        <div class="why">${esc(item.why)}</div>
      </div>
      <div class="side">
        <div class="price">${CONFIG.currency}${esc(item.price)}</div>
        <span class="badge">${esc(item.score)}</span>
      </div>
    </a>`;
}

function fillAiMsg(el, data){
  const cards = (data.items || []).map(cardHTML).join("");
  el.innerHTML = `<div>${esc(data.reply || "")}</div><div class="cards">${cards}</div>`;
  scrollDown();
}

let busy = false;
async function handleSend(text){
  if (busy || !text.trim()) return;
  busy = true; sendBtn.disabled = true;

  const hero = $("hero");
  if (hero) hero.remove();

  addUserMsg(text);
  const aiEl = addThinking();

  try{
    const data = await askFindly(text);
    fillAiMsg(aiEl, data);
  }catch(err){
    console.error(err);
    aiEl.textContent = "Something went wrong. Please try again.";
  }finally{
    busy = false; sendBtn.disabled = false; input.focus();
  }
}

const ph = $("ph");
let phIndex = 0;
ph.textContent = CONFIG.placeholders[0];
ph.classList.add("in");

function syncPh(){ ph.classList.toggle("hide", input.value.length > 0); }
input.addEventListener("input", syncPh);

setInterval(() => {
  if (document.activeElement === input || input.value) return;
  phIndex = (phIndex + 1) % CONFIG.placeholders.length;
  ph.classList.remove("in"); ph.classList.add("leave");
  setTimeout(() => {
    ph.textContent = CONFIG.placeholders[phIndex];
    ph.classList.remove("leave"); void ph.offsetWidth; ph.classList.add("in");
  }, 320);
}, CONFIG.placeholderEveryMs);

/* ==== СОСТОЯНИЕ И FIRESTORE ======================================== */
const GUEST_KEY = "findly_guest_used";
let state = { used: 0, user: null, plus: false };

try { state.used = parseInt(localStorage.getItem(GUEST_KEY) || "0", 10) || 0; } catch(e){}

async function saveState(){
  if (state.user && window.firebaseDb) {
    const { doc, setDoc } = window.dbFns;
    const docRef = doc(window.firebaseDb, "profiles", state.user.uid);
    await setDoc(docRef, {
      email: state.user.email,
      used: state.used,
      plus: state.plus
    }, { merge: true });
  } else {
    try { localStorage.setItem(GUEST_KEY, String(state.used)); } catch(e){}
  }
}

async function loadUserProfile(authUser){
  if (!authUser) {
    state.user = null;
    state.plus = false;
    state.used = parseInt(localStorage.getItem(GUEST_KEY) || "0", 10) || 0;
    updateUI();
    return;
  }

  state.user = { uid: authUser.uid, email: authUser.email };
  
  if (window.firebaseDb) {
    const { doc, getDoc } = window.dbFns;
    const docRef = doc(window.firebaseDb, "profiles", authUser.uid);
    const docSnap = await getDoc(docRef);
    
    if (docSnap.exists()) {
      const data = docSnap.data();
      const syncedUsed = Math.max(data.used || 0, state.used);
      state.used = syncedUsed;
      state.plus = !!data.plus;
      if (syncedUsed !== data.used) await saveState();
    } else {
      await saveState();
    }
  }
  updateUI();
}

// Ожидаем инициализации Firebase Auth перед привязкой слушателя
const checkAuthInterval = setInterval(() => {
  if (window.firebaseAuth && window.authFns) {
    clearInterval(checkAuthInterval);
    window.authFns.onAuthStateChanged(window.firebaseAuth, user => {
      loadUserProfile(user);
    });
  }
}, 100);

function limit(){ return state.plus ? Infinity : (state.user ? CONFIG.freeWithAccount : CONFIG.freeGuest); }
function left(){ return Math.max(0, limit() - state.used); }
function canSearch(){ return left() > 0; }

function updateUI(){
  $("counter").textContent = state.plus ? "Plus" : left() + " free left";
  $("accountBtn").textContent = state.user ? state.user.email.split("@")[0] : "Sign in";
}

function openModal(id){
  if (id === "authModal") $("authNote").textContent = `Free account = ${CONFIG.freeWithAccount} free searches.`;
  $(id).classList.add("show");
}
function closeModal(id){ $(id).classList.remove("show"); }
document.querySelectorAll("[data-close]").forEach(b => b.addEventListener("click", () => closeModal(b.dataset.close)));
document.querySelectorAll(".overlay").forEach(o => o.addEventListener("click", e => { if (e.target === o) closeModal(o.id); }));

function openGate(){
  if (!state.user && state.used < CONFIG.freeWithAccount) openModal("authModal");
  else openModal("payModal");
}

/* ==== РЕГИСТРАЦИЯ И ВХОД ЧЕРЕЗ FIREBASE AUTH ======================= */
let authMode = "signup";

async function registerUser(email, password, mode){
  const { createUserWithEmailAndPassword, signInWithEmailAndPassword } = window.authFns;
  let res;
  if (mode === "signup") {
    res = await createUserWithEmailAndPassword(window.firebaseAuth, email, password);
  } else {
    res = await signInWithEmailAndPassword(window.firebaseAuth, email, password);
  }
  return res.user;
}

function setAuthMode(m){
  authMode = m;
  $("authTitle").textContent  = m === "signup" ? "Create your account" : "Welcome back";
  $("authSubmit").textContent = m === "signup" ? "Create account" : "Sign in";
  $("authSwitch").textContent = m === "signup" ? "Already have an account? Sign in" : "New here? Create account";
  $("authPass").autocomplete  = m === "signup" ? "new-password" : "current-password";
}

$("authSwitch").addEventListener("click", () => setAuthMode(authMode === "signup" ? "signin" : "signup"));

$("authForm").addEventListener("submit", async e => {
  e.preventDefault();
  const email = $("authEmail").value.trim(), pass = $("authPass").value, err = $("authError");
  if (!/^\S+@\S+\.\S+$/.test(email)) { err.textContent = "Enter a valid email."; return; }
  if (pass.length < 8)              { err.textContent = "Password must be at least 8 characters."; return; }
  err.textContent = ""; $("authSubmit").disabled = true;
  try{
    const user = await registerUser(email, pass, authMode);
    await loadUserProfile(user);
    closeModal("authModal");
    $("authForm").reset();
  }catch(ex){
    err.textContent = ex.message || "Something went wrong.";
  }finally{
    $("authSubmit").disabled = false;
  }
});

$("accountBtn").addEventListener("click", async () => {
  if (state.user) {
    if (confirm("Sign out?")) {
      await window.authFns.signOut(window.firebaseAuth);
      state.user = null;
      state.plus = false;
      state.used = 0;
      localStorage.removeItem(GUEST_KEY);
      updateUI();
    }
  } else openModal("authModal");
});

/* ==== ПОДПИСКА ====================================================== */
async function startCheckout(){
  state.plus = true;
  await saveState();
  updateUI();
  closeModal("payModal");
}
$("subscribeBtn").addEventListener("click", startCheckout);

/* ==== СОБЫТИЯ ======================================================= */
$("startBtn").addEventListener("click", showChat);
$("brandBtn").addEventListener("click", showLanding);

form.addEventListener("submit", async e => {
  e.preventDefault();
  const text = input.value.trim();
  if (!text || busy) return;
  if (!canSearch()) { openGate(); return; }
  input.value = ""; syncPh();
  state.used++;
  updateUI();
  await saveState();
  handleSend(text);
});

$("planPrice").textContent = CONFIG.price;
setAuthMode("signup");
updateUI();
</script>
</body>
</html>
