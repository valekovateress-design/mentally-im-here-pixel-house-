<!DOCTYPE html>
<html lang="sk">
<head>
    <meta charset="UTF-8">
    <title>Terrys pixel home</title>
    <link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet">
    <style>
        body {
            display: flex; flex-direction: column; align-items: center;
            background-color: #1a1a1a; color: white; 
            font-family: 'Press Start 2P', cursive;
            margin: 0; padding: 20px; overflow: hidden;
        }
        canvas {
            border: 6px solid #444; background-color: #000;
            image-rendering: pixelated;
        }
        .game-btn {
            position: absolute; display: none; padding: 10px 15px;
            background: #ffcc00; border: 4px solid #000; cursor: pointer;
            color: black; z-index: 10;
            transform: translate(-50%, -140%);
            font-family: 'Press Start 2P', cursive;
            font-size: 10px;
            box-shadow: 4px 4px 0px #444;
        }
    </style>
</head>
<body>

    <div style="position: relative;">
        <canvas id="gameCanvas" width="800" height="800"></canvas>
        <button id="washBtn" class="game-btn" style="background:#00ffcc">UMYT SA</button>
        <button id="eatBtn" class="game-btn">JEST</button>
        <button id="sleepBtn" class="game-btn" style="background:#5d5dff;color:white">SPAT</button>
        <button id="wcBtn" class="game-btn" style="background:white;color:#664400">WC</button>
        <button id="wakeBtn" class="game-btn" style="background:white">ZBUDIT SA</button>
    </div>

<script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');

const imgAssets = {
    player: 'ja pixel.png',
    kuchyna: 'IMG_5288.JPG',
    spalna: 'IMG_5293.JPG',
    obyvacka: 'IMG_5289.JPG',
    zahrada: 'IMG_5294.JPG',
    kupelna: 'IMG_5286.jpg',
    kitty: 'kitty.png',
    sleepBubble: 'sleep_bubble.png'
};

const images = {};
let loadedCount = 0;
Object.keys(imgAssets).forEach(key => {
    images[key] = new Image();
    images[key].src = imgAssets[key];
    images[key].onload = () => { if (++loadedCount === Object.keys(imgAssets).length) start(); };
});

const player = {
    x: 400, y: 550, w: 60, h: 85, speed: 7,
    energy: 100, hunger: 100, hygiene: 100, bladder: 100, isSleeping: false
};
const kitty = { x: 350, y: 600, w: 40, h: 50 };

let currentRoom = 'spalna'; 
let bubbleYOffset = 0;

// Definícia dverí pre každú miestnosť
const rooms = {
    kuchyna: { bg: 'kuchyna', next: 'spalna', doors: [{x: 400, y: 750, to: 'spalna'}] },
    spalna: { 
        bg: 'spalna', 
        next: 'obyvacka', 
        doors: [
            {x: 750, y: 400, to: 'obyvacka'}, // Dvere vpravo do obývačky
            {x: 50, y: 400, to: 'kuchyna'}    // NOVÉ: Dvere vľavo do kuchyne
        ] 
    },
    obyvacka: { bg: 'obyvacka', next: 'zahrada', doors: [{x: 720, y: 550, to: 'zahrada'}] },
    zahrada: { bg: 'zahrada', next: 'kupelna', doors: [{x: 310, y: 550, to: 'kupelna'}] },
    kupelna: { bg: 'kupelna', next: 'kuchyna', doors: [{x: 700, y: 150, to: 'kuchyna'}] }
};

const keys = {};
window.addEventListener('keydown', e => keys[e.code] = true);
window.addEventListener('keyup', e => keys[e.code] = false);

document.getElementById('wakeBtn').onclick = () => player.isSleeping = false;
document.getElementById('sleepBtn').onclick = () => player.isSleeping = true;
document.getElementById('eatBtn').onclick = () => player.hunger = 100;
document.getElementById('washBtn').onclick = () => player.hygiene = 100;
document.getElementById('wcBtn').onclick = () => player.bladder = 100;

function update() {
    if (player.isSleeping) {
        if (player.energy < 100) player.energy += 0.2;
        bubbleYOffset = Math.sin(Date.now() / 300) * 10;
        updateBtnPos('wakeBtn', true);
        return; 
    }

    let moving = false;
    if (keys['ArrowUp'] && player.y > 50) { player.y -= player.speed; moving = true; }
    if (keys['ArrowDown'] && player.y < 720) { player.y += player.speed; moving = true; }
    if (keys['ArrowLeft'] && player.x > 50) { player.x -= player.speed; moving = true; }
    if (keys['ArrowRight'] && player.x < 720) { player.x += player.speed; moving = true; }

    kitty.x += (player.x - 50 - kitty.x) * 0.05;
    kitty.y += (player.y + 30 - kitty.y) * 0.05;

    if (moving) {
        player.energy -= 0.03; player.hunger -= 0.02; player.hygiene -= 0.02; player.bladder -= 0.03;
    }

    // Detekcia prechodu dverami
    const currentDoors = rooms[currentRoom].doors;
    currentDoors.forEach(door => {
        const dist = Math.sqrt(Math.pow(player.x + player.w/2 - door.x, 2) + Math.pow(player.y + player.h/2 - door.y, 2));
        if (dist < 50) {
            currentRoom = door.to;
            // Pozícia po vstupe do novej miestnosti (v strede)
            player.x = 400; 
            player.y = 400;
        }
    });

    hideAllBtns();
    if (currentRoom === 'spalna' && player.x > 300 && player.x < 500 && player.y > 250 && player.y < 500) {
        updateBtnPos('sleepBtn', true);
    }
    if (currentRoom === 'kuchyna' && player.y < 300) updateBtnPos('eatBtn', true);
    if (currentRoom === 'kupelna') {
        if (player.x < 300 && player.y > 400) updateBtnPos('washBtn', true);
        if (player.x > 500 && player.y < 400) updateBtnPos('wcBtn', true);
    }
}

function updateBtnPos(id, show) {
    const btn = document.getElementById(id);
    if (show) {
        btn.style.display = "block";
        btn.style.left = (player.x + player.w / 2) + "px";
        btn.style.top = (player.y - 10) + "px";
    }
}

function hideAllBtns() {
    ['washBtn', 'eatBtn', 'sleepBtn', 'wcBtn', 'wakeBtn'].forEach(id => {
        document.getElementById(id).style.display = "none";
    });
}

function drawStats() {
    ctx.fillStyle = "rgba(0, 0, 0, 0.7)";
    ctx.fillRect(10, 10, 260, 120);
    ctx.strokeStyle = "white";
    ctx.lineWidth = 3;
    ctx.strokeRect(10, 10, 260, 120);
    ctx.font = "10px 'Press Start 2P'";
    ctx.fillStyle = "#ff4d4d"; ctx.fillText("ENERGIA: " + Math.ceil(player.energy) + "%", 25, 35);
    ctx.fillStyle = "#ffcc00"; ctx.fillText("HLAD:    " + Math.ceil(player.hunger) + "%", 25, 60);
    ctx.fillStyle = "#00ffcc"; ctx.fillText("HYGIENA: " + Math.ceil(player.hygiene) + "%", 25, 85);
    ctx.fillStyle = "#ffff00"; ctx.fillText("POTREBA: " + Math.ceil(player.bladder) + "%", 25, 110);
}

function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    ctx.drawImage(images[rooms[currentRoom].bg], 0, 0, canvas.width, canvas.height);

    // Kreslenie dverí
    rooms[currentRoom].doors.forEach(door => {
        ctx.fillStyle = "rgba(255, 255, 255, 0.5)";
        ctx.beginPath(); ctx.arc(door.x, door.y, 20, 0, Math.PI*2); ctx.fill();
        ctx.fillStyle = "white"; 
        ctx.font = "8px 'Press Start 2P'";
        ctx.fillText("DVERE", door.x - 25, door.y - 30);
    });

    if (currentRoom === 'spalna') {
        ctx.fillStyle = "rgba(0, 100, 255, 0.2)";
        ctx.fillRect(300, 300, 200, 200); 
        ctx.fillStyle = "white"; 
        ctx.font = "8px 'Press Start 2P'";
        ctx.fillText("POSTEL", 360, 290);
    }

    ctx.drawImage(images.kitty, kitty.x, kitty.y, kitty.w, kitty.h);
    ctx.drawImage(images.player, player.x, player.y, player.w, player.h);

    if (player.isSleeping) {
        ctx.fillStyle = "rgba(0, 0, 50, 0.5)";
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        ctx.drawImage(images.sleepBubble, player.x - 10, player.y - 100 + bubbleYOffset, 80, 80);
    }

    drawStats();
}

function start() { loop(); }
function loop() { update(); draw(); requestAnimationFrame(loop); }
</script>
</body>
</html>
