<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Game Ular Tangga Interactive</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Font Awesome for Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Canvas Confetti -->
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Fredoka:wght@400;600;700&family=Nunito:wght@700;800&display=swap" rel="stylesheet">

    <style>
        body {
            font-family: 'Fredoka', sans-serif;
            background: linear-gradient(135deg, #0f172a 0%, #1e1b4b 100%);
            touch-action: manipulation;
        }

        /* 3D Dice styling */
        .dice-container {
            perspective: 1000px;
        }
        .dice {
            width: 70px;
            height: 70px;
            position: relative;
            transform-style: preserve-3d;
            transition: transform 1s cubic-bezier(0.2, 0.8, 0.3, 1.15);
            cursor: pointer;
        }
        .dice-face {
            position: absolute;
            width: 70px;
            height: 70px;
            background: #ffffff;
            border: 3px solid #cbd5e1;
            border-radius: 16px;
            display: grid;
            grid-template: repeat(3, 1fr) / repeat(3, 1fr);
            padding: 6px;
            box-shadow: inset 0 0 10px rgba(0, 0, 0, 0.15);
        }
        .dot {
            width: 12px;
            height: 12px;
            background-color: #ef4444;
            border-radius: 50%;
            align-self: center;
            justify-self: center;
            box-shadow: inset 0 2px 2px rgba(0,0,0,0.3);
        }
        .dot-black {
            background-color: #1e293b;
        }

        /* Dice face positions */
        .face-1 { transform: translateZ(35px); }
        .face-2 { transform: rotateY(180deg) translateZ(35px); }
        .face-3 { transform: rotateY(-90deg) translateZ(35px); }
        .face-4 { transform: rotateY(90deg) translateZ(35px); }
        .face-5 { transform: rotateX(-90deg) translateZ(35px); }
        .face-6 { transform: rotateX(90deg) translateZ(35px); }

        /* Board grid layout */
        .board-grid {
            display: grid;
            grid-template-columns: repeat(10, 1fr);
            grid-template-rows: repeat(10, 1fr);
            aspect-ratio: 1 / 1;
            position: relative;
            border-radius: 16px;
            overflow: hidden;
            box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.5), 0 8px 10px -6px rgba(0, 0, 0, 0.5);
        }

        /* Player token animations */
        .player-token {
            transition: all 0.4s cubic-bezier(0.34, 1.56, 0.64, 1);
            filter: drop-shadow(0px 4px 6px rgba(0, 0, 0, 0.4));
            z-index: 20;
        }

        /* Custom scrollbar */
        ::-webkit-scrollbar {
            width: 6px;
        }
        ::-webkit-scrollbar-track {
            background: rgba(15, 23, 42, 0.6);
        }
        ::-webkit-scrollbar-thumb {
            background: #475569;
            border-radius: 4px;
        }
    </style>
</head>
<body class="min-h-screen text-slate-100 flex flex-col justify-between items-center p-2 sm:p-4">

    <!-- Header -->
    <header class="text-center my-2 sm:my-3">
        <h1 class="text-3xl sm:text-5xl font-extrabold tracking-wider text-transparent bg-clip-text bg-gradient-to-r from-amber-400 via-rose-400 to-indigo-400 drop-shadow-md">
            🐍 ULAR TANGGA 🪜
        </h1>
        <p class="text-xs sm:text-sm text-slate-400 mt-1 font-medium">Capai Kotak 100 Pertama untuk Menang!</p>
    </header>

    <!-- Main Container -->
    <main class="w-full max-w-6xl flex flex-col lg:flex-row items-center lg:items-start justify-center gap-6 my-auto">
        
        <!-- Left Panel: Game Board -->
        <div class="relative w-full max-w-[550px] aspect-square flex-shrink-0">
            <!-- Board Canvas Overlay for Snakes & Ladders -->
            <div id="board" class="board-grid bg-slate-800 border-4 border-slate-700 relative">
                <!-- Squares injected by JS -->
            </div>
            <!-- Canvas for drawing Snakes & Ladders connections -->
            <canvas id="overlayCanvas" class="absolute inset-0 pointer-events-none z-10 w-full h-full"></canvas>
            <!-- Layer for Player Tokens -->
            <div id="tokensLayer" class="absolute inset-0 pointer-events-none z-20 w-full h-full"></div>
        </div>

        <!-- Right Panel: Controls & Status -->
        <div class="w-full max-w-[450px] flex flex-col gap-4">
            
            <!-- Turn & Game Action Card -->
            <div class="bg-slate-800/80 backdrop-blur-md p-5 rounded-2xl border border-slate-700 shadow-xl flex flex-col items-center text-center">
                
                <!-- Turn Indicator -->
                <div id="turnBadge" class="inline-flex items-center gap-2 px-4 py-1.5 rounded-full text-sm font-bold bg-slate-700 text-amber-400 mb-4 border border-slate-600">
                    <span class="w-3 h-3 rounded-full bg-amber-400 animate-pulse"></span>
                    <span id="turnText">Giliran: Player 1</span>
                </div>

                <!-- Dice Area -->
                <div class="flex items-center justify-center my-2 gap-6 w-full">
                    <div class="dice-container py-2">
                        <div id="dice" class="dice" onclick="rollDice()">
                            <!-- Face 1 -->
                            <div class="dice-face face-1">
                                <div class="dot dot-black col-start-2 row-start-2"></div>
                            </div>
                            <!-- Face 2 -->
                            <div class="dice-face face-2">
                                <div class="dot dot-black col-start-1 row-start-1"></div>
                                <div class="dot dot-black col-start-3 row-start-3"></div>
                            </div>
                            <!-- Face 3 -->
                            <div class="dice-face face-3">
                                <div class="dot dot-black col-start-1 row-start-1"></div>
                                <div class="dot dot-black col-start-2 row-start-2"></div>
                                <div class="dot dot-black col-start-3 row-start-3"></div>
                            </div>
                            <!-- Face 4 -->
                            <div class="dice-face face-4">
                                <div class="dot dot-black col-start-1 row-start-1"></div>
                                <div class="dot dot-black col-start-3 row-start-1"></div>
                                <div class="dot dot-black col-start-1 row-start-3"></div>
                                <div class="dot dot-black col-start-3 row-start-3"></div>
                            </div>
                            <!-- Face 5 -->
                            <div class="dice-face face-5">
                                <div class="dot dot-black col-start-1 row-start-1"></div>
                                <div class="dot dot-black col-start-3 row-start-1"></div>
                                <div class="dot dot-black col-start-2 row-start-2"></div>
                                <div class="dot dot-black col-start-1 row-start-3"></div>
                                <div class="dot dot-black col-start-3 row-start-3"></div>
                            </div>
                            <!-- Face 6 -->
                            <div class="dice-face face-6">
                                <div class="dot dot-black col-start-1 row-start-1"></div>
                                <div class="dot dot-black col-start-3 row-start-1"></div>
                                <div class="dot dot-black col-start-1 row-start-2"></div>
                                <div class="dot dot-black col-start-3 row-start-2"></div>
                                <div class="dot dot-black col-start-1 row-start-3"></div>
                                <div class="dot dot-black col-start-3 row-start-3"></div>
                            </div>
                        </div>
                    </div>

                    <div class="flex flex-col text-left">
                        <span class="text-xs text-slate-400 uppercase tracking-wider font-semibold">Hasil Dadu</span>
                        <span id="diceResultText" class="text-3xl font-black text-amber-400">?</span>
                    </div>
                </div>

                <!-- Roll Button -->
                <button id="rollBtn" onclick="rollDice()" class="w-full mt-3 py-3.5 px-6 rounded-xl font-extrabold text-lg bg-gradient-to-r from-amber-500 to-orange-500 hover:from-amber-400 hover:to-orange-400 text-slate-950 shadow-lg shadow-orange-500/30 transform active:scale-95 transition duration-150 flex items-center justify-center gap-2">
                    <i class="fa-solid fa-dice text-xl"></i> LEMPAR DADU
                </button>
            </div>

            <!-- Player Scores/Status Panel -->
            <div class="bg-slate-800/80 backdrop-blur-md p-4 rounded-2xl border border-slate-700 shadow-xl">
                <h3 class="text-sm font-bold text-slate-400 uppercase tracking-wider mb-3 flex items-center justify-between">
                    <span>Status Pemain</span>
                    <button onclick="toggleAudio()" id="audioBtn" class="text-slate-400 hover:text-amber-400 text-base transition">
                        <i class="fa-solid fa-volume-high" id="audioIcon"></i>
                    </button>
                </h3>
                <div id="playersList" class="grid grid-cols-2 gap-2.5">
                    <!-- Populated dynamically -->
                </div>
            </div>

            <!-- Options Panel / Setup -->
            <div class="bg-slate-800/80 backdrop-blur-md p-4 rounded-2xl border border-slate-700 shadow-xl flex items-center justify-between gap-2">
                <div class="flex flex-col">
                    <label for="playerCount" class="text-xs font-bold text-slate-400">JUMLAH PEMAIN</label>
                    <select id="playerCount" onchange="resetGame()" class="bg-slate-900 text-slate-200 text-sm font-bold border border-slate-700 rounded-lg p-2 mt-1 focus:outline-none focus:border-amber-500">
                        <option value="1">1 Pemain vs Bot AI</option>
                        <option value="2" selected>2 Pemain (P1 vs P2)</option>
                        <option value="3">3 Pemain</option>
                        <option value="4">4 Pemain</option>
                    </select>
                </div>
                <button onclick="resetGame()" class="py-2.5 px-4 bg-slate-700 hover:bg-slate-600 border border-slate-600 rounded-xl font-bold text-sm text-slate-200 transition flex items-center gap-1.5 self-end">
                    <i class="fa-solid fa-rotate-right"></i> Reset
                </button>
            </div>

            <!-- Game Legend / Info -->
            <div class="text-xs text-slate-400 bg-slate-900/40 p-3 rounded-xl border border-slate-800 flex justify-around text-center">
                <div class="flex items-center gap-1.5">
                    <span class="w-3 h-3 rounded-full bg-emerald-500 inline-block"></span>
                    <span>🪜 Tangga: Naik</span>
                </div>
                <div class="flex items-center gap-1.5">
                    <span class="w-3 h-3 rounded-full bg-rose-500 inline-block"></span>
                    <span>🐍 Ular: Turun</span>
                </div>
            </div>

        </div>
    </main>

    <!-- Winner Modal -->
    <div id="winnerModal" class="fixed inset-0 bg-slate-950/80 backdrop-blur-md flex items-center justify-center p-4 z-50 opacity-0 pointer-events-none transition-opacity duration-300">
        <div class="bg-slate-800 border-2 border-amber-500/50 rounded-3xl p-6 sm:p-8 max-w-sm w-full text-center shadow-2xl transform scale-90 transition-transform duration-300" id="winnerModalCard">
            <div class="w-20 h-20 bg-amber-500/20 text-amber-400 rounded-full flex items-center justify-center mx-auto mb-4 border border-amber-500/40 text-4xl animate-bounce">
                👑
            </div>
            <h2 class="text-2xl sm:text-3xl font-black text-white mb-1">PEMENANG!</h2>
            <p id="winnerText" class="text-amber-400 font-bold text-lg mb-6">Player 1 Menang!</p>
            <button onclick="closeWinnerModal(); resetGame();" class="w-full py-3.5 px-6 rounded-xl font-extrabold text-slate-950 bg-gradient-to-r from-amber-400 to-amber-500 hover:from-amber-300 hover:to-amber-400 shadow-lg shadow-amber-500/30 transition">
                Main Lagi <i class="fa-solid fa-play ml-1"></i>
            </button>
        </div>
    </div>

    <!-- Audio & Game Logic Scripts -->
    <script>
        /* --- Web Audio API Synth Effects --- */
        let audioEnabled = true;
        const AudioCtx = window.AudioContext || window.webkitAudioContext;
        let audioCtx = null;

        function initAudio() {
            if (!audioCtx) {
                audioCtx = new AudioCtx();
            }
        }

        function playSound(type) {
            if (!audioEnabled) return;
            initAudio();
            if (audioCtx.state === 'suspended') {
                audioCtx.resume();
            }

            const now = audioCtx.currentTime;

            if (type === 'roll') {
                // Noise / rattle sound for dice
                const osc = audioCtx.createOscillator();
                const gain = audioCtx.createGain();
                osc.type = 'square';
                osc.frequency.setValueAtTime(150, now);
                osc.frequency.exponentialRampToValueAtTime(40, now + 0.15);
                gain.gain.setValueAtTime(0.1, now);
                gain.gain.exponentialRampToValueAtTime(0.01, now + 0.15);
                osc.connect(gain);
                gain.connect(audioCtx.destination);
                osc.start(now);
                osc.stop(now + 0.15);
            } else if (type === 'step') {
                // Short crisp tap sound
                const osc = audioCtx.createOscillator();
                const gain = audioCtx.createGain();
                osc.type = 'sine';
                osc.frequency.setValueAtTime(300, now);
                osc.frequency.exponentialRampToValueAtTime(600, now + 0.05);
                gain.gain.setValueAtTime(0.15, now);
                gain.gain.exponentialRampToValueAtTime(0.01, now + 0.05);
                osc.connect(gain);
                gain.connect(audioCtx.destination);
                osc.start(now);
                osc.stop(now + 0.05);
            } else if (type === 'ladder') {
                // Ascending melody
                [261.63, 329.63, 392.00, 523.25].forEach((freq, idx) => {
                    const osc = audioCtx.createOscillator();
                    const gain = audioCtx.createGain();
                    osc.type = 'triangle';
                    osc.frequency.setValueAtTime(freq, now + idx * 0.08);
                    gain.gain.setValueAtTime(0.2, now + idx * 0.08);
                    gain.gain.exponentialRampToValueAtTime(0.01, now + idx * 0.08 + 0.15);
                    osc.connect(gain);
                    gain.connect(audioCtx.destination);
                    osc.start(now + idx * 0.08);
                    osc.stop(now + idx * 0.08 + 0.15);
                });
            } else if (type === 'snake') {
                // Descending wobble
                const osc = audioCtx.createOscillator();
                const gain = audioCtx.createGain();
                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(400, now);
                osc.frequency.exponentialRampToValueAtTime(100, now + 0.35);
                gain.gain.setValueAtTime(0.15, now);
                gain.gain.exponentialRampToValueAtTime(0.01, now + 0.35);
                osc.connect(gain);
                gain.connect(audioCtx.destination);
                osc.start(now);
                osc.stop(now + 0.35);
            } else if (type === 'win') {
                // Victory fanfare
                const notes = [523.25, 659.25, 783.99, 1046.50];
                notes.forEach((freq, idx) => {
                    const osc = audioCtx.createOscillator();
                    const gain = audioCtx.createGain();
                    osc.type = 'sine';
                    osc.frequency.setValueAtTime(freq, now + idx * 0.12);
                    gain.gain.setValueAtTime(0.25, now + idx * 0.12);
                    gain.gain.exponentialRampToValueAtTime(0.01, now + idx * 0.12 + 0.3);
                    osc.connect(gain);
                    gain.connect(audioCtx.destination);
                    osc.start(now + idx * 0.12);
                    osc.stop(now + idx * 0.12 + 0.3);
                });
            }
        }

        function toggleAudio() {
            audioEnabled = !audioEnabled;
            const icon = document.getElementById('audioIcon');
            if (audioEnabled) {
                icon.className = 'fa-solid fa-volume-high text-amber-400';
            } else {
                icon.className = 'fa-solid fa-volume-xmark text-slate-500';
            }
        }

        /* --- Game State & Data --- */
        const BOARD_SIZE = 10;
        const TOTAL_SQUARES = 100;

        // Player configurations
        const PLAYER_CONFIGS = [
            { id: 0, name: 'Pemain 1', color: '#ef4444', border: '#b91c1c', icon: '🔴', isAI: false },
            { id: 1, name: 'Pemain 2', color: '#3b82f6', border: '#1d4ed8', icon: '🔵', isAI: false },
            { id: 2, name: 'Pemain 3', color: '#10b981', border: '#047857', icon: '🟢', isAI: false },
            { id: 3, name: 'Pemain 4', color: '#f59e0b', border: '#b45309', icon: '🟡', isAI: false }
        ];

        // Snakes and Ladders mapping (Start -> End)
        const LADDERS = {
            4: 14,
            9: 31,
            20: 38,
            28: 84,
            40: 59,
            51: 67,
            63: 81,
            71: 91
        };

        const SNAKES = {
            17: 7,
            54: 34,
            62: 19,
            64: 60,
            87: 24,
            93: 73,
            95: 75,
            99: 78
        };

        let players = [];
        let activePlayerIdx = 0;
        let isRolling = false;
        let isMoving = false;

        /* --- Board & Grid Calculations --- */

        // Get (row, col) in 0-indexed terms from square number (1 to 100)
        function getSquareCoordinates(squareNum) {
            const index = squareNum - 1;
            const rowFromBottom = Math.floor(index / BOARD_SIZE);
            const row = (BOARD_SIZE - 1) - rowFromBottom; // 0 is top row
            
            let col;
            if (rowFromBottom % 2 === 0) {
                // Left to right
                col = index % BOARD_SIZE;
            } else {
                // Right to left (Boustrophedon style)
                col = (BOARD_SIZE - 1) - (index % BOARD_SIZE);
            }
            return { row, col };
        }

        // Initialize Board Squares HTML
        function buildBoard() {
            const boardEl = document.getElementById('board');
            boardEl.innerHTML = '';

            for (let row = 0; row < BOARD_SIZE; row++) {
                for (let col = 0; col < BOARD_SIZE; col++) {
                    const squareNum = getSquareNumber(row, col);
                    const squareDiv = document.createElement('div');
                    squareDiv.id = `square-${squareNum}`;

                    // Alternating cell colors
                    const isEvenRow = (BOARD_SIZE - 1 - row) % 2 === 0;
                    const isEvenCol = col % 2 === 0;
                    const isLight = (isEvenRow && isEvenCol) || (!isEvenRow && !isEvenCol);

                    let bgClass = isLight ? 'bg-slate-700/60' : 'bg-slate-800/80';

                    // Special background styling for Start and Finish
                    if (squareNum === 100) bgClass = 'bg-amber-500/30 font-black text-amber-300';
                    if (squareNum === 1) bgClass = 'bg-emerald-500/20 font-bold text-emerald-300';

                    squareDiv.className = `${bgClass} border border-slate-700/40 relative flex items-start justify-start p-1 select-none text-[10px] sm:text-xs font-bold text-slate-400`;
                    
                    // Square label
                    squareDiv.innerHTML = `<span>${squareNum}</span>`;
                    boardEl.appendChild(squareDiv);
                }
            }
        }

        function getSquareNumber(row, col) {
            const rowFromBottom = (BOARD_SIZE - 1) - row;
            if (rowFromBottom % 2 === 0) {
                return (rowFromBottom * BOARD_SIZE) + col + 1;
            } else {
                return (rowFromBottom * BOARD_SIZE) + (BOARD_SIZE - col);
            }
        }

        /* --- Drawing Canvas Layer (Snakes & Ladders Connections) --- */
        function drawOverlay() {
            const canvas = document.getElementById('overlayCanvas');
            const board = document.getElementById('board');
            const rect = board.getBoundingClientRect();

            canvas.width = rect.width;
            canvas.height = rect.height;

            const ctx = canvas.getContext('2d');
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            const cellWidth = canvas.width / BOARD_SIZE;
            const cellHeight = canvas.height / BOARD_SIZE;

            // Helper to get center pixel coords of a square
            function getCenter(sq) {
                const { row, col } = getSquareCoordinates(sq);
                return {
                    x: col * cellWidth + cellWidth / 2,
                    y: row * cellHeight + cellHeight / 2
                };
            }

            // Draw Ladders
            for (const [start, end] of Object.entries(LADDERS)) {
                const p1 = getCenter(Number(start));
                const p2 = getCenter(Number(end));

                ctx.save();
                ctx.strokeStyle = '#10b981'; // Green
                ctx.lineWidth = Math.max(4, cellWidth * 0.12);
                ctx.lineCap = 'round';

                // Parallel ladder rails
                const dx = p2.x - p1.x;
                const dy = p2.y - p1.y;
                const angle = Math.atan2(dy, dx);
                const offset = cellWidth * 0.15;

                const ox = Math.sin(angle) * offset;
                const oy = -Math.cos(angle) * offset;

                // Rail 1
                ctx.beginPath();
                ctx.moveTo(p1.x + ox, p1.y + oy);
                ctx.lineTo(p2.x + ox, p2.y + oy);
                ctx.stroke();

                // Rail 2
                ctx.beginPath();
                ctx.moveTo(p1.x - ox, p1.y - oy);
                ctx.lineTo(p2.x - ox, p2.y - oy);
                ctx.stroke();

                // Rungs
                ctx.strokeStyle = '#34d399';
                ctx.lineWidth = Math.max(2, cellWidth * 0.06);
                const steps = 6;
                for (let i = 1; i < steps; i++) {
                    const t = i / steps;
                    const rx = p1.x + dx * t;
                    const ry = p1.y + dy * t;
                    ctx.beginPath();
                    ctx.moveTo(rx + ox, ry + oy);
                    ctx.lineTo(rx - ox, ry - oy);
                    ctx.stroke();
                }

                ctx.restore();
            }

            // Draw Snakes
            for (const [start, end] of Object.entries(SNAKES)) {
                const head = getCenter(Number(start));
                const tail = getCenter(Number(end));

                ctx.save();
                ctx.strokeStyle = '#ef4444'; // Red snake
                ctx.lineWidth = Math.max(6, cellWidth * 0.18);
                ctx.lineCap = 'round';

                // Curved wavy body
                const midX = (head.x + tail.x) / 2 + (head.y > tail.y ? 30 : -30);
                const midY = (head.y + tail.y) / 2;

                ctx.beginPath();
                ctx.moveTo(head.x, head.y);
                ctx.quadraticCurveTo(midX, midY, tail.x, tail.y);
                ctx.stroke();

                // Snake Head
                ctx.fillStyle = '#b91c1c';
                ctx.beginPath();
                ctx.arc(head.x, head.y, cellWidth * 0.18, 0, Math.PI * 2);
                ctx.fill();

                // Snake Eyes
                ctx.fillStyle = '#ffffff';
                ctx.beginPath();
                ctx.arc(head.x - 3, head.y - 3, 3, 0, Math.PI * 2);
                ctx.arc(head.x + 3, head.y - 3, 3, 0, Math.PI * 2);
                ctx.fill();

                ctx.restore();
            }
        }

        /* --- Player Tokens & Setup --- */
        function createPlayerTokens() {
            const layer = document.getElementById('tokensLayer');
            layer.innerHTML = '';

            players.forEach((player, idx) => {
                const token = document.createElement('div');
                token.id = `player-token-${idx}`;
                token.className = 'player-token absolute flex items-center justify-center font-bold shadow-md rounded-full border-2 text-xs sm:text-sm';
                token.style.backgroundColor = player.color;
                token.style.borderColor = player.border;
                token.style.color = '#ffffff';
                token.innerText = idx + 1;

                layer.appendChild(token);
            });

            updateTokenPositions();
        }

        function updateTokenPositions() {
            const board = document.getElementById('board');
            const rect = board.getBoundingClientRect();
            const cellWidth = rect.width / BOARD_SIZE;
            const cellHeight = rect.height / BOARD_SIZE;
            const tokenSize = Math.max(18, cellWidth * 0.42);

            // Group players by position to offset overlapping tokens
            const posMap = {};

            players.forEach((p, idx) => {
                if (!posMap[p.position]) posMap[p.position] = [];
                posMap[p.position].push(idx);
            });

            Object.entries(posMap).forEach(([sq, pIndexes]) => {
                const { row, col } = getSquareCoordinates(Number(sq));
                const baseX = col * cellWidth + cellWidth / 2;
                const baseY = row * cellHeight + cellHeight / 2;

                pIndexes.forEach((pIdx, offsetIdx) => {
                    const token = document.getElementById(`player-token-${pIdx}`);
                    if (!token) return;

                    token.style.width = `${tokenSize}px`;
                    token.style.height = `${tokenSize}px`;

                    // Apply slight offset if multiple tokens on same square
                    let offsetX = 0;
                    let offsetY = 0;
                    if (pIndexes.length > 1) {
                        const angle = (offsetIdx / pIndexes.length) * Math.PI * 2;
                        offsetX = Math.cos(angle) * (tokenSize * 0.3);
                        offsetY = Math.sin(angle) * (tokenSize * 0.3);
                    }

                    token.style.left = `${baseX + offsetX - tokenSize / 2}px`;
                    token.style.top = `${baseY + offsetY - tokenSize / 2}px`;
                });
            });
        }

        /* --- Game Controls & Turn Logic --- */
        function resetGame() {
            const count = parseInt(document.getElementById('playerCount').value);
            players = [];

            for (let i = 0; i < count; i++) {
                const config = { ...PLAYER_CONFIGS[i] };
                if (count === 1 && i === 1) {
                    // Bot config
                    config.name = 'Bot AI 🤖';
                    config.isAI = true;
                }
                players.push({
                    ...config,
                    position: 1
                });
            }

            // If 1 player selected, add Bot AI as 2nd player automatically
            if (count === 1) {
                players.push({
                    ...PLAYER_CONFIGS[1],
                    name: 'Bot AI 🤖',
                    isAI: true,
                    position: 1
                });
            }

            activePlayerIdx = 0;
            isRolling = false;
            isMoving = false;

            document.getElementById('diceResultText').innerText = '?';
            updatePlayersListUI();
            updateTurnUI();
            createPlayerTokens();
            drawOverlay();
        }

        function updateTurnUI() {
            const activePlayer = players[activePlayerIdx];
            const badge = document.getElementById('turnBadge');
            const text = document.getElementById('turnText');
            const btn = document.getElementById('rollBtn');

            text.innerText = `Giliran: ${activePlayer.name}`;
            badge.style.borderColor = activePlayer.color;

            if (activePlayer.isAI) {
                btn.disabled = true;
                btn.classList.add('opacity-50', 'cursor-not-allowed');
                // Trigger AI roll automatically after short delay
                setTimeout(() => {
                    if (players[activePlayerIdx].isAI && !isRolling && !isMoving) {
                        rollDice();
                    }
                }, 1000);
            } else {
                btn.disabled = false;
                btn.classList.remove('opacity-50', 'cursor-not-allowed');
            }
        }

        function updatePlayersListUI() {
            const listEl = document.getElementById('playersList');
            listEl.innerHTML = '';

            players.forEach((p, idx) => {
                const isActive = idx === activePlayerIdx;
                const card = document.createElement('div');
                card.className = `p-2.5 rounded-xl border transition-all flex items-center justify-between ${
                    isActive 
                        ? 'bg-slate-700/90 border-amber-400 shadow-md ring-1 ring-amber-400/50' 
                        : 'bg-slate-900/50 border-slate-700/60 opacity-80'
                }`;

                card.innerHTML = `
                    <div class="flex items-center gap-2 overflow-hidden">
                        <span class="w-3.5 h-3.5 rounded-full flex-shrink-0" style="background-color: ${p.color}"></span>
                        <span class="text-xs sm:text-sm font-bold text-slate-200 truncate">${p.name}</span>
                    </div>
                    <span class="text-xs font-extrabold bg-slate-800 px-2 py-0.5 rounded-md text-amber-400 border border-slate-700">
                        ${p.position}
                    </span>
                `;
                listEl.appendChild(card);
            });
        }

        /* --- Rolling Dice & Movement --- */
        function rollDice() {
            if (isRolling || isMoving) return;
            isRolling = true;

            playSound('roll');

            const dice = document.getElementById('dice');
            const rollVal = Math.floor(Math.random() * 6) + 1;

            // Rotation angles corresponding to dice face showing on top
            const faceRotations = {
                1: { x: 0, y: 0 },
                2: { x: 0, y: -180 },
                3: { x: 0, y: 90 },
                4: { x: 0, y: -90 },
                5: { x: 90, y: 0 },
                6: { x: -90, y: 0 }
            };

            const targetRot = faceRotations[rollVal];
            // Add extra full turns for rolling animation
            const extraX = (Math.floor(Math.random() * 2) + 2) * 360;
            const extraY = (Math.floor(Math.random() * 2) + 2) * 360;

            dice.style.transform = `rotateX(${targetRot.x + extraX}deg) rotateY(${targetRot.y + extraY}deg)`;

            setTimeout(() => {
                document.getElementById('diceResultText').innerText = rollVal;
                isRolling = false;
                movePlayer(activePlayerIdx, rollVal);
            }, 1000);
        }

        async function movePlayer(playerIdx, steps) {
            isMoving = true;
            const player = players[playerIdx];

            for (let i = 0; i < steps; i++) {
                if (player.position >= TOTAL_SQUARES) break;
                player.position += 1;
                updateTokenPositions();
                updatePlayersListUI();
                playSound('step');
                await new Promise(r => setTimeout(r, 280));
            }

            // Check Ladder or Snake connection
            const currentPos = player.position;
            if (LADDERS[currentPos]) {
                await new Promise(r => setTimeout(r, 200));
                playSound('ladder');
                player.position = LADDERS[currentPos];
                updateTokenPositions();
                updatePlayersListUI();
            } else if (SNAKES[currentPos]) {
                await new Promise(r => setTimeout(r, 200));
                playSound('snake');
                player.position = SNAKES[currentPos];
                updateTokenPositions();
                updatePlayersListUI();
            }

            // Win Condition Check
            if (player.position === TOTAL_SQUARES) {
                playSound('win');
                showWinnerModal(player.name);
                isMoving = false;
                return;
            }

            // Switch turn (if roll is 6, player gets extra turn!)
            if (steps !== 6) {
                activePlayerIdx = (activePlayerIdx + 1) % players.length;
            }

            updatePlayersListUI();
            updateTurnUI();
            isMoving = false;
        }

        /* --- Winner Modal & Celebration --- */
        function showWinnerModal(winnerName) {
            document.getElementById('winnerText').innerText = `${winnerName} Menang! 🎉`;
            const modal = document.getElementById('winnerModal');
            const card = document.getElementById('winnerModalCard');

            modal.classList.remove('opacity-0', 'pointer-events-none');
            card.classList.remove('scale-90');
            card.classList.add('scale-100');

            // Confetti animation
            if (typeof confetti === 'function') {
                confetti({
                    particleCount: 100,
                    spread: 70,
                    origin: { y: 0.6 }
                });
            }
        }

        function closeWinnerModal() {
            const modal = document.getElementById('winnerModal');
            const card = document.getElementById('winnerModalCard');

            modal.classList.add('opacity-0', 'pointer-events-none');
            card.classList.remove('scale-100');
            card.classList.add('scale-90');
        }

        /* --- Event Listeners & Window Resize Handling --- */
        window.addEventListener('resize', () => {
            drawOverlay();
            updateTokenPositions();
        });

        window.onload = function() {
            buildBoard();
            resetGame();
        };
    </script>
</body>
</html>
