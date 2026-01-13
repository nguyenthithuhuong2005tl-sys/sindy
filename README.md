<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cyber Runner - Cuộc Chạy Đua Tương Lai</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            background-color: #0a0a12;
            color: #fff;
            font-family: 'Segoe UI', sans-serif;
            overflow: hidden;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
        }

        #game-container {
            position: relative;
            width: 800px;
            height: 400px;
            background: #111122;
            border: 4px solid #00f2ff;
            border-radius: 10px;
            overflow: hidden;
            box-shadow: 0 0 30px rgba(0, 242, 255, 0.2);
        }

        canvas {
            display: block;
        }

        #ui-layer {
            position: absolute;
            top: 10;
            left: 0;
            width: 100%;
            padding: 20px;
            box-sizing: border-box;
            pointer-events: none;
            display: flex;
            justify-content: space-between;
            font-size: 24px;
            font-weight: bold;
            color: #00f2ff;
            text-shadow: 0 0 10px rgba(0, 242, 255, 0.7);
        }

        #overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(10, 10, 20, 0.9);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 100;
        }

        h1 {
            font-size: 60px;
            margin: 0;
            color: #ff00ff;
            text-transform: uppercase;
            letter-spacing: 5px;
            text-shadow: 3px 3px #00f2ff;
        }

        p {
            font-size: 18px;
            color: #aaa;
            margin: 20px 0;
        }

        .btn {
            padding: 15px 40px;
            font-size: 22px;
            background: transparent;
            color: #00f2ff;
            border: 2px solid #00f2ff;
            cursor: pointer;
            text-transform: uppercase;
            transition: 0.3s;
            border-radius: 5px;
        }

        .btn:hover {
            background: #00f2ff;
            color: #000;
            box-shadow: 0 0 20px #00f2ff;
        }

        .hidden { display: none; }
    </style>
</head>
<body>

    <div id="game-container">
        <canvas id="gameCanvas"></canvas>
        
        <div id="ui-layer">
            <div>ĐIỂM: <span id="scoreVal">0</span></div>
            <div>TỐC ĐỘ: <span id="speedVal">1</span>x</div>
        </div>

        <div id="overlay">
            <h1 id="overlayTitle">CYBER RUNNER</h1>
            <p id="overlayDesc">Nhấn PHÍM CÁCH (SPACE) hoặc Chạm để Nhảy</p>
            <button class="btn" id="startBtn">BẮT ĐẦU</button>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreEl = document.getElementById('scoreVal');
        const speedEl = document.getElementById('speedVal');
        const overlay = document.getElementById('overlay');
        const startBtn = document.getElementById('startBtn');

        canvas.width = 800;
        canvas.height = 400;

        // Trạng thái game
        let gameActive = false;
        let score = 0;
        let gameSpeed = 5;
        let frameCount = 0;

        // Nhân vật chính
        const runner = {
            x: 100,
            y: 300,
            width: 40,
            height: 60,
            dy: 0,
            jumpForce: 15,
            gravity: 0.8,
            grounded: false,
            color: '#ff00ff'
        };

        // Danh sách vật cản
        let obstacles = [];
        let particles = [];

        // Điều khiển
        function jump() {
            if (runner.grounded && gameActive) {
                runner.dy = -runner.jumpForce;
                runner.grounded = false;
                createParticles(runner.x + 20, runner.y + 60, '#00f2ff');
            }
        }

        window.addEventListener('keydown', e => {
            if (e.code === 'Space') jump();
        });

        canvas.addEventListener('touchstart', e => {
            e.preventDefault();
            jump();
        });

        function createParticles(x, y, color) {
            for (let i = 0; i < 8; i++) {
                particles.push({
                    x, y,
                    vx: (Math.random() - 0.5) * 4,
                    vy: (Math.random() - 0.5) * 4,
                    life: 1,
                    color
                });
            }
        }

        function spawnObstacle() {
            const h = 30 + Math.random() * 50;
            const w = 20 + Math.random() * 30;
            obstacles.push({
                x: canvas.width,
                y: 360 - h,
                width: w,
                height: h,
                color: '#00f2ff'
            });
        }

        function resetGame() {
            score = 0;
            gameSpeed = 5;
            obstacles = [];
            particles = [];
            runner.y = 300;
            runner.dy = 0;
            gameActive = true;
            overlay.classList.add('hidden');
        }

        function gameOver() {
            gameActive = false;
            overlay.classList.remove('hidden');
            document.getElementById('overlayTitle').innerText = "KẾT THÚC";
            document.getElementById('overlayDesc').innerText = "Điểm của bạn: " + Math.floor(score);
            startBtn.innerText = "CHƠI LẠI";
        }

        function update() {
            if (!gameActive) return;

            score += 0.1;
            scoreEl.innerText = Math.floor(score);
            gameSpeed = 5 + (score / 100);
            speedEl.innerText = (gameSpeed / 5).toFixed(1);

            // Vật lý nhân vật
            runner.dy += runner.gravity;
            runner.y += runner.dy;

            if (runner.y + runner.height > 360) {
                runner.y = 360 - runner.height;
                runner.dy = 0;
                runner.grounded = true;
            }

            // Xử lý vật cản
            if (frameCount % Math.max(40, Math.floor(100 - gameSpeed)) === 0) {
                spawnObstacle();
            }

            for (let i = obstacles.length - 1; i >= 0; i--) {
                let o = obstacles[i];
                o.x -= gameSpeed;

                // Kiểm tra va chạm
                if (runner.x < o.x + o.width &&
                    runner.x + runner.width > o.x &&
                    runner.y < o.y + o.height &&
                    runner.y + runner.height > o.y) {
                    gameOver();
                }

                if (o.x + o.width < 0) obstacles.splice(i, 1);
            }

            // Xử lý hạt
            for (let i = particles.length - 1; i >= 0; i--) {
                let p = particles[i];
                p.x -= gameSpeed * 0.5;
                p.y += p.vy;
                p.life -= 0.02;
                if (p.life <= 0) particles.splice(i, 1);
            }

            frameCount++;
        }

        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Vẽ sàn (Neon Grid)
            ctx.strokeStyle = '#333366';
            ctx.beginPath();
            ctx.moveTo(0, 360);
            ctx.lineTo(canvas.width, 360);
            ctx.stroke();

            for (let i = 0; i < canvas.width; i += 40) {
                let offset = (frameCount * gameSpeed) % 40;
                ctx.beginPath();
                ctx.moveTo(i - offset, 360);
                ctx.lineTo(i - offset - 20, 400);
                ctx.stroke();
            }

            // Vẽ nhân vật (Khối Cyber)
            ctx.fillStyle = runner.color;
            ctx.shadowBlur = 15;
            ctx.shadowColor = runner.color;
            ctx.fillRect(runner.x, runner.y, runner.width, runner.height);
            
            // Mắt nhân vật
            ctx.fillStyle = '#fff';
            ctx.fillRect(runner.x + 25, runner.y + 10, 10, 5);

            // Vẽ vật cản (Trụ Neon)
            ctx.shadowBlur = 15;
            obstacles.forEach(o => {
                ctx.fillStyle = o.color;
                ctx.shadowColor = o.color;
                ctx.fillRect(o.x, o.y, o.width, o.height);
                // Hiệu ứng quét sáng
                ctx.fillStyle = 'rgba(255,255,255,0.3)';
                ctx.fillRect(o.x, o.y, 5, o.height);
            });

            // Vẽ hạt
            particles.forEach(p => {
                ctx.globalAlpha = p.life;
                ctx.fillStyle = p.color;
                ctx.fillRect(p.x, p.y, 4, 4);
            });
            ctx.globalAlpha = 1;
            ctx.shadowBlur = 0;

            requestAnimationFrame(() => {
                update();
                draw();
            });
        }

        startBtn.addEventListener('click', resetGame);
        draw();
    </script>
</body>
</html>
