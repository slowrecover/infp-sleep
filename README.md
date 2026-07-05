<div align="center">

# 🌙 INFP 安眠 / INFP Sleep

**知道该睡了，却放不下手机？一套睡前仪式，把手机放回客厅。**
**Know it's time to sleep but can't put the phone down? A bedtime ritual that ends with the phone leaving the room.**

[**→ 打开工具 / Open the tool ←**](https://slowrecover.github.io/infp-sleep/)

不联网 · 不登录 · 不留痕迹
No network · No login · No tracking

</div>

---

### 语言 / Language

[中文](#中文) · [English](#english)

---

## 中文

这不是一个"催你睡觉"的工具。它解决的是一个更具体的问题：**报复性熬夜**——白天没有一点真正属于自己的时间，晚上熬夜刷手机成了唯一能抢回来的自主感，越刷越舍不得放下。

这个工具不讲道理，直接给一套五步睡前仪式，走完之后手机就该放到客厅去了：

| 步骤 | 内容 |
|---|---|
| 🪥 洗漱 | 刷牙 3 分钟 + 洗脚 3 分钟，过程中每 10 秒轮换一句提示，把注意力拉回牙刷、水温这些具体的触感上 |
| 🗑️ 清空大脑 | 把脑子里转的念头写出来——这些字不会被保存，写完就是目的 |
| 🧘 冥想 | 自选 3/5/10/15 分钟，呼吸圆圈引导，可选开启语音提示和水流白噪音 |
| 🙏 感恩三件事 | 写下今天值得感恩的三件小事 |
| 📖 看书 | 提醒你把手机放到客厅，走完流程生成一行记录，可以复制到自己的笔记里 |

全程不联网、不用登录，所有内容只留在你自己手机的浏览器里。

**怎么用**

1. 手机浏览器打开 [slowrecover.github.io/infp-sleep](https://slowrecover.github.io/infp-sleep/)
2. 浏览器菜单里"添加到主屏幕"，以后像个小 App 一样点开用
3. 到了该睡觉却不想放手机的时候，打开它，走一遍五步

**背后的依据**

每一步都不是随便拍脑袋定的：

> 🦶 **暖脚能加速核心体温下降，是入睡的生理信号** — Kräuchi et al., *Nature* (1999)
> 📝 **睡前把念头写下来，比不写的人入睡明显更快** — Scullin et al. (2018, Baylor University)，认知卸载 / Zeigarnik 效应
> 🧘 **正念冥想降低睡前的认知和生理唤醒水平** — 失眠认知行为疗法（CBT-I）的标准组成部分
> 🙏 **睡前感恩书写能降低担忧、提升睡眠质量** — Digdon & Koble (2011, *Applied Psychology: Health and Well-Being*)
> 📵 **卧室不放手机、睡前阅读纸质书是经典睡眠卫生建议**

**说明**

- 纯前端页面，没有后端、没有数据库、不发任何网络请求
- 语音提示和水流白噪音都是浏览器原生 API 实时生成的（不是嵌入的音频文件）
- 记录只存在浏览器的 localStorage 里，换设备/清缓存就没了，请自己复制保存
- "清空大脑"那一步写的内容永远不会被保存——这是刻意的设计，不是 bug

---

## English

This isn't a tool that nags you to sleep. It targets something more specific: **revenge bedtime procrastination** — when your day leaves you with no time that's truly your own, and staying up scrolling becomes the only autonomy you get to claw back.

No lecture, just five concrete steps, ending with the phone leaving the room:

| Step | What it does |
|---|---|
| 🪥 Wash up | 3 min brushing + 3 min foot soak, with a new attention cue every 10 seconds pulling focus back to the physical sensations |
| 🗑️ Empty your mind | Write out whatever's spinning in your head — it's never saved, writing it is the whole point |
| 🧘 Meditate | Pick 3/5/10/15 minutes, a breathing circle to follow, optional voice cues and water-noise |
| 🙏 Three things you're grateful for | Write down three small things from today |
| 📖 Read | A reminder to leave your phone in the living room, plus a one-line summary you can copy into your own notes |

No network calls, no login — everything stays in your phone's browser.

**How to use it**

1. Open [slowrecover.github.io/infp-sleep](https://slowrecover.github.io/infp-sleep/) on your phone
2. Add it to your home screen from your browser menu
3. Open it whenever it's time to sleep but you can't put the phone down

**Why it works**

> 🦶 **Warm feet speed up the core-temperature drop that signals sleep onset** — Kräuchi et al., *Nature* (1999)
> 📝 **Writing down racing thoughts before bed helps you fall asleep faster than not writing** — Scullin et al. (2018, Baylor University); cognitive offloading / the Zeigarnik effect
> 🧘 **Mindfulness meditation lowers pre-sleep cognitive and physiological arousal** — a standard component of CBT-I (the gold-standard insomnia treatment)
> 🙏 **Gratitude writing before bed reduces worry and improves sleep quality** — Digdon & Koble (2011, *Applied Psychology: Health and Well-Being*)
> 📵 **No phone in the bedroom, read a physical book before sleep — classic sleep hygiene advice**

**Notes**

- Static front-end only — no backend, no database, no network requests
- Voice cues and water noise are generated live by native browser APIs, not embedded audio files
- Entries live in `localStorage` only — switch devices or clear your cache and they're gone
- The "empty your mind" step never saves what you write — that's intentional, not a bug
