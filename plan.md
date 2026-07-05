# INFP 安眠 / INFP Sleep Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 交付一个纯前端单文件的手机端睡前仪式引导工具（`infp-sleep-mobile.html`），六屏顺序引导（开场提示→洗漱→清空大脑→冥想→感恩三件事→看书），发布到 GitHub Pages（不发布 Claude Artifact）。

**Architecture:** 单文件 HTML/CSS/JS，无框架无依赖，与 `unstuck-mobile.html` 同一技术路线（卡片式单屏切换、CSS 变量主题、i18n 字典驱动的双语切换）。冥想步骤额外使用浏览器原生 Web Speech API（语音提示）和 Web Audio API（实时合成白噪音），不引入任何外部文件。

**Tech Stack:** 纯 HTML/CSS/JS（无框架、无外部依赖），Web Speech API，Web Audio API

## Global Constraints

- 记录文件路径：`C:\Users\lucky\ideas\raw.md`（供用户手动复制粘贴，工具本身不写入）
- 记录行格式：`- [YYYY-MM-DD HH:mm] [sleep] 感恩:<事1>；<事2>；<事3>`（三个感恩项用中文分号"；"分隔）
- 步骤2"清空大脑"输入的内容**绝不保存**到任何地方（不进 localStorage，不进记录行）
- 不联网、不发起任何网络请求，不引入任何外部音频/字体/脚本文件
- 语音提示和白噪音必须是浏览器原生 API 实时生成，不得嵌入音频文件
- 中英双语切换，右上角 `中/EN` 按钮，语言选择存 `localStorage['infpSleepLang']`
- 视觉沿用 unstuck-mobile.html 的 CSS 变量结构（`--bg` `--surface` `--ink` `--ink-muted` `--accent-calm` `--accent-action` `--border` `--on-accent`），同时支持系统亮暗色模式和 `data-theme` 覆盖，但配色换成夜间靛蓝/淡紫色调
- 流程是单向顺序前进，没有"返回上一步"功能；所有计时器都是提示性的，不阻止用户点击"下一步"提前跳过
- **只发布到 GitHub Pages（仓库 `slowrecover/infp-sleep`），不发布 Claude Artifact**
- 项目文件夹：`C:\Users\lucky\ideas\projects\infp-sleep\`，工具源文件为 `infp-sleep-mobile.html`

---

### Task 1: 创建完整的 infp-sleep-mobile.html

**Files:**
- Create: `C:\Users\lucky\ideas\projects\infp-sleep\infp-sleep-mobile.html`

**Interfaces:**
- Produces: 一个独立可打开的 HTML 文件，包含全部六屏逻辑、i18n、计时器、语音提示、白噪音、记录生成

- [ ] **Step 1: 创建 HTML 源文件**

新建文件 `C:\Users\lucky\ideas\projects\infp-sleep\infp-sleep-mobile.html`，完整内容：

```html
<title>INFP 安眠 / INFP Sleep</title>
<style>
  :root {
    --bg: #E7EAF2;
    --surface: #FFFFFF;
    --ink: #1B2036;
    --ink-muted: #5B6178;
    --accent-calm: #35406B;
    --accent-action: #7A6BA6;
    --border: #D6D9E6;
    --on-accent: #FFFFFF;
  }
  @media (prefers-color-scheme: dark) {
    :root {
      --bg: #0E1220;
      --surface: #171B2E;
      --ink: #E4E7F5;
      --ink-muted: #9BA0BE;
      --accent-calm: #7C8FC9;
      --accent-action: #B49CE0;
      --border: #2A3050;
      --on-accent: #0E1220;
    }
  }
  :root[data-theme="dark"] {
    --bg: #0E1220;
    --surface: #171B2E;
    --ink: #E4E7F5;
    --ink-muted: #9BA0BE;
    --accent-calm: #7C8FC9;
    --accent-action: #B49CE0;
    --border: #2A3050;
    --on-accent: #0E1220;
  }
  :root[data-theme="light"] {
    --bg: #E7EAF2;
    --surface: #FFFFFF;
    --ink: #1B2036;
    --ink-muted: #5B6178;
    --accent-calm: #35406B;
    --accent-action: #7A6BA6;
    --border: #D6D9E6;
    --on-accent: #FFFFFF;
  }
  * { box-sizing: border-box; }
  body {
    margin: 0;
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    background: var(--bg);
    color: var(--ink);
    font-family: "PingFang SC", "Noto Sans SC", "Microsoft YaHei", system-ui, sans-serif;
    padding: 24px;
  }
  .card {
    width: 100%;
    max-width: 420px;
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 20px;
    padding: 28px 24px;
    display: flex;
    flex-direction: column;
    gap: 20px;
  }
  .topbar { display: flex; align-items: center; justify-content: space-between; }
  .dots { display: flex; gap: 8px; justify-content: center; flex: 1; }
  .dot { width: 8px; height: 8px; border-radius: 50%; background: var(--border); transition: background 0.2s; }
  .dot.active { background: var(--accent-calm); }
  .dot.done { background: var(--accent-action); }
  .lang-toggle {
    border: 1px solid var(--border);
    background: transparent;
    color: var(--ink-muted);
    border-radius: 999px;
    padding: 4px 10px;
    font-size: 0.75rem;
    font-weight: 600;
    font-family: inherit;
    cursor: pointer;
  }
  .lang-toggle:hover { border-color: var(--accent-calm); color: var(--accent-calm); }
  .lang-toggle:focus-visible { outline: 2px solid var(--accent-calm); outline-offset: 2px; }
  .step { display: none; flex-direction: column; gap: 16px; }
  .step.active { display: flex; }
  h1 {
    font-family: "Songti SC", "Noto Serif SC", serif;
    font-size: 1.3rem;
    font-weight: 600;
    text-wrap: balance;
    margin: 0;
    color: var(--ink);
  }
  p.hint { margin: 0; color: var(--ink-muted); font-size: 0.9rem; line-height: 1.5; }
  input[type="text"], textarea {
    width: 100%;
    font-size: 1rem;
    font-family: inherit;
    color: var(--ink);
    background: var(--bg);
    border: 1px solid var(--border);
    border-radius: 12px;
    padding: 12px 14px;
    resize: none;
  }
  textarea { min-height: 120px; }
  input[type="text"]:focus, textarea:focus { outline: 2px solid var(--accent-calm); outline-offset: 1px; }
  .chips { display: flex; flex-wrap: wrap; gap: 8px; }
  .chip {
    border: 1px solid var(--border);
    background: transparent;
    color: var(--ink-muted);
    border-radius: 999px;
    padding: 6px 12px;
    font-size: 0.85rem;
    cursor: pointer;
    font-family: inherit;
  }
  .chip:hover { border-color: var(--accent-calm); color: var(--accent-calm); }
  .chip.selected { border-color: var(--accent-action); color: var(--accent-action); }
  .chip:focus-visible { outline: 2px solid var(--accent-calm); outline-offset: 2px; }
  button.primary {
    font-size: 1rem;
    font-weight: 600;
    font-family: inherit;
    color: var(--on-accent);
    background: var(--accent-calm);
    border: none;
    border-radius: 12px;
    padding: 14px;
    cursor: pointer;
  }
  button.primary:focus-visible { outline: 2px solid var(--ink); outline-offset: 2px; }
  button.secondary {
    font-size: 0.95rem;
    font-weight: 600;
    font-family: inherit;
    color: var(--accent-action);
    background: transparent;
    border: 1px solid var(--border);
    border-radius: 12px;
    padding: 12px;
    cursor: pointer;
  }
  .timer {
    font-family: ui-monospace, "SF Mono", "Cascadia Code", Consolas, monospace;
    font-variant-numeric: tabular-nums;
    font-size: 2.2rem;
    text-align: center;
    color: var(--accent-action);
  }
  .sub-timer { border: 1px solid var(--border); border-radius: 14px; padding: 14px; display: flex; flex-direction: column; gap: 10px; }
  .breathe-circle {
    width: 140px;
    height: 140px;
    border-radius: 50%;
    background: var(--accent-calm);
    opacity: 0.85;
    margin: 8px auto;
    transform: scale(0.6);
    transition: transform 4s ease-in-out, opacity 4s ease-in-out;
  }
  .breathe-circle.expand { transform: scale(1.0); opacity: 1; }
  .summary-box {
    font-family: ui-monospace, "SF Mono", Consolas, monospace;
    font-size: 0.85rem;
    background: var(--bg);
    border: 1px solid var(--border);
    border-radius: 12px;
    padding: 12px 14px;
    white-space: pre-wrap;
    word-break: break-word;
    color: var(--ink);
  }
  .copied { color: var(--accent-action); font-size: 0.85rem; text-align: center; min-height: 1.2em; }
  @media (prefers-reduced-motion: no-preference) {
    .step.active { animation: fade 0.25s ease; }
  }
  @keyframes fade {
    from { opacity: 0; transform: translateY(4px); }
    to { opacity: 1; transform: translateY(0); }
  }
</style>

<div class="card">
  <div class="topbar">
    <div class="dots" id="dots"></div>
    <button class="lang-toggle" id="langToggle" type="button">EN</button>
  </div>

  <div class="step active" data-step="0">
    <h1 data-i18n="openTitle"></h1>
    <p class="hint" data-i18n="openHint"></p>
    <button class="primary" data-next="1" data-i18n="start"></button>
  </div>

  <div class="step" data-step="1">
    <h1 data-i18n="s1Title"></h1>
    <div class="sub-timer">
      <p class="hint" data-i18n="s1Brush"></p>
      <div class="timer" id="brushTimer">3:00</div>
      <button class="secondary" id="brushStart" data-i18n="s1StartBrush"></button>
    </div>
    <div class="sub-timer">
      <p class="hint" data-i18n="s1Feet"></p>
      <div class="timer" id="feetTimer">3:00</div>
      <button class="secondary" id="feetStart" data-i18n="s1StartFeet"></button>
    </div>
    <button class="primary" data-next="2" data-i18n="next"></button>
  </div>

  <div class="step" data-step="2">
    <h1 data-i18n="s2Title"></h1>
    <p class="hint" data-i18n="s2Hint"></p>
    <textarea id="dumpInput" data-i18n-placeholder="s2Placeholder"></textarea>
    <button class="primary" data-next="3" data-i18n="s2Done"></button>
  </div>

  <div class="step" data-step="3">
    <h1 data-i18n="s3Title"></h1>
    <p class="hint" data-i18n="s3Hint"></p>
    <div class="chips" id="durationChips"></div>
    <div class="chips">
      <button class="chip" id="ttsToggle" type="button"></button>
      <button class="chip" id="noiseToggle" type="button"></button>
    </div>
    <div class="breathe-circle" id="breatheCircle"></div>
    <div class="timer" id="meditationTimer">10:00</div>
    <button class="secondary" id="meditationStart" data-i18n="s3Start"></button>
    <button class="primary" data-next="4" data-i18n="next"></button>
  </div>

  <div class="step" data-step="4">
    <h1 data-i18n="s4Title"></h1>
    <p class="hint" data-i18n="s4Hint"></p>
    <input type="text" id="gratitude1" data-i18n-placeholder="s4Placeholder1" />
    <input type="text" id="gratitude2" data-i18n-placeholder="s4Placeholder2" />
    <input type="text" id="gratitude3" data-i18n-placeholder="s4Placeholder3" />
    <button class="primary" data-next="5" data-i18n="next"></button>
  </div>

  <div class="step" data-step="5">
    <h1 data-i18n="s5Title"></h1>
    <p class="hint" data-i18n="s5Hint"></p>
    <div class="summary-box" id="summaryBox"></div>
    <div class="copied" id="copiedMsg"></div>
    <button class="primary" id="copyBtn" data-i18n="copy"></button>
    <button class="secondary" id="restartBtn" data-i18n="restart"></button>
  </div>
</div>

<script>
(function () {
  var i18n = {
    zh: {
      langToggle: "EN",
      openTitle: "开始之前",
      openHint: "先把明天要做的事写下来，记在哪里都可以，提前为明天做个打算。接下来的五步，就不用再惦记这些了。",
      start: "开始",
      s1Title: "洗漱",
      s1Brush: "刷牙 3 分钟",
      s1StartBrush: "开始刷牙计时",
      s1Feet: "洗脚 3 分钟",
      s1StartFeet: "开始洗脚计时",
      s2Title: "清空大脑",
      s2Hint: "这里写的都是垃圾，写完就倒掉，不会被保存，随便写。",
      s2Placeholder: "脑子里在转什么，都写下来……",
      s2Done: "倒空了",
      s3Title: "冥想",
      s3Hint: "选一个时长，安静下来。",
      s3VoiceLabel: "语音提示",
      s3NoiseLabel: "水流白噪音",
      onWord: "开",
      offWord: "关",
      s3Start: "开始冥想",
      meditationStartRunning: "冥想中……",
      breatheIn: "吸气",
      breatheOut: "呼气",
      s4Title: "感恩三件事",
      s4Hint: "今天有什么值得感恩的？",
      s4Placeholder1: "第一件……",
      s4Placeholder2: "第二件……",
      s4Placeholder3: "第三件……",
      s5Title: "看书",
      s5Hint: "把手机放到客厅，去看书。下面这行记录可以复制粘贴进 raw.md：",
      copy: "复制",
      copied: "已复制",
      restart: "再来一次",
      next: "下一步",
      durations: [3, 5, 10, 15],
      durationLabel: "分钟"
    },
    en: {
      langToggle: "中",
      openTitle: "Before you begin",
      openHint: "Write tomorrow's to-dos in your own to-do app first. The next five steps won't ask you to hold onto them.",
      start: "Start",
      s1Title: "Wash up",
      s1Brush: "Brush teeth, 3 minutes",
      s1StartBrush: "Start brushing timer",
      s1Feet: "Soak/wash feet, 3 minutes",
      s1StartFeet: "Start foot timer",
      s2Title: "Empty your mind",
      s2Hint: "Everything you write here is trash — it gets thrown out, not saved. Write freely.",
      s2Placeholder: "Whatever's spinning in your head, write it out...",
      s2Done: "Emptied",
      s3Title: "Meditate",
      s3Hint: "Pick a duration and settle down.",
      s3VoiceLabel: "Voice cues",
      s3NoiseLabel: "Water noise",
      onWord: "on",
      offWord: "off",
      s3Start: "Start meditating",
      meditationStartRunning: "Meditating...",
      breatheIn: "Breathe in",
      breatheOut: "Breathe out",
      s4Title: "Three things you're grateful for",
      s4Hint: "What's worth being grateful for today?",
      s4Placeholder1: "First...",
      s4Placeholder2: "Second...",
      s4Placeholder3: "Third...",
      s5Title: "Read",
      s5Hint: "Put your phone in the living room and go read. Paste this line into your log:",
      copy: "Copy",
      copied: "Copied",
      restart: "Start over",
      next: "Next",
      durations: [3, 5, 10, 15],
      durationLabel: "min"
    }
  };

  var lang = localStorage.getItem("infpSleepLang") || "zh";
  var totalSteps = 5;
  var state = { gratitude1: "", gratitude2: "", gratitude3: "" };
  var meditationMinutes = 10;
  var meditationInterval = null;
  var meditationSecondsLeft = 0;
  var breathInterval = null;
  var ttsOn = false;
  var noiseOn = false;
  var audioCtx = null;
  var noiseSource = null;
  var noiseGain = null;
  var brushInterval = null;
  var feetInterval = null;

  var dotsEl = document.getElementById("dots");
  for (var i = 1; i <= totalSteps; i++) {
    var d = document.createElement("div");
    d.className = "dot" + (i === 1 ? " active" : "");
    d.dataset.dot = i;
    dotsEl.appendChild(d);
  }

  function applyLanguage() {
    var dict = i18n[lang];
    document.querySelectorAll("[data-i18n]").forEach(function (el) {
      var key = el.dataset.i18n;
      if (dict[key] !== undefined) el.textContent = dict[key];
    });
    document.querySelectorAll("[data-i18n-placeholder]").forEach(function (el) {
      var key = el.dataset.i18nPlaceholder;
      if (dict[key] !== undefined) el.placeholder = dict[key];
    });
    document.getElementById("langToggle").textContent = dict.langToggle;
    document.documentElement.lang = lang === "zh" ? "zh-CN" : "en";
    renderDurationChips();
    updateToggleLabels();
  }

  document.getElementById("langToggle").addEventListener("click", function () {
    lang = lang === "zh" ? "en" : "zh";
    localStorage.setItem("infpSleepLang", lang);
    applyLanguage();
  });

  function showStep(n) {
    stopMeditation();
    document.querySelectorAll(".step").forEach(function (s) {
      s.classList.toggle("active", s.dataset.step === String(n));
    });
    document.querySelectorAll(".dot").forEach(function (d) {
      var idx = Number(d.dataset.dot);
      d.classList.toggle("active", idx === n);
      d.classList.toggle("done", idx < n);
    });
  }

  document.querySelectorAll("[data-next]").forEach(function (btn) {
    btn.addEventListener("click", function () {
      state.gratitude1 = document.getElementById("gratitude1").value.trim();
      state.gratitude2 = document.getElementById("gratitude2").value.trim();
      state.gratitude3 = document.getElementById("gratitude3").value.trim();
      var next = Number(btn.dataset.next);
      if (next === 5) finalizeEntry();
      showStep(next);
    });
  });

  // Step 1: brush + feet timers
  function startCountdown(displayId, seconds, onDone) {
    var el = document.getElementById(displayId);
    var remaining = seconds;
    function tick() {
      var m = Math.floor(remaining / 60);
      var s = remaining % 60;
      el.textContent = m + ":" + (s < 10 ? "0" : "") + s;
    }
    tick();
    var interval = setInterval(function () {
      remaining--;
      tick();
      if (remaining <= 0) {
        clearInterval(interval);
        if (onDone) onDone();
      }
    }, 1000);
    return interval;
  }

  document.getElementById("brushStart").addEventListener("click", function () {
    if (brushInterval) return;
    brushInterval = startCountdown("brushTimer", 180, function () { brushInterval = null; });
  });
  document.getElementById("feetStart").addEventListener("click", function () {
    if (feetInterval) return;
    feetInterval = startCountdown("feetTimer", 180, function () { feetInterval = null; });
  });

  // Step 3: meditation duration chips
  function renderDurationChips() {
    var dict = i18n[lang];
    var el = document.getElementById("durationChips");
    el.innerHTML = "";
    dict.durations.forEach(function (mins) {
      var b = document.createElement("button");
      b.className = "chip" + (mins === meditationMinutes ? " selected" : "");
      b.type = "button";
      b.textContent = mins + " " + dict.durationLabel;
      b.addEventListener("click", function () {
        meditationMinutes = mins;
        document.getElementById("meditationTimer").textContent = mins + ":00";
        renderDurationChips();
      });
      el.appendChild(b);
    });
  }

  function updateToggleLabels() {
    var dict = i18n[lang];
    document.getElementById("ttsToggle").textContent = dict.s3VoiceLabel + "：" + (ttsOn ? dict.onWord : dict.offWord);
    document.getElementById("noiseToggle").textContent = dict.s3NoiseLabel + "：" + (noiseOn ? dict.onWord : dict.offWord);
  }

  document.getElementById("ttsToggle").addEventListener("click", function () {
    ttsOn = !ttsOn;
    updateToggleLabels();
  });
  document.getElementById("noiseToggle").addEventListener("click", function () {
    noiseOn = !noiseOn;
    updateToggleLabels();
    if (!noiseOn) stopNoise();
  });

  function startNoise() {
    if (audioCtx) return;
    try {
      audioCtx = new (window.AudioContext || window.webkitAudioContext)();
      var bufferSize = 2 * audioCtx.sampleRate;
      var buffer = audioCtx.createBuffer(1, bufferSize, audioCtx.sampleRate);
      var data = buffer.getChannelData(0);
      for (var i = 0; i < bufferSize; i++) data[i] = Math.random() * 2 - 1;
      noiseSource = audioCtx.createBufferSource();
      noiseSource.buffer = buffer;
      noiseSource.loop = true;
      var filter = audioCtx.createBiquadFilter();
      filter.type = "lowpass";
      filter.frequency.value = 900;
      noiseGain = audioCtx.createGain();
      noiseGain.gain.value = 0.12;
      noiseSource.connect(filter);
      filter.connect(noiseGain);
      noiseGain.connect(audioCtx.destination);
      noiseSource.start(0);
    } catch (e) {}
  }

  function stopNoise() {
    if (noiseSource) {
      try { noiseSource.stop(); } catch (e) {}
      noiseSource.disconnect();
      noiseSource = null;
    }
    if (audioCtx) {
      try { audioCtx.close(); } catch (e) {}
      audioCtx = null;
    }
  }

  function speak(text) {
    if (!ttsOn || !window.speechSynthesis) return;
    try {
      var utter = new SpeechSynthesisUtterance(text);
      utter.lang = lang === "zh" ? "zh-CN" : "en-US";
      window.speechSynthesis.speak(utter);
    } catch (e) {}
  }

  function startBreathing() {
    var circle = document.getElementById("breatheCircle");
    var expanded = false;
    function cycle() {
      expanded = !expanded;
      circle.classList.toggle("expand", expanded);
      speak(expanded ? i18n[lang].breatheIn : i18n[lang].breatheOut);
    }
    cycle();
    breathInterval = setInterval(cycle, 4000);
  }

  function stopBreathing() {
    if (breathInterval) { clearInterval(breathInterval); breathInterval = null; }
    document.getElementById("breatheCircle").classList.remove("expand");
  }

  document.getElementById("meditationStart").addEventListener("click", function () {
    if (meditationInterval) return;
    meditationSecondsLeft = meditationMinutes * 60;
    if (noiseOn) startNoise();
    startBreathing();
    var el = document.getElementById("meditationTimer");
    function tick() {
      var m = Math.floor(meditationSecondsLeft / 60);
      var s = meditationSecondsLeft % 60;
      el.textContent = m + ":" + (s < 10 ? "0" : "") + s;
    }
    tick();
    meditationInterval = setInterval(function () {
      meditationSecondsLeft--;
      tick();
      if (meditationSecondsLeft <= 0) stopMeditation();
    }, 1000);
  });

  function stopMeditation() {
    if (meditationInterval) { clearInterval(meditationInterval); meditationInterval = null; }
    stopBreathing();
    stopNoise();
  }

  function pad(n) { return n < 10 ? "0" + n : String(n); }

  function formatLine() {
    var d = new Date();
    var ts = d.getFullYear() + "-" + pad(d.getMonth() + 1) + "-" + pad(d.getDate()) + " " + pad(d.getHours()) + ":" + pad(d.getMinutes());
    return "- [" + ts + "] [sleep] 感恩:" + state.gratitude1 + "；" + state.gratitude2 + "；" + state.gratitude3;
  }

  function finalizeEntry() {
    var line = formatLine();
    document.getElementById("summaryBox").textContent = line;
    try {
      var log = JSON.parse(localStorage.getItem("infpSleepLog") || "[]");
      log.push(line);
      localStorage.setItem("infpSleepLog", JSON.stringify(log));
    } catch (e) {}
  }

  document.getElementById("copyBtn").addEventListener("click", function () {
    var text = document.getElementById("summaryBox").textContent;
    var msg = document.getElementById("copiedMsg");
    function showCopied() {
      msg.textContent = i18n[lang].copied;
      setTimeout(function () { msg.textContent = ""; }, 2000);
    }
    if (navigator.clipboard && navigator.clipboard.writeText) {
      navigator.clipboard.writeText(text).then(showCopied).catch(function () { fallbackCopy(text, showCopied); });
    } else {
      fallbackCopy(text, showCopied);
    }
  });

  function fallbackCopy(text, cb) {
    var ta = document.createElement("textarea");
    ta.value = text;
    ta.style.position = "fixed";
    ta.style.opacity = "0";
    document.body.appendChild(ta);
    ta.select();
    try { document.execCommand("copy"); } catch (e) {}
    document.body.removeChild(ta);
    cb();
  }

  document.getElementById("restartBtn").addEventListener("click", function () {
    state = { gratitude1: "", gratitude2: "", gratitude3: "" };
    document.getElementById("gratitude1").value = "";
    document.getElementById("gratitude2").value = "";
    document.getElementById("gratitude3").value = "";
    document.getElementById("dumpInput").value = "";
    document.getElementById("brushTimer").textContent = "3:00";
    document.getElementById("feetTimer").textContent = "3:00";
    if (brushInterval) { clearInterval(brushInterval); brushInterval = null; }
    if (feetInterval) { clearInterval(feetInterval); feetInterval = null; }
    meditationMinutes = 10;
    document.getElementById("meditationTimer").textContent = "10:00";
    renderDurationChips();
    showStep(0);
  });

  applyLanguage();
})();
</script>
```

- [ ] **Step 2: 在浏览器中打开文件，走一遍开场屏**

在文件管理器里双击 `infp-sleep-mobile.html`（或用浏览器打开该文件路径）。

预期：看到"开始之前"提示 + "开始"按钮；右上角 `EN` 按钮点击后整个开场屏切换成英文，再点一次切回中文。点击"开始"后进入步骤1"洗漱"。

- [ ] **Step 3: 验证步骤1（洗漱）两段计时器**

点击"开始刷牙计时"，确认倒计时从 3:00 开始递减；点击"开始洗脚计时"，确认第二个倒计时独立递减，互不干扰。点击"下一步"，无需等计时器走完即可前进（计时器是提示性的）。

- [ ] **Step 4: 验证步骤2（清空大脑）**

确认看到"这里写的都是垃圾，写完就倒掉，不会被保存，随便写"的提示文字。在文本框里输入几句话，点击"倒空了"进入步骤3。刷新页面重新走一遍，确认之前输入的内容不会再出现（没有被保存/回填）。

- [ ] **Step 5: 验证步骤3（冥想）**

点击不同的时长词条（3/5/10/15 分钟），确认选中态高亮切换，且倒计时显示框的初始时间跟着变化。点击"开始冥想"，确认：
- 倒计时开始递减
- 中间的圆形按 4 秒一次的节奏放大缩小（呼吸动画）
- 点击"语音提示"词条切换为"开"后再次点击"开始冥想"，确认设备读出"吸气"/"呼气"（需要设备本身支持语音合成且未静音）
- 点击"水流白噪音"词条切换为"开"后再次点击"开始冥想"，确认能听到持续的白噪音（需要设备音量开启）
- 点击"下一步"后，确认呼吸动画、语音、白噪音都停止（没有继续在后台播放）

- [ ] **Step 6: 验证步骤4（感恩三件事）和步骤5（看书/记录）**

在三个输入框里各写一句话，点击"下一步"进入最后一屏。确认：
- 提示文字包含"把手机放到客厅，去看书"
- 记录框内容格式为 `- [今天日期 当前时间] [sleep] 感恩:<第一句>；<第二句>；<第三句>`
- 点击"复制"按钮，出现"已复制"提示，粘贴到别处验证内容正确
- 点击"再来一次"，确认所有输入清空、计时器重置、回到开场屏（步骤0）

- [ ] **Step 7: 验证 localStorage 持久化**

走完一次完整流程后，在浏览器开发者工具 Console 里运行：
```javascript
JSON.parse(localStorage.getItem("infpSleepLog"))
```
预期：返回一个数组，包含刚才生成的记录字符串。

- [ ] **Step 8: Commit**

```bash
cd "C:\Users\lucky\ideas\projects\infp-sleep"
git add infp-sleep-mobile.html
git commit -m "add infp-sleep-mobile.html: full bedtime ritual tool"
```

---

### Task 2: 发布到 GitHub Pages

**Files:**
- 无新文件；操作对象是已存在的 GitHub 仓库 `https://github.com/slowrecover/infp-sleep`（在设计阶段已创建并推送过 `design.md`）

**Interfaces:**
- Consumes: Task 1 产出的 `infp-sleep-mobile.html`
- Produces: 一个可通过手机浏览器直接访问的 GitHub Pages 网址

- [ ] **Step 1: 把工具文件复制为仓库根目录的 index.html 并推送**

GitHub Pages 默认从分支根目录的 `index.html` 提供服务，所以需要在仓库里放一份改名后的副本（项目文件夹里保留原名 `infp-sleep-mobile.html` 作为源文件）：

```bash
cp "C:\Users\lucky\ideas\projects\infp-sleep\infp-sleep-mobile.html" "C:\Users\lucky\ideas\projects\infp-sleep\index.html"
cd "C:\Users\lucky\ideas\projects\infp-sleep"
git add index.html
git commit -m "add index.html copy for GitHub Pages"
git push
```

- [ ] **Step 2: 开启 GitHub Pages**

```bash
gh api repos/slowrecover/infp-sleep/pages -X POST -f "source[branch]=master" -f "source[path]=/"
```

预期：返回 JSON，其中 `html_url` 字段是 `https://slowrecover.github.io/infp-sleep/`。

- [ ] **Step 3: 验证线上页面**

等待约 30-60 秒后运行：
```bash
curl -s -o /dev/null -w "%{http_code}" https://slowrecover.github.io/infp-sleep/
```
预期输出 `200`。再运行：
```bash
curl -s https://slowrecover.github.io/infp-sleep/ | head -c 200
```
预期能看到 `<title>INFP 安眠 / INFP Sleep</title>`，确认线上内容和本地一致。

用手机浏览器实际打开 `https://slowrecover.github.io/infp-sleep/`，走一遍 Task 1 里验证过的完整流程（开场→洗漱→清空大脑→冥想→感恩→看书/复制），确认手机上同样正常工作，然后可以"添加到主屏幕"。

---

## 注意事项

- 不发布 Claude Artifact——用户已明确要求这个工具只上线到 GitHub Pages
- `index.html` 和 `infp-sleep-mobile.html` 内容需要保持同步：以后修改工具时，先改 `infp-sleep-mobile.html`，再复制覆盖 `index.html`，一起提交推送
- Web Speech API 和 Web Audio API 的实际表现因浏览器/操作系统而异（部分浏览器需要用户先有一次交互动作才允许播放音频，本设计已通过"开始冥想"按钮点击触发，满足这个要求）
