<!DOCTYPE html><html lang="pt-BR"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover"><title>Tycoon Zoo Simulator</title>
<link href="https://fonts.googleapis.com/css2?family=VT323&display=swap" rel="stylesheet">
<style>
:root{color-scheme:dark;--bg:#1f1710;--pn:#33241a;--ln:#6b4a2e;--tx:#f3e7c8;--ac:#8fd14f;--go:#f4b83a;--rd:#e5533d;box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
html{scroll-padding-top:env(safe-area-inset-top,0px)}
html,body{height:100%;margin:0;background:var(--bg);color:var(--tx);font:20px/1.1 VT323,"Courier New",monospace;user-select:none;-webkit-user-select:none;touch-action:manipulation}
#app{display:flex;flex-direction:column;height:100%}
#top{display:flex;flex-wrap:wrap;gap:2px 12px;padding:4px 8px;background:var(--pn);border-bottom:3px solid var(--ln);align-items:center}
#top b{color:var(--go);font-weight:400}
#st{padding:3px 8px;background:#26190f;color:var(--ac);min-height:1.1em;font-size:18px}
#cvb{flex:1;min-height:0;display:flex;justify-content:center;align-items:center}
canvas{image-rendering:pixelated;touch-action:none;border:3px solid var(--ln);background:#000}
#bar{display:flex;gap:4px;padding:4px;background:var(--pn);border-top:3px solid var(--ln);overflow-x:auto}
button{font:inherit;color:var(--tx);background:#4a3322;border:2px solid var(--ln);padding:4px 8px;cursor:pointer;border-radius:0}
button:active{background:#6b4a2e}button:disabled{opacity:.4}
button:focus-visible{outline:2px solid var(--go)}
#bar button{white-space:nowrap;flex:1}
#sh{position:fixed;left:0;right:0;bottom:calc(52px + env(safe-area-inset-bottom,0px));max-height:58%;overflow:auto;background:var(--pn);border-top:3px solid var(--ln);padding:8px;display:none;z-index:5}
#sh h3{margin:0 0 6px;font-weight:400;color:var(--go);font-size:24px}
.g{display:grid;grid-template-columns:repeat(auto-fill,minmax(104px,1fr));gap:4px}
.t{text-align:center;line-height:1}
.row{display:flex;gap:8px;align-items:center;padding:4px 0;border-bottom:1px dashed var(--ln)}
.row img{width:40px;height:40px;image-rendering:pixelated;flex:none}.row div{flex:1}
small{color:#c9b68e}
</style></head><body><div id="app">
<div id="top"><span>💰<b id="m"></b></span><span>👥<b id="v"></b></span><span>⭐<b id="r"></b></span><span id="c"></span><span style="margin-left:auto"><button id="ps">⏸</button> <button id="sp">1x</button> <button id="sn">🔊</button> <button id="zo">➖</button><button id="zi">➕</button><button id="pt">✋</button><button id="dl">⬇️</button></span></div>
<div id="st"></div><div id="cvb"><canvas id="cv" width="448" height="288"></canvas></div>
<div id="sh"></div>
<div id="bar"><button onclick="openP('build')">🔨 Construir</button><button onclick="openP('sp')">🐞 Espécies</button><button onclick="pick('insp')">🔍 Inspecionar</button><button onclick="pick('land')">🗺️ Terrenos</button><button onclick="openP('staff')">👷 Equipe</button><button onclick="openP('stats')">📊 Stats</button><button onclick="openP('mis')">🎯 Missões</button><button onclick="openP('slots')">💾 Jogos</button></div></div>
<script>
const SRC='<!DOCTYPE html>'+document.documentElement.outerHTML;const $=id=>document.getElementById(id),R=Math.random,C=(v,a,b)=>Math.max(a,Math.min(b,v));
let W=28,H=18,ctx;const T=16,DAY=150,cv=$('cv'),SCR=cv.getContext('2d');ctx=SCR;
const PAL={k:'#222',r:'#d33',y:'#fc2',o:'#f80',g:'#4a4',G:'#272',b:'#36c',c:'#9ef',w:'#fff',B:'#864',d:'#531',p:'#f8c',P:'#94d',l:'#8c3'};
// [nome,descrição,recurso,comida,custo,custoComida,popularidade,vida(dias),espaço,voa,sprite]
const SP=[
['Joaninha','Bonita, popular e fácil de cuidar.','flower','Pulgões',60,1,3,12,2,0,['..kkkk..','.rrkkrr.','rkrkkrkr','rrrkkrrr','rkrkkrkr','.rrkkrr.','..rkkr..','........']],
['Abelha','Produz mel (+$3/dia) e atrai visitantes.','flower','Néctar',90,2,4,10,3,1,['..w..w..','..wwww..','.yyyyyy.','.kkkkkk.','.yyyyyy.','.kkkkkk.','..yyyy..','...kk...']],
['Borboleta','Colorida, ótima para jardins.','flower','Néctar',120,2,5,9,3,1,['P..kk..P','PP.kk.PP','PpPkkPpP','PPPkkPPP','.PPkkPP.','PpPkkPpP','PP.kk.PP','P......P']],
['Formiga','Vive em colônias, precisa de formigueiro.','hill','Migalhas',70,1,2,14,3,0,['..k..k..','...dd...','..dddd..','k..dd..k','.k.dd.k.','..dddd..','.k.dd.k.','k..dd..k']],
['Besouro-rinoceronte','Raro e resistente. Adora troncos.','log','Frutas',220,3,5,30,3,0,['...kk...','...kk...','..BBBB..','.BBBBBB.','.BdBBdB.','.BBBBBB.','k.BBBB.k','k..kk..k']],
['Louva-a-deus','Predador: pode atacar grilos do habitat.','tree','Moscas',160,3,4,12,4,0,['..l..l..','..llll..','.l.ll.l.','.l.ll.l.','..llll..','...ll...','...ll...','..l..l..']],
['Grilo','Barato e fácil de criar. Precisa de um tronco.','log','Folhas',40,1,2,10,2,0,['.k....k.','..BBBB..','.BBBBBB.','BBkBBBBB','.BBBBBB.','..BBBB..','.k.kk.k.','k..kk..k']],
['Libélula','Adulta e larva precisam de lagoa; a larva só cresce na água.','pond','Mosquitos',150,2,4,10,4,1,['cc.kk.cc','cc.kk.cc','...bb...','...bb...','cc.bb.cc','cc.bb.cc','...bb...','...bb...']],
['Vaga-lume','Brilha à noite e atrai visitantes noturnos.','tree','Néctar',180,2,3,8,3,1,['..w..w..','..wwww..','..kkkk..','.kkkkkk.','.kkkkkk.','.yyyyyy.','..yyyy..','...yy...']],
['Bicho-pau','Camufla-se entre galhos e folhas.','tree','Folhas',110,1,3,16,3,0,['...l....','...l....','..lll...','...l....','..lll...','...l....','..l.l...','.l...l..']]];
const F={flower:'Flores',tree:'Árvore',log:'Tronco',hill:'Formigueiro',pond:'Lagoa',rock:'Pedra'};
const BD={path:['Caminho',2,1],hab:['Habitat',12,1],tree:['Árvore',10,1],flower:['Flores',5,1],rock:['Pedra',6,1],log:['Tronco',12,1],hill:['Formigueiro',40,1],pond:['Lagoa',25,1],bench:['Banco',15,1],bin:['Lixeira',10,1],sign:['Placa',5,1],food:['Lanchonete',150,1],gift:['Presentes',200,2],fount:['Fonte',80,3],stat:['Estátua',120,4]};
const IC={path:'🟫',hab:'🟩',tree:'🌳',flower:'🌸',rock:'🪨',log:'🪵',hill:'⛰️',pond:'💧',bench:'🪑',bin:'🗑️',sign:'🪧',food:'🍔',gift:'🎁',fount:'⛲',stat:'🗿'};
const NAT=['tree','flower','rock','log','hill','pond'],STR=['bench','bin','sign','fount','stat'],SHOP=['food','gift'];
const LV=['Iniciante','Conhecido','Popular','Famoso','Lendário'];
const MS=[['Construa um habitat com 6 quadrados',()=>cnt('h')>=6,50],['Plante flores dentro do habitat',()=>RG.some(r=>r.f.flower),30],['Compre uma joaninha e ligue o habitat por um caminho',()=>A.some(a=>a.s==0),50],['Receba 10 visitantes',()=>tv>=10,100],['Contrate um tratador',()=>staff.t>=1,50],['Alcance 30 de reputação',()=>rep>=30,200],['Tenha 5 espécies diferentes',()=>nsp()>=5,300],['Tenha as 11 espécies',()=>nsp()>=11,1000]];
const SPC=SP.map(s=>{const c=document.createElement('canvas');c.width=c.height=10;const x=c.getContext('2d'),m=s[10],px=(i,j)=>!!(m[j]&&m[j][i]&&m[j][i]!='.');for(let j=-1;j<9;j++)for(let i=-1;i<9;i++)if(!px(i,j)&&(px(i+1,j)||px(i-1,j)||px(i,j+1)||px(i,j-1))){x.fillStyle='#1a1208';x.fillRect(i+1,j+1,1,1)}m.forEach((r,j)=>[...r].forEach((ch,i)=>{if(ch!='.'){x.fillStyle=PAL[ch];x.fillRect(i+1,j+1,1,1)}}));return c});
const icon=(i,v=0,st=2)=>{const k='u'+(v|0)+st;if(!SP[i][k])SP[i][k]=(()=>{const c=document.createElement('canvas');c.width=c.height=42;const x=c.getContext('2d');x.imageSmoothingEnabled=false;x.drawImage(st==3?SPE[i][v|0]:st==2?SPV[i][v|0]:SPL[i][v|0][st],0,0,42,42);return c.toDataURL()})();return `<img src="${SP[i][k]}">`};
let G,RG=[],RI=[],DG=[],A=[],V=[],money,rep,day,hour,tv,staff,price,U,mi,inc,exp,tool='insp',speed=1,hov=null,down=0,pn='',selC=null,toast='',toastT=0,idc=1,snd=1,ac,PL=[],slot=1,GY=8,cam={x:0,y:0},zm=1,WC=null,wx=null,mode=null,gst=null,pg=null,PT=new Map(),cf=null,ns=1;
const nsp=()=>new Set(A.map(a=>a.s)).size,cnt=k=>{let n=0;G.forEach(r=>r.forEach(c=>{if(c.k==k)n++}));return n},lvl=()=>1+Math.min(4,rep/20|0),night=()=>hour>=20||hour<5;
function bp(f,d=.08){if(!snd)return;try{ac=ac||new AudioContext();const o=ac.createOscillator(),g=ac.createGain();o.type='square';o.frequency.value=f;g.gain.value=.04;o.connect(g);g.connect(ac.destination);o.start();o.stop(ac.currentTime+d)}catch(e){}}
function msg(t){toast=t;toastT=3}
function earn(v){money+=v;inc+=v}
function newGame(){G=[];A=[];V=[];for(let y=0;y<H;y++){G[y]=[];for(let x=0;x<W;x++)G[y][x]={k:'g',d:0}}
for(let y=11;y<=14;y++)for(let x=3;x<=7;x++)if(!((x==3||x==7)&&(y==11||y==14)))G[y][x].k='w';
for(let x=0;x<=11;x++)G[8][x].k='p';for(let y=5;y<=10;y++)G[y][11].k='p';G[7][1].d='booth';G[8][0].d='gate';
let s=7;const rn=()=>(s=s*16807%2147483647)/2147483647;
for(let i=0;i<90;i++){const x=rn()*W|0,y=rn()*H|0,c=G[y][x];if(c.k!='g'||c.d||(y>=7&&y<=9&&x<13)||(x>=10&&x<=12&&y>=4&&y<=11))continue;const q=rn();c.d=q<.5?'tree':q<.8?'flower':'rock'}
money=600;rep=10;day=1;hour=8;tv=0;staff={t:0,f:0};price=8;U=18;initPL();mi=0;inc=0;exp=0;calc()}
function calc(){RG=[];RI=G.map(r=>r.map(()=>null));
for(let y=0;y<H;y++)for(let x=0;x<W;x++)if(G[y][x].k=='h'&&!RI[y][x]){const r={cl:[],f:{},an:[],dirt:0},q=[[x,y]];RI[y][x]=r;while(q.length){const [a,b]=q.pop();const c=G[b][a];r.cl.push([a,b,c]);if(c.d)r.f[c.d]=(r.f[c.d]||0)+1;for(const [dx,dy] of [[1,0],[-1,0],[0,1],[0,-1]]){const nx=a+dx,ny=b+dy;if(nx>=0&&ny>=0&&nx<W&&ny<H&&G[ny][nx].k=='h'&&!RI[ny][nx]){RI[ny][nx]=r;q.push([nx,ny])}}}RG.push(r)}
DG=G.map(r=>r.map(()=>99));const q=[[0,GY]];DG[GY][0]=0;while(q.length){const [a,b]=q.shift();for(const [dx,dy] of [[1,0],[-1,0],[0,1],[0,-1]]){const nx=a+dx,ny=b+dy;if(nx>=0&&ny>=0&&nx<W&&ny<H&&G[ny][nx].k=='p'&&DG[ny][nx]==99){DG[ny][nx]=DG[b][a]+1;q.push([nx,ny])}}}
const n=A.length;A=A.filter(a=>{if(RI[a.y|0][a.x|0])return true;earn(SP[a.s][4]/2);return false});if(A.length<n)msg('Insetos removidos foram vendidos.');rgAn()}
function rgAn(){RG.forEach(r=>{r.an=[];r.dirt=r.cl.reduce((s,c)=>s+(c[2].dt||0),0)/r.cl.length});A.forEach(a=>{const r=RI[a.y|0][a.x|0];if(r)r.an.push(a)})}
function use(x,y,first){if(x<0||y<0||x>=W||y>=H)return;if(!own(x,y)){if(first){tool=='land'?buyPlot(x,y):msg('Terreno à venda! Toque em 🗺️ Terrenos e depois no terreno escuro.')}return}const c=G[y][x],t=tool;
if(t=='insp'){const r=RI[y][x];if(r){selC=[x,y];openP('reg',1)}else if(first)msg('Toque em um habitat para ver os insetos.');return}
if(t.startsWith('buy')){const r=RI[y][x],i=+t.slice(4),s=SP[i];if(!r){msg('Toque dentro de um habitat.');return}
if(!r.f[s[2]]){msg('Falta '+F[s[2]]+' neste habitat!');return}if(money<s[4]){msg('Dinheiro insuficiente.');return}
money-=s[4];A.push({id:idc++,s:i,x:x+.5,y:y+.5,tx:x+.5,ty:y+.5,sat:100,hap:80,hp:100,age:0,hrs:0,eg:EG[i],v:R()*3|0});bp(880,.15);msg(s[0]+': ovo colocado! Vai eclodir, virar casulo e depois adulto.');tool='insp';rgAn();return}
if(t=='erase'){if(c.d=='booth'||c.d=='gate'||c.k=='w')return;let v=0;if(c.d){v=BD[c.d][1];c.d=0}else if(c.k=='p'||c.k=='h'){v=BD[c.k=='p'?'path':'hab'][1];c.k='g';c.dt=0}else return;earn(v/2);inc-=v/2;bp(300);calc();return}
const b=BD[t];if(!b)return;if(lvl()<b[2]){msg('Bloqueado: reputação nível '+b[2]);return}
let ok=false;if(t=='path'||t=='hab')ok=c.k=='g'&&!c.d;else if(NAT.includes(t))ok=(c.k=='g'||c.k=='h')&&!c.d;else if(STR.includes(t))ok=(c.k=='g'||c.k=='p')&&!c.d;else if(SHOP.includes(t))ok=c.k=='g'&&!c.d;
if(!ok){if(first)msg('Não dá para construir aqui.');return}if(money<b[1]){msg('Dinheiro insuficiente.');return}
money-=b[1];if(t=='path')c.k='p';else if(t=='hab')c.k='h';else c.d=t;bp(520);calc()}
function pick(t){tool=t;closeP();const n=t=='insp'?'Inspecionar: toque em um habitat':t=='erase'?'Demolir (reembolsa 50%)':BD[t]?'Construir: '+BD[t][0]+' ($'+BD[t][1]+')':'';msg(n)}
function ecost(){return 300+250*((U-18)/4)}
function expand(){if(U>=W){msg('Terreno já está no máximo.');return}const c=ecost();if(money<c){msg('Precisa de $'+c);return}money-=c;U+=4;bp(700,.2);msg('Terreno expandido!');openP('build',1)}
function feedR(r){let n=0;r.an.forEach(a=>{if(a.sat<100){const c=SP[a.s][5];if(money>=c){money-=c;a.sat=100;n++}}});return n}
function cleanR(r){r.cl.forEach(c=>c[2].dt=0)}
function sell(id){const i=A.findIndex(a=>a.id==id);if(i<0)return;earn(SP[A[i].s][4]/2);inc-=SP[A[i].s][4]/2;A.splice(i,1);rgAn();bp(300);P()}
function hourly(){rgAn();RG.forEach(r=>{if(!r.an.length)return;const d=r.an.length*1.5/r.cl.length;r.cl.forEach(c=>c[2].dt=Math.min(100,(c[2].dt||0)+d))});rgAn();
for(let i=0;i<staff.t;i++){let n=0;for(const a of A.filter(a=>a.sat<60).sort((a,b)=>a.sat-b.sat)){if(n++>=4)break;const c=SP[a.s][5];if(money>=c){money-=c;a.sat=100}}}
for(let i=0;i<staff.f;i++){const r=RG.slice().sort((a,b)=>b.dirt-a.dirt)[0];if(r&&r.dirt>10)cleanR(r)}rgAn();
for(const a of A.slice()){const s=SP[a.s],r=RI[a.y|0][a.x|0];if(a.eg>0){if(!(a.s==7&&!r.f.pond)){a.eg--;if(a.eg==0){msg(s[0]+': o ovo eclodiu! 🐣');bp(900,.2)}}continue}if(a.hrs!=null){const dry=a.s==7&&stg(a)==0&&!r.f.pond;if(dry){a.hp-=4;if(!a.wm){a.wm=1;msg('A larva da libélula precisa de água! Coloque uma lagoa no habitat.')}}else{a.wm=0;a.hrs++}const ns=stg(a);if(a.sg!=null&&ns>a.sg){msg(s[0]+(ns==1?' virou casulo!':' virou adulto! 🦋'));bp(990,.2)}a.sg=ns}const sg=stg(a);a.sat=Math.max(0,a.sat-(sg==1?0:sg==0?4:5));let tg=100-(100-a.sat)*.5-r.dirt*.5;if(r.an.reduce((n,q)=>n+SP[q.s][8],0)>r.cl.length)tg-=30;if(!r.f[s[2]])tg-=40;a.hap+=(tg-a.hap)*.3;
if(a.sat<10||a.hap<20)a.hp-=6;else a.hp=Math.min(100,a.hp+2);
if((a.s==5||a.s==10)&&R()<.05){const p=r.an.find(q=>q.s==6||(a.s==10&&q.s==3));if(p){A.splice(A.indexOf(p),1);msg('O '+s[0].toLowerCase()+' comeu um '+SP[p.s][0].toLowerCase()+'!');rgAn()}}
if(a.hp<=0){A.splice(A.indexOf(a),1);rep=Math.max(0,rep-3);msg('💀 '+s[0]+' morreu por maus cuidados! Reputação caiu.');bp(150,.3);rgAn()}}}
function daily(){let sal=staff.t*25+staff.f*20,mt=0;G.forEach(r=>r.forEach(c=>{if(c.k=='h')mt+=.5;if(SHOP.includes(c.d))mt+=5}));mt=Math.round(mt);
const hn=A.filter(a=>a.s==1&&a.hap>40).length*3;earn(hn);money-=sal+mt;exp+=sal+mt;
for(const a of A.slice()){if(!(a.eg>0))a.age++;if(a.age>=SP[a.s][7]){A.splice(A.indexOf(a),1);msg(SP[a.s][0]+' morreu de velhice.')}}
if(A.length){const av=A.reduce((s,a)=>s+a.hap,0)/A.length;rep=C(rep+(av-60)/40,0,100)}
day++;rgAn();msg(`Dia ${day}: receitas $${Math.round(inc)}, despesas $${Math.round(exp)}`+(money<0?' — no vermelho!':''));save();inc=0;exp=0}
function spawn(dt){const pop=A.reduce((s,a)=>s+SP[a.s][6]*(a.eg>0?0:[.3,.15,1][stg(a)])*(night()?(a.s==8?2.5:a.s==10?1.6:.4):1),0);if(!pop||V.length>=70)return;
const p=(.03+pop*.012+rep*.0015)*Math.max(.15,1.4-price/14);if(R()<p*dt){V.push({x:0,y:GY,px:0,py:GY,lx:-1,ly:-1,t:0,n:0,max:18+R()*30|0,sat:0,seen:new Set,out:0,ate:0,col:['#e5533d','#3a82d4','#f4b83a','#b57bff','#4caf50'][R()*5|0]});earn(price);tv++;bp(1000,.05)}}
function stepV(v,i){v.n++;const night_=night();
for(const a of A)if(!v.seen.has(a.id)&&Math.hypot(a.x-v.x-.5,a.y-v.y-.5)<2.6){v.seen.add(a.id);v.sat+=SP[a.s][6]*(a.eg>0?0:[.3,.15,1][stg(a)])*a.hap/100*(night_?(a.s==8?2.5:a.s==10?1.6:.4):1)*.5}
const ns=[];for(let dy=-1;dy<=1;dy++)for(let dx=-1;dx<=1;dx++){const c=(G[v.y+dy]||[])[v.x+dx];if(!c)continue;if(c.d=='food'&&!v.ate&&R()<.5){earn(6);v.ate=1;v.sat+=1}if(c.d=='gift'&&R()<.1){earn(12);v.sat+=1}if(c.d&&!ns.includes(c.d)&&(NAT.includes(c.d)||STR.includes(c.d))){ns.push(c.d);v.sat+=.25}}
if(!v.out&&v.n>=v.max)v.out=1;let o=[];for(const [dx,dy] of [[1,0],[-1,0],[0,1],[0,-1]]){const c=(G[v.y+dy]||[])[v.x+dx];if(c&&c.k=='p')o.push([v.x+dx,v.y+dy])}
if(v.out){if(DG[v.y][v.x]==0){const f=C(v.sat*9,0,100);rep=C(rep+(f-45)/45*.8,0,100);V.splice(i,1);return}o=o.filter(p=>DG[p[1]][p[0]]<DG[v.y][v.x]);if(!o.length){V.splice(i,1);return}}
else{const f=o.filter(p=>p[0]!=v.lx||p[1]!=v.ly);if(f.length)o=f}
if(o.length){const p=o[R()*o.length|0];v.lx=v.x;v.ly=v.y;v.x=p[0];v.y=p[1]}}
function update(dt){const g=dt*speed;if(!g)return;const ph=hour;hour+=g/DAY*24;if(hour>=24){hour-=24}
if((hour|0)!=(ph|0)){if((hour|0)==0)daily();hourly()}
spawn(g);for(let i=V.length-1;i>=0;i--){const v=V[i];v.t+=g;if(v.t>=.35){v.t=0;stepV(v,i)}if(V[i]===v){v.px+=(v.x-v.px)*Math.min(1,g*10);v.py+=(v.y-v.py)*Math.min(1,g*10)}}
for(const a of A){const s=SP[a.s],d=Math.hypot(a.tx-a.x,a.ty-a.y);if(d<.05){const r=RI[a.y|0][a.x|0];if(r){let L=r.cl;if(a.s==7&&stg(a)==0){const w=L.filter(q=>q[2].d=='pond');if(w.length)L=w}const c=L[R()*L.length|0];a.tx=c[0]+.2+R()*.6;a.ty=c[1]+.2+R()*.6}a.w=R()*2}else{const sp=(s[9]?1.2:.5)*g*(a.eg>0?0:[.5,0,1][stg(a)]);a.x+=(a.tx-a.x)/d*Math.min(sp,d);a.y+=(a.ty-a.y)/d*Math.min(sp,d)}}}
let mt=0;function mis(){if(mi<MS.length&&MS[mi][1]()){earn(MS[mi][2]);inc-=MS[mi][2];msg('🎉 Missão cumprida! +$'+MS[mi][2]);bp(1100,.3);mi++;if(pn=='mis')P()}}
const rc=(x,y,w,h,c)=>{ctx.fillStyle=c;ctx.fillRect(x,y,w,h)};
function deco(d,hh){switch(d){
case'tree':rc(7,9,2,6,'#6b4423');rc(3,3,10,7,'#2e7d32');rc(2,5,12,4,'#2e7d32');rc(5,1,6,3,'#2e7d32');rc(4,4,2,2,'#4caf50');rc(9,6,2,2,'#4caf50');break;
case'flower':{const a=['#ff5c8a','#ffd23f','#fff','#b57bff'];rc(4,8,1,5,'#2e7d32');rc(10,6,1,6,'#2e7d32');rc(7,10,1,4,'#2e7d32');rc(3,7,3,3,a[hh%4]);rc(9,5,3,3,a[(hh+1)%4]);rc(6,9,3,3,a[(hh+2)%4]);break}
case'rock':rc(3,8,10,6,'#888');rc(4,7,7,2,'#9a9a9a');rc(5,8,3,1,'#bbb');rc(3,13,10,1,'#666');break;
case'log':rc(1,7,14,6,'#7a4a24');rc(1,7,14,1,'#945c2f');rc(1,8,3,4,'#c99a5e');rc(2,9,1,2,'#7a4a24');break;
case'hill':rc(2,10,12,4,'#7a5230');rc(4,7,8,4,'#8a6038');rc(6,5,4,3,'#9a6e44');rc(7,10,2,3,'#2a1a0c');break;
case'pond':rc(1,3,14,10,'#3a82d4');rc(3,2,10,12,'#3a82d4');rc(4,5,4,1,'#9fd0f8');rc(9,9,3,1,'#9fd0f8');break;
case'bench':rc(2,8,12,2,'#8a5a2b');rc(2,11,12,2,'#a8703a');rc(3,13,2,2,'#4a2f18');rc(11,13,2,2,'#4a2f18');break;
case'bin':rc(5,5,6,9,'#6f7d86');rc(4,4,8,2,'#4a5860');rc(6,7,4,1,'#9fb0b9');break;
case'sign':rc(7,6,2,9,'#6b4423');rc(2,2,12,6,'#e8d09a');rc(3,3,10,4,'#c59b4a');break;
case'fount':rc(1,7,14,7,'#9aa4ad');rc(2,8,12,5,'#3a82d4');rc(7,2,2,8,'#bfe6ff');rc(5,4,6,1,'#bfe6ff');break;
case'stat':rc(4,11,8,4,'#8d8d8d');rc(6,4,4,8,'#b0b0b0');rc(5,2,6,4,'#c4c4c4');break;
case'food':rc(1,6,14,9,'#e8e0cc');rc(0,3,16,4,'#d63a2f');rc(2,3,3,4,'#fff');rc(8,3,3,4,'#fff');rc(6,9,4,6,'#5a3b24');break;
case'gift':rc(1,6,14,9,'#b57bff');rc(0,3,16,4,'#6a3fb5');rc(6,9,4,6,'#5a3b24');rc(11,8,3,3,'#ffd23f');break;
case'booth':rc(1,6,14,9,'#4a8fd6');rc(0,3,16,4,'#ffd23f');rc(4,8,8,4,'#bfe6ff');rc(1,14,14,1,'#2d5f94');break;
case'gate':rc(0,0,3,16,'#8a5a2b');rc(13,0,3,16,'#8a5a2b');rc(0,0,16,4,'#d63a2f');rc(5,1,6,2,'#fff');break}}
function tile(x,y,c,tm){const h=((x*73856093)^(y*19349663))>>>0,hh=h%16;ctx.setTransform(1,0,0,1,x*T,y*T);
if(c.k=='w'){rc(0,0,16,16,'#3a82d4');const o=(tm*4+hh)%16|0;rc(o,4,3,1,'#7db8f0');rc((o+8)%16,10,3,1,'#7db8f0')}
else if(c.k=='p'){rc(0,0,16,16,'#cdaa70');rc(hh,hh,2,1,'#b08a52');rc(15-hh,8,1,1,'#a07a44');rc(3,12,2,1,'#e0c590')}
else{const hb=c.k=='h';rc(0,0,16,16,hb?'#86c85f':'#5aa447');rc(hh,hh*3%14,1,2,hb?'#6ba84a':'#3f8a35');rc(hh*5%15,hh*7%14,1,2,'#3f8a35');
if(hb){const nb=(a,b)=>{const q=(G[b]||[])[a];return q&&q.k=='h'};const fc='#8a5a2b';if(!nb(x,y-1))rc(0,0,16,2,fc);if(!nb(x,y+1))rc(0,14,16,2,fc);if(!nb(x-1,y))rc(0,0,2,16,fc);if(!nb(x+1,y))rc(14,0,2,16,fc);
if((c.dt||0)>40){rc(3,4,3,2,'#7a5a3a');rc(10,10,3,2,'#7a5a3a')}}}
if(c.d)deco(c.d,hh)}
function na(){if(hour>=7&&hour<18)return 0;if(hour>=20||hour<5)return .55;return hour>=18?(hour-18)/2*.55:(7-hour)/2*.55}
function draw(tm){ctx.setTransform(1,0,0,1,0,0);for(let y=0;y<H;y++)for(let x=0;x<W;x++)tile(x,y,G[y][x],tm);ctx.setTransform(1,0,0,1,0,0);
if(U<W){rc(U*T,0,(W-U)*T,H*T,'rgba(0,0,0,.6)')}
for(const a of A){const s=SP[a.s],b=s[9]?Math.sin(tm*8+a.id)*1.5:(Math.sin(tm*10)>0&&Math.hypot(a.tx-a.x,a.ty-a.y)>.05?1:0);ctx.drawImage(SPC[a.s],Math.round(a.x*T-4),Math.round(a.y*T-4+b))}
for(const v of V){const x=Math.round(v.px*T+5),y=Math.round(v.py*T+3+(Math.sin(tm*12+v.n)>0?1:0));rc(x,y,5,4,'#f1c8a0');rc(x-1,y+4,7,5,v.col);rc(x,y+9,2,3,'#334');rc(x+3,y+9,2,3,'#334')}
const a=na();if(a>0){rc(0,0,W*T,H*T,`rgba(12,16,60,${a})`);for(const q of A)if(q.s==8){const g=.25*a/.55;ctx.fillStyle=`rgba(255,240,100,${g})`;ctx.beginPath();ctx.arc(q.x*T,q.y*T,10,0,7);ctx.fill();ctx.beginPath();ctx.arc(q.x*T,q.y*T,5,0,7);ctx.fill();rc(q.x*T-1,q.y*T+1,2,2,'#ffff9a')}}
if(hov&&hov[0]>=0&&hov[0]<U&&hov[1]>=0&&hov[1]<H){ctx.strokeStyle=tool=='erase'?'#ff5c4a':'#fff';ctx.lineWidth=1;ctx.strokeRect(hov[0]*T+.5,hov[1]*T+.5,T-1,T-1)}}
function bar(n){return `<b style="color:${n<30?'#e5533d':n<60?'#f4b83a':'#8fd14f'}">${Math.round(n)}%</b>`}
function P(){const s=$('sh');if(!pn){s.style.display='none';return}s.style.display='block';let h='';
if(pn=='build'){h='<h3>Construir</h3><div class=g>'+Object.keys(BD).map(k=>{const [n,c,l]=BD[k],lk=lvl()<l;return `<button class=t ${lk?'disabled':''} onclick="pick('${k}')">${IC[k]}<br>${n}<br><small>${lk?'🔒 nível '+l:'$'+c}</small></button>`}).join('')+`<button class=t onclick="pick('erase')">💥<br>Demolir<br><small>reembolsa 50%</small></button><button class=t onclick="expand()">🗺️<br>Terrenos<br><small>a partir de $${ecost()}</small></button></div>`}
if(pn=='sp'){h='<h3>Espécies</h3>'+SP.map((s,i)=>`<div class=row>${icon(i)}<div>${s[0]}<br><small>${s[1]}<br>Precisa: ${F[s[2]]} · Come: ${s[3]} · Espaço: ${s[8]} · Vida: ${s[7]} dias · Popularidade: ${'★'.repeat(s[6])} · Comida: $${s[5]}</small></div><button onclick="pick('buy:${i}')">$${s[4]}</button></div>`).join('')}
if(pn=='reg'){const r=selC&&RI[selC[1]][selC[0]];if(!r){pn='';P();return}rgAn();h=`<h3>Habitat (${r.cl.length} quadrados)</h3>Limpeza ${bar(100-r.dirt)} · ${Object.entries(r.f).map(([k,v])=>F[k]+' ×'+v).join(', ')||'sem recursos'}<br><button onclick="feedR(RI[selC[1]][selC[0]]);P()">🍽️ Alimentar</button> <button onclick="cleanR(RI[selC[1]][selC[0]]);P()">🧹 Limpar</button>`+(r.an.length?r.an.map(a=>`<div class=row>${icon(a.s,a.v,a.eg>0?3:stg(a))}<div>${SP[a.s][0]} <small>${a.eg>0?'ovo · eclode em '+a.eg+'h':['larva','casulo','adulto'][stg(a)]} · ${a.age}/${SP[a.s][7]} dias</small><br>Saciedade ${bar(a.sat)} Humor ${bar(a.hap)} Saúde ${bar(a.hp)}</div><button onclick="sell(${a.id})">Vender $${SP[a.s][4]/2}</button></div>`).join(''):'<p>Vazio. Abra Espécies e compre um inseto (precisa do recurso certo).</p>')}
if(pn=='staff'){const row=(k,n,sal,d)=>`<div class=row><div>${n}: <b>${staff[k]}</b><br><small>$${sal}/dia · ${d}</small></div><button onclick="staff.${k}=Math.max(0,staff.${k}-1);P()">−</button><button onclick="staff.${k}<5?staff.${k}++:0;P()">Contratar</button></div>`;h='<h3>Equipe</h3>'+row('t','🧑‍🌾 Tratadores',25,'alimentam 4 insetos famintos por hora')+row('f','🧹 Faxineiros',20,'limpam 1 habitat sujo por hora')}
if(pn=='stats'){const n=A.length,av=n?A.reduce((s,a)=>s+a.hap,0)/n:0;h=`<h3>Estatísticas</h3>Dia ${day} · Nível: ${LV[lvl()-1]} (${Math.round(rep)}/100)<br>Visitantes totais: ${tv} · Insetos: ${n} (${nsp()} espécies) · Humor médio ${bar(av)}<br>Hoje: receitas $${Math.round(inc)} · despesas $${Math.round(exp)}<br><div class=row><div>Ingresso: <b>$${price}</b><br><small>mais caro = menos visitantes</small></div><button onclick="price=Math.max(1,price-1);P()">−</button><button onclick="price=Math.min(30,price+1);P()">+</button></div><button onclick="save(1)">💾 Salvar</button> <button onclick="openP('slots')">🔄 Novo jogo / Jogos</button>`}
if(pn=='mis'){h='<h3>Missões</h3>'+MS.map((m,i)=>`<div class=row><div>${i<mi?'✅':i==mi?'➡️':'⬜'} ${m[0]}<br><small>Recompensa $${m[2]}</small></div></div>`).join('')}
if(pn=='slots')h=slotsHTML();if(pn=='newg')h=newHTML();s.innerHTML=h}
function openP(n,keep){if(pn==n&&!keep){closeP();return}pn=n;P()}
function closeP(){pn='';P()}
function save(m){try{localStorage.setItem('tzs1',JSON.stringify({G,A,money,rep,day,hour,tv,staff,price,U,mi,idc,PL}));if(m)msg('Jogo salvo!')}catch(e){if(m)msg('Não foi possível salvar.')}}
function load(){try{const d=JSON.parse(localStorage.getItem('tzs1'));if(!d)return 0;({G,A,money,rep,day,hour,tv,staff,price,U,mi,idc,PL}=d);if(!PL||!PL.length)initPL();V=[];inc=0;exp=0;calc();return 1}catch(e){return 0}}
function fit(){const b=$('cvb').getBoundingClientRect(),s=Math.min((b.width-8)/448,(b.height-8)/288);cv.style.width=448*s+'px';cv.style.height=288*s+'px'}
function hv(e){const r=cv.getBoundingClientRect();hov=[Math.floor((e.clientX-r.left)/r.width*W),Math.floor((e.clientY-r.top)/r.height*H)]}
cv.addEventListener('pointerdown',pdown);
cv.addEventListener('pointermove',pmove);
addEventListener('pointerup',pup);addEventListener('resize',fit);
$('ps').onclick=()=>{speed=speed?0:1;$('sp').textContent='1x'};
$('sp').onclick=()=>{speed=speed>=3?1:speed+1;$('sp').textContent=speed+'x'};
$('sn').onclick=()=>{snd=!snd;$('sn').textContent=snd?'🔊':'🔇'};
if(!load())newGame();fit();msg('Bem-vindo! Missão: '+MS[mi][0]);
let last=performance.now(),tm=0,ui=0;
function loop(n){const dt=Math.min(.1,(n-last)/1000);last=n;tm+=dt;update(dt);draw(tm);
toastT-=dt;ui+=dt;if(ui>.25){ui=0;mis();$('m').textContent='$'+Math.floor(money);$('m').style.color=money<0?'#e5533d':'';$('v').textContent=V.length;$('r').textContent=LV[lvl()-1]+' '+Math.round(rep);$('c').textContent='Dia '+day+' · '+String(hour|0).padStart(2,'0')+'h'+(night()?' 🌙':' ☀️');$('ps').textContent=speed?'⏸':'▶';
$('st').textContent=toastT>0?toast:(mi<MS.length?'🎯 '+MS[mi][0]:'Zoo completo! Continue expandindo.')}
requestAnimationFrame(loop)}
function initPL(){PL=[];for(let j=0;j<6;j++){PL[j]=[];for(let i=0;i<7;i++)PL[j][i]=i<=3&&j>=1&&j<=4}}
const own=(x,y)=>!!(PL[y/3|0]||[])[x/4|0],pc=()=>PL.flat().filter(Boolean).length;
function plotCost(){return 150+40*(pc()-16)}
function ecost(){return plotCost()}
function expand(){pick('land')}
function adj(i,j){return [[1,0],[-1,0],[0,1],[0,-1]].some(([a,b])=>(PL[j+b]||[])[i+a])}
function buyPlot(x,y){const i=x/4|0,j=y/3|0;if(!adj(i,j)){msg('Compre primeiro um terreno vizinho às suas terras.');return}const c=plotCost();if(money<c){msg('Precisa de $'+c+' para este terreno.');return}money-=c;PL[j][i]=true;bp(700,.2);msg('Terreno comprado! Já pode construir nele.')}
function pick(t){tool=t;closeP();msg(t=='insp'?'Inspecionar: toque em um habitat':t=='erase'?'Demolir (reembolsa 50%)':t=='land'?'Toque em um terreno escuro com placa para comprar ($'+plotCost()+')':BD[t]?'Construir: '+BD[t][0]+' ($'+BD[t][1]+')':'')}
function deco0(d,hh){const sh=(x,y,w)=>rc(x,y,w,2,'rgba(0,0,0,.22)');switch(d){
case'tree':sh(3,13,11);rc(7,9,3,5,'#5b3a1e');rc(7,9,1,5,'#7a5230');rc(2,5,12,6,'#2a7030');rc(3,2,10,5,'#37903d');rc(5,0,6,3,'#46a64c');rc(1,6,1,4,'#1d4f21');rc(14,6,1,4,'#1d4f21');rc(4,3,3,2,'#6cc66f');rc(9,6,3,2,'#5ab85e');rc(3,9,10,2,'#225a27');rc(5,5,1,1,'#ff6b6b');rc(10,3,1,1,'#ff6b6b');break;
case'flower':{const a=['#ff5c8a','#ffd23f','#fff','#b57bff'];[[3,6],[9,4],[6,9],[11,10]].forEach(([x,y],i)=>{const k=a[(hh+i)%4];rc(x+1,y+2,1,5,'#2e7d32');rc(x,y+1,3,1,k);rc(x+1,y,1,3,k);rc(x+1,y+1,1,1,'#ffd23f')});rc(4,12,2,1,'#3f9a46');break}
case'rock':sh(2,13,12);rc(3,8,10,6,'#7d7d7d');rc(4,6,7,3,'#9a9a9a');rc(5,6,3,1,'#c4c4c4');rc(3,13,10,1,'#555');rc(2,9,1,4,'#555');rc(13,9,1,4,'#555');rc(9,8,3,3,'#8a8a8a');rc(5,5,5,1,'#555');break;
case'log':sh(1,13,14);rc(2,6,12,7,'#7a4a24');rc(2,6,12,1,'#a36a38');rc(4,9,7,1,'#5e3a19');rc(6,11,5,1,'#5e3a19');rc(1,7,3,5,'#d6a468');rc(2,8,1,3,'#9a6c3a');rc(1,6,3,1,'#3d2410');rc(1,12,3,1,'#3d2410');rc(12,5,3,2,'#5ab85e');break;
case'hill':sh(1,13,14);rc(2,10,12,4,'#7a5230');rc(4,7,8,4,'#8a6038');rc(6,5,4,3,'#9d7044');rc(6,5,4,1,'#b88556');rc(7,10,2,3,'#1e1209');rc(3,12,1,1,'#3a2410');rc(12,11,1,1,'#3a2410');break;
case'pond':rc(2,4,12,9,'#d9c48a');rc(3,3,10,11,'#d9c48a');rc(3,5,10,7,'#3a86d8');rc(4,4,8,9,'#3a86d8');rc(5,6,4,1,'#9fd0f8');rc(8,9,3,1,'#9fd0f8');rc(6,11,3,1,'#2f74c4');rc(10,5,3,2,'#3f9a46');break;
case'bench':sh(2,14,12);rc(2,5,12,2,'#a8703a');rc(2,5,12,1,'#d49a56');rc(2,8,12,3,'#8a5a2b');rc(2,8,12,1,'#c58a4a');rc(3,11,2,3,'#3d2410');rc(11,11,2,3,'#3d2410');break;
case'bin':sh(4,14,8);rc(5,5,6,9,'#6f8a7a');rc(4,3,8,3,'#3f5a4a');rc(5,5,1,9,'#8fa89a');rc(6,8,4,1,'#3f5a4a');rc(6,10,4,1,'#3f5a4a');break;
case'sign':sh(5,14,6);rc(7,7,2,8,'#5e3a19');rc(1,1,14,7,'#5e3a19');rc(2,2,12,5,'#f0dca4');rc(4,3,8,1,'#a8703a');rc(4,5,5,1,'#a8703a');break;
case'fount':sh(1,14,14);rc(1,8,14,6,'#8a949c');rc(1,8,14,1,'#c0c8ce');rc(3,9,10,4,'#3a86d8');rc(7,3,2,6,'#bfe6ff');rc(6,2,4,2,'#8a949c');rc(4+(tm*6%3|0),5,1,1,'#fff');rc(10-(tm*6%3|0),6,1,1,'#fff');break;
case'stat':sh(3,14,10);rc(3,11,10,4,'#8a8a8a');rc(3,11,10,1,'#c4c4c4');rc(6,5,4,7,'#b8b8b8');rc(6,5,1,7,'#dcdcdc');rc(5,1,6,5,'#c8c8c8');rc(5,1,1,5,'#e8e8e8');rc(8,3,1,1,'#555');break;
case'food':rc(2,7,12,8,'#f0e6d0');rc(0,3,16,5,'#d63a2f');for(let i=0;i<16;i+=4)rc(i,3,2,5,'#fff');rc(0,8,16,1,'#8f241c');rc(3,10,10,2,'#8a5a2b');rc(5,9,2,1,'#e5a33d');rc(9,9,2,1,'#e5533d');rc(1,14,14,1,'#8a7a5a');break;
case'gift':rc(2,7,12,8,'#e8d4f8');rc(0,3,16,5,'#7b4fc4');for(let i=0;i<16;i+=4)rc(i,3,2,5,'#ffd23f');rc(0,8,16,1,'#4f2f8a');rc(4,10,8,4,'#bfe6ff');rc(5,11,3,3,'#ffd23f');rc(9,11,3,3,'#ff5c8a');rc(13,0,2,3,'#ff5c8a');rc(14,3,1,1,'#555');break;
case'booth':rc(2,7,12,8,'#5a9ae0');rc(0,3,16,5,'#f4b83a');for(let i=0;i<16;i+=4)rc(i,3,2,5,'#fff2c0');rc(0,8,16,1,'#a8741a');rc(4,9,8,4,'#d6f0ff');rc(4,13,8,1,'#2d5f94');rc(6,10,4,2,'#f4b83a');rc(4,0,8,3,'#d63a2f');rc(5,1,6,1,'#fff');break;
case'gate':rc(0,0,3,16,'#8a5a2b');rc(13,0,3,16,'#8a5a2b');rc(0,0,1,16,'#c58a4a');rc(0,0,16,5,'#d63a2f');rc(0,0,16,1,'#ff7a66');rc(3,1,10,3,'#fff6d6');rc(5,2,6,1,'#d63a2f');rc(0,5,16,1,'#7a1f18');break}}
function tile(x,y,c,tm){const h=((x*73856093)^(y*19349663))>>>0,hh=h%16,h2=(h>>4)%16,h3=(h>>8)%16;ctx.setTransform(1,0,0,1,x*T,y*T);const nb=(a,b,k)=>{const q=(G[b]||[])[a];return !!q&&(q.k==k||(k=='w'&&q.d=='pond'))};
if(c.k=='w'){rc(0,0,16,16,'#3a86d8');rc(0,8,16,8,'#2f74c4');const o=(tm*5+hh)%16|0;rc(o,3,4,1,'#8cc4f4');rc((o+9)%16,11,3,1,'#6aa9e6');const sd='#e3cf94',fm='#d6efff';if(!nb(x,y-1,'w')){rc(0,0,16,2,sd);rc(0,2,16,1,fm)}if(!nb(x,y+1,'w')){rc(0,14,16,2,sd);rc(0,13,16,1,fm)}if(!nb(x-1,y,'w')){rc(0,0,2,16,sd);rc(2,0,1,16,fm)}if(!nb(x+1,y,'w')){rc(14,0,2,16,sd);rc(13,0,1,16,fm)}if(hh==7){rc(6,6,5,3,'#3f9a46');rc(8,6,1,1,'#ff9ac0')}}
else if(c.k=='p'){rc(0,0,16,16,'#d9bb80');[[h2,2,3,2],[h3,9,4,2],[h2*3%12,12,3,2]].forEach(([a,b,w,q])=>{rc(a,b,w,q,'#c0a068');rc(a,b,w,1,'#e8d09a')});const ed='#9c7e4c';if(!nb(x,y-1,'p'))rc(0,0,16,1,ed);if(!nb(x,y+1,'p'))rc(0,15,16,1,ed);if(!nb(x-1,y,'p'))rc(0,0,1,16,ed);if(!nb(x+1,y,'p'))rc(15,0,1,16,ed)}
else{const hb=c.k=='h';rc(0,0,16,16,hb?((x+y)&1?'#8ccf66':'#82c55d'):((x+y)&1?'#5fae4a':'#58a644'));const g1=hb?'#6fb04c':'#3f8a35';rc(h2,h3%12+1,1,2,g1);rc(h2+1,h3%12,1,2,g1);rc(hh*5%14,h2*7%13,1,2,g1);rc(hh*3%15,h3*5%15,1,1,hb?'#a8e486':'#7fcc62');if(!hb&&hh==3){rc(11,6,1,1,'#fff');rc(11,7,1,1,'#2e7d32')}if(!hb&&hh==9){rc(4,10,1,1,'#ffd23f');rc(4,11,1,1,'#2e7d32')}
if(hb){const fc='#9a5f2c',fl='#d49a56',fs='#5e3a19',po='#7a4a22';if(!nb(x,y-1,'h')){rc(0,0,16,3,fc);rc(0,0,16,1,fl);rc(0,3,16,1,fs);rc(2,0,2,5,po);rc(10,0,2,5,po)}if(!nb(x,y+1,'h')){rc(0,12,16,3,fc);rc(0,12,16,1,fl);rc(0,15,16,1,fs);rc(2,11,2,5,po);rc(10,11,2,5,po)}if(!nb(x-1,y,'h')){rc(0,0,3,16,fc);rc(0,0,1,16,fl);rc(3,0,1,16,fs);rc(0,2,5,2,po);rc(0,10,5,2,po)}if(!nb(x+1,y,'h')){rc(13,0,3,16,fc);rc(13,0,1,16,fl);rc(15,0,1,16,fs);rc(11,2,5,2,po);rc(11,10,5,2,po)}if((c.dt||0)>40){rc(3,6,3,2,'#7a5a3a');rc(9,10,3,2,'#6a4a2a');rc(11,5,2,1,'#6a4a2a')}}}
if(c.d)deco(c.d,hh)}
function ani(a,tm){const s=SP[a.s],mv=Math.hypot(a.tx-a.x,a.ty-a.y)>.05,b=s[9]?Math.round(Math.sin(tm*8+a.id)*2):(mv&&Math.sin(tm*12+a.id)>0?-1:0),x=Math.round(a.x*T),y=Math.round(a.y*T),w=s[9]&&Math.sin(tm*30+a.id)>0?16:20;ctx.fillStyle='rgba(0,0,0,.22)';ctx.fillRect(x-5,y+(s[9]?8:6),10,3);ctx.drawImage(SPC[a.s],x-w/2,y-12+b*2,w,20);if(a.sat<30||a.hap<35){rc(x+6,y-15,2,4,'#e5533d');rc(x+6,y-10,2,1,'#e5533d')}}
function vis(v,tm){const x=Math.round(v.px*T+5),y=Math.round(v.py*T+2),w=Math.sin(tm*12+v.n)>0?1:0,hc=['#3a2410','#e0b040','#222','#8a3a1a'][v.max%4];ctx.fillStyle='rgba(0,0,0,.22)';ctx.fillRect(x-1,y+13,8,2);rc(x,y+1,6,4,'#f1c8a0');rc(x,y,6,2,hc);rc(x+1,y+3,1,1,'#222');rc(x+4,y+3,1,1,'#222');rc(x-1,y+5,8,5,v.col);rc(x+2,y+5,2,5,'rgba(255,255,255,.25)');rc(x-1,y+5,1,5,'rgba(0,0,0,.2)');rc(x,y+10,2,3-w,'#2b3050');rc(x+4,y+10,2,2+w,'#2b3050')}
function draw(tm){ctx.imageSmoothingEnabled=false;ctx.setTransform(1,0,0,1,0,0);for(let y=0;y<H;y++)for(let x=0;x<W;x++)tile(x,y,G[y][x],tm);ctx.setTransform(1,0,0,1,0,0);
[...A.map(o=>[o.y,0,o]),...V.map(o=>[o.py+.6,1,o])].sort((p,q)=>p[0]-q[0]).forEach(([,t,o])=>t?vis(o,tm):ani(o,tm));
for(let j=0;j<6;j++)for(let i=0;i<7;i++)if(!PL[j][i]){const x0=i*64,y0=j*48,by=tool=='land'&&adj(i,j);rc(x0,y0,64,48,'rgba(14,9,4,'+(by?.45:.62)+')');for(let k=0;k<64;k+=8)rc(x0+k,y0,1,48,'rgba(0,0,0,.18)');for(let k=0;k<48;k+=8)rc(x0,y0+k,64,1,'rgba(0,0,0,.18)');if(by){const cx=x0+32,cy=y0+26;rc(cx-1,cy,2,10,'#5e3a19');rc(cx-14,cy-12,28,13,'#5e3a19');rc(cx-13,cy-11,26,11,'#f0dca4');ctx.fillStyle='#3a2410';ctx.font='12px VT323,monospace';ctx.textAlign='center';ctx.fillText('$'+plotCost(),cx,cy-2);ctx.strokeStyle='#8fd14f';ctx.lineWidth=1;ctx.strokeRect(x0+1.5,y0+1.5,61,45)}}
const a=na();if(a>0){rc(0,0,W*T,H*T,`rgba(12,16,60,${a})`);for(const q of A)if(q.s==8){const g=.25*a/.55;ctx.fillStyle=`rgba(255,240,100,${g})`;ctx.beginPath();ctx.arc(q.x*T,q.y*T,10,0,7);ctx.fill();ctx.beginPath();ctx.arc(q.x*T,q.y*T,5,0,7);ctx.fill();rc(q.x*T-1,q.y*T+1,2,2,'#ffff9a')}}
if(hov&&hov[0]>=0&&hov[0]<W&&hov[1]>=0&&hov[1]<H){ctx.lineWidth=1;if(tool=='land'){ctx.strokeStyle='#f4b83a';ctx.strokeRect((hov[0]/4|0)*64+.5,(hov[1]/3|0)*48+.5,63,47)}else if(own(hov[0],hov[1])){ctx.strokeStyle=tool=='erase'?'#ff5c4a':'#fff';ctx.strokeRect(hov[0]*T+.5,hov[1]*T+.5,T-1,T-1)}}}
Object.assign(PAL,{R:'#a11',u:'#cfeaff',h:'#c98a50'});
const HS=[
['...kkk','..kwkk','.rrrrk','rkkrrk','rkkrrk','rrrrrk','rrrrrk','rrkkrk','rrkkrk','.rrrrk','..RRRk','....kk'],
['..k...','..kkkk','..kkkk','wwyyyy','wwwyyy','wwwkkk','.wyyyy','..kkkk','..yyyy','..kkkk','...yyy','....kk'],
['..k...','...k..','PP...k','PPPP.k','PwPPPk','PPPPPk','.PPPPk','.pPPPk','..pPPk','..pPPk','...pPk','....pk'],
['..k...','...k..','...ddd','...ddd','kk..dd','.k..dd','kk..dd','...ddd','..dddB','..ddBB','...ddd','....dd'],
['.....k','.....k','....kk','...kkk','..BBBd','.BBBBd','BBBBBd','BhhBBd','BhBBBd','BBBBBd','.BBBBd','..BBdd'],
['..k...','...lll','...kll','.l..ll','.ll.ll','..l.ll','...lll','...lll','..lGll','..lGGl','..GGGl','...GGl'],
['k.....','.k....','..kBBB','..BkBB','..BBBB','.BBBBd','kBhBBd','kkhBBd','k.BBBd','kk.BBd','..BBdd','...k.k'],
['...bbb','...bbb','uuuucb','ucuucb','.....b','uuuucb','ucuucb','.....b','.....c','.....b','.....c','.....b'],
['..k...','...kkk','...rrr','..rrrr','.kkkkd','.kkkkd','.kkkkd','..kkkd','..yyyy','..yyww','...yyw','....yy'],
['.k....','..k...','....ll','.....l','.kkk.l','k...kB','.....l','.kkk.l','k...kB','.....l','.kkk.l','k...kd']];
const VR=[[{},{r:'#f90',R:'#b60'},{r:'#fd3',R:'#ca0'}],[{},{y:'#f90'},{y:'#e8e2a8'}],[{},{P:'#36c',p:'#8cf'},{P:'#f80',p:'#fc4'}],[{},{d:'#222',B:'#555'},{d:'#a33',B:'#d66'}],[{},{B:'#2d4a3a',h:'#5a8a6a',d:'#142a1f'},{B:'#a14a2a',h:'#e08a5a',d:'#4a1a0a'}],[{},{l:'#b08a50',G:'#6a4a2a'},{l:'#f5b8d0',G:'#d27aa0'}],[{},{B:'#333',h:'#666',d:'#111'},{B:'#6a8a3a',h:'#9ac05a',d:'#2a3a12'}],[{},{b:'#c33',c:'#f77'},{b:'#2a9a4a',c:'#6ad88a'}],[{},{y:'#9f4'},{y:'#7ef'}],[{},{l:'#a07840',B:'#6a4a20'},{l:'#9a9a6a',B:'#6a6a44'}]];
const SPV=HS.map((rows,i)=>VR[i].map(vm=>{const m=rows.map(r=>r+[...r].reverse().join('')),c=document.createElement('canvas');c.width=c.height=14;const x=c.getContext('2d'),px=(a,b)=>!!(m[b]&&m[b][a]&&m[b][a]!='.');for(let j=-1;j<13;j++)for(let a=-1;a<13;a++)if(!px(a,j)&&(px(a+1,j)||px(a-1,j)||px(a,j+1)||px(a,j-1))){x.fillStyle='#1a1208';x.fillRect(a+1,j+1,1,1)}m.forEach((r,j)=>[...r].forEach((ch,a)=>{if(ch!='.'){x.fillStyle=vm[ch]||PAL[ch];x.fillRect(a+1,j+1,1,1)}}));return c}));
function ani(a,tm){const s=SP[a.s],mv=Math.hypot(a.tx-a.x,a.ty-a.y)>.05,b=s[9]?Math.round(Math.sin(tm*8+a.id)*1.5):(mv&&Math.sin(tm*12+a.id)>0?-1:0),x=Math.round(a.x*T),y=Math.round(a.y*T),w=s[9]&&Math.sin(tm*30+a.id)>0?12:14,q=Math.PI/4;if(mv)a.ang=Math.round((Math.atan2(a.ty-a.y,a.tx-a.x)+Math.PI/2)/q)*q;ctx.fillStyle='rgba(0,0,0,.2)';ctx.fillRect(x-4,y+(s[9]?7:5),8,2);ctx.save();ctx.translate(x,y+b);ctx.rotate(a.ang||0);ctx.drawImage(SPV[a.s][a.v|0],-w/2,-7,w,14);ctx.restore();if(a.sat<30||a.hap<35){rc(x+5,y-10,2,3,'#e5533d');rc(x+5,y-6,2,1,'#e5533d')}}
function stg(a){if(a.hrs==null)return 2;const l=SP[a.s][7],L=Math.max(1,Math.round(l*.12))*24,Q=L+Math.max(1,Math.round(l*.08))*24;return a.hrs<L?0:a.hrs<Q?1:2}
const HL=['...aaa','..akaa','..aaaa','..bbbb','kaaaaa','.aaaaa','.bbbbb','kaaaaa','.aaaaa','..bbbb','..aaaa','...bbb'],HP=['.....k','.....k','...bbb','..baaa','.bbbbb','.bwaaa','.baaaa','.bbbbb','..baaa','..bbbb','...bbb','....bb'];
const bake=(rows,vm)=>{const m=rows.map(r=>r+[...r].reverse().join('')),c=document.createElement('canvas');c.width=c.height=14;const x=c.getContext('2d'),px=(a,b)=>!!(m[b]&&m[b][a]&&m[b][a]!='.');for(let j=-1;j<13;j++)for(let a=-1;a<13;a++)if(!px(a,j)&&(px(a+1,j)||px(a-1,j)||px(a,j+1)||px(a,j-1))){x.fillStyle='#1a1208';x.fillRect(a+1,j+1,1,1)}m.forEach((r,j)=>[...r].forEach((ch,a)=>{if(ch!='.'){x.fillStyle=vm[ch]||PAL[ch];x.fillRect(a+1,j+1,1,1)}}));return c};
const mixc=(c,m,t)=>c.map((v,i)=>Math.round(v+(m[i]-v)*t)),rgb=c=>'rgb('+c.join()+')';
const SPL=SPV.map(vs=>vs.map(cv=>{const d=cv.getContext('2d').getImageData(0,0,14,14).data,n={};for(let p=0;p<d.length;p+=4){if(d[p+3]<200)continue;const r=d[p],g=d[p+1],b=d[p+2];if(r==26&&g==18&&b==8)continue;if(r>225&&g>225&&b>225)continue;const k=r+','+g+','+b;n[k]=(n[k]||0)+1}const base=Object.keys(n).sort((u,v)=>n[v]-n[u])[0].split(',').map(Number),mk=(rows,m,t)=>{const a=mixc(base,m,t);return bake(rows,{a:rgb(a),b:rgb(mixc(a,[0,0,0],.3))})};return [mk(HL,[232,232,150],.4),mk(HP,[138,106,64],.5)]}));
function ani(a,tm){if(a.eg>0){drawEgg(a,tm);return}const s=SP[a.s],st=stg(a),mv=Math.hypot(a.tx-a.x,a.ty-a.y)>.05,x=Math.round(a.x*T),y=Math.round(a.y*T),q=Math.PI/4;
if(st==1){ctx.fillStyle='rgba(0,0,0,.2)';ctx.fillRect(x-3,y+6,6,2);ctx.save();ctx.translate(x,y-8);ctx.rotate(Math.sin(tm*2+a.id)*.09);ctx.drawImage(SPL[a.s][a.v|0][1],-7,0);ctx.restore()}
else{const lv=st==0,b=!lv&&s[9]?Math.round(Math.sin(tm*8+a.id)*1.5):(mv&&Math.sin(tm*12+a.id)>0?-1:0),w=!lv&&s[9]&&Math.sin(tm*30+a.id)>0?12:14;if(mv)a.ang=Math.round((Math.atan2(a.ty-a.y,a.tx-a.x)+Math.PI/2)/q)*q;ctx.fillStyle='rgba(0,0,0,.2)';ctx.fillRect(x-4,y+(!lv&&s[9]?7:5),8,2);ctx.save();ctx.translate(x,y+b);ctx.rotate(a.ang||0);ctx.drawImage(lv?SPL[a.s][a.v|0][0]:SPV[a.s][a.v|0],-w/2,-7,w,14);ctx.restore()}
if(a.sat<30||a.hap<35){rc(x+5,y-10,2,3,'#e5533d');rc(x+5,y-6,2,1,'#e5533d')}}
Object.assign(PAL,{s:'#ff9a8a',W:'#f4ecd0',V:'#d6c79a',T:'#c9a66b',N:'#2f6f2a',L:'#d6ff5a',S:'#555566',E:'#6a6a3a',O:'#e8801a',D:'#4a2f18'});
VR[0][1].s='#ffc27a';VR[0][2].s='#fff09a';
const NHS=[
['...kkk','..kwkk','.kwkkk','.rssrk','rrkkrk','rrkkrk','rrrrrk','rkkrrk','rkkrrk','.RrrRk','.RRRRk','..RRkk'],
['.k....','..kwkk','..kkkk','uuyyyy','uwuyyy','uuukkk','wuyyyy','.ukkkk','..yyyy','..kkkk','...yyk','....kk'],
['..k...','...k..','kPP..k','kPPP.k','kPwPPk','kPPpPk','.kPPPk','.kpPPk','..kpPk','..kpPk','...kpk','....kk'],
['..k...','...k..','...ddd','...dBd','kk..dd','.k..Bd','kk..dd','..kddd','..dBBd','.ddBBd','.dddBd','..dddd'],
['.....k','.....k','....kk','...kkk','..hBBd','.BhBBd','BBhBBd','BhhBBd','BhBBBd','BBBBBd','.BBBdd','..Bddd'],
['..k...','...lll','..kwll','kl..ll','kll.ll','.ll.ll','...lll','..GGll','..GGGl','..GGGl','...GGl','....Gl'],
['k.....','.k....','..kBBB','..BkBB','..BBBB','.BhhBd','kBhBBd','kkBhBd','k.BBBd','kk.BBd','..BBdd','...k.k'],
['...bbb','..cbbb','uuuucb','ucucub','uuuucb','.uuucb','ucucub','uuuucb','.....b','.....c','.....b','.....b'],
['..k...','...kkk','..rrrr','.rrkkk','.kkkkd','.kdkkd','.kdkkd','..kkkd','..yyyy','.yywwy','..yyww','...yyy'],
['.k....','..k...','....ll','.....B','.kkk.l','k...kB','.....l','.kkk.l','k...kB','.....l','.kkk.l','k...kd']];
const LPD=[
[['...SSS','..SSSS','kSaSSS','.SSSSS','kSSaSS','.SSSSS','kSaSSS','.SSSSS','..SaSS','..SSSS','...SSS','....SS'],['...bbb','..bsaa','.bsaaa','baakaa','baaaaa','bakaaa','baaaaa','.baaka','.bbaaa','..bbaa','...bbb','....kk']],
[['...VVV','..VkWW','.VWWWW','.VVVVV','VWWWWW','VWWWWW','VVVVVV','VWWWWW','.WWWWW','.VVVVV','..WWWW','...VVV'],['...WWW','..kWWW','..WWWW','.VWWWW','.aaaaa','.VWWWW','.aaaaa','..VWWW','..aaaa','..VWWW','...VVW','....VV']],
[['...kkk','..kwkk','..gggg','.gggaa','.NNNNN','.gggaa','.NNNNN','.gggaa','.NNNNN','..ggaa','..NNNN','...yyy'],['.....k','....kk','..lggg','.lgggg','.ggggg','.gyggg','.gggNg','..gggg','..NNgg','...NNg','....NN','.....N']],
[['......','......','....DD','...VWW','..VWWW','..VWWW','..VVVV','..VWWW','...VWW','....VV','......','......'],['......','..TTTT','.TWTTT','.VVVVV','.TTTTT','.TWTTT','.VVVVV','.TTTTT','..TTTT','...TTT','......','......']],
[['...DDD','..DDDD','.WWWWW','VWOWWW','.WWWWW','VWOWWW','.WWWWW','VWOWWW','.WWWWW','.VWWWW','..VWWW','...VVV'],['.....D','.....D','...OOO','..OOOO','.OOhOO','.OhOOO','.OOOOT','.OhOOT','.OOOOT','..OOOT','..OOTT','...TTT']],
[['......','..k...','...lll','..kwll','kl..ll','.l..ll','...lll','...lgl','...ggl','..lggg','....gl','......'],['.....B','.....B','...TTT','..TVTT','..TTTT','.TTVTT','.TTTTT','.TVTTT','..TTTT','..TVTT','...TTT','.....B']],
[['k.....','.k....','..kaaa','..akaa','..aaaa','.kaaaa','kkbbbb','k.aaaa','kkbbbb','k.aaaa','..bbbb','...k.k'],['.k....','..k...','..VWWW','..WaWW','..WWWW','.VWWWW','.VWVWW','.WWWWW','..VVVV','..WWWW','..VVVV','...WWW']],
[['..EEEE','.EwEEE','..EEEE','k.EEEE','kkEEEE','.kEaEE','k.EEEE','kkEEEE','..EEaE','...EEE','...EEE','..k.k.'],['.....g','.....g','...EEE','..EEEE','..EwEE','..EEEE','..kEEE','..kEaE','..EEaE','...EEE','.....g','.....g']],
[['...kkk','..SSSS','LSSSSS','kkkkkk','LSSSSS','kkkkkk','LSSSSS','kkkkkk','LSSSSS','..SSSS','...LLL','....LL'],['...WWW','..kWWW','..WWWW','.WWWWW','.WLLLL','.WLLLL','.WLLLL','.WWLLL','..WLLL','..WWLL','...WWL','....WW']],
[['......','.k....','..k.ll','.....l','.kk..l','.....B','.kk..l','.....B','..k..l','.....l','......','......'],['......','......','...TTT','..TTTT','..DDDD','.DhDDD','.DDDDh','.DhDDD','.DDDDD','..DDDD','...DDD','......']]];
const MK=['r','y','P','d','B','l','B','b','y','l'];
const hx=h=>{if(h.length==4)h='#'+[...h.slice(1)].map(c=>c+c).join('');return [1,3,5].map(i=>parseInt(h.slice(i,i+2),16))};
SPV.forEach((vs,i)=>vs.forEach((_,j)=>{vs[j]=bake(NHS[i],VR[i][j]);const vm={...VR[i][j]},a=vm[MK[i]]||PAL[MK[i]];vm.a=a;vm.b=rgb(mixc(hx(a),[0,0,0],.3));if(i==8)vm.L=vm.y||PAL.y;SPL[i][j]=[bake(LPD[i][0],vm),bake(LPD[i][1],vm)]}));
function deco(d,hh){if(d!='pond')return deco0(d,hh);const m=ctx.getTransform(),x=Math.round(m.e/T),y=Math.round(m.f/T),w=(a,b)=>{const q=(G[b]||[])[a];return !!q&&(q.d=='pond'||q.k=='w')},n=w(x,y-1),s=w(x,y+1),e=w(x+1,y),o=w(x-1,y),sd='#d9c48a',wt='#3a86d8';
const box=(l,t,r,b,c)=>{rc(l+(o?0:1),t,r-l-(o?0:1)-(e?0:1),b-t,c);rc(l,t+(n?0:1),r-l,b-t-(n?0:1)-(s?0:1),c)};
box(o?0:1,n?0:1,e?16:15,s?16:15,sd);
if(n&&o&&!w(x-1,y-1))rc(0,0,2,2,sd);if(n&&e&&!w(x+1,y-1))rc(14,0,2,2,sd);if(s&&o&&!w(x-1,y+1))rc(0,14,2,2,sd);if(s&&e&&!w(x+1,y+1))rc(14,14,2,2,sd);
box(o?0:3,n?0:3,e?16:13,s?16:13,wt);
if(n&&o&&!w(x-1,y-1))rc(0,0,3,3,sd);if(n&&e&&!w(x+1,y-1))rc(13,0,3,3,sd);if(s&&o&&!w(x-1,y+1))rc(0,13,3,3,sd);if(s&&e&&!w(x+1,y+1))rc(13,13,3,3,sd);
if(!s)rc(o?0:3,12,16-(o?0:3)-(e?0:3),1,'#2f74c4');const q=(tm*4+hh)%9|0;rc(3+q,5,3,1,'#9fd0f8');rc(3+(q+4)%9,9,3,1,'#6aa9e6')}
const sk=n=>'tzs_s'+n;
function slotInfo(n){try{const r=localStorage.getItem(sk(n));return r?JSON.parse(r):null}catch(e){return null}}
function save(m){try{localStorage.setItem(sk(slot),JSON.stringify({G,A,money,rep,day,hour,tv,staff,price,U,mi,idc,PL}));localStorage.setItem('tzs_cur',slot);if(m)msg('Jogo salvo no espaço '+slot+'!')}catch(e){if(m)msg('Não foi possível salvar.')}}
function apply(d){({G,A,money,rep,day,hour,tv,staff,price,U,mi,idc,PL}=d);if(!PL||!PL.length)initPL();V=[];inc=0;exp=0;tool='insp';calc()}
function load(){try{const c=localStorage.getItem('tzs_cur');if(c)slot=+c;let r=localStorage.getItem(sk(slot));if(!r&&!c)r=localStorage.getItem('tzs1');if(!r)return 0;apply(JSON.parse(r));return 1}catch(e){return 0}}
function slotsHTML(){let h='<h3>Jogos salvos</h3>';for(let n=1;n<=3;n++){const d=slotInfo(n),cur=n==slot;h+='<div class=row><div>'+(cur?'▶ ':'')+'<b>Espaço '+n+'</b>'+(cur?' (jogando)':'')+'<br><small>'+(d?'Dia '+d.day+' · $'+Math.floor(d.money)+' · '+d.A.length+' insetos · '+LV[Math.min(4,d.rep/20|0)]:'Vazio')+'</small></div>'+(d&&!cur?'<button onclick="goSlot('+n+')">Carregar</button>':'')+(cur?'<button onclick="newSlot('+n+')">Reiniciar</button>':!d?'<button onclick="newSlot('+n+')">Novo jogo</button>':'')+(d?'<button onclick="delSlot('+n+')">🗑️</button>':'')+'</div>'}return h+'<p><small>O jogo salva sozinho no espaço atual a cada dia.</small></p>'}
function goSlot(n){save();const d=slotInfo(n);if(!d)return;slot=n;try{localStorage.setItem('tzs_cur',n)}catch(e){}apply(d);msg('Espaço '+n+' carregado.');P()}
function newSlot(n){if(slotInfo(n)&&!confirm('Apagar o espaço '+n+' e começar um novo zoológico?'))return;if(n!=slot)save();slot=n;newGame();try{localStorage.setItem('tzs_cur',n)}catch(e){}save();msg('Novo zoológico criado no espaço '+n+'!');closeP()}
function delSlot(n){if(!confirm('Apagar o espaço '+n+'?'))return;try{localStorage.removeItem(sk(n))}catch(e){}if(n==slot){newGame();save()}P()}
function resizeWorld(){if(!WC)WC=document.createElement('canvas');WC.width=W*T;WC.height=H*T;wx=WC.getContext('2d');wx.imageSmoothingEnabled=false}
function clampCam(){const vw=448/zm,vh=288/zm;cam.x=W*T<=vw?(W*T-vw)/2:C(cam.x,0,W*T-vw);cam.y=H*T<=vh?(H*T-vh)/2:C(cam.y,0,H*T-vh)}
function camHome(){zm=1;cam.x=0;cam.y=(GY+.5)*T-144;clampCam()}
function zoomAt(f,p){p=p||[224,144];const a=cam.x+p[0]/zm,b=cam.y+p[1]/zm;zm=C(zm*f,.4,4);cam.x=a-p[0]/zm;cam.y=b-p[1]/zm;clampCam()}
function spt(e){const r=cv.getBoundingClientRect();return [(e.clientX-r.left)/r.width*448,(e.clientY-r.top)/r.height*288]}
function hv(e){const p=spt(e);hov=[Math.floor((cam.x+p[0]/zm)/T),Math.floor((cam.y+p[1]/zm)/T)]}
function pdown(e){try{cv.setPointerCapture(e.pointerId)}catch(_){}const p=spt(e);PT.set(e.pointerId,p);hv(e);if(PT.size>=2){mode='gest';down=0;const [a,b]=[...PT.values()];gst={d:Math.hypot(a[0]-b[0],a[1]-b[1])||1,m:[(a[0]+b[0])/2,(a[1]+b[1])/2]};return}pg={x:p[0],y:p[1],cx:cam.x,cy:cam.y};if(e.button>0||tool=='pan')mode='pan';else if(['path','hab','erase'].includes(tool)){mode='build';down=1;use(hov[0],hov[1],1)}else mode='tap'}
function pmove(e){const p=spt(e);if(PT.has(e.pointerId))PT.set(e.pointerId,p);hv(e);if(PT.size>=2&&gst){const [a,b]=[...PT.values()],d=Math.hypot(a[0]-b[0],a[1]-b[1])||1,m=[(a[0]+b[0])/2,(a[1]+b[1])/2];zoomAt(d/gst.d,m);cam.x-=(m[0]-gst.m[0])/zm;cam.y-=(m[1]-gst.m[1])/zm;clampCam();gst={d,m};return}if(PT.size!=1||!pg)return;if(mode=='pan'||(mode=='tap'&&Math.hypot(p[0]-pg.x,p[1]-pg.y)>6)){mode='pan';cam.x=pg.cx-(p[0]-pg.x)/zm;cam.y=pg.cy-(p[1]-pg.y)/zm;clampCam()}else if(mode=='build'&&down)use(hov[0],hov[1])}
function pup(e){hv(e);PT.delete(e.pointerId);if(mode=='tap'&&PT.size==0&&hov)use(hov[0],hov[1],1);if(PT.size<2)gst=null;if(PT.size==0){mode=null;down=0}}
addEventListener('pointercancel',e=>{PT.delete(e.pointerId);mode=null;down=0;gst=null});cv.addEventListener('contextmenu',e=>e.preventDefault());
cv.addEventListener('wheel',e=>{e.preventDefault();zoomAt(e.deltaY<0?1.15:1/1.15,spt(e))},{passive:false});
addEventListener('keydown',e=>{const k=e.key,d=28/zm;if(k=='ArrowLeft'||k=='a')cam.x-=d;else if(k=='ArrowRight'||k=='d')cam.x+=d;else if(k=='ArrowUp'||k=='w')cam.y-=d;else if(k=='ArrowDown'||k=='s')cam.y+=d;else if(k=='+'||k=='=')zoomAt(1.2);else if(k=='-')zoomAt(1/1.2);else return;if(k.startsWith('Arrow'))e.preventDefault();clampCam()});
$('zi').onclick=()=>zoomAt(1.3);$('zo').onclick=()=>zoomAt(1/1.3);$('pt').onclick=()=>{tool=tool=='pan'?'insp':'pan';closeP();msg(tool=='pan'?'Câmera: arraste para mover o mapa. Toque em ✋ de novo para sair.':'Modo inspecionar')};
function initPL(){const pw=Math.ceil(W/4),ph=Math.ceil(H/3),jc=GY/3|0;PL=[];for(let j=0;j<ph;j++){PL[j]=[];for(let i=0;i<pw;i++)PL[j][i]=i<=3&&j>=jc-1&&j<=jc+2}}
function plotCost(){return Math.min(2500,150+40*(pc()-16))}
function gen(w,h,seed){W=w;H=h;GY=Math.floor(h/2)-1;const s0=seed||7;let s=s0;const rn=()=>(s=s*16807%2147483647)/2147483647,hs=(a,b)=>{let t=Math.imul(a,374761393)+Math.imul(b,668265263)+Math.imul(s0,1274126177)|0;t=Math.imul(t^(t>>>13),1274126177);return((t^(t>>>16))>>>0)/4294967295};
G=[];for(let y=0;y<h;y++){G[y]=[];for(let x=0;x<w;x++)G[y][x]={k:'g',d:0}}
const nl=0;for(let i=0;i<nl;i++){const cx=rn()*w,cy=rn()*h,rx=2+rn()*(w>50?7:3),ry=2+rn()*(h>30?5:3);if(cx<20&&Math.abs(cy-GY)<9)continue;for(let y=Math.max(0,(cy-ry-1)|0);y<Math.min(h,cy+ry+2);y++)for(let x=Math.max(0,(cx-rx-1)|0);x<Math.min(w,cx+rx+2);x++){const dx=(x-cx)/rx,dy=(y-cy)/ry;if(dx*dx+dy*dy<.8+rn()*.3)G[y][x].k='w'}}

for(let x=0;x<=11;x++){G[GY][x].k='p';G[GY][x].d=0}for(let y=GY-3;y<=GY+2;y++){G[y][11].k='p';G[y][11].d=0}
const jc=GY/3|0;for(let y=0;y<h;y++)for(let x=0;x<w;x++){const c=G[y][x];if(c.k!='g')continue;if((y>=GY-1&&y<=GY+1&&x<13)||(x>=10&&x<=12&&y>=GY-4&&y<=GY+3))continue;const st=x<16&&y>=(jc-1)*3&&y<(jc+3)*3,nz=.5*hs(x>>3,y>>3)+.5*hs(x>>2,y>>2);let d=0;if(nz>.62){if(rn()<.5)d='tree'}else if(rn()<.1){const q=rn();d=q<.4?'flower':q<.7?'rock':'tree'}if(st&&rn()<.5)d=0;c.d=d}
G[GY-1][1].k='g';G[GY-1][1].d='booth';G[GY][0].d='gate'}
function newGame(w,h,seed){gen(w||28,h||18,seed||7);A=[];V=[];money=600;rep=10;day=1;hour=8;tv=0;staff={t:0,f:0};price=8;U=18;initPL();mi=0;inc=0;exp=0;tool='insp';resizeWorld();calc();camHome()}
function save(m){try{localStorage.setItem(sk(slot),JSON.stringify({W,H,GY,G,A,money,rep,day,hour,tv,staff,price,U,mi,idc,PL}));localStorage.setItem('tzs_cur',slot);if(m)msg('Jogo salvo no espaço '+slot+'!')}catch(e){if(m)msg('Não foi possível salvar (mapa grande demais?).')}}
function apply(d){W=d.W||28;H=d.H||18;GY=d.GY||8;({G,A,money,rep,day,hour,tv,staff,price,U,mi,idc,PL}=d);if(!PL||!PL.length)initPL();V=[];G.forEach(r=>r.forEach(c=>{if(c.k=='w')c.k='g'}));inc=0;exp=0;tool='insp';resizeWorld();calc();camHome()}
function askC(t,fn){cf={t,fn};P()}
function cfY(){const f=cf.fn;cf=null;f();P()}
function slotsHTML(){let h='<h3>Jogos salvos</h3>';if(cf)h+='<div class=row style="background:#5a2a1a"><div>⚠️ '+cf.t+'</div><button onclick="cfY()">Sim</button><button onclick="cf=null;P()">Não</button></div>';for(let n=1;n<=3;n++){const d=slotInfo(n),cur=n==slot;h+='<div class=row><div>'+(cur?'▶ ':'')+'<b>Espaço '+n+'</b>'+(cur?' (jogando)':'')+'<br><small>'+(d?'Dia '+d.day+' · $'+Math.floor(d.money)+' · '+d.A.length+' insetos · mapa '+(d.W||28)+'×'+(d.H||18):'Vazio')+'</small></div>'+(d&&!cur?'<button onclick="goSlot('+n+')">Carregar</button>':'')+'<button onclick="newSlot('+n+')">'+(d?'Novo mapa':'Novo jogo')+'</button>'+(d?'<button onclick="delSlot('+n+')">🗑️ Apagar</button>':'')+'</div>'}return h+'<p><small>O jogo salva sozinho no espaço atual a cada 15 segundos e ao fechar.</small></p>'}
function newHTML(){const b=(t,w,h,r)=>'<button class=t onclick="startNew('+w+','+h+','+r+')">'+t+'<br><small>'+w+'×'+h+' quadrados<br>'+Math.ceil(w/4)*Math.ceil(h/3)+' lotes</small></button>';return '<h3>Novo zoológico — espaço '+ns+'</h3><div class=g>'+b('🌱 Clássico',28,18,0)+b('🎲 Pequeno',40,27,1)+b('🎲 Grande',64,42,1)+b('🏔️ Gigante',100,100,1)+b('🌌 Gigantesco',160,120,1)+'</div><p><small>Os mapas 🎲 geram lagos, florestas e flores aleatórios. Se este espaço já tem jogo, ele será substituído.</small></p>'}
function newSlot(n){ns=n;pn='newg';P()}
function startNew(w,h,r){if(ns!=slot)save();slot=ns;newGame(w,h,r?(Date.now()%2147483000)+1:7);try{localStorage.setItem('tzs_cur',slot)}catch(e){}save();closeP();msg('Novo zoológico no espaço '+slot+'! Arraste para mover a câmera; use ➕➖ ou a roda do mouse para zoom.')}
function delSlot(n){askC('Apagar o espaço '+n+' para sempre?',()=>{try{localStorage.removeItem(sk(n))}catch(e){}if(n==slot){newGame();msg('Espaço apagado. Começando um jogo novo.')}else msg('Espaço '+n+' apagado.')})}
function draw(tm){ctx=wx;ctx.imageSmoothingEnabled=false;clampCam();const vw=448/zm,vh=288/zm,x0=Math.max(0,Math.floor(cam.x/T)),y0=Math.max(0,Math.floor(cam.y/T)),x1=Math.min(W-1,Math.ceil((cam.x+vw)/T)),y1=Math.min(H-1,Math.ceil((cam.y+vh)/T));
for(let y=y0;y<=y1;y++)for(let x=x0;x<=x1;x++)tile(x,y,G[y][x],tm);ctx.setTransform(1,0,0,1,0,0);
const ins=(px,py)=>px>=x0-1&&px<=x1+2&&py>=y0-1&&py<=y1+2;[...A.filter(o=>ins(o.x,o.y)).map(o=>[o.y,0,o]),...V.filter(o=>ins(o.px,o.py)).map(o=>[o.py+.6,1,o])].sort((p,q)=>p[0]-q[0]).forEach(([,t,o])=>t?vis(o,tm):ani(o,tm));
for(let j=0;j<PL.length;j++)for(let i=0;i<PL[j].length;i++){if(PL[j][i]||i*4>x1||i*4+4<x0||j*3>y1||j*3+3<y0)continue;const px0=i*64,py0=j*48,by=tool=='land'&&adj(i,j);rc(px0,py0,64,48,'rgba(14,9,4,'+(by?.45:.62)+')');for(let k=0;k<64;k+=8)rc(px0+k,py0,1,48,'rgba(0,0,0,.18)');for(let k=0;k<48;k+=8)rc(px0,py0+k,64,1,'rgba(0,0,0,.18)');if(by){const cx=px0+32,cy=py0+26;rc(cx-1,cy,2,10,'#5e3a19');rc(cx-14,cy-12,28,13,'#5e3a19');rc(cx-13,cy-11,26,11,'#f0dca4');ctx.fillStyle='#3a2410';ctx.font='12px VT323,monospace';ctx.textAlign='center';ctx.fillText('$'+plotCost(),cx,cy-2);ctx.strokeStyle='#8fd14f';ctx.lineWidth=1;ctx.strokeRect(px0+1.5,py0+1.5,61,45)}}
const a=na();if(a>0){rc(x0*T,y0*T,(x1-x0+1)*T,(y1-y0+1)*T,`rgba(12,16,60,${a})`);for(const q of A)if(q.s==8&&ins(q.x,q.y)){const g=.25*a/.55;ctx.fillStyle=`rgba(255,240,100,${g})`;ctx.beginPath();ctx.arc(q.x*T,q.y*T,10,0,7);ctx.fill();ctx.beginPath();ctx.arc(q.x*T,q.y*T,5,0,7);ctx.fill();rc(q.x*T-1,q.y*T+1,2,2,'#ffff9a')}}
if(hov&&hov[0]>=0&&hov[0]<W&&hov[1]>=0&&hov[1]<H&&tool!='pan'){ctx.lineWidth=1;if(tool=='land'){ctx.strokeStyle='#f4b83a';ctx.strokeRect((hov[0]/4|0)*64+.5,(hov[1]/3|0)*48+.5,63,47)}else if(own(hov[0],hov[1])){ctx.strokeStyle=tool=='erase'?'#ff5c4a':'#fff';ctx.strokeRect(hov[0]*T+.5,hov[1]*T+.5,T-1,T-1)}}
ctx=SCR;SCR.setTransform(1,0,0,1,0,0);SCR.imageSmoothingEnabled=false;SCR.fillStyle='#0b0805';SCR.fillRect(0,0,448,288);SCR.drawImage(WC,cam.x,cam.y,vw,vh,0,0,448,288)}
const EG=[10,12,14,10,24,18,12,16,14,20];
const EGR=[
['......','......','.....o','....oy','..o.oy','.oy.oy','.oy.oy','..o.oy','.ggggg','gggGgg','.ggggg','......'],
['......','...yyy','..yooo','.yoooW','.yooWW','.yooWW','.yoooW','..yooo','...yyy','......','......','......'],
['......','......','......','..lw.l','..ll.l','.ggggg','gggGgg','gggggg','.ggGgg','..gggg','......','......'],
['......','......','....WW','..WWWV','.WWVWW','.WVWWW','..WWVW','....WW','......','......','......','......'],
['......','......','...VVV','..VWWW','.VWWWW','.VWWWW','.VWWWW','..VWWW','dddddd','dBdBdd','dddddd','......'],
['......','......','......','.WW.WW','.WW.WW','.VV.VV','BBBBBB','BBBBBB','......','......','......','......'],
['......','......','......','.V..V.','.W..W.','.W..W.','dddddd','dTdTdd','dddddd','......','......','......'],
['......','......','...ccc','..cuuu','.cukuu','.cuuku','.cukuu','..cuuu','...ccc','.....g','.....g','.....g'],
['......','......','......','..LW.L','..LL.L','......','.NNNNN','NNgNgN','.NNgNN','..NNNN','......','......'],
['......','......','......','.lk..l','.ll..l','..B..B','......','......','......','......','......','......']];
const SPE=EGR.map((rows,i)=>VR[i].map(v0=>{const vm={...v0};vm.a=vm[MK[i]]||PAL[MK[i]];if(i==8)vm.L=vm.y||PAL.y;return bake(rows,vm)}));
function drawEgg(a,tm){const x=Math.round(a.x*T),y=Math.round(a.y*T),w=a.eg<=2&&Math.sin(tm*14+a.id)>0?1:0;ctx.fillStyle='rgba(0,0,0,.2)';ctx.fillRect(x-4,y+4,8,2);ctx.drawImage(SPE[a.s][a.v|0],x-7+w,y-8)}
SP.push(['Escorpião','Predador noturno e resistente. Esconde-se entre pedras e atrai visitantes à noite.','rock','Grilos e baratas',200,3,5,25,3,0,[]]);
VR.push([{},{B:'#2a2a38',h:'#555570',d:'#0f0f18'},{B:'#d1a85c',h:'#f0d79a',d:'#8a6a30'}]);MK.push('B');EG.push(22);
const NS=['.dd...','dBBd..','dBBBd.','.ddBBd','..dBBh','.kdBhB','.kdBBB','kkdBhB','..dBBd','...ddB','....dB','.....R'],NL=['......','......','.VV...','VWWV..','.VVWWV','..VWWW','.kVWaW','.kVWWW','..VWaV','...VWW','....VW','.....a'],NP=['......','......','...VVV','..VWWW','.VWkWW','.VWWWW','.VVVVV','.VWWWW','..VWWV','..VVVV','...VVa','.....a'],NE=['......','......','......','...WW.','..WWWV','.WWVWW','.TWWTW','TTTTTT','TSTTST','TTTTTT','......','......'];
SPV.push(VR[10].map(v=>bake(NS,v)));
SPL.push(VR[10].map(v0=>{const vm={...v0},a=vm.B||PAL.B;vm.a=a;vm.b=rgb(mixc(hx(a),[0,0,0],.3));return [bake(NL,vm),bake(NP,vm)]}));
SPE.push(VR[10].map(v0=>{const vm={...v0};vm.a=vm.B||PAL.B;return bake(NE,vm)}));
setInterval(()=>{if(G)save()},15000);document.addEventListener('visibilitychange',()=>{if(document.hidden)save()});addEventListener('pagehide',()=>save());
$('dl').onclick=async()=>{let d=null;try{d=window.claude?await claude.use('downloads'):null}catch(e){}if(d){try{await d.save({filename:'tycoon-zoo.html',data:SRC});msg('Download do jogo iniciado!')}catch(e){msg(e&&e.code=='declined'?'Download cancelado.':'Não foi possível baixar aqui.')}return}const a=document.createElement('a');a.href=URL.createObjectURL(new Blob([SRC],{type:'text/html'}));a.download='tycoon-zoo.html';document.body.appendChild(a);a.click();a.remove();msg('Baixando o jogo...')};
setInterval(()=>{if(pn=='reg'||pn=='stats'||pn=='staff')P()},1500);
requestAnimationFrame(loop);
</script></body></html>
