<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <title>Game Chim Bay - Flappy Bird</title>
    <style>
        canvas {
            border: 1px solid black;
        }
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background-color: #87CEEB;
        }
    </style>
</head>
<body>
    <canvas id="game" width="400" height="600"></canvas>

    <script>
        const canvas = document.getElementById('game');
        const ctx = canvas.getContext('2d');

        // Thông số chim
        const bird = {
            x: 50,
            y: 300,
            width: 20,
            height: 20,
            gravity: 0.6,
            lift: -10,
            velocity: 0
        };

        // Thông số ống
        const pipes = [];
        const pipeWidth = 50;
        const pipeGap = 150;
        let pipeFrequency = 90; // Tần suất xuất hiện ống (frame)
        let frameCount = 0;

        let score = 0;
        let gameOver = false;

        // Vẽ chim
        function drawBird() {
            ctx.fillStyle = '#FFD700'; // Màu vàng
            ctx.fillRect(bird.x, bird.y, bird.width, bird.height);
        }

        // Cập nhật vị trí chim
        function updateBird() {
            bird.velocity += bird.gravity;
            bird.y += bird.velocity;

            // Giới hạn chim không bay ra ngoài
            if (bird.y + bird.height > canvas.height) {
                bird.y = canvas.height - bird.height;
                bird.velocity = 0;
                gameOver = true;
            }
            if (bird.y < 0) {
                bird.y = 0;
                bird.velocity = 0;
            }
        }

        // Tạo ống mới
        function createPipe() {
            const pipeHeight = Math.floor(Math.random() * (canvas.height - pipeGap - 100)) + 50;
            pipes.push({
                x: canvas.width,
                top: pipeHeight,
                bottom: canvas.height - pipeHeight - pipeGap,
                counted: false
            });
        }

        // Vẽ ống
        function drawPipes() {
            ctx.fillStyle = '#228B22'; // Màu xanh lá
            pipes.forEach(pipe => {
                // Ống trên
                ctx.fillRect(pipe.x, 0, pipeWidth, pipe.top);
                // Ống dưới
                ctx.fillRect(pipe.x, canvas.height - pipe.bottom, pipeWidth, pipe.bottom);
            });
        }

        // Cập nhật ống
        function updatePipes() {
            pipes.forEach(pipe => {
                pipe.x -= 2; // Tốc độ di chuyển ống

                // Kiểm tra va chạm
                if (
                    bird.x + bird.width > pipe.x &&
                    bird.x < pipe.x + pipeWidth &&
                    (bird.y < pipe.top || bird.y + bird.height > canvas.height - pipe.bottom)
                ) {
                    gameOver = true;
                }

                // Tính điểm
                if (!pipe.counted && bird.x > pipe.x + pipeWidth) {
                    score++;
                    pipe.counted = true;
                }
            });

            // Xóa ống ra khỏi màn hình
            if (pipes.length && pipes[0].x + pipeWidth < 0) {
                pipes.shift();
            }

            // Tạo ống mới
            frameCount++;
            if (frameCount >= pipeFrequency) {
                createPipe();
                frameCount = 0;
            }
        }

        // Vẽ điểm số
        function drawScore() {
            ctx.fillStyle = '#000';
            ctx.font = '20px Arial';
            ctx.fillText('Score: ' + score, 10, 30);
        }

        // Điều khiển chim
        document.addEventListener('keydown', (e) => {
            if (e.code === 'Space') {
                bird.velocity = bird.lift;
            }
        });

        // Reset game
        function resetGame() {
            bird.y = 300;
            bird.velocity = 0;
            pipes.length = 0;
            score = 0;
            gameOver = false;
            frameCount = 0;
            createPipe(); // Tạo ống đầu tiên
            loop();
        }

        // Vòng lặp game
        function loop() {
            if (gameOver) {
                ctx.fillStyle = '#FF0000';
                ctx.font = '40px Arial';
                ctx.fillText('Game Over!', 80, 300);
                ctx.fillText('Score: ' + score, 120, 350);
                ctx.font = '20px Arial';
                ctx.fillText('Nhấn Space để chơi lại', 100, 400);
                document.addEventListener('keydown', (e) => {
                    if (e.code === 'Space' && gameOver) {
                        resetGame();
                    }
                }, { once: true });
                return;
            }

            // Xóa màn hình
            ctx.fillStyle = '#87CEEB';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Cập nhật và vẽ
            updateBird();
            drawBird();
            updatePipes();
            drawPipes();
            drawScore();

            requestAnimationFrame(loop);
        }

        // Bắt đầu game
        createPipe();
        loop();
    </script>
</body>
</html>
