<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Jackpot Protocol v15 – ONLINE</title>
<script src="https://unpkg.com/peerjs@1.5.2/dist/peerjs.min.js"></script>

<style>
    body { margin:0; overflow:hidden; background:#020005; font-family:'Courier New', monospace; cursor:crosshair; color: #00ffff; }
    canvas { display:block; }
    
    /* UI OVERLAYS */
    .screen-overlay { 
        position:absolute; top:0; left:0; width:100%; height:100%; 
        display:flex; flex-direction:column; justify-content:center; align-items:center; 
        background:rgba(2,0,5,0.98); z-index:10; 
    }
    
    h1 { font-size: 40px; text-shadow: 0 0 10px #00ffff; margin-bottom: 30px; }
    
    .btn { 
        padding:15px 40px; margin-top:15px; font-size:20px; 
        border:2px solid #00ffff; background:rgba(0,255,255,0.1); 
        color:#00ffff; font-weight:bold; cursor:pointer; font-family: inherit;
        transition: 0.2s;
    }
    .btn:hover { background:#00ffff; color:#000; box-shadow: 0 0 20px #00ffff; }
    
    .btn-secondary { border-color:#ff00ff; color:#ff00ff; background:rgba(255,0,255,0.1); }
    .btn-secondary:hover { background:#ff00ff; color:#000; box-shadow: 0 0 20px #ff00ff; }
    
    input { 
        padding:15px; font-size:18px; margin-top:10px; width: 300px;
        background: #111; border: 1px solid #555; color: white; text-align: center; font-family: inherit;
    }
    input:focus { border-color: #00ffff; outline: none; }

    #mp-status { margin-top: 20px; color: #ffff00; font-size: 14px; min-height: 20px;}
    .code-display { font-size: 24px; color: #00ff00; margin: 10px; user-select: text; border: 1px dashed #00ff00; padding: 10px; cursor: pointer; }
</style>
</head>

<body>

<div id="start-screen" class="screen-overlay">
    <h1>JACKPOT PROTOCOL</h1>
    <button class="btn" onclick="startGame(false)">SINGLE PLAYER</button>
    <button class="btn btn-secondary" onclick="showScreen('mp-screen')">MULTIPLAYER</button>
</div>

<div id="mp-screen" class="screen-overlay" style="display:none">
    <h1>MULTIPLAYER LOBBY</h1>
    <input id="mp-name" placeholder="ENTER YOUR CODENAME" maxlength="10">
    
    <div style="display:flex; gap: 20px; margin-top: 20px;">
        <div style="display:flex; flex-direction:column;">
            <button class="btn" onclick="MP.createRoom()">CREATE ROOM</button>
        </div>
        <div style="display:flex; flex-direction:column;">
            <input id="mp-room-code" placeholder="ENTER ROOM CODE">
            <button class="btn btn-secondary" onclick="MP.joinRoom()">JOIN ROOM</button>
        </div>
    </div>
    
    <p id="mp-status">Waiting for input...</p>
    <button class="btn" onclick="showScreen('start-screen')" style="margin-top: 50px; border-color: #555; color: #888;">BACK</button>
</div>

<div id="lobby-screen" class="screen-overlay" style="display:none">
    <h1>WAITING FOR PLAYER...</h1>
    <p>SHARE THIS CODE:</p>
    <div id="room-code-display" class="code-display" onclick="MP.copyCode()">...</div>
    <p id="lobby-msg" style="color:#aaa">Waiting for connection...</p>
</div>

<canvas id="gameCanvas"></canvas>

<script>
/* =====================================================
   GAME CONFIGURATION
===================================================== */
const CONFIG = {
    width: window.innerWidth,
    height: window.innerHeight,
    playerSpeed: 5,
    colors: { local: '#00ffff', remote: '#ff00ff' }
};

const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
canvas.width = CONFIG.width;
canvas.height = CONFIG.height;

// Handle window resize
window.onresize = () => {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
};

/* =====================================================
   MULTIPLAYER ENGINE (PeerJS)
===================================================== */
const MP = {
    peer: null,
    conn: null,
    isHost: false,
    isConnected: false,
    myId: null,
    myName: "Unknown",
    
    // Status Logger
    log: (msg) => {
        document.getElementById('mp-status').innerText = ">> " + msg;
        document.getElementById('lobby-msg').innerText = msg;
        console.log("[MP]", msg);
    },

    // 1. Initialize Peer
    init: () => {
        return new Promise((resolve, reject) => {
            MP.peer = new Peer(null, { debug: 2 });
            
            MP.peer.on('open', (id) => {
                MP.myId = id;
                console.log('My peer ID is: ' + id);
                resolve(id);
            });

            MP.peer.on('error', (err) => {
                MP.log("Error: " + err.type);
            });
        });
    },

    // 2. Create Room (Host)
    createRoom: async () => {
        const name = document.getElementById('mp-name').value || "Host";
        MP.myName = name;
        MP.isHost = true;
        MP.log("Initializing Network...");

        await MP.init();
        
        showScreen('lobby-screen');
        document.getElementById('room-code-display').innerText = MP.myId;
        MP.log("Room Created. Waiting for player...");

        // Listen for incoming connection
        MP.peer.on('connection', (c) => {
            MP.conn = c;
            MP.setupConnection();
        });
    },

    // 3. Join Room (Client)
    joinRoom: async () => {
        const name = document.getElementById('mp-name').value || "Client";
        const code = document.getElementById('mp-room-code').value.trim();
        if(!code) return MP.log("Enter a Room Code!");
        
        MP.myName = name;
        MP.isHost = false;
        MP.log("Connecting...");

        await MP.init();
        
        MP.conn = MP.peer.connect(code, {
            metadata: { name: MP.myName }
        });
        
        MP.conn.on('open', () => {
            MP.setupConnection();
            MP.log("Connected to Host!");
        });
        
        MP.conn.on('error', (err) => MP.log("Connection Failed"));
    },

    // 4. Setup Data Handling
    setupConnection: () => {
        MP.isConnected = true;
        
        // Handle incoming data
        MP.conn.on('data', (data) => {
            if(MP.isHost) {
                // HOST receives Input from Client
                if(data.type === 'input') {
                    if(window.game) window.game.handleRemoteInput(data);
                }
                // HOST receives Handshake
                if(data.type === 'handshake') {
                    if(window.game) window.game.addRemotePlayer(data.name);
                }
            } else {
                // CLIENT receives Game State from Host
                if(data.type === 'state') {
                    if(window.game) window.game.applyState(data);
                }
                // CLIENT receives Start command
                if(data.type === 'start') {
                    startGame(true);
                }
            }
        });

        // If Host, start game immediately once connected
        if(MP.isHost) {
            MP.log("Player Connected! Starting...");
            setTimeout(() => {
                startGame(true);
                // Tell client to start
                MP.send({ type: 'start' }); 
            }, 1000);
        } else {
            // If Client, send name
            MP.send({ type: 'handshake', name: MP.myName });
            MP.log("Waiting for Host to start...");
        }
    },

    send: (data) => {
        if(MP.conn && MP.conn.open) {
            MP.conn.send(data);
        }
    },

    copyCode: () => {
        navigator.clipboard.writeText(MP.myId);
        alert("Room Code Copied!");
    }
};

/* =====================================================
   GAME ENGINE (The missing part!)
===================================================== */
class Game {
    constructor(isMultiplayer) {
        this.isMultiplayer = isMultiplayer;
        this.running = true;
        
        // My Player
        this.player = {
            x: canvas.width / 2 - 50,
            y: canvas.height / 2,
            size: 30,
            color: CONFIG.colors.local,
            name: MP.myName,
            hp: 100
        };

        // Remote Player (The Other Guy)
        this.remotePlayer = null;

        // Inputs
        this.keys = {};
        window.addEventListener('keydown', e => this.keys[e.key] = true);
        window.addEventListener('keyup', e => this.keys[e.key] = false);

        // Start Loop
        this.loop();
    }

    // Called on Host when Client connects
    addRemotePlayer(name) {
        this.remotePlayer = {
            x: canvas.width / 2 + 50,
            y: canvas.height / 2,
            size: 30,
            color: CONFIG.colors.remote,
            name: name,
            keys: {} // Remote keys
        };
    }

    // Called on Host when Client sends input
    handleRemoteInput(data) {
        if(this.remotePlayer) {
            this.remotePlayer.keys = data.keys;
        }
    }

    // Called on Client when Host sends state
    applyState(state) {
        // Update my own position based on what host says (authoritative)
        // Or strictly update the other entities. 
        // For simplicity, we trust the host for EVERYTHING.
        
        this.player = state.p2; // In client view, 'p2' is me
        this.remotePlayer = state.p1; // 'p1' is host
    }

    update() {
        if(!this.running) return;

        // === HOST LOGIC ===
        if(MP.isHost || !this.isMultiplayer) {
            // Move Me
            this.moveEntity(this.player, this.keys);

            // Move Remote Player (based on inputs received)
            if(this.remotePlayer) {
                this.moveEntity(this.remotePlayer, this.remotePlayer.keys);
            }

            // Send State to Client
            if(this.isMultiplayer && MP.isConnected) {
                MP.send({
                    type: 'state',
                    p1: this.player,      // Host
                    p2: this.remotePlayer // Client
                });
            }
        } 
        
        // === CLIENT LOGIC ===
        else {
            // Send Inputs to Host
            if(MP.isConnected) {
                MP.send({
                    type: 'input',
                    keys: this.keys
                });
            }
            // Note: Client doesn't calculate movement, it just renders what Host sends
        }
    }

    moveEntity(entity, keys) {
        if(!keys) return;
        if(keys['w'] || keys['ArrowUp']) entity.y -= CONFIG.playerSpeed;
        if(keys['s'] || keys['ArrowDown']) entity.y += CONFIG.playerSpeed;
        if(keys['a'] || keys['ArrowLeft']) entity.x -= CONFIG.playerSpeed;
        if(keys['d'] || keys['ArrowRight']) entity.x += CONFIG.playerSpeed;
    }

    draw() {
        // Clear Screen
        ctx.fillStyle = 'rgba(2, 0, 5, 0.4)'; // Trails effect
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        // Draw Me
        this.drawPlayer(this.player);

        // Draw Remote
        if(this.remotePlayer) {
            this.drawPlayer(this.remotePlayer);
        }
    }

    drawPlayer(p) {
        if(!p) return;
        // Glow
        ctx.shadowBlur = 15;
        ctx.shadowColor = p.color;
        
        // Body
        ctx.fillStyle = p.color;
        ctx.fillRect(p.x, p.y, p.size, p.size);
        
        // Name
        ctx.shadowBlur = 0;
        ctx.fillStyle = "#fff";
        ctx.font = "14px Courier New";
        ctx.textAlign = "center";
        ctx.fillText(p.name, p.x + p.size/2, p.y - 10);
    }

    loop() {
        this.update();
        this.draw();
        requestAnimationFrame(() => this.loop());
    }
}

/* =====================================================
   APP FLOW
===================================================== */
function showScreen(id) {
    document.querySelectorAll('.screen-overlay').forEach(s => s.style.display = 'none');
    document.getElementById(id).style.display = 'flex';
}

function startGame(isMultiplayer) {
    document.querySelectorAll('.screen-overlay').forEach(s => s.style.display = 'none');
    
    // Create Game Instance
    if(window.game) window.game.running = false;
    window.game = new Game(isMultiplayer);
    
    // If single player, add a dummy target to shoot/interact (optional)
    if(!isMultiplayer) {
        // Just empty sandbox for now
    }
}

</script>
</body>
</html>
