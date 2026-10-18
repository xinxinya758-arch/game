# game<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>水池深處的回音 - POOLCORE ECHOES</title>
  <style>
    /* ==========================================
       1. 基礎重置與豎屏滿版佈局
       ========================================== */
    * { box-sizing: border-box; margin: 0; padding: 0; user-select: none; }
    
    body {
      background-color: #03080c;
      color: #e0e1dd;
      font-family: 'Courier New', Courier, monospace;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      overflow: hidden;
    }

    #game-container {
      width: 100vw;
      height: 100vh;
      max-width: 430px; /* 模擬手機螢幕寬度 */
      position: relative;
      display: flex;
      flex-direction: row;
      border: 1px solid #1f3a4b;
      overflow: hidden;
      background-color: #0d1b2a;
      background-size: cover;
      background-position: center;
      background-repeat: no-repeat;
    }

    /* ==========================================
       2. 豎向水位線 (Vertical Water Gauge)
       ========================================== */
    #vertical-water-bar {
      width: 14px;
      height: 100%;
      background: rgba(5, 15, 25, 0.9);
      border-right: 2px solid #00b4d8;
      position: relative;
      display: flex;
      align-items: flex-end;
      z-index: 20;
    }

    #water-fill {
      width: 100%;
      height: 10%; /* 初始水位 */
      background: linear-gradient(0deg, rgba(0,180,216,0.9) 0%, rgba(144,224,239,0.5) 100%);
      box-shadow: 0 0 10px #00b4d8;
      transition: height 0.6s cubic-bezier(0.25, 1, 0.5, 1);
    }

    /* ==========================================
       3. 遊戲主顯示區域
       ========================================== */
    #main-content {
      flex: 1;
      display: flex;
      flex-direction: column;
      position: relative;
      background: rgba(13, 27, 42, 0.45);
    }

    /* 頂部功能按鈕列 */
    #top-bar {
      padding: 10px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      z-index: 10;
      background: rgba(8, 18, 28, 0.6);
    }

    .ui-btn {
      background: rgba(13, 27, 42, 0.85);
      color: #90e0ef;
      border: 1px solid #00b4d8;
      padding: 6px 12px;
      border-radius: 4px;
      font-size: 0.8rem;
      cursor: pointer;
    }

    #autosave-indicator {
      position: absolute;
      top: 45px;
      right: 10px;
      font-size: 0.7rem;
      color: #90e0ef;
      background: rgba(0,0,0,0.7);
      padding: 3px 6px;
      border-radius: 3px;
      opacity: 0;
      transition: opacity 0.5s ease;
      z-index: 30;
    }

    /* 配角探索區 / 守關立繪顯示區 */
    #stage-viewport {
      flex: 1;
      display: flex;
      align-items: flex-end;
      justify-content: center;
      position: relative;
      overflow: hidden;
    }

    /* 守關角色立繪 */
    #character-sprite {
      height: 85%;
      max-width: 100%;
      object-fit: contain;
      filter: drop-shadow(0px 4px 8px rgba(0, 0, 0, 0.8));
      transition: all 0.3s ease;
    }

    /* 可點擊的小配角圖案/按鈕 */
    .npc-element {
      position: absolute;
      background: rgba(0, 180, 216, 0.2);
      border: 1px dashed #48cae4;
      color: #fff;
      padding: 8px 12px;
      border-radius: 20px;
      font-size: 0.75rem;
      cursor: pointer;
      backdrop-filter: blur(2px);
      animation: pulse 2s infinite alternate;
    }

    @keyframes pulse {
      0% { transform: scale(1); box-shadow: 0 0 0 rgba(0,180,216,0.4); }
      100% { transform: scale(1.05); box-shadow: 0 0 10px rgba(0,180,216,0.8); }
    }

    /* 對話框 */
    #dialogue-box {
      background: rgba(10, 20, 30, 0.92);
      border-top: 2px solid #415a77;
      padding: 12px 15px;
      min-height: 85px;
    }

    #speaker-name {
      color: #90e0ef;
      font-weight: bold;
      font-size: 0.9rem;
      margin-bottom: 5px;
    }

    #dialogue-text {
      font-size: 0.85rem;
      line-height: 1.4;
      color: #e0e1dd;
    }

    /* 選項列表區 */
    #options-container {
      background: #0a141e;
      padding: 10px;
      display: flex;
      flex-direction: column;
      gap: 8px;
      min-height: 120px;
    }

    .option-btn {
      background: #1b263b;
      color: #e0e1dd;
      border: 1px solid #415a77;
      padding: 10px 12px;
      border-radius: 4px;
      text-align: left;
      font-size: 0.8rem;
      cursor: pointer;
    }

    .option-btn:active {
      background: #00b4d8;
      color: #000;
    }

    /* ==========================================
       4. 全螢幕 Overlays (登錄介面與過場動畫)
       ========================================== */
    .overlay {
      position: absolute;
      top: 0; left: 0; width: 100%; height: 100%;
      background: rgba(5, 10, 14, 0.95);
      z-index: 100;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: space-between;
      padding: 50px 20px;
      text-align: center;
    }

    /* 【登錄介面專屬背景】使用您提供的黃色小鴨圖片 */
    #start-screen {
      background-image: linear-gradient(180deg, rgba(8, 20, 29, 0.75) 0%, rgba(13, 31, 45, 0.3) 50%, rgba(5, 12, 18, 0.85) 100%), 
                        url('71042.jpg'); /* 🖼️ 教程：登錄背景圖路徑 */
      background-size: cover;
      background-position: center;
    }

    .title-main {
      font-size: 1.8rem;
      color: #83c5be;
      letter-spacing: 3px;
      text-shadow: 0 0 10px rgba(131, 197, 190, 0.6), 2px 2px 0px #000;
      margin-bottom: 8px;
    }

    .menu-box {
      width: 100%;
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 12px;
    }

    .menu-btn {
      width: 80%;
      padding: 12px;
      background: rgba(13, 27, 42, 0.85);
      color: #e0e1dd;
      border: 1px solid #48cae4;
      border-radius: 4px;
      font-size: 0.95rem;
      cursor: pointer;
      backdrop-filter: blur(4px);
    }

    .menu-btn:disabled {
      border-color: #354f52;
      color: #526760;
      cursor: not-allowed;
    }

    /* 過場動畫分鏡容器 */
    #cutscene-screen {
      display: none;
      justify-content: center;
      background: #050a0e;
    }

    .cutscene-img {
      width: 100%;
      height: 60%;
      object-fit: cover;
      border: 1px solid #00b4d8;
      border-radius: 4px;
      margin-bottom: 20px;
    }
  </style>
</head>
<body>

<div id="game-container">

  <div id="autosave-indicator">💾 已自動保存</div>

  <!-- 豎向水位線 -->
  <div id="vertical-water-bar">
    <div id="water-fill"></div>
  </div>

  <!-- 遊戲主要內容區域 -->
  <div id="main-content">
    <div id="top-bar">
      <button class="ui-btn" onclick="openNotebook()">📖 線索筆記</button>
      <button class="ui-btn" onclick="confirmReset()">🔄 選單</button>
    </div>

    <!-- 角色立繪與配角點擊區 -->
    <div id="stage-viewport">
      <!-- 小配角 1 -->
      <div id="npc-1" class="npc-element" style="bottom: 40%; left: 15%; display:none;" onclick="interactNPC(1)">
        🐤 小鴨子
      </div>
      <!-- 小配角 2 -->
      <div id="npc-2" class="npc-element" style="bottom: 25%; right: 15%; display:none;" onclick="interactNPC(2)">
        🪓 生鏽的斧頭
      </div>
      
      <!-- 🖼️ 教程：守關角色立繪標籤 -->
      <img id="character-sprite" src="" alt="立繪" style="display: none;">
    </div>

    <!-- 對話框 -->
    <div id="dialogue-box">
      <div id="speaker-name">角色名</div>
      <p id="dialogue-text">準備進入池核世界...</p>
    </div>

    <!-- 選項按鈕區 -->
    <div id="options-container"></div>
  </div>

  <!-- ==========================================
       5. 登錄首頁 Overlay
       ========================================== -->
  <div id="start-screen" class="overlay">
    <div style="margin-top: 30px;">
      <div class="title-main">水池深處的回音</div>
      <div style="font-size:0.8rem; color:#64dfdf; letter-spacing: 3px;">POOLCORE ECHOES</div>
    </div>

    <div id="menu-buttons" class="menu-box">
      <button id="continue-btn" class="menu-btn" onclick="loadGame()">繼續遊戲</button>
      <button class="menu-btn" onclick="showNameInput()">新遊戲</button>
    </div>

    <!-- 名字輸入方塊 -->
    <div id="name-input-box" style="display:none; width: 85%; background: rgba(10,22,31,0.9); padding: 20px; border: 1px solid #48cae4; border-radius: 6px;">
      <p style="color: #90e0ef; font-size: 0.85rem; margin-bottom: 10px;">請輸入主人公的名字：</p>
      <input type="text" id="name-input" placeholder="名字..." maxlength="8" style="width: 90%; padding: 8px; margin-bottom: 15px; text-align: center; background:#0a141e; border:1px solid #778da9; color:#fff;">
      <button class="menu-btn" style="width: 100%;" onclick="startNewGame()">進入池核</button>
    </div>

    <div style="font-size:0.7rem; color:#526760;">POOLCORE ADVENTURE GAME v1.0</div>
  </div>

  <!-- ==========================================
       6. 三分鏡過場動畫 Overlay
       ========================================== -->
  <div id="cutscene-screen" class="overlay">
    <div style="width:100%;">
      <!-- 🖼️ 教程：過場動畫圖片顯示位置 -->
      <img id="cutscene-img" class="cutscene-img" src="assets/cutscene_1.png" alt="過場分鏡" 
           onerror="this.style.display='none'; document.getElementById('cutscene-placeholder').style.display='flex';">
      
      <!-- 圖片載入失敗時顯示的文字替代框 -->
      <div id="cutscene-placeholder" style="display:none; width:100%; height:220px; background:#1b263b; border:1px solid #00b4d8; align-items:center; justify-content:center; color:#90e0ef; margin-bottom:20px; padding:15px; font-size:0.9rem;">
        [分鏡畫面]
      </div>

      <p id="cutscene-text" style="font-size:0.9rem; line-height:1.5; color:#e0e1dd; min-height:60px;">
        分鏡描述...
      </p>
    </div>

    <button class="menu-btn" style="width:100%;" onclick="nextCutscene()">點擊繼續 ▶</button>
  </div>

</div>

<!-- ==========================================
     7. 遊戲核心 JavaScript 邏輯
     ========================================== -->
<script>
  /* ---------------------------------------------------
     🖼️ 圖片路徑配置資料庫 (IMAGE ASSETS DATABASE)
     您可以將此處替換為您生成的實際圖片檔名！
  --------------------------------------------------- */
  const ASSETS = {
    // 關卡背景圖
    backgrounds: {
      tutorial: 'assets/bg_tutorial.png',
      stage1: 'assets/bg_stage1.png'
    },
    // 過場動畫分鏡圖 (3個分鏡)
    cutscenes: [
      'assets/cutscene_1.png', // 1. 主人公趴在泳池瓷磚上
      'assets/cutscene_2.png', // 2. 主角抬起頭露出茫然神情
      'assets/cutscene_3.png'  // 3. 一隻手伸手向主角
    ],
    // 守關角色立繪字典 (4種表情)
    characters: {
      meng: { // 新手嚮導「夢」
        normal: 'assets/meng_normal.png',
        smile:  'assets/meng_smile.png',
        frown:  'assets/meng_frown.png',
        open:   'assets/meng_open.png'
      },
      ling: { // 關卡1「凌齊安」
        normal: 'assets/ling_normal.png',
        smile:  'assets/ling_smile.png',
        frown:  'assets/ling_frown.png',
        open:   'assets/ling_open.png'
      }
    }
  };

  /* ---------------------------------------------------
     全局遊戲狀態 (GAME STATE)
  --------------------------------------------------- */
  let gameState = {
    playerName: "",
    currentStage: "tutorial_explore", // tutorial_explore, tutorial_boss, stage1, etc.
    dialogueRound: 1,
    waterLevel: 10,
    notebookClues: [],
    cutsceneIndex: 0
  };

  const SAVE_KEY = "POOLCORE_GAME_SAVE_V1";

  // 檢查本地是否有存檔
  window.onload = function() {
    const savedData = localStorage.getItem(SAVE_KEY);
    const continueBtn = document.getElementById('continue-btn');
    if (savedData) {
      continueBtn.disabled = false;
      continueBtn.innerText = "繼續遊戲 (有存檔)";
    } else {
      continueBtn.disabled = true;
      continueBtn.innerText = "繼續遊戲";
    }
  };

  // 自動保存與提示
  function saveGame() {
    localStorage.setItem(SAVE_KEY, JSON.stringify(gameState));
    const indicator = document.getElementById('autosave-indicator');
    indicator.style.opacity = '1';
    setTimeout(() => { indicator.style.opacity = '0'; }, 1500);
  }

  function loadGame() {
    const savedData = localStorage.getItem(SAVE_KEY);
    if (savedData) {
      gameState = JSON.parse(savedData);
      document.getElementById('start-screen').style.display = 'none';
      renderCurrentState();
    }
  }

  // 新遊戲流程
  function showNameInput() {
    document.getElementById('menu-buttons').style.display = 'none';
    document.getElementById('name-input-box').style.display = 'block';
  }

  function startNewGame() {
    const name = document.getElementById('name-input').value.trim();
    if (!name) { alert("請輸入名字！"); return; }

    gameState = {
      playerName: name,
      currentStage: "cutscene",
      dialogueRound: 1,
      waterLevel: 10,
      notebookClues: [],
      cutsceneIndex: 0
    };

    saveGame();
    document.getElementById('start-screen').style.display = 'none';
    startCutscene();
  }

  /* ---------------------------------------------------
     過場動畫系統 (CUTSCENE SYSTEM)
  --------------------------------------------------- */
  const cutsceneTexts = [
    "冰冷的觸感從臉頰傳來。你趴在地上，地面是游泳池邊濕滑的藍白色瓷磚...",
    "你艱難地抬起頭，環顧四周那一片沉寂無人的湛藍池水，眼神充滿了茫然...",
    "突然，水面波光粼粼，一隻泛著微光、修長的手向你伸手伸了過來——"
  ];

  function startCutscene() {
    document.getElementById('cutscene-screen').style.display = 'flex';
    updateCutsceneUI();
  }

  function updateCutsceneUI() {
    const idx = gameState.cutsceneIndex;
    document.getElementById('cutscene-text').innerText = cutsceneTexts[idx];
    
    // 設定過場圖片 src
    const imgEl = document.getElementById('cutscene-img');
    const placeholder = document.getElementById('cutscene-placeholder');
    imgEl.src = ASSETS.cutscenes[idx];
    imgEl.style.display = 'block';
    placeholder.style.display = 'none';
    placeholder.innerText = `[分鏡 ${idx + 1} 畫面]`;
  }

  function nextCutscene() {
    gameState.cutsceneIndex++;
    if (gameState.cutsceneIndex < 3) {
      updateCutsceneUI();
    } else {
      // 動畫結束，進入新手教程探索階段
      document.getElementById('cutscene-screen').style.display = 'none';
      gameState.currentStage = "tutorial_explore";
      saveGame();
      renderCurrentState();
    }
  }

  /* ---------------------------------------------------
     場景渲染與邏輯驅動
  --------------------------------------------------- */
  function renderCurrentState() {
    // 1. 更新豎向水位線高度
    document.getElementById('water-fill').style.height = gameState.waterLevel + '%';

    // 2. 階段判定：新手教程探索階段
    if (gameState.currentStage === "tutorial_explore") {
      setBackground(ASSETS.backgrounds.tutorial);
      setSprite(null); // 隱藏守關角色
      document.getElementById('npc-1').style.display = 'block';
      document.getElementById('npc-2').style.display = 'block';
      
      document.getElementById('speaker-name').innerText = "旁白";
      document.getElementById('dialogue-text').innerText = "觀察一下四周吧，點擊泳池邊的物品尋求線索。搜集完後將面對守關者。";
      
      renderOptions([
        { text: "與「夢」對話 (進入守關決戰)", action: () => enterTutorialBoss() }
      ]);
    }
    // 新手教程守關對話階段
    else if (gameState.currentStage === "tutorial_boss") {
      document.getElementById('npc-1').style.display = 'none';
      document.getElementById('npc-2').style.display = 'none';

      document.getElementById('speaker-name').innerText = "夢";

      if (gameState.dialogueRound === 1) {
        setSprite(ASSETS.characters.meng.normal);
        document.getElementById('dialogue-text').innerText = `「親愛的 ${gameState.playerName}，這裡就是你以後會待的地方啦！是不是感覺很溫暖？」`;
        
        renderOptions([
          { text: "1. 你他媽到底是誰，我為什麼會在這裡！", correct: false, wrongResp: "「嘿！講話放尊重點！」", wrongExpr: ASSETS.characters.meng.frown },
          { text: "2. 這裡看起來很溫暖，但那把斧頭…令人有些不安。", correct: true, nextExpr: ASSETS.characters.meng.smile },
          { text: "3. 太好了！那我可以在這裡一直睡下去嗎？", correct: false, wrongResp: "「你真的明白這裡是什麼地方嗎？」", wrongExpr: ASSETS.characters.meng.frown }
        ]);
      } else if (gameState.dialogueRound === 2) {
        setSprite(ASSETS.characters.meng.smile);
        document.getElementById('dialogue-text').innerText = "「那麼…你想要繼續深入前進，還是留在原地呢？」";
        
        renderOptions([
          { text: "⭕ 選擇前進：與其留在此處，不如繼續前進。", action: () => chooseEndingChoice("GO") },
          { text: "⭕ 選擇留在原地：這裡很安全，我不願嘗試往前走。", action: () => chooseEndingChoice("STAY") }
        ]);
      }
    }
  }

  /* ---------------------------------------------------
     小配角互動與筆記本紀錄
  --------------------------------------------------- */
  function interactNPC(id) {
    if (id === 1) {
      addClue("［享受］：小鴨子說這裡很舒服，但隱約感覺背脊發涼。");
      document.getElementById('dialogue-text').innerText = "小鴨子：「嘿！第一次來到這兒？你會好好享受這裡的！」（線索已記錄至筆記本）";
    } else if (id === 2) {
      addClue("［怪異］：安詳的泳池邊卻放著一把危險的斧頭。");
      document.getElementById('dialogue-text').innerText = "我：「為什麼這裡會有一把生鏽的斧頭…？」（線索已記錄至筆記本）";
    }
  }

  function addClue(clueText) {
    if (!gameState.notebookClues.includes(clueText)) {
      gameState.notebookClues.push(clueText);
      saveGame();
    }
  }

  function enterTutorialBoss() {
    if (gameState.notebookClues.length < 2) {
      alert("建議先點擊小鴨子和斧頭收集完線索後再與守關角色對話！");
    }
    gameState.currentStage = "tutorial_boss";
    gameState.dialogueRound = 1;
    saveGame();
    renderCurrentState();
  }

  /* ---------------------------------------------------
     對話選項判定與水位懲罰
  --------------------------------------------------- */
  function handleOptionClick(opt) {
    if (opt.action) { opt.action(); return; }

    if (opt.correct) {
      // 答對：進入下一輪對話
      gameState.dialogueRound++;
      saveGame();
      renderCurrentState();
    } else {
      // 答錯：水位上升 +25%，切換為生氣/皺眉表情
      gameState.waterLevel += 25;
      saveGame();
      
      if (opt.wrongExpr) setSprite(opt.wrongExpr);
      if (opt.wrongResp) document.getElementById('dialogue-text').innerText = opt.wrongResp;
      
      document.getElementById('water-fill').style.height = gameState.waterLevel + '%';

      // 水位線淹沒懲罰判定
      if (gameState.waterLevel >= 85) {
        setTimeout(() => {
          alert("水位已升高至頂部，你被冰冷的水淹沒了……關卡重置。");
          gameState.waterLevel = 10;
          gameState.dialogueRound = 1;
          saveGame();
          renderCurrentState();
        }, 500);
      }
    }
  }

  function chooseEndingChoice(type) {
    if (type === "GO") {
      alert("你選擇了繼續深入。解鎖下一關【 Stage 1 - 雨中盛開的櫻花 (凌齊安) 】！");
      // 後續在此處擴充進入關卡 1 的邏輯...
    } else {
      alert("你選擇了留在原地。你與小鴨子一起留在平靜的水池中，觸發【結局 A：沉溺/融為池水】。");
    }
  }

  /* ---------------------------------------------------
     UI 輔助函數 (替換圖片與表情)
  --------------------------------------------------- */
  function setSprite(srcPath) {
    const sprite = document.getElementById('character-sprite');
    if (srcPath) {
      sprite.src = srcPath;
      sprite.style.display = 'block';
    } else {
      sprite.style.display = 'none';
    }
  }

  function setBackground(bgPath) {
    document.getElementById('game-container').style.backgroundImage = `url('${bgPath}')`;
  }

  function renderOptions(opts) {
    const container = document.getElementById('options-container');
    container.innerHTML = "";
    opts.forEach(opt => {
      const btn = document.createElement('button');
      btn.className = 'option-btn';
      btn.innerText = opt.text;
      btn.onclick = () => handleOptionClick(opt);
      container.appendChild(btn);
    });
  }

  function openNotebook() {
    const clues = gameState.notebookClues.length > 0 
      ? gameState.notebookClues.map(c => "• " + c).join("\n\n")
      : "目前筆記本中尚未記錄任何關鍵字。";
    alert(`【 ${gameState.playerName} 的線索筆記本 】\n\n${clues}`);
  }

  function confirmReset() {
    if (confirm("要回到主選單嗎？（目前進度已自動保存）")) {
      location.reload();
    }
  }
</script>

</body>
</html>
