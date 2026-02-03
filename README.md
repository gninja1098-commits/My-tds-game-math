<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Neon Defense TDS</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body { margin: 0; background-color: #0f172a; color: #f8fafc; font-family: 'Inter', sans-serif; overflow: hidden; touch-action: none; }
        canvas { display: block; cursor: crosshair; }
        .ui-panel { background: rgba(15, 23, 42, 0.9); backdrop-filter: blur(8px); border: 1px solid rgba(255, 255, 255, 0.1); }
        .tower-card { transition: all 0.2s; cursor: pointer; }
        .tower-card:hover:not(.disabled) { transform: translateY(-4px); border-color: #38bdf8; }
        .tower-card.active { border-color: #38bdf8; background: rgba(56, 189, 248, 0.2); box-shadow: 0 0 15px rgba(56, 189, 248, 0.2); }
        .lobby-bg { background: radial-gradient(circle at center, #1e293b 0%, #0f172a 100%); }
        .selection-box { transition: all 0.2s; border: 2px solid transparent; cursor: pointer; }
        .selection-box.active { border-color: #38bdf8; background: rgba(56, 189, 248, 0.1); }
        .loadout-slot { border: 2px dashed rgba(255,255,255,0.1); transition: all 0.2s; position: relative; }
        .loadout-slot.filled { border-style: solid; border-color: #38bdf8; background: rgba(56, 189, 248, 0.05); }
        
        #toast { transition: transform 0.3s ease, opacity 0.3s; transform: translateY(100px); }
        #toast.show { transform: translateY(0); opacity: 1; }

        #ai-badge {
            position: absolute;
            top: 1rem;
            right: 1rem;
            background: rgba(15, 23, 42, 0.6);
            backdrop-filter: blur(4px);
            border: 1px solid rgba(56, 189, 248, 0.3);
            padding: 4px 12px;
            border-radius: 9999px;
            font-size: 10px;
            font-weight: 800;
            color: #38bdf8;
            letter-spacing: 0.05em;
            pointer-events: none;
            z-index: 100;
            display: flex;
            align-items: center;
            gap: 6px;
        }
        .ai-pulse {
            width: 6px;
            height: 6px;
            background: #38bdf8;
            border-radius: 50%;
            box-shadow: 0 0 8px #38bdf8;
            animation: pulse-blue 2s infinite;
        }
        @keyframes pulse-blue {
            0% { opacity: 1; transform: scale(1); }
            50% { opacity: 0.4; transform: scale(1.2); }
            100% { opacity: 1; transform: scale(1); }
        }
    </style>
</head>
<body class="flex flex-col h-screen overflow-hidden">

    <div id="ai-badge">
        <div class="ai-pulse"></div>
        AI GENERATED
    </div>

    <!-- LOBBY SCREEN -->
    <div id="lobby-screen" class="absolute inset-0 z-50 lobby-bg flex flex-col items-center justify-center p-6 overflow-y-auto">
        <div class="mb-6 text-center">
            <h1 class="text-6xl font-black text-sky-400 tracking-tighter drop-shadow-[0_0_15px_rgba(56,189,248,0.5)]">NEON DEFENSE</h1>
            <div class="flex gap-4 justify-center mt-4">
                <div class="bg-amber-500/20 px-4 py-1 rounded-full border border-amber-500/50 inline-flex items-center gap-2">
                    <span class="text-amber-400 font-black" id="lobby-credits">0</span>
                    <span class="text-[10px] text-amber-500 font-bold uppercase">Credits</span>
                </div>
                <div id="user-display" class="bg-sky-500/10 px-4 py-1 rounded-full border border-sky-500/30 text-[10px] text-sky-400 flex items-center gap-2 font-mono cursor-pointer hover:bg-sky-500/20">
                    ID: Loading...
                </div>
            </div>
        </div>

        <div class="grid grid-cols-1 lg:grid-cols-3 gap-6 w-full max-w-7xl pb-10">
            <!-- Mission Config -->
            <div class="ui-panel p-6 rounded-3xl space-y-4 flex flex-col">
                <h3 class="text-xs font-bold text-slate-500 uppercase tracking-widest">Mission Parameters</h3>
                <div class="space-y-3">
                    <div class="grid grid-cols-3 gap-2">
                        <button class="selection-box active p-2 rounded-xl bg-slate-800/50 text-[10px] font-bold" data-map="cyber">CYBER</button>
                        <button class="selection-box p-2 rounded-xl bg-slate-800/50 text-[10px] font-bold" data-map="desert">DESERT</button>
                        <button class="selection-box p-2 rounded-xl bg-slate-800/50 text-[10px] font-bold" data-map="frozen">FROZEN</button>
                    </div>
                    <div class="grid grid-cols-3 gap-2">
                        <button class="selection-box p-2 rounded-xl bg-slate-800/50 text-[10px] font-bold" data-mode="easy">EASY</button>
                        <button class="selection-box active p-2 rounded-xl bg-slate-800/50 text-[10px] font-bold" data-mode="normal">NORMAL</button>
                        <button class="selection-box p-2 rounded-xl bg-slate-800/50 text-[10px] font-bold" data-mode="hard">HARD</button>
                    </div>
                </div>

                <div id="mode-intel" class="bg-sky-500/5 border border-sky-500/20 rounded-2xl p-4 space-y-2">
                    <h4 id="intel-title" class="text-[10px] font-black text-sky-400 uppercase tracking-widest">NORMAL INTEL</h4>
                    <div class="grid grid-cols-2 gap-y-2 text-[9px] font-mono">
                        <div class="text-slate-500">WAVES: <span id="intel-waves" class="text-slate-200">20</span></div>
                        <div class="text-slate-500">HP MULT: <span id="intel-hp" class="text-slate-200">1.0x</span></div>
                        <div class="text-slate-500">START $: <span id="intel-gold" class="text-slate-200">500</span></div>
                        <div class="text-slate-500">REWARD: <span id="intel-reward" class="text-amber-400">600 CR</span></div>
                    </div>
                </div>

                <button id="play-btn" class="w-full bg-sky-500 hover:bg-sky-400 text-white font-black py-4 rounded-xl text-xl transition-all shadow-lg shadow-sky-500/20">
                    START MISSION
                </button>
                <button id="sync-btn" class="text-[10px] text-slate-500 hover:text-sky-400 font-bold uppercase tracking-widest mt-2 underline">Sync Data from Cloud</button>
            </div>

            <!-- Arsenal -->
            <div class="ui-panel p-6 rounded-3xl">
                <h3 class="text-xs font-bold text-slate-500 uppercase tracking-widest mb-4">Active Loadout (Max 5)</h3>
                <div id="loadout-container" class="grid grid-cols-5 gap-2 mb-6">
                    <div class="loadout-slot h-20 rounded-xl flex flex-col items-center justify-center text-[8px] font-bold text-slate-600" data-slot="0">EMPTY</div>
                    <div class="loadout-slot h-20 rounded-xl flex flex-col items-center justify-center text-[8px] font-bold text-slate-600" data-slot="1">EMPTY</div>
                    <div class="loadout-slot h-20 rounded-xl flex flex-col items-center justify-center text-[8px] font-bold text-slate-600" data-slot="2">EMPTY</div>
                    <div class="loadout-slot h-20 rounded-xl flex flex-col items-center justify-center text-[8px] font-bold text-slate-600" data-slot="3">EMPTY</div>
                    <div class="loadout-slot h-20 rounded-xl flex flex-col items-center justify-center text-[8px] font-bold text-slate-600" data-slot="4">EMPTY</div>
                </div>
                <h3 class="text-xs font-bold text-slate-500 uppercase tracking-widest mb-4">Owned Towers</h3>
                <div id="arsenal-list" class="grid grid-cols-1 gap-2 max-h-[300px] overflow-y-auto pr-2"></div>
            </div>

            <!-- Shop -->
            <div class="ui-panel p-6 rounded-3xl flex flex-col">
                <h3 class="text-xs font-bold text-slate-500 uppercase tracking-widest mb-4">Shop</h3>
                <div id="special-shop" class="mb-4">
                    <!-- Special items like 5x speed -->
                </div>
                <div id="shop-list" class="space-y-3 overflow-y-auto max-h-[400px] pr-2"></div>
            </div>
        </div>
    </div>

    <!-- HUD -->
    <div id="game-ui" class="ui-panel p-4 flex justify-between items-center z-10 hidden">
        <div class="flex space-x-8 items-center">
            <div class="flex flex-col"><span class="text-[10px] uppercase text-slate-500 font-bold">Health</span><span id="health-val" class="text-xl font-black text-rose-500">20</span></div>
            <div class="flex flex-col"><span class="text-[10px] uppercase text-slate-500 font-bold">Credits</span><span id="gold-val" class="text-xl font-black text-amber-400">$0</span></div>
            <div class="flex flex-col"><span class="text-[10px] uppercase text-slate-500 font-bold">Wave</span><span id="wave-val" class="text-xl font-black text-sky-400">0/0</span></div>
            <div class="h-8 w-[1px] bg-white/10 mx-2"></div>
            <button id="speed-toggle" class="bg-slate-800 hover:bg-slate-700 text-sky-400 font-black px-4 py-2 rounded-lg border border-sky-500/30 text-xs">SPEED: 1X</button>
        </div>
        <button id="next-wave-btn" class="bg-sky-500 hover:bg-sky-400 text-white font-bold py-2 px-6 rounded-lg transition-colors">NEXT WAVE</button>
    </div>

    <div id="game-container" class="relative flex-grow overflow-hidden bg-slate-900">
        <canvas id="gameCanvas"></canvas>
        <div id="active-slots-ui" class="absolute bottom-6 left-1/2 -translate-x-1/2 flex gap-3 pointer-events-none hidden"></div>
        
        <!-- Tower Context Menu -->
        <div id="tower-info" class="absolute hidden ui-panel p-4 rounded-xl w-60 shadow-2xl z-20">
            <div id="info-name" class="font-black text-sky-400 border-b border-white/10 pb-2 mb-2">TOWER</div>
            <div id="upgrade-stats" class="text-[10px] text-slate-400 mb-3 space-y-1">
                <!-- Dynamic stat preview -->
            </div>
            <button id="upgrade-btn" class="w-full bg-amber-500 text-slate-950 font-black py-2 rounded-lg mb-2 text-xs flex flex-col items-center">
                <span>UPGRADE</span>
                <span id="upgrade-price-tag" class="text-[9px] opacity-80">$0</span>
            </button>
            <button id="sell-btn" class="w-full bg-slate-800 text-slate-400 py-1 rounded-lg text-[10px]">SELL</button>
        </div>
    </div>

    <div id="status-screen" class="absolute inset-0 z-[60] bg-slate-950/90 hidden flex-col items-center justify-center p-6 text-center backdrop-blur-md">
        <h2 id="status-title" class="text-6xl font-black mb-2 tracking-tighter">MISSION COMPLETE</h2>
        <p id="status-msg" class="text-slate-400 mb-8">Base secured.</p>
        <button id="retry-btn" class="bg-white text-slate-950 font-black py-4 px-12 rounded-xl hover:bg-sky-400 transition-colors">RETURN TO LOBBY</button>
    </div>

    <div id="toast" class="fixed bottom-10 right-10 z-[100] bg-slate-800 border border-sky-500/50 text-white px-6 py-3 rounded-2xl shadow-2xl text-xs font-bold opacity-0 pointer-events-none">
        NOTIFICATION
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, getDoc, collection } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Global Configuration
        const firebaseConfig = JSON.parse(__firebase_config);
        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'neon-tds-global-save';

        const TOWER_DATA = {
            gunner: { cost: 100, range: 140, fireRate: 450, color: '#38bdf8', damage: 22, speed: 10, label: 'GUNNER', desc: 'Cheap, reliable fire. (BUFFED)', unlockPriceMult: 1 },
            sniper: { cost: 300, range: 450, fireRate: 2500, color: '#10b981', damage: 150, speed: 30, label: 'SNIPER', desc: 'Huge range, slow fire.', unlockPriceMult: 1.5 },
            minigunner: { cost: 400, range: 120, fireRate: 120, color: '#fbbf24', damage: 6, speed: 15, label: 'MINIGUNNER', desc: 'No reload downtime.', unlockPriceMult: 2 },
            cryo: { cost: 350, range: 150, fireRate: 1000, color: '#60a5fa', damage: 10, speed: 12, label: 'CRYO TECH', desc: 'Slows enemies.', isSlow: true, slowAmount: 0.7, unlockPriceMult: 2.5 },
            incinerator: { cost: 450, range: 110, fireRate: 100, color: '#f97316', damage: 4, label: 'INCINERATOR', desc: 'AoE burn damage.', isFire: true, unlockPriceMult: 3 },
            tesla: { cost: 500, range: 130, fireRate: 1500, color: '#a78bfa', damage: 80, label: 'TESLA COIL', desc: 'Chain lightning.', isChain: true, unlockPriceMult: 3.5 },
            bomber: { cost: 500, range: 200, fireRate: 1800, color: '#f472b6', damage: 70, speed: 8, splash: 100, label: 'BOMBER', desc: 'Splash damage.', unlockPriceMult: 4 },
            plasma: { cost: 750, range: 180, fireRate: 1500, color: '#fb923c', damage: 160, speed: 5, splash: 60, label: 'PLASMA CASTER', desc: 'Heavy energy damage.', unlockPriceMult: 5 },
            railgun: { cost: 1000, range: 600, fireRate: 4500, color: '#e2e8f0', damage: 450, label: 'RAILGUN', desc: 'Pierce everything.', isPiercing: true, unlockPriceMult: 6 },
            bank: { cost: 450, color: '#facc15', label: 'BANK', desc: 'Generates $150/wave.', isEco: true, unlockPriceMult: 2 },
            commander: { cost: 550, range: 200, color: '#ffffff', label: 'COMMANDER', desc: 'Buffs fire rate of nearby towers.', isSupport: true, buffType: 'fireRate', buffValue: 0.8, unlockPriceMult: 4 },
            medic: { cost: 400, range: 120, color: '#4ade80', label: 'MEDIC', desc: 'Restores 1 base HP per wave (max +5).', isMedic: true, unlockPriceMult: 3 },
            dronemaster: { cost: 650, range: 300, fireRate: 5000, color: '#2dd4bf', label: 'DRONE MASTER', desc: 'Deploys seeker drones.', isSummoner: true, summonType: 'drone', unlockPriceMult: 4.5 }
        };

        const ENEMY_TYPES = [
            { name: 'Scout', color: '#fbbf24', speed: 2.2, hp: 0.8, size: 10 },
            { name: 'Speedster', color: '#34d399', speed: 4.5, hp: 0.5, size: 8 },
            { name: 'Tank', color: '#f43f5e', speed: 0.8, hp: 9.0, size: 22 },
            { name: 'Boss', color: '#ffffff', speed: 0.5, hp: 150.0, size: 45, isBoss: true }
        ];

        const MODE_CONFIG = {
            easy: { waves: 15, hpMult: 0.6, gold: 1000, baseHealth: 50, reward: 200 },
            normal: { waves: 25, hpMult: 1.0, gold: 500, baseHealth: 20, reward: 750 },
            hard: { waves: 40, hpMult: 3.5, gold: 400, baseHealth: 10, reward: 3000 }
        };

        let globalData = { credits: 0, unlocked: ['gunner', 'sniper'], loadout: ['gunner', 'sniper'], hasHyperDrive: false };
        let gameState = {
            inGame: false, health: 0, initialHealth: 0, gold: 0, wave: 0, maxWaves: 0,
            selectedTowerType: null, activeTowerIndex: null,
            towers: [], enemies: [], projectiles: [], summons: [], path: [],
            currentMap: 'cyber', currentMode: 'normal',
            gameSpeed: 1
        };

        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const container = document.getElementById('game-container');

        // --- FIRESTORE ---
        async function saveProgress() {
            if (!auth.currentUser) return;
            try {
                const userDoc = doc(db, 'artifacts', appId, 'users', auth.currentUser.uid, 'profile', 'data');
                await setDoc(userDoc, { ...globalData, lastSaved: Date.now() });
            } catch (e) { console.error("Save failed", e); }
        }

        async function syncData() {
            if (!auth.currentUser) return;
            try {
                const userDoc = doc(db, 'artifacts', appId, 'users', auth.currentUser.uid, 'profile', 'data');
                const snap = await getDoc(userDoc);
                if (snap.exists()) {
                    globalData = { ...globalData, ...snap.data() };
                    updateLobbyUI();
                    showToast("Profile Synced");
                } else { await saveProgress(); }
            } catch (e) { showToast("Sync Failed"); }
        }

        function showToast(msg) {
            const t = document.getElementById('toast');
            t.innerText = msg;
            t.classList.add('show');
            setTimeout(() => t.classList.remove('show'), 3000);
        }

        // --- AUTH ---
        onAuthStateChanged(auth, async (user) => {
            if (user) {
                document.getElementById('user-display').innerText = `ID: ${user.uid.slice(0, 8)}...`;
                await syncData();
            }
        });

        async function initAuth() {
            if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                await signInWithCustomToken(auth, __initial_auth_token);
            } else { await signInAnonymously(auth); }
        }

        // --- UI & LOBBY ---
        function updateIntel() {
            const config = MODE_CONFIG[gameState.currentMode];
            document.getElementById('intel-title').innerText = `${gameState.currentMode.toUpperCase()} INTEL`;
            document.getElementById('intel-waves').innerText = config.waves;
            document.getElementById('intel-hp').innerText = `${config.hpMult}x`;
            document.getElementById('intel-gold').innerText = `${config.gold}`;
            document.getElementById('intel-reward').innerText = `${config.reward} CR`;
            const panel = document.getElementById('mode-intel');
            const colors = { easy: 'emerald', normal: 'sky', hard: 'rose' };
            panel.className = `bg-${colors[gameState.currentMode]}-500/5 border border-${colors[gameState.currentMode]}-500/20 rounded-2xl p-4 space-y-2`;
        }

        function updateLobbyUI() {
            document.getElementById('lobby-credits').innerText = globalData.credits;
            
            // Special Shop Items
            const specialShop = document.getElementById('special-shop');
            specialShop.innerHTML = '';
            if (!globalData.hasHyperDrive) {
                const div = document.createElement('div');
                div.className = "p-4 rounded-xl bg-violet-500/10 border border-violet-500/30 flex justify-between items-center mb-4";
                div.innerHTML = `<div><div class="font-black text-sm text-violet-400 italic">HYPER-DRIVE</div><div class="text-[9px] text-slate-500 uppercase font-bold">Unlocks 5X Combat Speed</div></div>
                    <button class="bg-violet-500 text-white font-black px-4 py-1 rounded-lg text-xs disabled:opacity-30" ${globalData.credits < 2000 ? 'disabled' : ''}>2000 CR</button>`;
                div.querySelector('button').onclick = async () => {
                    if (globalData.credits >= 2000) {
                        globalData.credits -= 2000; globalData.hasHyperDrive = true;
                        updateLobbyUI(); await saveProgress(); showToast("HYPER-DRIVE UNLOCKED");
                    }
                };
                specialShop.appendChild(div);
            }

            // Tower Shop Items
            const shopList = document.getElementById('shop-list');
            shopList.innerHTML = '';
            Object.keys(TOWER_DATA).forEach(id => {
                if (globalData.unlocked.includes(id)) return;
                const data = TOWER_DATA[id];
                const price = Math.floor(data.cost * (data.unlockPriceMult || 4));
                const item = document.createElement('div');
                item.className = "p-4 rounded-xl bg-slate-800/30 border border-white/5 flex justify-between items-center";
                item.innerHTML = `<div><div class="font-bold text-sm" style="color:${data.color}">${data.label}</div><div class="text-[9px] text-slate-500">${data.desc}</div></div>
                    <button class="bg-amber-500 text-slate-900 font-black px-3 py-1 rounded-lg text-[10px] disabled:opacity-30" ${globalData.credits < price ? 'disabled' : ''}>${price} CR</button>`;
                item.querySelector('button').onclick = async () => {
                    if (globalData.credits >= price) {
                        globalData.credits -= price; globalData.unlocked.push(id);
                        updateLobbyUI(); await saveProgress(); showToast(`${data.label} UNLOCKED`);
                    }
                };
                shopList.appendChild(item);
            });

            // Arsenal Loadout
            const arsenalList = document.getElementById('arsenal-list');
            arsenalList.innerHTML = '';
            globalData.unlocked.forEach(id => {
                const data = TOWER_DATA[id];
                const equipped = globalData.loadout.includes(id);
                const item = document.createElement('div');
                item.className = "bg-slate-800/40 p-3 rounded-xl flex justify-between items-center border border-white/5 mb-1";
                item.innerHTML = `<div><div class="text-xs font-bold" style="color:${data.color}">${data.label}</div><div class="text-[9px] text-slate-500">${data.desc}</div></div>
                    <button class="text-[9px] px-3 py-1 rounded-lg font-bold ${equipped ? 'bg-rose-500' : 'bg-sky-500'} text-white">${equipped ? 'REMOVE' : 'EQUIP'}</button>`;
                item.querySelector('button').onclick = async () => {
                    if (equipped) globalData.loadout = globalData.loadout.filter(l => l !== id);
                    else if (globalData.loadout.length < 5) globalData.loadout.push(id);
                    updateLobbyUI(); await saveProgress();
                };
                arsenalList.appendChild(item);
            });

            const slots = document.querySelectorAll('.loadout-slot');
            slots.forEach((s, i) => {
                const type = globalData.loadout[i];
                if(type) { s.classList.add('filled'); s.innerHTML = `<div class="text-[9px] font-bold" style="color:${TOWER_DATA[type].color}">${TOWER_DATA[type].label}</div>`; }
                else { s.classList.remove('filled'); s.innerHTML = 'EMPTY'; }
            });
            updateIntel();
        }

        // --- GAMEPLAY ---
        function startGame() {
            if(globalData.loadout.length === 0) return;
            const config = MODE_CONFIG[gameState.currentMode];
            gameState.inGame = true; gameState.wave = 0; gameState.maxWaves = config.waves;
            gameState.health = config.baseHealth; gameState.initialHealth = config.baseHealth;
            gameState.gold = config.gold; gameState.towers = []; gameState.enemies = []; gameState.projectiles = []; gameState.summons = [];
            gameState.gameSpeed = 1;
            gameState.activeTowerIndex = null;
            gameState.selectedTowerType = null;
            
            document.getElementById('lobby-screen').classList.add('hidden');
            document.getElementById('game-ui').classList.remove('hidden');
            document.getElementById('active-slots-ui').classList.remove('hidden');
            document.getElementById('ai-badge').classList.add('hidden');
            document.getElementById('speed-toggle').innerText = "SPEED: 1X";
            
            renderInGameLoadout(); 
            generatePath(); 
            updateStats();
        }

        function generatePath() {
            const w = canvas.width, h = canvas.height;
            if (gameState.currentMap === 'cyber') gameState.path = [{x:-50, y:h*0.5}, {x:w*0.2, y:h*0.5}, {x:w*0.2, y:h*0.2}, {x:w*0.8, y:h*0.2}, {x:w*0.8, y:h*0.8}, {x:w+50, y:h*0.8}];
            else if (gameState.currentMap === 'desert') gameState.path = [{x:-50, y:h*0.2}, {x:w*0.5, y:h*0.8}, {x:w+50, y:h*0.2}];
            else gameState.path = [{x:w*0.5, y:-50}, {x:w*0.5, y:h*0.5}, {x:w+50, y:h*0.5}];
        }

        function spawnWave() {
            if(gameState.enemies.length > 0) return;
            gameState.wave++; updateStats();
            
            gameState.towers.forEach(t => {
                if(t.isEco) gameState.gold += 150;
                if(t.isMedic && gameState.health < gameState.initialHealth + 5) gameState.health++;
            });
            
            let count = 10 + (gameState.wave * 3);
            let spawned = 0;
            const hpMult = MODE_CONFIG[gameState.currentMode].hpMult;
            const difficultyScale = gameState.wave <= 3 ? (0.6 + (gameState.wave * 0.1)) : 1.0;

            const interval = setInterval(() => {
                if(!gameState.inGame) { clearInterval(interval); return; }
                const isBoss = (gameState.wave === gameState.maxWaves && spawned === count - 1);
                let et = isBoss ? ENEMY_TYPES.find(e => e.isBoss) : ENEMY_TYPES[Math.floor(Math.random() * (ENEMY_TYPES.length - 1))];
                
                const baseHp = (30 + (gameState.wave * 40)) * hpMult * et.hp * difficultyScale;
                const baseSpeed = et.speed * difficultyScale;

                gameState.enemies.push({
                    x: gameState.path[0].x, y: gameState.path[0].y, targetIdx: 1,
                    maxHp: baseHp, hp: baseHp, speed: baseSpeed, baseSpeed: baseSpeed,
                    radius: et.size, gold: et.isBoss ? 1000 : 10 + gameState.wave,
                    color: et.color, slowTimer: 0
                });
                
                spawned++;
                if(spawned >= count) clearInterval(interval);
            }, (600 - (gameState.wave * 5)) / gameState.gameSpeed);
        }

        function renderInGameLoadout() {
            const bar = document.getElementById('active-slots-ui');
            bar.innerHTML = '';
            globalData.loadout.forEach(type => {
                const div = document.createElement('div');
                div.className = `tower-card ui-panel p-3 rounded-xl pointer-events-auto min-w-[100px] text-center ${gameState.selectedTowerType === type ? 'active' : ''}`;
                div.onclick = () => { 
                    gameState.selectedTowerType = (gameState.selectedTowerType === type) ? null : type; 
                    gameState.activeTowerIndex = null; // Clear upgrade info if picking a new tower to build
                    document.getElementById('tower-info').classList.add('hidden');
                    renderInGameLoadout(); 
                };
                div.innerHTML = `<div style="color:${TOWER_DATA[type].color}" class="font-bold text-xs">${TOWER_DATA[type].label}</div>
                    <div class="text-amber-400 text-[9px] font-mono mt-1">$${TOWER_DATA[type].cost}</div>`;
                bar.appendChild(div);
            });
        }

        function showTowerUpgradeInfo(idx) {
            const t = gameState.towers[idx];
            const info = document.getElementById('tower-info');
            const statsDiv = document.getElementById('upgrade-stats');
            const upBtn = document.getElementById('upgrade-btn');
            const priceTag = document.getElementById('upgrade-price-tag');
            
            gameState.activeTowerIndex = idx;
            gameState.selectedTowerType = null; // Unselect any tower type ready for building
            renderInGameLoadout();

            document.getElementById('info-name').innerText = `${t.type.toUpperCase()} (LEVEL ${t.level})`;
            
            if (t.isEco || t.isMedic) {
                statsDiv.innerHTML = `<div class="text-amber-400">MAX LEVEL REACHED</div>`;
                upBtn.disabled = true;
                priceTag.innerText = "N/A";
            } else {
                const cost = Math.floor(TOWER_DATA[t.type].cost * 1.5 * t.level);
                priceTag.innerText = `$${cost}`;
                upBtn.disabled = false;
                statsDiv.innerHTML = `
                    <div class="flex justify-between"><span>Damage:</span> <span class="text-emerald-400">${Math.floor(t.damage)} ➔ ${Math.floor(t.damage * 1.6)}</span></div>
                    <div class="flex justify-between"><span>Range:</span> <span class="text-emerald-400">${Math.floor(t.range)} ➔ ${Math.floor(t.range + 20)}</span></div>
                    <div class="flex justify-between"><span>Fire Rate:</span> <span class="text-emerald-400">${Math.floor(t.fireRate || 0)}ms ➔ ${Math.floor((t.fireRate || 0) * 0.85)}ms</span></div>
                `;
            }
            info.style.left = `${Math.min(t.x, canvas.width - 240)}px`;
            info.style.top = `${Math.min(t.y, canvas.height - 180)}px`;
            info.classList.remove('hidden');
        }

        // --- LOOP & RENDER ---
        function update() {
            // Apply game speed multiplication to loops
            for(let step = 0; step < gameState.gameSpeed; step++) {
                // Enemy Movement
                for(let i=gameState.enemies.length-1; i>=0; i--) {
                    const e = gameState.enemies[i];
                    const speed = e.slowTimer > 0 ? e.baseSpeed * 0.5 : e.baseSpeed;
                    if(e.slowTimer > 0) e.slowTimer -= 16;
                    const target = gameState.path[e.targetIdx];
                    const angle = Math.atan2(target.y-e.y, target.x-e.x);
                    e.x += Math.cos(angle) * speed; e.y += Math.sin(angle) * speed;
                    if(Math.hypot(e.x-target.x, e.y-target.y) < 5) {
                        e.targetIdx++;
                        if(e.targetIdx >= gameState.path.length) { gameState.health--; gameState.enemies.splice(i, 1); updateStats(); }
                    }
                }

                const now = Date.now();
                gameState.towers.forEach(t => {
                    if(t.isEco || t.isSupport || t.isMedic) return;
                    let frMult = 1.0;
                    gameState.towers.filter(buff => buff.buffType === 'fireRate').forEach(buff => { if(Math.hypot(t.x-buff.x, t.y-buff.y) < buff.range) frMult *= buff.buffValue; });
                    
                    // We adjust firing logic for high speeds by checking effective delta time
                    if(now - t.lastShot > (t.fireRate * frMult / gameState.gameSpeed)) {
                        if (t.isSummoner) {
                            if (gameState.summons.length < 12) gameState.summons.push({ x: t.x, y: t.y, speed: 4, damage: 100 + (t.level * 50), radius: 8, color: t.color, type: 'drone' });
                            t.lastShot = now;
                        } else {
                            const inRange = gameState.enemies.filter(e => Math.hypot(e.x-t.x, e.y-t.y) < t.range);
                            if(inRange.length > 0) {
                                const target = inRange.sort((a,b) => b.targetIdx - a.targetIdx)[0];
                                if(t.isChain) {
                                    let curr = target, hits = 0; const seen = [];
                                    while(curr && hits < 4) { curr.hp -= t.damage; seen.push(curr); hits++; curr = gameState.enemies.find(e => !seen.includes(e) && Math.hypot(e.x-curr.x, e.y-curr.y) < 100); }
                                } else if(t.isFire) inRange.forEach(e => e.hp -= t.damage);
                                else if(t.isPiercing) {
                                    const angle = Math.atan2(target.y-t.y, target.x-t.x);
                                    gameState.enemies.forEach(e => { if(distToSegment(e, t, {x: t.x + Math.cos(angle)*t.range, y: t.y + Math.sin(angle)*t.range}) < 25) e.hp -= t.damage; });
                                } else {
                                    gameState.projectiles.push({ x: t.x, y: t.y, target, speed: t.speed, damage: t.damage, color: t.color, splash: t.splash || 0, isSlow: t.isSlow });
                                }
                                t.lastShot = now;
                            }
                        }
                    }
                });

                // Projectile Movement
                for(let i=gameState.projectiles.length-1; i>=0; i--) {
                    const p = gameState.projectiles[i];
                    if(!gameState.enemies.includes(p.target)) { gameState.projectiles.splice(i, 1); continue; }
                    const angle = Math.atan2(p.target.y-p.y, p.target.x-p.x);
                    p.x += Math.cos(angle) * p.speed; p.y += Math.sin(angle) * p.speed;
                    if(Math.hypot(p.x-p.target.x, p.y-p.target.y) < 10) {
                        if(p.splash > 0) gameState.enemies.forEach(e => { if(Math.hypot(e.x-p.x, e.y-p.y) < p.splash) e.hp -= p.damage; });
                        else { p.target.hp -= p.damage; if(p.isSlow) p.target.slowTimer = 1500; }
                        gameState.projectiles.splice(i, 1);
                    }
                }

                // Summon Movement
                for(let i=gameState.summons.length-1; i>=0; i--) {
                    const s = gameState.summons[i];
                    const target = gameState.enemies.sort((a,b) => Math.hypot(a.x-s.x, a.y-s.y) - Math.hypot(b.x-s.x, b.y-s.y))[0];
                    if (target) {
                        const angle = Math.atan2(target.y-s.y, target.x-s.x);
                        s.x += Math.cos(angle) * s.speed; s.y += Math.sin(angle) * s.speed;
                        if(Math.hypot(s.x-target.x, s.y-target.y) < 15) { target.hp -= s.damage; gameState.summons.splice(i, 1); }
                    } else { s.x += (Math.random()-0.5)*2; s.y += (Math.random()-0.5)*2; }
                }

                // Cleanup Dead
                for(let i=gameState.enemies.length-1; i>=0; i--) {
                    if(gameState.enemies[i].hp <= 0) { gameState.gold += gameState.enemies[i].gold; gameState.enemies.splice(i, 1); updateStats(); }
                }
            }
            if(gameState.wave === gameState.maxWaves && gameState.enemies.length === 0 && gameState.wave > 0) endMission(true);
        }

        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            if(!gameState.inGame) return;

            // Draw Path
            ctx.strokeStyle = '#1e293b'; ctx.lineWidth = 45; ctx.lineCap = 'round'; ctx.lineJoin = 'round';
            ctx.beginPath(); ctx.moveTo(gameState.path[0].x, gameState.path[0].y);
            gameState.path.forEach(p => ctx.lineTo(p.x, p.y)); ctx.stroke();

            // Draw Selected Tower Range (Visual Enhancement)
            if (gameState.activeTowerIndex !== null) {
                const t = gameState.towers[gameState.activeTowerIndex];
                ctx.beginPath();
                ctx.arc(t.x, t.y, t.range || 50, 0, Math.PI * 2);
                ctx.fillStyle = t.color + '11';
                ctx.fill();
                ctx.setLineDash([10, 5]);
                ctx.strokeStyle = t.color + '66';
                ctx.lineWidth = 2;
                ctx.stroke();
                ctx.setLineDash([]);
            }

            // Draw Towers
            gameState.towers.forEach(t => {
                if (t.isSupport || t.isMedic) {
                    ctx.beginPath(); ctx.strokeStyle = t.color + '44'; ctx.setLineDash([5, 5]);
                    ctx.arc(t.x, t.y, t.range, 0, Math.PI*2); ctx.stroke(); ctx.setLineDash([]);
                }
                ctx.fillStyle = t.color; ctx.beginPath();
                if(t.isEco || t.isMedic) ctx.arc(t.x, t.y, 14, 0, Math.PI*2);
                else if(t.isSupport) { ctx.moveTo(t.x, t.y-16); ctx.lineTo(t.x+16, t.y); ctx.lineTo(t.x, t.y+16); ctx.lineTo(t.x-16, t.y); }
                else { ctx.moveTo(t.x, t.y-16); ctx.lineTo(t.x+14, t.y+12); ctx.lineTo(t.x-14, t.y+12); }
                ctx.fill();
            });

            // Draw Enemies
            gameState.enemies.forEach(e => {
                ctx.fillStyle = e.color; ctx.beginPath(); ctx.arc(e.x, e.y, e.radius, 0, Math.PI*2); ctx.fill();
                ctx.fillStyle = '#1e293b'; ctx.fillRect(e.x-15, e.y-25, 30, 4);
                ctx.fillStyle = '#f43f5e'; ctx.fillRect(e.x-15, e.y-25, 30*(e.hp/e.maxHp), 4);
            });

            // Projectiles/Summons
            gameState.projectiles.forEach(p => { ctx.fillStyle = p.color; ctx.beginPath(); ctx.arc(p.x, p.y, 4, 0, Math.PI*2); ctx.fill(); });
            gameState.summons.forEach(s => { ctx.fillStyle = s.color; ctx.beginPath(); ctx.moveTo(s.x, s.y-8); ctx.lineTo(s.x+8, s.y+4); ctx.lineTo(s.x-8, s.y+4); ctx.fill(); });
            
            // Build ghost tower
            if(gameState.selectedTowerType) {
                const d = TOWER_DATA[gameState.selectedTowerType];
                ctx.globalAlpha = 0.2; ctx.fillStyle = isBlocked(lastMouseX, lastMouseY) ? '#ef4444' : d.color;
                ctx.beginPath(); ctx.arc(lastMouseX, lastMouseY, d.range || 50, 0, Math.PI*2); ctx.fill(); ctx.globalAlpha = 1.0;
            }
        }

        async function endMission(success) {
            gameState.inGame = false;
            const screen = document.getElementById('status-screen');
            if(success) {
                globalData.credits += MODE_CONFIG[gameState.currentMode].reward;
                await saveProgress();
                document.getElementById('status-title').innerText = "MISSION SUCCESS";
            } else document.getElementById('status-title').innerText = "MISSION FAILED";
            screen.classList.remove('hidden');
        }

        function distToSegment(p, v, w) {
            const l2 = Math.pow(v.x-w.x,2) + Math.pow(v.y-w.y,2);
            if(l2 == 0) return Math.hypot(p.x-v.x, p.y-v.y);
            let t = ((p.x-v.x)*(w.x-v.x) + (p.y-v.y)*(w.y-v.y))/l2;
            t = Math.max(0, Math.min(1, t));
            return Math.hypot(p.x-(v.x+t*(w.x-v.x)), p.y-(v.y+t*(w.y-v.y)));
        }

        function isBlocked(x, y) {
            for(let i=0; i<gameState.path.length-1; i++) if(distToSegment({x,y}, gameState.path[i], gameState.path[i+1]) < 35) return true;
            for(let t of gameState.towers) if(Math.hypot(t.x-x, t.y-y) < 35) return true;
            return false;
        }

        function updateStats() {
            document.getElementById('gold-val').innerText = `$${Math.floor(gameState.gold)}`;
            document.getElementById('health-val').innerText = Math.max(0, Math.floor(gameState.health));
            document.getElementById('wave-val').innerText = `${gameState.wave}/${gameState.maxWaves}`;
            if(gameState.health <= 0) endMission(false);
        }

        let lastMouseX = 0, lastMouseY = 0;
        canvas.addEventListener('mousemove', e => {
            const rect = canvas.getBoundingClientRect();
            lastMouseX = e.clientX - rect.left; lastMouseY = e.clientY - rect.top;
        });

        canvas.addEventListener('mousedown', () => {
            if(!gameState.inGame) return;
            const clickedIdx = gameState.towers.findIndex(t => Math.hypot(t.x-lastMouseX, t.y-lastMouseY) < 25);
            
            if(clickedIdx !== -1) { 
                showTowerUpgradeInfo(clickedIdx); 
                return; 
            }
            
            // If we click off a tower, hide info and clear active range visual
            document.getElementById('tower-info').classList.add('hidden');
            gameState.activeTowerIndex = null;
            
            if(!gameState.selectedTowerType) return;
            
            const data = TOWER_DATA[gameState.selectedTowerType];
            if(gameState.gold >= data.cost && !isBlocked(lastMouseX, lastMouseY)) {
                gameState.gold -= data.cost;
                gameState.towers.push({ ...data, x: lastMouseX, y: lastMouseY, level: 1, lastShot: 0, damage: data.damage || 0, range: data.range || 0, type: gameState.selectedTowerType });
                updateStats();
            }
        });

        document.getElementById('upgrade-btn').onclick = (e) => {
            e.stopPropagation();
            const t = gameState.towers[gameState.activeTowerIndex];
            const cost = Math.floor(TOWER_DATA[t.type].cost * 1.5 * t.level);
            if(gameState.gold >= cost) {
                gameState.gold -= cost;
                t.level++; t.damage *= 1.6; t.range += 20; if(t.fireRate) t.fireRate *= 0.85;
                if (t.isSupport && t.buffValue) t.buffValue *= 0.95;
                updateStats(); 
                showTowerUpgradeInfo(gameState.activeTowerIndex); // Refresh UI stats
            }
        };

        document.getElementById('sell-btn').onclick = (e) => {
            e.stopPropagation();
            gameState.gold += Math.floor(TOWER_DATA[gameState.towers[gameState.activeTowerIndex].type].cost * 0.5);
            gameState.towers.splice(gameState.activeTowerIndex, 1);
            document.getElementById('tower-info').classList.add('hidden');
            gameState.activeTowerIndex = null;
            updateStats();
        };

        document.getElementById('speed-toggle').onclick = () => {
            if (gameState.gameSpeed === 1) gameState.gameSpeed = 2;
            else if (gameState.gameSpeed === 2 && globalData.hasHyperDrive) gameState.gameSpeed = 5;
            else gameState.gameSpeed = 1;
            document.getElementById('speed-toggle').innerText = `SPEED: ${gameState.gameSpeed}X`;
        };

        document.getElementById('next-wave-btn').onclick = spawnWave;
        document.getElementById('retry-btn').onclick = () => {
            document.getElementById('status-screen').classList.add('hidden');
            document.getElementById('lobby-screen').classList.remove('hidden');
            document.getElementById('game-ui').classList.add('hidden');
            document.getElementById('active-slots-ui').classList.add('hidden');
            document.getElementById('ai-badge').classList.remove('hidden');
            updateLobbyUI();
        };

        function gameLoop() { if(gameState.inGame) update(); draw(); requestAnimationFrame(gameLoop); }
        window.onload = async () => {
            canvas.width = container.clientWidth; canvas.height = container.clientHeight;
            updateLobbyUI(); await initAuth(); requestAnimationFrame(gameLoop);
        };
        document.getElementById('play-btn').onclick = startGame;
        document.getElementById('sync-btn').onclick = syncData;
        document.querySelectorAll('[data-map], [data-mode]').forEach(btn => {
            btn.onclick = () => {
                const group = btn.dataset.map ? 'map' : 'mode';
                document.querySelectorAll(`[data-${group}]`).forEach(b => b.classList.remove('active'));
                btn.classList.add('active');
                if(btn.dataset.map) gameState.currentMap = btn.dataset.map;
                else { gameState.currentMode = btn.dataset.mode; updateIntel(); }
            };
        });
    </script>
</body>
</html>
