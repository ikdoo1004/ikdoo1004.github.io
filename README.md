<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Colorful Roulette</title>
  <style>
    body {
      font-family: sans-serif;
      display: flex;
      flex-direction: column;
      align-items: center;
      margin-top: 30px;
      background: #1e1e1e;
      color: #fff;
    }
    canvas {
      border-radius: 50%;
      border: 8px solid #555;
      box-shadow: 0 0 30px rgba(255, 255, 255, 0.2);
      transition: transform 0.8s ease;
      z-index: 0;
      margin-bottom: 40px;
    }
    #controls {
      margin-bottom: 30px;
      display: flex;
      gap: 10px;
    }
    input {
      padding: 10px;
      border-radius: 6px;
      border: none;
      font-size: 18px;
      width: 220px;
    }
    button {
      padding: 10px 20px;
      font-size: 18px;
      border: none;
      border-radius: 6px;
      cursor: pointer;
      background: #28a745;
      color: white;
    }
    #rollBtn {
      margin-top: 40px;
      font-size: 22px;
      padding: 14px 40px;
      background: #007bff;
    }
    #arrow {
      position: absolute;
      width: 0;
      height: 0;
      border-left: 15px solid transparent;
      border-right: 15px solid transparent;
      border-bottom: 30px solid red;
      top: 0;
      left: calc(50% - 15px);
      z-index: 2;
    }
    #winnerBox {
      position: absolute;
      top: 150px;
      left: 0;
      right: 0;
      text-align: center;
      font-size: 40px;
      font-weight: bold;
      color: white;
      background: rgba(0, 0, 0, 0.85);
      padding: 25px;
      display: none;
      z-index: 3;
      border-radius: 10px;
      box-shadow: 0 0 20px #000;
    }
  </style>
  <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
</head>
<body>
  <div id="arrow"></div>
  <canvas id="roulette" width="400" height="400"></canvas>
  <div id="winnerBox"></div>

  <div id="controls">
    <input type="text" id="optionInput" placeholder="Enter option..." />
    <button onclick="addOption()">Add</button>
  </div>

  <button id="rollBtn" onclick="spinWheel()">Roll!</button>

  <script>
    const canvas = document.getElementById('roulette');
    const ctx = canvas.getContext('2d');
    const winnerBox = document.getElementById('winnerBox');
    let options = [];
    let angle = 0;
    let isSpinning = false;
    const maxOptions = 8;

    function drawWheel() {
      const radius = canvas.width / 2;
      const centerX = canvas.width / 2;
      const centerY = canvas.height / 2;
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      const angleStep = 2 * Math.PI / options.length;

      options.forEach((opt, i) => {
        const startAngle = angle + i * angleStep;
        const endAngle = startAngle + angleStep;
        ctx.beginPath();
        ctx.moveTo(centerX, centerY);
        ctx.arc(centerX, centerY, radius, startAngle, endAngle);
        ctx.fillStyle = opt.color;
        ctx.fill();

        const rgb = hexToRgb(opt.color);
        const brightness = (rgb.r * 299 + rgb.g * 587 + rgb.b * 114) / 1000;
        const textColor = brightness > 125 ? "black" : "white";

        ctx.save();
        ctx.translate(centerX, centerY);
        ctx.rotate(startAngle + angleStep / 2);
        ctx.textAlign = "right";
        ctx.fillStyle = textColor;
        ctx.font = "18px sans-serif";
        ctx.fillText(opt.label, radius - 10, 10);
        ctx.restore();
      });
    }

    function hexToRgb(hex) {
      let r, g, b;
      if (hex.startsWith("hsl")) {
        const [h, s, l] = hex.match(/\d+/g).map(Number);
        [r, g, b] = hslToRgb(h / 360, s / 100, l / 100);
        return { r: r * 255, g: g * 255, b: b * 255 };
      }
      return { r: 0, g: 0, b: 0 };
    }

    function hslToRgb(h, s, l) {
      let r, g, b;
      if (s === 0) {
        r = g = b = l;
      } else {
        const hue2rgb = (p, q, t) => {
          if (t < 0) t += 1;
          if (t > 1) t -= 1;
          if (t < 1 / 6) return p + (q - p) * 6 * t;
          if (t < 1 / 2) return q;
          if (t < 2 / 3) return p + (q - p) * (2 / 3 - t) * 6;
          return p;
        };
        const q = l < 0.5 ? l * (1 + s) : l + s - l * s;
        const p = 2 * l - q;
        r = hue2rgb(p, q, h + 1 / 3);
        g = hue2rgb(p, q, h);
        b = hue2rgb(p, q, h - 1 / 3);
      }
      return [r, g, b];
    }

    function getRandomColor() {
      return `hsl(${Math.floor(Math.random() * 360)}, 80%, 60%)`;
    }

    function addOption() {
      const input = document.getElementById('optionInput');
      const label = input.value.trim();
      if (label && options.length < maxOptions) {
        options.push({ label, color: getRandomColor() });
        input.value = '';
        drawWheel();
      }
    }

    function spinWheel() {
      if (isSpinning || options.length === 0) return;
      isSpinning = true;
      winnerBox.style.display = 'none';
      canvas.style.transform = 'scale(1)';

      let velocity = Math.random() * 0.3 + 0.25;
      const deceleration = 0.002;

      const spinner = setInterval(() => {
        angle += velocity;
        velocity -= deceleration;
        drawWheel();

        if (velocity <= 0) {
          clearInterval(spinner);
          isSpinning = false;

          const normalizedAngle = (angle % (2 * Math.PI));
          const anglePerSegment = 2 * Math.PI / options.length;
          const index = Math.floor(((Math.PI * 3 / 2 - normalizedAngle + 2 * Math.PI) % (2 * Math.PI)) / anglePerSegment);
          const winner = options[index];

          setTimeout(() => {
            canvas.style.transform = 'scale(1.4)';
            winnerBox.textContent = winner.label + '!';
            winnerBox.style.display = 'block';

            confetti({
              particleCount: 100,
              spread: 70,
              origin: { y: 0.4 }
            });

            setTimeout(() => {
              canvas.style.transform = 'scale(1)';
            }, 3000);

          }, 300);
        }
      }, 16);
    }
  </script>
</body>
</html>
