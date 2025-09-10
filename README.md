```css name=TiTaPa.com/frontend/style.css
body { font-family: sans-serif; background: #0d0d1a; color: #fff; text-align:center; }
header { padding: 50px; }
button { margin:10px; padding:10px 20px; cursor:pointer; }
form { display:flex; flex-direction:column; max-width:300px; margin:auto; }
input, textarea { margin:5px 0; padding:8px; border-radius:5px; border:none; }
```

```javascript name=TiTaPa.com/frontend/app.js
const translations = {
  ar: {
    title: "TiTaPa",
    subtitle: "منصة SaaS متقدمة لمحاكاة الذكاء الاصطناعي والتحليلات الذكية",
    startBtn: "ابدأ الآن"
  },
  en: {
    title: "TiTaPa",
    subtitle: "Advanced SaaS Platform for AI Simulation & Smart Analytics",
    startBtn: "Get Started"
  }
};

let currentLang = "ar";
document.getElementById("lang-toggle").addEventListener("click", () => {
  currentLang = currentLang === "ar" ? "en" : "ar";
  document.getElementById("title").textContent = translations[currentLang].title;
  document.getElementById("subtitle").textContent = translations[currentLang].subtitle;
  document.getElementById("start-btn").textContent = translations[currentLang].startBtn;
  document.getElementById("lang-toggle").textContent = currentLang === "ar" ? "EN" : "AR";
});
```

```html name=TiTaPa.com/fullstack/frontend/index.html
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>TiTaPa - منصة الذكاء الاصطناعي</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <header>
    <h1 id="title">TiTaPa</h1>
    <p id="subtitle">منصة SaaS متقدمة لمحاكاة الذكاء الاصطناعي والتحليلات الذكية</p>
    <button id="start-btn">ابدأ الآن</button>
    <button id="lang-toggle">EN</button>
  </header>

  <section>
    <form>
      <input type="text" name="name" placeholder="الاسم">
      <input type="email" name="email" placeholder="البريد">
      <input type="text" name="subject" placeholder="الموضوع">
      <textarea name="message" placeholder="اكتب رسالتك"></textarea>
      <button type="submit">إرسال</button>
    </form>
  </section>

  <script src="app.js"></script>
</body>
</html>
```

```css name=TiTaPa.com/fullstack/frontend/style.css
body { font-family: sans-serif; background: #0d0d1a; color: #fff; text-align:center; }
header { padding: 50px; }
button { margin:10px; padding:10px 20px; cursor:pointer; }
form { display:flex; flex-direction:column; max-width:300px; margin:auto; }
input, textarea { margin:5px 0; padding:8px; border-radius:5px; border:none; }
```

```javascript name=TiTaPa.com/fullstack/frontend/app.js
const translations = {
  ar: {
    title: "TiTaPa",
    subtitle: "منصة SaaS متقدمة لمحاكاة الذكاء الاصطناعي والتحليلات الذكية",
    startBtn: "ابدأ الآن"
  },
  en: {
    title: "TiTaPa",
    subtitle: "Advanced SaaS Platform for AI Simulation & Smart Analytics",
    startBtn: "Get Started"
  }
};

let currentLang = "ar";
document.getElementById("lang-toggle").addEventListener("click", () => {
  currentLang = currentLang === "ar" ? "en" : "ar";
  document.getElementById("title").textContent = translations[currentLang].title;
  document.getElementById("subtitle").textContent = translations[currentLang].subtitle;
  document.getElementById("start-btn").textContent = translations[currentLang].startBtn;
  document.getElementById("lang-toggle").textContent = currentLang === "ar" ? "EN" : "AR";
});

document.querySelector("form").addEventListener("submit", async (e) => {
  e.preventDefault();

  const formData = {
    name: e.target.querySelector("input[name='name']").value,
    email: e.target.querySelector("input[name='email']").value,
    subject: e.target.querySelector("input[name='subject']").value,
    message: e.target.querySelector("textarea[name='message']").value,
  };

  try {
    const res = await fetch("https://YOUR-BACKEND.onrender.com/api/contact", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(formData),
    });
    const data = await res.json();
    if (data.success) {
      alert("✅ تم إرسال رسالتك بنجاح!");
      e.target.reset();
    } else {
      alert("⚠️ خطأ: " + data.error);
    }
  } catch (err) {
    alert("❌ مشكلة في الاتصال بالخادم");
  }
});
```

```javascript name=TiTaPa.com/fullstack/backend/server.js
const express = require("express");
const bodyParser = require("body-parser");
const cors = require("cors");
const sqlite3 = require("sqlite3").verbose();

const app = express();
const PORT = process.env.PORT || 3000;

app.use(bodyParser.json());
app.use(cors());

const db = new sqlite3.Database("./database.db");

db.serialize(() => {
  db.run("CREATE TABLE IF NOT EXISTS messages (id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT, email TEXT, subject TEXT, message TEXT)");
});

app.post("/api/contact", (req, res) => {
  const { name, email, subject, message } = req.body;
  if (!name || !email || !message) {
    return res.json({ success: false, error: "Missing fields" });
  }
  db.run("INSERT INTO messages (name, email, subject, message) VALUES (?, ?, ?, ?)",
    [name, email, subject, message],
    function(err) {
      if (err) return res.json({ success: false, error: err.message });
      res.json({ success: true, id: this.lastID });
    });
});

app.get("/api/messages", (req, res) => {
  db.all("SELECT * FROM messages", [], (err, rows) => {
    if (err) return res.json({ success: false, error: err.message });
    res.json({ success: true, messages: rows });
  });
});

app.listen(PORT, () => console.log(`Server running on http://localhost:${PORT}`));
```

```json name=TiTaPa.com/fullstack/backend/package.json
{
  "name": "titapa-backend",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "body-parser": "^1.20.2",
    "cors": "^2.8.5",
    "express": "^4.18.2",
    "sqlite3": "^5.1.6"
  }
}
```

````text name=TiTaPa.com/LICENSE.txt
Copyright (c) 2025 Boo90  
All rights reserved.  

This software is proprietary and licensed for commercial use only.  
Redistribution, modification, or resale without written permission is strictly prohibited.  

Unauthorized use of this code, in whole or in part, is a violation of copyright law.  
````

````markdown name=TiTaPa.com/README.md
# 🌐 TiTaPa.com

**TiTaPa** هي منصة SaaS متقدمة لمحاكاة الذكاء الاصطناعي والتحليلات الذكية.  
هذا المستودع يحتوي على نسختين:  
1. Frontend فقط (ثنائية اللغة + Formspree).  
2. Fullstack (Frontend + Backend + SQLite).  

⚠️ جميع الحقوق محفوظة © 2025 Boo90 – هذا المشروع تجاري Proprietary، لا يُسمح بإعادة النشر أو الاستخدام إلا بإذن خطي.
````    
