# Tugas2_105841110222_IlfauzaFebriantyFaisal_5C

Nama : Ilfauza Febrianty Faisal
NIM : 105841110222
Kelas : 5C

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Bubble Shooter Game</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            font-family: 'Roboto', Arial, sans-serif;
            background: linear-gradient(120deg, #1a73e8, #a4508b, #ff8c42);
            background-size: 200% 200%;
            animation: gradientShift 10s ease infinite;
            color: white;
        }
        
        @keyframes gradientShift {
            0% {
                background-position: 0% 50%;
            }
            50% {
                background-position: 100% 50%;
            }
            100% {
                background-position: 0% 50%;
            }
        }
        
        canvas {
            display: block;
        }
        
        #overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.8);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            animation: fadeIn 1s ease forwards;
        }
        
        @keyframes fadeIn {
            from {
                opacity: 0;
            }
            to {
                opacity: 1;
            }
        }
        
        #overlay h1 {
            font-size: 3em;
            margin-bottom: 10px;
        }
        
        #overlay h5 {
            font-size: 1.2em;
            margin-top: 5px;
            color: #ddd;
        }
        
        button {
            font-size: 1.2em;
            padding: 15px 40px;
            margin: 10px 0;
            border: none;
            border-radius: 8px;
            background: linear-gradient(90deg, #ff5722, #ff9800);
            color: white;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
            cursor: pointer;
            transition: transform 0.2s, box-shadow 0.2s;
        }
        
        button:hover {
            transform: scale(1.05);
            box-shadow: 0 6px 15px rgba(0, 0, 0, 0.5);
        }
        
        #restartButton {
            background: linear-gradient(90deg, #03a9f4, #2196f3);
        }
        
        @media (max-width: 768px) {
            #overlay h1 {
                font-size: 2em;
            }
            button {
                font-size: 1em;
                padding: 10px 20px;
            }
        }
    </style>
</head>

<body>
    <div id="overlay">
        <h1>Bubble Shooter Game</h1>
        <h5>Ready to test your skills?</h5>
        <button id="startButton">Start Game</button>
    </div>
    <canvas id="gameCanvas"></canvas>

    <script>
        // Mengambil elemen dari HTML
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const overlay = document.getElementById('overlay');
        const startButton = document.getElementById('startButton');

        // Menetapkan ukuran canvas sesuai ukuran jendela
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;

        // Deklarasi variabel game
        let player, bubbles, score, isGameRunning, speedMultiplier, powerUps, timer, timeLimit;

        // Kelas dasar untuk objek game (bubble, player, dll.)
        class GameObject {
            constructor(x, y, size, color) {
                this.x = x;
                this.y = y;
                this.size = size;
                this.color = color;
            }

            draw() {
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.fill();
                ctx.closePath();
            }
        }

        // Kelas Player yang mewarisi GameObject
        class Player extends GameObject {
            constructor(x, y, size, color) {
                super(x, y, size, color);
                this.bullets = []; // Menyimpan peluru yang ditembakkan
            }

            shoot() {
                this.bullets.push(new GameObject(this.x, this.y - this.size - 10, 5, 'blue')); // Menembakkan peluru
            }

            updateBullets() {
                // Memperbarui posisi peluru
                this.bullets.forEach((bullet, index) => {
                    bullet.y -= 10; // Kecepatan peluru
                    if (bullet.y < 0) {
                        this.bullets.splice(index, 1); // Menghapus peluru jika melewati batas atas
                    }
                });
            }

            draw() {
                super.draw(); // Menggambar player
                this.bullets.forEach(bullet => bullet.draw()); // Menggambar setiap peluru
            }
        }

        // Kelas Bubble yang mewarisi GameObject
        class Bubble extends GameObject {
            constructor(x, y, size, color, isBoom = false, speed = 1) {
                super(x, y, size, color);
                this.isBoom = isBoom; // Menandakan apakah bubble adalah 'boom' (ledakan)
                this.speed = speed; // Kecepatan pergerakan bubble
            }

            move() {
                this.y += this.speed; // Menggerakkan bubble ke bawah
                if (this.y - this.size > canvas.height) {
                    this.y = -this.size; // Jika bubble melewati batas bawah layar, posisinya direset
                    this.x = Math.random() * canvas.width; // Posisi acak di atas layar
                }
            }
        }

        // Fungsi untuk memunculkan bubble
        function spawnBubbles(count) {
            bubbles = []; // Menghapus bubble yang ada
            for (let i = 0; i < count; i++) {
                const size = Math.random() * 30 + 10;
                const x = Math.random() * canvas.width;
                const y = Math.random() * canvas.height - canvas.height;
                const isBoom = Math.random() < 0.1;
                const color = isBoom ? 'red' : ['blue', 'green', 'yellow', 'purple', 'orange'][Math.floor(Math.random() * 5)];
                const speed = Math.random() * 2 + speedMultiplier;
                bubbles.push(new Bubble(x, y, size, color, isBoom, speed)); // Menambahkan bubble baru
            }
        }

        // Fungsi untuk inisialisasi game
        function initGame() {
            player = new Player(canvas.width / 2, canvas.height - 50, 20, 'red'); // Membuat pemain
            score = 0; // Skor dimulai dari 0
            isGameRunning = true; // Menandakan game sedang berjalan
            speedMultiplier = 1; // Kecepatan awal bubble
            timer = 0; // Menghitung waktu permainan
            timeLimit = 60; // Batas waktu permainan
            spawnBubbles(10); // Memunculkan 10 bubble
            gameLoop(); // Memulai loop permainan
        }

        // Fungsi untuk memperbarui status game
        function update() {
            player.updateBullets(); // Memperbarui posisi peluru

            bubbles.forEach((bubble, bubbleIndex) => {
                bubble.move(); // Menggerakkan bubble

                player.bullets.forEach((bullet, bulletIndex) => {
                    const dx = bubble.x - bullet.x;
                    const dy = bubble.y - bullet.y;
                    const distance = Math.sqrt(dx * dx + dy * dy); // Menghitung jarak antara bubble dan peluru

                    if (distance < bubble.size + bullet.size) { // Jika peluru mengenai bubble
                        if (bubble.isBoom) {
                            gameOver(); // Jika bubble ledakan, game selesai
                        }

                        score += 1; // Menambah skor
                        bubbles.splice(bubbleIndex, 1); // Menghapus bubble yang terkena peluru
                        player.bullets.splice(bulletIndex, 1); // Menghapus peluru yang mengenai bubble
                    }
                });
            });

            if (bubbles.length === 0) {
                spawnBubbles(10); // Jika semua bubble habis, spawn bubble baru
                speedMultiplier += 0.2; // Meningkatkan kecepatan bubble
            }

            timer++;
            if (timer >= timeLimit * 60) {
                gameOver(); // Jika waktu habis, game selesai
            }

            if (!isGameRunning) {
                return false;
            }

            return true;
        }

        // Fungsi untuk menggambar objek di layar
        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height); // Membersihkan layar
            ctx.font = '20px Arial';
            ctx.fillStyle = 'white';
            ctx.fillText(`Score: ${score}`, 10, 30); // Menampilkan skor
            ctx.fillText(`Time: ${Math.floor(timer / 60)}s`, canvas.width - 120, 30); // Menampilkan waktu
            player.draw(); // Menggambar player
            bubbles.forEach(bubble => bubble.draw()); // Menggambar semua bubble
        }

        // Fungsi untuk menjalankan game loop
        function gameLoop() {
            if (update()) {
                draw();
                requestAnimationFrame(gameLoop); // Memanggil game loop berikutnya
            }
        }

        // Fungsi untuk menangani game over
        function gameOver() {
            isGameRunning = false; // Menandakan game selesai
            overlay.innerHTML = `
                <h1>Game Over! Final Score: ${score}</h1>
                <h5>Thank you for playing</h5>
                <button id="restartButton">Restart Game</button>`;
            document.getElementById('restartButton').addEventListener('click', () => {
                overlay.style.display = 'none';
                initGame(); // Memulai ulang permainan
            });
            overlay.style.display = 'flex'; // Menampilkan overlay game over
        }

        // Event listener untuk memulai game
        startButton.addEventListener('click', () => {
            overlay.style.display = 'none'; // Menghilangkan overlay
            initGame(); // Inisialisasi game
        });

        // Event listener untuk menggerakkan player sesuai posisi mouse
        window.addEventListener('mousemove', (event) => {
            const rect = canvas.getBoundingClientRect();
            player.x = Math.max(player.size, Math.min(event.clientX - rect.left, canvas.width - player.size)); // Membatasi gerakan player
        });

        // Event listener untuk menembakkan peluru
        window.addEventListener('click', () => {
            if (isGameRunning) {
                player.shoot(); // Menembakkan peluru jika game berjalan
            }
        });
    </script>
</body>

</html>
