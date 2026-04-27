<!DOCTYPE html>

<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Whispr — Anonymous Messages</title>
<link href="https://fonts.googleapis.com/css2?family=Playfair+Display:ital,wght@0,400;0,700;1,400&family=DM+Sans:wght@300;400;500&display=swap" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
<style>
  :root {
    --ink: #0e0b0b;
    --cream: #f5f0e8;
    --blush: #e8c4b0;
    --rose: #c4614a;
    --muted: #8a7d72;
    --card: #faf7f2;
    --green: #4caf80;
  }
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    background: var(--cream);
    color: var(--ink);
    font-family: 'DM Sans', sans-serif;
    min-height: 100vh;
    overflow-x: hidden;
  }
  body::before {
    content: '';
    position: fixed;
    inset: 0;
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='300' height='300'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='300' height='300' filter='url(%23noise)' opacity='0.035'/%3E%3C/svg%3E");
    pointer-events: none;
    z-index: 0;
  }

nav {
display: flex;
justify-content: space-between;
align-items: center;
padding: 1.4rem 2.5rem;
border-bottom: 1px solid rgba(0,0,0,0.08);
position: relative;
z-index: 10;
}
.logo { font-family: ‘Playfair Display’, serif; font-size: 1.6rem; letter-spacing: -0.5px; }
.logo span { color: var(–rose); font-style: italic; }
.nav-right { display: flex; gap: 0.8rem; align-items: center; }
.nav-btn {
background: var(–ink); color: var(–cream); border: none;
padding: 0.55rem 1.3rem; font-family: ‘DM Sans’, sans-serif;
font-size: 0.85rem; cursor: pointer; border-radius: 100px;
transition: background 0.2s;
}
.nav-btn:hover { background: var(–rose); }
.nav-btn.outline {
background: transparent; color: var(–ink);
border: 1.5px solid rgba(0,0,0,0.2);
}
.nav-btn.outline:hover { border-color: var(–rose); color: var(–rose); background: transparent; }

/* PAGES */
.page { display: none; position: relative; z-index: 1; }
.page.active { display: block; }

/* LANDING */
.hero {
display: grid; grid-template-columns: 1fr 1fr;
gap: 3rem; max-width: 1100px; margin: 0 auto;
padding: 5rem 2.5rem 3rem; align-items: center;
}
.hero-text h1 {
font-family: ‘Playfair Display’, serif;
font-size: clamp(2.6rem, 5vw, 4rem);
line-height: 1.12; margin-bottom: 1.2rem; letter-spacing: -1px;
}
.hero-text h1 em { font-style: italic; color: var(–rose); }
.hero-text p { color: var(–muted); font-size: 1rem; line-height: 1.7; max-width: 400px; margin-bottom: 2rem; font-weight: 300; }
.hero-actions { display: flex; gap: 0.8rem; flex-wrap: wrap; }
.btn-primary {
background: var(–rose); color: white; border: none;
padding: 0.8rem 2rem; font-family: ‘DM Sans’, sans-serif;
font-size: 0.95rem; cursor: pointer; border-radius: 100px;
font-weight: 500; transition: transform 0.15s, box-shadow 0.15s;
box-shadow: 0 4px 20px rgba(196,97,74,0.3);
}
.btn-primary:hover { transform: translateY(-2px); box-shadow: 0 8px 28px rgba(196,97,74,0.35); }

.hero-visual { position: relative; height: 380px; }
.letter-card {
position: absolute; background: var(–card);
border: 1px solid rgba(0,0,0,0.08); border-radius: 12px;
padding: 1.2rem 1.5rem; box-shadow: 0 8px 40px rgba(0,0,0,0.08);
font-size: 0.88rem; line-height: 1.6; max-width: 230px;
animation: float 6s ease-in-out infinite;
}
.letter-card .sender { font-size: 0.72rem; color: var(–muted); margin-top: 0.7rem; font-style: italic; }
.letter-card:nth-child(1) { top: 20px; left: 10px; animation-delay: 0s; transform: rotate(-2deg); }
.letter-card:nth-child(2) { top: 80px; right: 0; animation-delay: 1.5s; transform: rotate(1.5deg); }
.letter-card:nth-child(3) { bottom: 30px; left: 50px; animation-delay: 3s; transform: rotate(-1deg); }
.letter-card .tag {
display: inline-block; background: var(–blush); color: var(–rose);
font-size: 0.65rem; padding: 0.2rem 0.6rem; border-radius: 100px;
margin-bottom: 0.6rem; font-weight: 500; letter-spacing: 0.5px; text-transform: uppercase;
}
@keyframes float {
0%, 100% { transform: translateY(0) rotate(var(–r, -2deg)); }
50% { transform: translateY(-10px) rotate(var(–r, -2deg)); }
}

/* AUTH */
.auth-container {
max-width: 420px; margin: 4rem auto; padding: 0 1.5rem;
}
.auth-box {
background: var(–card); border: 1px solid rgba(0,0,0,0.08);
border-radius: 20px; padding: 2.5rem;
box-shadow: 0 8px 40px rgba(0,0,0,0.06);
}
.auth-box h2 { font-family: ‘Playfair Display’, serif; font-size: 1.8rem; margin-bottom: 0.4rem; }
.auth-box p { color: var(–muted); font-size: 0.88rem; margin-bottom: 2rem; }
.input-field {
width: 100%; background: var(–cream);
border: 1.5px solid rgba(0,0,0,0.1); border-radius: 12px;
padding: 0.85rem 1rem; font-family: ‘DM Sans’, sans-serif;
font-size: 0.95rem; color: var(–ink); outline: none;
margin-bottom: 1rem; transition: border-color 0.2s;
}
.input-field:focus { border-color: var(–rose); }
.full-btn {
width: 100%; background: var(–rose); color: white; border: none;
padding: 0.9rem; border-radius: 12px; font-family: ‘DM Sans’, sans-serif;
font-size: 0.95rem; font-weight: 500; cursor: pointer;
transition: transform 0.15s; margin-bottom: 0.8rem;
box-shadow: 0 4px 20px rgba(196,97,74,0.3);
}
.full-btn:hover { transform: translateY(-1px); }
.full-btn:disabled { opacity: 0.6; cursor: not-allowed; transform: none; }
.full-btn.dark { background: var(–ink); box-shadow: none; }
.full-btn.dark:hover { background: #2a1a14; }
.auth-switch { text-align: center; font-size: 0.82rem; color: var(–muted); }
.auth-switch a { color: var(–rose); cursor: pointer; text-decoration: none; }
.divider { display: flex; align-items: center; gap: 0.8rem; margin: 1rem 0; }
.divider::before, .divider::after { content: ‘’; flex: 1; height: 1px; background: rgba(0,0,0,0.1); }
.divider span { font-size: 0.75rem; color: var(–muted); }

/* SETUP USERNAME */
.prefix-input {
display: flex; align-items: center; background: var(–cream);
border: 1.5px solid rgba(0,0,0,0.1); border-radius: 12px;
margin-bottom: 1rem; overflow: hidden; transition: border-color 0.2s;
}
.prefix-input:focus-within { border-color: var(–rose); }
.prefix {
padding: 0.85rem 0.8rem 0.85rem 1rem; font-size: 0.85rem;
color: var(–muted); white-space: nowrap;
border-right: 1px solid rgba(0,0,0,0.08); background: rgba(0,0,0,0.02);
}
.prefix-input input {
flex: 1; border: none; background: transparent;
padding: 0.85rem 1rem; font-family: ‘DM Sans’, sans-serif;
font-size: 0.95rem; color: var(–ink); outline: none;
}

/* INBOX */
.app-container { max-width: 700px; margin: 0 auto; padding: 3rem 1.5rem 5rem; }
.page-title { font-family: ‘Playfair Display’, serif; font-size: 2rem; margin-bottom: 0.4rem; letter-spacing: -0.5px; }
.page-sub { color: var(–muted); font-size: 0.9rem; margin-bottom: 2rem; font-weight: 300; }

.link-box {
background: var(–card); border: 1px solid rgba(0,0,0,0.1);
border-radius: 16px; padding: 1.8rem; margin-bottom: 1.5rem;
box-shadow: 0 4px 24px rgba(0,0,0,0.05);
}
.link-box label { display: block; font-size: 0.78rem; text-transform: uppercase; letter-spacing: 1px; color: var(–muted); margin-bottom: 0.6rem; font-weight: 500; }
.link-row { display: flex; gap: 0.6rem; align-items: center; }
.link-input {
flex: 1; background: var(–cream); border: 1.5px solid rgba(0,0,0,0.1);
border-radius: 10px; padding: 0.7rem 1rem; font-family: ‘DM Sans’, sans-serif;
font-size: 0.88rem; color: var(–ink); outline: none;
}
.copy-btn {
background: var(–ink); color: var(–cream); border: none;
padding: 0.7rem 1.2rem; border-radius: 10px; font-size: 0.85rem;
cursor: pointer; font-family: ‘DM Sans’, sans-serif; transition: background 0.2s; white-space: nowrap;
}
.copy-btn:hover { background: var(–rose); }

.upgrade-banner {
background: linear-gradient(135deg, var(–ink) 0%, #2a1a14 100%);
color: var(–cream); border-radius: 16px; padding: 1.5rem 1.8rem;
display: flex; justify-content: space-between; align-items: center;
gap: 1rem; margin-bottom: 1.5rem;
}
.upgrade-banner h3 { font-family: ‘Playfair Display’, serif; font-size: 1.05rem; margin-bottom: 0.2rem; }
.upgrade-banner p { font-size: 0.8rem; opacity: 0.65; font-weight: 300; }
.upgrade-btn {
background: var(–rose); color: white; border: none;
padding: 0.6rem 1.4rem; border-radius: 100px; font-size: 0.82rem;
font-family: ‘DM Sans’, sans-serif; font-weight: 500; cursor: pointer;
white-space: nowrap; transition: transform 0.15s;
}
.upgrade-btn:hover { transform: scale(1.04); }

.section-label { font-size: 0.78rem; text-transform: uppercase; letter-spacing: 1px; color: var(–muted); margin-bottom: 1rem; font-weight: 500; }
.messages-feed { display: flex; flex-direction: column; gap: 1rem; }

.msg-card {
background: var(–card); border: 1px solid rgba(0,0,0,0.07);
border-radius: 14px; padding: 1.4rem 1.6rem;
box-shadow: 0 2px 12px rgba(0,0,0,0.04);
animation: fadeUp 0.4s ease both; position: relative; overflow: hidden;
}
.msg-card::before {
content: ‘”’; position: absolute; top: -10px; left: 14px;
font-family: ‘Playfair Display’, serif; font-size: 6rem;
color: var(–blush); line-height: 1; pointer-events: none;
}
.msg-text { font-size: 0.95rem; line-height: 1.7; padding-top: 0.4rem; position: relative; z-index: 1; }
.msg-meta { display: flex; justify-content: space-between; align-items: center; margin-top: 1rem; }
.msg-time { font-size: 0.75rem; color: var(–muted); font-style: italic; }
.reveal-btn {
background: var(–blush); color: var(–rose); border: none;
padding: 0.35rem 0.9rem; border-radius: 100px; font-size: 0.75rem;
font-family: ‘DM Sans’, sans-serif; font-weight: 500; cursor: pointer; transition: background 0.2s;
}
.reveal-btn:hover { background: var(–rose); color: white; }

.empty-state { text-align: center; padding: 3rem 1rem; color: var(–muted); }
.empty-state .icon { font-size: 2.5rem; margin-bottom: 0.8rem; }
.empty-state p { font-size: 0.9rem; line-height: 1.6; }

/* SEND PAGE */
.send-box {
background: var(–card); border: 1px solid rgba(0,0,0,0.08);
border-radius: 20px; padding: 2rem;
box-shadow: 0 8px 40px rgba(0,0,0,0.06);
}
.recipient-badge {
display: inline-flex; align-items: center; gap: 0.5rem;
background: var(–blush); color: var(–rose); padding: 0.4rem 1rem;
border-radius: 100px; font-size: 0.82rem; font-weight: 500; margin-bottom: 1.5rem;
}
.msg-textarea {
width: 100%; min-height: 160px; background: var(–cream);
border: 1.5px solid rgba(0,0,0,0.1); border-radius: 12px;
padding: 1rem; font-family: ‘DM Sans’, sans-serif; font-size: 0.95rem;
color: var(–ink); resize: vertical; outline: none; transition: border-color 0.2s; line-height: 1.6;
}
.msg-textarea:focus { border-color: var(–rose); }
.char-count { text-align: right; font-size: 0.75rem; color: var(–muted); margin-top: 0.4rem; margin-bottom: 1.2rem; }
.send-btn {
width: 100%; background: var(–rose); color: white; border: none;
padding: 0.95rem; border-radius: 12px; font-family: ‘DM Sans’, sans-serif;
font-size: 1rem; font-weight: 500; cursor: pointer;
transition: transform 0.15s, box-shadow 0.15s;
box-shadow: 0 4px 20px rgba(196,97,74,0.3);
}
.send-btn:hover { transform: translateY(-2px); }
.send-btn:disabled { opacity: 0.6; cursor: not-allowed; transform: none; }
.privacy-note { text-align: center; font-size: 0.78rem; color: var(–muted); margin-top: 1rem; font-style: italic; }

.success-screen { text-align: center; padding: 4rem 1rem; }
.success-icon { font-size: 3.5rem; margin-bottom: 1.2rem; animation: pop 0.5s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
.success-screen h2 { font-family: ‘Playfair Display’, serif; font-size: 1.8rem; margin-bottom: 0.6rem; }
.success-screen p { color: var(–muted); font-size: 0.9rem; line-height: 1.6; }

/* MODAL */
.modal-overlay {
display: none; position: fixed; inset: 0;
background: rgba(14,11,11,0.5); z-index: 100;
align-items: center; justify-content: center; backdrop-filter: blur(4px);
}
.modal-overlay.open { display: flex; }
.modal {
background: var(–card); border-radius: 20px; padding: 2rem;
max-width: 380px; width: 90%; box-shadow: 0 24px 80px rgba(0,0,0,0.2);
animation: fadeUp 0.3s ease; text-align: center;
}
.modal h3 { font-family: ‘Playfair Display’, serif; font-size: 1.4rem; margin-bottom: 0.5rem; }
.modal p { color: var(–muted); font-size: 0.88rem; line-height: 1.6; margin-bottom: 1.5rem; }
.modal-price { font-family: ‘Playfair Display’, serif; font-size: 2.5rem; color: var(–rose); margin-bottom: 1.5rem; }
.modal-price span { font-size: 1rem; color: var(–muted); font-family: ‘DM Sans’, sans-serif; }
.modal-btn {
width: 100%; background: var(–rose); color: white; border: none;
padding: 0.85rem; border-radius: 12px; font-family: ‘DM Sans’, sans-serif;
font-size: 0.95rem; cursor: pointer; margin-bottom: 0.6rem; transition: transform 0.15s;
}
.modal-btn:hover { transform: translateY(-1px); }
.modal-close { background: transparent; color: var(–muted); border: none; font-size: 0.82rem; cursor: pointer; font-family: ‘DM Sans’, sans-serif; padding: 0.3rem; }

.back-btn {
background: none; border: none; color: var(–muted); font-size: 0.85rem;
cursor: pointer; font-family: ‘DM Sans’, sans-serif; padding: 0 0 1.5rem;
display: flex; align-items: center; gap: 0.4rem; transition: color 0.2s;
}
.back-btn:hover { color: var(–ink); }

.loading { text-align: center; padding: 3rem; color: var(–muted); }
.spinner {
width: 32px; height: 32px; border: 3px solid var(–blush);
border-top-color: var(–rose); border-radius: 50%;
animation: spin 0.8s linear infinite; margin: 0 auto 1rem;
}
@keyframes spin { to { transform: rotate(360deg); } }

.toast {
position: fixed; bottom: 2rem; left: 50%;
transform: translateX(-50%) translateY(80px);
background: var(–ink); color: var(–cream); padding: 0.75rem 1.5rem;
border-radius: 100px; font-size: 0.85rem; z-index: 200;
transition: transform 0.3s cubic-bezier(0.34, 1.56, 0.64, 1); pointer-events: none;
}
.toast.show { transform: translateX(-50%) translateY(0); }

.error-msg { color: var(–rose); font-size: 0.82rem; margin-bottom: 0.8rem; text-align: center; }

@keyframes fadeUp { from { opacity: 0; transform: translateY(16px); } to { opacity: 1; transform: translateY(0); } }
@keyframes pop { from { transform: scale(0); opacity: 0; } to { transform: scale(1); opacity: 1; } }

@media (max-width: 640px) {
.hero { grid-template-columns: 1fr; padding: 3rem 1.5rem 2rem; }
.hero-visual { display: none; }
nav { padding: 1.2rem 1.5rem; }
}
</style>

</head>
<body>

<!-- NAV -->

<nav>
  <div class="logo">Whis<span>pr</span></div>
  <div class="nav-right" id="navRight">
    <button class="nav-btn outline" onclick="showPage('login')">Log In</button>
    <button class="nav-btn" onclick="showPage('signup')">Sign Up</button>
  </div>
</nav>

<!-- TOAST -->

<div class="toast" id="toast"></div>

<!-- REVEAL MODAL -->

<div class="modal-overlay" id="revealModal">
  <div class="modal">
    <div style="font-size:2rem;margin-bottom:0.8rem">🔍</div>
    <h3>Reveal the Sender</h3>
    <p>Unlock who sent this message. One-time purchase — only for this message.</p>
    <div class="modal-price">$0.99 <span>/ reveal</span></div>
    <button class="modal-btn" onclick="showToast('Payment coming soon! 💳');closeModal()">Unlock Now</button>
    <button class="modal-close" onclick="closeModal()">Maybe later</button>
  </div>
</div>

<!-- LANDING -->

<div id="page-landing" class="page active">
  <div class="hero">
    <div class="hero-text">
      <h1>Send a message, <em>stay unknown</em></h1>
      <p>Share your link. Let anyone send you anonymous messages — heartfelt, funny, or totally mysterious.</p>
      <div class="hero-actions">
        <button class="btn-primary" onclick="showPage('signup')">Create My Link</button>
      </div>
    </div>
    <div class="hero-visual">
      <div class="letter-card">
        <span class="tag">💌 Secret</span>
        <p>I've always admired how you carry yourself. You have no idea.</p>
        <div class="sender">— Anonymous</div>
      </div>
      <div class="letter-card">
        <span class="tag">😂 Funny</span>
        <p>You're the reason I laugh at inappropriate moments. No regrets.</p>
        <div class="sender">— Anonymous</div>
      </div>
      <div class="letter-card">
        <span class="tag">💬 Honest</span>
        <p>Your advice last year genuinely changed things for me. Thank you.</p>
        <div class="sender">— Anonymous</div>
      </div>
    </div>
  </div>
</div>

<!-- SIGN UP -->

<div id="page-signup" class="page">
  <div class="auth-container">
    <button class="back-btn" onclick="showPage('landing')">← Back</button>
    <div class="auth-box">
      <h2>Create account</h2>
      <p>Sign up to get your personal anonymous message link.</p>
      <div class="error-msg" id="signupError"></div>
      <input class="input-field" type="email" id="signupEmail" placeholder="Email address">
      <input class="input-field" type="password" id="signupPassword" placeholder="Password (min 6 characters)">
      <button class="full-btn" id="signupBtn" onclick="signUp()">Create Account →</button>
      <div class="auth-switch">Already have an account? <a onclick="showPage('login')">Log in</a></div>
    </div>
  </div>
</div>

<!-- LOG IN -->

<div id="page-login" class="page">
  <div class="auth-container">
    <button class="back-btn" onclick="showPage('landing')">← Back</button>
    <div class="auth-box">
      <h2>Welcome back</h2>
      <p>Log in to see your anonymous messages.</p>
      <div class="error-msg" id="loginError"></div>
      <input class="input-field" type="email" id="loginEmail" placeholder="Email address">
      <input class="input-field" type="password" id="loginPassword" placeholder="Password">
      <button class="full-btn" id="loginBtn" onclick="logIn()">Log In →</button>
      <div class="auth-switch">No account? <a onclick="showPage('signup')">Sign up free</a></div>
    </div>
  </div>
</div>

<!-- SET USERNAME -->

<div id="page-setup" class="page">
  <div class="auth-container">
    <div class="auth-box">
      <h2>Pick your username</h2>
      <p>This is the link you'll share with everyone.</p>
      <div class="error-msg" id="setupError"></div>
      <div class="prefix-input">
        <span class="prefix" id="sitePrefix">whispr.netlify.app/</span>
        <input type="text" id="usernameInput" placeholder="yourname" maxlength="20">
      </div>
      <p style="font-size:0.78rem;color:var(--muted);margin-bottom:1.2rem">Letters, numbers and underscores only. Min 3 characters.</p>
      <button class="full-btn" id="setupBtn" onclick="saveUsername()">Save & Go to Inbox →</button>
    </div>
  </div>
</div>

<!-- INBOX -->

<div id="page-inbox" class="page">
  <div class="app-container">
    <h2 class="page-title" id="inboxTitle">Your Inbox</h2>
    <p class="page-sub">Share your link and watch the messages arrive.</p>

```
<div class="link-box">
  <label>Your Link — Share This!</label>
  <div class="link-row">
    <input class="link-input" id="myLink" readonly>
    <button class="copy-btn" onclick="copyLink()">Copy</button>
  </div>
</div>

<div class="upgrade-banner">
  <div>
    <h3>✨ Whispr Pro</h3>
    <p>Reveal senders, no ads, unlimited messages.</p>
  </div>
  <button class="upgrade-btn" onclick="showToast('Pro coming soon! 🚀')">$2.99/mo</button>
</div>

<p class="section-label">Messages</p>
<div id="messagesFeed" class="messages-feed">
  <div class="loading"><div class="spinner"></div>Loading messages...</div>
</div>

<br>
<button class="back-btn" onclick="logOut()" style="color:var(--rose)">← Log Out</button>
```

  </div>
</div>

<!-- SEND MESSAGE -->

<div id="page-send" class="page">
  <div class="app-container">
    <h2 class="page-title">Send a message</h2>
    <p class="page-sub">100% anonymous. They'll never know it's you.</p>
    <div id="sendContent">
      <div class="send-box">
        <div class="recipient-badge">✉️ <span id="recipientName">someone</span></div>
        <textarea class="msg-textarea" id="msgTextarea" placeholder="Write something honest, kind, or mysterious..." maxlength="500" oninput="updateChar()"></textarea>
        <div class="char-count"><span id="charCount">0</span>/500</div>
        <button class="send-btn" id="sendMsgBtn" onclick="sendMessage()" disabled>Send Anonymously →</button>
        <p class="privacy-note">🔒 Your identity is completely hidden</p>
      </div>
    </div>
  </div>
</div>

<script>
  // ---- SUPABASE INIT ----
  const SUPABASE_URL = 'https://fgtlriiksbqepcszugde.supabase.co';
  const SUPABASE_KEY = 'sb_publishable_iGnsMNU3RF4280bIA8yIBg_29JBdmlJ';
  const { createClient } = supabase;
  const db = createClient(SUPABASE_URL, SUPABASE_KEY);

  let currentUser = null;
  let currentUsername = null;
  let sendToUsername = null;

  // ---- INIT ----
  async function init() {
    // Check if this is a /u/username URL
    const path = window.location.pathname;
    const match = path.match(/\/u\/([a-zA-Z0-9_]+)/);
    if (match) {
      sendToUsername = match[1];
      showPage('send');
      document.getElementById('recipientName').textContent = sendToUsername;
      return;
    }

    // Check hash for username (for Netlify single page)
    const hash = window.location.hash;
    const hashMatch = hash.match(/^#\/u\/([a-zA-Z0-9_]+)/);
    if (hashMatch) {
      sendToUsername = hashMatch[1];
      showPage('send');
      document.getElementById('recipientName').textContent = sendToUsername;
      return;
    }

    const { data: { session } } = await db.auth.getSession();
    if (session) {
      currentUser = session.user;
      await loadUserProfile();
    }
  }

  async function loadUserProfile() {
    const { data } = await db.from('profiles').select('username').eq('id', currentUser.id).single();
    if (data && data.username) {
      currentUsername = data.username;
      updateNav(true);
      showInbox();
    } else {
      updateNav(true);
      showPage('setup');
    }
  }

  function updateNav(loggedIn) {
    const nav = document.getElementById('navRight');
    if (loggedIn) {
      nav.innerHTML = `<button class="nav-btn" onclick="showInbox()">My Inbox</button>`;
    } else {
      nav.innerHTML = `
        <button class="nav-btn outline" onclick="showPage('login')">Log In</button>
        <button class="nav-btn" onclick="showPage('signup')">Sign Up</button>
      `;
    }
  }

  // ---- PAGES ----
  function showPage(name) {
    document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
    document.getElementById('page-' + name).classList.add('active');
    window.scrollTo(0, 0);
  }

  function showInbox() {
    const siteUrl = window.location.origin + window.location.pathname;
    document.getElementById('myLink').value = siteUrl + '#/u/' + currentUsername;
    document.getElementById('inboxTitle').textContent = 'Hey, ' + currentUsername + ' 👋';
    showPage('inbox');
    loadMessages();
  }

  // ---- AUTH ----
  async function signUp() {
    const email = document.getElementById('signupEmail').value.trim();
    const password = document.getElementById('signupPassword').value;
    const btn = document.getElementById('signupBtn');
    const err = document.getElementById('signupError');
    err.textContent = '';

    if (!email || !password) { err.textContent = 'Please fill in all fields.'; return; }
    if (password.length < 6) { err.textContent = 'Password must be at least 6 characters.'; return; }

    btn.disabled = true; btn.textContent = 'Creating account...';
    const { data, error } = await db.auth.signUp({ email, password });

    if (error) {
      err.textContent = error.message;
      btn.disabled = false; btn.textContent = 'Create Account →';
      return;
    }

    currentUser = data.user;
    updateNav(true);
    btn.disabled = false; btn.textContent = 'Create Account →';
    showPage('setup');
  }

  async function logIn() {
    const email = document.getElementById('loginEmail').value.trim();
    const password = document.getElementById('loginPassword').value;
    const btn = document.getElementById('loginBtn');
    const err = document.getElementById('loginError');
    err.textContent = '';

    if (!email || !password) { err.textContent = 'Please fill in all fields.'; return; }

    btn.disabled = true; btn.textContent = 'Logging in...';
    const { data, error } = await db.auth.signInWithPassword({ email, password });

    if (error) {
      err.textContent = 'Wrong email or password.';
      btn.disabled = false; btn.textContent = 'Log In →';
      return;
    }

    currentUser = data.user;
    btn.disabled = false; btn.textContent = 'Log In →';
    await loadUserProfile();
  }

  async function logOut() {
    await db.auth.signOut();
    currentUser = null; currentUsername = null;
    updateNav(false);
    showPage('landing');
  }

  // ---- USERNAME SETUP ----
  async function saveUsername() {
    const username = document.getElementById('usernameInput').value.trim().toLowerCase();
    const btn = document.getElementById('setupBtn');
    const err = document.getElementById('setupError');
    err.textContent = '';

    if (!username || username.length < 3) { err.textContent = 'Min 3 characters.'; return; }
    if (!/^[a-zA-Z0-9_]+$/.test(username)) { err.textContent = 'Only letters, numbers, underscores.'; return; }

    btn.disabled = true; btn.textContent = 'Saving...';

    // Check if username taken
    const { data: existing } = await db.from('profiles').select('id').eq('username', username).single();
    if (existing) {
      err.textContent = 'That username is taken. Try another!';
      btn.disabled = false; btn.textContent = 'Save & Go to Inbox →';
      return;
    }

    const { error } = await db.from('profiles').upsert({ id: currentUser.id, username, email: currentUser.email });
    if (error) {
      err.textContent = 'Something went wrong. Try again.';
      btn.disabled = false; btn.textContent = 'Save & Go to Inbox →';
      return;
    }

    currentUsername = username;
    btn.disabled = false; btn.textContent = 'Save & Go to Inbox →';
    showInbox();
  }

  // ---- MESSAGES ----
  async function loadMessages() {
    const feed = document.getElementById('messagesFeed');
    feed.innerHTML = '<div class="loading"><div class="spinner"></div>Loading...</div>';

    const { data, error } = await db.from('messages')
      .select('*')
      .eq('recipient_username', currentUsername)
      .order('created_at', { ascending: false });

    if (error || !data || data.length === 0) {
      feed.innerHTML = `<div class="empty-state"><div class="icon">📭</div><p>No messages yet.<br>Share your link to get started!</p></div>`;
      return;
    }

    feed.innerHTML = data.map((m, i) => `
      <div class="msg-card" style="animation-delay:${i * 0.08}s">
        <div class="msg-text">${escapeHtml(m.content)}</div>
        <div class="msg-meta">
          <span class="msg-time">${timeAgo(m.created_at)}</span>
          <button class="reveal-btn" onclick="openRevealModal()">👁 Reveal sender</button>
        </div>
      </div>
    `).join('');
  }

  async function sendMessage() {
    const content = document.getElementById('msgTextarea').value.trim();
    if (!content || !sendToUsername) return;

    const btn = document.getElementById('sendMsgBtn');
    btn.disabled = true; btn.textContent = 'Sending...';

    const { error } = await db.from('messages').insert({
      recipient_username: sendToUsername,
      content: content
    });

    if (error) {
      btn.disabled = false; btn.textContent = 'Send Anonymously →';
      showToast('Something went wrong. Try again.');
      return;
    }

    document.getElementById('sendContent').innerHTML = `
      <div class="success-screen">
        <div class="success-icon">💌</div>
        <h2>Message sent!</h2>
        <p>Your anonymous message is on its way.<br>They'll never know it was you.</p>
        <button class="btn-primary" style="margin-top:1.5rem" onclick="location.reload()">Send Another</button>
      </div>
    `;
  }

  // ---- HELPERS ----
  function updateChar() {
    const len = document.getElementById('msgTextarea').value.length;
    document.getElementById('charCount').textContent = len;
    document.getElementById('sendMsgBtn').disabled = len < 5;
  }

  function copyLink() {
    const val = document.getElementById('myLink').value;
    navigator.clipboard.writeText(val).catch(() => {});
    showToast('Link copied! Share it everywhere 🎉');
  }

  function openRevealModal() { document.getElementById('revealModal').classList.add('open'); }
  function closeModal() { document.getElementById('revealModal').classList.remove('open'); }

  function showToast(msg) {
    const t = document.getElementById('toast');
    t.textContent = msg; t.classList.add('show');
    setTimeout(() => t.classList.remove('show'), 2800);
  }

  function escapeHtml(text) {
    return text.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;');
  }

  function timeAgo(dateStr) {
    const diff = (Date.now() - new Date(dateStr)) / 1000;
    if (diff < 60) return 'Just now';
    if (diff < 3600) return Math.floor(diff/60) + ' min ago';
    if (diff < 86400) return Math.floor(diff/3600) + ' hours ago';
    return Math.floor(diff/86400) + ' days ago';
  }

  // ---- START ----
  init();
</script>

</body>
</html>
