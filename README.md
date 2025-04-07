<!DOCTYPE html>
<html lang="zh-Hant">
<head>
  <meta charset="UTF-8" />
  <title>冒險者成長之路</title>
  <link href="https://fonts.googleapis.com/css2?family=MedievalSharp&display=swap" rel="stylesheet">
  <style>
    body {
      font-family: 'MedievalSharp', cursive;
      background: url('https://images.unsplash.com/photo-1605972405835-04e8c5a6bde4') no-repeat center center fixed;
      background-size: cover;
      color: #fff;
      padding: 20px;
    }

    h1, h2 {
      color: gold;
      text-shadow: 2px 2px 4px #000;
    }

    .character-card {
      background: rgba(0, 0, 0, 0.7);
      padding: 15px;
      margin-bottom: 15px;
      border: 2px solid gold;
      border-radius: 8px;
      position: relative;
    }

    .xp-bar {
      height: 20px;
      background: #444;
      margin-top: 5px;
      position: relative;
      border-radius: 10px;
    }

    .xp-progress {
      background: linear-gradient(to right, gold, orange);
      height: 100%;
      width: 0;
      border-radius: 10px;
    }

    button {
      margin: 3px;
      padding: 6px 10px;
      background: #222;
      color: #fff;
      border: 1px solid gold;
      border-radius: 4px;
      cursor: pointer;
    }

    button:hover {
      background: gold;
      color: #000;
    }

    input, select {
      padding: 5px;
      font-family: 'MedievalSharp', cursive;
    }

    .job-img {
      width: 80px;
      height: 80px;
      object-fit: contain;
      position: absolute;
      top: 10px;
      right: 10px;
    }

    .level-up {
      position: absolute;
      top: 10px;
      left: 50%;
      transform: translateX(-50%);
      font-size: 28px;
      font-weight: bold;
      color: gold;
      text-shadow: 0 0 10px #fff, 0 0 20px gold;
      animation: glow 1.5s ease-out;
    }

    @keyframes glow {
      0% { opacity: 0; transform: translateX(-50%) scale(1); }
      30% { opacity: 1; transform: translateX(-50%) scale(1.2); }
      60% { opacity: 1; transform: translateX(-50%) scale(1); }
      100% { opacity: 0; transform: translateX(-50%) scale(1); }
    }

    .login {
      background: rgba(0,0,0,0.8);
      padding: 20px;
      border: 2px solid gold;
      max-width: 400px;
      margin: 100px auto;
      border-radius: 10px;
      text-align: center;
    }
  </style>
</head>
<body>

  <div id="loginScreen" class="login">
    <h2>冒險者登入</h2>
    <input type="text" id="usernameInput" placeholder="請輸入使用者名稱" />
    <br /><br />
    <button onclick="login()">登入</button>
  </div>

  <div id="gameScreen" style="display:none;">
    <h1>冒險者成長之路</h1>

    <div>
      <h2>新增角色</h2>
      <input type="text" id="newName" placeholder="角色名稱" />
      <select id="newClass">
        <option value="法師">法師</option>
        <option value="弓箭手">弓箭手</option>
        <option value="劍士">劍士</option>
        <option value="盜賊">盜賊</option>
        <option value="祭司">祭司</option>
      </select>
      <button onclick="createCharacter()">創建角色</button>
    </div>

    <hr />

    <h2>所有角色</h2>
    <div id="characterList"></div>
  </div>

  <script>
    let currentUser = null;
    const levelXp = [10, 13, 16, 19, 22, 25, 28, 31, 34, 37];
    const jobImages = {
      '法師': 'https://i.imgur.com/ZIiC03U.png',
      '弓箭手': 'https://i.imgur.com/fEvLu5K.png',
      '劍士': 'https://i.imgur.com/yFHp9Jd.png',
      '盜賊': 'https://i.imgur.com/hzA1ljy.png',
      '祭司': 'https://i.imgur.com/UxqZcxG.png'
    };

    function login() {
      const username = document.getElementById("usernameInput").value.trim();
      if (!username) return alert("請輸入使用者名稱！");
      currentUser = username;
      if (!localStorage.getItem("adventurer_game_" + currentUser)) {
        localStorage.setItem("adventurer_game_" + currentUser, JSON.stringify([]));
      }
      document.getElementById("loginScreen").style.display = "none";
      document.getElementById("gameScreen").style.display = "block";
      renderCharacters();
    }

    function getCharacters() {
      return JSON.parse(localStorage.getItem("adventurer_game_" + currentUser) || "[]");
    }

    function saveCharacters(chars) {
      localStorage.setItem("adventurer_game_" + currentUser, JSON.stringify(chars));
    }

    function createCharacter() {
      const name = document.getElementById("newName").value.trim();
      const job = document.getElementById("newClass").value;
      if (!name) return alert("請輸入名稱！");
      const chars = getCharacters();
      if (chars.find(c => c.name === name)) return alert("角色名稱已存在！");
      chars.push({ name, job, level: 1, xp: 0 });
      saveCharacters(chars);
      renderCharacters();
      document.getElementById("newName").value = "";
    }

    function addXP(name, amount) {
      const chars = getCharacters();
      const char = chars.find(c => c.name === name);
      if (!char) return;
      let leveledUp = false;

      char.xp += amount;
      while (true) {
        const required = levelXp[char.level - 1] || 100000;
        if (char.xp >= required) {
          char.xp -= required;
          char.level++;
          leveledUp = true;
        } else break;
      }

      saveCharacters(chars);
      renderCharacters();

      if (leveledUp) {
        const box = document.querySelector(`[data-name="${name}"]`);
        const msg = document.createElement("div");
        msg.className = "level-up";
        msg.innerText = "Level Up!!!";
        box.appendChild(msg);
        setTimeout(() => msg.remove(), 2000);
      }
    }

    function renderCharacters() {
      const chars = getCharacters();
      const container = document.getElementById("characterList");
      container.innerHTML = "";

      chars.forEach(char => {
        const required = levelXp[char.level - 1] || 100000;
        const percent = Math.min((char.xp / required) * 100, 100);

        const div = document.createElement("div");
        div.className = "character-card";
        div.dataset.name = char.name;

        div.innerHTML = `
          <strong>${char.name}（${char.job}）</strong><br>
          等級：${char.level}<br>
          經驗值：${char.xp} / ${required}
          <div class="xp-bar"><div class="xp-progress" style="width: ${percent}%"></div></div>
          <button onclick="addXP('${char.name}', 1)">邀約客戶 +1 XP</button>
          <button onclick="addXP('${char.name}', 2)">Demo 客戶 +2 XP</button>
          <button onclick="addXP('${char.name}', 3)">刷卡客戶 +3 XP</button>
          <img src="${jobImages[char.job]}" class="job-img" alt="${char.job}">
        `;
        container.appendChild(div);
      });
    }
  </script>
</body>
</html>
