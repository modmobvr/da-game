<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Jackpot Protocol v15 – CO-OP</title>

<style>
/* === ORIGINAL STYLES (UNCHANGED) === */
body{margin:0;overflow:hidden;background:#020005;font-family:'Courier New',monospace;cursor:crosshair}
canvas{display:block}
.screen-overlay{position:absolute;top:0;left:0;width:100%;height:100%;display:flex;flex-direction:column;justify-content:center;align-items:center;background:rgba(2,0,5,.97);color:#fff;z-index:10}
.btn{padding:15px 40px;margin-top:15px;font-size:20px;border:2px solid #00ffff;background:rgba(0,255,255,.1);color:#00ffff;font-weight:bold;cursor:pointer}
.btn:hover{background:#00ffff;color:#000}
.btn-secondary{border-color:#ff00ff;color:#ff00ff}
.btn-secondary:hover{background:#ff00ff;color:#000}
input{padding:10px;font-size:18px;margin-top:10px}
</style>
</head>

<body>

<!-- MULTIPLAYER MENU -->
<div id="mp-screen" class="screen-overlay" style="display:none">
  <h1>MULTIPLAYER</h1>
  <input id="mp-name" placeholder="Your Name">
  <input id="mp-room" placeholder="Room Code">
  <button class="btn" onclick="MP.create()">CREATE ROOM</button>
  <button class="btn btn-secondary" onclick="MP.join()">JOIN ROOM</button>
  <p id="mp-status"></p>
  <button class="btn btn-secondary" onclick="MP.cancel()">BACK</button>
</div>

<canvas id="gameCanvas"></canvas>

<script>
/* =====================================================
   MULTIPLAYER ENGINE (HOST-AUTHORITATIVE CO-OP)
===================================================== */
const MP={
 enabled:false,isHost:false,id:Math.random().toString(36).slice(2),
 peer:null,conn:null,room:null,players:{},
 open(){document.getElementById("mp-screen").style.display="flex"},
 cancel(){document.getElementById("mp-screen").style.display="none"},
 log(t){document.getElementById("mp-status").innerText=t},

 create(){
  this.enabled=true;this.isHost=true;
  this.room=Math.random().toString(36).slice(2,7).toUpperCase();
  this.start();this.log("ROOM: "+this.room);
 },
 join(){
  this.enabled=true;this.isHost=false;
  this.room=document.getElementById("mp-room").value.toUpperCase();
  this.start();this.log("JOINING "+this.room);
 },
 start(){
  this.peer=new RTCPeerConnection({iceServers:[{urls:"stun:stun.l.google.com:19302"}]});
  const ch=this.peer.createDataChannel("g");
  ch.onopen=()=>this.log("CONNECTED");
  ch.onmessage=e=>this.on(JSON.parse(e.data));
  this.conn=ch;
  this.peer.onicecandidate=e=>{
   if(e.candidate)localStorage.setItem(this.room+"_ice",JSON.stringify(e.candidate))
  };
  if(this.isHost){
   this.peer.createOffer().then(o=>{
    this.peer.setLocalDescription(o);
    localStorage.setItem(this.room+"_offer",JSON.stringify(o));
   });
  }else{
   const off=JSON.parse(localStorage.getItem(this.room+"_offer"));
   this.peer.setRemoteDescription(off);
   this.peer.createAnswer().then(a=>{
    this.peer.setLocalDescription(a);
    localStorage.setItem(this.room+"_answer",JSON.stringify(a));
   });
  }
  setTimeout(()=>this.finish(),1200);
 },
 finish(){
  const ice=JSON.parse(localStorage.getItem(this.room+"_ice"));
  if(ice)this.peer.addIceCandidate(ice);
  if(this.isHost){
   const ans=JSON.parse(localStorage.getItem(this.room+"_answer"));
   if(ans)this.peer.setRemoteDescription(ans);
  }
 },
 send(d){if(this.conn?.readyState==="open")this.conn.send(JSON.stringify(d))},
 on(d){
  if(d.t==="input"&&this.isHost)window.game.applyInput(d);
  if(d.t==="state"&&!this.isHost)window.game.applyState(d.s);
 }
};

/* === GAME CODE CONTINUES BELOW === */
<script>
/* =====================================================
   CO-OP INTEGRATION HOOKS
===================================================== */

// --- NETWORK-AWARE GAME EXTENSIONS ---
Game.prototype.applyInput = function (data) {
  // Called on HOST only
  if (!this.remoteInputs) this.remoteInputs = {};
  this.remoteInputs[data.id] = data;
};

Game.prototype.applyState = function (state) {
  // Called on CLIENT only
  this.entities.enemies = state.enemies;
  this.entities.projectiles = state.projectiles;
  this.entities.items = state.items;
  this.score = state.score;

  state.players.forEach(p => {
    if (p.id === MP.id) return;
    if (!this.remotePlayers) this.remotePlayers = {};
    if (!this.remotePlayers[p.id]) {
      const rp = new Player(p.x, p.y, "glitch");
      rp.isRemote = true;
      rp.id = p.id;
      rp.name = p.name;
      this.remotePlayers[p.id] = rp;
    }
    Object.assign(this.remotePlayers[p.id], p);
  });
};

// --- PATCH UPDATE LOOP ---
const originalUpdate = Game.prototype.update;
Game.prototype.update = function () {
  if (MP.enabled) {
    if (MP.isHost) {
      originalUpdate.call(this);

      // Broadcast state
      if (this.frameCount % 3 === 0) {
        MP.send({
          t: "state",
          s: {
            players: [
              {
                id: MP.id,
                x: this.entities.player.x,
                y: this.entities.player.y,
                hp: this.entities.player.hp,
                name: document.getElementById("mp-name")?.value || "HOST"
              }
            ],
            enemies: this.entities.enemies,
            projectiles: this.entities.projectiles,
            items: this.entities.items,
            score: this.score
          }
        });
      }
    } else {
      // CLIENT: send input only
      MP.send({
        t: "input",
        id: MP.id,
        keys: this.keys,
        mouse: this.mouse
      });
    }
  } else {
    originalUpdate.call(this);
  }
};

// --- DRAW REMOTE PLAYERS ---
const originalDraw = Game.prototype.draw;
Game.prototype.draw = function () {
  originalDraw.call(this);
  if (this.remotePlayers) {
    Object.values(this.remotePlayers).forEach(p => {
      p.draw(this.ctx);
      this.ctx.fillStyle = "#fff";
      this.ctx.font = "14px Courier New";
      this.ctx.fillText(p.name || "PLAYER", p.x - 20, p.y - 40);
    });
  }
};

// --- MULTIPLAYER BUTTON ---
document.addEventListener("DOMContentLoaded", () => {
  const btn = document.createElement("button");
  btn.className = "btn btn-secondary";
  btn.innerText = "MULTIPLAYER";
  btn.onclick = () => MP.open();
  document.querySelector("#start-screen")?.appendChild(btn);
});

// --- START GAME ---
window.game = new Game();
</script>

</body>
</html>
