# MIXUULS.github.io
<!doctype html>
<html>
<head>
<meta charset="utf-8">
<title>H5,200行代码实现粒子漩涡特效</title>
<style>
html,body{
	margin:0px;
	width:100%;
	height:100%;
	overflow:hidden;
  background:#000;
}
#canvas{
	position:absolute;
	width:100%;
	height:100%;
}
</style>
</head>
<body>
<canvas id="canvas"></canvas>
<script>
function project3D(x,y,z,vars){
	var p,d;
	x-=vars.camX;
	y-=vars.camY-8;
	z-=vars.camZ;
	p=Math.atan2(x,z);
	d=Math.sqrt(x*x+z*z);
	x=Math.sin(p-vars.yaw)*d;
	z=Math.cos(p-vars.yaw)*d;
	p=Math.atan2(y,z);
	d=Math.sqrt(y*y+z*z);
	y=Math.sin(p-vars.pitch)*d;
	z=Math.cos(p-vars.pitch)*d;
	var rx1=-1000;
	var ry1=1;
	var rx2=1000;
	var ry2=1;
	var rx3=0;
	var ry3=0;
	var rx4=x;
	var ry4=z;
	var uc=(ry4-ry3)*(rx2-rx1)-(rx4-rx3)*(ry2-ry1);
	var ua=((rx4-rx3)*(ry1-ry3)-(ry4-ry3)*(rx1-rx3))/uc;
	var ub=((rx2-rx1)*(ry1-ry3)-(ry2-ry1)*(rx1-rx3))/uc;
	if(!z)z=0.000000001;
	if(ua>0&&ua<1&&ub>0&&ub<1){
		return {
			x:vars.cx+(rx1+ua*(rx2-rx1))*vars.scale,
			y:vars.cy+y/z*vars.scale,
			d:(x*x+y*y+z*z)
		};
	}else{
		return { d:-1 };
	}
}
function elevation(x,y,z){
	var dist = Math.sqrt(x*x+y*y+z*z);
	if(dist && z/dist>=-1 && z/dist <=1) return Math.acos(z / dist);
	return 0.00000001;
}
function rgb(col){
	col += 0.000001;
	var r = parseInt((0.5+Math.sin(col)*0.5)*16);
	var g = parseInt((0.5+Math.cos(col)*0.5)*16);
	var b = parseInt((0.5-Math.sin(col)*0.5)*16);
	return "#"+r.toString(16)+g.toString(16)+b.toString(16);
}
function interpolateColors(RGB1,RGB2,degree){
	var w2=degree;
	var w1=1-w2;
	return [w1*RGB1[0]+w2*RGB2[0],w1*RGB1[1]+w2*RGB2[1],w1*RGB1[2]+w2*RGB2[2]];
}
function rgbArray(col){
	col += 0.000001;
	var r = parseInt((0.5+Math.sin(col)*0.5)*256);
	var g = parseInt((0.5+Math.cos(col)*0.5)*256);
	var b = parseInt((0.5-Math.sin(col)*0.5)*256);
	return [r, g, b];
}
function colorString(arr){
	var r = parseInt(arr[0]);
	var g = parseInt(arr[1]);
	var b = parseInt(arr[2]);
	return "#"+("0" + r.toString(16) ).slice (-2)+("0" + g.toString(16) ).slice (-2)+("0" + b.toString(16) ).slice (-2);
}
function process(vars){
	if(vars.points.length<vars.initParticles) for(var i=0;i<5;++i) spawnParticle(vars);
	var p,d,t;	
	p = Math.atan2(vars.camX, vars.camZ);
	d = Math.sqrt(vars.camX * vars.camX + vars.camZ * vars.camZ);
	d -= Math.sin(vars.frameNo / 80) / 25;
	t = Math.cos(vars.frameNo / 300) / 165;
	vars.camX = Math.sin(p + t) * d;
	vars.camZ = Math.cos(p + t) * d;
	vars.camY = -Math.sin(vars.frameNo / 220) * 15;
	vars.yaw = Math.PI + p + t;
	vars.pitch = elevation(vars.camX, vars.camZ, vars.camY) - Math.PI / 2;	
	var t;
	for(var i=0;i<vars.points.length;++i){		
		x=vars.points[i].x;
		y=vars.points[i].y;
		z=vars.points[i].z;
		d=Math.sqrt(x*x+z*z)/1.0075;
		t=.1/(1+d*d/5);
		p=Math.atan2(x,z)+t;
		vars.points[i].x=Math.sin(p)*d;
		vars.points[i].z=Math.cos(p)*d;
		vars.points[i].y+=vars.points[i].vy*t*((Math.sqrt(vars.distributionRadius)-d)*2);
		if(vars.points[i].y>vars.vortexHeight/2 || d<.25){
			vars.points.splice(i,1);
			spawnParticle(vars);
		}
	}
}
function drawFloor(vars){	
	var x,y,z,d,point,a;
	for (var i = -25; i <= 25; i += 1) {
		for (var j = -25; j <= 25; j += 1) {
			x = i*2;
			z = j*2;
			y = vars.floor;
			d = Math.sqrt(x * x + z * z);
			point = project3D(x, y-d*d/85, z, vars);
			if (point.d != -1) {
				size = 1 + 15000 / (1 + point.d);
				a = 0.15 - Math.pow(d / 50, 4) * 0.15;
				if (a > 0) {
					vars.ctx.fillStyle = colorString(interpolateColors(rgbArray(d/26-vars.frameNo/40),[0,128,32],.5+Math.sin(d/6-vars.frameNo/8)/2));
					vars.ctx.globalAlpha = a;
					vars.ctx.fillRect(point.x-size/2,point.y-size/2,size,size);
				}
			}
		}
	}		
	vars.ctx.fillStyle = "#82f";
	for (var i = -25; i <= 25; i += 1) {
		for (var j = -25; j <= 25; j += 1) {
			x = i*2;
			z = j*2;
			y = -vars.floor;
			d = Math.sqrt(x * x + z * z);
			point = project3D(x, y+d*d/85, z, vars);
			if (point.d != -1) {
				size = 1 + 15000 / (1 + point.d);
				a = 0.15 - Math.pow(d / 50, 4) * 0.15;
				if (a > 0) {
					vars.ctx.fillStyle = colorString(interpolateColors(rgbArray(-d/26-vars.frameNo/40),[32,0,128],.5+Math.sin(-d/6-vars.frameNo/8)/2));
					vars.ctx.globalAlpha = a;
					vars.ctx.fillRect(point.x-size/2,point.y-size/2,size,size);
				}
			}
		}
	}		
}
function sortFunction(a,b){
	return b.dist-a.dist;
}
function draw(vars){
	vars.ctx.globalAlpha=.15;
	vars.ctx.fillStyle="#000";
	vars.ctx.fillRect(0, 0, canvas.width, canvas.height);
	drawFloor(vars);	
	var point,x,y,z,a;
	for(var i=0;i<vars.points.length;++i){
		x=vars.points[i].x;
		y=vars.points[i].y;
		z=vars.points[i].z;
		point=project3D(x,y,z,vars);
		if(point.d != -1){
			vars.points[i].dist=point.d;
			size=1+vars.points[i].radius/(1+point.d);
			d=Math.abs(vars.points[i].y);
			a = .8 - Math.pow(d / (vars.vortexHeight/2), 1000) * .8;
			vars.ctx.globalAlpha=a>=0&&a<=1?a:0;
			vars.ctx.fillStyle=rgb(vars.points[i].color);
			if(point.x>-1&&point.x<vars.canvas.width&&point.y>-1&&point.y<vars.canvas.height)vars.ctx.fillRect(point.x-size/2,point.y-size/2,size,size);
		}
	}
	vars.points.sort(sortFunction);
}
function spawnParticle(vars){
 
	var p,ls;
	pt={};
	p=Math.PI*2*Math.random();
	ls=Math.sqrt(Math.random()*vars.distributionRadius);
	pt.x=Math.sin(p)*ls;
	pt.y=-vars.vortexHeight/2;
	pt.vy=vars.initV/20+Math.random()*vars.initV;
	pt.z=Math.cos(p)*ls;
	pt.radius=200+800*Math.random();
	pt.color=pt.radius/1000+vars.frameNo/250;
	vars.points.push(pt);	
}
function frame(vars) {
	if(vars === undefined){
		var vars={};
		vars.canvas = document.querySelector("canvas");
		vars.ctx = vars.canvas.getContext("2d");
		vars.canvas.width = document.body.clientWidth;
		vars.canvas.height = document.body.clientHeight;
		window.addEventListener("resize", function(){
			vars.canvas.width = document.body.clientWidth;
			vars.canvas.height = document.body.clientHeight;
			vars.cx=vars.canvas.width/2;
			vars.cy=vars.canvas.height/2;
		}, true);
		vars.frameNo=0;
 
		vars.camX = 0;
		vars.camY = 0;
		vars.camZ = -14;
		vars.pitch = elevation(vars.camX, vars.camZ, vars.camY) - Math.PI / 2;
		vars.yaw = 0;
		vars.cx=vars.canvas.width/2;
		vars.cy=vars.canvas.height/2;
		vars.bounding=10;
		vars.scale=500;
		vars.floor=26.5;
 
		vars.points=[];
		vars.initParticles=700;
		vars.initV=.01;
		vars.distributionRadius=800;
		vars.vortexHeight=25;
	}
	vars.frameNo++;
	requestAnimationFrame(function() {
		frame(vars);
	});
	process(vars);
	draw(vars);
}
frame();
</script>
</body>
</html>
 <!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>木鱼科技制作人生重开模拟器 - 休闲小游戏 - </title>
<meta name="Description" content="木鱼科技小游戏免费在线玩人生重开模拟器小游戏,想知道人生重开模拟器怎么玩么？来木鱼科技制作人生重开模拟器看最新最全的人生重开模拟器攻略和秘籍吧。快告诉你的朋友吧!" />
<meta name="Keywords" content="人生重开模拟器,人生重开模拟器怎么玩,人生重开模拟器小游戏,人生重开模拟器攻略秘籍,人生重开模拟器在线玩" />
<meta property="og:title" content="木鱼科技人生重开模拟器H5游戏" />
<meta property="og:site_name" content="木鱼科技小游戏" />
<meta property="og:url" content="http://m.7k7k.com/swf/204220.htm" />
<meta property="og:image" content="https://i3.7k7kimg.cn/game/300200/205/204220_588815.jpg" />
<link rel="canonical" href="http://www.7k7k.com/swf/204220.htm" />
    <link rel="stylesheet" href="http://www.7k7kjs.cn/static/mqike/common/base.css?v=26">
    <link rel="stylesheet"
    href="http://www.7k7kjs.cn/static/mqike/td/css/player.css?v=26">
    <script src="http://www.7k7kjs.cn/static/mqike/common/js/jq.js?v=26"></script>
    <script src="http://www.7k7kjs.cn/static/mqike/common/js/mqike.js?v=26"></script>
    <script type="text/javascript" src="http://www.7k7kjs.cn/static/mqike/td/js/inobounce.js?v=26"></script>
    <script type="text/javascript">
        document.domain="7k7k.com";
        var gameInfo = {
            isWeb: 0,
            gameId: 204220 ,
            gameName:"人生重开模拟器" ,
            gameUrl:"http://flash.7k7k.com/cms/cms10/20211224/1950067852/lifeRestart-master/view/index.html" ,
            gamePic:"https://i4.7k7kimg.cn/game/140140/205/204220_534922.jpg",
            iconWidth: "140"
   }
        var ggInfo = {
        "gg_pic":"https://i1.7k7kimg.cn/game/240400/205/game_zu_shu_204075_417700.jpg",
    "gg_icon":"https://i4.7k7kimg.cn/game/orig/205/204075_004878.png",
    "gg_title":"禅游斗地主",
    "gg_desc":"超高人气的禅游斗地主，全新场景，3D画质",
    "gg_link":"http://h5.7k7k.com/mb/mb2/61cbd3a00e81a3148f48f964820063a8.html?gid=c3435432347bb636c57546fa4024bbf7&\tid=94519&\qs=1"
}    </script>
    <script type="text/javascript" src="http://www.7k7kjs.cn/static/mqike/td/js/player.js?v=26"></script>
</head>
<body ontouchmove="event.preventDefault()">
    <div id="player">
        <iframe src="http://flash.7k7k.com/cms/cms10/20211224/1950067852/lifeRestart-master/view/index.html" id="ifm" frameborder="0" width="100%" height="100%" scrolling='no' ></iframe>
    </div>
    <script type="text/javascript" src="http://www.7k7kjs.cn/qike/qikecore.200628.3.js"></script>
    <script>
        try {
            var vuid = qikecore.account.getVUID();
            vuid && \qikecore.utils.analysis.add( 'player.game.id', {
                "uid": vuid,
                "gid": gameInfo.gameId,
                "wap": 1
            });
        } catch( e ) { }
    </script>
    <script>
        (function(){
            var src = "https://jspassport.ssl.qhimg.com/11.0.1.js?d182b3f28525f2db83acfaaf6e696dba";
            document.write('<script src="'   src   '" id="sozz"><\/script>');
        })();
    </script>
    <script src="//www.7k7kjs.cn/static/common/pub.js?v=1.0.2"></script>
    <link rel="stylesheet" href="http://www.7k7kjs.cn/static/mqike/login/css/login-fcm-m.css?v=26">
    <script src="http://www.7k7kjs.cn/static/mqike/login/js/login-m-fcm2.js?v=1643858118"></script>
</body>
</html>
