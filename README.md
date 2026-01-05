<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Jackpot Protocol: v15.0 Armageddon</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            overflow: hidden;
            background-color: #020005;
            font-family: 'Courier New', Courier, monospace;
            user-select: none;
            cursor: crosshair;
        }
        canvas {
            display: block;
        }
        #ui-layer {
            position: absolute;
            top: 20px;
            left: 20px;
            color: #fff;
            font-size: 20px;
            pointer-events: none;
            z-index: 5;
            text-shadow: 2px 2px 0 #000;
            width: 300px;
        }
        .hud-bar {
            margin-bottom: 5px;
            font-weight: bold;
        }
        .hud-value { color: #00ffff; }
        
        #xp-container {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 10px;
            background: #111;
            z-index: 6;
        }
        #xp-bar {
            width: 0%;
            height: 100%;
            background: #00ffff;
            box-shadow: 0 0 10px #00ffff;
            transition: width 0.1s;
        }

        .screen-overlay {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            background: rgba(2, 0, 5, 0.98);
            color: white;
            z-index: 10;
            transition: opacity 0.3s;
        }
        h1 {
            font-size: 70px;
            margin: 0;
            color: #ff00ff;
            text-shadow: 0 0 30px #ff00ff, 4px 4px 0px #00ffff;
            text-transform: uppercase;
            letter-spacing: 5px;
            text-align: center;
        }
        p { font-size: 20px; color: #ccc; margin-top: 15px; max-width: 600px; text-align: center; line-height: 1.5; }
        .btn {
            margin-top: 20px;
            padding: 15px 40px;
            font-size: 20px;
            background: rgba(0, 255, 255, 0.1);
            color: #00ffff;
            border: 2px solid #00ffff;
            box-shadow: 0 0 20px #00ffff;
            cursor: pointer;
            transition: all 0.1s;
            font-family: inherit;
            font-weight: bold;
            text-transform: uppercase;
            min-width: 250px;
        }
        .btn:hover {
            background: #00ffff;
            color: #000;
            box-shadow: 0 0 50px #00ffff;
            transform: scale(1.05);
        }
        .btn-secondary {
            border-color: #ff00ff;
            color: #ff00ff;
            box-shadow: 0 0 20px #ff00ff;
        }
        .btn-secondary:hover {
            background: #ff00ff;
            box-shadow: 0 0 50px #ff00ff;
            color: #000;
        }
        
        #shop-screen { display: none; }
        .shop-grid {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 20px;
            margin-top: 30px;
            max-width: 900px;
        }
        .shop-item {
            border: 1px solid #333;
            background: rgba(20, 20, 30, 0.9);
            padding: 15px;
            text-align: center;
        }
        .shop-item h3 { margin: 0 0 10px 0; color: #00ffff; }
        .shop-item p { font-size: 14px; color: #aaa; height: 40px; }
        .shop-cost { color: #ffd700; font-weight: bold; margin: 10px 0; }
        .currency-display {
            position: absolute;
            top: 50px; right: 20px;
            font-size: 24px;
            color: #ffd700;
            text-shadow: 0 0 10px #ffd700;
            z-index: 20;
        }

        #char-select { display: flex; flex-direction: row; gap: 30px; margin-top: 30px; }
        .char-card {
            width: 240px;
            padding: 20px;
            border: 2px solid #333;
            background: rgba(10, 10, 20, 0.9);
            text-align: center;
            cursor: pointer;
            transition: 0.2s;
            position: relative;
            overflow: hidden;
        }
        .char-card:hover {
            border-color: #00ffff;
            box-shadow: 0 0 30px rgba(0, 255, 255, 0.4);
            transform: translateY(-10px);
        }
        .char-title { color: #fff; font-size: 22px; margin-bottom: 10px; font-weight: bold; }
        .char-desc { color: #aaa; font-size: 13px; line-height: 1.4; }
        
        #scanlines {
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: linear-gradient(rgba(18, 16, 16, 0) 50%, rgba(0, 0, 0, 0.25) 50%), linear-gradient(90deg, rgba(255, 0, 0, 0.06), rgba(0, 255, 0, 0.02), rgba(0, 0, 255, 0.06));
            background-size: 100% 4px, 6px 100%;
            pointer-events: none;
            z-index: 100;
        }
        #warning-overlay {
            position: absolute;
            top: 30%;
            width: 100%;
            text-align: center;
            font-size: 60px;
            color: #ff0000;
            font-weight: bold;
            text-shadow: 0 0 30px #ff0000;
            display: none;
            z-index: 8;
            letter-spacing: 10px;
            animation: blink 0.2s infinite;
        }
        #swarm-overlay {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(255, 0, 0, 0.1);
            pointer-events: none;
            display: none;
            z-index: 7;
            box-shadow: inset 0 0 100px #ff0000;
        }
        @keyframes blink { 0% {opacity:1;} 50% {opacity:0;} 100% {opacity:1;} }
    </style>
</head>
<body>

    <div id="xp-container"><div id="xp-bar"></div></div>
    <div id="scanlines"></div>
    <div id="swarm-overlay"></div>
    <div id="warning-overlay">⚠️ BOSS APPROACHING ⚠️</div>
    
    <div class="currency-display">SHARDS: <span id="shard-display">0</span></div>

    <div id="ui-layer">
        <div class="hud-bar">LVL <span id="level-display" class="hud-value">1</span></div>
        <div class="hud-bar">SCORE <span id="score-display" class="hud-value">0</span></div>
        <div class="hud-bar">HP <span id="hp-display" class="hud-value">100</span>%</div>
        <div class="hud-bar">LUCK <span id="luck-display" class="hud-value">0</span>%</div>
        <div class="hud-bar">WEAPONS <span id="weapon-display" class="hud-value">1</span></div>
    </div>

    <div id="start-screen" class="screen-overlay">
        <h1>Jackpot Protocol</h1>
        <p style="color: #00ffff; text-shadow: 0 0 10px #00ffff; font-weight: bold;">/// V15.0: ARMAGEDDON ///</p>
        <p>10 of Each Weapon. Infinite Scaling. Total Chaos.</p>
        <button class="btn" id="start-btn">INITIATE RUN</button>
        <button class="btn btn-secondary" id="shop-btn">BLACK MARKET</button>
    </div>

    <div id="shop-screen" class="screen-overlay">
        <h1>BLACK MARKET</h1>
        <div style="color: #ffd700; font-size: 24px; margin-bottom: 20px;">AVAILABLE SHARDS: <span id="shop-shards">0</span></div>
        <div class="shop-grid" id="shop-container"></div>
        <button class="btn" onclick="game.closeShop()">RETURN</button>
    </div>

    <div id="char-select-screen" class="screen-overlay" style="display: none;">
        <h1 style="font-size: 40px;">SELECT OPERATIVE</h1>
        <div id="char-select">
            <div class="char-card" onclick="game.selectChar('glitch')">
                <div class="char-title" style="color:#00ffff">THE GLITCH</div>
                <div class="char-desc">Balanced.<br>Weapon: Plasma Pistol<br><br><i>"Adaptable."</i></div>
            </div>
            <div class="char-card" onclick="game.selectChar('heavy')">
                <div class="char-title" style="color:#00ff00">HEAVY BYTE</div>
                <div class="char-desc">High HP.<br>Weapon: Shotgun<br><br><i>"Tank."</i></div>
            </div>
            <div class="char-card" onclick="game.selectChar('runner')">
                <div class="char-title" style="color:#ff00ff">NEON RUNNER</div>
                <div class="char-desc">Fast.<br>Weapon: SMG<br><br><i>"Speed."</i></div>
            </div>
        </div>
        <button class="btn btn-secondary" style="margin-top:20px;" onclick="location.reload()">BACK</button>
    </div>

    <canvas id="gameCanvas"></canvas>

<script>
/**
 * JACKPOT PROTOCOL: INFINITE SPIN v15.0
 */

// --- SAVE SYSTEM ---
const SaveSystem = {
    data: {
        shards: 0,
        upgrades: { dmg: 0, hp: 0, luck: 0, greed: 0 }
    },
    init() {
        const saved = localStorage.getItem('jackpot_protocol_v15');
        if (saved) this.data = JSON.parse(saved);
        this.updateUI();
    },
    save() {
        localStorage.setItem('jackpot_protocol_v15', JSON.stringify(this.data));
        this.updateUI();
    },
    addShards(amount) {
        this.data.shards += Math.floor(amount);
        this.save();
    },
    updateUI() {
        const el = document.getElementById('shard-display');
        if(el) el.innerText = this.data.shards;
        const shopEl = document.getElementById('shop-shards');
        if(shopEl) shopEl.innerText = this.data.shards;
    }
};

const META_UPGRADES = [
    { id: 'dmg', name: 'CORE OVERCLOCK', desc: '+10% Base Damage', cost: 100, scale: 1.5, max: 10 },
    { id: 'hp', name: 'REINFORCED HULL', desc: '+10 Max HP', cost: 80, scale: 1.5, max: 10 },
    { id: 'luck', name: 'RNG MANIPULATOR', desc: '+5 Luck', cost: 150, scale: 1.5, max: 10 },
    { id: 'greed', name: 'DATA MINER', desc: '+10% XP Gain', cost: 120, scale: 1.5, max: 10 }
];

// --- AUDIO ---
const AudioEngine = {
    ctx: null,
    init() {
        if (!this.ctx) this.ctx = new (window.AudioContext || window.webkitAudioContext)();
        if (this.ctx.state === 'suspended') this.ctx.resume();
    },
    playTone(freq, type, duration, vol = 0.1, slideTo = null) {
        if (!this.ctx) return;
        const t = this.ctx.currentTime;
        const osc = this.ctx.createOscillator();
        const gain = this.ctx.createGain();
        osc.type = type;
        osc.frequency.setValueAtTime(freq, t);
        if (slideTo) osc.frequency.exponentialRampToValueAtTime(slideTo, t + duration);
        gain.gain.setValueAtTime(vol, t);
        gain.gain.exponentialRampToValueAtTime(0.01, t + duration);
        osc.connect(gain);
        gain.connect(this.ctx.destination);
        osc.start();
        osc.stop(t + duration);
    },
    playShoot(type) {
        if (type === 'SHOTGUN') this.playTone(80, 'square', 0.2, 0.03, 30);
        else if (type === 'SMG') this.playTone(300, 'sawtooth', 0.05, 0.02, 150);
        else if (type === 'RAILGUN') this.playTone(800, 'square', 0.3, 0.03, 100);
        else if (type === 'ROCKET') this.playTone(100, 'sawtooth', 0.4, 0.05, 50);
        else if (type === 'BOOMERANG') this.playTone(400, 'sine', 0.1, 0.02);
        else if (type === 'FLAME') this.playTone(100, 'sawtooth', 0.1, 0.1); 
        else this.playTone(200, 'triangle', 0.1, 0.02, 50);
    },
    playHit() { this.playTone(100, 'sawtooth', 0.05, 0.01, 50); },
    playExplosion() {
        if (!this.ctx) return;
        const t = this.ctx.currentTime;
        const osc = this.ctx.createOscillator();
        const gain = this.ctx.createGain();
        osc.frequency.setValueAtTime(100, t);
        osc.frequency.exponentialRampToValueAtTime(10, t + 0.5);
        gain.gain.setValueAtTime(0.15, t);
        gain.gain.exponentialRampToValueAtTime(0.01, t + 0.5);
        osc.connect(gain);
        gain.connect(this.ctx.destination);
        osc.start();
        osc.stop(t + 0.5);
    },
    playPowerup() { 
        this.playTone(400, 'sine', 0.1, 0.1); 
        setTimeout(() => this.playTone(600, 'sine', 0.2, 0.1), 100);
    },
    playAlert() {
        this.playTone(200, 'sawtooth', 0.5, 0.2);
        setTimeout(() => this.playTone(150, 'sawtooth', 0.5, 0.2), 400);
    },
    playJackpot() {
        if (!this.ctx) return;
        const t = this.ctx.currentTime;
        const osc = this.ctx.createOscillator();
        const gain = this.ctx.createGain();
        osc.type = 'square';
        osc.frequency.setValueAtTime(220, t);
        osc.frequency.linearRampToValueAtTime(880, t + 1.5);
        gain.gain.setValueAtTime(0.1, t);
        gain.gain.linearRampToValueAtTime(0, t + 2.0);
        osc.connect(gain);
        gain.connect(this.ctx.destination);
        osc.start();
        osc.stop(t + 2.0);
    }
};

// --- CONFIG ---
const CONFIG = {
    MAX_PER_WEAPON: 10,
    COLORS: {
        BG: '#020005',
        GRID: '#1a102e',
        PLAYER: '#00ffff',
        XP: '#00ffaa', 
        TEXT_DMG: '#ffffff',
        TEXT_CRIT: '#ffd700',
        CARD_BG: '#0a0a2a',
    }
};

// --- DATA ---
const CHARACTERS = {
    glitch: { name: "The Glitch", hp: 60, speed: 2.2, weapon: 'PISTOL', color: '#00ffff' },
    heavy: { name: "Heavy Byte", hp: 100, speed: 1.5, weapon: 'SHOTGUN', color: '#00ff00' },
    runner: { name: "Neon Runner", hp: 40, speed: 3.0, weapon: 'SMG', color: '#ff00ff' }
};

const WEAPONS = {
    PISTOL: { name: "Plasma Pistol", damage: 25, rate: 40, count: 1, spread: 0.05, speed: 8, color: '#fff' },
    SMG: { name: "Micro SMG", damage: 12, rate: 10, count: 1, spread: 0.2, speed: 10, color: '#ff00ff' },
    SHOTGUN: { name: "Combat Shotgun", damage: 18, rate: 70, count: 4, spread: 0.4, speed: 8, color: '#ff0000' },
    RAILGUN: { name: "Railgun", damage: 100, rate: 100, count: 1, spread: 0, speed: 20, pierce: 50, color: '#00ffff' },
    ROCKET: { name: "RPG", damage: 150, rate: 120, count: 1, spread: 0, speed: 12, explosive: true, radius: 120, color: '#ffaa00' },
    BOOMERANG: { name: "Data Disc", damage: 45, rate: 45, count: 1, spread: 0, speed: 14, return: true, pierce: 999, color: '#00ff00' },
    FLAME: { name: "Firewall", damage: 5, rate: 5, count: 1, spread: 0.3, speed: 7, life: 30, pierce: 999, color: '#ff5500' },
    SAW: { name: "Ripper", damage: 30, rate: 60, count: 1, spread: 0, speed: 5, life: 200, pierce: 999, color: '#aaaaaa' }
};

const ENEMIES = {
    SWARMER: { hp: 10, speed: 1.2, size: 14, color: '#ff00ff', score: 5, shape: 'diamond', mass: 1 },
    HUNTER: { hp: 35, speed: 0.9, size: 24, color: '#ff3333', score: 10, shape: 'triangle', mass: 2 },
    BRUTE: { hp: 120, speed: 0.6, size: 45, color: '#ff8800', score: 50, shape: 'square', mass: 5 },
    BOSS: { hp: 3000, speed: 0.7, size: 90, color: '#ffffff', score: 1000, shape: 'skull', mass: 100 }
};

function generateUpgrade(rarity, existingChoices, player) {
    let upgrade = null;
    let attempts = 0;
    const isDup = (u) => existingChoices.some(c => c.name === u.name);

    while (!upgrade || isDup(upgrade) || attempts < 20) {
        attempts++;
        
        // 30% Weapon Chance
        if (Math.random() < 0.3) {
            const weaponKeys = Object.keys(WEAPONS);
            const key = weaponKeys[Math.floor(Math.random() * weaponKeys.length)];
            const currentCount = player.weapons.filter(w => w.key === key).length;
            if (currentCount < CONFIG.MAX_PER_WEAPON) {
                upgrade = {
                    id: 'w_' + key + Date.now(),
                    name: `WEAPON: ${WEAPONS[key].name}`,
                    desc: `Add Weapon (${currentCount + 1}/${CONFIG.MAX_PER_WEAPON})`,
                    rarity: rarity,
                    apply: (p) => p.equipWeapon(key)
                };
            }
        } 
        
        // Relics (New Passive Items)
        if (!upgrade && Math.random() < 0.15) {
            const relics = [
                { name: "MOM'S KNIFE", desc: "Melee range orbital blade", apply: p => p.addSpecialOrbital('KNIFE') },
                { name: "HOLY WATER", desc: "Leave damaging trail", apply: p => p.trail = true },
                { name: "STOPWATCH", desc: "Slows enemies by 25%", apply: p => p.enemySpeedMult *= 0.75 },
                { name: "THE HALO", desc: "Blocks one hit (Rechargeable)", apply: p => p.addShield() },
                { name: "BLOODY TEAR", desc: "Projectiles seek targets", apply: p => p.modifiers.homingChance += 0.2 },
                { name: "ATTRACTORB", desc: "+100% Magnet Range", apply: p => p.magnetRange += 100 }
            ];
            const r = relics[Math.floor(Math.random()*relics.length)];
            upgrade = { id: 'relic_'+Date.now(), name: r.name, desc: r.desc, rarity: 1, apply: r.apply };
        }

        if (!upgrade && Math.random() < 0.2) {
            // Chance Based Modifiers
            const mods = [
                { name: "GAMBLER'S ROUNDS", desc: "20% Chance for 4x Damage", apply: p => p.modifiers.critChance += 0.2 },
                { name: "SPLITTER", desc: "15% Chance to split bullets", apply: p => p.modifiers.splitChance += 0.15 },
                { name: "BOUNCING BETTY", desc: "25% Chance to ricochet", apply: p => p.modifiers.ricochetChance += 0.25 },
                { name: "EXECUTIONER", desc: "10% Chance to insta-kill non-bosses", apply: p => p.modifiers.executeChance += 0.1 }
            ];
            const m = mods[Math.floor(Math.random()*mods.length)];
            upgrade = { id: 'mod_' + Date.now(), name: m.name, desc: m.desc, rarity: rarity, apply: m.apply };
        } 
        
        if (!upgrade) {
            // Stats
            const types = [
                { id: 'dmg', lbl: 'DAMAGE', stat: 'damageMult', base: 10, range: 15 },
                { id: 'rate', lbl: 'FIRE RATE', stat: 'rateMult', base: 5, range: 10, inverse: true }, 
                { id: 'size', lbl: 'SIZE', stat: 'projSizeMult', base: 10, range: 20 },
                { id: 'speed', lbl: 'SPEED', stat: 'speed', base: 5, range: 10 },
                { id: 'hp', lbl: 'MAX HP', stat: 'maxHp', base: 10, range: 30, direct: true }, 
                { id: 'luck', lbl: 'LUCK', stat: 'luck', base: 5, range: 15, direct: true }
            ];
            const type = types[Math.floor(Math.random() * types.length)];
            const mult = rarity === 2 ? 3 : (rarity === 1 ? 2 : 1);
            const val = Math.floor((type.base + Math.random() * type.range) * mult);
            
            let desc = '';
            let apply = null;

            if (type.inverse) {
                desc = `-${val}% Cooldown`;
                apply = (p) => p[type.stat] *= (1 - (val/100));
            } else if (type.direct) {
                desc = `+${val} ${type.lbl}`;
                apply = (p) => { 
                    p[type.stat] += val; 
                    if(type.id === 'hp') p.hp += val; 
                };
            } else {
                desc = `+${val}% ${type.lbl}`;
                apply = (p) => p[type.stat] *= (1 + (val/100));
            }
            
            upgrade = {
                id: type.id + Date.now(),
                name: `${type.lbl} PROTOCOL`,
                desc: desc,
                rarity: rarity,
                apply: apply
            };
        }
        
        if (!isDup(upgrade)) break;
    }
    // Fallback if null
    if (!upgrade) upgrade = { id: 'fallback', name: 'REPAIR', desc: 'Full Heal', rarity: 0, apply: p => p.hp = p.maxHp };
    return upgrade;
}

// --- CLASSES ---

class Projectile {
    constructor(x, y, angle, stats, ownerStats) {
        this.x = x; this.y = y;
        this.speed = stats.speed; 
        this.vx = Math.cos(angle) * this.speed;
        this.vy = Math.sin(angle) * this.speed;
        this.color = stats.color;
        
        // Crit & Modifiers
        const isCrit = Math.random() < (0.05 + (ownerStats.luck * 0.01));
        const isGambler = Math.random() < ownerStats.modifiers.critChance; // 4x dmg
        
        let dmg = stats.damage * ownerStats.damageMult;
        if (isCrit) dmg *= ownerStats.critDmgMult;
        if (isGambler) dmg *= 4.0;

        this.damage = dmg;
        this.isCrit = isCrit || isGambler;
        this.life = stats.life || 100;
        if (stats.name === 'Data Disc') this.life = 150;
        
        this.maxLife = this.life;
        this.penetration = (stats.pierce || 1) + ownerStats.basePierce;
        this.return = stats.return || false;
        this.returning = false;
        this.owner = ownerStats; 
        
        // Chance Modifiers
        this.ricochet = ownerStats.ricochet || (Math.random() < ownerStats.modifiers.ricochetChance);
        this.split = Math.random() < ownerStats.modifiers.splitChance;
        this.execute = Math.random() < ownerStats.modifiers.executeChance;
        this.homing = Math.random() < ownerStats.modifiers.homingChance;
        
        const baseW = stats.name === 'Railgun' ? 40 : (stats.name === 'RPG' ? 20 : 10);
        const baseH = stats.name === 'Railgun' ? 10 : (stats.name === 'RPG' ? 12 : 10);
        this.width = baseW * ownerStats.projSizeMult;
        this.height = baseH * ownerStats.projSizeMult;
        
        this.angle = angle;
        this.explosive = ownerStats.explosive || stats.explosive || false;
        this.radius = stats.radius || 60;
        this.chain = ownerStats.chainLightning;
        this.corrosive = ownerStats.corrosive;
        this.hitList = [];
    }
    
    update(bounds, enemies) {
        // Homing Logic
        if (this.homing && enemies.length > 0) {
            // Find closest
            let closest = null;
            let minDist = 300; // Search range
            for (let e of enemies) {
                const d = Math.sqrt((e.x-this.x)**2 + (e.y-this.y)**2);
                if (d < minDist) { minDist = d; closest = e; }
            }
            if (closest) {
                const targetAngle = Math.atan2(closest.y - this.y, closest.x - this.x);
                // Lerp angle
                const currentAngle = Math.atan2(this.vy, this.vx);
                let diff = targetAngle - currentAngle;
                // Normalize
                while (diff <= -Math.PI) diff += Math.PI*2;
                while (diff > Math.PI) diff -= Math.PI*2;
                
                const newAngle = currentAngle + diff * 0.1; // Turn speed
                this.vx = Math.cos(newAngle) * this.speed;
                this.vy = Math.sin(newAngle) * this.speed;
                this.angle = newAngle;
            }
        }

        if (this.return) {
            if (!this.returning && this.life < this.maxLife * 0.7) this.returning = true;
            if (this.returning) {
                const dx = this.owner.x - this.x;
                const dy = this.owner.y - this.y;
                const dist = Math.sqrt(dx*dx + dy*dy);
                if (dist < 20) { this.life = 0; return; }
                this.vx += (dx/dist) * 1.5;
                this.vy += (dy/dist) * 1.5;
                const s = Math.sqrt(this.vx*this.vx+this.vy*this.vy);
                if(s>this.speed*1.5) { this.vx=(this.vx/s)*this.speed*1.5; this.vy=(this.vy/s)*this.speed*1.5; }
                this.x += this.vx; this.y += this.vy;
                this.angle += 0.5;
                return;
            }
        }
        
        this.x += this.vx; this.y += this.vy;

        if (this.ricochet) {
            if (this.x <= 0 || this.x >= bounds.width) {
                this.vx *= -1;
                this.x = Math.max(1, Math.min(bounds.width-1, this.x));
                this.angle = Math.atan2(this.vy, this.vx);
            }
            if (this.y <= 0 || this.y >= bounds.height) {
                this.vy *= -1;
                this.y = Math.max(1, Math.min(bounds.height-1, this.y));
                this.angle = Math.atan2(this.vy, this.vx);
            }
        }
        this.life--;
    }
    
    draw(ctx) {
        ctx.save();
        ctx.translate(this.x, this.y);
        ctx.rotate(this.angle);
        ctx.fillStyle = this.isCrit ? '#ffd700' : this.color;
        ctx.shadowBlur = this.isCrit ? 20 : 10;
        ctx.shadowColor = this.color;
        ctx.fillRect(-this.width/2, -this.height/2, this.width, this.height);
        ctx.restore();
    }
}

class Enemy {
    constructor(x, y, typeConf, difficulty) {
        this.x = x; this.y = y;
        this.type = typeConf;
        this.hp = typeConf.hp * difficulty;
        this.maxHp = this.hp;
        this.speed = typeConf.speed + (Math.random() * 0.1);
        this.size = typeConf.size;
        this.color = typeConf.color;
        this.score = typeConf.score;
        this.mass = typeConf.mass; // Knockback resistance
        this.angle = 0;
        this.flash = 0;
        this.rot = 0;
        this.pushX = 0;
        this.pushY = 0;
        this.poisonStacks = 0;
    }
    update(player) {
        const dx = player.x - this.x;
        const dy = player.y - this.y;
        this.angle = Math.atan2(dy, dx);
        
        const globalSpeedMult = player.enemySpeedMult || 1.0;
        this.x += Math.cos(this.angle) * this.speed * globalSpeedMult + this.pushX;
        this.y += Math.sin(this.angle) * this.speed * globalSpeedMult + this.pushY;
        
        this.pushX *= 0.9; this.pushY *= 0.9;
        this.rot += 0.05;
        if(this.flash > 0) this.flash--;

        if (this.poisonStacks > 0 && Math.random() < 0.1) {
            this.hp -= this.poisonStacks;
            this.flash = 1;
        }
    }
    draw(ctx) {
        ctx.save();
        ctx.translate(this.x, this.y);
        ctx.rotate(this.rot);
        
        if (this.flash > 0) {
            ctx.fillStyle = '#fff';
            ctx.shadowBlur = 30; ctx.shadowColor = '#fff';
        } else {
            ctx.fillStyle = '#000';
            ctx.strokeStyle = this.poisonStacks > 0 ? '#00ff00' : this.color;
            ctx.lineWidth = 3;
            ctx.shadowBlur = 10;
            ctx.shadowColor = this.color;
        }

        ctx.beginPath();
        if (this.type.shape === 'diamond') {
            ctx.moveTo(0, -this.size/2); ctx.lineTo(this.size/2, 0); ctx.lineTo(0, this.size/2); ctx.lineTo(-this.size/2, 0);
        } else if (this.type.shape === 'square') {
            ctx.rect(-this.size/2, -this.size/2, this.size, this.size);
        } else if (this.type.shape === 'skull') {
            ctx.arc(0, -10, this.size/2, 0, Math.PI*2);
            ctx.rect(-20, 10, 40, 30);
        } else {
            ctx.moveTo(this.size/2, 0); ctx.lineTo(-this.size/2, this.size/2); ctx.lineTo(-this.size/2, -this.size/2);
        }
        ctx.closePath();
        
        if (this.flash > 0) ctx.fill(); else { ctx.stroke(); ctx.fill(); }

        if (this.type.shape === 'skull') {
            ctx.fillStyle = 'red';
            ctx.fillRect(-40, -60, 80 * (Math.max(0,this.hp)/this.maxHp), 10);
        }
        ctx.restore();
    }
}

class Player {
    constructor(x, y, charKey) {
        SaveSystem.init();
        const stats = CHARACTERS[charKey];
        const meta = SaveSystem.data.upgrades;
        
        this.x = x; this.y = y;
        this.size = 24;
        this.speed = stats.speed;
        
        const baseDmgMult = 1 + (meta.dmg * 0.1);
        const bonusHp = meta.hp * 10;
        
        this.hp = stats.hp + bonusHp;
        this.maxHp = stats.hp + bonusHp;
        this.color = stats.color;
        
        this.weapons = [];
        this.equipWeapon(stats.weapon);
        
        this.damageMult = 1.0 * baseDmgMult;
        this.rateMult = 1.0;
        this.critDmgMult = 2.0;
        this.projSizeMult = 1.0;
        this.extraProj = 0;
        this.basePierce = 0;
        this.magnetRange = 100;
        this.luck = meta.luck * 5; 
        this.xpMult = 1 + (meta.greed * 0.1);
        this.vampirism = 0;
        this.ricochet = false;
        this.enemySpeedMult = 1.0; 
        
        // Modifiers for chance-based effects
        this.modifiers = {
            critChance: 0, // Gambler
            splitChance: 0,
            ricochetChance: 0,
            executeChance: 0,
            homingChance: 0
        };
        
        // Relics
        this.trail = false;
        this.trailTimer = 0;
        this.shield = 0;
        this.shieldMax = 0;
        this.shieldTimer = 0;
        
        this.orbitals = [];
        this.specialOrbitals = []; // Knife etc
        this.rearShot = false;
        this.sideShot = false;
        this.explosive = false;
        this.chainLightning = false;
        this.corrosive = false;
        this.execute = false;
        
        this.xp = 0;
        this.level = 1;
        this.nextLevel = 5;
        this.feverTime = 0;
        this.rot = 0;
    }

    fireBullet(angle, projectiles, weaponConfig) {
        projectiles.push(new Projectile(this.x, this.y, angle, weaponConfig, this));
    }

    aimAndFire(mouseX, mouseY, projectiles) {
        // Weapons now fire relative to aim, distributed if multiple
        const aimAngle = Math.atan2(mouseY - this.y, mouseX - this.x);
        
        this.weapons.forEach(w => {
            if (w.cooldown > 0) {
                w.cooldown--;
                return;
            }

            let currentRate = w.config.rate * this.rateMult;
            if (this.feverTime > 0) currentRate *= 0.2;

            AudioEngine.playShoot(w.key);

            // Use the weapon's fixed offset
            const finalAngle = aimAngle + w.offset;

            const count = w.config.count + this.extraProj;
            for(let i=0; i<count; i++) {
                const spreadAngle = (i - (count-1)/2) * 0.12 + (Math.random()-0.5) * w.config.spread;
                this.fireBullet(finalAngle + spreadAngle, projectiles, w.config);
            }
            if (this.rearShot) this.fireBullet(finalAngle + Math.PI, projectiles, w.config);
            if (this.sideShot) {
                this.fireBullet(finalAngle + Math.PI/2, projectiles, w.config);
                this.fireBullet(finalAngle - Math.PI/2, projectiles, w.config);
            }

            w.cooldown = currentRate;
        });
    }

    update(keys, bounds, particles) {
        if (this.feverTime > 0) this.feverTime--;
        
        let dx = 0, dy = 0;
        if(keys['w']) dy--; if(keys['s']) dy++;
        if(keys['a']) dx--; if(keys['d']) dx++;
        
        if (dx!==0 || dy!==0) {
            const len = Math.sqrt(dx*dx + dy*dy);
            this.x += (dx/len) * this.speed;
            this.y += (dy/len) * this.speed;
        }

        this.x = Math.max(15, Math.min(bounds.width-15, this.x));
        this.y = Math.max(15, Math.min(bounds.height-15, this.y));
        this.rot += 0.05;
        this.orbitals.forEach(o => o.update());
        this.specialOrbitals.forEach(o => o.update());
        
        // Trail Logic
        if (this.trail) {
            this.trailTimer--;
            if (this.trailTimer <= 0) {
                this.trailTimer = 5;
                particles.push({ x: this.x, y: this.y, vx: 0, vy: 0, life: 100, size: 10, color: '#00ffaa', damage: 10 });
            }
        }
        
        // Shield Recharge
        if (this.shield < this.shieldMax) {
            this.shieldTimer++;
            if (this.shieldTimer > 900) { // 15 sec
                this.shield++;
                this.shieldTimer = 0;
                AudioEngine.playPowerup();
            }
        }
    }

    draw(ctx) {
        ctx.save();
        ctx.translate(this.x, this.y);
        
        if(this.feverTime > 0) {
            ctx.beginPath();
            ctx.arc(0, 0, 40 + Math.sin(Date.now()/50)*5, 0, Math.PI*2);
            ctx.strokeStyle = '#ffd700'; ctx.lineWidth = 3; ctx.stroke();
        }
        
        // Shield Visual
        if (this.shield > 0) {
            ctx.beginPath();
            ctx.arc(0, 0, 32, 0, Math.PI*2);
            ctx.strokeStyle = '#ffffff'; ctx.lineWidth = 2; ctx.setLineDash([5, 5]);
            ctx.stroke();
            ctx.setLineDash([]);
        }

        ctx.rotate(this.rot);
        ctx.fillStyle = '#000'; ctx.strokeStyle = this.color; ctx.lineWidth = 3;
        ctx.shadowBlur = 15; ctx.shadowColor = this.color;
        
        ctx.strokeRect(-12, -12, 24, 24);
        ctx.beginPath();
        ctx.moveTo(0, -12); ctx.lineTo(12, 0); ctx.lineTo(0, 12); ctx.lineTo(-12, 0);
        ctx.fill(); ctx.stroke();

        ctx.restore();
        this.orbitals.forEach(o => o.draw(ctx));
        this.specialOrbitals.forEach(o => o.draw(ctx));

        // Health Bar
        if (this.hp < this.maxHp) {
            const hpPct = Math.max(0, this.hp / this.maxHp);
            ctx.fillStyle = 'red';
            ctx.fillRect(this.x - 20, this.y - 35, 40, 5);
            ctx.fillStyle = '#00ff00';
            ctx.fillRect(this.x - 20, this.y - 35, 40 * hpPct, 5);
        }
    }

    equipWeapon(key) {
        // Recalculate all offsets when a weapon is added
        // We just push the new weapon, then iterate
        this.weapons.push({
            key: key,
            config: WEAPONS[key],
            cooldown: 0,
            offset: 0 // placeholder
        });
        
        const count = this.weapons.length;
        const step = (Math.PI * 2) / (count || 1);
        this.weapons.forEach((w, i) => w.offset = i * step);
    }

    addOrbital() {
        const count = this.orbitals.length;
        this.orbitals.push(new Orbital(this, (Math.PI*2 / (count+1)) * count));
        this.orbitals.forEach((o, i) => o.offset = (Math.PI*2 / this.orbitals.length) * i);
    }
    
    addSpecialOrbital(type) {
        if (type === 'KNIFE') {
            this.specialOrbitals.push({
                angle: 0, dist: 40, size: 20, damage: 30, type: 'KNIFE',
                update: function() { this.angle += 0.15; },
                draw: function(ctx) { 
                    ctx.save();
                    ctx.translate(window.game.entities.player.x, window.game.entities.player.y);
                    ctx.rotate(this.angle);
                    ctx.translate(this.dist, 0);
                    ctx.rotate(Math.PI/2); // Point outward
                    ctx.fillStyle = '#ff0000';
                    ctx.beginPath(); ctx.moveTo(0, -10); ctx.lineTo(5, 10); ctx.lineTo(-5, 10); ctx.fill();
                    ctx.restore();
                }
            });
        }
    }
    
    addShield() {
        this.shieldMax++;
        this.shield++;
    }
}

class Orbital {
    constructor(player, offset) {
        this.player = player;
        this.offset = offset;
        this.angle = 0;
        this.size = 14;
        this.damage = 10 * player.damageMult;
    }
    update() {
        this.angle += 0.08;
        this.x = this.player.x + Math.cos(this.angle + this.offset) * 70;
        this.y = this.player.y + Math.sin(this.angle + this.offset) * 70;
    }
    draw(ctx) {
        ctx.beginPath();
        ctx.arc(this.x, this.y, this.size, 0, Math.PI*2);
        ctx.fillStyle = '#fff';
        ctx.shadowColor = '#00ffff';
        ctx.shadowBlur = 10;
        ctx.fill();
    }
}

class Item {
    constructor(x, y, type, value) {
        this.x = x; this.y = y;
        this.type = type;
        this.value = value;
        this.radius = type === 'XP' ? 8 : (type === 'CHEST' ? 15 : 14);
        this.magnetized = false;
        this.vx = (Math.random() - 0.5) * 6;
        this.vy = (Math.random() - 0.5) * 6;
        this.angle = Math.random();
    }
    update(player) {
        this.angle += 0.1;
        this.x += this.vx; this.y += this.vy;
        this.vx *= 0.9; this.vy *= 0.9;

        const dx = player.x - this.x;
        const dy = player.y - this.y;
        const dist = Math.sqrt(dx*dx + dy*dy);
        const range = player.magnetRange + (player.feverTime > 0 ? 500 : 0);

        if (dist < range || this.magnetized) {
            this.magnetized = true;
            this.x += (dx * 0.15);
            this.y += (dy * 0.15);
        }
        return dist < (player.size + this.radius);
    }
    draw(ctx) {
        ctx.save();
        ctx.translate(this.x, this.y);
        if (this.type === 'XP') {
            ctx.rotate(this.angle);
            ctx.fillStyle = CONFIG.COLORS.XP;
            ctx.shadowBlur = 10; ctx.shadowColor = CONFIG.COLORS.XP;
            ctx.beginPath(); ctx.moveTo(0, -8); ctx.lineTo(8, 0); ctx.lineTo(0, 8); ctx.lineTo(-8, 0); ctx.fill();
        } else if (this.type === 'CHEST') {
            ctx.rotate(this.angle * 0.5);
            ctx.fillStyle = '#ffd700';
            ctx.shadowBlur = 20; ctx.shadowColor = '#ffd700';
            ctx.fillRect(-10, -10, 20, 20);
            ctx.strokeStyle = '#fff'; ctx.lineWidth = 2; ctx.strokeRect(-10, -10, 20, 20);
        } else {
            ctx.shadowBlur = 15;
            if (this.type === 'MAGNET') { ctx.fillStyle = '#00ffff'; ctx.fillText("🧲", -10, 5); }
            else if (this.type === 'NUKE') { ctx.fillStyle = '#00ff00'; ctx.fillText("☢️", -10, 5); }
            else if (this.type === 'HEALTH') { ctx.fillStyle = '#ff0000'; ctx.fillText("❤️", -10, 5); }
        }
        ctx.restore();
    }
}

// --- SLOT MACHINE ---
class SlotMachine {
    constructor() {
        this.active = false;
        this.phase = 'SPIN';
        this.reels = [0,0,0];
        this.timer = 0;
        this.choices = [];
        this.rarity = 0;
    }
    start(luck) {
        this.active = true;
        this.phase = 'SPIN';
        this.timer = 40;
        this.reels = [0,0,0];
        this.choices = [];
        this.luck = luck;
    }
    update() {
        if (!this.active) return;
        if (this.phase === 'SPIN') {
            this.timer--;
            if(this.timer % 5 === 0) {
                this.reels = this.reels.map(() => Math.floor(Math.random()*3));
                AudioEngine.playTone(800 + Math.random()*200, 'square', 0.05);
            }
            if(this.timer <= 0) this.determineOutcome();
        }
    }
    determineOutcome() {
        const r = Math.random() * 100;
        const legendaryChance = 5 + (this.luck * 0.1);
        const rareChance = 30 + (this.luck * 0.2);

        if (r < legendaryChance) { this.rarity = 2; this.reels=[2,2,2]; AudioEngine.playJackpot(); }
        else if (r < rareChance) { this.rarity = 1; this.reels=[1,1,1]; AudioEngine.playPowerup(); }
        else { this.rarity = 0; this.reels=[0,0,0]; AudioEngine.playPowerup(); }
        
        while(this.choices.length < 3) {
            this.choices.push(generateUpgrade(this.rarity, this.choices, window.game.entities.player));
        }
        this.phase = 'SELECT';
    }
    draw(ctx, w, h) {
        if (!this.active) return;
        ctx.fillStyle = 'rgba(0,0,0,0.95)';
        ctx.fillRect(0,0,w,h);
        
        const cx = w/2, cy = h/2;
        const colors = ['#00ffff', '#ff00ff', '#ffd700'];
        const syms = ['🍒','⚡','7️⃣'];
        const cardColors = ['#00ffff', '#ff00ff', '#ffd700', '#aa00ff']; 
        
        ctx.font = '80px Arial';
        ctx.textAlign = 'center';
        this.reels.forEach((r, i) => {
            ctx.fillStyle = colors[r];
            ctx.shadowColor = colors[r];
            ctx.shadowBlur = 30;
            ctx.fillText(syms[r], cx - 100 + i*100, cy - 180);
        });

        if (this.phase === 'SELECT') {
            ctx.fillStyle = '#fff';
            ctx.font = '30px Courier New';
            ctx.shadowBlur = 0;
            ctx.fillText("SELECT REWARD", cx, cy - 80);

            const cardW = 240, cardH = 320, gap = 30;
            const startX = (w - (3*cardW + 2*gap))/2;
            const startY = cy - 30;

            this.choices.forEach((c, i) => {
                const x = startX + i*(cardW+gap);
                const color = cardColors[c.rarity] || '#00ffff';
                
                ctx.fillStyle = CONFIG.COLORS.CARD_BG;
                ctx.fillRect(x, startY, cardW, cardH);
                ctx.strokeStyle = color;
                ctx.lineWidth = 3;
                ctx.strokeRect(x, startY, cardW, cardH);
                
                ctx.fillStyle = color;
                ctx.font = 'bold 22px Courier New';
                ctx.fillText(c.name, x+cardW/2, startY+50);
                
                ctx.fillStyle = '#ccc';
                ctx.font = '16px Courier New';
                this.wrapText(ctx, c.desc, x+cardW/2, startY+120, 200, 24);
                
                const typeText = c.rarity === 3 ? "CORRUPTED" : (c.rarity === 2 ? "LEGENDARY" : (c.rarity === 1 ? "RARE" : "COMMON"));
                ctx.font = '14px Courier New';
                ctx.fillStyle = color;
                ctx.fillText(typeText, x+cardW/2, startY+cardH-30);
            });
        }
    }
    wrapText(ctx, text, x, y, maxWidth, lineHeight) {
        const words = text.split(' ');
        let line = '';
        for(let n = 0; n < words.length; n++) {
            const testLine = line + words[n] + ' ';
            const metrics = ctx.measureText(testLine);
            if (metrics.width > maxWidth && n > 0) {
                ctx.fillText(line, x, y);
                line = words[n] + ' ';
                y += lineHeight;
            } else line = testLine;
        }
        ctx.fillText(line, x, y);
    }
}

// --- GAME ---
class Game {
    constructor() {
        this.canvas = document.getElementById('gameCanvas');
        this.ctx = this.canvas.getContext('2d');
        this.resize();
        
        SaveSystem.init();

        this.keys = {};
        this.mouse = { x: 0, y: 0 };
        this.state = 'MENU';
        
        this.entities = {
            player: null,
            enemies: [],
            projectiles: [],
            particles: [],
            items: [],
            texts: []
        };
        
        this.slotMachine = new SlotMachine();
        this.score = 0;
        this.earnings = 0; 
        this.frameCount = 0;
        this.shake = 0;
        this.difficulty = 1;
        this.spawnDebt = 0; 

        window.addEventListener('keydown', e => this.keys[e.key] = true);
        window.addEventListener('keyup', e => this.keys[e.key] = false);
        window.addEventListener('mousemove', e => {
            const rect = this.canvas.getBoundingClientRect();
            this.mouse.x = e.clientX - rect.left;
            this.mouse.y = e.clientY - rect.top;
        });
        window.addEventListener('resize', () => this.resize());
        this.canvas.addEventListener('mousedown', e => this.handleClick(e));

        document.getElementById('start-btn').onclick = () => { AudioEngine.init(); this.ui('start'); };
        document.getElementById('shop-btn').onclick = () => { this.ui('shop'); };

        requestAnimationFrame(t => this.loop(t));
    }

    resize() {
        this.width = window.innerWidth;
        this.height = window.innerHeight;
        this.canvas.width = this.width;
        this.canvas.height = this.height;
    }

    openShop() {
        document.getElementById('start-screen').style.display = 'none';
        document.getElementById('shop-screen').style.display = 'flex';
        this.renderShop();
    }

    closeShop() {
        document.getElementById('shop-screen').style.display = 'none';
        document.getElementById('start-screen').style.display = 'flex';
    }

    renderShop() {
        const container = document.getElementById('shop-container');
        container.innerHTML = '';
        META_UPGRADES.forEach(u => {
            const currentLevel = SaveSystem.data.upgrades[u.id] || 0;
            const cost = Math.floor(u.cost * Math.pow(u.scale, currentLevel));
            const div = document.createElement('div');
            div.className = 'shop-item';
            div.innerHTML = `
                <h3>${u.name} (Lvl ${currentLevel})</h3>
                <p>${u.desc}</p>
                <div class="shop-cost">${cost} SHARDS</div>
                <button class="btn" style="padding: 5px 20px; font-size: 16px; margin-top:5px; min-width:auto;">BUY</button>
            `;
            const btn = div.querySelector('button');
            if (currentLevel >= u.max) {
                btn.innerText = "MAXED";
                btn.disabled = true;
                btn.style.opacity = 0.5;
            } else if (SaveSystem.data.shards >= cost) {
                btn.onclick = () => {
                    SaveSystem.data.shards -= cost;
                    SaveSystem.data.upgrades[u.id]++;
                    SaveSystem.save();
                    this.renderShop();
                };
            } else {
                btn.disabled = true;
                btn.style.opacity = 0.5;
            }
            container.appendChild(div);
        });
    }

    selectChar(key) {
        document.getElementById('char-select-screen').style.display = 'none';
        this.start(key);
    }

    start(charKey) {
        this.entities.player = new Player(this.width/2, this.height/2, charKey);
        this.entities.enemies = [];
        this.entities.projectiles = [];
        this.entities.particles = [];
        this.entities.items = [];
        this.entities.texts = [];
        this.score = 0;
        this.earnings = 0;
        this.frameCount = 0;
        this.difficulty = 1;
        this.spawnDebt = 0;
        this.state = 'PLAYING';
        this.updateUI();
    }

    ui(screen) {
        document.getElementById('start-screen').style.display = 'none';
        document.getElementById('shop-screen').style.display = 'none';
        document.getElementById('char-select-screen').style.display = 'none';
        if(screen === 'start') document.getElementById('char-select-screen').style.display = 'flex';
        if(screen === 'shop') document.getElementById('shop-screen').style.display = 'flex';
        if(screen === 'menu') document.getElementById('start-screen').style.display = 'flex';
    }

    handleClick(e) {
        if (this.state === 'LEVEL_UP' && this.slotMachine.phase === 'SELECT') {
            const rect = this.canvas.getBoundingClientRect();
            const mx = e.clientX - rect.left;
            const my = e.clientY - rect.top;
            const cardW = 240, gap = 30;
            const startX = (this.width - (3*cardW + 2*gap))/2;
            const startY = this.height/2 - 30;

            this.slotMachine.choices.forEach((c, i) => {
                const x = startX + i*(cardW+gap);
                if (mx >= x && mx <= x+cardW && my >= startY && my <= startY+320) {
                    c.apply(this.entities.player);
                    
                    // Check for rapid level up (XP Overflow)
                    if (this.entities.player.xp >= this.entities.player.nextLevel) {
                        this.entities.player.xp -= this.entities.player.nextLevel;
                        this.entities.player.nextLevel = Math.floor(this.entities.player.nextLevel * 1.5);
                        this.entities.player.level++;
                        // Restart spin immediately
                        this.slotMachine.start(this.entities.player.luck);
                    } else {
                        this.state = 'PLAYING';
                        this.slotMachine.active = false;
                    }
                }
            });
        }
        if (this.state === 'GAME_OVER') {
            const cx = this.width / 2;
            const cy = this.height / 2;
            const rect = this.canvas.getBoundingClientRect();
            const mx = e.clientX - rect.left;
            const my = e.clientY - rect.top;
            
            // Retry
            if (mx > cx - 125 && mx < cx + 125 && my > cy + 40 && my < cy + 90) {
                this.selectChar(this.entities.player.charKey || 'glitch'); 
                this.ui('start');
            }
            
            // Change Char
            if (mx > cx - 125 && mx < cx + 125 && my > cy + 100 && my < cy + 150) {
                 this.ui('start');
            }
            
            // Shop
            if (mx > cx - 125 && mx < cx + 125 && my > cy + 160 && my < cy + 210) {
                 this.ui('shop');
            }
        }
    }

    spawnEnemy() {
        if (this.entities.enemies.length > 3000) return; 

        const side = Math.floor(Math.random() * 4);
        let x, y;
        const pad = 100;
        if (side === 0) { x = Math.random()*this.width; y = -pad; }
        else if (side === 1) { x = this.width+pad; y = Math.random()*this.height; }
        else if (side === 2) { x = Math.random()*this.width; y = this.height+pad; }
        else { x = -pad; y = Math.random()*this.height; }

        let type = ENEMIES.SWARMER;
        const rand = Math.random();
        
        if (this.frameCount > 10000 && this.frameCount % 10000 < 120 && !this.entities.enemies.some(e => e.type === ENEMIES.BOSS)) {
            type = ENEMIES.BOSS;
        } 
        else if (this.frameCount > 6000 && rand < 0.1) type = ENEMIES.BRUTE;
        else if (this.frameCount > 2000 && rand < 0.2) type = ENEMIES.HUNTER;
        
        this.entities.enemies.push(new Enemy(x, y, type, this.difficulty));
    }

    addParticles(x, y, color, count) {
        for(let i=0; i<count; i++) {
            this.entities.particles.push({
                x, y, color, 
                vx: (Math.random()-0.5)*8, vy: (Math.random()-0.5)*8,
                life: 40, size: Math.random()*4+2
            });
        }
    }

    update() {
        if (this.state !== 'PLAYING') {
            if (this.state === 'LEVEL_UP') this.slotMachine.update();
            return;
        }

        const { player, enemies, projectiles, items, particles } = this.entities;

        // --- LEVEL UP LOGIC MOVED OUTSIDE ITEM LOOP ---
        while (player.xp >= player.nextLevel && this.state === 'PLAYING') {
            player.xp -= player.nextLevel;
            player.nextLevel = Math.floor(player.nextLevel * 1.5);
            player.level++;
            this.state = 'LEVEL_UP';
            this.slotMachine.start(player.luck);
            return; // Exit update immediately to handle UI
        }

        this.frameCount++;
        
        const timeMins = this.frameCount / 3600; 
        this.difficulty = 1 + (timeMins * 0.5); 
        const spawnsPerSec = 1.5 * Math.pow(2.2, timeMins);
        this.spawnDebt += spawnsPerSec / 60;

        const swarmInterval = 3600;
        const isSwarming = this.frameCount > swarmInterval && (this.frameCount % swarmInterval) < 600;
        
        if (isSwarming) {
            this.spawnDebt += 0.5; 
            document.getElementById('swarm-overlay').style.display = 'block';
            if (this.frameCount % 60 === 0) AudioEngine.playAlert();
        } else {
            document.getElementById('swarm-overlay').style.display = 'none';
        }
        
        const bossTime = 10000;
        if (this.frameCount > bossTime - 300 && this.frameCount < bossTime) {
            document.getElementById('warning-overlay').style.display = 'block';
            if (this.frameCount % 60 === 0) AudioEngine.playAlert();
        } else document.getElementById('warning-overlay').style.display = 'none';

        let loops = 0;
        while (this.spawnDebt >= 1 && loops < 20) {
            this.spawnEnemy();
            this.spawnDebt--;
            loops++;
        }

        player.update(this.keys, {width: this.width, height: this.height}, particles);
        player.aimAndFire(this.mouse.x, this.mouse.y, projectiles);

        projectiles.forEach(p => p.update({width: this.width, height: this.height}, enemies));

        for(let i=particles.length-1; i>=0; i--) {
            let p = particles[i];
            if (p.damage) { // Damaging trail
                // Simple AABB check
                enemies.forEach(e => {
                    if (Math.abs(e.x - p.x) < 20 && Math.abs(e.y - p.y) < 20) {
                        e.hp -= p.damage; e.flash = 2;
                    }
                });
            }
            p.x += p.vx; p.y += p.vy;
            p.life--;
            if(p.life <= 0) particles.splice(i,1);
        }
        
        for (let i = enemies.length-1; i>=0; i--) {
            const e = enemies[i];
            e.update(player); 

            const distSq = (e.x-player.x)**2 + (e.y-player.y)**2;
            const hitDist = (e.size/2 + player.size/2)**2;
            
            if (distSq < hitDist) {
                if (player.shield > 0) {
                    player.shield--;
                    e.pushX = -(ex/dist)*30; e.pushY = -(ey/dist)*30;
                    AudioEngine.playPowerup();
                } else if (player.feverTime > 0) { 
                    e.hp = 0; 
                } else {
                    player.hp -= 5;
                    this.shake = 8;
                    AudioEngine.playHit();
                    const angle = Math.atan2(player.y - e.y, player.x - e.x);
                    player.x += Math.cos(angle) * 20;
                    player.y += Math.sin(angle) * 20;
                    if(player.hp <= 0) { 
                        this.state = 'GAME_OVER'; 
                        AudioEngine.playExplosion();
                        this.earnings = Math.floor(this.score / 5);
                        SaveSystem.addShards(this.earnings);
                        this.earnings = earnings;
                    }
                }
                this.updateUI();
            }

            player.orbitals.forEach(o => {
                if ((e.x-o.x)**2 + (e.y-o.y)**2 < (e.size+o.size)**2) {
                    e.hp -= o.damage; e.flash = 2;
                }
            });
            player.specialOrbitals.forEach(o => {
               // Simple distance check for knife
               const ox = player.x + Math.cos(o.angle)*o.dist;
               const oy = player.y + Math.sin(o.angle)*o.dist;
               if(Math.sqrt((e.x-ox)**2 + (e.y-oy)**2) < e.size+o.size) e.hp -= o.damage;
            });
            
            this.entities.particles.forEach(pt => {
                if(pt.damage && Math.abs(e.x-pt.x)<20 && Math.abs(e.y-pt.y)<20) e.hp -= pt.damage;
            });
        }

        for(let p of this.entities.projectiles) {
            const targets = enemies; // Using direct loop for restored version
            for(let e of targets) {
                if(!e.hp > 0) continue;
                const d = (e.x-p.x)**2 + (e.y-p.y)**2;
                if(d < (e.size+p.width)**2) {
                    // Check hit list
                    if (p.hitList.includes(e)) continue;
                    p.hitList.push(e);

                    e.hp -= p.damage;
                    e.pushX = (p.vx/p.speed)*(2/(e.mass||1));
                    e.pushY = (p.vy/p.speed)*(2/(e.mass||1));
                    p.pierce--;
                    
                    // Effects
                    if (p.explosive) {
                        AudioEngine.playExplosion(); this.shake = 4;
                        for (let k of enemies) {
                             if (Math.abs(k.x - p.x) < 80 && Math.abs(k.y - p.y) < 80) {
                                 k.hp -= p.damage * 0.5; k.flash = 2;
                             }
                        }
                        this.addParticles(p.x, p.y, '#ffaa00', 8);
                        p.life = 0; 
                    } else if (p.chain) {
                        for (let k of enemies) {
                            if (k !== e && Math.abs(k.x - e.x) < 150 && Math.abs(k.y - e.y) < 150) {
                                k.hp -= p.damage * 0.5; k.flash = 2;
                                this.ctx.strokeStyle = '#00ffff';
                                this.ctx.beginPath(); this.ctx.moveTo(e.x, e.y); this.ctx.lineTo(k.x, k.y); this.ctx.stroke();
                                break; 
                            }
                        }
                    } else if (p.corrosive) {
                        e.poisonStacks += 1;
                    }
                    else {
                        AudioEngine.playHit();
                        this.addParticles(e.x, e.y, e.color, 2);
                    }

                    if(p.penetration <= 0) {
                         p.life = 0; 
                         break;
                    }

                    if(p.isCrit) e.hp -= p.damage;
                    
                    if (player.execute && !p.execute && e.hp < e.maxHp * 0.2 && e.type !== ENEMIES.BOSS) { 
                        e.hp = 0; 
                    }
                    else if (p.execute && e.type !== ENEMIES.BOSS && Math.random() < 0.1) {
                        e.hp = 0; // Executioner mod
                    }
                    
                    if (player.vampirism > 0 && Math.random() < player.vampirism) {
                        player.hp = Math.min(player.maxHp, player.hp + 2);
                        this.updateUI();
                    }

                    this.entities.texts.push({
                        x: e.x, y: e.y, text: Math.floor(p.damage), life: 30, 
                        color: p.isCrit ? CONFIG.COLORS.TEXT_CRIT : CONFIG.COLORS.TEXT_DMG
                    });

                    if(e.hp <= 0) {
                         // Check already dead to avoid double drop?
                         // Handled by loop order or removal below
                    }
                }
            }
        }

        // Cleanup Dead Enemies & Drop Loot
        for(let i=this.entities.enemies.length-1; i>=0; i--) {
            let e = this.entities.enemies[i];
            if(e.hp <= 0) {
                this.score += e.score;
                this.addParticles(e.x, e.y, e.color, 5);
                
                // Loot Logic
                if (e.type.shape === 'skull') { // Boss Loot
                    items.push(new Item(e.x, e.y, 'JACKPOT'));
                    items.push(new Item(e.x + 20, e.y, 'HEALTH', 100));
                    for(let k=0; k<10; k++) items.push(new Item(e.x + Math.random()*40-20, e.y + Math.random()*40-20, 'XP', 10));
                } else {
                    const luckFactor = player.luck * 0.001;
                    if (Math.random() < 0.0005 + luckFactor) items.push(new Item(e.x, e.y, 'NUKE'));
                    else if (Math.random() < 0.0005 + luckFactor) items.push(new Item(e.x, e.y, 'MAGNET'));
                    else if (Math.random() < 0.001 + luckFactor) items.push(new Item(e.x, e.y, 'HEALTH', 50));
                    else items.push(new Item(e.x, e.y, 'XP', 1));
                }
                
                this.entities.enemies.splice(i, 1);
            }
        }

        for(let i=items.length-1; i>=0; i--) {
            if(items[i].update(player)) {
                const it = items[i];
                items.splice(i, 1);
                
                if (it.type === 'XP' || it.type === 'JACKPOT') {
                    const val = it.type === 'JACKPOT' ? 500 : 1;
                    player.xp += val * player.xpMult;
                    AudioEngine.playHit(); 
                } else if (it.type === 'CHEST') {
                    AudioEngine.playJackpot();
                    player.xp += 1000;
                    player.hp = player.maxHp;
                    this.score += 5000;
                    this.shake = 10;
                } else if (it.type === 'NUKE') {
                    AudioEngine.playExplosion();
                    // Nuke turns enemies to gems instead of deleting
                    this.entities.enemies.forEach(e => {
                        items.push(new Item(e.x, e.y, 'XP', 1));
                        this.addParticles(e.x, e.y, '#ffaa00', 5);
                    });
                    this.entities.enemies = [];
                    this.shake = 20;
                } else if (it.type === 'MAGNET') {
                    AudioEngine.playPowerup();
                    player.magnetRange = 3000;
                    setTimeout(() => player.magnetRange = 100, 3000);
                } else if (it.type === 'HEALTH') {
                    AudioEngine.playPowerup();
                    player.hp = Math.min(player.maxHp, player.hp + it.value);
                }
                this.updateUI();
            }
        }
        
        if(this.shake > 0) this.shake *= 0.9;
        if(this.shake < 0.5) this.shake = 0;
    }

    draw() {
        const ctx = this.ctx;
        ctx.fillStyle = CONFIG.COLORS.BG;
        ctx.fillRect(0, 0, this.width, this.height);
        
        ctx.save();
        ctx.translate((Math.random()-0.5)*this.shake, (Math.random()-0.5)*this.shake);
        
        ctx.strokeStyle = CONFIG.COLORS.GRID;
        ctx.lineWidth = 1;
        ctx.beginPath();
        const size=60;
        const offset = (this.frameCount * 0.5) % size;
        for(let x=0; x<this.width; x+=size) { ctx.moveTo(x,0); ctx.lineTo(x,this.height); }
        for(let y=offset-size; y<this.height; y+=size) { ctx.moveTo(0,y); ctx.lineTo(this.width,y); }
        ctx.stroke();

        this.entities.items.forEach(i => i.draw(ctx));
        this.entities.particles.forEach(p => {
            ctx.fillStyle = p.color;
            ctx.fillRect(p.x, p.y, p.size, p.size);
        });
        this.entities.enemies.forEach(e => e.draw(ctx));
        if(this.entities.player) this.entities.player.draw(ctx);
        this.entities.projectiles.forEach(p => p.draw(ctx));
        
        this.entities.texts.forEach(t => {
            ctx.fillStyle = t.color;
            ctx.font = 'bold 20px Courier New';
            ctx.fillText(t.text, t.x, t.y);
            t.y--;
        });

        if (this.state === 'PLAYING') {
            ctx.strokeStyle = '#00ffff';
            ctx.lineWidth = 2;
            ctx.beginPath();
            ctx.arc(this.mouse.x, this.mouse.y, 12, 0, Math.PI*2);
            ctx.moveTo(this.mouse.x-18, this.mouse.y); ctx.lineTo(this.mouse.x+18, this.mouse.y);
            ctx.moveTo(this.mouse.x, this.mouse.y-18); ctx.lineTo(this.mouse.x, this.mouse.y+18);
            ctx.stroke();
        }

        if (this.state === 'LEVEL_UP') this.slotMachine.draw(ctx, this.width, this.height);
        
        if (this.state === 'GAME_OVER') {
            ctx.fillStyle = 'rgba(0,0,0,0.9)';
            ctx.fillRect(0,0,this.width,this.height);
            
            ctx.fillStyle = '#ff0000';
            ctx.font = '80px Courier New';
            ctx.textAlign = 'center';
            ctx.fillText("CRITICAL FAILURE", this.width/2, this.height/2 - 80);
            
            ctx.font = '40px Courier New';
            ctx.fillStyle = '#ffd700';
            ctx.fillText(`+${this.earnings} DATA SHARDS`, this.width/2, this.height/2 - 20);
            
            // Buttons visual
            ctx.strokeStyle = '#00ffff';
            ctx.strokeRect(this.width/2 - 125, this.height/2 + 40, 250, 50);
            ctx.fillStyle = '#00ffff';
            ctx.font = '24px Courier New';
            ctx.fillText("RETRY", this.width/2, this.height/2 + 72);
            
            ctx.strokeRect(this.width/2 - 125, this.height/2 + 100, 250, 50);
            ctx.fillText("CHANGE HOST", this.width/2, this.height/2 + 132);
            
            ctx.strokeStyle = '#ff00ff';
            ctx.fillStyle = '#ff00ff';
            ctx.strokeRect(this.width/2 - 125, this.height/2 + 160, 250, 50);
            ctx.fillText("BLACK MARKET", this.width/2, this.height/2 + 192);
        }

        ctx.restore();
    }
    
    updateUI() {
        if(!this.entities.player) return;
        document.getElementById('hp-display').innerText = Math.floor(this.entities.player.hp);
        document.getElementById('weapon-display').innerText = this.entities.player.weapons.length;
        const xpP = (this.entities.player.xp / this.entities.player.nextLevel)*100;
        document.getElementById('xp-bar').style.width = Math.min(100, xpP) + "%";
        document.getElementById('score-display').innerText = this.score;
        document.getElementById('luck-display').innerText = Math.floor(this.entities.player.luck);
        document.getElementById('level-display').innerText = this.entities.player.level;
    }

    loop(t) {
        this.update();
        this.draw();
        requestAnimationFrame(tm => this.loop(tm));
    }
    
    closeShop() {
        document.getElementById('shop-screen').style.display = 'none';
        document.getElementById('start-screen').style.display = 'flex';
    }
}

// MAKE GAME GLOBAL
window.game = new Game();

</script>
</body>
</html>
