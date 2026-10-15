# Gamefaiz
Game
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gojo Satoru - Infinity Dodge Game</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }

        body {
            background-color: #0b091a;
            color: #fff;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            overflow: hidden;
            touch-action: none; /* Mencegah layar geser saat pakai joystick */
        }

        /* Area Game */
        #game-container {
            width: 100%;
            max-width: 380px;
            height: 480px;
            background: linear-gradient(135deg, #120d29, #1f1338);
            border: 3px solid #8a2be2;
            border-radius: 15px;
            position: relative;
            overflow: hidden;
            box-shadow: 0 0 25px rgba(138, 43, 226, 0.6);
        }

        #score {
            position: absolute;
            top: 15px;
            left: 15px;
            font-size: 18px;
            font-weight: bold;
            color: #00ffff;
            text-shadow: 0 0 8px #00ffff;
            z-index: 10;
        }

        /* Karakter Gojo */
        #gojo {
            width: 45px;
            height: 45px;
            background: #00ffff;
            border-radius: 50%;
            position: absolute;
            bottom: 20px;
            left: 165px;
            box-shadow: 0 0 15px #00ffff, 0 0 30px #8a2be2;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 22px;
            z-index: 5;
        }

        /* Blok Kutukan Musuh */
        .obstacle {
            width: 35px;
            height: 35px;
            background: #ff3366;
            position: absolute;
            top: -40px;
            border-radius: 8px;
            box-shadow: 0 0 12px #ff3366;
        }

        /* Game Over Screen */
        #game-over {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            text-align: center;
            display: none;
            background: rgba(10, 8, 25, 0.95);
            padding: 25px;
            border-radius: 12px;
            border: 2px solid #ff3366;
            z-index: 20;
            width: 80%;
        }

        #game-over h2 {
            color: #ff3366;
            margin-bottom: 10px;
            font-size: 22px;
        }

        #restart-btn {
            padding: 10px 20px;
            background: #8a2be2;
            border: none;
            color: white;
            font-weight: bold;
            border-radius: 6px;
            cursor: pointer;
            margin-top: 15px;
            box-shadow: 0 0 10px #8a2be2;
        }

        #restart-btn:hover {
            background: #9b51e0;
        }

        /* Bagian Kontrol Joystick */
        #controls {
            margin-top: 25px;
            display: flex;
            align-items: center;
            justify-content: center;
            width: 100%;
        }

        #joystick-base {
            width: 110px;
            height: 110px;
            background: rgba(138, 43, 226, 0.15);
            border: 2px solid rgba(138, 43, 226, 0.5);
            border-radius: 50%;
            position: relative;
            display: flex;
            align-items: center;
            justify-content: center;
            touch-action: none;
        }

        #joystick-stick {
            width: 50px;
            height: 50px;
            background: radial-gradient(circle, #00ffff 0%, #8a2be2 100%);
            border-radius: 50%;
            position: absolute;
            box-shadow: 0 0 12px #00ffff;
            cursor: pointer;
        }
    </style>
</head>
<body>

    <!-- Area Game -->
    <div id="game-container">
        <div id="score">Skor: 0</div>
        <div id="gojo">👁️</div>
        <div id="game-over">
            <h2>Mugen Ditembus!</h2>
            <p id="final-score">Skor Kamu: 0</p>
            <button id="restart-btn" onclick="restartGame()">Coba Lagi</button>
        </div>
    </div>

    <!-- Joystick Interaktif -->
    <div id="controls">
        <div id="joystick-base">
            <div id="joystick-stick"></div>
        </div>
    </div>

    <script>
        const gameContainer = document.getElementById('game-container');
        const gojo = document.getElementById('gojo');
        const scoreDisplay = document.getElementById('score');
        const gameOverScreen = document.getElementById('game-over');
        const finalScoreDisplay = document.getElementById('final-score');
        
        const joystickBase = document.getElementById('joystick-base');
        const joystickStick = document.getElementById('joystick-stick');

        let gojoX = 167;
        let gojoY = 415;
        let score = 0;
        let isGameOver = false;
        let obstacles = [];
        let gameInterval;
        let obstacleTimer;

        // Joystick Logic Variables
        let joystickCenter = { x: 0, y: 0 };
        let touchId = null;
        let joystickVector = { x: 0, y: 0 };

        function updateJoystickCenter() {
            const rect = joystickBase.getBoundingClientRect();
            joystickCenter.x = rect.left + rect.width / 2;
            joystickCenter.y = rect.top + rect.height / 2;
        }

        window.addEventListener('resize', updateJoystickCenter);
        updateJoystickCenter();

        // Event listener untuk Joystick (Mendukung Mouse & Touchscreen HP)
        joystickBase.addEventListener('pointerdown', (e) => {
            touchId = e.pointerId;
            joystickBase.setPointerCapture(touchId);
            handleJoystickMove(e);
        });

        joystickBase.addEventListener('pointermove', (e) => {
            if (e.pointerId === touchId) {
                handleJoystickMove(e);
            }
        });

        joystickBase.addEventListener('pointerup', (e) => {
            if (e.pointerId === touchId) {
                resetJoystick();
            }
        });

        function handleJoystickMove(e) {
            const maxDist = 30; // Batas radius gerakan stik
            const dx = e.clientX - joystickCenter.x;
            const dy = e.clientY - joystickCenter.y;
            const distance = Math.sqrt(dx * dx + dy * dy);

            let angle = Math.atan2(dy, dx);
            let dist = Math.min(distance, maxDist);

            let stickX = Math.cos(angle) * dist;
            let stickY = Math.sin(angle) * dist;

            joystickStick.style.transform = `translate(${stickX}px, ${stickY}px)`;

            // Vektor arah gerakan (-1 sampai 1)
            joystickVector.x = stickX / maxDist;
            joystickVector.y = stickY / maxDist;
        }

        function resetJoystick() {
            touchId = null;
            joystickStick.style.transform = `translate(0px, 0px)`;
            joystickVector.x = 0;
            joystickVector.y = 0;
        }

        function startGame() {
            gojoX = 167;
            gojoY = 415;
            score = 0;
            isGameOver = false;
            obstacles.forEach(obs => obs.element.remove());
            obstacles = [];
            scoreDisplay.innerText = "Skor: 0";
            gameOverScreen.style.display = 'none';

            gameInterval = setInterval(updateGame, 20);
            obstacleTimer = setInterval(spawnObstacle, 900);
        }

        function spawnObstacle() {
            if (isGameOver) return;
            const obsElement = document.createElement('div');
            obsElement.classList.add('obstacle');
            const randomX = Math.random() * (gameContainer.clientWidth - 35);
            obsElement.style.left = randomX + 'px';
            gameContainer.appendChild(obsElement);

            obstacles.push({
                element: obsElement,
                x: randomX,
                y: -40,
                speed: 3.5 + Math.random() * 2.5
            });
        }

        function updateGame() {
            if (isGameOver) return;

            // Kecepatan gerak Gojo berdasarkan joystick
            const speed = 5.5;
            gojoX += joystickVector.x * speed;
            gojoY += joystickVector.y * speed;

            // Batasan area game (agar Gojo tidak keluar kotak)
            const maxX = gameContainer.clientWidth - gojo.clientWidth;
            const maxY = gameContainer.clientHeight - gojo.clientHeight;

            if (gojoX < 0) gojoX = 0;
            if (gojoX > maxX) gojoX = maxX;
            if (gojoY < 0) gojoY = 0;
            if (gojoY > maxY) gojoY = maxY;

            gojo.style.left = gojoX + 'px';
            gojo.style.top = gojoY + 'px';

            // Update posisi rintangan (kutukan)
            for (let i = obstacles.length - 1; i >= 0; i--) {
                let obs = obstacles[i];
                obs.y += obs.speed;
                obs.element.style.top = obs.y + 'px';

                // Sistem Deteksi Tabrakan (Colision Detection)
                if (
                    gojoX < obs.x + 35 &&
                    gojoX + 45 > obs.x &&
                    gojoY < obs.y + 35 &&
                    gojoY + 45 > obs.y
                ) {
                    endGame();
                }

                // Jika rintangan melewati batas bawah, hapus dan tambah skor
                if (obs.y > gameContainer.clientHeight) {
                    obs.element.remove();
                    obstacles.splice(i, 1);
                    score += 10;
                    scoreDisplay.innerText = "Skor: " + score;
                }
            }
        }

        function endGame() {
            isGameOver = true;
            clearInterval(gameInterval);
            clearInterval(obstacleTimer);
            finalScoreDisplay.innerText = "Skor Kamu: " + score;
            gameOverScreen.style.display = 'block';
        }

        function restartGame() {
            startGame();
        }

        // Jalankan game pertama kali
        startGame();
    </script>
</body>
</html>
