[trungthu.html](https://github.com/user-attachments/files/22703910/trungthu.html)
<!doctype html>
<html lang="vi">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Trung Thu</title>
  <style>
    :root {
      --accent: #ff7f50;
      --accent-2: #ff5722;
    }

    html, body {
      height: 100%;
      margin: 0;
      font-family: Inter, "Segoe UI", system-ui, sans-serif;
      background: #060614;
      overflow: hidden;
    }

    .stage {
      position: relative;
      min-height: 100vh;
      overflow: hidden;
    }

    canvas#starfield {
      display: block;
      width: 100%;
      height: 100%;
    }

    .moon {
      position: fixed;
      top: 20px;
      right: 20px;
      width: 90px;
      height: 90px;
      background: radial-gradient(circle at 30% 30%, #fff 0%, #f0f0f0 45%, #dcdcdc 100%);
      border-radius: 50%;
      box-shadow: 0 0 20px rgba(255, 255, 210, 0.6);
      pointer-events: none;
      z-index: 2;
    }

    #lantern-container {
      position: fixed;
      bottom: 0;
      left: 0;
      width: 100%;
      height: 100%;
      pointer-events: none;
      overflow: hidden;
      z-index: 10;
    }

    .lantern {
      position: absolute;
      pointer-events: auto;
      width: 40px;
      user-select: none;
    }

    @keyframes swing {
      0%   { transform: rotate(-5deg); }
      50%  { transform: rotate(5deg); }
      100% { transform: rotate(-5deg); }
    }

    .swing { animation: swing 2s ease-in-out infinite; }

    #wish-popup {
      position: fixed;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      background: rgba(255, 255, 255, 0.96);
      color: var(--accent-2);
      padding: 18px 14px;
      border-radius: 12px;
      font-size: 1rem;
      font-weight: 700;
      display: none;
      z-index: 9999;
      box-shadow: 0 4px 16px rgba(0, 0, 0, 0.3);
      text-align: center;
      max-width: 85%;
      box-sizing: border-box;
    }

    #hint {
      position: fixed;
      bottom: 10vh;
      left: 50%;
      transform: translateX(-50%);
      padding: 8px 14px;
      background: linear-gradient(90deg, rgba(255, 127, 80, 0.95), rgba(255, 87, 34, 0.95));
      color: white;
      border-radius: 999px;
      font-weight: 600;
      z-index: 9998;
      box-shadow: 0 4px 14px rgba(0, 0, 0, 0.25);
      opacity: 0;
      transition: opacity .35s ease;
      pointer-events: none;
      font-size: 0.9rem;
    }

    .controls {
      position: fixed;
      left: 12px;
      top: 12px;
      z-index: 10;
      color: #fff;
      font-size: 14px;
      background: rgba(0, 0, 0, 0.2);
      padding: 6px 10px;
      border-radius: 8px;
    }

    .music-toggle {
      cursor: pointer;
      margin-left: 8px;
      color: #fff;
      opacity: 0.95;
    }

    @media (max-width:480px) {
      .moon { width: 70px; height: 70px; top: 10px; right: 10px; }
      #wish-popup { font-size: 0.85rem; padding: 12px 10px; }
    }
  </style>
</head>
<body>
  <div class="stage">
    <canvas id="starfield"></canvas>
    <div class="moon"></div>
    <div id="lantern-container"></div>
    <div id="wish-popup"></div>
    <div id="hint">Chạm màn hình để mở nhạc và thả đèn ✨</div>
    <div class="controls">
      <span class="music-toggle" id="musicToggle"></span>
    </div>
  </div>

  <audio id="bg-music" preload="none" loop>
    <source src="filenhac.mp3" type="audio/mp3">
  </audio>

  <script>
    // ========== Sao & Sao băng ==========
    const canvas = document.getElementById('starfield');
    const ctx = canvas.getContext('2d');
    let w, h, stars = [], meteors = [];

    function resizeCanvas() {
      w = canvas.width = innerWidth;
      h = canvas.height = innerHeight;
      stars = [];
      const count = Math.round((w * h) / 5000); // giảm số lượng sao cho nhẹ
      for (let i = 0; i < count; i++) {
        stars.push({
          x: Math.random() * w,
          y: Math.random() * h,
          r: Math.random() * 0.9 + 0.1,
          a: Math.random() * 0.8 + 0.2,
          t: Math.random() * 0.02 + 0.002
        });
      }
    }

    function drawStars() {
      ctx.clearRect(0, 0, w, h);
      for (const s of stars) {
        s.a += (Math.random() > 0.5 ? 1 : -1) * s.t;
        s.a = Math.max(0.05, Math.min(1, s.a));
        ctx.beginPath();
        ctx.globalAlpha = s.a;
        ctx.fillStyle = '#fff';
        ctx.arc(s.x, s.y, s.r, 0, Math.PI * 2);
        ctx.fill();
      }
      ctx.globalAlpha = 1;
    }

    function createMeteor() {
      const startX = Math.random() * w;
      const startY = Math.random() * (h / 3);
      const speed = Math.random() * 6 + 4;
      meteors.push({
        x: startX, y: startY,
        vx: speed + 4, vy: speed / 2,
        len: Math.random() * 80 + 60, a: 1
      });
    }

    function drawMeteors() {
      for (let i = meteors.length - 1; i >= 0; i--) {
        const m = meteors[i];
        const x2 = m.x - m.len;
        const y2 = m.y - m.len / 2;
        const g = ctx.createLinearGradient(m.x, m.y, x2, y2);
        g.addColorStop(0, `rgba(255,255,255,${m.a})`);
        g.addColorStop(1, 'rgba(255,255,255,0)');
        ctx.strokeStyle = g;
        ctx.lineWidth = 2;
        ctx.beginPath();
        ctx.moveTo(m.x, m.y);
        ctx.lineTo(x2, y2);
        ctx.stroke();
        m.x += m.vx; m.y += m.vy; m.a -= 0.02;
        if (m.a <= 0 || m.x > w + 100 || m.y > h + 100) meteors.splice(i, 1);
      }
    }

    function loop() {
      drawStars();
      drawMeteors();
      if (meteors.length < 5 && Math.random() < 0.006) createMeteor();
      requestAnimationFrame(loop);
    }

    window.addEventListener('resize', resizeCanvas);
    resizeCanvas();
    loop();

    // ========== Đèn lồng ==========
    const lanternContainer = document.getElementById('lantern-container');
    const wishes = [
      "Anh chúc em một mùa trăng an lành, ấm áp 🌕",
      "Có em cạnh bên thì ngày nào cũng là ngày đặc biệt ✨",
      "Mong mọi ước mơ của em đều thành sự thật 💫"
    ];

    function createLantern() {
      if (lanternContainer.childElementCount >= 25) return; // giới hạn số lượng
      const lantern = document.createElement("img");
      lantern.src = "https://github.com/Panbap/Trungthu/blob/main/den.png?raw=true";
      lantern.className = "lantern swing";

      const layer = Math.floor(Math.random() * 3) + 1;
      let size = 40, duration = 10000, opacity = 0.8;

      if (layer === 1) { size = 20 + Math.random() * 20; duration = 14000; opacity = 0.5; }
      else if (layer === 2) { size = 30 + Math.random() * 30; duration = 12000; opacity = 0.7; }
      else { size = 40 + Math.random() * 30; duration = 10000; opacity = 0.9; }

      lantern.style.width = size + "px";
      lantern.style.left = Math.random() * 90 + "vw";
      lantern.style.bottom = "0px";
      lantern.style.opacity = opacity;

      lanternContainer.appendChild(lantern);

      const drift = Math.random() * 120 - 60;
      const up = 100 + Math.random() * 30;
      lantern.animate([
        { transform: "translate(0,0)", opacity: opacity },
        { transform: `translate(${drift}px, -${up}vh)`, opacity: 0 }
      ], { duration, easing: "linear", fill: "forwards" });

      setTimeout(() => lantern.remove(), duration);

      lantern.addEventListener("click", (e) => {
        e.stopPropagation();
        const wishPopup = document.getElementById('wish-popup');
        wishPopup.textContent = wishes[Math.floor(Math.random() * wishes.length)];
        wishPopup.style.display = 'block';
      });
    }

    // thay setInterval bằng loop đồng bộ
    let lastLantern = 0;
    function lanternLoop(ts) {
      if (ts - lastLantern > 1200) {
        createLantern();
        lastLantern = ts;
      }
      requestAnimationFrame(lanternLoop);
    }
    requestAnimationFrame(lanternLoop);

    // ========== Âm nhạc ==========
    const bg = document.getElementById('bg-music');
    const musicToggle = document.getElementById('musicToggle');
    let musicStarted = false;

    function startMusic() {
      if (musicStarted || !bg) return;
      bg.volume = 0.0;
      bg.play().then(() => {
        let v = 0;
        const i = setInterval(() => {
          v += 0.05;
          if (v >= 0.8) { v = 0.8; clearInterval(i); }
          bg.volume = v;
        }, 120);
        musicStarted = true;
        musicToggle.textContent = '🔊';
        const hint = document.getElementById('hint');
        hint.style.opacity = '1';
        setTimeout(() => hint.style.opacity = '0', 3000);
      }).catch(() => {});
    }

    musicToggle.addEventListener('click', (e) => {
      e.stopPropagation();
      if (!musicStarted) { startMusic(); return; }
      if (bg.paused) { bg.play(); musicToggle.textContent = '🔊'; }
      else { bg.pause(); musicToggle.textContent = '🔈'; }
    });

    document.addEventListener('pointerdown', function onceStart() {
      startMusic();
      document.removeEventListener('pointerdown', onceStart);
    });

    // click ra ngoài để tắt popup
    document.addEventListener('click', (e) => {
      const pop = document.getElementById('wish-popup');
      if (pop && pop.style.display === 'block') {
        pop.style.display = 'none';
      }
    });

    // tạo vài đèn ban đầu
    for (let i = 0; i < 3; i++) { setTimeout(createLantern, i * 800); }
  </script>
</body>
</html>
