<div align="center">

<!-- BACTERIA INTERACTION ANIMATION -->
<style>
  #bacteria-canvas { display: block; border-radius: 12px; background: #0d1a0f; margin: 0 auto; }
  .bacteria-wrap { padding: 0; line-height: 0; text-align: center; }
</style>
<div class="bacteria-wrap">
<canvas id="bacteria-canvas" width="680" height="220"></canvas>
</div>
<script>
(function(){
const cv = document.getElementById('bacteria-canvas');
if(!cv) return;
const ctx = cv.getContext('2d');
const W = 680, H = 220;

const COLORS = [
  {body:'#4a9e6b', mem:'#2d6b47', nuc:'#1a4a2e'},
  {body:'#7a6e2a', mem:'#5c521e', nuc:'#3d380f'},
  {body:'#2e6e7a', mem:'#1e505c', nuc:'#0f343d'},
  {body:'#6e3a7a', mem:'#4e255c', nuc:'#31163a'},
  {body:'#7a3a2e', mem:'#5c251e', nuc:'#3d150f'},
];

function rand(a,b){return a+Math.random()*(b-a);}

class Bacterium {
  constructor(x,y,colorIdx){
    this.x=x; this.y=y;
    this.vx=rand(-0.5,0.5); this.vy=rand(-0.5,0.5);
    this.angle=rand(0,Math.PI*2);
    this.angV=rand(-0.008,0.008);
    this.rx=rand(14,22); this.ry=rand(7,11);
    this.c=COLORS[colorIdx%COLORS.length];
    this.flagella=Array.from({length:rand(1,3)|0},()=>({phase:rand(0,Math.PI*2),speed:rand(0.04,0.09),amp:rand(8,18),side:Math.random()<0.5?1:-1}));
    this.pili=Array.from({length:rand(2,5)|0},()=>({angle:rand(0,Math.PI*2),len:rand(12,28),grow:rand(0.005,0.015),maxLen:rand(20,35),phase:rand(0,Math.PI*2)}));
    this.dividing=false; this.divProgress=0;
    this.divTimer=rand(800,2400)|0;
    this.age=0; this.signal=0;
    this.signalTimer=rand(120,400)|0;
    this.pulseR=0; this.pulsing=false;
  }
  update(t, bacteria){
    this.age++; this.angle+=this.angV;
    this.x+=this.vx; this.y+=this.vy;
    if(this.x<this.rx+5){this.vx+=0.05;}
    if(this.x>W-this.rx-5){this.vx-=0.05;}
    if(this.y<this.ry+5){this.vy+=0.05;}
    if(this.y>H-this.ry-5){this.vy-=0.05;}
    this.vx*=0.995; this.vy*=0.995;
    this.vx+=rand(-0.04,0.04); this.vy+=rand(-0.04,0.04);
    const spd=Math.sqrt(this.vx*this.vx+this.vy*this.vy);
    if(spd>1.2){this.vx*=1.2/spd; this.vy*=1.2/spd;}
    for(let b of bacteria){
      if(b===this) continue;
      const dx=b.x-this.x, dy=b.y-this.y;
      const d=Math.sqrt(dx*dx+dy*dy);
      if(d<60&&d>2){const f=(d<30)?-0.004:0.001; this.vx+=dx/d*f; this.vy+=dy/d*f;}
    }
    this.signalTimer--;
    if(this.signalTimer<=0){this.pulsing=true; this.pulseR=0; this.signalTimer=rand(200,500)|0;}
    if(this.pulsing){this.pulseR+=1.5; if(this.pulseR>55) this.pulsing=false;}
    if(!this.dividing){this.divTimer--; if(this.divTimer<=0) this.dividing=true;}
    if(this.dividing){this.divProgress+=0.008;}
    for(let p of this.pili){p.phase+=p.grow; p.len=p.maxLen*0.5*(1+Math.sin(p.phase));}
  }
  draw(ctx, t){
    ctx.save(); ctx.translate(this.x, this.y); ctx.rotate(this.angle);
    for(let i=0;i<this.flagella.length;i++){
      const f=this.flagella[i]; const base=f.side*(this.ry);
      ctx.save(); ctx.strokeStyle=this.c.mem+'aa'; ctx.lineWidth=1.2; ctx.beginPath(); ctx.moveTo(-this.rx, base);
      for(let s=0;s<=20;s++){const prog=s/20; ctx.lineTo(-this.rx-prog*30, base+Math.sin(prog*Math.PI*2.5+f.phase+t*f.speed)*f.amp*(1-prog*0.5));}
      ctx.stroke(); f.phase+=0.02; ctx.restore();
    }
    for(let p of this.pili){
      const px=Math.cos(p.angle-this.angle)*this.rx; const py=Math.sin(p.angle-this.angle)*this.ry;
      const ex=px+Math.cos(p.angle-this.angle)*p.len; const ey=py+Math.sin(p.angle-this.angle)*p.len;
      ctx.save(); ctx.strokeStyle=this.c.body+'88'; ctx.lineWidth=0.8; ctx.beginPath(); ctx.moveTo(px,py); ctx.lineTo(ex,ey); ctx.stroke();
      ctx.fillStyle=this.c.mem; ctx.beginPath(); ctx.arc(ex,ey,1.5,0,Math.PI*2); ctx.fill(); ctx.restore();
    }
    const rx=this.dividing?this.rx*(1+this.divProgress*0.3):this.rx;
    const ry=this.dividing?this.ry*(1-this.divProgress*0.25):this.ry;
    ctx.save();
    const grad=ctx.createRadialGradient(0,0,ry*0.3,0,0,rx);
    grad.addColorStop(0,this.c.body+'ff'); grad.addColorStop(0.7,this.c.body+'cc'); grad.addColorStop(1,this.c.mem+'88');
    ctx.fillStyle=grad; ctx.strokeStyle=this.c.mem; ctx.lineWidth=1.5;
    ctx.beginPath(); ctx.ellipse(0,0,rx,ry,0,0,Math.PI*2); ctx.fill(); ctx.stroke(); ctx.restore();
    if(this.dividing&&this.divProgress>0.1){ctx.save(); ctx.strokeStyle=this.c.nuc+'cc'; ctx.lineWidth=1.5; ctx.beginPath(); ctx.moveTo(0,-ry); ctx.lineTo(0,ry); ctx.stroke(); ctx.restore();}
    ctx.save(); ctx.fillStyle=this.c.nuc+'dd'; ctx.beginPath(); ctx.ellipse(this.dividing?-rx*0.25:0,0,rx*0.35,ry*0.45,0.3,0,Math.PI*2); ctx.fill();
    if(this.dividing&&this.divProgress>0.3){ctx.beginPath(); ctx.ellipse(rx*0.25,0,rx*0.3,ry*0.4,-0.2,0,Math.PI*2); ctx.fill();}
    ctx.restore();
    ctx.fillStyle=this.c.mem+'bb';
    for(let i=0;i<6;i++){const ra=i/6*Math.PI*2+t*0.002; const rr=rx*0.5; ctx.beginPath(); ctx.arc(Math.cos(ra)*rr*0.7,Math.sin(ra)*rr*0.4,1.5,0,Math.PI*2); ctx.fill();}
    ctx.restore();
    if(this.pulsing){ctx.save(); ctx.globalAlpha=Math.max(0,(1-this.pulseR/55)*0.35); ctx.strokeStyle=this.c.body; ctx.lineWidth=1; ctx.beginPath(); ctx.arc(this.x,this.y,this.pulseR,0,Math.PI*2); ctx.stroke(); ctx.restore();}
  }
}

class Nutrient {
  constructor(){this.x=rand(0,W); this.y=rand(0,H); this.vx=rand(-0.2,0.2); this.vy=rand(-0.2,0.2); this.r=rand(1.5,3); this.alpha=rand(0.3,0.7); this.color=`hsl(${rand(80,160)|0},60%,65%)`;}
  update(){this.x+=this.vx; this.y+=this.vy; if(this.x<0||this.x>W)this.vx*=-1; if(this.y<0||this.y>H)this.vy*=-1;}
  draw(ctx){ctx.save(); ctx.globalAlpha=this.alpha; ctx.fillStyle=this.color; ctx.beginPath(); ctx.arc(this.x,this.y,this.r,0,Math.PI*2); ctx.fill(); ctx.restore();}
}

function drawConnections(ctx,bacteria){
  for(let i=0;i<bacteria.length;i++){for(let j=i+1;j<bacteria.length;j++){
    const dx=bacteria[j].x-bacteria[i].x,dy=bacteria[j].y-bacteria[i].y,d=Math.sqrt(dx*dx+dy*dy);
    if(d<80){ctx.save(); ctx.globalAlpha=(1-d/80)*0.18; ctx.strokeStyle='#6aaa88'; ctx.lineWidth=0.8; ctx.beginPath(); ctx.moveTo(bacteria[i].x,bacteria[i].y); ctx.lineTo(bacteria[j].x,bacteria[j].y); ctx.stroke(); ctx.restore();}
  }}
}

let bacteria=Array.from({length:18},(_,i)=>new Bacterium(rand(60,W-60),rand(40,H-40),i%5));
let nutrients=Array.from({length:40},()=>new Nutrient());
let t=0, divQueue=[];

function loop(){
  ctx.clearRect(0,0,W,H);
  ctx.save(); ctx.globalAlpha=0.04; ctx.strokeStyle='#4a9e6b'; ctx.lineWidth=0.5;
  for(let x=0;x<W;x+=40){ctx.beginPath();ctx.moveTo(x,0);ctx.lineTo(x,H);ctx.stroke();}
  for(let y=0;y<H;y+=40){ctx.beginPath();ctx.moveTo(0,y);ctx.lineTo(W,y);ctx.stroke();}
  ctx.restore();
  for(let n of nutrients){n.update();n.draw(ctx);}
  drawConnections(ctx,bacteria);
  for(let b of bacteria){b.update(t,bacteria);b.draw(ctx,t);if(b.dividing&&b.divProgress>=1&&bacteria.length<32)divQueue.push(b);}
  for(let b of divQueue){
    const idx=bacteria.indexOf(b);
    if(idx>-1){
      const off=10,nx=b.x+Math.cos(b.angle+Math.PI/2)*off,ny=b.y+Math.sin(b.angle+Math.PI/2)*off;
      b.x-=Math.cos(b.angle+Math.PI/2)*off; b.y-=Math.sin(b.angle+Math.PI/2)*off;
      b.dividing=false; b.divProgress=0; b.divTimer=rand(900,2500)|0; b.rx=rand(14,22); b.ry=rand(7,11);
      const nb=new Bacterium(nx,ny,(idx+1)%5); nb.vx=b.vx+rand(-0.3,0.3); nb.vy=b.vy+rand(-0.3,0.3);
      bacteria.push(nb); if(bacteria.length>32)bacteria.splice(0,1);
    }
  }
  divQueue=[]; t++; requestAnimationFrame(loop);
}
loop();
})();
</script>

<!-- ANIMATED SNAKE CONTRIBUTION GRAPH -->
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/Arun17796/Arun17796/output/github-contribution-grid-snake-dark.svg" />
  <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Arun17796/Arun17796/output/github-contribution-grid-snake.svg" />
  <img alt="github contribution grid snake animation" src="https://raw.githubusercontent.com/Arun17796/Arun17796/output/github-contribution-grid-snake.svg" />
</picture>

<!-- TYPING ANIMATION -->
[![Typing SVG](https://readme-typing-svg.demolab.com?font=JetBrains+Mono&weight=500&size=18&pause=1000&color=7A9E6E&background=00000000&center=true&vCenter=true&width=750&lines=🌿+Microbial+Ecologist+%7C+Bioinformatician;🧬+Host-Microbiome+Interactions+%26+Clinical+Metagenomics;⚙️+Snakemake+%7C+Nextflow+%7C+HPC+Pipeline+Architect;🔬+16S+%7C+WGS+%7C+Functional+Annotation+%7C+MAGs;🌱+Every+microbe+has+a+story+—+I+sequence+it)](https://git.io/typing-svg)

<!-- NATURE ANIMATED DIVIDER -->
<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/grass.png" width="100%"/>

</div>

## 🌱 About Me

```yaml
Name    : Arunmozhi Bharathi Achudhan
Location: Tamil Nadu, India
Focus:
  - Microbial ecology & alpha/beta diversity analytics
  - Functional annotation & metabolic pathway reconstruction
  - Host-microbiome interactions (clinical metagenomics)
  - Scalable bioinformatics pipeline development
Philosophy: "The microbiome is not noise — it is signal we haven't decoded yet."
```

<div align="center">
<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/grass.png" width="100%"/>
</div>

## 🧫 Research Focus

<div align="center">

| 🌿 Microbial Ecology | 🔬 Functional Annotation | 🩺 Clinical Metagenomics | ⚙️ Pipeline Engineering |
|:---:|:---:|:---:|:---:|
| Alpha/Beta diversity | HUMAnN3 · COG · KEGG | Host-microbiome crosstalk | Snakemake · Nextflow |
| Phylogenetic inference | Pathway reconstruction | Disease-associated dysbiosis | HPC · SLURM · WDL |
| Rare biosphere taxa | CAZyme profiling | Resistome & virulome | Containerised workflows |

</div>

<div align="center">
<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/grass.png" width="100%"/>
</div>

## 🛠️ Tools & Stack

<!-- LANGUAGES & FRAMEWORKS -->
<div align="center">

![Python](https://img.shields.io/badge/Python-3B5323?style=for-the-badge&logo=python&logoColor=d4c5a9)
![R](https://img.shields.io/badge/R_/_Bioconductor-5C4827?style=for-the-badge&logo=r&logoColor=d4c5a9)
![Shell](https://img.shields.io/badge/Shell_Scripting-2E4A1E?style=for-the-badge&logo=gnubash&logoColor=d4c5a9)
![Nextflow](https://img.shields.io/badge/Nextflow-4A7C59?style=for-the-badge&logo=nextflow&logoColor=white)
![Snakemake](https://img.shields.io/badge/Snakemake-6B8F3E?style=for-the-badge&logo=snakemake&logoColor=white)

</div>

<!-- BIOINFORMATICS TOOLS -->
<div align="center">

![QIIME2](https://img.shields.io/badge/QIIME2-795548?style=for-the-badge&logoColor=white)
![MetaPhlAn](https://img.shields.io/badge/MetaPhlAn4-5D4037?style=for-the-badge&logoColor=white)
![Kraken2](https://img.shields.io/badge/Kraken2-4E342E?style=for-the-badge&logoColor=white)
![HUMAnN3](https://img.shields.io/badge/HUMAnN3-6D4C41?style=for-the-badge&logoColor=white)
![DRAM](https://img.shields.io/badge/DRAM-8D6E63?style=for-the-badge&logoColor=white)
![CheckM](https://img.shields.io/badge/CheckM2-7B5E57?style=for-the-badge&logoColor=white)
![GTDB-Tk](https://img.shields.io/badge/GTDB--Tk-9C7B4F?style=for-the-badge&logoColor=white)
![Bowtie2](https://img.shields.io/badge/Bowtie2-A1887F?style=for-the-badge&logoColor=white)

</div>

<!-- HPC & INFRA -->
<div align="center">

![SLURM](https://img.shields.io/badge/SLURM_/_HPC-3E2723?style=for-the-badge&logo=linux&logoColor=d4c5a9)
![Docker](https://img.shields.io/badge/Docker-4A6741?style=for-the-badge&logo=docker&logoColor=white)
![Singularity](https://img.shields.io/badge/Singularity-556B2F?style=for-the-badge&logoColor=white)
![Git](https://img.shields.io/badge/Git-6B5B3E?style=for-the-badge&logo=git&logoColor=white)

</div>

<div align="center">
<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/grass.png" width="100%"/>
</div>

## 🧬 Metagenomic Workflow Expertise

```
🌿 Raw Reads (Illumina/Nanopore)
    │
    ├─► 🧹 QC: fastp / Trimmomatic / NanoFilt
    │
    ├─► 🧫 Host decontamination: Bowtie2 / Minimap2
    │
    ├─► 🔭 Taxonomic profiling: Kraken2+Bracken / MetaPhlAn4
    │
    ├─► ⚗️  Functional profiling: HUMAnN3 → KEGG / MetaCyc / eggNOG
    │
    ├─► 🏗️  Assembly: MEGAHIT / metaSPAdes / Flye (long-read)
    │
    ├─► 🗂️  Binning: MetaBAT2 / SemiBin2 + DAS_Tool
    │
    ├─► ✅ MAG QC: CheckM2 → GTDB-Tk → DRAM annotation
    │
    └─► 📊 Diversity & stats: phyloseq / vegan / MaAsLin2 / ANCOM-BC
```

<div align="center">
<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/grass.png" width="100%"/>
</div>

## 🌍 Connect

<div align="center">

[![GitHub](https://img.shields.io/badge/GitHub-Arun17796-2E4A1E?style=for-the-badge&logo=github&logoColor=d4c5a9)](https://github.com/Arun17796)
[![Email](https://img.shields.io/badge/Email-arunmozhiachudhan@gmail.com-5C4827?style=for-the-badge&logo=gmail&logoColor=d4c5a9)](mailto:arunmozhiachudhan@gmail.com)
[![ORCID](https://img.shields.io/badge/ORCID-0000--0001--8170--4605-4A7C40?style=for-the-badge&logo=orcid&logoColor=white)](https://orcid.org/0000-0001-8170-4605)

</div>

<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&customColorList=6,11,20&height=120&section=footer&fontColor=d4c5a9&animation=fadeIn" />

*"In a gram of soil lives a universe — I'm just trying to read it."*

![Profile Views](https://komarev.com/ghpvc/?username=Arun17796&color=7a9e6e&style=flat-square&label=Profile+Views)

</div>
