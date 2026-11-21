
<!DOCTYPE html>
<html lang="vi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Cá Lớn Nuốt Cá Bé</title>
<style>
* { box-sizing: border-box; }
body {
  margin: 0;
  background: #001e38;
  color: white;
  font-family: Arial, sans-serif;
  text-align: center;
}
h1 { color: #ffe66d; }
#game {
  display: block;
  margin: 10px auto;
  width: min(95vw, 900px);
  height: auto;
  border: 4px solid #fff;
  border-radius: 15px;
  background: #087eae;
}
.info {
  font-size: 20px;
  margin: 10px;
}
button {
  padding: 12px 25px;
  background: #ffb703;
  border: none;
  border-radius: 10px;
  font-size: 18px;
  cursor: pointer;
}
</style>
</head>
<body>

<h1>🐠 CÁ LỚN NUỐT CÁ BÉ 🐟</h1>

<div class="info">
  Điểm: <span id="score">0</span>
  | Cấp độ: <span id="level">1</span>
</div>

<canvas id="game" width="900" height="550"></canvas>

<button onclick="restart()">Chơi lại</button>
<p>Di chuyển chuột hoặc chạm màn hình để điều khiển cá.</p>
<p>Ăn cá nhỏ để lớn lên. Tránh cá lớn hơn bạn!</p>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

let player, fishes, score, gameOver, target;

function restart() {
  score = 0;
  gameOver = false;
  target = {x:450, y:275};
  player = {x:450, y:275, size:22};
  fishes = [];

  for(let i=0; i<18; i++) spawnFish();

  updateInfo();
  requestAnimationFrame(loop);
}

function spawnFish() {
  let size = 10 + Math.random()*35;
  fishes.push({
    x: Math.random()*900,
    y: Math.random()*550,
    size: size,
    speed: 0.5 + Math.random()*1.5,
    dir: Math.random() < 0.5 ? -1 : 1,
    color: `hsl(${Math.random()*360},80%,60%)`
  });
}

function updateInfo() {
  document.getElementById("score").textContent = score;
  document.getElementById("level").textContent =
    Math.floor(player.size/22);
}

canvas.addEventListener("mousemove", e => {
  const r = canvas.getBoundingClientRect();
  target.x = (e.clientX-r.left)*canvas.width/r.width;
  target.y = (e.clientY-r.top)*canvas.height/r.height;
});

canvas.addEventListener("touchmove", e => {
  e.preventDefault();
  const r = canvas.getBoundingClientRect();
  target.x = (e.touches[0].clientX-r.left)*canvas.width/r.width;
  target.y = (e.touches[0].clientY-r.top)*canvas.height/r.height;
},{passive:false});

function drawFish(x,y,size,color,dir=1) {
  ctx.save();
  ctx.translate(x,y);
  ctx.scale(dir,1);

  ctx.fillStyle=color;
  ctx.beginPath();
  ctx.ellipse(0,0,size,size*0.65,0,0,Math.PI*2);
  ctx.fill();

  ctx.beginPath();
  ctx.moveTo(-size*0.7,0);
  ctx.lineTo(-size*1.4,-size*0.6);
  ctx.lineTo(-size*1.4,size*0.6);
  ctx.closePath();
  ctx.fill();

  ctx.fillStyle="white";
  ctx.beginPath();
  ctx.arc(size*0.45,-size*0.15,size*0.18,0,Math.PI*2);
  ctx.fill();

  ctx.fillStyle="black";
  ctx.beginPath();
  ctx.arc(size*0.5,-size*0.15,size*0.09,0,Math.PI*2);
  ctx.fill();

  ctx.restore();
}

function loop() {
  if(gameOver) return;

  ctx.clearRect(0,0,900,550);

  // Đại dương
  let bg = ctx.createLinearGradient(0,0,0,550);
  bg.addColorStop(0,"#19bce5");
  bg.addColorStop(1,"#00436f");
  ctx.fillStyle=bg;
  ctx.fillRect(0,0,900,550);

  // Bong bóng
  ctx.fillStyle="rgba(255,255,255,0.3)";
  for(let i=0;i<15;i++){
    let x=(i*83)%900;
    let y=(Date.now()/30+i*57)%550;
    ctx.beginPath();
    ctx.arc(x,y,4+(i%4),0,Math.PI*2);
    ctx.fill();
  }

  // Di chuyển cá người chơi
  player.x += (target.x-player.x)*0.06;
  player.y += (target.y-player.y)*0.06;

  player.x = Math.max(player.size,Math.min(900-player.size,player.x));
  player.y = Math.max(player.size,Math.min(550-player.size,player.y));

  // Cá khác
  for(let i=fishes.length-1;i>=0;i--){
    let f=fishes[i];
    f.x += f.speed*f.dir;

    if(f.x < -60) f.dir=1;
    if(f.x > 960) f.dir=-1;

    f.y += Math.sin(Date.now()/500+i)*0.5;

    drawFish(f.x,f.y,f.size,f.color,f.dir);

    let d=Math.hypot(player.x-f.x,player.y-f.y);

    if(d < player.size+f.size*0.7){
      if(player.size > f.size*0.9){
        score += Math.round(f.size);
        player.size += 1.5;
        fishes.splice(i,1);
        spawnFish();
        updateInfo();
      } else if(f.size > player.size*1.1){
        gameOver=true;
        ctx.fillStyle="rgba(0,0,0,0.7)";
        ctx.fillRect(0,0,900,550);
        ctx.fillStyle="white";
        ctx.font="bold 48px Arial";
        ctx.textAlign="center";
        ctx.fillText("GAME OVER!",450,250);
        ctx.font="25px Arial";
        ctx.fillText("Điểm của bạn: "+score,450,300);
        return;
      }
    }
  }

  drawFish(player.x,player.y,player.size,"#ffdf45");
  requestAnimationFrame(loop);
}

restart();
</script>
</body>
</html>
