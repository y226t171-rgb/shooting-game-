<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>スペースシューター</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            overflow: hidden;
            background-color: #050510;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            color: #fff;
            touch-action: none; /* タッチ操作時のスクロールを防ぐ */
        }

        canvas {
            display: block;
            width: 100vw;
            height: 100vh;
        }

        #ui-layer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
        }

        .screen {
            display: none;
            text-align: center;
            background: rgba(0, 0, 0, 0.7);
            padding: 40px;
            border-radius: 15px;
            border: 1px solid #0df;
            box-shadow: 0 0 20px rgba(0, 221, 255, 0.5);
            pointer-events: auto;
        }

        .screen.active {
            display: flex;
            flex-direction: column;
            align-items: center;
            animation: fadeIn 0.5s ease-out;
        }

        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(-20px); }
            to { opacity: 1; transform: translateY(0); }
        }

        h1 {
            font-size: 2.5rem;
            margin: 0 0 20px 0;
            color: #0df;
            text-shadow: 0 0 10px #0df;
            letter-spacing: 2px;
        }

        p {
            font-size: 1.1rem;
            margin-bottom: 30px;
            color: #ccc;
            line-height: 1.5;
        }

        button {
            padding: 15px 40px;
            font-size: 1.5rem;
            background: linear-gradient(45deg, #0df, #0055ff);
            color: #fff;
            border: none;
            border-radius: 30px;
            cursor: pointer;
            font-weight: bold;
            text-transform: uppercase;
            letter-spacing: 1px;
            box-shadow: 0 5px 15px rgba(0, 221, 255, 0.4);
            transition: all 0.2s;
        }

        button:hover {
            transform: scale(1.05) translateY(-2px);
            box-shadow: 0 8px 20px rgba(0, 221, 255, 0.6);
        }

        button:active {
            transform: scale(0.95);
        }

        .difficulty-buttons {
            display: flex;
            gap: 15px;
            margin-bottom: 20px;
            flex-wrap: wrap;
            justify-content: center;
        }

        .diff-btn {
            padding: 12px 25px;
            font-size: 1.2rem;
            border-radius: 25px;
        }
        
        .btn-easy { background: linear-gradient(45deg, #0f0, #009900); }
        .btn-normal { background: linear-gradient(45deg, #0df, #0055ff); }
        .btn-hard { background: linear-gradient(45deg, #f05, #990000); }

        #hud {
            position: absolute;
            top: 20px;
            left: 20px;
            display: none;
            font-size: 1.5rem;
            font-weight: bold;
            font-family: monospace;
            text-shadow: 0 0 5px #0df;
            color: #0df;
            pointer-events: none;
        }

        #controls-container {
            position: absolute;
            top: 20px;
            right: 20px;
            display: none;
            align-items: center;
            gap: 15px;
            z-index: 10;
        }

        #volume-control {
            display: flex;
            align-items: center;
            gap: 5px;
            background: rgba(0, 0, 0, 0.6);
            padding: 5px 12px;
            border-radius: 20px;
            border: 1px solid #0df;
            box-shadow: 0 0 10px rgba(0, 221, 255, 0.3);
        }

        #volume-slider {
            width: 80px;
            cursor: pointer;
            accent-color: #0df;
        }

        #quit-btn {
            padding: 8px 16px;
            font-size: 1rem;
            background: rgba(255, 0, 85, 0.3);
            color: #fff;
            border: 1px solid #f05;
            border-radius: 8px;
            cursor: pointer;
            text-shadow: 0 0 5px #f05;
            box-shadow: 0 0 10px rgba(255, 0, 85, 0.3);
            transition: all 0.2s;
        }

        #quit-btn:hover {
            background: rgba(255, 0, 85, 0.6);
            transform: scale(1.05);
        }

        .mobile-hint {
            display: none;
            font-size: 0.9rem;
            color: #888;
            margin-top: 15px;
        }

        @media (max-width: 768px) {
            h1 { font-size: 2rem; }
            button { font-size: 1.2rem; padding: 12px 30px; }
            .mobile-hint { display: block; }
        }
    </style>
</head>
<body>

    <canvas id="gameCanvas"></canvas>
    
    <div id="hud">SCORE: <span id="score">0</span></div>
    
    <div id="controls-container">
        <div id="volume-control">
            <span style="font-size: 1.2rem;">🔊</span>
            <input type="range" id="volume-slider" min="0" max="1" step="0.01" value="0.5">
        </div>
        <button id="quit-btn">QUIT</button>
    </div>

    <div id="ui-layer">
        <!-- スタート画面 -->
        <div id="start-screen" class="screen active">
            <h1>スペースシューター</h1>
            <p>
                操作方法<br>
                マウス移動 または 画面スワイプで自機を移動<br>
                弾は自動で発射されます<br>
                <span style="color: #ff0; font-size: 0.9em;">※音が鳴りますので音量にご注意ください</span>
            </p>
            <div class="difficulty-buttons">
                <button class="diff-btn btn-easy" data-level="EASY">EASY</button>
                <button class="diff-btn btn-normal" data-level="NORMAL">NORMAL</button>
                <button class="diff-btn btn-hard" data-level="HARD">HARD</button>
            </div>
            <div class="mobile-hint">※スマートフォンは横画面推奨</div>
        </div>

        <!-- ゲームオーバー画面 -->
        <div id="gameover-screen" class="screen">
            <h1 style="color: #f05; text-shadow: 0 0 10px #f05;">GAME OVER</h1>
            <p>最終スコア: <strong id="final-score" style="font-size: 1.5em; color: #fff;">0</strong></p>
            <button id="restart-btn">RETRY</button>
        </div>
    </div>

    <script>
        /**
         * Web Audio APIによる効果音・BGM生成
         */
        let audioCtx;
        let masterGain;

        function initAudio() {
            if (!audioCtx) {
                audioCtx = new (window.AudioContext || window.webkitAudioContext)();
                
                // マスターボリュームノードの作成
                masterGain = audioCtx.createGain();
                masterGain.connect(audioCtx.destination);
                
                // スライダーの初期値を適用
                const volSlider = document.getElementById('volume-slider');
                masterGain.gain.value = volSlider.value;
                
                // スライダー操作時に音量を変更
                volSlider.addEventListener('input', (e) => {
                    masterGain.gain.value = e.target.value;
                });
            }
            if (audioCtx.state === 'suspended') {
                audioCtx.resume();
            }
        }

        // BGM用のデータと状態
        let nextNoteTime = 0;
        let bgmStep = 0;
        const bgmTempo = 140; // BPM
        const noteDuration = 60.0 / bgmTempo / 2; // 8分音符の長さ

        // メロディの周波数配列（0は休符）
        const bgmNotes = [
            392.00, 0, 392.00, 0, 466.16, 0, 523.25, 0, // G4, -, G4, -, A#4, -, C5, -
            392.00, 0, 392.00, 0, 523.25, 0, 587.33, 0, // G4, -, G4, -, C5, -, D5, -
            392.00, 0, 392.00, 0, 466.16, 0, 523.25, 0, // G4, -, G4, -, A#4, -, C5, -
            587.33, 0, 523.25, 0, 466.16, 0, 466.16, 0  // D5, -, C5, -, A#4, -, A#4, -
        ];
        
        // ベースラインの周波数配列
        const bassNotes = [
            196.00, 196.00, 196.00, 196.00, // G3
            196.00, 196.00, 196.00, 196.00,
            155.56, 155.56, 155.56, 155.56, // D#3
            174.61, 174.61, 174.61, 174.61  // F3
        ];

        function scheduleBGM() {
            if (!audioCtx || gameState !== 'PLAYING') return;
            
            // タブが裏に回るなどして時間が進みすぎた場合の補正
            if (nextNoteTime < audioCtx.currentTime) {
                nextNoteTime = audioCtx.currentTime + 0.05;
            }

            // 少し先までスケジュールを登録しておく
            while (nextNoteTime < audioCtx.currentTime + 0.1) {
                playBGMNoteAtTime(nextNoteTime, bgmStep);
                nextNoteTime += noteDuration;
                bgmStep++;
            }
        }

        function playBGMNoteAtTime(time, step) {
            // 1. メロディの再生
            const noteIndex = step % bgmNotes.length;
            const freq = bgmNotes[noteIndex];
            if (freq > 0) {
                const osc = audioCtx.createOscillator();
                const gain = audioCtx.createGain();
                osc.type = 'square'; // レトロな矩形波
                osc.frequency.value = freq;
                
                osc.connect(gain);
                gain.connect(masterGain); // マスターボリュームへ接続
                
                // 音の減衰エンベロープ
                gain.gain.setValueAtTime(0.04, time);
                gain.gain.exponentialRampToValueAtTime(0.001, time + noteDuration - 0.01);
                
                osc.start(time);
                osc.stop(time + noteDuration);
            }
            
            // 2. ベース音の再生 (2ステップ=4分音符ごとに1回)
            if (step % 2 === 0) {
                const bassIndex = Math.floor((step % (bassNotes.length * 2)) / 2);
                const bassFreq = bassNotes[bassIndex];
                if (bassFreq) {
                    const oscBase = audioCtx.createOscillator();
                    const gainBase = audioCtx.createGain();
                    oscBase.type = 'triangle'; // 丸みのある波形
                    oscBase.frequency.value = bassFreq / 2; // さらに1オクターブ下げる
                    
                    oscBase.connect(gainBase);
                    gainBase.connect(masterGain);
                    
                    gainBase.gain.setValueAtTime(0.08, time);
                    gainBase.gain.exponentialRampToValueAtTime(0.001, time + (noteDuration * 2) - 0.01);
                    
                    oscBase.start(time);
                    oscBase.stop(time + (noteDuration * 2));
                }
            }
        }

        function playSound(type) {
            if (!audioCtx) return;

            const oscillator = audioCtx.createOscillator();
            const gainNode = audioCtx.createGain();
            
            oscillator.connect(gainNode);
            gainNode.connect(masterGain); // destination ではなく masterGain に接続
            
            const now = audioCtx.currentTime;
            
            switch(type) {
                case 'shoot':
                    // ピュン！という短い高音
                    oscillator.type = 'square';
                    oscillator.frequency.setValueAtTime(880, now);
                    oscillator.frequency.exponentialRampToValueAtTime(110, now + 0.1);
                    gainNode.gain.setValueAtTime(0.03, now); // 連射するので音量は控えめ
                    gainNode.gain.exponentialRampToValueAtTime(0.01, now + 0.1);
                    oscillator.start(now);
                    oscillator.stop(now + 0.1);
                    break;
                case 'hit':
                    // カチッという短い音
                    oscillator.type = 'sawtooth';
                    oscillator.frequency.setValueAtTime(300, now);
                    oscillator.frequency.exponentialRampToValueAtTime(50, now + 0.05);
                    gainNode.gain.setValueAtTime(0.05, now);
                    gainNode.gain.exponentialRampToValueAtTime(0.01, now + 0.05);
                    oscillator.start(now);
                    oscillator.stop(now + 0.05);
                    break;
                case 'explosion':
                    // ボーンという低めの爆発音
                    oscillator.type = 'square';
                    oscillator.frequency.setValueAtTime(150, now);
                    oscillator.frequency.exponentialRampToValueAtTime(20, now + 0.3);
                    gainNode.gain.setValueAtTime(0.15, now);
                    gainNode.gain.exponentialRampToValueAtTime(0.01, now + 0.3);
                    oscillator.start(now);
                    oscillator.stop(now + 0.3);
                    break;
                case 'gameover':
                    // ドカーンという大きな音
                    oscillator.type = 'sawtooth';
                    oscillator.frequency.setValueAtTime(200, now);
                    oscillator.frequency.exponentialRampToValueAtTime(10, now + 1.0);
                    gainNode.gain.setValueAtTime(0.3, now);
                    gainNode.gain.exponentialRampToValueAtTime(0.01, now + 1.0);
                    oscillator.start(now);
                    oscillator.stop(now + 1.0);
                    break;
            }
        }

        /**
         * ゲームのメインロジック
         */
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        
        // UI要素
        const uiLayer = document.getElementById('ui-layer');
        const startScreen = document.getElementById('start-screen');
        const gameoverScreen = document.getElementById('gameover-screen');
        const hud = document.getElementById('hud');
        const scoreElement = document.getElementById('score');
        const finalScoreElement = document.getElementById('final-score');
        const diffButtons = document.querySelectorAll('.diff-btn');
        const restartBtn = document.getElementById('restart-btn');
        const controlsContainer = document.getElementById('controls-container');
        const quitBtn = document.getElementById('quit-btn');

        // キャンバスサイズをウィンドウに合わせる
        function resizeCanvas() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        }
        window.addEventListener('resize', resizeCanvas);
        resizeCanvas();

        // ゲームの状態管理
        let gameState = 'START'; // 'START', 'PLAYING', 'GAMEOVER'
        let animationId;
        let score = 0;
        let frameCount = 0;

        // 難易度の設定
        const DIFFICULTIES = {
            EASY: { speedMin: 1.5, speedMax: 3.5, spawnBase: 80, spawnMin: 40 },
            NORMAL: { speedMin: 2.5, speedMax: 5.5, spawnBase: 50, spawnMin: 20 },
            HARD: { speedMin: 4.0, speedMax: 8.0, spawnBase: 25, spawnMin: 10 }
        };
        let currentDifficulty = DIFFICULTIES.NORMAL;

        // ゲーム内エンティティの配列
        let player;
        let bullets = [];
        let enemies = [];
        let particles = [];
        let stars = [];

        // マウス・タッチの入力座標
        let targetX = canvas.width / 2;

        /**
         * ユーティリティ関数
         */
        function randomBetween(min, max) {
            return Math.random() * (max - min) + min;
        }

        function calculateDistance(p1, p2) {
            const dx = p1.x - p2.x;
            const dy = p1.y - p2.y;
            return Math.sqrt(dx * dx + dy * dy);
        }

        /**
         * 星クラス（背景のスクロール表現）
         */
        class Star {
            constructor() {
                this.x = Math.random() * canvas.width;
                this.y = Math.random() * canvas.height;
                this.size = Math.random() * 2;
                this.speed = Math.random() * 3 + 0.5;
                this.brightness = Math.random();
            }
            update() {
                this.y += this.speed;
                if (this.y > canvas.height) {
                    this.y = 0;
                    this.x = Math.random() * canvas.width;
                }
            }
            draw() {
                ctx.fillStyle = `rgba(255, 255, 255, ${this.brightness})`;
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2);
                ctx.fill();
            }
        }

        /**
         * プレイヤークラス（自機）
         */
        class Player {
            constructor() {
                this.radius = 15;
                this.x = canvas.width / 2;
                this.y = canvas.height - 60;
                this.color = '#0df';
                this.speed = 8;
                this.lastShotTime = 0;
                this.fireRate = 150; // ミリ秒間隔
            }
            update() {
                // 目標座標（マウス/タッチ位置）に向かって滑らかに移動
                const dx = targetX - this.x;
                this.x += dx * 0.2; // 0.2は追従速度の係数

                // 画面外に出ないように制限
                if (this.x < this.radius) this.x = this.radius;
                if (this.x > canvas.width - this.radius) this.x = canvas.width - this.radius;
                
                // Y座標は画面下部に固定（リサイズ対応）
                this.y = canvas.height - Math.max(60, canvas.height * 0.1);

                // 自動射撃
                const now = Date.now();
                if (now - this.lastShotTime > this.fireRate) {
                    this.shoot();
                    this.lastShotTime = now;
                }
            }
            shoot() {
                bullets.push(new Bullet(this.x, this.y - this.radius));
                // 小さな発射エフェクト
                createParticles(this.x, this.y - this.radius, 3, this.color, 2);
                // 発射音
                playSound('shoot');
            }
            draw() {
                ctx.save();
                ctx.translate(this.x, this.y);
                
                // ネオン効果
                ctx.shadowBlur = 15;
                ctx.shadowColor = this.color;
                
                // 自機の描画（上向きの三角形）
                ctx.beginPath();
                ctx.moveTo(0, -this.radius * 1.5);
                ctx.lineTo(this.radius, this.radius);
                ctx.lineTo(-this.radius, this.radius);
                ctx.closePath();
                ctx.fillStyle = this.color;
                ctx.fill();
                
                // コア部分
                ctx.beginPath();
                ctx.arc(0, 0, this.radius * 0.4, 0, Math.PI * 2);
                ctx.fillStyle = '#fff';
                ctx.fill();

                ctx.restore();
            }
        }

        /**
         * 弾クラス
         */
        class Bullet {
            constructor(x, y) {
                this.x = x;
                this.y = y;
                this.radius = 4;
                this.speed = 12;
                this.color = '#0df';
                this.markedForDeletion = false;
            }
            update() {
                this.y -= this.speed;
                // 画面外に出たら削除フラグを立てる
                if (this.y < -this.radius) {
                    this.markedForDeletion = true;
                }
            }
            draw() {
                ctx.save();
                ctx.shadowBlur = 10;
                ctx.shadowColor = this.color;
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.fill();
                ctx.restore();
            }
        }

        /**
         * 敵クラス
         */
        class Enemy {
            constructor() {
                this.radius = randomBetween(15, 30);
                this.x = randomBetween(this.radius, canvas.width - this.radius);
                this.y = -this.radius;
                
                // スコアが高くなるほど速く、硬くなるように調整
                const speedMultiplier = 1 + (score / 1500); // 難易度ごとにベースが違うので緩やかに
                this.speed = randomBetween(currentDifficulty.speedMin, currentDifficulty.speedMax) * speedMultiplier;
                
                this.hp = Math.floor(this.radius / 10); // 大きい敵ほどHPが高い
                this.maxHp = this.hp;
                
                // 色は赤〜紫系のランダム
                const hue = randomBetween(300, 360);
                this.color = `hsl(${hue}, 100%, 50%)`;
                
                this.markedForDeletion = false;
                this.angle = randomBetween(0, Math.PI * 2);
                this.spinSpeed = randomBetween(-0.05, 0.05);
                this.sides = Math.floor(randomBetween(3, 7)); // 多角形の頂点数
            }
            update() {
                this.y += this.speed;
                this.angle += this.spinSpeed;
                
                if (this.y > canvas.height + this.radius) {
                    this.markedForDeletion = true;
                }
            }
            draw() {
                ctx.save();
                ctx.translate(this.x, this.y);
                ctx.rotate(this.angle);
                
                ctx.shadowBlur = 15;
                ctx.shadowColor = this.color;
                ctx.strokeStyle = this.color;
                ctx.lineWidth = 3;
                
                // 多角形の描画
                ctx.beginPath();
                for (let i = 0; i < this.sides; i++) {
                    const a = (Math.PI * 2 / this.sides) * i;
                    const vx = Math.cos(a) * this.radius;
                    const vy = Math.sin(a) * this.radius;
                    if (i === 0) ctx.moveTo(vx, vy);
                    else ctx.lineTo(vx, vy);
                }
                ctx.closePath();
                ctx.stroke();

                // ダメージを受けている場合は中を少し塗りつぶす
                if (this.hp < this.maxHp) {
                    ctx.fillStyle = `rgba(255, 255, 255, 0.3)`;
                    ctx.fill();
                }

                ctx.restore();
            }
        }

        /**
         * パーティクルクラス（爆発エフェクト）
         */
        class Particle {
            constructor(x, y, color, speedMultiplier = 1) {
                this.x = x;
                this.y = y;
                this.radius = randomBetween(1, 3);
                this.color = color;
                
                const angle = randomBetween(0, Math.PI * 2);
                const velocity = randomBetween(1, 5) * speedMultiplier;
                this.vx = Math.cos(angle) * velocity;
                this.vy = Math.sin(angle) * velocity;
                
                this.life = 1.0;
                this.decay = randomBetween(0.02, 0.05);
                this.markedForDeletion = false;
            }
            update() {
                this.x += this.vx;
                this.y += this.vy;
                this.life -= this.decay;
                
                if (this.life <= 0) {
                    this.markedForDeletion = true;
                }
            }
            draw() {
                ctx.save();
                ctx.globalAlpha = Math.max(0, this.life);
                ctx.shadowBlur = 5;
                ctx.shadowColor = this.color;
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.fill();
                ctx.restore();
            }
        }

        function createParticles(x, y, count, color, speedMultiplier = 1) {
            for (let i = 0; i < count; i++) {
                particles.push(new Particle(x, y, color, speedMultiplier));
            }
        }

        /**
         * ゲームの初期化
         */
        function initGame() {
            player = new Player();
            bullets = [];
            enemies = [];
            particles = [];
            score = 0;
            frameCount = 0;
            scoreElement.innerText = score;
            targetX = canvas.width / 2;
            
            // 背景の星を初期生成
            if (stars.length === 0) {
                for (let i = 0; i < 100; i++) {
                    stars.push(new Star());
                }
            }
        }

        /**
         * ゲームループ
         */
        function animate() {
            if (gameState !== 'PLAYING') return;

            // BGMのスケジューリングを実行
            scheduleBGM();

            // 画面クリア（残像効果を残すために半透明の黒で塗りつぶす）
            ctx.fillStyle = 'rgba(5, 5, 16, 0.3)';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            frameCount++;

            // 敵のスポーン（スコアが上がるとスポーン間隔が短くなる）
            const spawnRate = Math.max(currentDifficulty.spawnMin, currentDifficulty.spawnBase - Math.floor(score / 200));
            if (frameCount % spawnRate === 0) {
                enemies.push(new Enemy());
            }

            // 星の更新と描画
            stars.forEach(star => {
                star.update();
                star.draw();
            });

            // プレイヤーの更新と描画
            player.update();
            player.draw();

            // 弾の更新と描画
            bullets.forEach((bullet, index) => {
                bullet.update();
                bullet.draw();
            });

            // パーティクルの更新と描画
            particles.forEach(particle => {
                particle.update();
                particle.draw();
            });

            // 敵の更新、描画、衝突判定
            enemies.forEach(enemy => {
                enemy.update();
                enemy.draw();

                // 自機と敵の衝突判定 (ゲームオーバー)
                const distToPlayer = calculateDistance(player, enemy);
                if (distToPlayer < player.radius + enemy.radius * 0.8) { // 少し判定を甘くする
                    gameOver();
                }

                // 弾と敵の衝突判定
                bullets.forEach(bullet => {
                    if (bullet.markedForDeletion) return;

                    const distToBullet = calculateDistance(bullet, enemy);
                    if (distToBullet < bullet.radius + enemy.radius) {
                        // 衝突発生
                        bullet.markedForDeletion = true;
                        enemy.hp--;

                        createParticles(bullet.x, bullet.y, 5, '#fff', 0.5); // ヒットエフェクト
                        playSound('hit'); // ヒット音

                        if (enemy.hp <= 0) {
                            enemy.markedForDeletion = true;
                            // 爆発エフェクト
                            createParticles(enemy.x, enemy.y, enemy.radius, enemy.color);
                            playSound('explosion'); // 爆発音
                            
                            // スコア加算 (大きい敵ほど高得点)
                            const points = Math.floor(enemy.radius);
                            score += points;
                            scoreElement.innerText = score;
                        }
                    }
                });
            });

            // 削除フラグが立ったオブジェクトを配列から除去
            bullets = bullets.filter(bullet => !bullet.markedForDeletion);
            enemies = enemies.filter(enemy => !enemy.markedForDeletion);
            particles = particles.filter(particle => !particle.markedForDeletion);

            animationId = requestAnimationFrame(animate);
        }

        /**
         * 状態遷移処理
         */
        function startGame() {
            initAudio(); // オーディオを初期化
            gameState = 'PLAYING';
            startScreen.classList.remove('active');
            gameoverScreen.classList.remove('active');
            hud.style.display = 'block';
            controlsContainer.style.display = 'flex'; // コンテナごと表示
            initGame();
            
            // BGMの開始タイミングをリセット
            if (audioCtx) {
                nextNoteTime = audioCtx.currentTime + 0.1;
                bgmStep = 0;
            }

            animate();
        }

        function gameOver() {
            gameState = 'GAMEOVER';
            cancelAnimationFrame(animationId);
            
            playSound('gameover'); // ゲームオーバー音
            controlsContainer.style.display = 'none'; // コンテナごと隠す

            // プレイヤーの爆発エフェクトを描画して停止
            createParticles(player.x, player.y, 50, player.color, 2);
            particles.forEach(p => { p.update(); p.draw(); }); // 1フレームだけ更新して描画
            
            setTimeout(() => {
                hud.style.display = 'none';
                finalScoreElement.innerText = score;
                gameoverScreen.classList.add('active');
            }, 500); // 爆発を見せるために少し遅らせる
        }

        /**
         * 入力イベントリスナー
         */
        // PC: マウス移動
        canvas.addEventListener('mousemove', (e) => {
            if (gameState === 'PLAYING') {
                targetX = e.clientX;
            }
        });

        // スマホ: タッチ移動
        canvas.addEventListener('touchmove', (e) => {
            if (gameState === 'PLAYING') {
                e.preventDefault(); // スクロール防止
                targetX = e.touches[0].clientX;
            }
        }, { passive: false });

        // タッチ開始時にも座標を更新
        canvas.addEventListener('touchstart', (e) => {
            if (gameState === 'PLAYING') {
                targetX = e.touches[0].clientX;
            }
        }, { passive: true });

        // ボタンイベント
        diffButtons.forEach(btn => {
            btn.addEventListener('click', (e) => {
                const level = e.target.getAttribute('data-level');
                currentDifficulty = DIFFICULTIES[level];
                startGame();
            });
        });

        restartBtn.addEventListener('click', () => {
            gameoverScreen.classList.remove('active');
            startScreen.classList.add('active');
            gameState = 'START';
            drawBackgroundOnly(); // スタート画面の背景アニメーションを再開
        });

        // 途中でやめるボタン
        quitBtn.addEventListener('click', () => {
            if (gameState === 'PLAYING') {
                gameState = 'START';
                cancelAnimationFrame(animationId);
                hud.style.display = 'none';
                controlsContainer.style.display = 'none';
                startScreen.classList.add('active');
                drawBackgroundOnly(); // 背景アニメーションを再開
            }
        });

        // 初期背景描画 (メニュー画面の後ろ用)
        for (let i = 0; i < 100; i++) {
            stars.push(new Star());
        }
        function drawBackgroundOnly() {
            if (gameState !== 'PLAYING') {
                ctx.fillStyle = '#050510';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                stars.forEach(star => {
                    star.update();
                    star.draw();
                });
                requestAnimationFrame(drawBackgroundOnly);
            }
        }
        drawBackgroundOnly();

    </script>
</body>
</html>
