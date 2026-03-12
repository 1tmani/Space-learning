# Space-learning
```html id="8mv4qx"
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Particle Output Screen System</title>
<style>
body{
margin:0;
overflow:hidden;
background:black;
}
canvas{
display:block;
}
#upload{
position:fixed;
top:20px;
left:20px;
z-index:10;
}
#output{
position:fixed;
right:20px;
top:20px;
width:320px;
height:180px;
border:1px solid rgba(255,255,255,0.25);
background:black;
z-index:10;
}
</style>
</head>
<body>

<input type="file" id="upload" multiple accept="image/*">
<canvas id="c"></canvas>
<canvas id="output"></canvas>

<script>

const canvas=document.getElementById("c")
const ctx=canvas.getContext("2d")

const output=document.getElementById("output")
const octx=output.getContext("2d")

canvas.width=window.innerWidth
canvas.height=window.innerHeight

output.width=320
output.height=180

const total=65000

let particles=[]
let targets=[]
let memory=[]
let motionSensors=[]
let cameraSensors=[]
let launched=[]
let time=0

const palette=[
"#ffffff",
"#ff6600",
"#ffaa00",
"#00ffff",
"#6699ff",
"#ff66cc",
"#00ff99"
]

class Particle{

constructor(role,i){

this.role=role
this.i=i
this.x=Math.random()*canvas.width
this.y=Math.random()*canvas.height
this.tx=this.x
this.ty=this.y
this.vx=0
this.vy=0
this.color=palette[Math.floor(Math.random()*palette.length)]
this.learn=0

}

update(){

let dx=this.tx-this.x
let dy=this.ty-this.y

this.vx+=dx*0.010
this.vy+=dy*0.010

this.vx*=0.89
this.vy*=0.89

this.x+=this.vx
this.y+=this.vy

}

draw(){

ctx.fillStyle=this.color
ctx.fillRect(this.x,this.y,1.5,1.5)

}

}

class LaunchParticle{

constructor(x,y,color){

this.x=x
this.y=y
this.vx=(Math.random()-0.5)*8
this.vy=(Math.random()-0.5)*8
this.color=color

}

update(){

this.x+=this.vx
this.y+=this.vy
this.vx*=0.98
this.vy*=0.98

}

draw(){

ctx.fillStyle=this.color
ctx.fillRect(this.x,this.y,2,2)

}

}

particles.push(new Particle("core",0))

for(let i=1;i<=10;i++){

let p=new Particle("motion",i)
particles.push(p)
motionSensors.push(p)

}

for(let i=11;i<=30;i++){

let p=new Particle("camera",i)
particles.push(p)
cameraSensors.push(p)

}

for(let i=31;i<total;i++){

particles.push(new Particle("field",i))

}

function extractImage(img){

let off=document.createElement("canvas")
let octx2=off.getContext("2d")

off.width=canvas.width
off.height=canvas.height

let ratio=Math.min(canvas.width/img.width,canvas.height/img.height)*0.7

let w=img.width*ratio
let h=img.height*ratio

octx2.drawImage(img,(canvas.width-w)/2,(canvas.height-h)/2,w,h)

let data=octx2.getImageData(0,0,off.width,off.height).data

targets=[]
memory=[]

for(let y=0;y<off.height;y+=3){

for(let x=0;x<off.width;x+=3){

let i=(y*off.width+x)*4

if(data[i+3]>120){

let r=data[i]
let g=data[i+1]
let b=data[i+2]

if(r+g+b>80){

targets.push({x,y,color:`rgb(${r},${g},${b})`})
memory.push({x,y,color:`rgb(${r},${g},${b})`})

}

}

}

}

}

function coreMind(){

let core=particles[0]

if(memory.length>0){

let t=memory[Math.floor(time*2)%memory.length]

core.tx+=(t.x-core.tx)*0.18
core.ty+=(t.y-core.ty)*0.18
core.color=t.color

}

}

function motionLayer(){

let core=particles[0]

for(let i=0;i<motionSensors.length;i++){

let p=motionSensors[i]

let a=(Math.PI*2/motionSensors.length)*i+time*0.03

p.tx=core.x+Math.cos(a)*90
p.ty=core.y+Math.sin(a)*90
p.color="#00ffff"

}

}

function cameraLayer(){

let core=particles[0]

for(let i=0;i<cameraSensors.length;i++){

let p=cameraSensors[i]

let a=(Math.PI*2/cameraSensors.length)*i-time*0.01

p.tx=core.x+Math.cos(a)*180
p.ty=core.y+Math.sin(a)*180
p.color="#ffaa00"

}

}

function fieldLearning(){

for(let i=31;i<particles.length;i++){

let p=particles[i]

let source=i%2===0?motionSensors[i%motionSensors.length]:cameraSensors[i%cameraSensors.length]

p.tx+=(source.tx-p.tx)*0.07
p.ty+=(source.ty-p.ty)*0.07

if(memory.length>0){

let t=memory[i%memory.length]

p.tx+=(t.x-p.tx)*0.004
p.ty+=(t.y-p.ty)*0.004
p.color=t.color

}

}

}

function launchOutput(){

if(time%20===0 && memory.length>0){

let core=particles[0]

for(let i=0;i<6;i++){

launched.push(new LaunchParticle(core.x,core.y,core.color))

}

}

for(let i=0;i<launched.length;i++){

launched[i].update()
launched[i].draw()

}

if(launched.length>1200){

launched.splice(0,6)

}

}

function outputScreen(){

octx.fillStyle="rgba(0,0,0,0.18)"
octx.fillRect(0,0,output.width,output.height)

for(let i=0;i<particles.length;i+=18){

let p=particles[i]

let x=(p.x/canvas.width)*output.width
let y=(p.y/canvas.height)*output.height

octx.fillStyle=p.color
octx.fillRect(x,y,1.4,1.4)

}

}

function animate(){

time++

ctx.fillStyle="rgba(0,0,0,0.07)"
ctx.fillRect(0,0,canvas.width,canvas.height)

coreMind()
motionLayer()
cameraLayer()
fieldLearning()

for(let i=0;i<particles.length;i++){

particles[i].update()
particles[i].draw()

}

launchOutput()
outputScreen()

requestAnimationFrame(animate)

}

animate()

document.getElementById("upload").addEventListener("change",e=>{

const files=e.target.files

for(let f=0;f<files.length;f++){

let img=new Image()

img.onload=function(){

extractImage(img)

}

img.src=URL.createObjectURL(files[f])

}

})

window.addEventListener("resize",()=>{

canvas.width=window.innerWidth
canvas.height=window.innerHeight

})

</script>

</body>
</html>
```
