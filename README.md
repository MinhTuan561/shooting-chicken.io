<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Trò Chơi Bắn Gà</title>
    <style>
        body { margin: 0; display: flex; justify-content: center; align-items: center; height: 100vh; background-color: #87CEEB; }
        canvas { border: 2px solid #000; }
        #controls {
            position: absolute;
            bottom: 20px;
            display: flex;
            justify-content: center;
            width: 100%;
        }
        .button {
            background-color: #fff;
            border: 2px solid #000;
            padding: 10px 20px;
            margin: 0 10px;
            cursor: pointer;
            font-size: 20px;
        }
        #restartButton {
            display: none; /* Ẩn nút chơi lại ban đầu */
            position: absolute;
            bottom: 20px; /* Đặt ở góc dưới */
            right: 20px; /* Đặt ở góc phải */
            background-color: #fff;
            border: 2px solid #000;
            padding: 15px 30px;
            font-size: 20px;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <canvas id="gameCanvas" width="800" height="600"></canvas>
    <div id="controls">
        <div class="button" id="leftButton">Trái</div>
        <div class="button" id="shootButton">Bắn</div>
        <div class="button" id="rightButton">Phải</div>
    </div>
    <button id="restartButton">Chơi lại</button>
    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        let blocks = [];
        let bullets = [];
        let enemyBullets = []; // Đạn từ gà
        let score = 0;
        let gunX = 400; // Vị trí ban đầu của súng
        let startTime = Date.now(); // Thời gian bắt đầu trò chơi
        const timeLimit = 60; // Giới hạn thời gian là 60 giây (1 phút)
        let gameOver = false; // Biến kiểm tra trạng thái trò chơi
        let round = 1; // Biến theo dõi vòng chơi
        let blockDirection = 1; // Hướng di chuyển của block (1: phải, -1: trái)

        let lastBulletTime = 0; // Thời gian bắn viên đạn cuối cùng
        const bulletInterval = 1000; // Thời gian giữa các viên đạn (1000ms cho 1 viên/giây)

        function resetGame() {
            blocks = [];
            bullets = [];
            enemyBullets = []; // Đặt lại đạn từ gà
            score = 0;
            gunX = 400;
            startTime = Date.now();
            gameOver = false;
            round = 1; // Đặt lại vòng chơi về 1
            document.getElementById('restartButton').style.display = 'none'; // Ẩn nút chơi lại
            createBlocks(); // Tạo block cho vòng chơi đầu tiên
            gameLoop(); // Bắt đầu vòng lặp trò chơi
        }

        function createBlocks() {
            blocks = []; // Xóa các block cũ
            const blockCount = round === 1 ? 10 : 5; // Số block cho mỗi vòng
            const rows = 1; // Số hàng
            const blockWidth = 50; // Chiều rộng hình con gà
            const blockHeight = 50; // Chiều cao hình con gà
            const gapX = (canvas.width - (blockWidth * blockCount)) / (blockCount + 1); // Khoảng cách ngang

            for (let i = 0; i < blockCount; i++) {
                const x = gapX + i * (blockWidth + gapX);
                const y = 100; // Đặt vị trí y cho hàng
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
            ctx.fillStyle = 'blue'; // Màu đạn của gà
            enemyBullets.forEach(bullet => {
                ctx.fillRect(bullet.x, bullet.y, bullet.width, bullet.height);
            });
        }

        function updateBlocks() {
            blocks.forEach(block => {
                block.x += blockDirection * 2; // Di chuyển block
            });

            // Kiểm tra va chạm với biên
            const firstBlock = blocks[0];
            const lastBlock = blocks[blocks.length - 1];

            if (firstBlock.x <= 0 || lastBlock.x + lastBlock.width >= canvas.width) {
                blockDirection *= -1; // Đảo ngược hướng di chuyển
            }
        }

        function updateBullets() {
            bullets.forEach(bullet => {
                bullet.y -= 5; // Tốc độ bắn của đạn
            });
            bullets = bullets.filter(bullet => bullet.y > 0);
        }

        function updateEnemyBullets() {
            enemyBullets.forEach(bullet => {
                bullet.y += 3; // Tốc độ bắn của đạn gà
            });
            enemyBullets = enemyBullets.filter(bullet => bullet.y < canvas.height);
        }

        function checkCollision() {
            bullets.forEach((bullet, bulletIndex) => {
                blocks.forEach((block, blockIndex) => {
                    if (bullet.x < block.x + block.width &&
                        bullet.x + bullet.width > block.x &&
                        bullet.y < block.y + block.height &&
                        bullet.y + bullet.height > block.y) {
                        // Xóa block và đạn khi có va chạm
                        blocks.splice(blockIndex, 1);
                        bullets.splice(bulletIndex, 1);
                        score++;
                    }
                });
            });

            // Kiểm tra va chạm giữa đạn của gà và súng
            enemyBullets.forEach((enemyBullet, bulletIndex) => {
                if (enemyBullet.x < gunX + 50 && enemyBullet.x + enemyBullet.width > gunX &&
                    enemyBullet.y + enemyBullet.height > 550) {
                    gameOver = true; // Nếu bị trúng đạn, trò chơi kết thúc
                    document.getElementById('restartButton').style.display = 'block'; // Hiện nút chơi lại
                }
            });
        }

        function spawnEnemyBullets() {
            const currentTime = Date.now();
            if (round === 2) {
                blocks.forEach(block => {
                    if (currentTime - lastBulletTime > bulletInterval) { // Bắn mỗi giây
                        enemyBullets.push({ x: block.x + 22, y: block.y + 40, width: 5, height: 10 }); // Vị trí bắn
                    }
                });
                lastBulletTime = currentTime; // Cập nhật thời gian bắn viên đạn cuối
            }
        }

        function gameLoop() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            drawGun();
            drawBlocks();
            drawBullets();
            drawEnemyBullets();
            updateBlocks();
            updateBullets();
            updateEnemyBullets();
            checkCollision();
            spawnEnemyBullets(); // Tạo đạn từ gà

            // Tính toán thời gian đã trôi qua
            const elapsedTime = Math.floor((Date.now() - startTime) / 1000); // Thời gian tính bằng giây

            ctx.fillStyle = 'black';
            ctx.font = '20px Arial';
            ctx.fillText('Điểm: ' + score, 10, 20);
            ctx.fillText('Thời gian: ' + (timeLimit - elapsedTime) + ' giây', 10, 50); // Hiển thị thời gian

            // Kiểm tra điều kiện thắng
            if (round === 2 && blocks.length === 0) {
                ctx.fillStyle = 'green';
                ctx.font = '40px Arial';
                ctx.fillText('Bạn đã thắng!', canvas.width / 2 - 100, canvas.height / 2);
                gameOver = true; // Đánh dấu trò chơi đã kết thúc
                document.getElementById('restartButton').style.display = 'block'; // Hiện nút chơi lại
                return; // Dừng vòng lặp trò chơi
            }

            // Kiểm tra điều kiện thua
            if (elapsedTime >= timeLimit) {
                ctx.fillStyle = 'red';
                ctx.font = '40px Arial';
                ctx.fillText('Thời gian đã hết!', canvas.width / 2 - 150, canvas.height / 2);
                gameOver = true; // Đánh dấu trò chơi đã kết thúc
                document.getElementById('restartButton').style.display = 'block'; // Hiện nút chơi lại
                return; // Dừng vòng lặp trò chơi
            }

            // Kiểm tra điều kiện chuyển vòng
            if (blocks.length === 0) {
                round++; // Tăng vòng chơi
                createBlocks(); // Tạo block cho vòng chơi tiếp theo
                score += 5; // Thưởng điểm khi qua vòng
            }

            requestAnimationFrame(gameLoop);
        }

        // Điều khiển bằng bàn phím
        document.addEventListener('keydown', (event) => {
            if (!gameOver) {
                switch (event.key) {
                    case 'ArrowLeft':
                        gunX -= 15; // Di chuyển sang trái
                        if (gunX < 0) gunX = 0; // Giới hạn không cho súng ra ngoài
                        break;
                    case 'ArrowRight':
                        gunX += 15; // Di chuyển sang phải
                        if (gunX > canvas.width - 50) gunX = canvas.width - 50; // Giới hạn không cho súng ra ngoài
                        break;
                    case ' ':
                        bullets.push({ x: gunX + 22, y: 540, width: 5, height: 10 }); // Vị trí bắn
                        break;
                }
            }
        });

        document.getElementById('restartButton').addEventListener('click', resetGame); // Thêm sự kiện cho nút chơi lại

        resetGame(); // Khởi động trò chơi
    </script>
</body>
</html>
