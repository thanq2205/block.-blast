<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Block Blast Mini</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            background: #1a1a1a;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            overflow: hidden;
            color: white;
            user-select: none;
        }

        #game-container {
            position: relative;
            width: 320px;
            height: 560px;
            background: #222;
            border: 4px solid #444;
            border-radius: 12px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.5);
            overflow: hidden;
        }

        /* Giao diện Màn hình Bắt đầu */
        #start-screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.85);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 10;
        }

        #start-screen h1 {
            font-size: 28px;
            margin-bottom: 20px;
            color: #ffcc00;
            text-shadow: 0 2px 4px rgba(0,0,0,0.5);
        }

        .btn-green {
            background-color: #2ecc71;
            color: white;
            border: none;
            padding: 15px 40px;
            font-size: 20px;
            font-weight: bold;
            border-radius: 30px;
            cursor: pointer;
            box-shadow: 0 4px 15px rgba(46, 204, 113, 0.4);
            transition: all 0.2s ease;
        }

        .btn-green:hover {
            background-color: #27ae60;
            transform: scale(1.05);
        }

        /* Bảng hiển thị Điểm */
        #score-board {
            position: absolute;
            top: 10px;
            left: 15px;
            font-size: 18px;
            font-weight: bold;
            z-index: 5;
        }

        /* Khu vực chơi game */
        #play-area {
            position: absolute;
            top: 40px;
            left: 0;
            width: 320px;
            height: 520px;
        }

        /* Khối màu */
        .block {
            position: absolute;
            width: 40px;
            height: 40px;
            border-radius: 4px;
            box-sizing: border-box;
            border: 1px solid rgba(0,0,0,0.2);
            box-shadow: inset 0 0 8px rgba(0,0,0,0.3);
        }

        /* Màn hình Thua (Game Over) */
        #game-over-screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.9);
            display: none;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 10;
        }

        #game-over-screen h2 {
            font-size: 36px;
            color: #e74c3c;
            margin-bottom: 10px;
        }

        #game-over-screen p {
            font-size: 18px;
            margin-bottom: 20px;
        }

        #controls-hint {
            position: absolute;
            bottom: 5px;
            width: 100%;
            text-align: center;
            font-size: 11px;
            color: #777;
            z-index: 5;
        }
    </style>
</head>
<body>

    <div id="game-container">
        <!-- Điểm số -->
        <div id="score-board">Điểm: <span id="score">0</span></div>

        <!-- Màn hình bắt đầu -->
        <div id="start-screen">
            <h1>BLOCK BLAST</h1>
            <button class="btn-green" id="start-btn">Bắt Đầu</button>
        </div>

        <!-- Vùng chơi chính -->
        <div id="play-area"></div>

        <!-- Hướng dẫn -->
        <div id="controls-hint">← / →: Di chuyển | ↓: Rơi nhanh</div>

        <!-- Màn hình Game Over -->
        <div id="game-over-screen">
            <h2>GAME OVER</h2>
            <p>Điểm đạt được: <span id="final-score">0</span></p>
            <button class="btn-green" id="restart-btn">Chơi Lại</button>
        </div>
    </div>

    <script>
        const startScreen = document.getElementById('start-screen');
        const startBtn = document.getElementById('start-btn');
        const playArea = document.getElementById('play-area');
        const scoreDisplay = document.getElementById('score');
        const gameOverScreen = document.getElementById('game-over-screen');
        const restartBtn = document.getElementById('restart-btn');
        const finalScoreDisplay = document.getElementById('final-score');

        const COLS = 8; // 8 cột (320px / 40px)
        const ROWS = 13; // 13 hàng (520px / 40px)
        let grid = []; // Mảng lưu trữ các khối đã cố định trên bàn chơi

        let score = 0;
        let gameInterval = null;
        let currentBlock = null;
        let isPlaying = false;
        let currentThemeIndex = 0;

        // Các bộ màu sắc sẽ thay đổi khi hoàn thành 1 hàng
        const themes = [
            ['#e74c3c', '#3498db', '#f1c40f', '#9b59b6', '#e67e22'],
            ['#1abc9c', '#e84393', '#00cec9', '#fdcb6e', '#ff7675'],
            ['#00b894', '#0984e3', '#6c5ce7', '#ffeaa7', '#d63031'],
            ['#e17055', '#fd79a8', '#a29bfe', '#55efc4', '#ffeaa7']
        ];

        startBtn.addEventListener('click', startGame);
        restartBtn.addEventListener('click', startGame);

        function startGame() {
            startScreen.style.display = 'none';
            gameOverScreen.style.display = 'none';
            playArea.innerHTML = '';
            score = 0;
            scoreDisplay.textContent = score;
            currentThemeIndex = 0;
            isPlaying = true;

            // Khởi tạo lưới trống (grid)
            grid = Array.from({ length: ROWS }, () => Array(COLS).fill(null));

            spawnBlock();

            if (gameInterval) clearInterval(gameInterval);
            gameInterval = setInterval(updateGame, 25); // Tốc độ nhanh hơn một chút
        }

        // Tạo khối mới rơi từ trên xuống
        function spawnBlock() {
            if (!isPlaying) return;

            const activeColors = themes[currentThemeIndex];
            const color = activeColors[Math.floor(Math.random() * activeColors.length)];

            let col = Math.floor(Math.random() * COLS);
            let row = 0;

            // Kiểm tra nếu vị trí xuất phát đã bị chặn -> Game Over
            if (grid[row][col] !== null) {
                triggerGameOver();
                return;
            }

            const element = document.createElement('div');
            element.classList.add('block');
            element.style.backgroundColor = color;
            playArea.appendChild(element);

            currentBlock = {
                col: col,
                x: col * 40,
                y: 0,
                color: color,
                element: element
            };
            updateBlockPosition();
        }

        function updateBlockPosition() {
            currentBlock.element.style.left = currentBlock.x + 'px';
            currentBlock.element.style.top = currentBlock.y + 'px';
        }

        // Vòng lặp cập nhật chuyển động
        function updateGame() {
            if (!isPlaying || !currentBlock) return;

            // Tốc độ rơi nhanh hơn (2 pixel mỗi khung hình)
            currentBlock.y += 2;
            
            let targetRow = Math.floor((currentBlock.y + 20) / 40);

            // Kiểm tra xem có chạm đáy hoặc chạm khối phía dưới chưa
            if (targetRow >= ROWS || (targetRow >= 0 && grid[targetRow][currentBlock.col] !== null)) {
                // Cố định vị trí nằm sát ngay phía trên vật cản hoặc đáy
                let finalRow = targetRow - 1;
                if (finalRow < 0) finalRow = 0;

                // Nếu vị trí cố định đã đầy hoặc chạm đỉnh
                if (grid[finalRow][currentBlock.col] !== null) {
                    triggerGameOver();
                    return;
                }

                // Cố định khối vào mảng grid
                currentBlock.y = finalRow * 40;
                updateBlockPosition();

                grid[finalRow][currentBlock.col] = {
                    element: currentBlock.element,
                    color: currentBlock.color
                };

                // Kiểm tra xem hàng này đã được lấp đầy chưa (đủ 8 ô trong hàng)
                checkAndClearRows();

                // Tạo khối tiếp theo
                spawnBlock();
            } else {
                updateBlockPosition();
            }
        }

        // Kiểm tra và xóa hàng khi được lấp đầy
        function checkAndClearRows() {
            let cleared = false;

            for (let r = 0; r < ROWS; r++) {
                let isRowFull = true;
                for (let c = 0; c < COLS; c++) {
                    if (grid[r][c] === null) {
                        isRowFull = false;
                        break;
                    }
                }

                // Nếu hàng đã lấp đầy các khối
                if (isRowFull) {
                    cleared = true;
                    score += 10;
                    scoreDisplay.textContent = score;

                    // Xóa các phần tử DOM của hàng đó
                    for (let c = 0; c < COLS; c++) {
                        if (grid[r][c] && grid[r][c].element) {
                            grid[r][c].element.remove();
                        }
                        grid[r][c] = null;
                    }

                    // Kéo các hàng phía trên rơi xuống 1 ô
                    for (let moveRow = r; moveRow > 0; moveRow--) {
                        for (let c = 0; c < COLS; c++) {
                            grid[moveRow][c] = grid[moveRow - 1][c];
                            if (grid[moveRow][c] !== null) {
                                grid[moveRow][c].element.style.top = (moveRow * 40) + 'px';
                            }
                        }
                    }
                    // Làm sạch hàng trên cùng
                    for (let c = 0; c < COLS; c++) {
                        grid[0][c] = null;
                    }
                }
            }

            // Khi hoàn thành/lấp đầy một hàng, đổi sang bộ màu sắc mới
            if (cleared) {
                currentThemeIndex = (currentThemeIndex + 1) % themes.length;
            }
        }

        // Điều khiển phím
        window.addEventListener('keydown', (e) => {
            if (!isPlaying || !currentBlock) return;

            if (e.key === 'ArrowLeft') {
                let newCol = currentBlock.col - 1;
                let currentRow = Math.floor(currentBlock.y / 40);
                if (newCol >= 0 && (currentRow < 0 || grid[currentRow][newCol] === null)) {
                    currentBlock.col = newCol;
                    currentBlock.x = newCol * 40;
                    updateBlockPosition();
                }
            } else if (e.key === 'ArrowRight') {
                let newCol = currentBlock.col + 1;
                let currentRow = Math.floor(currentBlock.y / 40);
                if (newCol < COLS && (currentRow < 0 || grid[currentRow][newCol] === null)) {
                    currentBlock.col = newCol;
                    currentBlock.x = newCol * 40;
                    updateBlockPosition();
                }
            } else if (e.key === 'ArrowDown') {
                // Phím mũi tên xuống giúp rơi nhanh hơn nữa
                currentBlock.y += 15;
            }
        });

        function triggerGameOver() {
            isPlaying = false;
            clearInterval(gameInterval);
            if (currentBlock && currentBlock.element) {
                currentBlock.element.remove();
            }
            finalScoreDisplay.textContent = score;
            gameOverScreen.style.display = 'flex';
        }
    </script>
</body>
</html>
