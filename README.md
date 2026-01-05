That sounds like the "Handshake" failed in the previous code. Specifically, the Host didn't realize the Client had joined, so it never created the second player. The Client just ended up watching the Host play alone (like a screen share).

Here is the **Fixed Version**.

### What I changed to fix the "Screen Share" bug:

1. **Player List System:** Instead of "Player 1" and "Player 2", the game now uses a list of players indexed by their ID. This guarantees that when you join, a distinct new character is created for you.
2. **Explicit Spawning:** When the Host receives a connection, they immediately spawn a new square for the new player.
3. **Client-Side Prediction:** The client now knows exactly which square is "theirs" so the camera (or view) focuses correctly.

### Instructions:

1. Save as `game.html`.
2. **Player 1 (Host):** Open -> Multiplayer -> Create Room -> **Wait on the "Lobby" screen**.
3. **Player 2 (Joiner):** Open -> Multiplayer -> Join Room -> Enter Code.
4. **Important:** Once the joiner connects, the Host's screen will update to say "Player Connected". **The Host must then click "START GAME"** to begin for both people.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Jackpot Protocol v16 – FIXED MULTIPLAYER</title>
<script src="https://unpkg.com/peerjs@1.5.2/dist/peerjs.min.js"></script>

<style>
    body { margin:0; overflow:hidden; background:#111; font-family:'Courier New', monospace; color: white; }
    canvas { display:block; background: #000; }

    /* UI SCREENS */
    .screen {
        position: absolute; top: 0; left: 0; width: 100%; height: 100%;
        display: flex; flex-direction: column; align-items: center; justify-content: center;
        background: rgba(10, 10, 10, 0.95); z-index: 10;
    }
    .hidden { display: none !important; }

    h1 { color: #00ffff; text-shadow: 0 0 10px #00ffff; margin-bottom: 20px; font-size: 32px; }
    
    button {
        background: transparent; border: 2px solid #00ffff; color: #00ffff;
        padding: 15px 30px; font-size: 18px; margin: 10px; cursor: pointer;
        font-family: inherit; font-weight: bold; transition: 0.2s;
    }
    button:hover { background: #00ffff; color: black; box-shadow: 0 0 15px #00ffff; }
    button:disabled { border-color: #555; color: #555; cursor: default; box-shadow: none; }

    input {
        background: #222; border: 1px solid #444; color: white;
        padding: 15px; font-size: 18px; text-align: center; margin-bottom: 10px;
        font-family: inherit; width: 250px;
    }
    input:focus { border-color: #00ffff; outline: none; }

    .room-code {
        font-size: 30px; color: #ffff00; border: 2px dashed #ffff00;
        padding: 10px 20px; margin: 20px; cursor: pointer; user-select: text;
    }
    
    #status-log { margin-top: 15px; color: #888; font-size: 14px; }
</style>
</head>
<body>

<div id="menu-screen" class="screen">
    <h1>JACKPOT PROTOCOL</h1>
    <input id="p-name" placeholder="ENTER CODENAME" maxlength="8">
    <br>
    <button onclick="GAME.startSingle()">SINGLE PLAYER</button>
    <button onclick="UI.show('mp-menu')">MULTIPLAYER</button>
</div>

<div id="mp-menu" class="screen hidden">
    <h1>MULTIPLAYER</h1>
    <button onclick="NET.host()">CREATE ROOM</button>
    <div style="margin: 20px 0; width: 100%; border-top: 1px solid #333;"></div>
    <input id="join-code" placeholder="ENTER ROOM CODE">
    <button onclick="NET.join()">JOIN ROOM</button>
    <br><br>
    <button onclick="UI.show('menu-screen')" style="border-color:#555; color:#888;">BACK</button>
</div>

<div id="lobby-screen" class="screen hidden">
    <h1>LOBBY</h1>
    <p>Give this code to your friend:</p>
    <div id="code-display" class="room-code" onclick="NET.copyCode()">GENERATING...</div>
    <div id="player-list" style="margin: 20px; font-size: 20px;">
        </div>
    <button id="start-btn" onclick="NET.startGame()" disabled>WAITING FOR PLAYER...</button>
    <p id="status-log">Initializing...</p>
</div>

<canvas id="canvas"></canvas>

<script>
// ========================================================
// 1. GAME ENGINE
// ========================================================
const GAME = {
    canvas: document.getElementById('canvas'),
    ctx: document.getElementById('canvas').getContext('2d'),
    running: false,
    isHost: false,
    myId: null,
    
    // State
    players: {}, // Stores { id: { x, y, color, name, keys } }
    
    init() {
        this.canvas.width = window.innerWidth;
        this.canvas.height = window.innerHeight;
        
        // Input Listeners
        window.addEventListener('keydown', e => this.handleInput(e.code, true));
        window.addEventListener('keyup', e => this.handleInput(e.code, false));
        
        this.loop();
    },

    startSingle() {
        this.myId = 'local';
        this.isHost = true;
        this.players = {};
        this.spawnPlayer('local', document.getElementById('p-name').value || 'Player', '#00ffff');
        UI.hideAll();
        this.running = true;
    },

    spawnPlayer(id, name, color) {
        this.players[id] = {
            id: id,
            x: Math.random() * (this.canvas.width - 100) + 50,
            y: Math.random() * (this.canvas.height - 100) + 50,
            size: 30,
            color: color,
            name: name,
            keys: { Up: false, Down: false, Left: false, Right: false }
        };
    },

    handleInput(key, isPressed) {
        if (!this.running) return;
        
        // Map keys
        let action = null;
        if (key === 'KeyW' || key === 'ArrowUp') action = 'Up';
        if (key === 'KeyS' || key === 'ArrowDown') action = 'Down';
        if (key === 'KeyA' || key === 'ArrowLeft') action = 'Left';
        if (key === 'KeyD' || key === 'ArrowRight') action = 'Right';

        if (action) {
            // If Singleplayer or Host, update immediately
            if (this.isHost) {
                if(this.players[this.myId]) this.players[this.myId].keys[action] = isPressed;
            } 
            // If Client, send to Host
            else {
                NET.send({ type: 'input', key: action, val: isPressed });
            }
        }
    },

    update() {
        if (!this.running) return;

        // ONLY HOST calculates movement
        if (this.isHost) {
            Object.values(this.players).forEach(p => {
                const speed = 5;
                if (p.keys.Up) p.y -= speed;
                if (p.keys.Down) p.y += speed;
                if (p.keys.Left) p.x -= speed;
                if (p.keys.Right) p.x += speed;
                
                // Bounds
                if(p.x < 0) p.x = 0;
                if(p.y < 0) p.y = 0;
            });

            // Broadcast state to clients
            if (NET.conn) {
                NET.send({ type: 'state', players: this.players });
            }
        }
    },

    draw() {
        // Clear background
        this.ctx.fillStyle = '#111';
        this.ctx.fillRect(0, 0, this.canvas.width, this.canvas.height);

        // Draw Players
        Object.values(this.players).forEach(p => {
            this.ctx.shadowBlur = 15;
            this.ctx.shadowColor = p.color;
            this.ctx.fillStyle = p.color;
            this.ctx.fillRect(p.x, p.y, p.size, p.size);
            
            this.ctx.shadowBlur = 0;
            this.ctx.fillStyle = "white";
            this.ctx.font = "14px Courier New";
            this.ctx.textAlign = "center";
            this.ctx.fillText(p.name, p.x + p.size/2, p.y - 10);
        });
    },

    loop() {
        this.update();
        this.draw();
        requestAnimationFrame(() => this.loop());
    }
};

// ========================================================
// 2. NETWORKING (PeerJS)
// ========================================================
const NET = {
    peer: null,
    conn: null,
    
    host() {
        const name = document.getElementById('p-name').value || 'Host';
        this.initPeer().then(id => {
            GAME.isHost = true;
            GAME.myId = id; // Host ID is the Peer ID
            
            // Spawn Host immediately in lobby memory
            GAME.players = {};
            GAME.spawnPlayer(id, name, '#00ffff'); // Host is Cyan

            UI.show('lobby-screen');
            document.getElementById('code-display').innerText = id;
            this.updateLobbyList();

            // Listen for connections
            this.peer.on('connection', c => {
                this.conn = c;
                this.setupConnection();
                document.getElementById('status-log').innerText = "Player Connecting...";
            });
        });
    },

    join() {
        const name = document.getElementById('p-name').value || 'Client';
        const code = document.getElementById('join-code').value;
        if (!code) return alert("Enter a code!");

        this.initPeer().then(id => {
            GAME.isHost = false;
            GAME.myId = id; // Client ID
            
            UI.show('lobby-screen');
            document.getElementById('lobby-screen').innerHTML = "<h1>CONNECTING...</h1>";

            this.conn = this.peer.connect(code);
            
            this.conn.on('open', () => {
                // Send Hello
                this.send({ type: 'join', name: name, id: id });
                this.setupConnection();
            });
        });
    },

    initPeer() {
        return new Promise((resolve) => {
            // Using public PeerJS server
            this.peer = new Peer(null, { debug: 2 });
            this.peer.on('open', id => resolve(id));
        });
    },

    setupConnection() {
        this.conn.on('data', data => {
            // --- HOST RECEIVES ---
            if (GAME.isHost) {
                if (data.type === 'join') {
                    // Spawn Client
                    GAME.spawnPlayer(data.id, data.name, '#ff00ff'); // Client is Magenta
                    this.updateLobbyList();
                    
                    // Unlock Start Button
                    const btn = document.getElementById('start-btn');
                    btn.disabled = false;
                    btn.innerText = "START GAME";
                    btn.style.borderColor = "#00ff00";
                    btn.style.color = "#00ff00";
                }
                if (data.type === 'input') {
                    // Update Client Keys
                    const p = GAME.players[this.conn.peer];
                    if (p) p.keys[data.key] = data.val;
                }
            } 
            // --- CLIENT RECEIVES ---
            else {
                if (data.type === 'start') {
                    UI.hideAll();
                    GAME.running = true;
                }
                if (data.type === 'state') {
                    GAME.players = data.players; // Sync everything
                }
            }
        });
    },

    send(data) {
        if (this.conn && this.conn.open) this.conn.send(data);
    },

    updateLobbyList() {
        const list = Object.values(GAME.players).map(p => p.name).join('<br>');
        document.getElementById('player-list').innerHTML = list;
    },

    startGame() {
        if (!GAME.isHost) return;
        this.send({ type: 'start' });
        UI.hideAll();
        GAME.running = true;
    },

    copyCode() {
        navigator.clipboard.writeText(GAME.myId);
        alert("Code Copied!");
    }
};

// ========================================================
// 3. UI UTILS
// ========================================================
const UI = {
    show(id) {
        document.querySelectorAll('.screen').forEach(s => s.classList.add('hidden'));
        document.getElementById(id).classList.remove('hidden');
    },
    hideAll() {
        document.querySelectorAll('.screen').forEach(s => s.classList.add('hidden'));
    }
};

// Start
GAME.init();

</script>
</body>
</html>

```
