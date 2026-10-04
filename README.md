<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Don't Look Down - Multiplayer Platformer</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Outfit:wght@400;600;800;900&family=Press+Start+2P&display=swap" rel="stylesheet">

    <style>
        * {
            user-select: none;
            -webkit-user-select: none;
            touch-action: manipulation;
        }
        body {
            font-family: 'Outfit', sans-serif;
            background-color: #0d0f18;
            color: #fff;
            overflow: hidden;
        }
        .font-pixel {
            font-family: 'Press Start 2P', cursive;
        }
        canvas {
            display: block;
            width: 100vw;
            height: 100vh;
            background: #090a10;
        }
        .glass-panel {
            background: rgba(18, 22, 36, 0.85);
            backdrop-filter: blur(12px);
            border: 1px solid rgba(255, 255, 255, 0.12);
        }
        .glow-cyan {
            box-shadow: 0 0 20px rgba(6, 182, 212, 0.4);
        }
        .glow-amber {
            box-shadow: 0 0 20px rgba(245, 158, 11, 0.4);
        }
        .glow-red {
            box-shadow: 0 0 25px rgba(239, 68, 68, 0.6);
        }
        /* Custom scrollbars */
        ::-webkit-scrollbar {
            width: 6px;
        }
        ::-webkit-scrollbar-track {
            background: rgba(0, 0, 0, 0.2);
        }
        ::-webkit-scrollbar-thumb {
            background: rgba(255, 255, 255, 0.2);
            border-radius: 3px;
        }
    </style>
</head>
<body class="relative w-screen h-screen overflow-hidden select-none">

    <!-- Game Canvas -->
    <canvas id="gameCanvas"></canvas>

    <!-- Top HUD Bar -->
    <div id="hud" class="hidden absolute top-4 left-4 right-4 flex items-center justify-between pointer-events-none z-10">
        <!-- Left Stats (Height & Max Height) -->
        <div class="flex items-center gap-3 pointer-events-auto">
            <div class="glass-panel px-4 py-2 rounded-xl flex items-center gap-3 border-l-4 border-cyan-400 glow-cyan">
                <i class="fa-solid me-1 fa-mountain text-cyan-400 text-xl"></i>
                <div>
                    <div class="text-xs text-slate-400 uppercase tracking-wider font-bold">Current Height</div>
                    <div class="text-2xl font-extrabold text-cyan-300 font-pixel" id="hudHeight">0 <span class="text-xs">FT</span></div>
                </div>
            </div>

            <div class="glass-panel px-4 py-2 rounded-xl flex items-center gap-3 border-l-4 border-emerald-400">
                <i class="fa-solid fa-trophy text-emerald-400 text-lg"></i>
                <div>
                    <div class="text-xs text-slate-400 uppercase tracking-wider font-bold">Peak Altitude</div>
                    <div class="text-lg font-bold text-emerald-300" id="hudMaxHeight">0 FT</div>
                </div>
            </div>
        </div>

        <!-- Center Room & Fall Warning Info -->
        <div class="hidden md:flex flex-col items-center pointer-events-auto">
            <div class="glass-panel px-4 py-1.5 rounded-full text-xs font-semibold text-slate-300 flex items-center gap-2">
                <span class="w-2 h-2 rounded-full bg-emerald-400 animate-pulse"></span>
                <span>Room Code: <strong class="text-white" id="hudRoomCode">GLOBAL</strong></span>
                <span class="text-slate-500">|</span>
                <i class="fa-solid fa-users text-slate-400"></i>
                <span id="hudOnlineCount">1 Online</span>
            </div>
            <div class="text-[10px] text-amber-300/80 mt-1 font-bold tracking-wide">
                <i class="fa-solid fa-triangle-exclamation"></i> DON'T FALL > 10 FT OR RESET TO BASE!
            </div>
        </div>

        <!-- Right Stats (Energy Coins & Shop Trigger) -->
        <div class="flex items-center gap-3 pointer-events-auto">
            <div class="glass-panel px-4 py-2 rounded-xl flex items-center gap-3 border-l-4 border-amber-400 glow-amber">
                <i class="fa-solid fa-bolt text-amber-400 text-xl animate-bounce"></i>
                <div>
                    <div class="text-xs text-slate-400 uppercase tracking-wider font-bold">Energy Coins</div>
                    <div class="text-2xl font-extrabold text-amber-300 font-pixel" id="hudCoins">0</div>
                </div>
            </div>

            <button id="openShopBtn" class="glass-panel hover:bg-slate-800/80 active:scale-95 transition px-4 py-3 rounded-xl flex items-center gap-2 font-bold text-amber-400 border border-amber-500/30">
                <i class="fa-solid fa-store text-lg"></i>
                <span class="hidden sm:inline">Upgrades Shop</span>
            </button>
        </div>
    </div>

    <!-- Live Leaderboard Sidebar (Toggleable) -->
    <div id="leaderboardBox" class="hidden absolute top-20 right-4 w-64 glass-panel rounded-2xl p-4 z-10 pointer-events-auto border border-slate-700/50 shadow-2xl">
        <div class="flex items-center justify-between pb-2 mb-3 border-b border-slate-700/60">
            <div class="flex items-center gap-2 font-extrabold text-sm text-slate-200">
                <i class="fa-solid fa-crown text-amber-400"></i>
                <span>SUMMIT CLIMBERS</span>
            </div>
            <span class="text-[10px] bg-cyan-900/60 text-cyan-300 px-2 py-0.5 rounded-full uppercase font-bold" id="leaderboardCount">0 ACTIVE</span>
        </div>
        <div id="leaderboardList" class="space-y-2 max-h-56 overflow-y-auto pr-1">
            <!-- Dynamic entries populated via JS -->
        </div>
    </div>

    <!-- Fall Penalty Alert Pop-up Banner -->
    <div id="fallAlert" class="hidden fixed inset-0 pointer-events-none flex items-center justify-center z-30 transition-all">
        <div class="glass-panel bg-red-950/80 border-2 border-red-500/80 rounded-3xl p-6 px-10 text-center glow-red transform scale-110 animate-bounce">
            <i class="fa-solid fa-skull-crossbones text-red-400 text-5xl mb-2"></i>
            <h2 class="text-2xl font-black text-white uppercase tracking-wider font-pixel">FELL > 10 FEET!</h2>
            <p class="text-red-200 font-bold mt-1 text-sm">Teleported back to Safe Base (0 FT)</p>
        </div>
    </div>

    <!-- Vertical Altitude Mini-Gauge (Right side) -->
    <div id="altitudeGauge" class="hidden absolute left-4 top-28 bottom-24 w-4 glass-panel rounded-full z-10 overflow-hidden flex flex-col justify-end p-0.5">
        <div id="altitudeBar" class="w-full bg-gradient-to-t from-cyan-500 via-emerald-400 to-amber-400 rounded-full transition-all duration-200" style="height: 0%;"></div>
    </div>

    <!-- Mobile On-Screen Touch Controls -->
    <div id="touchControls" class="hidden md:hidden fixed bottom-6 left-4 right-4 flex items-center justify-between z-20 pointer-events-none">
        <!-- D-Pad Left/Right -->
        <div class="flex gap-3 pointer-events-auto">
            <button id="btnLeft" class="w-16 h-16 rounded-2xl glass-panel active:bg-cyan-500/40 text-white text-2xl flex items-center justify-center border border-white/20 active:scale-90">
                <i class="fa-solid fa-arrow-left"></i>
            </button>
            <button id="btnRight" class="w-16 h-16 rounded-2xl glass-panel active:bg-cyan-500/40 text-white text-2xl flex items-center justify-center border border-white/20 active:scale-90">
                <i class="fa-solid fa-arrow-right"></i>
            </button>
        </div>
        <!-- Jump & Double Jump Button -->
        <div class="pointer-events-auto">
            <button id="btnJump" class="w-20 h-20 rounded-3xl glass-panel bg-cyan-600/40 active:bg-cyan-400/60 text-white text-2xl flex flex-col items-center justify-center border-2 border-cyan-400/60 active:scale-90 glow-cyan">
                <i class="fa-solid fa-angles-up"></i>
                <span class="text-[10px] font-black uppercase tracking-tighter mt-0.5">JUMP</span>
            </button>
        </div>
    </div>

    <!-- Shop Modal Overlay -->
    <div id="shopModal" class="hidden fixed inset-0 glass-panel bg-black/80 flex items-center justify-center z-40 p-4">
        <div class="bg-slate-900 border border-slate-700/80 rounded-3xl max-w-xl w-full p-6 shadow-2xl relative overflow-hidden">
            <!-- Header -->
            <div class="flex items-center justify-between border-b border-slate-800 pb-4 mb-4">
                <div class="flex items-center gap-3">
                    <div class="w-12 h-12 rounded-2xl bg-amber-500/20 text-amber-400 border border-amber-500/30 flex items-center justify-center text-2xl">
                        <i class="fa-solid fa-store"></i>
                    </div>
                    <div>
                        <h2 class="text-xl font-extrabold text-white">Climber Upgrades Shop</h2>
                        <p class="text-xs text-slate-400">Boost your jump, speed, and energy earnings!</p>
                    </div>
                </div>
                <button id="closeShopBtn" class="w-10 h-10 rounded-xl bg-slate-800 hover:bg-slate-700 text-slate-300 text-lg flex items-center justify-center">
                    <i class="fa-solid fa-xmark"></i>
                </button>
            </div>

            <!-- Current Balance Display -->
            <div class="bg-slate-950/60 rounded-2xl p-3 px-4 mb-4 flex items-center justify-between border border-slate-800">
                <span class="text-xs font-bold text-slate-400 uppercase">Available Energy Coins</span>
                <span id="shopCoinsDisplay" class="text-xl font-extrabold text-amber-400 font-pixel">0</span>
            </div>

            <!-- Upgrade Items List -->
            <div class="space-y-3 max-h-80 overflow-y-auto pr-1" id="shopItemsContainer">
                <!-- Populated dynamically via JS -->
            </div>
        </div>
    </div>

    <!-- Lobby / Join Room Overlay Modal -->
    <div id="lobbyModal" class="fixed inset-0 bg-slate-950/90 backdrop-blur-md flex items-center justify-center z-50 p-4">
        <div class="glass-panel border border-slate-700 rounded-3xl max-w-md w-full p-6 text-center shadow-2xl relative">
            <div class="w-20 h-20 mx-auto rounded-3xl bg-gradient-to-tr from-cyan-500 via-blue-600 to-indigo-600 flex items-center justify-center text-white text-4xl mb-4 shadow-lg shadow-cyan-500/20">
                <i class="fa-solid fa-mountain text-3xl"></i>
            </div>
            <h1 class="text-2xl font-black tracking-wide text-white uppercase font-pixel mb-1">DON'T LOOK DOWN</h1>
            <p class="text-slate-400 text-xs font-medium mb-6">Multiplayer Mountain Climbing Challenge</p>

            <form id="joinForm" class="space-y-4 text-left">
                <div>
                    <label class="block text-xs font-bold text-slate-300 uppercase mb-1">Player Nickname</label>
                    <input type="text" id="playerNameInput" maxlength="12" placeholder="e.g. SummitKing" required
                        class="w-full bg-slate-900 border border-slate-700 rounded-xl px-4 py-3 text-white font-bold placeholder-slate-500 focus:outline-none focus:border-cyan-400 transition">
                </div>

                <div>
                    <label class="block text-xs font-bold text-slate-300 uppercase mb-1">Room Code (Optional)</label>
                    <input type="text" id="roomCodeInput" maxlength="8" placeholder="GLOBAL" uppercase
                        class="w-full bg-slate-900 border border-slate-700 rounded-xl px-4 py-3 text-white font-bold uppercase tracking-widest placeholder-slate-500 focus:outline-none focus:border-cyan-400 transition">
                </div>

                <div>
                    <label class="block text-xs font-bold text-slate-300 uppercase mb-2">Avatar Color</label>
                    <div class="flex items-center justify-between gap-2" id="colorPicker">
                        <button type="button" data-color="#06b6d4" class="color-btn w-10 h-10 rounded-full bg-cyan-500 border-2 border-white ring-2 ring-cyan-400/50 transition transform scale-110"></button>
                        <button type="button" data-color="#10b981" class="color-btn w-10 h-10 rounded-full bg-emerald-500 border-2 border-transparent transition"></button>
                        <button type="button" data-color="#f59e0b" class="color-btn w-10 h-10 rounded-full bg-amber-500 border-2 border-transparent transition"></button>
                        <button type="button" data-color="#ec4899" class="color-btn w-10 h-10 rounded-full bg-pink-500 border-2 border-transparent transition"></button>
                        <button type="button" data-color="#8b5cf6" class="color-btn w-10 h-10 rounded-full bg-purple-500 border-2 border-transparent transition"></button>
                    </div>
                </div>

                <div class="bg-amber-950/40 border border-amber-500/30 rounded-xl p-3 text-[11px] text-amber-200/90 leading-tight flex items-start gap-2 mt-2">
                    <i class="fa-solid fa-triangle-exclamation text-amber-400 text-sm mt-0.5"></i>
                    <div>
                        <strong class="text-amber-300">CUSTOM FALL RULE:</strong> Falling downward by more than 10 ft before touching a safe platform teleports you straight back to the Starting Base!
                    </div>
                </div>

                <button type="submit" class="w-full py-4 rounded-xl bg-gradient-to-r from-cyan-500 via-blue-600 to-indigo-600 hover:from-cyan-400 hover:to-indigo-500 text-white font-black text-sm uppercase tracking-wider shadow-lg shadow-cyan-500/30 transition transform active:scale-95 mt-2">
                    Enter Summit & Play <i class="fa-solid fa-arrow-right ml-1"></i>
                </button>
            </form>
        </div>
    </div>

    <!-- Firebase Imports -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, doc, setDoc, onSnapshot, deleteDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Firebase Configuration setup
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'dont-look-down-v1';
        const firebaseConfig = typeof __firebase_config !== 'undefined' 
            ? JSON.parse(__firebase_config) 
            : { apiKey: "demo-key", authDomain: "demo.firebaseapp.com", projectId: "demo-project" };

        let app, db, auth;
        let currentUser = null;
        let isFirebaseReady = false;

        try {
            app = initializeApp(firebaseConfig);
            db = getFirestore(app);
            auth = getAuth(app);

            // Authenticate first before Firestore operations
            const authInit = async () => {
                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                    await signInWithCustomToken(auth, __initial_auth_token);
                } else {
                    await signInAnonymously(auth);
                }
                currentUser = auth.currentUser;
                isFirebaseReady = true;
                console.log("Firebase Auth initialized:", currentUser?.uid);
            };
            authInit().catch(err => console.warn("Firebase Auth fallback mode:", err));
        } catch (e) {
            console.warn("Firebase setup notice:", e);
        }

        // Export references to window scope for game engine access
        window.GameFirebase = {
            db,
            appId,
            getUser: () => auth?.currentUser || { uid: 'local_' + Math.random().toString(36).substr(2, 6) },
            isReady: () => isFirebaseReady
        };
    </script>

    <script>
        /* Web Audio API Sound Generator */
        class SoundEngine {
            constructor() {
                this.ctx = null;
            }
            init() {
                if (!this.ctx) {
                    this.ctx = new (window.AudioContext || window.webkitAudioContext)();
                }
                if (this.ctx.state === 'suspended') {
                    this.ctx.resume();
                }
            }
            playJump() {
                if (!this.ctx) return;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sine';
                osc.frequency.setValueAtTime(250, this.ctx.currentTime);
                osc.frequency.exponentialRampToValueAtTime(600, this.ctx.currentTime + 0.15);
                gain.gain.setValueAtTime(0.2, this.ctx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, this.ctx.currentTime + 0.15);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start();
                osc.stop(this.ctx.currentTime + 0.15);
            }
            playDoubleJump() {
                if (!this.ctx) return;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'triangle';
                osc.frequency.setValueAtTime(450, this.ctx.currentTime);
                osc.frequency.exponentialRampToValueAtTime(900, this.ctx.currentTime + 0.18);
                gain.gain.setValueAtTime(0.25, this.ctx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, this.ctx.currentTime + 0.18);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start();
                osc.stop(this.ctx.currentTime + 0.18);
            }
            playCoin() {
                if (!this.ctx) return;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sine';
                osc.frequency.setValueAtTime(880, this.ctx.currentTime);
                osc.frequency.setValueAtTime(1320, this.ctx.currentTime + 0.08);
                gain.gain.setValueAtTime(0.15, this.ctx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, this.ctx.currentTime + 0.2);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start();
                osc.stop(this.ctx.currentTime + 0.2);
            }
            playSpring() {
                if (!this.ctx) return;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(200, this.ctx.currentTime);
                osc.frequency.linearRampToValueAtTime(700, this.ctx.currentTime + 0.25);
                gain.gain.setValueAtTime(0.2, this.ctx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, this.ctx.currentTime + 0.25);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start();
                osc.stop(this.ctx.currentTime + 0.25);
            }
            playFallReset() {
                if (!this.ctx) return;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(500, this.ctx.currentTime);
                osc.frequency.exponentialRampToValueAtTime(80, this.ctx.currentTime + 0.5);
                gain.gain.setValueAtTime(0.3, this.ctx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, this.ctx.currentTime + 0.5);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start();
                osc.stop(this.ctx.currentTime + 0.5);
            }
            playBuy() {
                if (!this.ctx) return;
                const now = this.ctx.currentTime;
                [523.25, 659.25, 783.99, 1046.50].forEach((freq, idx) => {
                    const osc = this.ctx.createOscillator();
                    const gain = this.ctx.createGain();
                    osc.type = 'sine';
                    osc.frequency.value = freq;
                    gain.gain.setValueAtTime(0.12, now + idx * 0.05);
                    gain.gain.exponentialRampToValueAtTime(0.001, now + idx * 0.05 + 0.12);
                    osc.connect(gain);
                    gain.connect(this.ctx.destination);
                    osc.start(now + idx * 0.05);
                    osc.stop(now + idx * 0.05 + 0.12);
                });
            }
        }
        const audio = new SoundEngine();
    </script>

    <script>
        /* Main Game Engine */
        const FEET_SCALE = 30; // 30 pixels = 1 foot altitude
        const MAX_ALTITUDE_FEET = 1000;
        const FALL_RESET_THRESHOLD_FEET = 10; // Rule: Falling >10ft teleports to base!
        const FALL_RESET_PIXELS = FALL_RESET_THRESHOLD_FEET * FEET_SCALE; // 300px drop

        // Canvas Setup
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        function resizeCanvas() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        }
        resizeCanvas();
        window.addEventListener('resize', resizeCanvas);

        // Player State
        const player = {
            id: null,
            name: 'Climber',
            color: '#06b6d4',
            roomCode: 'GLOBAL',
            x: 0,
            y: 0,
            vx: 0,
            vy: 0,
            width: 28,
            height: 38,
            isGrounded: false,
            doubleJumpsLeft: 1,
            maxDoubleJumps: 1,
            moveSpeed: 6,
            jumpPower: 13.5,
            coins: 0,
            coinMultiplier: 1,
            height: 0,
            maxHeight: 0,
            fallPeakY: 0, // Recorded top Y coordinate when starting a jump/fall
            isFalling: false,
            trailType: 'sparkles' // sparkles, fire, rainbow, neon
        };

        // Upgrades Store Definition
        const shopUpgrades = {
            speed: { name: 'Speed Boost', level: 0, maxLevel: 5, baseCost: 15, scale: 2, desc: '+15% Move Speed' },
            jump: { name: 'Jump Force', level: 0, maxLevel: 5, baseCost: 20, scale: 2.2, desc: '+12% Jump Height' },
            doubleJump: { name: 'Triple Jump', level: 0, maxLevel: 1, baseCost: 150, scale: 1, desc: 'Unlocks a 3rd Mid-Air Jump!' },
            multiplier: { name: 'Coin Multiplier', level: 0, maxLevel: 4, baseCost: 35, scale: 2.5, desc: 'Earn 2x/3x/4x Coins' },
            trailFire: { name: 'Flame Trail', level: 0, maxLevel: 1, baseCost: 50, scale: 1, desc: 'Fire particle trail cosmetic' },
            trailRainbow: { name: 'Rainbow Trail', level: 0, maxLevel: 1, baseCost: 100, scale: 1, desc: 'Rainbow glow particle trail' }
        };

        // Inputs & Mobile Keys
        const keys = { left: false, right: false, up: false, jumpPressed: false };

        // World Data
        let platforms = [];
        let springPads = [];
        let energyCoins = [];
        let particles = [];
        let remotePlayers = {};

        // Camera Tracking
        const camera = { x: 0, y: 0 };

        /* 1. Procedural World Generator (Summit Mountain Map) */
        function generateWorld() {
            platforms = [];
            springPads = [];
            energyCoins = [];

            const worldWidth = 1200;
            const startX = -worldWidth / 2;

            // Safe Starting Base Ground (Height 0 ft)
            platforms.push({
                x: startX,
                y: 0,
                w: worldWidth,
                h: 60,
                type: 'base',
                color: '#1e293b'
            });

            // Altitude checkpoint tiers up to 1000 FT
            let currentY = -70;
            let currentHeightFt = 0;

            while (currentHeightFt < MAX_ALTITUDE_FEET) {
                currentHeightFt = Math.floor(Math.abs(currentY) / FEET_SCALE);

                // Determine platform layout density based on altitude
                const numPlatformsInTier = Math.floor(2 + Math.random() * 3);
                const tierSpacing = 80 + Math.random() * 40; // Reachable by single/double jump

                for (let i = 0; i < numPlatformsInTier; i++) {
                    const pWidth = 100 + Math.random() * 120;
                    const pX = startX + 50 + Math.random() * (worldWidth - 100 - pWidth);
                    const pY = currentY - (Math.random() * 30);

                    const isMoving = Math.random() < 0.25;
                    const isSpring = !isMoving && Math.random() < 0.2;

                    const platform = {
                        x: pX,
                        y: pY,
                        w: pWidth,
                        h: 18,
                        type: isMoving ? 'moving' : 'normal',
                        color: isMoving ? '#a855f7' : '#0284c7',
                        vx: isMoving ? (Math.random() > 0.5 ? 2 : -2) : 0,
                        minX: pX - 120,
                        maxX: pX + 120
                    };

                    platforms.push(platform);

                    // Add Bounce Spring Pad
                    if (isSpring) {
                        springPads.push({
                            x: pX + pWidth / 2 - 15,
                            y: pY - 12,
                            w: 30,
                            h: 12,
                            isCompressed: false
                        });
                    }

                    // Add Collectible Coins
                    if (Math.random() < 0.6) {
                        energyCoins.push({
                            x: pX + pWidth / 2,
                            y: pY - 30,
                            radius: 10,
                            collected: false,
                            value: 5
                        });
                    }
                }

                currentY -= tierSpacing;
            }

            // Summit Peak Platform at Max Height
            platforms.push({
                x: startX + worldWidth / 2 - 200,
                y: currentY - 50,
                w: 400,
                h: 30,
                type: 'summit',
                color: '#f59e0b'
            });
        }

        // Spawn player at base safe ground
        function respawnAtBase() {
            player.x = -player.width / 2;
            player.y = -38;
            player.vx = 0;
            player.vy = 0;
            player.isGrounded = true;
            player.isFalling = false;
            player.fallPeakY = player.y;
            player.doubleJumpsLeft = player.maxDoubleJumps;
        }

        /* 2. Particle Effects System */
        function createParticles(x, y, color, count = 10, speedMult = 1) {
            for (let i = 0; i < count; i++) {
                particles.push({
                    x: x,
                    y: y,
                    vx: (Math.random() - 0.5) * 6 * speedMult,
                    vy: (Math.random() - 0.5) * 6 * speedMult,
                    size: 3 + Math.random() * 4,
                    color: color,
                    alpha: 1,
                    life: 1
                });
            }
        }

        function updateParticles() {
            for (let i = particles.length - 1; i >= 0; i--) {
                const p = particles[i];
                p.x += p.vx;
                p.y += p.vy;
                p.alpha -= 0.03;
                p.size *= 0.95;
                if (p.alpha <= 0) {
                    particles.splice(i, 1);
                }
            }
        }

        function drawParticles() {
            particles.forEach(p => {
                ctx.save();
                ctx.globalAlpha = Math.max(0, p.alpha);
                ctx.fillStyle = p.color;
                ctx.beginPath();
                ctx.arc(p.x - camera.x, p.y - camera.y, p.size, 0, Math.PI * 2);
                ctx.fill();
                ctx.restore();
            });
        }

        let unsubscribeSync = null;

        function initMultiplayerSync() {
            const checkFirebase = setInterval(() => {
                if (window.GameFirebase && window.GameFirebase.isReady()) {
                    clearInterval(checkFirebase);
                    const { db, appId, getUser } = window.GameFirebase;
                    player.id = getUser().uid;

                    // Publish location periodically
                    setInterval(() => {
                        if (!player.id || !db) return;
                        try {
                            const playerRef = doc(db, 'artifacts', appId, 'public', 'data', 'players', player.id);
                            setDoc(playerRef, {
                                name: player.name,
                                color: player.color,
                                roomCode: player.roomCode,
                                x: Math.round(player.x),
                                y: Math.round(player.y),
                                height: player.height,
                                maxHeight: player.maxHeight,
                                trail: player.trailType,
                                updated: Date.now()
                            }, { merge: true });
                        } catch (err) {
                            console.warn("Sync send error:", err);
                        }
                    }, 120);

                    // Subscribe to all players in the room
                    const playersCol = collection(db, 'artifacts', appId, 'public', 'data', 'players');
                    
                    unsubscribeSync = onSnapshot(playersCol, (snapshot) => {
                        const now = Date.now();
                        const updatedRemotes = {};

                        snapshot.forEach(docSnap => {
                            const pData = docSnap.data();
                            // Filter by current player's room and exclude self or stale users (>12s)
                            if (docSnap.id !== player.id && 
                                pData.roomCode === player.roomCode && 
                                (now - (pData.updated || 0) < 12000)) {
                                
                                updatedRemotes[docSnap.id] = {
                                    ...pData,
                                    id: docSnap.id
                                };
                            }
                        });

                        remotePlayers = updatedRemotes;
                        updateLeaderboardUI();
                    }, (err) => {
                        console.warn("Sync listen error:", err);
                    });
                }
            }, 300);
        }

        function updatePlayerPhysics() {
            // Horizontal Movement
            if (keys.left) player.vx = -player.moveSpeed;
            else if (keys.right) player.vx = player.moveSpeed;
            else player.vx *= 0.78; // Friction

            player.x += player.vx;

            // Apply World Gravity
            player.vy += 0.58; 
            player.y += player.vy;

            // Calculate current Height in Feet
            player.height = Math.max(0, Math.floor(Math.abs(player.y + player.height) / FEET_SCALE));
            if (player.height > player.maxHeight) {
                player.maxHeight = player.height;
            }

            // --- CRITICAL RULE MECHANIC: FALLING TRACKER & 10 FT PENALTY ---
            if (player.vy > 0) {
                if (!player.isFalling) {
                    player.isFalling = true;
                    player.fallPeakY = player.y; // Record peak Y location when fall starts
                }
                
                // Distance fallen so far in pixels
                const fallDistancePixels = player.y - player.fallPeakY;

                // Check if fallen over 10 ft (300px) mid-air OR before landing
                if (fallDistancePixels > FALL_RESET_PIXELS) {
                    trigger10FtFallPenalty();
                    return;
                }
            } else if (player.vy < -1) {
                // Moving upwards (reset falling tracker)
                player.isFalling = false;
                player.fallPeakY = player.y;
            }

            // Collision with Moving Platforms
            platforms.forEach(p => {
                if (p.type === 'moving') {
                    p.x += p.vx;
                    if (p.x < p.minX || p.x > p.maxX) p.vx *= -1;
                }
            });

            // Platform Collision Detection
            let onGroundThisFrame = false;

            platforms.forEach(p => {
                // Standard top landing collision
                if (
                    player.x + player.width > p.x &&
                    player.x < p.x + p.w &&
                    player.y + player.height >= p.y &&
                    player.y + player.height <= p.y + p.h + player.vy &&
                    player.vy >= 0
                ) {
                    // Check fall distance before landing safely
                    const finalDropPixels = player.y - player.fallPeakY;
                    if (player.isFalling && finalDropPixels > FALL_RESET_PIXELS) {
                        trigger10FtFallPenalty();
                        return;
                    }

                    // Safe landing!
                    player.y = p.y - player.height;
                    player.vy = 0;
                    player.isGrounded = true;
                    player.isFalling = false;
                    player.doubleJumpsLeft = player.maxDoubleJumps;
                    player.fallPeakY = player.y;
                    onGroundThisFrame = true;

                    // If landing on moving platform, carry momentum
                    if (p.type === 'moving') {
                        player.x += p.vx;
                    }
                }
            });

            if (!onGroundThisFrame) {
                player.isGrounded = false;
            }

            // Spring Pad Bounce Collisions
            springPads.forEach(s => {
                if (
                    player.x + player.width > s.x &&
                    player.x < s.x + s.w &&
                    player.y + player.height >= s.y &&
                    player.y + player.height <= s.y + s.h + 10 &&
                    player.vy > 0
                ) {
                    player.vy = -22; // High bounce boost!
                    player.isGrounded = false;
                    player.isFalling = false;
                    player.fallPeakY = player.y;
                    player.doubleJumpsLeft = player.maxDoubleJumps;
                    audio.playSpring();
                    createParticles(s.x + s.w / 2, s.y, '#f59e0b', 15);
                }
            });

            // Energy Coins Pickup
            energyCoins.forEach(coin => {
                if (!coin.collected) {
                    const dx = (player.x + player.width / 2) - coin.x;
                    const dy = (player.y + player.height / 2) - coin.y;
                    const dist = Math.sqrt(dx * dx + dy * dy);

                    if (dist < coin.radius + player.width / 2) {
                        coin.collected = true;
                        const totalGained = coin.value * player.coinMultiplier;
                        player.coins += totalGained;
                        audio.playCoin();
                        createParticles(coin.x, coin.y, '#f59e0b', 12);
                    }
                }
            });

            // Player Cosmetics Particles (Trails)
            if (Math.abs(player.vx) > 1 || Math.abs(player.vy) > 1) {
                const trailColor = player.trailType === 'fire' ? '#ef4444' : 
                                  player.trailType === 'rainbow' ? `hsl(${Date.now() % 360}, 100%, 60%)` : 
                                  player.color;
                createParticles(player.x + player.width / 2, player.y + player.height, trailColor, 1, 0.5);
            }
        }

        /* Trigger 10 FT Fall Reset Penalty */
        function trigger10FtFallPenalty() {
            audio.playFallReset();
            respawnAtBase();

            // Show visual pop-up alert
            const fallAlert = document.getElementById('fallAlert');
            fallAlert.classList.remove('hidden');
            setTimeout(() => {
                fallAlert.classList.add('hidden');
            }, 1800);

            createParticles(player.x + player.width / 2, player.y + player.height / 2, '#ef4444', 30, 2);
        }

        /* Perform Jump / Double Jump */
        function performJump() {
            audio.init();

            if (player.isGrounded) {
                // Primary Ground Jump
                player.vy = -player.jumpPower;
                player.isGrounded = false;
                player.isFalling = false;
                player.fallPeakY = player.y;
                audio.playJump();
                createParticles(player.x + player.width / 2, player.y + player.height, '#ffffff', 8);
            } else if (player.doubleJumpsLeft > 0) {
                // Mid-Air Double Jump
                player.doubleJumpsLeft--;
                player.vy = -player.jumpPower * 0.95;
                player.isFalling = false;
                player.fallPeakY = player.y;
                audio.playDoubleJump();
                createParticles(player.x + player.width / 2, player.y + player.height, player.color, 16, 1.5);
            }
        }

        function updateCamera() {
            // Smooth Camera follow local player
            const targetCamX = player.x + player.width / 2 - canvas.width / 2;
            const targetCamY = player.y + player.height / 2 - canvas.height / 2;

            camera.x += (targetCamX - camera.x) * 0.1;
            camera.y += (targetCamY - camera.y) * 0.1;
        }

        function drawWorld() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Draw Background Altitude Grid / Gradient
            const skyGradient = ctx.createLinearGradient(0, 0, 0, canvas.height);
            skyGradient.addColorStop(0, '#090a10');
            skyGradient.addColorStop(1, '#111827');
            ctx.fillStyle = skyGradient;
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Draw Platforms
            platforms.forEach(p => {
                const screenX = p.x - camera.x;
                const screenY = p.y - camera.y;

                if (screenX + p.w > -100 && screenX < canvas.width + 100 &&
                    screenY + p.h > -100 && screenY < canvas.height + 100) {

                    ctx.save();
                    if (p.type === 'base') {
                        // Safe Base Ground Graphics
                        ctx.fillStyle = '#1e293b';
                        ctx.fillRect(screenX, screenY, p.w, p.h);
                        ctx.fillStyle = '#10b981';
                        ctx.fillRect(screenX, screenY, p.w, 6); // Green Safe Top Strip
                    } else if (p.type === 'summit') {
                        // Gold Summit Peak
                        ctx.fillStyle = '#f59e0b';
                        ctx.shadowColor = '#f59e0b';
                        ctx.shadowBlur = 15;
                        ctx.fillRect(screenX, screenY, p.w, p.h);
                    } else {
                        // Regular / Moving Platforms
                        ctx.fillStyle = p.color;
                        ctx.shadowColor = p.color;
                        ctx.shadowBlur = p.type === 'moving' ? 10 : 4;
                        ctx.beginPath();
                        ctx.roundRect(screenX, screenY, p.w, p.h, 6);
                        ctx.fill();
                    }
                    ctx.restore();
                }
            });

            // Draw Spring Pads
            springPads.forEach(s => {
                const screenX = s.x - camera.x;
                const screenY = s.y - camera.y;

                ctx.save();
                ctx.fillStyle = '#f59e0b';
                ctx.fillRect(screenX, screenY, s.w, s.h);
                ctx.fillStyle = '#fef08a';
                ctx.fillRect(screenX + 4, screenY - 3, s.w - 8, 4);
                ctx.restore();
            });

            // Draw Energy Coins
            energyCoins.forEach(coin => {
                if (!coin.collected) {
                    const screenX = coin.x - camera.x;
                    const screenY = coin.y - camera.y;

                    ctx.save();
                    ctx.fillStyle = '#fbbf24';
                    ctx.shadowColor = '#fbbf24';
                    ctx.shadowBlur = 12;
                    ctx.beginPath();
                    ctx.arc(screenX, screenY + Math.sin(Date.now() / 200) * 3, coin.radius, 0, Math.PI * 2);
                    ctx.fill();
                    ctx.restore();
                }
            });

            // Draw Particles
            drawParticles();

            // Draw Remote Multiplayer Players
            Object.values(remotePlayers).forEach(rPlayer => {
                const screenX = rPlayer.x - camera.x;
                const screenY = rPlayer.y - camera.y;

                ctx.save();
                ctx.globalAlpha = 0.85;

                // Remote Character body
                ctx.fillStyle = rPlayer.color || '#ec4899';
                ctx.beginPath();
                ctx.roundRect(screenX, screenY, 28, 38, 8);
                ctx.fill();

                // Eyes
                ctx.fillStyle = '#fff';
                ctx.fillRect(screenX + 6, screenY + 10, 5, 8);
                ctx.fillRect(screenX + 17, screenY + 10, 5, 8);

                // Nametag & Altitude overlay
                ctx.fillStyle = '#ffffff';
                ctx.font = 'bold 11px Outfit, sans-serif';
                ctx.textAlign = 'center';
                ctx.fillText(rPlayer.name || 'Climber', screenX + 14, screenY - 14);

                ctx.fillStyle = '#38bdf8';
                ctx.font = 'bold 10px Outfit, sans-serif';
                ctx.fillText(`${rPlayer.height || 0} FT`, screenX + 14, screenY - 3);

                ctx.restore();
            });

            // Draw Local Player Character
            const localScreenX = player.x - camera.x;
            const localScreenY = player.y - camera.y;

            ctx.save();
            // Outer Glow Aura
            ctx.shadowColor = player.color;
            ctx.shadowBlur = 15;

            // Character body
            ctx.fillStyle = player.color;
            ctx.beginPath();
            ctx.roundRect(localScreenX, localScreenY, player.width, player.height, 8);
            ctx.fill();

            // Visor / Eyes
            ctx.fillStyle = '#ffffff';
            ctx.fillRect(localScreenX + 6, localScreenY + 10, 6, 8);
            ctx.fillRect(localScreenX + 16, localScreenY + 10, 6, 8);
            ctx.fillStyle = '#0f172a';
            ctx.fillRect(localScreenX + (player.vx < 0 ? 6 : 8), localScreenY + 12, 3, 4);
            ctx.fillRect(localScreenX + (player.vx < 0 ? 16 : 18), localScreenY + 12, 3, 4);

            // Double Jump Indicator Dots below player
            for (let i = 0; i < player.maxDoubleJumps; i++) {
                ctx.fillStyle = i < player.doubleJumpsLeft ? '#38bdf8' : '#64748b';
                ctx.beginPath();
                ctx.arc(localScreenX + 8 + i * 12, localScreenY + player.height + 6, 3, 0, Math.PI * 2);
                ctx.fill();
            }

            // Local Nametag
            ctx.fillStyle = '#ffffff';
            ctx.font = 'extrabold 12px Outfit, sans-serif';
            ctx.textAlign = 'center';
            ctx.fillText(player.name, localScreenX + player.width / 2, localScreenY - 16);

            ctx.fillStyle = '#38bdf8';
            ctx.font = 'extrabold 11px Outfit, sans-serif';
            ctx.fillText(`${player.height} FT`, localScreenX + player.width / 2, localScreenY - 4);

            ctx.restore();
        }

        function updateHUD() {
            document.getElementById('hudHeight').innerHTML = `${player.height} <span class="text-xs">FT</span>`;
            document.getElementById('hudMaxHeight').innerText = `${player.maxHeight} FT`;
            document.getElementById('hudCoins').innerText = player.coins;
            document.getElementById('hudRoomCode').innerText = player.roomCode;

            // Altitude Gauge height percentage
            const gaugePercent = Math.min(100, (player.height / MAX_ALTITUDE_FEET) * 100);
            document.getElementById('altitudeBar').style.height = `${gaugePercent}%`;
        }

        function updateLeaderboardUI() {
            const listContainer = document.getElementById('leaderboardList');
            if (!listContainer) return;

            // Gather all active climbers
            const allClimbers = [
                { id: 'local', name: player.name, height: player.height, maxHeight: player.maxHeight, color: player.color, isSelf: true },
                ...Object.values(remotePlayers).map(r => ({ ...r, isSelf: false }))
            ];

            // Sort by Max Height descending
            allClimbers.sort((a, b) => (b.maxHeight || b.height || 0) - (a.maxHeight || a.height || 0));

            document.getElementById('leaderboardCount').innerText = `${allClimbers.length} ACTIVE`;
            document.getElementById('hudOnlineCount').innerText = `${allClimbers.length} Online`;

            listContainer.innerHTML = '';
            allClimbers.forEach((climber, index) => {
                const entry = document.createElement('div');
                entry.className = `flex items-center justify-between p-2 rounded-xl text-xs font-bold ${climber.isSelf ? 'bg-cyan-500/20 border border-cyan-400/40 text-cyan-200' : 'bg-slate-800/50 text-slate-300'}`;
                entry.innerHTML = `
                    <div class="flex items-center gap-2 overflow-hidden">
                        <span class="w-5 text-slate-400 font-pixel text-[10px]">#${index + 1}</span>
                        <span class="w-3 h-3 rounded-full flex-shrink-0" style="background-color: ${climber.color || '#06b6d4'}"></span>
                        <span class="truncate max-w-[90px]">${climber.name} ${climber.isSelf ? '(You)' : ''}</span>
                    </div>
                    <span class="text-emerald-400 font-pixel text-[10px]">${climber.maxHeight || climber.height || 0} FT</span>
                `;
                listContainer.appendChild(entry);
            });
        }

        /* Upgrades Shop Renderer & Purchase Logic */
        function renderShopUI() {
            document.getElementById('shopCoinsDisplay').innerText = player.coins;
            const container = document.getElementById('shopItemsContainer');
            container.innerHTML = '';

            Object.entries(shopUpgrades).forEach(([key, item]) => {
                const cost = Math.floor(item.baseCost * Math.pow(item.scale, item.level));
                const isMax = item.level >= item.maxLevel;
                const canAfford = player.coins >= cost && !isMax;

                const card = document.createElement('div');
                card.className = 'bg-slate-950/80 border border-slate-800 p-3 rounded-2xl flex items-center justify-between gap-3';
                card.innerHTML = `
                    <div>
                        <div class="flex items-center gap-2">
                            <h4 class="font-bold text-sm text-white">${item.name}</h4>
                            <span class="text-[10px] bg-slate-800 text-slate-300 px-2 py-0.5 rounded-full font-bold">Lvl ${item.level}/${item.maxLevel}</span>
                        </div>
                        <p class="text-xs text-slate-400 mt-0.5">${item.desc}</p>
                    </div>
                    <button data-key="${key}" ${!canAfford ? 'disabled' : ''} 
                        class="px-4 py-2 rounded-xl font-bold text-xs flex items-center gap-1.5 transition ${isMax ? 'bg-slate-800 text-slate-500' : canAfford ? 'bg-amber-500 hover:bg-amber-400 text-slate-950 shadow-md shadow-amber-500/20 active:scale-95' : 'bg-slate-800 text-slate-500 opacity-60'}">
                        <i class="fa-solid fa-bolt"></i>
                        <span>${isMax ? 'MAXED' : cost + ' Coins'}</span>
                    </button>
                `;

                const btn = card.querySelector('button');
                btn.addEventListener('click', () => buyUpgrade(key));
                container.appendChild(card);
            });
        }

        function buyUpgrade(key) {
            const item = shopUpgrades[key];
            const cost = Math.floor(item.baseCost * Math.pow(item.scale, item.level));

            if (player.coins >= cost && item.level < item.maxLevel) {
                player.coins -= cost;
                item.level++;
                audio.playBuy();

                // Apply upgrade benefits
                if (key === 'speed') player.moveSpeed += 1;
                if (key === 'jump') player.jumpPower += 1.2;
                if (key === 'doubleJump') player.maxDoubleJumps = 2;
                if (key === 'multiplier') player.coinMultiplier += 1;
                if (key === 'trailFire') player.trailType = 'fire';
                if (key === 'trailRainbow') player.trailType = 'rainbow';

                renderShopUI();
                updateHUD();
            }
        }

        /* Keyboard & Touch Event Listeners */
        window.addEventListener('keydown', e => {
            if (e.code === 'KeyA' || e.code === 'ArrowLeft') keys.left = true;
            if (e.code === 'KeyD' || e.code === 'ArrowRight') keys.right = true;
            if ((e.code === 'KeyW' || e.code === 'ArrowUp' || e.code === 'Space') && !keys.jumpPressed) {
                keys.jumpPressed = true;
                performJump();
            }
        });

        window.addEventListener('keyup', e => {
            if (e.code === 'KeyA' || e.code === 'ArrowLeft') keys.left = false;
            if (e.code === 'KeyD' || e.code === 'ArrowRight') keys.right = false;
            if (e.code === 'KeyW' || e.code === 'ArrowUp' || e.code === 'Space') keys.jumpPressed = false;
        });

        // Touch Controls Wiring
        const btnLeft = document.getElementById('btnLeft');
        const btnRight = document.getElementById('btnRight');
        const btnJump = document.getElementById('btnJump');

        btnLeft.addEventListener('touchstart', (e) => { e.preventDefault(); keys.left = true; });
        btnLeft.addEventListener('touchend', (e) => { e.preventDefault(); keys.left = false; });
        btnRight.addEventListener('touchstart', (e) => { e.preventDefault(); keys.right = true; });
        btnRight.addEventListener('touchend', (e) => { e.preventDefault(); keys.right = false; });
        btnJump.addEventListener('touchstart', (e) => { e.preventDefault(); performJump(); });

        // Lobby Color Selection
        document.querySelectorAll('.color-btn').forEach(btn => {
            btn.addEventListener('click', (e) => {
                document.querySelectorAll('.color-btn').forEach(b => b.classList.remove('ring-2', 'ring-cyan-400/50', 'scale-110'));
                e.currentTarget.classList.add('ring-2', 'ring-cyan-400/50', 'scale-110');
                player.color = e.currentTarget.dataset.color;
            });
        });

        // Form Submit -> Join Lobby & Start Game
        document.getElementById('joinForm').addEventListener('submit', (e) => {
            e.preventDefault();
            audio.init();

            const nameInput = document.getElementById('playerNameInput').value.trim();
            const roomInput = document.getElementById('roomCodeInput').value.trim().toUpperCase();

            if (nameInput) player.name = nameInput;
            player.roomCode = roomInput || 'GLOBAL';

            document.getElementById('lobbyModal').classList.add('hidden');
            document.getElementById('hud').classList.remove('hidden');
            document.getElementById('leaderboardBox').classList.remove('hidden');
            document.getElementById('altitudeGauge').classList.remove('hidden');
            document.getElementById('touchControls').classList.remove('hidden');

            generateWorld();
            respawnAtBase();
            initMultiplayerSync();
        });

        // Shop Modal Open/Close Handlers
        document.getElementById('openShopBtn').addEventListener('click', () => {
            renderShopUI();
            document.getElementById('shopModal').classList.remove('hidden');
        });
        document.getElementById('closeShopBtn').addEventListener('click', () => {
            document.getElementById('shopModal').classList.add('hidden');
        });

        function gameLoop() {
            updatePlayerPhysics();
            updateParticles();
            updateCamera();
            drawWorld();
            updateHUD();

            requestAnimationFrame(gameLoop);
        }

        // Initialize world map and start game loop on load
        window.onload = () => {
            generateWorld();
            respawnAtBase();
            requestAnimationFrame(gameLoop);
        };
    </script>
</body>
</html>
