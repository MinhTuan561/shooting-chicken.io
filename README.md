# shooting-chicken.io<!DOCTYPE html>
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Trò Chơi Bắn Gà</title>
    <style>
        body { margin: 0; display: flex; justify-content: center; align-items: center; height: 100vh; background-color: #87CEEB; }
        canvas { border: 2px solid #000; }
        #restartButton { display: none; margin-top: 20px; }
    </style>
</head>
<body>
    <canvas id="gameCanvas" width="800" height="600"></canvas>
    <button id="restartButton">Chơi lại</button>
    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        let blocks = [];
        let bullets = [];
        let enemyBullets = [];
        let score = 0;
        let gunX = 400; // Vị trí ban đầu của súng
        let startTime = Date.now();
        const timeLimit = 60;
        let gameOver = false;
        let round = 1;
        let blockDirection = 1;

        let lastBulletTime = 0;
        const bulletInterval = 1000;

        function resetGame() {
            blocks = [];
            bullets = [];
            enemyBullets = [];
            score = 0;
            gunX = 400;
            startTime = Date.now();
            gameOver = false;
            round = 1;
            document.getElementById('restartButton').style.display = 'none';
            createBlocks();
            gameLoop();
        }

        function createBlocks() {
            blocks = [];
            const blockCount = round === 1 ? 10 : 5;
            const rows = 1;
            const blockWidth = 50;
            const blockHeight = 50;
            const gapX = (canvas.width - (blockWidth * blockCount)) / (blockCount + 1);

            for (let i = 0; i < blockCount; i++) {
                const x = gapX + i * (blockWidth + gapX);
                const y = 100;
                blocks.push({ x: x, y: y, width: blockWidth, height: blockHeight });
            }
        }

        function drawGun() {
            ctx.fillStyle = 'black';
            ctx.fillRect(gunX, 550, 50, 10); // Vẽ súng
        }

        function drawBlocks() {
            blocks.forEach(block => {
                drawChicken(block.x, block.y); // Vẽ hình con gà
            });
        }

        function drawChicken(x, y) {
            ctx.fillStyle = 'yellow';
            ctx.beginPath();
            ctx.moveTo(x + 25, y); // Đỉnh đầu
            ctx.lineTo(x, y + 40); // Cánh trái
            ctx.lineTo(x + 50, y + 40); // Cánh phải
            ctx.closePath();
            ctx.fill(); // Vẽ thân
            ctx.fillStyle = 'orange';
            ctx.beginPath();
            ctx.arc(x + 25, y + 20, 15, 0, Math.PI * 2); // Đầu
            ctx.fill();
            ctx.fillStyle = 'black';
            ctx.beginPath();
            ctx.arc(x + 20, y + 15, 3, 0, Math.PI * 2); // Mắt trái
            ctx.fill();
            ctx.beginPath();
            ctx.arc(x + 30, y + 15, 3, 0, Math.PI * 2); // Mắt phải
            ctx.fill();
        }

        function drawBullets() {
            ctx.fillStyle = 'red';
            bullets.forEach(bullet => {
                ctx.fillRect(bullet.x, bullet.y, bullet.width, bullet.height);
            });
        }

        function drawEnemyBullets() {
            ctx.fillStyle = 'blue';
            enemyBullets.forEach(bullet => {
                ctx.fillRect(bullet.x, bullet.y, bullet.width, bullet.height);
            });
        }

        // Các hàm update và check collision tương tự như trước

        // Xử lý sự kiện touchstart
        canvas.addEventListener('touchstart', (event) => {
            event.preventDefault();
            handleTouchStart(event.touches[0].clientX, event.touches[0].clientY);
        });

        // Xử lý sự kiện touchmove
        canvas.addEventListener('touchmove', (event) => {
            event.preventDefault();
            handleTouchMove(event.touches[0].clientX, event.touches[0].clientY);
        });

        // Xử lý sự kiện touchend
        canvas.addEventListener('touchend', (event) => {
            event.preventDefault();
            handleTouchEnd();
        });

        function handleTouchStart(x, y) {
            if (x > gunX && x < gunX + 50 && y > 550) {
                shoot();
            }
        }

        function handleTouchMove(x, y) {
            gunX = Math.max(0, Math.min(canvas.width - 50, x - 25));
        }

        function handleTouchEnd() {
            // Không cần xử lý gì thêm
        }

        function gameLoop() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            drawGun();
            drawBlocks();
            drawBullets();
            drawEnemyBullets();
            // Các đoạn code khác như trước
            requestAnimationFrame(gameLoop);
        }

        resetGame();
    </script>
</body>
</html>
