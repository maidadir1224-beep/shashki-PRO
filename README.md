<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Шашки PRO</title>
<style>
*{box-sizing:border-box}
body{margin:0;min-height:100vh;text-align:center;color:white;font-family:Arial,sans-serif;background:radial-gradient(circle at top,#35435b,#10131d 75%)}
h1{color:#f4ce83;font-size:30px;letter-spacing:2px;text-shadow:0 0 20px #d79a45;margin:18px 0 5px}
.sub{color:#aab6c9;margin-bottom:15px}
.panel{width:min(94%,500px);margin:12px auto;padding:13px;background:#ffffff10;border:1px solid #ffffff20;border-radius:15px}
.row{display:flex;justify-content:center;gap:8px;flex-wrap:wrap;margin:9px 0}
button{border:1px solid #ffffff30;border-radius:10px;padding:11px 15px;background:#303c50;color:white;font-size:14px;font-weight:bold;cursor:pointer}
button:disabled{opacity:.45}
button.active{background:linear-gradient(135deg,#f4c16d,#bb792e);color:#23180c}
input{padding:11px;border-radius:8px;border:1px solid #77849a;background:#f4f4f4;width:min(95%,360px);margin:4px}
#status{font-size:18px;padding:12px;color:#ffe0a0}

/* Доска всегда квадратная и не меняет размер при ходе */
#board{
 width:min(94vw,560px);
 height:min(94vw,560px);
 min-width:min(94vw,560px);
 min-height:min(94vw,560px);
 max-width:560px;
 max-height:560px;
 margin:12px auto;
 padding:0;
 box-sizing:border-box;
 display:grid;
 grid-template-columns:repeat(8,minmax(0,1fr));
 grid-template-rows:repeat(8,minmax(0,1fr));
 border:6px solid #a87543;
 border-radius:8px;
 overflow:hidden;
 box-shadow:0 12px 40px #000b;
 position:relative;
 flex-shrink:0;
 contain:layout paint;
}
.cell{position:relative;display:flex;align-items:center;justify-content:center;background:#ecd4ad;cursor:pointer;min-width:0;min-height:0}
.cell.dark{background:linear-gradient(145deg,#795039,#4b2d1d)}
.cell.selected{background:#d5ad37!important;box-shadow:inset 0 0 0 4px #ffe68a}
.cell.possible:after{content:"";width:25%;height:25%;border-radius:50%;background:#62f18a;box-shadow:0 0 15px #62ff8a}
.piece{width:79%;height:79%;border-radius:50%;display:flex;align-items:center;justify-content:center;font-size:clamp(18px,5vw,34px);font-weight:bold;z-index:2}
.white{background:radial-gradient(circle at 35% 25%,#fff,#d4d2cc 65%,#999);border:4px solid white;color:#91601d;box-shadow:0 5px 8px #0009}
.black{background:radial-gradient(circle at 35% 25%,#656875,#171821 65%,#050509);border:4px solid #777b89;color:#ffd66d;box-shadow:0 5px 8px #000b}
#message{min-height:25px;color:#f4ce83;margin:10px}
.small{font-size:12px;color:#bdc7d7;line-height:1.6}
.room{background:#ffffff12;border:1px solid #ffffff25;border-radius:9px;padding:9px;margin:7px 0;text-align:left}
.room button{float:right;padding:7px 10px}
</style>
</head>
<body>
<h1>♛ ШАШКИ PRO ♛</h1>
<div class="sub">РУССКИЕ ШАШКИ</div>

<div class="panel">
 <div>РЕЖИМ ИГРЫ</div>
 <div class="row">
  <button id="botMode" class="active" onclick="setMode('bot')">🤖 Против бота</button>
  <button id="twoMode" onclick="setMode('two')">👥 Сам с собой</button>
  <button id="onlineMode" onclick="setMode('online')">🌐 Онлайн</button>
 </div>
 <div id="difficultyArea">
  <div>СЛОЖНОСТЬ БОТА</div>
  <div class="row">
   <button onclick="setLevel(1)" id="easy">Лёгкий</button>
   <button onclick="setLevel(2)" id="medium" class="active">Средний</button>
   <button onclick="setLevel(3)" id="hard">Сложный</button>
  </div>
 </div>
 <div id="status">Ход белых</div>
</div>

<div class="panel" id="onlinePanel" style="display:none">
 <h3>🌐 ОНЛАЙН-СЕРВЕРЫ</h3>
 <input id="roomName" placeholder="Название комнаты" maxlength="24"><br>
 <input id="roomPassword" type="password" placeholder="Пароль (необязательно)" maxlength="40"><br>
 <div class="row">
  <button onclick="createRoom()">➕ Создать комнату</button>
  <button onclick="refreshRooms()">🔄 Обновить список</button>
 </div>
 <div id="roomStatus" class="small">Подключение к Firebase...</div>
 <div id="roomList"></div>
 <div id="currentRoom" class="small"></div>
 <button onclick="leaveRoom()">Выйти из комнаты</button>
</div>

<div id="board"></div>
<div id="message">Выберите свою шашку</div>
<button onclick="newGame()">↻ Новая игра</button>

<div class="panel" style="text-align:left">
 <b>Правила:</b>
 <div class="small">
 • Обязательное взятие. Шашки бьют вперёд и назад.<br>
 • Дамка ходит на любое расстояние.<br>
 • Можно делать несколько взятий подряд.<br>
 • При достижении последней линии шашка становится дамкой.
 </div>
</div>

<script src="https://www.gstatic.com/firebasejs/10.14.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.14.1/firebase-auth-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.14.1/firebase-database-compat.js"></script>
<script>
const firebaseConfig = {
  apiKey: "AIzaSyD9LhyRzCSVH83PKBzNO6yVJqdSveg3yLU",
  authDomain: "sjufu-bd527.firebaseapp.com",
  databaseURL: "https://sjufu-bd527-default-rtdb.europe-west1.firebasedatabase.app",
  projectId: "sjufu-bd527",
  storageBucket: "sjufu-bd527.firebasestorage.app",
  messagingSenderId: "538229764774",
  appId: "1:538229764774:web:02ab54b5a9d3e91e6bba67"
};
firebase.initializeApp(firebaseConfig);
const db=firebase.database();
const auth=firebase.auth();

let board=[],turn="white",selected=null,moves=[],gameOver=false;
let mode="bot",level=2,botThinking=false,gameId=0;
let myUid=null,myRole=null,roomId=null,roomRef=null,roomListener=null;
let applyingRemote=false;

function clone(b){return b.map(row=>row.map(p=>p?{...p}:null))}
function inside(r,c){return r>=0&&r<8&&c>=0&&c<8}
function opponent(color){return color==="white"?"black":"white"}
function initialBoard(){
 let b=Array.from({length:8},()=>Array(8).fill(null));
 for(let r=0;r<3;r++)for(let c=0;c<8;c++)if((r+c)%2===1)b[r][c]={color:"black",king:false};
 for(let r=5;r<8;r++)for(let c=0;c<8;c++)if((r+c)%2===1)b[r][c]={color:"white",king:false};
 return b;
}
function newGame(){
 if(mode==="online"&&roomId){if(myRole==="host")sendState({board:initialBoard(),turn:"white",gameOver:false});return}
 gameId++;board=initialBoard();turn="white";selected=null;moves=[];gameOver=false;botThinking=false;
 document.getElementById("message").textContent="Выберите свою шашку";
 updateStatus();render();
}
function setMode(m){
 if(roomId)leaveRoom();
 mode=m;
 document.getElementById("botMode").classList.toggle("active",m==="bot");
 document.getElementById("twoMode").classList.toggle("active",m==="two");
 document.getElementById("onlineMode").classList.toggle("active",m==="online");
 document.getElementById("difficultyArea").style.display=m==="bot"?"block":"none";
 document.getElementById("onlinePanel").style.display=m==="online"?"block":"none";
 newGame();
}
function setLevel(n){level=n;["easy","medium","hard"].forEach((id,i)=>document.getElementById(id).classList.toggle("active",[1,2,3][i]===n));newGame()}

function getCaptures(b,r,c){
 const p=b[r][c];if(!p)return [];
 const out=[],dirs=[[-1,-1],[-1,1],[1,-1],[1,1]];
 for(const [dr,dc] of dirs){
  if(!p.king){
   const mr=r+dr,mc=c+dc,lr=r+2*dr,lc=c+2*dc;
   if(inside(lr,lc)&&b[mr][mc]&&b[mr][mc].color!==p.color&&!b[lr][lc])out.push({r:lr,c:lc,capture:{r:mr,c:mc}});
  }else{
   let nr=r+dr,nc=c+dc;
   while(inside(nr,nc)&&!b[nr][nc]){nr+=dr;nc+=dc}
   if(!inside(nr,nc)||b[nr][nc].color===p.color)continue;
   const cr=nr,cc=nc;nr+=dr;nc+=dc;
   while(inside(nr,nc)&&!b[nr][nc]){out.push({r:nr,c:nc,capture:{r:cr,c:cc}});nr+=dr;nc+=dc}
  }
 }
 return out;
}
function captureSequences(b,r,c,path=[],taken=[]){
 const opts=getCaptures(b,r,c);
 if(!opts.length)return taken.length?[{path,taken}]:[];
 let results=[];
 for(const m of opts){
  const nb=clone(b),p=nb[r][c];nb[r][c]=null;nb[m.capture.r][m.capture.c]=null;nb[m.r][m.c]=p;
  if(!p.king&&((p.color==="white"&&m.r===0)||(p.color==="black"&&m.r===7)))p.king=true;
  results.push(...captureSequences(nb,m.r,m.c,path.concat([{r:m.r,c:m.c}]),taken.concat([m.capture])));
 }
 return results;
}
function simpleMoves(b,r,c){
 const p=b[r][c],out=[];if(!p)return out;
 if(p.king){
  for(const dr of [-1,1])for(const dc of [-1,1]){let nr=r+dr,nc=c+dc;while(inside(nr,nc)&&!b[nr][nc]){out.push({from:{r,c},path:[{r:nr,c:nc}],taken:[]});nr+=dr;nc+=dc}}
 }else{
  const dr=p.color==="white"?-1:1;
  for(const dc of [-1,1]){const nr=r+dr,nc=c+dc;if(inside(nr,nc)&&!b[nr][nc])out.push({from:{r,c},path:[{r:nr,c:nc}],taken:[]})}
 }
 return out;
}
function legalTurns(b,color){
 let captureExists=false;
 for(let r=0;r<8;r++)for(let c=0;c<8;c++)if(b[r][c]?.color===color&&getCaptures(b,r,c).length)captureExists=true;
 const out=[];
 for(let r=0;r<8;r++)for(let c=0;c<8;c++)if(b[r][c]?.color===color){
  if(captureExists){
   const seq=captureSequences(b,r,c);
   for(const m of seq)out.push({...m,from:{r,c}});
  }else out.push(...simpleMoves(b,r,c));
 }
 return out;
}
function applyTurn(b,m){
 const nb=clone(b),p=nb[m.from.r][m.from.c];nb[m.from.r][m.from.c]=null;
 for(let i=0;i<m.path.length;i++){if(m.taken[i])nb[m.taken[i].r][m.taken[i].c]=null;const pos=m.path[i];if(!p.king&&((p.color==="white"&&pos.r===0)||(p.color==="black"&&pos.r===7)))p.king=true}
 const end=m.path[m.path.length-1];nb[end.r][end.c]=p;return nb;
}
function evaluate(b,color){
 let s=0;for(let r=0;r<8;r++)for(let c=0;c<8;c++){const p=b[r][c];if(!p)continue;let v=p.king?8:3;if(!p.king)v+=p.color==="black"?r*.12:(7-r)*.12;s+=p.color===color?v:-v}return s;
}
function minimax(b,depth,color,ai,alpha,beta){
 const turns=legalTurns(b,color);if(!turns.length)return color===ai?-10000-depth:10000+depth;if(depth===0)return evaluate(b,ai);
 const max=color===ai;let best=max?-Infinity:Infinity;
 turns.sort((a,b)=>b.taken.length-a.taken.length);
 for(const m of turns){const v=minimax(applyTurn(b,m),depth-1,opponent(color),ai,alpha,beta);if(max){best=Math.max(best,v);alpha=Math.max(alpha,best)}else{best=Math.min(best,v);beta=Math.min(beta,best)}if(beta<=alpha)break}
 return best;
}
function chooseBotMove(){
 const list=legalTurns(board,"black");if(!list.length)return null;
 if(level===1)return list[Math.floor(Math.random()*list.length)];
 let best=-Infinity,choices=[];for(const m of list){const v=minimax(applyTurn(board,m),level===2?2:3,"white","black",-Infinity,Infinity);if(v>best){best=v;choices=[m]}else if(v===best)choices.push(m)}
 return choices[Math.floor(Math.random()*choices.length)];
}
function updateStatus(){
 if(gameOver)return;
 let label="Ход "+(turn==="white"?"белых":"чёрных");
 if(mode==="bot"&&botThinking)label="🤖 Бот думает...";
 if(mode==="online"&&roomId)label="Онлайн: "+(turn==="white"?"белые":"чёрные")+" · Вы играете за "+(myRole==="host"?"белых":"чёрных");
 document.getElementById("status").textContent=label;
}
function finishTurn(){
 turn=opponent(turn);selected=null;moves=[];
 if(!legalTurns(board,turn).length){gameOver=true;document.getElementById("status").textContent="🏆 Победили "+(turn==="white"?"чёрные":"белые")+"!";document.getElementById("message").textContent="Игра окончена!"}
 else updateStatus();
}

/* Плавно перемещает копию шашки, не двигая саму доску */
function animatePiece(startRect,endR,endC){
 const cells=document.querySelectorAll("#board .cell");
 const destination=cells[endR*8+endC];
 const targetPiece=destination&&destination.querySelector(".piece");
 if(!targetPiece)return;
 const endRect=targetPiece.getBoundingClientRect();
 const moving=targetPiece.cloneNode(true);
 targetPiece.style.visibility="hidden";
 moving.style.position="fixed";
 moving.style.left=startRect.left+"px";
 moving.style.top=startRect.top+"px";
 moving.style.width=endRect.width+"px";
 moving.style.height=endRect.height+"px";
 moving.style.margin="0";
 moving.style.zIndex="9999";
 moving.style.pointerEvents="none";
 moving.style.transition="none";
 moving.style.transform="translate3d(0,0,0)";
 moving.style.willChange="transform";
 document.body.appendChild(moving);
 void moving.offsetWidth;
 let finished=false;
 function cleanup(){
  if(finished)return;
  finished=true;
  moving.remove();
  if(targetPiece.isConnected)targetPiece.style.visibility="";
 }
 moving.addEventListener("transitionend",cleanup,{once:true});
 requestAnimationFrame(()=>{
  moving.style.transition="transform 320ms cubic-bezier(.2,.75,.25,1)";
  moving.style.transform="translate3d("+(endRect.left-startRect.left)+"px,"+(endRect.top-startRect.top)+"px,0)";
 });
 setTimeout(cleanup,420);
}
function makeMove(m){
 const sourceCell=document.querySelectorAll("#board .cell")[m.from.r*8+m.from.c];
 const sourcePiece=sourceCell&&sourceCell.querySelector(".piece");
 const startRect=sourcePiece?sourcePiece.getBoundingClientRect():null;
 board=applyTurn(board,m);
 finishTurn();
 render();
 const end=m.path[m.path.length-1];
 if(startRect)animatePiece(startRect,end.r,end.c);
 if(mode==="online"&&roomId)sendState({board,turn,gameOver});
 if(mode==="bot"&&turn==="black"&&!gameOver)startBot();
}
function startBot(){
 botThinking=true;updateStatus();const id=gameId;
 setTimeout(()=>{if(id!==gameId||gameOver||mode!=="bot")return;botThinking=false;const m=chooseBotMove();if(m)makeMove(m)},350);
}
function clickCell(r,c){
 if(gameOver||botThinking)return;
 if(mode==="online"&&(!roomId||!myRole||(myRole==="host"&&turn!=="white")||(myRole==="guest"&&turn!=="black")))return;
 if(selected){
  const m=moves.find(x=>x.path.length===1&&x.path[0].r===r&&x.path[0].c===c);
  if(m){makeMove(m);selected=null;moves=[];return}
  const full=moves.find(x=>x.path[x.path.length-1].r===r&&x.path[x.path.length-1].c===c);
  if(full){makeMove(full);selected=null;moves=[];return}
 }
 if(board[r][c]&&board[r][c].color===turn){
  const opts=legalTurns(board,turn).filter(m=>m.from.r===r&&m.from.c===c);
  if(!opts.length){document.getElementById("message").textContent="Нет доступных ходов или обязательно нужно бить";return}
  selected={r,c};moves=opts;document.getElementById("message").textContent=opts.some(m=>m.taken.length)?"Выберите конечную клетку взятия":"Выберите ход";
 }else{selected=null;moves=[]}
 render();
}
function render(){
 const el=document.getElementById("board");el.innerHTML="";
 for(let r=0;r<8;r++)for(let c=0;c<8;c++){
  const cell=document.createElement("div");cell.className="cell "+((r+c)%2?"dark":"");
  if(selected&&selected.r===r&&selected.c===c)cell.classList.add("selected");
  if(moves.some(m=>{const p=m.path[m.path.length-1];return p.r===r&&p.c===c}))cell.classList.add("possible");
  const p=board[r]?.[c];if(p){const piece=document.createElement("div");piece.className="piece "+p.color;if(p.king)piece.textContent="♛";cell.appendChild(piece)}
  cell.onclick=()=>clickCell(r,c);el.appendChild(cell);
 }
}
function setMessage(t){document.getElementById("roomStatus").textContent=t}
async function passwordHash(value){
 if(!value)return "";
 const bytes=new TextEncoder().encode(value);
 const digest=await crypto.subtle.digest("SHA-256",bytes);
 return Array.from(new Uint8Array(digest)).map(x=>x.toString(16).padStart(2,"0")).join("");
}
auth.signInAnonymously().then(cred=>{
 myUid=cred.user.uid;setMessage("Firebase подключён. Можно создавать комнаты.");refreshRooms();
}).catch(err=>setMessage("Ошибка входа Firebase: "+err.message));
function refreshRooms(){
 if(!myUid){setMessage("Подожди подключения...");return}
 db.ref("rooms").once("value").then(snap=>{
  const list=document.getElementById("roomList");list.innerHTML="";
  const rooms=snap.val()||{};let count=0;
  Object.entries(rooms).forEach(([id,room])=>{
   if(room.guestUid&&room.guestUid!==myUid)return;
   if(!room.hostUid)return;
   count++;const div=document.createElement("div");div.className="room";
   const title=document.createElement("b");title.textContent=room.name||"Комната";
   const info=document.createElement("span");info.className="small";info.textContent=room.guestUid?" · занята":" · свободна";
   const btn=document.createElement("button");btn.textContent="Войти";btn.onclick=()=>joinRoom(id,room.passwordHash||"");
   div.append(title,info,btn);list.appendChild(div);
  });
  if(!count)list.textContent="Пока нет свободных комнат.";
 }).catch(e=>setMessage("Не удалось загрузить комнаты: "+e.message));
}
async function createRoom(){
 if(!myUid){setMessage("Подожди подключения к Firebase.");return}
 if(roomId){setMessage("Сначала выйди из текущей комнаты.");return}
 const name=document.getElementById("roomName").value.trim()||"Комната";
 const pass=document.getElementById("roomPassword").value;
 const id=db.ref("rooms").push().key;
 const data={name,hostUid:myUid,passwordHash:await passwordHash(pass),state:{board:initialBoard(),turn:"white",gameOver:false},createdAt:Date.now()};
 try{await db.ref("rooms/"+id).set(data);connectRoom(id,"host");setMessage("Комната создана. Ожидаем второго игрока.");refreshRooms()}
 catch(e){setMessage("Ошибка создания: "+e.message)}
}
async function joinRoom(id,expectedHash){
 if(!myUid||roomId)return;
 const pass=document.getElementById("roomPassword").value;
 if(expectedHash&&await passwordHash(pass)!==expectedHash){setMessage("Неверный пароль.");return}
 const ref=db.ref("rooms/"+id);
 try{
  const result=await ref.child("guestUid").transaction(current=>current||myUid);
  if(!result.committed||result.snapshot.val()!==myUid){setMessage("Комната уже занята.");refreshRooms();return}
  connectRoom(id,"guest");setMessage("Ты подключился к комнате.");refreshRooms()
 }catch(e){setMessage("Ошибка входа: "+e.message)}
}
function findRemoteMove(oldBoard,nextBoard,nextTurn){
 const movingColor=opponent(nextTurn);
 let from=null,to=null;
 for(let r=0;r<8;r++)for(let c=0;c<8;c++){
  if(oldBoard[r]?.[c]?.color===movingColor&&!nextBoard[r]?.[c])from={r,c};
  if(nextBoard[r]?.[c]?.color===movingColor&&!oldBoard[r]?.[c])to={r,c};
 }
 return from&&to?{from,to}:null;
}
function connectRoom(id,role){
 roomId=id;myRole=role;roomRef=db.ref("rooms/"+id);
 document.getElementById("currentRoom").textContent="В комнате: "+id+" · Ваша сторона: "+(role==="host"?"белые":"чёрные");
 mode="online";document.getElementById("onlineMode").classList.add("active");document.getElementById("botMode").classList.remove("active");document.getElementById("twoMode").classList.remove("active");
 roomListener=snap=>{
  const data=snap.val();if(!data){leaveRoom(false);setMessage("Комната удалена.");return}
  if(role==="guest"&&!data.guestUid){setMessage("Ожидание подключения...");return}
  if(data.state&&data.state.board){
   const nextBoard=data.state.board;
   const remoteMove=findRemoteMove(board,nextBoard,data.state.turn);
   const sourceCell=remoteMove?document.querySelectorAll("#board .cell")[remoteMove.from.r*8+remoteMove.from.c]:null;
   const sourcePiece=sourceCell&&sourceCell.querySelector(".piece");
   const startRect=sourcePiece?sourcePiece.getBoundingClientRect():null;
   board=nextBoard;turn=data.state.turn;gameOver=!!data.state.gameOver;selected=null;moves=[];updateStatus();render();
   if(remoteMove&&startRect)animatePiece(startRect,remoteMove.to.r,remoteMove.to.c);
  }
  setMessage(role==="host"?(data.guestUid?"Игрок подключился!":"Ожидаем второго игрока..."):"Ты в комнате. Твой ход — когда ход чёрных.");
 };
 roomRef.on("value",roomListener);
}
function sendState(state){
 if(!roomRef)return;
 roomRef.child("state").set(state).catch(e=>setMessage("Ошибка отправки хода: "+e.message));
}
function leaveRoom(removeIfHost=true){
 if(roomRef&&roomListener)roomRef.off("value",roomListener);
 if(roomRef&&myRole==="host"&&removeIfHost)roomRef.remove().catch(()=>{});
 else if(roomRef&&myRole==="guest"&&removeIfHost)roomRef.child("guestUid").set("");
 roomRef=null;roomListener=null;roomId=null;myRole=null;
 document.getElementById("currentRoom").textContent="";
 if(mode==="online"){board=initialBoard();turn="white";gameOver=false;selected=null;moves=[];updateStatus();render()}
 refreshRooms();
}
newGame();
</script>
</body>
</html>
