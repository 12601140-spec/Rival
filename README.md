<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no, maximum-scale=1.0">
  <title>99日の絶望 - 3Dホラーサバイバル</title>
  <!-- Tailwind CSS -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- Three.js (r128) -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
  <!-- Tone.js for Audio -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.8.49/Tone.js"></script>
  
  <style>
    * {
      user-select: none;
      -webkit-user-select: none;
      -webkit-touch-callout: none;
      touch-action: manipulation;
    }
    body, html {
      margin: 0;
      padding: 0;
      width: 100%;
      height: 100%;
      overflow: hidden;
      background-color: #020204;
      font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
      color: #e2e8f0;
    }
    canvas {
      display: block;
      width: 100vw;
      height: 100vh;
    }
    ::-webkit-scrollbar {
      width: 6px;
    }
    ::-webkit-scrollbar-track {
      background: rgba(15, 23, 42, 0.8);
    }
    ::-webkit-scrollbar-thumb {
      background: #dc2626;
      border-radius: 4px;
    }
    @keyframes pulse-red {
      0%, 100% { box-shadow: 0 0 15px rgba(220, 38, 38, 0.4); }
      50% { box-shadow: 0 0 30px rgba(220, 38, 38, 0.8); }
    }
    .blood-glow {
      animation: pulse-red 2s infinite;
    }
    .vignette-day {
      background: radial-gradient(circle, transparent 80%, rgba(0,0,0,0.2) 100%);
    }
    .vignette-night {
      background: radial-gradient(circle, transparent 40%, rgba(5,8,22,0.8) 90%);
    }
    .joystick-base {
      background: rgba(255, 255, 255, 0.12);
      border: 2px solid rgba(255, 255, 255, 0.3);
    }
    .joystick-thumb {
      background: rgba(220, 38, 38, 0.75);
      border: 2px solid rgba(239, 68, 68, 0.9);
      box-shadow: 0 0 10px rgba(220, 38, 38, 0.6);
    }
  </style>
</head>
<body class="bg-black text-slate-100 overflow-hidden w-screen h-screen m-0 p-0">

  <!-- MAIN APP CONTAINER (NO DIV ALLOWED!) -->
  <main class="relative w-full h-full overflow-hidden select-none">
    
    <!-- 3D Game Canvas -->
    <canvas id="game-canvas" class="w-full h-full absolute inset-0 z-0 block"></canvas>

    <!-- Horror / Sky Atmospheric Overlay -->
    <section id="vignette-overlay" class="vignette-day absolute inset-0 z-10 pointer-events-none transition-all duration-1000"></section>

    <!-- HUD Overlay Section -->
    <section id="hud-layer" class="absolute inset-0 z-20 pointer-events-none flex flex-col justify-between p-3 md:p-6">
      
      <!-- Top Bar: Stats, Day Info, Coins -->
      <header class="flex justify-between items-start w-full pointer-events-auto">
        <!-- Day & Time Status -->
        <article class="bg-slate-900/80 backdrop-blur-md border border-amber-600/60 rounded-xl p-3 shadow-lg min-w-[140px] md:min-w-[190px]">
          <p class="text-xs text-amber-400 font-bold tracking-widest uppercase">生存日数</p>
          <p id="hud-day" class="text-2xl md:text-3xl font-black text-amber-500 tracking-wider">DAY 1</p>
          <p id="hud-time-phase" class="text-xs text-yellow-300 font-medium mt-0.5">☀️ 昼 (安全探索)</p>
          <section class="w-full bg-slate-800 h-1.5 rounded-full mt-2 overflow-hidden">
            <span id="hud-time-bar" class="block h-full bg-amber-400 w-full transition-all duration-300"></span>
          </section>
        </article>

        <!-- Player Health, Battery & Coins -->
        <article class="flex flex-col gap-2 items-end">
          <section class="bg-slate-900/80 backdrop-blur-md border border-slate-700/60 rounded-xl p-2.5 px-4 shadow-lg flex items-center gap-3">
            <span class="text-xl">💰</span>
            <p id="hud-coins" class="text-xl md:text-2xl font-bold text-yellow-400">1,000</p>
          </section>

          <section class="bg-slate-900/80 backdrop-blur-md border border-red-900/60 rounded-xl p-2.5 shadow-lg w-48 md:w-64">
            <header class="flex justify-between text-xs font-bold mb-1">
              <span class="text-red-400">体 力 (HP)</span>
              <span id="hud-hp-text" class="text-slate-200">100 / 100</span>
            </header>
            <section class="w-full bg-slate-800 h-3 rounded-full overflow-hidden p-0.5 border border-slate-700">
              <span id="hud-hp-bar" class="block h-full bg-gradient-to-r from-red-700 to-red-500 rounded-full w-full transition-all duration-200"></span>
            </section>
          </section>

          <!-- Flashlight Battery Indicator -->
          <section class="bg-slate-900/80 backdrop-blur-md border border-slate-700/60 rounded-xl p-2 shadow-lg w-36 flex items-center gap-2">
            <span class="text-xs">🔦</span>
            <section class="w-full bg-slate-800 h-2 rounded-full overflow-hidden">
              <span id="hud-battery-bar" class="block h-full bg-cyan-400 w-full"></span>
            </section>
          </section>
        </article>
      </header>

      <!-- Boss Health Bar Container -->
      <article id="boss-hud" class="self-center w-11/12 max-w-xl bg-slate-950/90 border-2 border-red-600 rounded-2xl p-3 shadow-2xl backdrop-blur-lg hidden pointer-events-auto">
        <header class="flex justify-between items-center mb-1">
          <span id="boss-name" class="text-red-500 font-extrabold text-sm md:text-base tracking-wider uppercase">⚔️ 漆黒の魔王</span>
          <span id="boss-hp-text" class="text-xs text-red-300 font-mono">1000 / 1000</span>
        </header>
        <section class="w-full bg-slate-900 h-4 rounded-xl p-0.5 border border-red-900 overflow-hidden">
          <span id="boss-hp-bar" class="block h-full bg-gradient-to-r from-red-800 via-red-600 to-amber-500 rounded-lg w-full transition-all duration-200"></span>
        </section>
      </article>

      <!-- Prompt for Interaction (Chests) -->
      <aside id="interact-prompt" class="self-center bg-amber-500/90 text-slate-950 font-bold px-6 py-2 rounded-full shadow-lg border border-yellow-300 hidden text-sm md:text-base animate-bounce">
        📦 宝箱を開ける [タップ / スペース]
      </aside>

      <!-- Center On-Screen Banner Notifications -->
      <aside id="banner-msg" class="self-center bg-slate-900/90 border border-amber-500 text-amber-200 px-6 py-3 rounded-2xl text-center text-sm md:text-lg font-bold shadow-2xl transition-all opacity-0 pointer-events-none transform translate-y-4">
        昼が訪れた！ 建物内の宝箱を探そう
      </aside>

      <!-- Bottom Controls Area -->
      <footer class="flex justify-between items-end w-full pointer-events-auto relative">
        
        <!-- Left Virtual Joystick Zone (Mobile Touch) -->
        <aside id="joystick-zone" class="w-32 h-32 md:w-40 md:h-40 rounded-full joystick-base relative flex items-center justify-center touch-none">
          <span id="joystick-stick" class="w-12 h-12 md:w-16 md:h-16 rounded-full joystick-thumb absolute transition-transform"></span>
        </aside>

        <!-- Center Inventory, Weapons & Trap Quickslots -->
        <nav class="flex gap-1.5 md:gap-2 bg-slate-900/90 backdrop-blur-md p-2 rounded-2xl border border-slate-800 shadow-2xl overflow-x-auto max-w-[55vw]">
          <!-- Dynamic Weapon Slots -->
          <section id="weapon-slots-container" class="flex gap-1.5"></section>
          
          <span class="w-px bg-slate-700 my-1"></span>

          <!-- Dynamic Trap Slots -->
          <section id="trap-slots-container" class="flex gap-1.5"></section>

          <span class="w-px bg-slate-700 my-1"></span>

          <!-- Heal Potion Slot -->
          <button id="slot-potion" onclick="game.usePotion()" class="w-11 h-11 md:w-13 md:h-13 bg-slate-800 border border-slate-700 rounded-xl flex flex-col items-center justify-center relative active:scale-95 shrink-0">
            <span class="text-lg">🧪</span>
            <span id="potion-count" class="absolute top-0.5 right-1 text-[9px] bg-red-600 px-1 rounded-full font-bold">x2</span>
            <span class="text-[8px] text-slate-400">回復</span>
          </button>
          <!-- Battery Recharge Slot -->
          <button id="slot-battery" onclick="game.useBattery()" class="w-11 h-11 md:w-13 md:h-13 bg-slate-800 border border-slate-700 rounded-xl flex flex-col items-center justify-center relative active:scale-95 shrink-0">
            <span class="text-lg">🔋</span>
            <span id="battery-count" class="absolute top-0.5 right-1 text-[9px] bg-cyan-600 px-1 rounded-full font-bold">x1</span>
            <span class="text-[8px] text-slate-400">電池</span>
          </button>
        </nav>

        <!-- Right Action Buttons Zone -->
        <section class="flex flex-col gap-2 items-end">
          <section class="flex gap-2">
            <!-- Place Trap Button -->
            <button onclick="game.placeCurrentTrap()" class="w-12 h-12 rounded-2xl bg-indigo-700 hover:bg-indigo-600 border border-indigo-400 text-white font-bold flex items-center justify-center shadow-lg active:scale-90 text-xl">
              🪤
            </button>
            <!-- Shop Toggle Button -->
            <button onclick="game.toggleShop()" class="w-12 h-12 rounded-2xl bg-amber-600 hover:bg-amber-500 border border-amber-300 text-white font-bold flex items-center justify-center shadow-lg active:scale-90 text-xl">
              🛒
            </button>
            <!-- Flashlight Toggle -->
            <button onclick="game.toggleFlashlight()" class="w-12 h-12 rounded-2xl bg-slate-800 border border-slate-600 text-white font-bold flex items-center justify-center shadow-lg active:scale-90 text-xl">
              🔦
            </button>
          </section>

          <!-- Attack Button (Large Mobile Touch Target) -->
          <button id="btn-attack" class="w-20 h-20 md:w-24 md:h-24 rounded-full bg-gradient-to-br from-red-600 to-amber-700 border-4 border-amber-400 text-white font-black text-xl shadow-2xl flex items-center justify-center active:scale-90 blood-glow">
            攻撃
          </button>
        </section>

      </footer>

    </section>

    <!-- SHOP MODAL (NO DIV TAGS!) -->
    <aside id="shop-modal" class="absolute inset-0 z-30 bg-slate-950/95 backdrop-blur-xl flex flex-col p-4 md:p-8 hidden">
      <header class="flex justify-between items-center border-b border-amber-900/60 pb-4 mb-4">
        <section>
          <h2 class="text-2xl md:text-3xl font-extrabold text-amber-500 tracking-wider">闇の生存ショップ</h2>
          <p class="text-xs md:text-sm text-slate-400">コインで武器・防具・強化アイテム・設置トラップを購入できます</p>
        </section>
        <section class="flex items-center gap-4">
          <section class="bg-slate-900 border border-yellow-500/50 px-4 py-2 rounded-xl flex items-center gap-2">
            <span>💰 所持:</span>
            <span id="shop-coins" class="text-xl font-bold text-yellow-400">1000</span>
          </section>
          <button onclick="game.toggleShop()" class="bg-red-800 hover:bg-red-700 text-white font-bold px-4 py-2 rounded-xl active:scale-95">
            ✕ 閉じる
          </button>
        </section>
      </header>

      <!-- Shop Navigation Tabs -->
      <nav class="flex gap-2 mb-4 border-b border-slate-800 pb-2">
        <button onclick="game.switchShopTab('weapons')" id="tab-btn-weapons" class="px-4 py-2 rounded-xl text-sm font-bold bg-amber-700 text-white">武器・銃</button>
        <button onclick="game.switchShopTab('armors')" id="tab-btn-armors" class="px-4 py-2 rounded-xl text-sm font-bold bg-slate-800 text-slate-300">防具</button>
        <button onclick="game.switchShopTab('traps')" id="tab-btn-traps" class="px-4 py-2 rounded-xl text-sm font-bold bg-slate-800 text-slate-300">設置トラップ</button>
        <button onclick="game.switchShopTab('upgrades')" id="tab-btn-upgrades" class="px-4 py-2 rounded-xl text-sm font-bold bg-slate-800 text-slate-300">能力強化</button>
      </nav>

      <!-- Shop Items List Grid -->
      <main id="shop-items-grid" class="flex-1 overflow-y-auto grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4 pr-2">
        <!-- Dynamically Populated via JS -->
      </main>
    </aside>

    <!-- GAME OVER SCREEN (NO DIV TAGS) -->
    <aside id="gameover-modal" class="absolute inset-0 z-40 bg-black/95 flex flex-col items-center justify-center p-6 text-center hidden">
      <h1 class="text-5xl md:text-7xl font-black text-red-600 tracking-widest blood-glow mb-4">YOU DIED</h1>
      <p class="text-xl md:text-2xl text-slate-300 mb-2">暗闇の恐怖に呑み込まれた...</p>
      <p id="gameover-stats" class="text-slate-400 mb-8 font-mono">生存日数: DAY 1 | 撃破数: 0 | 獲得賞金: 0</p>
      <button onclick="location.reload()" class="bg-gradient-to-r from-red-700 to-amber-700 border-2 border-amber-500 text-white font-black text-xl px-10 py-4 rounded-2xl shadow-2xl hover:scale-105 active:scale-95 transition-all">
        🔄 絶望に再び挑む
      </button>
    </aside>

  </main>

  <script>
    class SoundEngine {
      constructor() {
        this.synth = null;
        this.noiseSynth = null;
        this.isReady = false;
      }
      init() {
        if (this.isReady) return;
        try {
          this.synth = new Tone.PolySynth(Tone.Synth).toDestination();
          this.synth.volume.value = -12;
          this.noiseSynth = new Tone.NoiseSynth({
            noise: { type: 'white' },
            envelope: { attack: 0.01, decay: 0.15, sustain: 0 }
          }).toDestination();
          this.noiseSynth.volume.value = -10;
          Tone.start();
          this.isReady = true;
        } catch(e) {
          console.log("Audio defer");
        }
      }
      playAttack(isGun = false) {
        if (!this.isReady) return;
        if (isGun) {
          this.noiseSynth.triggerAttackRelease("16n");
        } else {
          this.synth.triggerAttackRelease(["C3", "F3"], "16n");
        }
      }
      playHit() {
        if (!this.isReady) return;
        this.noiseSynth.triggerAttackRelease("32n");
      }
      playCoin() {
        if (!this.isReady) return;
        this.synth.triggerAttackRelease("E5", "16n");
      }
      playBuy() {
        if (!this.isReady) return;
        this.synth.triggerAttackRelease(["C4", "G4", "C5"], "8n");
      }
      playNightScream() {
        if (!this.isReady) return;
        this.synth.triggerAttackRelease(["A2", "D3", "F3"], "2n");
      }
      playExplosion() {
        if (!this.isReady) return;
        this.noiseSynth.triggerAttackRelease("8n");
      }
    }

    const soundFx = new SoundEngine();

    const MAP_SIZE = 300; // 300m x 300m Map size
    const HALF_MAP = MAP_SIZE / 2;

    // Database of 15 Enemy Types with unique visuals & AI behaviors
    const ENEMY_TYPES = [
      { id: 0, name: "シカ人間", hp: 60, atk: 12, speed: 2.8, bounty: 35, color: 0x8b5a2b, scale: [1.1, 2.0, 1.1], skill: 'dash', model: 'deer' },
      { id: 1, name: "フクロウ人間", hp: 50, atk: 15, speed: 3.5, bounty: 45, color: 0x4a3b32, scale: [1.0, 1.7, 1.0], skill: 'fly', model: 'owl' },
      { id: 2, name: "エイリアン・グレイ", hp: 70, atk: 18, speed: 4.2, bounty: 55, color: 0x22d3ee, scale: [0.9, 1.9, 0.9], skill: 'pounce', model: 'alien_grey' },
      { id: 3, name: "エイリアン・ウォリアー", hp: 120, atk: 25, speed: 3.2, bounty: 80, color: 0x0284c7, scale: [1.3, 2.3, 1.3], skill: 'tackle', model: 'alien_warrior' },
      { id: 4, name: "エイリアン・アブダクター", hp: 90, atk: 22, speed: 4.0, bounty: 75, color: 0xa855f7, scale: [1.1, 2.1, 1.1], skill: 'fly', model: 'alien_mother' },
      { id: 5, name: "略奪者", hp: 110, atk: 24, speed: 3.0, bounty: 75, color: 0x9a3412, scale: [1.2, 1.9, 1.2], skill: 'tackle', model: 'raider' },
      { id: 6, name: "迷いのゾンビ", hp: 80, atk: 18, speed: 2.0, bounty: 50, color: 0x3f6212, scale: [1.0, 1.8, 1.0], skill: 'flank', model: 'zombie' },
      { id: 7, name: "影の這いずる者", hp: 65, atk: 22, speed: 4.5, bounty: 65, color: 0x0f172a, scale: [1.4, 0.8, 1.5], skill: 'pounce', model: 'crawler' },
      { id: 8, name: "骨の剣士", hp: 100, atk: 28, speed: 3.4, bounty: 85, color: 0xe2e8f0, scale: [1.0, 1.9, 1.0], skill: 'dash', model: 'skeleton' },
      { id: 9, name: "鎌持ちの死神", hp: 140, atk: 35, speed: 3.6, bounty: 110, color: 0x18181b, scale: [1.3, 2.3, 1.3], skill: 'pounce', model: 'reaper' },
      { id: 10, name: "狂気の道化師", hp: 95, atk: 30, speed: 4.6, bounty: 100, color: 0xc084fc, scale: [0.9, 1.7, 0.9], skill: 'flank', model: 'jester' },
      { id: 11, name: "火炎の悪魔", hp: 160, atk: 40, speed: 2.8, bounty: 140, color: 0xd97706, scale: [1.4, 2.4, 1.4], skill: 'tackle', model: 'demon' },
      { id: 12, name: "影の暗殺者", hp: 130, atk: 50, speed: 5.2, bounty: 160, color: 0x020617, scale: [1.0, 1.8, 1.0], skill: 'pounce', model: 'assassin' },
      { id: 13, name: "[ボス] 漆黒の魔王", hp: 1200, atk: 75, speed: 2.8, bounty: 1000, color: 0x991b1b, scale: [3.2, 4.5, 3.2], isBoss: true, model: 'boss' },
      { id: 14, name: "[スーパーボス] 終焉の神", hp: 4500, atk: 140, speed: 3.2, bounty: 5000, color: 0x7e22ce, scale: [4.8, 6.5, 4.8], isBoss: true, model: 'superboss' }
    ];

    // Traps Database (5 Placeable Types)
    const TRAPS_DB = [
      { id: 'bear', name: 'トラバサミ', price: 120, color: 0x475569, icon: '🪤', desc: '触れた敵の動きを止め大ダメージを与える' },
      { id: 'spike', name: 'スパイクフロア', price: 180, color: 0x94a3b8, icon: '🗡️', desc: '範囲内の敵に持続ダメージを与える' },
      { id: 'mine', name: '火炎地雷', price: 250, color: 0xef4444, icon: '💣', desc: '敵が接近すると爆発して広範囲を炎上させる' },
      { id: 'frost', name: '冷凍トラップ', price: 200, color: 0x38bdf8, icon: '❄️', desc: '周囲の敵を凍結させ移動速度を80%低下させる' },
      { id: 'turret', name: '聖なるタレット', price: 500, color: 0xf59e0b, icon: '🗼', desc: '自動で周囲の敵を狙撃する防衛タレット' }
    ];

    // Weapons Database
    const WEAPONS = [
      { id: 0, name: "パチンコ", atk: 30, range: 30.0, price: 0, isGun: true, owned: true, icon: "🎯", desc: "初期装備の弾を飛ばすパチンコ" },
      { id: 1, name: "木の棒", atk: 40, range: 10.0, price: 0, isGun: false, owned: true, icon: "🪵", desc: "広めに振り回せる木製の棒" },
      { id: 2, name: "鉄のトゲバット", atk: 75, range: 12.0, price: 350, isGun: false, owned: false, icon: "🏏", desc: "トゲ付きの凶悪なバット" },
      { id: 3, name: "ハンドガン", atk: 100, range: 40.0, price: 800, isGun: true, owned: false, icon: "🔫", desc: "遠距離から狙撃できるピストル" },
      { id: 4, name: "アサルトライフル", atk: 180, range: 50.0, price: 2200, isGun: true, owned: false, icon: "💥", desc: "高い威力と連射性能を持つ自動小銃" },
      { id: 5, name: "散弾銃ショットガン", atk: 320, range: 25.0, price: 5500, icon: "🔥", isGun: true, owned: false, desc: "至近距離で絶大なダメージを与える散弾銃" },
      { id: 6, name: "聖なるプラズマガン", atk: 650, range: 60.0, price: 15000, icon: "🌟", isGun: true, owned: false, desc: "全ての闇を穿つ究極の光学兵器" }
    ];

    // Armors Database
    const ARMORS = [
      { id: 0, name: "ボロの服", def: 0, price: 0, desc: "防御効果なし" },
      { id: 1, name: "皮のベスト", def: 18, price: 400, desc: "ダメージ18%カット" },
      { id: 2, name: "鉄の重甲冑", def: 38, price: 1800, desc: "ダメージ38%カット" },
      { id: 3, name: "暗黒ドラゴンメイル", def: 60, price: 5500, desc: "ダメージ60%カット" },
      { id: 4, name: "神聖エイギスの盾衣", def: 80, price: 16000, desc: "ダメージ80%カット" }
    ];

    class WorldBuilder {
      constructor(scene) {
        this.scene = scene;
        this.buildings = []; // Stores collision hitboxes and entrances
        this.chests = [];
      }

      generateWorld() {
        // Ground Mesh (300m x 300m) with rough height variation
        const groundGeo = new THREE.PlaneGeometry(MAP_SIZE, MAP_SIZE, 48, 48);
        const pos = groundGeo.attributes.position;
        for (let i = 0; i < pos.count; i++) {
          const x = pos.getX(i);
          const y = pos.getY(i);
          const zNoise = Math.sin(x * 0.04) * Math.cos(y * 0.04) * 0.8;
          pos.setZ(i, zNoise);
        }
        groundGeo.computeVertexNormals();

        const groundMat = new THREE.MeshStandardMaterial({
          color: 0x2e3d2a,
          roughness: 0.9,
          metalness: 0.05
        });
        const ground = new THREE.Mesh(groundGeo, groundMat);
        ground.rotation.x = -Math.PI / 2;
        ground.receiveShadow = true;
        this.scene.add(ground);

        // Map Boundary Walls
        const wallMat = new THREE.MeshBasicMaterial({ color: 0x1e293b, wireframe: true, transparent: true, opacity: 0.2 });
        const fenceGeo = new THREE.BoxGeometry(MAP_SIZE, 15, 2);
        
        const northWall = new THREE.Mesh(fenceGeo, wallMat); northWall.position.set(0, 7.5, -HALF_MAP);
        const southWall = new THREE.Mesh(fenceGeo, wallMat); southWall.position.set(0, 7.5, HALF_MAP);
        const eastWall = new THREE.Mesh(fenceGeo, wallMat); eastWall.position.set(HALF_MAP, 7.5, 0); eastWall.rotation.y = Math.PI/2;
        const westWall = new THREE.Mesh(fenceGeo, wallMat); westWall.position.set(-HALF_MAP, 7.5, 0); westWall.rotation.y = Math.PI/2;
        this.scene.add(northWall, southWall, eastWall, westWall);

        // Plant Environment Elements
        this.populateEnvironment();

        // Construct 30 Enterable Horror Buildings with Interiors and Chests
        this.create30EnterableBuildings();
      }

      populateEnvironment() {
        // Trees
        const trunkGeo = new THREE.CylinderGeometry(0.3, 0.5, 6, 6);
        const foliageGeo = new THREE.ConeGeometry(2.5, 6, 6);
        const trunkMat = new THREE.MeshStandardMaterial({ color: 0x3d2314, roughness: 0.9 });
        const foliageMat = new THREE.MeshStandardMaterial({ color: 0x1e3a1e, roughness: 0.8 });

        for (let i = 0; i < 70; i++) {
          const x = (Math.random() - 0.5) * (MAP_SIZE - 20);
          const z = (Math.random() - 0.5) * (MAP_SIZE - 20);
          if (Math.abs(x) < 12 && Math.abs(z) < 12) continue;

          const treeGroup = new THREE.Group();
          const trunk = new THREE.Mesh(trunkGeo, trunkMat);
          trunk.position.y = 3;
          const foliage = new THREE.Mesh(foliageGeo, foliageMat);
          foliage.position.y = 7;

          treeGroup.add(trunk, foliage);
          treeGroup.position.set(x, 0, z);
          const scale = 0.8 + Math.random() * 0.6;
          treeGroup.scale.set(scale, scale, scale);
          this.scene.add(treeGroup);
        }

        // Rocks
        const rockGeo = new THREE.DodecahedronGeometry(1, 1);
        const rockMat = new THREE.MeshStandardMaterial({ color: 0x52525b, roughness: 0.95 });

        for (let i = 0; i < 50; i++) {
          const x = (Math.random() - 0.5) * (MAP_SIZE - 20);
          const z = (Math.random() - 0.5) * (MAP_SIZE - 20);
          const rock = new THREE.Mesh(rockGeo, rockMat);
          const s = 0.6 + Math.random() * 1.5;
          rock.scale.set(s, s * 0.7, s);
          rock.position.set(x, s * 0.35, z);
          rock.rotation.set(Math.random(), Math.random(), Math.random());
          this.scene.add(rock);
        }
      }

      create30EnterableBuildings() {
        const buildingNames = [
          "廃倉庫", "古びた神社", "崩壊した民家", "不気味な時計塔", "風車跡",
          "墓地と大墓石", "廃病院", "荒廃した学校", "暗黒の礼拝堂", "高層給水塔",
          "ガソリンスタンド跡", "老朽化工場", "監視塔", "ログハウス", "壊れた温室",
          "古びた灯台", "地下バンカー入口", "呪われた断頭台", "石造のアーチ橋", "儀式の巨石群",
          "崩壊した風車", "廃坑道", "錆びた電柱塔", "廃バス集積地", "不気味な納屋",
          "廃館ホテル", "石の砦", "鳥居回廊", "破滅の祭壇", "安全コンテナハウス"
        ];

        for (let i = 0; i < 30; i++) {
          const angle = (i / 30) * Math.PI * 2 + (Math.random() * 0.15);
          const radius = 35 + Math.random() * 100;
          const x = Math.cos(angle) * radius;
          const z = Math.sin(angle) * radius;

          const buildingGroup = new THREE.Group();
          buildingGroup.position.set(x, 0, z);

          // Build enterable structure with walls, doorway & interior floor
          const dimensions = this.buildEnterableStructure(buildingGroup, i);

          this.scene.add(buildingGroup);

          // Register collision boundary
          this.buildings.push({
            name: buildingNames[i],
            x: x,
            z: z,
            width: dimensions.width,
            depth: dimensions.depth,
            doorwayZ: dimensions.depth / 2
          });

          // Spawn Treasure Chest Inside Building
          if (Math.random() < 0.8) {
            this.spawnChestInside(x, z);
          }
        }
      }

      buildEnterableStructure(group, index) {
        const matWall = new THREE.MeshStandardMaterial({ color: 0x334155, roughness: 0.8 });
        const matFloor = new THREE.MeshStandardMaterial({ color: 0x1e293b, roughness: 0.9 });
        const matRoof = new THREE.MeshStandardMaterial({ color: 0x0f172a, roughness: 0.9 });

        const width = 12 + (index % 4) * 2;
        const depth = 12 + (index % 3) * 3;
        const height = 6;
        const wallThick = 0.6;
        const doorWidth = 3.5;

        // Interior Floor
        const floorGeo = new THREE.BoxGeometry(width - 0.2, 0.2, depth - 0.2);
        const floor = new THREE.Mesh(floorGeo, matFloor);
        floor.position.y = 0.1;
        group.add(floor);

        // North Wall
        const backWall = new THREE.Mesh(new THREE.BoxGeometry(width, height, wallThick), matWall);
        backWall.position.set(0, height / 2, -depth / 2);
        group.add(backWall);

        // East Wall
        const rightWall = new THREE.Mesh(new THREE.BoxGeometry(wallThick, height, depth), matWall);
        rightWall.position.set(width / 2, height / 2, 0);
        group.add(rightWall);

        // West Wall
        const leftWall = new THREE.Mesh(new THREE.BoxGeometry(wallThick, height, depth), matWall);
        leftWall.position.set(-width / 2, height / 2, 0);
        group.add(leftWall);

        // South Wall (With Open Doorway Entrance)
        const frontWallLeftWidth = (width - doorWidth) / 2;
        const frontWallLeft = new THREE.Mesh(new THREE.BoxGeometry(frontWallLeftWidth, height, wallThick), matWall);
        frontWallLeft.position.set(-width / 2 + frontWallLeftWidth / 2, height / 2, depth / 2);

        const frontWallRight = new THREE.Mesh(new THREE.BoxGeometry(frontWallLeftWidth, height, wallThick), matWall);
        frontWallRight.position.set(width / 2 - frontWallLeftWidth / 2, height / 2, depth / 2);

        group.add(frontWallLeft, frontWallRight);

        // Roof
        const roof = new THREE.Mesh(new THREE.BoxGeometry(width + 0.8, wallThick, depth + 0.8), matRoof);
        roof.position.set(0, height + wallThick / 2, 0);
        group.add(roof);

        return { width: width, depth: depth };
      }

      spawnChestInside(bx, bz) {
        const chestGroup = new THREE.Group();
        const chestMat = new THREE.MeshStandardMaterial({ color: 0x854d0e, roughness: 0.6 });
        const goldMat = new THREE.MeshStandardMaterial({ color: 0xeab308, metalness: 0.9, roughness: 0.2 });

        const box = new THREE.Mesh(new THREE.BoxGeometry(1.2, 0.8, 0.8), chestMat);
        box.position.y = 0.4;

        const band = new THREE.Mesh(new THREE.BoxGeometry(1.25, 0.15, 0.85), goldMat);
        band.position.y = 0.4;

        chestGroup.add(box, band);
        chestGroup.position.set(bx + (Math.random() - 0.5) * 4, 0, bz + (Math.random() - 0.5) * 4);
        this.scene.add(chestGroup);

        this.chests.push({
          mesh: chestGroup,
          opened: false
        });
      }

      checkBuildingCollision(pos, radius = 0.8) {
        for (let b of this.buildings) {
          const minX = b.x - b.width / 2 - radius;
          const maxX = b.x + b.width / 2 + radius;
          const minZ = b.z - b.depth / 2 - radius;
          const maxZ = b.z + b.depth / 2 + radius;

          // Doorway Opening check (Front wall gap)
          const isNearDoorway = Math.abs(pos.x - b.x) < 1.6 && Math.abs(pos.z - (b.z + b.depth / 2)) < 1.5;
          
          if (isNearDoorway) continue; // Allow entry through door

          if (pos.x > minX && pos.x < maxX && pos.z > minZ && pos.z < maxZ) {
            // Push out from wall closest boundary
            const dxMin = Math.abs(pos.x - minX);
            const dxMax = Math.abs(pos.x - maxX);
            const dzMin = Math.abs(pos.z - minZ);
            const dzMax = Math.abs(pos.z - maxZ);

            const minDist = Math.min(dxMin, dxMax, dzMin, dzMax);

            if (minDist === dxMin) pos.x = minX;
            else if (minDist === dxMax) pos.x = maxX;
            else if (minDist === dzMin) pos.z = minZ;
            else pos.z = maxZ;
          }
        }
      }
    }

    class EnemyManager {
      constructor(scene) {
        this.scene = scene;
        this.enemies = [];
        this.projectiles = []; // Dark orbs / Boss skill shots
      }

      spawnEnemy(typeData, playerPos, dayNum) {
        const group = new THREE.Group();

        const mat = new THREE.MeshStandardMaterial({
          color: typeData.color,
          roughness: 0.6,
          metalness: 0.2
        });

        // Base Body Geometry
        const bodyGeo = new THREE.CylinderGeometry(0.45, 0.35, 1.5, 8);
        const body = new THREE.Mesh(bodyGeo, mat);
        body.position.y = 0.75;
        group.add(body);

        // Head
        let headGeo = new THREE.SphereGeometry(0.35, 8, 8);
        if (typeData.model === 'alien_grey' || typeData.model === 'alien_warrior' || typeData.model === 'alien_mother') {
          headGeo = new THREE.SphereGeometry(0.45, 8, 8);
          headGeo.scale(1, 1.5, 1.2);
        }

        const headMat = typeData.isBoss ? 
          new THREE.MeshStandardMaterial({ color: 0xef4444, emissive: 0x7f1d1d }) : mat;
        const head = new THREE.Mesh(headGeo, headMat);
        head.position.y = 1.7;
        group.add(head);

        // Eyes
        const eyeMat = new THREE.MeshBasicMaterial({ color: typeData.model.includes('alien') ? 0x000000 : 0xef4444 });
        const eyeL = new THREE.Mesh(new THREE.SphereGeometry(0.08, 4, 4), eyeMat);
        eyeL.position.set(0.12, 1.75, 0.32);
        const eyeR = new THREE.Mesh(new THREE.SphereGeometry(0.08, 4, 4), eyeMat);
        eyeR.position.set(-0.12, 1.75, 0.32);
        group.add(eyeL, eyeR);

        // Attachments
        if (typeData.model === 'deer') {
          const antlerMat = new THREE.MeshStandardMaterial({ color: 0x3f2e21 });
          const a1 = new THREE.Mesh(new THREE.CylinderGeometry(0.04, 0.04, 0.8), antlerMat);
          a1.position.set(0.2, 2.2, 0); a1.rotation.z = -0.4;
          const a2 = new THREE.Mesh(new THREE.CylinderGeometry(0.04, 0.04, 0.8), antlerMat);
          a2.position.set(-0.2, 2.2, 0); a2.rotation.z = 0.4;
          group.add(a1, a2);
        } else if (typeData.model === 'owl') {
          const beak = new THREE.Mesh(new THREE.ConeGeometry(0.1, 0.25, 4), new THREE.MeshBasicMaterial({ color: 0xf59e0b }));
          beak.position.set(0, 1.65, 0.38); beak.rotation.x = Math.PI/2;
          group.add(beak);
        }

        // Arms
        const armGeo = new THREE.BoxGeometry(0.18, 0.85, 0.18);
        const armL = new THREE.Mesh(armGeo, mat); armL.position.set(0.55, 1.1, 0);
        const armR = new THREE.Mesh(armGeo, mat); armR.position.set(-0.55, 1.1, 0);
        group.add(armL, armR);

        group.scale.set(typeData.scale[0], typeData.scale[1], typeData.scale[2]);

        // Spawn position around player
        const spawnDist = 30 + Math.random() * 30;
        const angle = Math.random() * Math.PI * 2;
        const spawnX = Math.max(-HALF_MAP + 5, Math.min(HALF_MAP - 5, playerPos.x + Math.cos(angle) * spawnDist));
        const spawnZ = Math.max(-HALF_MAP + 5, Math.min(HALF_MAP - 5, playerPos.z + Math.sin(angle) * spawnDist));
        
        group.position.set(spawnX, typeData.skill === 'fly' ? 4 : 0, spawnZ);
        this.scene.add(group);

        const dayScale = 1 + (dayNum * 0.12);
        const enemyObj = {
          mesh: group,
          armL: armL,
          armR: armR,
          id: typeData.id,
          name: typeData.name,
          maxHp: Math.round(typeData.hp * dayScale),
          hp: Math.round(typeData.hp * dayScale),
          atk: Math.round(typeData.atk * dayScale),
          speed: typeData.speed * (1 + Math.min(0.5, dayNum * 0.02)),
          bounty: Math.round(typeData.bounty * (1 + dayNum * 0.1)),
          isBoss: typeData.isBoss || false,
          skill: typeData.skill || 'dash',
          attackCooldown: 0,
          skillCooldown: 2 + Math.random() * 3,
          isPouncing: false,
          isFreezed: false,
          freezeTimer: 0,
          animTime: Math.random() * 10
        };

        this.enemies.push(enemyObj);
        return enemyObj;
      }

      updateEnemies(delta, playerPos, world, onHitPlayer) {
        for (let i = this.enemies.length - 1; i >= 0; i--) {
          const e = this.enemies[i];

          // Freeze status effect handling
          if (e.isFreezed) {
            e.freezeTimer -= delta;
            if (e.freezeTimer <= 0) e.isFreezed = false;
          }

          const currentSpeed = e.isFreezed ? e.speed * 0.2 : e.speed;

          e.animTime += delta * 6;
          e.skillCooldown -= delta;

          const dx = playerPos.x - e.mesh.position.x;
          const dz = playerPos.z - e.mesh.position.z;
          const dist = Math.sqrt(dx * dx + dz * dz);

          e.mesh.lookAt(playerPos.x, e.mesh.position.y, playerPos.z);

          // AI Special Skill Execution
          if (e.skillCooldown <= 0 && dist < 22 && !e.isFreezed) {
            this.executeEnemySkill(e, playerPos, dx, dz, dist, onHitPlayer);
            e.skillCooldown = e.isBoss ? 3.5 : (4.5 + Math.random() * 3);
          }

          // Flying Enemy Movement
          if (e.skill === 'fly') {
            e.mesh.position.y = 3.5 + Math.sin(e.animTime) * 1.2;
          }

          // Standard Movement AI
          if (dist > 2.2) {
            let moveDirX = dx / dist;
            let moveDirZ = dz / dist;

            // Flanking Movement logic
            if (e.skill === 'flank' && dist > 6) {
              moveDirX += Math.cos(e.animTime) * 0.6;
              moveDirZ += Math.sin(e.animTime) * 0.6;
            }

            e.mesh.position.x += moveDirX * currentSpeed * delta;
            e.mesh.position.z += moveDirZ * currentSpeed * delta;

            // Check Building Collision
            world.checkBuildingCollision(e.mesh.position, 0.9);

            e.armL.rotation.x = Math.sin(e.animTime) * 0.5;
            e.armR.rotation.x = -Math.sin(e.animTime) * 0.5;
          } else {
            // Melee Attack Range
            e.attackCooldown -= delta;
            e.armL.rotation.x = Math.sin(e.animTime * 3) * 1.2;
            e.armR.rotation.x = -Math.sin(e.animTime * 3) * 1.2;

            if (e.attackCooldown <= 0) {
              e.attackCooldown = 1.2;
              onHitPlayer(e.atk);
              soundFx.playHit();
            }
          }
        }

        // Update Dark Orb Projectiles
        for (let pIdx = this.projectiles.length - 1; pIdx >= 0; pIdx--) {
          const p = this.projectiles[pIdx];
          p.mesh.position.addScaledVector(p.dir, p.speed * delta);
          p.life -= delta;

          const pDist = p.mesh.position.distanceTo(playerPos);
          if (pDist < 1.5) {
            onHitPlayer(p.damage);
            soundFx.playHit();
            this.scene.remove(p.mesh);
            this.projectiles.splice(pIdx, 1);
            continue;
          }

          if (p.life <= 0) {
            this.scene.remove(p.mesh);
            this.projectiles.splice(pIdx, 1);
          }
        }
      }

      executeEnemySkill(e, playerPos, dx, dz, dist, onHitPlayer) {
        if (e.isBoss) {
          // 5 BOSS ATTACK PATTERNS (かっこいい技)
          const pattern = Math.floor(Math.random() * 5);
          if (pattern === 0) { // Pattern 1: Ground Shockwave
            const ringGeo = new THREE.RingGeometry(0.5, 6, 16);
            const ringMat = new THREE.MeshBasicMaterial({ color: 0xef4444, side: THREE.DoubleSide, transparent: true, opacity: 0.8 });
            const ring = new THREE.Mesh(ringGeo, ringMat);
            ring.rotation.x = -Math.PI / 2;
            ring.position.copy(e.mesh.position);
            this.scene.add(ring);
            soundFx.playExplosion();
            setTimeout(() => this.scene.remove(ring), 600);
            if (dist < 7) onHitPlayer(e.atk * 1.3);

          } else if (pattern === 1) { // Pattern 2: Shadow Teleport
            e.mesh.position.set(playerPos.x + (Math.random() - 0.5) * 4, 0, playerPos.z + (Math.random() - 0.5) * 4);
            soundFx.playNightScream();

          } else if (pattern === 2) { // Pattern 3: Homing Dark Orb Barrage
            for (let i = 0; i < 3; i++) {
              const orbGeo = new THREE.SphereGeometry(0.4, 8, 8);
              const orbMat = new THREE.MeshBasicMaterial({ color: 0xa855f7 });
              const orb = new THREE.Mesh(orbGeo, orbMat);
              orb.position.set(e.mesh.position.x, 2, e.mesh.position.z);
              
              const dir = new THREE.Vector3().subVectors(playerPos, orb.position).normalize();
              this.scene.add(orb);
              this.projectiles.push({ mesh: orb, dir: dir, speed: 18, damage: e.atk * 0.8, life: 3.5 });
            }

          } else if (pattern === 3) { // Pattern 4: Screech Shockwave
            soundFx.playNightScream();
            if (dist < 12) onHitPlayer(e.atk * 0.9);

          } else if (pattern === 4) { // Pattern 5: Flame Nova
            soundFx.playExplosion();
            if (dist < 9) onHitPlayer(e.atk * 1.5);
          }
        } else {
          // Standard Enemies Skills
          if (e.skill === 'pounce') { // Pounce
            e.mesh.position.x += (dx / dist) * 6;
            e.mesh.position.z += (dz / dist) * 6;
            if (dist < 3) onHitPlayer(e.atk * 1.2);
          } else if (e.skill === 'tackle') { // Tackle
            e.mesh.position.x += (dx / dist) * 4;
            e.mesh.position.z += (dz / dist) * 4;
            if (dist < 2.5) onHitPlayer(e.atk * 1.3);
          }
        }
      }

      removeEnemy(index) {
        const e = this.enemies[index];
        this.scene.remove(e.mesh);
        this.enemies.splice(index, 1);
      }
    }

    class WeaponRenderer {
      constructor(camera) {
        this.camera = camera;
        this.weaponHolder = new THREE.Group();
        this.camera.add(this.weaponHolder);
        
        // FPV Right Hand position
        this.weaponHolder.position.set(0.35, -0.3, -0.55);
        this.currentMesh = null;
        this.isAttackingAnim = false;
        this.animTimer = 0;
      }

      renderWeapon(weaponIndex) {
        if (this.currentMesh) {
          this.weaponHolder.remove(this.currentMesh);
        }

        const group = new THREE.Group();
        const w = WEAPONS[weaponIndex];

        // 3D Player Arm & Hand Base
        const armMat = new THREE.MeshStandardMaterial({ color: 0xe2e8f0, roughness: 0.8 });
        const arm = new THREE.Mesh(new THREE.CylinderGeometry(0.045, 0.05, 0.45), armMat);
        arm.position.set(0.05, -0.15, 0.2);
        arm.rotation.x = Math.PI / 3;
        arm.rotation.z = -0.2;
        group.add(arm);

        if (w.id === 0) { // Slingshot (パチンコ)
          const woodMat = new THREE.MeshStandardMaterial({ color: 0x78350f, roughness: 0.8 });
          const bandMat = new THREE.MeshBasicMaterial({ color: 0xef4444 });
          
          const handle = new THREE.Mesh(new THREE.CylinderGeometry(0.025, 0.025, 0.25), woodMat);
          handle.position.y = -0.05;
          const forkL = new THREE.Mesh(new THREE.CylinderGeometry(0.018, 0.018, 0.15), woodMat);
          forkL.position.set(-0.06, 0.1, 0); forkL.rotation.z = -0.3;
          const forkR = new THREE.Mesh(new THREE.CylinderGeometry(0.018, 0.018, 0.15), woodMat);
          forkR.position.set(0.06, 0.1, 0); forkR.rotation.z = 0.3;

          this.slingshotBand = new THREE.Mesh(new THREE.BoxGeometry(0.14, 0.008, 0.008), bandMat);
          this.slingshotBand.position.set(0, 0.15, -0.02);

          group.add(handle, forkL, forkR, this.slingshotBand);
        } else if (w.id === 1) { // Wooden Stick
          const woodMat = new THREE.MeshStandardMaterial({ color: 0x92400e, roughness: 0.9 });
          const stick = new THREE.Mesh(new THREE.CylinderGeometry(0.03, 0.035, 1.0, 8), woodMat);
          stick.rotation.x = Math.PI / 3.2;
          group.add(stick);
        } else if (w.id === 2) { // Spiked Bat
          const batMat = new THREE.MeshStandardMaterial({ color: 0x475569, metalness: 0.8, roughness: 0.3 });
          const bat = new THREE.Mesh(new THREE.CylinderGeometry(0.05, 0.025, 1.1, 8), batMat);
          bat.rotation.x = Math.PI / 3.2;
          group.add(bat);
        } else if (w.isGun) { // Gun models
          const gunMat = new THREE.MeshStandardMaterial({ color: w.id === 6 ? 0x0284c7 : 0x1e293b, metalness: 0.9, roughness: 0.2 });
          const barrel = new THREE.Mesh(new THREE.BoxGeometry(0.08, 0.1, 0.45), gunMat);
          const handle = new THREE.Mesh(new THREE.BoxGeometry(0.06, 0.18, 0.08), gunMat);
          handle.position.set(0, -0.1, 0.1); handle.rotation.x = -0.2;
          group.add(barrel, handle);

          if (w.id >= 4) {
            const longBarrel = new THREE.Mesh(new THREE.CylinderGeometry(0.025, 0.025, 0.35), gunMat);
            longBarrel.rotation.x = Math.PI / 2;
            longBarrel.position.set(0, 0.02, -0.3);
            group.add(longBarrel);
          }
        }

        this.currentMesh = group;
        this.weaponHolder.add(this.currentMesh);
      }

      triggerAttackAnim(isGun) {
        this.isAttackingAnim = true;
        this.animTimer = 0;
        this.isGunType = isGun;
      }

      update(delta) {
        if (!this.isAttackingAnim || !this.currentMesh) return;

        this.animTimer += delta * 12;

        if (this.isGunType) {
          const recoil = Math.sin(this.animTimer * Math.PI) * 0.12;
          this.currentMesh.position.z = recoil;
          this.currentMesh.rotation.x = recoil * 1.5;
        } else {
          const swing = Math.sin(this.animTimer * Math.PI) * 0.8;
          this.currentMesh.rotation.x = -swing;
          this.currentMesh.rotation.y = -swing * 0.5;
        }

        if (this.animTimer >= 1.0) {
          this.isAttackingAnim = false;
          this.currentMesh.position.set(0, 0, 0);
          this.currentMesh.rotation.set(0, 0, 0);
        }
      }
    }

    class TrapManager {
      constructor(scene) {
        this.scene = scene;
        this.placedTraps = [];
      }

      placeTrap(trapType, position) {
        const trapData = TRAPS_DB.find(t => t.id === trapType);
        if (!trapData) return;

        const group = new THREE.Group();
        const mat = new THREE.MeshStandardMaterial({ color: trapData.color, roughness: 0.5 });

        if (trapType === 'bear') {
          const base = new THREE.Mesh(new THREE.BoxGeometry(1.2, 0.1, 1.2), mat);
          const jaw1 = new THREE.Mesh(new THREE.BoxGeometry(1.2, 0.3, 0.1), mat); jaw1.position.set(0, 0.15, 0.5);
          const jaw2 = new THREE.Mesh(new THREE.BoxGeometry(1.2, 0.3, 0.1), mat); jaw2.position.set(0, 0.15, -0.5);
          group.add(base, jaw1, jaw2);
        } else if (trapType === 'turret') {
          const base = new THREE.Mesh(new THREE.CylinderGeometry(0.6, 0.8, 1.2, 8), mat);
          const head = new THREE.Mesh(new THREE.SphereGeometry(0.4, 8, 8), new THREE.MeshBasicMaterial({ color: 0xef4444 }));
          head.position.y = 0.8;
          group.add(base, head);
        } else {
          const base = new THREE.Mesh(new THREE.CylinderGeometry(1.0, 1.0, 0.12, 12), mat);
          group.add(base);
        }

        group.position.set(position.x, 0.06, position.z);
        this.scene.add(group);

        this.placedTraps.push({
          type: trapType,
          mesh: group,
          pos: position.clone(),
          cooldown: 0
        });
      }

      updateTraps(delta, enemies, onRemoveEnemy) {
        for (let i = this.placedTraps.length - 1; i >= 0; i--) {
          const t = this.placedTraps[i];
          t.cooldown -= delta;

          for (let eIdx = enemies.length - 1; eIdx >= 0; eIdx--) {
            const e = enemies[eIdx];
            const dist = t.pos.distanceTo(e.mesh.position);

            if (dist < 2.0 && t.cooldown <= 0) {
              if (t.type === 'bear') {
                e.hp -= 150;
                e.isFreezed = true;
                e.freezeTimer = 3.0;
                soundFx.playHit();
                this.removeTrap(i);
                break;
              } else if (t.type === 'mine') {
                e.hp -= 300;
                soundFx.playExplosion();
                this.removeTrap(i);
                break;
              } else if (t.type === 'frost') {
                e.isFreezed = true;
                e.freezeTimer = 5.0;
                t.cooldown = 1.0;
              } else if (t.type === 'turret') {
                e.hp -= 45;
                t.cooldown = 0.6;
                soundFx.playAttack(true);
              }

              if (e.hp <= 0) {
                onRemoveEnemy(eIdx, e);
              }
            }
          }
        }
      }

      removeTrap(index) {
        const t = this.placedTraps[index];
        this.scene.remove(t.mesh);
        this.placedTraps.splice(index, 1);
      }
    }

    class GameEngine {
      constructor() {
        this.coins = 1000;
        this.day = 1;
        this.isNight = false;
        this.dayTimeLeft = 40;
        this.kills = 0;
        this.totalBountyEarned = 0;

        this.maxHp = 100;
        this.hp = 100;
        this.selectedWeaponIndex = 0;
        this.selectedArmorIndex = 0;
        this.selectedTrapIndex = 0;

        this.potions = 2;
        this.batteries = 1;
        this.flashlightOn = false;
        this.batteryLevel = 100;

        this.inventoryTraps = { bear: 1, spike: 1, mine: 0, frost: 0, turret: 0 };
        this.upgrades = { hpBoost: 0, atkBoost: 0, speedBoost: 0 };
        this.joystickVector = { x: 0, y: 0 };
        this.cameraEuler = new THREE.Euler(0, 0, 0, 'YXZ');

        this.currentShopTab = 'weapons';

        this.init3D();
        this.initControls();
        this.initQuickslotsUI();
        this.initShopUI();
        this.startLoop();
      }

      init3D() {
        const canvas = document.getElementById('game-canvas');
        this.renderer = new THREE.WebGLRenderer({ canvas: canvas, antialias: true });
        this.renderer.setSize(window.innerWidth, window.innerHeight);
        this.renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));

        this.scene = new THREE.Scene();
        
        // Daytime Bright Sky & Fog
        this.scene.background = new THREE.Color(0x87ceeb);
        this.scene.fog = new THREE.FogExp2(0x87ceeb, 0.005);

        this.camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 400);
        this.camera.position.set(0, 1.7, 0);

        // Ambient Light (Bright during day)
        this.ambientLight = new THREE.AmbientLight(0xffffff, 0.85);
        this.scene.add(this.ambientLight);

        // Sun Directional Light
        this.sunLight = new THREE.DirectionalLight(0xfffbeb, 1.2);
        this.sunLight.position.set(100, 150, 50);
        this.scene.add(this.sunLight);

        // Visible Sun Mesh
        this.sunMesh = new THREE.Mesh(
          new THREE.SphereGeometry(8, 16, 16),
          new THREE.MeshBasicMaterial({ color: 0xfde047 })
        );
        this.sunMesh.position.set(120, 160, 60);
        this.scene.add(this.sunMesh);

        // Moon Directional Light
        this.moonLight = new THREE.DirectionalLight(0x60a5fa, 0.0);
        this.moonLight.position.set(-100, 120, -50);
        this.scene.add(this.moonLight);

        // Visible Moon Mesh
        this.moonMesh = new THREE.Mesh(
          new THREE.SphereGeometry(6, 16, 16),
          new THREE.MeshBasicMaterial({ color: 0xe0f2fe })
        );
        this.moonMesh.position.set(-120, 140, -60);
        this.moonMesh.visible = false;
        this.scene.add(this.moonMesh);

        // Flashlight
        this.flashlight = new THREE.SpotLight(0xfff5ea, 2.5, 45, Math.PI / 5, 0.4, 1);
        this.flashlight.visible = false;
        this.camera.add(this.flashlight);
        this.flashlight.position.set(0, 0, 0);
        this.flashlight.target.position.set(0, 0, -1);
        this.camera.add(this.flashlight.target);

        // Sub-managers
        this.weaponRenderer = new WeaponRenderer(this.camera);
        this.weaponRenderer.renderWeapon(this.selectedWeaponIndex);

        this.world = new WorldBuilder(this.scene);
        this.world.generateWorld();

        this.enemyMgr = new EnemyManager(this.scene);
        this.trapMgr = new TrapManager(this.scene);

        this.ufos = []; // Day 3 UFOs

        window.addEventListener('resize', () => {
          this.camera.aspect = window.innerWidth / window.innerHeight;
          this.camera.updateProjectionMatrix();
          this.renderer.setSize(window.innerWidth, window.innerHeight);
        });
      }

      initControls() {
        const joyZone = document.getElementById('joystick-zone');
        const joyStick = document.getElementById('joystick-stick');
        let joyTouchId = null;
        let joyCenter = { x: 0, y: 0 };

        joyZone.addEventListener('touchstart', (e) => {
          soundFx.init();
          const touch = e.changedTouches[0];
          joyTouchId = touch.identifier;
          const rect = joyZone.getBoundingClientRect();
          joyCenter = { x: rect.left + rect.width / 2, y: rect.top + rect.height / 2 };
          this.updateJoystick(touch.clientX, touch.clientY, joyStick, joyCenter);
        }, { passive: false });

        joyZone.addEventListener('touchmove', (e) => {
          for (let touch of e.changedTouches) {
            if (touch.identifier === joyTouchId) {
              this.updateJoystick(touch.clientX, touch.clientY, joyStick, joyCenter);
            }
          }
        }, { passive: false });

        const endJoystick = (e) => {
          for (let touch of e.changedTouches) {
            if (touch.identifier === joyTouchId) {
              joyTouchId = null;
              this.joystickVector = { x: 0, y: 0 };
              joyStick.style.transform = `translate(0px, 0px)`;
            }
          }
        };
        joyZone.addEventListener('touchend', endJoystick);
        joyZone.addEventListener('touchcancel', endJoystick);

        // Touch Camera Rotation
        let cameraTouchId = null;
        let lastTouchPos = { x: 0, y: 0 };

        window.addEventListener('touchstart', (e) => {
          soundFx.init();
          for (let touch of e.changedTouches) {
            if (touch.clientX > window.innerWidth / 2 && cameraTouchId === null) {
              if (e.target.closest('button') || e.target.closest('aside')) continue;
              cameraTouchId = touch.identifier;
              lastTouchPos = { x: touch.clientX, y: touch.clientY };
            }
          }
        });

        window.addEventListener('touchmove', (e) => {
          for (let touch of e.changedTouches) {
            if (touch.identifier === cameraTouchId) {
              const dx = touch.clientX - lastTouchPos.x;
              const dy = touch.clientY - lastTouchPos.y;
              lastTouchPos = { x: touch.clientX, y: touch.clientY };

              this.cameraEuler.y -= dx * 0.005;
              this.cameraEuler.x -= dy * 0.005;
              this.cameraEuler.x = Math.max(-Math.PI / 3, Math.min(Math.PI / 3, this.cameraEuler.x));
            }
          }
        });

        const endCamera = (e) => {
          for (let touch of e.changedTouches) {
            if (touch.identifier === cameraTouchId) cameraTouchId = null;
          }
        };
        window.addEventListener('touchend', endCamera);
        window.addEventListener('touchcancel', endCamera);

        // Keyboard Controls
        this.keys = {};
        window.addEventListener('keydown', (e) => {
          soundFx.init();
          this.keys[e.code] = true;
          if (e.code === 'KeyE') this.toggleShop();
          if (e.code === 'KeyF') this.toggleFlashlight();
          if (e.code === 'Space') {
            if (this.currentChestPrompt) {
              this.openChest(this.currentChestPrompt);
            } else {
              this.attack();
            }
          }
        });
        window.addEventListener('keyup', (e) => this.keys[e.code] = false);

        document.getElementById('btn-attack').addEventListener('click', () => this.attack());
        document.getElementById('interact-prompt').addEventListener('click', () => {
          if (this.currentChestPrompt) this.openChest(this.currentChestPrompt);
        });
      }

      updateJoystick(clientX, clientY, stickElem, center) {
        const maxDist = 45;
        let dx = clientX - center.x;
        let dy = clientY - center.y;
        const dist = Math.sqrt(dx * dx + dy * dy);

        if (dist > maxDist) {
          dx = (dx / dist) * maxDist;
          dy = (dy / dist) * maxDist;
        }

        stickElem.style.transform = `translate(${dx}px, ${dy}px)`;
        this.joystickVector = { x: dx / maxDist, y: dy / maxDist };
      }

      attack() {
        const currentWeapon = WEAPONS[this.selectedWeaponIndex];
        soundFx.playAttack(currentWeapon.isGun);

        this.weaponRenderer.triggerAttackAnim(currentWeapon.isGun);

        const totalAtk = (currentWeapon.atk + (this.upgrades.atkBoost * 15));
        const attackRange = currentWeapon.range;

        const playerPos = this.camera.position;
        const forward = new THREE.Vector3(0, 0, -1).applyEuler(this.cameraEuler);

        for (let i = this.enemyMgr.enemies.length - 1; i >= 0; i--) {
          const e = this.enemyMgr.enemies[i];
          const toEnemy = new THREE.Vector3().subVectors(e.mesh.position, playerPos);
          const dist = toEnemy.length();

          if (dist <= attackRange) {
            toEnemy.normalize();
            const angle = forward.angleTo(toEnemy);
            const maxAngle = currentWeapon.isGun ? Math.PI / 3 : Math.PI / 1.8;
            
            if (angle < maxAngle) {
              e.hp -= totalAtk;
              soundFx.playHit();

              e.mesh.position.addScaledVector(toEnemy, 0.6);

              if (e.hp <= 0) {
                this.onEnemyKilled(i, e);
              }
            }
          }
        }
      }

      onEnemyKilled(index, enemyObj) {
        this.coins += enemyObj.bounty;
        this.totalBountyEarned += enemyObj.bounty;
        this.kills++;
        soundFx.playCoin();
        this.showBanner(`撃破！ +${enemyObj.bounty} コイン獲得`);
        this.enemyMgr.removeEnemy(index);
      }

      updateDayCycle(delta) {
        this.dayTimeLeft -= delta;

        if (this.dayTimeLeft <= 0) {
          this.isNight = !this.isNight;
          const vignette = document.getElementById('vignette-overlay');

          if (this.isNight) {
            // NIGHT PHASE TRANSITION
            this.dayTimeLeft = 60;
            this.showBanner(`🌙 NIGHT ${this.day}: 恐怖の夜が訪れた...`);
            soundFx.playNightScream();

            this.scene.background = new THREE.Color(0x0b1120);
            this.scene.fog.color = new THREE.Color(0x0b1120);
            this.scene.fog.density = 0.015;

            this.ambientLight.intensity = 0.3;
            this.sunLight.intensity = 0.0;
            this.sunMesh.visible = false;

            this.moonLight.intensity = 0.6;
            this.moonMesh.visible = true;

            vignette.className = "vignette-night absolute inset-0 z-10 pointer-events-none transition-all duration-1000";

            this.spawnNightWave();
          } else {
            // DAY PHASE TRANSITION
            this.day++;
            this.dayTimeLeft = 40;
            this.showBanner(`☀️ DAY ${this.day}: 日が昇った！宝箱とショップで準備せよ`);

            this.scene.background = new THREE.Color(0x87ceeb);
            this.scene.fog.color = new THREE.Color(0x87ceeb);
            this.scene.fog.density = 0.005;

            this.ambientLight.intensity = 0.85;
            this.sunLight.intensity = 1.2;
            this.sunMesh.visible = true;

            this.moonLight.intensity = 0.0;
            this.moonMesh.visible = false;

            vignette.className = "vignette-day absolute inset-0 z-10 pointer-events-none transition-all duration-1000";

            const bonus = 120 + (this.day * 30);
            this.coins += bonus;
            this.showBanner(`生存報酬: +${bonus} コイン`);
            document.getElementById('boss-hud').classList.add('hidden');

            // Remove UFOs if daytime
            this.clearUFOs();
          }
        }

        // Night Spawning
        if (this.isNight && Math.random() < 0.03 && this.enemyMgr.enemies.length < (3 + Math.floor(this.day * 0.5))) {
          const randTypeIndex = Math.floor(Math.random() * Math.min(13, 2 + Math.floor(this.day / 2)));
          this.enemyMgr.spawnEnemy(ENEMY_TYPES[randTypeIndex], this.camera.position, this.day);
        }

        this.updateHUD();
      }

      spawnNightWave() {
        if (this.day === 3) {
          // DAY 3 SPECIAL EVENT: 4 GIANT UFOs & ALIEN INVASION
          this.showBanner("⚠️ DAY 3: 巨大UFO襲来！エイリアン大群が発生！");
          this.spawn4UFOs();

          for (let i = 0; i < 7; i++) {
            const alienType = ENEMY_TYPES[2 + (i % 3)]; // Grey, Warrior, Mother
            this.enemyMgr.spawnEnemy(alienType, this.camera.position, this.day);
          }
          return;
        }

        const isSuperBossDay = (this.day % 50 === 0);
        const isBossDay = (this.day % 5 === 0);

        if (isSuperBossDay) {
          const boss = this.enemyMgr.spawnEnemy(ENEMY_TYPES[14], this.camera.position, this.day);
          this.setupBossHUD(boss);
        } else if (isBossDay) {
          const boss = this.enemyMgr.spawnEnemy(ENEMY_TYPES[13], this.camera.position, this.day);
          this.setupBossHUD(boss);
        } else {
          const count = Math.min(6, 2 + Math.floor(this.day * 0.3));
          for (let i = 0; i < count; i++) {
            const typeIndex = Math.floor(Math.random() * Math.min(13, 2 + Math.floor(this.day / 3)));
            this.enemyMgr.spawnEnemy(ENEMY_TYPES[typeIndex], this.camera.position, this.day);
          }
        }
      }

      spawn4UFOs() {
        const ufoGeo = new THREE.CylinderGeometry(8, 12, 3, 16);
        const ufoMat = new THREE.MeshStandardMaterial({ color: 0x334155, metalness: 0.9, roughness: 0.2 });
        const lightMat = new THREE.MeshBasicMaterial({ color: 0x22d3ee });

        const positions = [
          { x: -60, z: -60 }, { x: 60, z: -60 },
          { x: -60, z: 60 }, { x: 60, z: 60 }
        ];

        positions.forEach(p => {
          const group = new THREE.Group();
          const disc = new THREE.Mesh(ufoGeo, ufoMat);
          const dome = new THREE.Mesh(new THREE.SphereGeometry(5, 12, 12), lightMat);
          dome.position.y = 1.5;

          group.add(disc, dome);
          group.position.set(p.x, 35, p.z);
          this.scene.add(group);
          this.ufos.push(group);
        });
      }

      clearUFOs() {
        this.ufos.forEach(ufo => this.scene.remove(ufo));
        this.ufos = [];
      }

      setupBossHUD(boss) {
        const bossHud = document.getElementById('boss-hud');
        document.getElementById('boss-name').innerText = `⚔️ ${boss.name}`;
        bossHud.classList.remove('hidden');
      }

      updateHUD() {
        document.getElementById('hud-day').innerText = `DAY ${this.day}`;
        document.getElementById('hud-time-phase').innerText = this.isNight ? "🌙 夜 (生存フェーズ)" : "☀️ 昼 (安全探索)";
        document.getElementById('hud-coins').innerText = this.coins.toLocaleString();
        document.getElementById('hud-hp-text').innerText = `${Math.ceil(this.hp)} / ${this.maxHp}`;
        document.getElementById('hud-hp-bar').style.width = `${Math.max(0, (this.hp / this.maxHp) * 100)}%`;
        
        const totalPhaseTime = this.isNight ? 60 : 40;
        document.getElementById('hud-time-bar').style.width = `${(this.dayTimeLeft / totalPhaseTime) * 100}%`;

        const activeBoss = this.enemyMgr.enemies.find(e => e.isBoss);
        if (activeBoss) {
          document.getElementById('boss-hp-text').innerText = `${Math.max(0, activeBoss.hp)} / ${activeBoss.maxHp}`;
          document.getElementById('boss-hp-bar').style.width = `${Math.max(0, (activeBoss.hp / activeBoss.maxHp) * 100)}%`;
        }

        document.getElementById('potion-count').innerText = `x${this.potions}`;
        document.getElementById('battery-count').innerText = `x${this.batteries}`;

        // Check Nearby Treasure Chests
        let nearChest = null;
        for (let c of this.world.chests) {
          if (!c.opened && c.mesh.position.distanceTo(this.camera.position) < 3.0) {
            nearChest = c;
            break;
          }
        }

        const prompt = document.getElementById('interact-prompt');
        if (nearChest) {
          this.currentChestPrompt = nearChest;
          prompt.classList.remove('hidden');
        } else {
          this.currentChestPrompt = null;
          prompt.classList.add('hidden');
        }
      }

      openChest(chestObj) {
        chestObj.opened = true;
        this.scene.remove(chestObj.mesh);

        const keys = Object.keys(this.inventoryTraps);
        const rewardTrap = keys[Math.floor(Math.random() * keys.length)];
        this.inventoryTraps[rewardTrap]++;

        const trapName = TRAPS_DB.find(t => t.id === rewardTrap).name;
        soundFx.playCoin();
        this.showBanner(`📦 宝箱を発見！ [${trapName}] を獲得！`);
        this.initQuickslotsUI();
      }

      placeCurrentTrap() {
        const trapTypes = TRAPS_DB.map(t => t.id);
        const selectedType = trapTypes[this.selectedTrapIndex];

        if (this.inventoryTraps[selectedType] > 0) {
          this.inventoryTraps[selectedType]--;
          this.trapMgr.placeTrap(selectedType, this.camera.position);
          this.showBanner(`${TRAPS_DB[this.selectedTrapIndex].name} を設置した！`);
          this.initQuickslotsUI();
        } else {
          this.showBanner("❌ トラップの所持数がありません");
        }
      }

      initQuickslotsUI() {
        // Weapon Quickslots
        const wContainer = document.getElementById('weapon-slots-container');
        wContainer.innerHTML = '';
        WEAPONS.forEach((w, idx) => {
          if (!w.owned) return;
          const isSelected = (this.selectedWeaponIndex === idx);
          const btn = document.createElement('button');
          btn.className = `w-11 h-11 md:w-13 md:h-13 bg-slate-800 border-2 ${isSelected ? 'border-amber-400 bg-amber-950/60' : 'border-slate-700'} rounded-xl flex flex-col items-center justify-center relative active:scale-95 shrink-0`;
          btn.onclick = () => this.selectWeapon(idx);
          btn.innerHTML = `
            <span class="text-lg">${w.icon}</span>
            <span class="text-[8px] text-slate-300 mt-0.5 truncate max-w-full px-0.5">${w.name}</span>
          `;
          wContainer.appendChild(btn);
        });

        // Trap Quickslots
        const tContainer = document.getElementById('trap-slots-container');
        tContainer.innerHTML = '';
        TRAPS_DB.forEach((t, idx) => {
          const count = this.inventoryTraps[t.id] || 0;
          const isSelected = (this.selectedTrapIndex === idx);
          const btn = document.createElement('button');
          btn.className = `w-11 h-11 md:w-13 md:h-13 bg-slate-800 border-2 ${isSelected ? 'border-indigo-400 bg-indigo-950/60' : 'border-slate-700'} rounded-xl flex flex-col items-center justify-center relative active:scale-95 shrink-0`;
          btn.onclick = () => { this.selectedTrapIndex = idx; this.initQuickslotsUI(); };
          btn.innerHTML = `
            <span class="text-lg">${t.icon}</span>
            <span class="absolute top-0.5 right-1 text-[9px] bg-indigo-600 px-1 rounded-full font-bold">x${count}</span>
            <span class="text-[8px] text-slate-400 truncate max-w-full px-0.5">${t.name}</span>
          `;
          tContainer.appendChild(btn);
        });
      }

      selectWeapon(idx) {
        if (WEAPONS[idx].owned) {
          this.selectedWeaponIndex = idx;
          this.weaponRenderer.renderWeapon(idx);
          this.initQuickslotsUI();
        }
      }

      usePotion() {
        if (this.potions > 0 && this.hp < this.maxHp) {
          this.potions--;
          this.hp = Math.min(this.maxHp, this.hp + 60);
          this.showBanner("💚 体力を60回復した！");
          soundFx.playBuy();
        }
      }

      useBattery() {
        if (this.batteries > 0) {
          this.batteries--;
          this.batteryLevel = 100;
          document.getElementById('hud-battery-bar').style.width = '100%';
          this.showBanner("🔋 懐中電灯を充電した！");
          soundFx.playBuy();
        }
      }

      toggleFlashlight() {
        this.flashlightOn = !this.flashlightOn;
        this.flashlight.visible = this.flashlightOn;
      }

      showBanner(msg) {
        const banner = document.getElementById('banner-msg');
        banner.innerText = msg;
        banner.classList.remove('opacity-0', 'translate-y-4');
        banner.classList.add('opacity-100', 'translate-y-0');
        setTimeout(() => {
          banner.classList.add('opacity-0', 'translate-y-4');
          banner.classList.remove('opacity-100', 'translate-y-0');
        }, 3200);
      }

      initShopUI() {
        this.renderShopItems();
      }

      toggleShop() {
        const modal = document.getElementById('shop-modal');
        modal.classList.toggle('hidden');
        document.getElementById('shop-coins').innerText = this.coins.toLocaleString();
        if (!modal.classList.contains('hidden')) {
          this.renderShopItems();
        }
      }

      switchShopTab(tab) {
        this.currentShopTab = tab;
        ['weapons', 'armors', 'traps', 'upgrades'].forEach(t => {
          const btn = document.getElementById(`tab-btn-${t}`);
          if (t === tab) {
            btn.className = "px-4 py-2 rounded-xl text-sm font-bold bg-amber-700 text-white";
          } else {
            btn.className = "px-4 py-2 rounded-xl text-sm font-bold bg-slate-800 text-slate-300";
          }
        });
        this.renderShopItems();
      }

      renderShopItems() {
        const grid = document.getElementById('shop-items-grid');
        grid.innerHTML = '';

        if (this.currentShopTab === 'weapons') {
          WEAPONS.forEach((w, idx) => {
            const isOwned = w.owned;
            const card = document.createElement('article');
            card.className = "bg-slate-900 border border-slate-800 rounded-2xl p-4 flex flex-col justify-between";
            card.innerHTML = `
              <header class="flex justify-between items-center mb-2">
                <span class="text-3xl">${w.icon}</span>
                <span class="text-xs font-bold text-amber-400">攻撃力 +${w.atk}</span>
              </header>
              <main class="mb-4">
                <h3 class="font-bold text-lg text-slate-100">${w.name}</h3>
                <p class="text-xs text-slate-400 mt-1">${w.desc}</p>
              </main>
              <footer>
                ${isOwned ? 
                  `<button disabled class="w-full bg-slate-700 text-slate-400 font-bold py-2 rounded-xl text-xs">所持済み</button>` :
                  `<button onclick="game.buyWeapon(${idx})" class="w-full bg-amber-600 hover:bg-amber-500 text-white font-bold py-2 rounded-xl text-xs">💰 ${w.price} コインで購入</button>`
                }
              </footer>
            `;
            grid.appendChild(card);
          });
        } else if (this.currentShopTab === 'armors') {
          ARMORS.forEach((a, idx) => {
            const isOwned = (this.selectedArmorIndex === idx);
            const card = document.createElement('article');
            card.className = "bg-slate-900 border border-slate-800 rounded-2xl p-4 flex flex-col justify-between";
            card.innerHTML = `
              <header class="flex justify-between items-center mb-2">
                <span class="text-3xl">🛡️</span>
                <span class="text-xs font-bold text-cyan-400">軽減 ${a.def}%</span>
              </header>
              <main class="mb-4">
                <h3 class="font-bold text-lg text-slate-100">${a.name}</h3>
                <p class="text-xs text-slate-400 mt-1">${a.desc}</p>
              </main>
              <footer>
                ${isOwned ? 
                  `<button disabled class="w-full bg-slate-700 text-slate-400 font-bold py-2 rounded-xl text-xs">装備中</button>` :
                  `<button onclick="game.buyArmor(${idx})" class="w-full bg-cyan-600 hover:bg-cyan-500 text-white font-bold py-2 rounded-xl text-xs">💰 ${a.price} コインで装備</button>`
                }
              </footer>
            `;
            grid.appendChild(card);
          });
        } else if (this.currentShopTab === 'traps') {
          TRAPS_DB.forEach((t) => {
            const card = document.createElement('article');
            card.className = "bg-slate-900 border border-slate-800 rounded-2xl p-4 flex flex-col justify-between";
            card.innerHTML = `
              <header class="flex justify-between items-center mb-2">
                <span class="text-3xl">${t.icon}</span>
                <span class="text-xs font-bold text-indigo-400">所持: ${this.inventoryTraps[t.id] || 0}</span>
              </header>
              <main class="mb-4">
                <h3 class="font-bold text-lg text-slate-100">${t.name}</h3>
                <p class="text-xs text-slate-400 mt-1">${t.desc}</p>
              </main>
              <footer>
                <button onclick="game.buyTrap('${t.id}', ${t.price})" class="w-full bg-indigo-600 hover:bg-indigo-500 text-white font-bold py-2 rounded-xl text-xs">💰 ${t.price} コインで購入</button>
              </footer>
            `;
            grid.appendChild(card);
          });
        } else {
          const items = [
            { type: 'potion', name: "回復ポーション", price: 150, icon: "🧪", desc: "HPを60回復する" },
            { type: 'battery', name: "電灯バッテリー", price: 100, icon: "🔋", desc: "懐中電灯を充電" },
            { type: 'hpUpgrade', name: "最大HP拡張 (+20)", price: 400 * (this.upgrades.hpBoost + 1), icon: "❤️", desc: "最大体力の上限を永久に増加" },
            { type: 'atkUpgrade', name: "攻撃力強化 (+15)", price: 500 * (this.upgrades.atkBoost + 1), icon: "⚡", desc: "全ての攻撃ダメージをアップ" }
          ];

          items.forEach((item) => {
            const card = document.createElement('article');
            card.className = "bg-slate-900 border border-slate-800 rounded-2xl p-4 flex flex-col justify-between";
            card.innerHTML = `
              <header class="flex justify-between items-center mb-2">
                <span class="text-3xl">${item.icon}</span>
              </header>
              <main class="mb-4">
                <h3 class="font-bold text-lg text-slate-100">${item.name}</h3>
                <p class="text-xs text-slate-400 mt-1">${item.desc}</p>
              </main>
              <footer>
                <button onclick="game.buyItem('${item.type}', ${item.price})" class="w-full bg-emerald-600 hover:bg-emerald-500 text-white font-bold py-2 rounded-xl text-xs">💰 ${item.price} コインで購入</button>
              </footer>
            `;
            grid.appendChild(card);
          });
        }
      }

      buyWeapon(idx) {
        const w = WEAPONS[idx];
        if (this.coins >= w.price) {
          this.coins -= w.price;
          w.owned = true;
          this.selectedWeaponIndex = idx;
          soundFx.playBuy();
          this.showBanner(`${w.name} を購入・装備した！`);
          this.weaponRenderer.renderWeapon(idx);
          this.initQuickslotsUI();
          this.renderShopItems();
          this.updateHUD();
        } else {
          this.showBanner("❌ コインが足りません");
        }
      }

      buyArmor(idx) {
        const a = ARMORS[idx];
        if (this.coins >= a.price) {
          this.coins -= a.price;
          this.selectedArmorIndex = idx;
          soundFx.playBuy();
          this.showBanner(`${a.name} を装備した！`);
          this.renderShopItems();
          this.updateHUD();
        } else {
          this.showBanner("❌ コインが足りません");
        }
      }

      buyTrap(trapId, price) {
        if (this.coins >= price) {
          this.coins -= price;
          this.inventoryTraps[trapId] = (this.inventoryTraps[trapId] || 0) + 1;
          soundFx.playBuy();
          this.showBanner("トラップを購入した！");
          this.initQuickslotsUI();
          this.renderShopItems();
          this.updateHUD();
        } else {
          this.showBanner("❌ コインが足りません");
        }
      }

      buyItem(type, price) {
        if (this.coins >= price) {
          this.coins -= price;
          soundFx.playBuy();
          if (type === 'potion') this.potions++;
          if (type === 'battery') this.batteries++;
          if (type === 'hpUpgrade') {
            this.upgrades.hpBoost++;
            this.maxHp += 20;
            this.hp += 20;
          }
          if (type === 'atkUpgrade') this.upgrades.atkBoost++;

          this.showBanner("購入完了！");
          this.renderShopItems();
          this.updateHUD();
        } else {
          this.showBanner("❌ コインが足りません");
        }
      }

      startLoop() {
        let lastTime = performance.now();

        const animate = (now) => {
          requestAnimationFrame(animate);
          const delta = Math.min(0.1, (now - lastTime) / 1000);
          lastTime = now;

          this.updatePlayerMovement(delta);
          this.world.checkBuildingCollision(this.camera.position, 0.7);

          this.enemyMgr.updateEnemies(delta, this.camera.position, this.world, (damage) => this.takeDamage(damage));
          this.trapMgr.updateTraps(delta, this.enemyMgr.enemies, (idx, e) => this.onEnemyKilled(idx, e));

          this.updateDayCycle(delta);
          this.weaponRenderer.update(delta);

          this.camera.quaternion.setFromEuler(this.cameraEuler);

          if (this.flashlightOn && this.batteryLevel > 0) {
            this.batteryLevel -= delta * 0.8;
            document.getElementById('hud-battery-bar').style.width = `${Math.max(0, this.batteryLevel)}%`;
            if (this.batteryLevel <= 0) {
              this.flashlightOn = false;
              this.flashlight.visible = false;
            }
          }

          this.renderer.render(this.scene, this.camera);
        };

        requestAnimationFrame(animate);
      }

      updatePlayerMovement(delta) {
        let moveX = 0;
        let moveZ = 0;

        if (this.joystickVector.x !== 0 || this.joystickVector.y !== 0) {
          moveX = this.joystickVector.x;
          moveZ = this.joystickVector.y;
        }

        if (this.keys['KeyW'] || this.keys['ArrowUp']) moveZ = -1;
        if (this.keys['KeyS'] || this.keys['ArrowDown']) moveZ = 1;
        if (this.keys['KeyA'] || this.keys['ArrowLeft']) moveX = -1;
        if (this.keys['KeyD'] || this.keys['ArrowRight']) moveX = 1;

        if (moveX !== 0 || moveZ !== 0) {
          const moveDir = new THREE.Vector3(moveX, 0, moveZ).normalize();
          const yawEuler = new THREE.Euler(0, this.cameraEuler.y, 0, 'YXZ');
          moveDir.applyEuler(yawEuler);

          const moveSpeed = 6.8 + (this.upgrades.speedBoost * 0.6);
          this.camera.position.x += moveDir.x * moveSpeed * delta;
          this.camera.position.z += moveDir.z * moveSpeed * delta;

          this.camera.position.x = Math.max(-HALF_MAP + 2, Math.min(HALF_MAP - 2, this.camera.position.x));
          this.camera.position.z = Math.max(-HALF_MAP + 2, Math.min(HALF_MAP - 2, this.camera.position.z));
        }
      }

      takeDamage(rawDamage) {
        const armorDef = ARMORS[this.selectedArmorIndex].def;
        const actualDamage = Math.max(1, rawDamage * (1 - armorDef / 100));
        this.hp -= actualDamage;

        if (this.hp <= 0) {
          this.hp = 0;
          this.triggerGameOver();
        }
        this.updateHUD();
      }

      triggerGameOver() {
        document.getElementById('gameover-modal').classList.remove('hidden');
        document.getElementById('gameover-stats').innerText = 
          `生存日数: DAY ${this.day} | 撃破数: ${this.kills} 体 | 獲得賞金: ${this.totalBountyEarned} コイン`;
      }
    }

    let game;
    window.onload = () => {
      game = new GameEngine();
    };
  </script>
</body>
</html>
