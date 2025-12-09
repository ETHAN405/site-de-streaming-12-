# site-de-streaming-12-<!doctype html>
<html lang="fr">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Streamify — Minecraft Live Blocks 3D Light & Shadow</title>
<style>
body {
  margin:0;
  font-family:Inter,system-ui;
  background:#000;
  color:white;
  overflow:hidden;
}
.nav {
  padding:15px 30px;
  font-size:28px;
  font-weight:900;
  background:#111;
  letter-spacing:1px;
  color:#e50914;
  position:fixed;
  top:0;
  left:0;
  right:0;
  z-index:1000;
}
.container {
  display:flex;
  gap:20px;
  margin-top:60px;
  max-width:1500px;
  margin-left:auto;
  margin-right:auto;
  padding:20px;
}
.player {
  flex:1;
  border-radius:12px;
  overflow:hidden;
  position:relative;
}
video {
  width:100%;
  border-radius:12px;
  cursor:pointer;
}
.sidebar {
  width:300px;
  background:#111;
  border-radius:8px;
  padding:16px;
}
.sidebar h3 {
  margin-top:0;
  font-size:20px;
  border-bottom:1px solid #222;
  padding-bottom:8px;
}
.film {
  display:flex;
  align-items:center;
  gap:12px;
  background:#222;
  border-radius:6px;
  padding:10px;
  margin-bottom:10px;
  cursor:pointer;
}
.film:hover { background:#333; }
.film img {
  width:50px;
  height:70px;
  object-fit:cover;
  border-radius:4px;
}
.popup {
  position:fixed;
  pointer-events:none;
  z-index:9999;
  image-rendering: pixelated;
  border-radius:4px;
  transform-origin: center;
}
.particle {
  position:fixed;
  width:8px;
  height:8px;
  border-radius:2px;
  pointer-events:none;
  z-index:10000;
}
</style>
</head>
<body>

<div class="nav">STREAMIFY</div>

<div class="container">
  <div class="player">
    <video id="video" controls src="https://www.w3schools.com/html/mov_bbb.mp4"></video>
  </div>
  <aside class="sidebar">
    <h3>Épisodes</h3>
    <div class="film">
      <img src="https://i.imgur.com/0rVeh4O.jpeg" alt="Big Buck Bunny">
      Big Buck Bunny
    </div>
    <div class="film">
      <img src="https://i.imgur.com/0rVeh4O.jpeg" alt="Episode 2">
      Episode 2
    </div>
    <div class="film">
      <img src="https://i.imgur.com/0rVeh4O.jpeg" alt="Episode 3">
      Episode 3
    </div>
  </aside>
</div>

<!-- Sons par type de bloc -->
<audio id="sound1" src="https://freesound.org/data/previews/256/256113_3263906-lq.mp3"></audio>
<audio id="sound2" src="https://freesound.org/data/previews/331/331912_3248244-lq.mp3"></audio>
<audio id="sound3" src="https://freesound.org/data/previews/249/249152_4486188-lq.mp3"></audio>

<script>
const blockImages = [
  {src:'https://www.pngkit.com/png/detail/443-4430639_block-minecraft-minecraft-grass-block-transparent.png', sound:'sound1'},
  {src:'https://www.pngkit.com/png/detail/182-1827471_minecraft-grass-block-grass-block.png', sound:'sound2'},
  {src:'https://www.pikpng.com/pngl/m/586-5861478_grass-block-minecraft-grass-block-grid-clipart.png', sound:'sound3'}
];

const video = document.getElementById('video');
const blocks = [];
let interval = null;
let wind = 0;

// Vent dynamique toutes les 5 secondes
setInterval(() => {
  wind = (Math.random()*2 - 1) * 1.5;
}, 5000);

// Crée un bloc avec profondeur, parallax, rotation, vent et lumière
function createBlock(){
  const data = blockImages[Math.floor(Math.random() * blockImages.length)];
  const img = document.createElement('img');
  img.src = data.src;
  img.dataset.sound = data.sound;
  img.className = 'popup';
  
  const size = 30 + Math.random()*70;
  img.style.width = size+'px';
  img.style.height = size+'px';
  
  img.style.left = Math.random() * (window.innerWidth - size) + 'px';
  img.style.top = '-'+size+'px';
  
  img.speed = size/15 + Math.random();
  
  const maxHorizontal = (100 - size)/20;
  img.vx = (Math.random()*maxHorizontal*2) - maxHorizontal;
  
  img.rotation = Math.random()*360;
  img.vr = (Math.random()*2 - 1)*2;
  
  img.bounced = false;
  img.size = size; // pour calcul lumière/ombre
  
  document.body.appendChild(img);
  blocks.push(img);
}

// Particules
function createParticles(x, y){
  for(let i=0;i<12;i++){
    const p = document.createElement('div');
    p.className = 'particle';
    p.style.left = x + 'px';
    p.style.top = y + 'px';
    p.style.backgroundColor = ['#4CAF50','#8B4513','#A0522D'][Math.floor(Math.random()*3)];
    document.body.appendChild(p);

    const angle = Math.random()*2*Math.PI;
    const speed = 1 + Math.random()*3;
    let vx = Math.cos(angle)*speed;
    let vy = Math.sin(angle)*speed;
    let life = 0;
    const maxLife = 30 + Math.random()*20;

    function animate(){
      life++;
      p.style.left = parseFloat(p.style.left) + vx + 'px';
      p.style.top = parseFloat(p.style.top) + vy + 'px';
      vy += 0.1;
      if(life<maxLife){
        requestAnimationFrame(animate);
      } else {
        p.remove();
      }
    }
    animate();
  }
}

// Anime les blocs avec lumière/ombre et rebond
function animateBlocks(){
  for(let i=blocks.length-1;i>=0;i--){
    const b = blocks[i];
    let top = parseFloat(b.style.top);
    let left = parseFloat(b.style.left);
    
    top += b.speed;
    left += b.vx + wind * (parseFloat(b.style.width)/100);
    
    if(left < 0) left = 0;
    if(left > window.innerWidth - parseFloat(b.style.width)) left = window.innerWidth - parseFloat(b.style.width);
    
    b.rotation += b.vr;
    
    // Lumière/ombre : plus le bloc est bas, plus sombre
    const lightFactor = 1 - top/window.innerHeight; 
    b.style.filter = `brightness(${0.7 + 0.3*lightFactor}) drop-shadow(2px 4px 4px rgba(0,0,0,0.6))`;
    
    b.style.transform = `rotate(${b.rotation}deg)`;
    
    const bottomLimit = window.innerHeight - parseFloat(b.style.height);
    
    if(top >= bottomLimit){
      if(!b.bounced){
        createParticles(left + parseFloat(b.style.width)/2, bottomLimit + parseFloat(b.style.height)/2);
        const sound = document.getElementById(b.dataset.sound);
        sound.currentTime = 0;
        sound.play();
        b.bounced = true;
        b.speed = -b.speed/2;
      } else {
        if(top < bottomLimit) b.style.top = top + 'px';
        else {
          b.remove();
          blocks.splice(i,1);
        }
      }
    }
    
    b.style.top = top + 'px';
    b.style.left = left + 'px';
  }
  requestAnimationFrame(animateBlocks);
}

// Pluie continue
video.onplay = () => {
  if(!interval){
    interval = setInterval(() => {
      for(let i=0;i<12;i++) createBlock();
    }, 100);
    animateBlocks();
  }
};
</script>
</body>
</html>
