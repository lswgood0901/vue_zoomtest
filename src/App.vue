<template>
  <div id="app">
  </div>
</template>

<script>
import * as d3 from 'd3' 
import flareJson from './assets/flare.json'
export default {
  name: 'App',
  components: {
  },
  data() {
    return {
    sampleNode: flareJson,
    svg : null,
    svgMap : null,
    svgmini : null,
    miniMap : null,
    minimapScatter : null,
    width : 500,
    height: 500,
    simulation : null,
    miniSimulation: null,
    newSimulation: null,
    svgBrush: null,
    brushSelection: null,
    k : 1,
    tx : 0,
    ty :0,
    scatter: null,
    node: [
      {"name":"A",
      "id":"A0",
      x : 20,
      y : 40},
      {"name": "B",
      "id":"B0",
      x : 100,
      y : 100},
      {"name": "C",
      "id":"C0",
      x : 300,
      y : 300},
      {"name": "D",
      "id":"D0",
      x : 250,
      y : 10},
      {"name": "E",
      "id":"E0",
      x : 10,
      y : 300},
      {"name": "F",
      "id":"F0",
      x : 48,
      y : 12},
      {"name": "G",
      "id":"G0",
      x : 48,
      y : 178},
      {"name": "H",
      "id":"H0",
      x : 90,
      y : 178},
      {"name": "I",
      "id":"I0",
      x : 67,
      y : 117},
      {"name": "J",
      "id":"J0",
      x : 242,
      y : 105}
      ],
    arrayK : [0,0,0],
    arrayTx:[0,0,0,0,0],
    arrayTy:[0,0,0,0,0],
    zooms: null,
    brush: null,
    i : 0,
    clicki :0,
    bbox: null,
    arrayBbox: [],
    eventType : null,
    tFlag: null,
    bFlag: true,
    uFlag: null,
    userActionStatus: [0,0,0,0,0],
    t: null,
    scaleX : null,
    scaleY : null,
    universeChange : null,
    previousUniverse: null,
    previousCluster: null,
    }
  },
  methods: {
    
    isBrushed: (brushCoords, cx, cy)=>{ // eslint-disable-line no-unused-vars
      const x0 = brushCoords[0][0];
      const x1 = brushCoords[1][0];
      const y0 = brushCoords[0][1];
      const y1 = brushCoords[1][1];
      return x0 > cx || cx > x1 || y0 > cy || cy > y1;
    },
    delay: (ms)=>{
      const wakeUpTime = Date.now() + ms
      while (Date.now() < wakeUpTime) {//
      }
    },
    sleep: (ms)=>{
      return new Promise((r) => setTimeout(r,ms))
    }

  },
  
  watch: {

    i(){
      console.log(d3.selectAll('.inboxCircle.presentCircle, .outboxCircle.presentCircle'))
      d3.selectAll('.inboxCircle.presentCircle, .outboxCircle.presentCircle')
        .classed('inboxCircle', (d)=>{
          return !this.isBrushed(this.brushSelection, d.x, d.y)
        })
        .classed('outboxCircle', (d)=>{
          return this.isBrushed(this.brushSelection, d.x, d.y)
        })
        
      let istepGroup = []
      for(let istep = 0; istep < d3.selectAll('.inboxCircle.presentCircle')._groups[0].length; istep ++){
        istepGroup.push(d3.selectAll('.inboxCircle.presentCircle')._groups[0][istep].id[2])
        }
      
      let result = {}
      istepGroup.forEach((x) => { 
        result[x] = (result[x] || 0)+1
      })

      // for(let bstep = 0; bstep<this.scatter._groups[0].length ; bstep++){
      //   if (result[bstep] / d3.selectAll('#B'+bstep)._groups[0][0].children.length > 0.45){
      //     d3.selectAll('#A' + bstep)
      //       .attr('class', 'inbox presentCircle')
      //     d3.selectAll(d3.selectAll('#B' + bstep)._groups[0][0].children).attr('class', 'inboxCircle presentCircle')
      //   } else {
      //     d3.selectAll('#A' + bstep)
      //       .attr('class', 'outofBox presentCircle')
      //     d3.selectAll(d3.selectAll('#B' + bstep)._groups[0][0].children).attr('class', 'outboxCircle presentCircle')
      //   }
      // } //inbox 안에 있는 circle들 모두 inboxCircle

      // if(d3.selectAll('.inbox.presentCircle')._groups[0].length == 0){
      //   let brushMx = (this.brushSelection[0][0] + this.brushSelection[1][0]) / 2
      //   let brushMy = (this.brushSelection[0][1] + this.brushSelection[1][1]) / 2
      //   let wholeXY = []
      //   let disXY = []
      //   for(let dstep = 0; dstep<this.scatter._groups[0].length; dstep++){
      //     wholeXY[dstep]= [this.scatter._groups[0][dstep].cx.baseVal.value, this.scatter._groups[0][dstep].cy.baseVal.value]
      //     disXY[dstep] = Math.pow(wholeXY[dstep][0] - brushMx, 2) + 
      //                     Math.pow(wholeXY[dstep][1] - brushMy, 2)
      //     }
      //   d3.select('#'+this.scatter._groups[0][disXY.indexOf(Math.min(...disXY))].id)
      //     .attr('class', 'inbox presentCircle')
      //   d3.selectAll(d3.selectAll('#B' + this.scatter._groups[0][disXY.indexOf(Math.min(...disXY))].id[1])._groups[0][0].children)
      //     .attr('class', 'inboxCircle presentCircle')
      // } // view안에 클러스터가 하나도 없을경우 뷰 중심점과 가장 가까운 클러스터로 이동

      //BD에 inbox 클래스 전송
      //최종 클러스터 넘버 받기 

      let d3length = d3.selectAll('.inbox.presentCircle')._groups[0].length
      let eachCx1 = []
      let eachCx2 = []
      let eachCy1 = []
      let eachCy2 = []
      for(let step = 0; step < d3length; step++){
        eachCx1.push(d3.selectAll('.inbox.presentCircle')._groups[0][step].cx.baseVal.value - d3.selectAll('.inbox.presentCircle')._groups[0][step].r.baseVal.value)
        eachCx2.push(d3.selectAll('.inbox.presentCircle')._groups[0][step].cx.baseVal.value + d3.selectAll('.inbox.presentCircle')._groups[0][step].r.baseVal.value)
        eachCy1.push(d3.selectAll('.inbox.presentCircle')._groups[0][step].cy.baseVal.value - d3.selectAll('.inbox.presentCircle')._groups[0][step].r.baseVal.value)
        eachCy2.push(d3.selectAll('.inbox.presentCircle')._groups[0][step].cy.baseVal.value + d3.selectAll('.inbox.presentCircle')._groups[0][step].r.baseVal.value)
      }
      let selectedBBox = [[Math.min(...eachCx1), Math.min(...eachCy1)], [Math.max(...eachCx2), Math.max(...eachCy2)]]
      let bboxX1 = selectedBBox[0][0]
      let bboxY1 = selectedBBox[0][1]
      let bboxX2 = selectedBBox[1][0]
      let bboxY2 = selectedBBox[1][1]
      let bboxHeight = Math.max(bboxX2-bboxX1, bboxY2-bboxY1) // eslint-disable-line no-unused-vars
      this.bFlag = false
      this.svgMap.call(
        this.zooms.transform,
        d3.zoomIdentity.translate(this.scaleX(this.width / bboxHeight)(-bboxX1), this.scaleY(this.width / bboxHeight)(-bboxY1)).scale(this.width / bboxHeight)
      )
      this.bFlag = true
      this.arrayK=[0,0,0]
      eachCx1 = []
      eachCx2 = []
      eachCy1 = []
      eachCy2 = []
      selectedBBox = []
      istepGroup = []
      result = {}
      console.log("보정")
    },
    clicki(){
      let r = d3.selectAll('.inbox.presentCircle')._groups[0][0].r.baseVal.value
      let dx = d3.selectAll('.inbox.presentCircle')._groups[0][0].cx.baseVal.value - r
      let dy = d3.selectAll('.inbox.presentCircle')._groups[0][0].cy.baseVal.value - r
      this.bFlag = false
      this.svgMap.call(
        this.zooms.transform,
        d3.zoomIdentity.translate(this.scaleX(this.width / (2*r))(-dx), this.scaleY(this.width / (2*r))(-dy)).scale(this.width / (2*r))
      )
      d3.selectAll(this.childScatter)
        .classed('inboxCircle', (d)=>{
          return !this.isBrushed(this.brushSelection, d.x, d.y)
        })
        .classed('outboxCircle', (d)=>{
          return this.isBrushed(this.brushSelection, d.x, d.y)
        })
      this.bFlag = true
      this.arrayK=[0,0,0]
      console.log('클릭 보정')
    },
    t(){
      if(this.t == 2){
        let startTime = Date.now()
        let endTime = startTime + 500
        for(let cstep = 0; cstep < 10; cstep++){
          const centerX = this.parentNodes[cstep].x
          const centerY = this.parentNodes[cstep].y
          this.childSimulation = d3.forceSimulation(this.parentNodes[cstep].children)
            .force('charge', d3.forceManyBody().strength(2))
            .force('center', d3.forceCenter(centerX, centerY))
            .force('collision', d3.forceCollide().radius(4.5))
            .on('tick', ()=>{
              if(Date.now() < endTime) {
              return this.childScatter
                .attr('cx', (d)=>{
                  return d.x
                })
                .attr('cy', (d)=>{
                  return d.y
                })
                .attr('id', (d,i)=>{
                  return 'B' + '_' + d.data.group + '_' + i
                })
              } else {
                this.childSimulation.stop()
                
              }
            })
        }
      }
      if(this.t == 4) {
        this.i = this.i + 1
      }

    },
    universeChange(){
      // this.uFlag = true
      console.log('newUniverse')
      this.bFlag = false
      this.tFlag = false
      // outofBox cx, cy, r 데이터 저장
      d3.selectAll('.outofBox').classed('previousCircle', true).classed('presentCircle', false)
      d3.selectAll('.inbox').classed('previousCircle', true).classed('presentCircle', false)
      d3.selectAll('.outboxCircle').classed('previousCircle', true).classed('presentCircle', false)
      d3.selectAll('.inboxCircle').classed('previousCircle', true).classed('presentCircle', false)
      // 서버에 universe & data 전송 
      // 서버에 데이터 요청 
      // 다시 force simualtion // 
      this.inClustergCam = this.gCam.append('g')
      this.inClustergCam
        .selectAll('g')
        .data(this.node)
        .join('g')
        .append('circle')
        .attr('cx', (d)=>{
          return d.x
        })
        .attr('cy', (d)=>{
          return d.y
        })
        .attr('r', 10)
        .attr('class', 'inbox presentCircle')
        console.log(d3.selectAll('.inboxCircle.previousCircle'))
      // d3.selectAll('.previousCircle').classed('previousCircle', false)
      // this.uFlag = false
      // this.tFlag = true
        
    },
    reCluster(){
      //서버에 데이터 요청
      //remove all circle
      //simualtion
    },
    previousUniverse(){
      //
    }
  },


  created() {
    //서버에 데이터 요청 
  },
  mounted() {
    this.parentNodes = d3.hierarchy(this.sampleNode).children
    console.log(this.parentNodes)
    this.scaleX = (k)=>{
      return d3.scaleLinear([0,this.width],[0,this.width * k])
    }
    this.scaleY = (k)=>{
      return d3.scaleLinear([0,this.height],[0,this.height * k])
    }
    setInterval(()=>{
      this.userActionStatus.unshift(0)
      this.userActionStatus.pop()
      for(let step = 1; step < 5; step++){
        if(this.userActionStatus[step]==1){
          this.tFlag = true          
       }
      }
      if(this.tFlag == true && this.userActionStatus[0]==0 && 
        this.userActionStatus[1]==0 && this.userActionStatus[2]==0 &&
        this.userActionStatus[3]==0 && this.userActionStatus[4]==0){
        this.i = this.i + 1
        this.tFlag = false
        console.log('시간보정')
      }
    }, 100)
    this.initialT = setInterval(()=>{
      this.t = this.t + 1
    }, 250)

    this.svg = d3.select('#app')
      .append('svg')
      .attr('width', 500)
      .attr('height', 500)
    this.svgmini = d3.select('#app')
      .append('svg')
      .attr('width', 500)
      .attr('height', 500)
    this.svgMap = this.svg
      .append('svg')
      .attr('width', this.width)
      .attr('height', this.height)
    this.miniMap = this.svgmini
      .append('svg')
      .attr('width', this.width)
      .attr('height', this.height)
    
    let startTime = Date.now()
    let endTime = startTime + 500

    const ticked = ()=>{
      if(Date.now() < endTime){
        return this.scatter
        .attr('cx', (d)=>{ 
          return d.x
        })
        .attr('cy', (d)=> {
          return d.y
        })
      } else {
        this.simulation.stop()
      }     
    }
   
    const miniMapTicked = ()=>{
      if(Date.now() < endTime){
        return this.minimapScatter
        .attr('cx', (d)=>{ 
          return d.x
        })
        .attr('cy', (d)=> {
          return d.y
        })
      } else {
        this.miniSimulation.stop()
      }
    }


    this.gCam = this.svgMap.append('g')
    this.gCam.append('rect')
      .attr('width', this.width)
      .attr('height', this.height)
      .attr('fill-opacity', 0)
    
    this.childgCam = this.svgMap.append('g')
      .selectAll('g')
      .data(this.parentNodes)
      .join('g')
      .attr("id", (d,i)=>{
        return 'B' + this.parentNodes[i].children[0].data.group
      })
    this.scatter = this.gCam
      .selectAll('circle')
      .data(this.parentNodes)
      .join('circle')
      .attr("r", (d,i)=>{
          return  this.parentNodes[i].children.length * 1.25
      })
      .attr("class", 'inbox presentCircle')
      .attr("id", (d,i)=>{
        return 'A' + this.parentNodes[i].children[0].data.group
      })
      .on('click', (event)=>{
        if(d3.selectAll('.inbox.presentCircle')._groups[0].length == 2) {
          this.universeChange = this.universeChange + 1
        } else if(!this.uFlag && !event.ctrlKey){
          d3.selectAll('.inbox.presentCircle').attr('class', 'outofBox presentCircle')
          console.log(event.target.id)
          d3.selectAll('#' + event.target.id).attr('class', 'inbox presentCircle')
          this.clicki = this.clicki + 1
        }
      })
      .on('mouseover', (event)=>{
        if(!event.ctrlKey){
          d3.select('#'+event.target.id).style('opacity', 0.1)
          // console.log(event.target.cx.baseVal.value)
          this.overlay = this.gCam.append('rect')
            .attr('x',()=>{
              return event.target.cx.baseVal.value-event.target.r.baseVal.value * Math.sqrt(0.5, 2)
            })
            .attr('y', event.target.cy.baseVal.value-event.target.r.baseVal.value * Math.sqrt(0.5, 2))
            .attr('width', event.target.r.baseVal.value * Math.sqrt(2, 2))
            .attr('height', event.target.r.baseVal.value * Math.sqrt(2, 2))
            .attr('fill', 'orange')
            .attr('fill-opacity', 0.5)
          this.overlay.lower()
          this.gCam.raise()
        }
      })
      .on('mouseout', (event)=>{
        if(!event.ctrlKey && event.target.style.opacity == 0.1){
          // this.sleep(1000).then(() =>  d3.select('#'+event.target.id).style('opacity', 0.5))
          d3.select('#'+event.target.id).style('opacity', 0.5)
          this.overlay.remove()
          this.gCam.lower()
        }
      })


    this.minimapScatter = this.miniMap
      .selectAll('g')
      .data(this.parentNodes)
      .join('g')
      .append('circle')
      .attr("r", (d,i)=>{
          return  this.parentNodes[i].children.length * 1.25
      })
      .attr("class", 'inbox presentCircle')
      .attr("id", (d,i)=>{
        return 'A' + this.parentNodes[i].children[0].data.group
      })
  
    this.simulation = d3.forceSimulation(this.parentNodes)
      .force('charge', d3.forceManyBody().strength(20))
      .force('center', d3.forceCenter(this.width / 2, this.height / 2))
      .force('collision', d3.forceCollide().radius(30))
      .on('tick', ticked)

    this.miniSimulation =  d3.forceSimulation(this.parentNodes) 
      .force('charge', d3.forceManyBody().strength(20))
      .force('center', d3.forceCenter(this.width / 2, this.height / 2))
      .force('collision', d3.forceCollide().radius(30))
      .on('tick', miniMapTicked)
    
    this.childScatter = this.childgCam
      .selectAll('circle')
      .data((d,i)=>{
        return this.parentNodes[i].children
      })
      .join('circle')
      .attr('r', 1.5)
      .attr('class', 'inboxCircle presentCircle')
    
    this.onZoom = (event)=>{
      if ((event.type == 'brush')) 
        {return null}

      this.k = event.transform.k
      this.tx = event.transform.x 
      this.ty = event.transform.y 
      if(this.bFlag == true){
        this.userActionStatus.unshift(1)
        this.userActionStatus.pop()
        this.arrayK.unshift(this.k)
        this.arrayK.pop()
        if(this.arrayK[2]>this.arrayK[1] && this.arrayK[1]>this.arrayK[0] == false &&
          this.arrayK[0] !=0 && this.arrayK[1] != 0 && this.arrayK[2] != 0){
          console.log('줌아웃 변환 보정')
          this.tFlag = false
          this.userActionStatus = [0,0,0,0,0]
          this.i = this.i + 1
        }
        else if(this.arrayK[2]<this.arrayK[1] && this.arrayK[1]<this.arrayK[0] == false &&
          this.arrayK[0] !=0 && this.arrayK[1] != 0 && this.arrayK[2] != 0){
          console.log('줌인 변환 보정')
          this.tFlag = false
          this.userActionStatus = [0,0,0,0,0]
          this.i = this.i + 1
        }
        else if(this.arrayK[2]==this.arrayK[1] && this.arrayK[1]!=this.arrayK[0] &&
          this.arrayK[0] !=0 && this.arrayK[1] != 0 && this.arrayK[2] != 0){
          console.log('패닝 변환 보정')
          this.tFlag = false
          this.userActionStatus = [0,0,0,0,0]
          this.i = this.i + 1
        }
      }
      if(this.uFlag == true){
        this.childgCam.attr('transform', event.transform)
        this.brush.move(this.svgBrush, [
          [this.scaleX(this.k).invert(-this.tx), 
            this.scaleY(this.k).invert(-this.ty)],
          [this.scaleX(this.k).invert(-this.tx + this.width), 
            this.scaleY(this.k).invert(-this.ty + this.height)],
      ])
      } else {
        this.gCam.attr('transform', event.transform)
        this.childgCam.attr('transform', event.transform)
        this.brush.move(this.svgBrush, [
          [this.scaleX(this.k).invert(-this.tx), 
            this.scaleY(this.k).invert(-this.ty)],
          [this.scaleX(this.k).invert(-this.tx + this.width), 
            this.scaleY(this.k).invert(-this.ty + this.height)],
      ])
      }
      
    }

    this.zooms = d3.zoom()
      .on('zoom', this.onZoom)
      
    this.svgBrush = this.miniMap
      .append('g')

    this.onBrush = (event)=>{
      if (event.type == 'zoom') 
        {return null;}

      if (Array.isArray(event.selection)){
        this.brushX = event.selection[0][0]
        this.brushY = event.selection[0][1]
        this.brushSelection = event.selection
        this.zoomscale = d3.zoomTransform(this.svgMap.node()).k
      }
    }

    this.brush = d3.brush()
      .extent([[0,0], [this.width, this.height]])
      .on("brush", this.onBrush)

    this.svgMap.call(this.zooms)
      .on("dblclick.zoom", null)


    this.svgBrush.call(this.brush)
    this.brush.move(this.svgBrush, [
      [0,0],
      [this.width * this.k, this.height * this.k]
    ])
    this.miniMap.selectAll('.handle').remove()
    this.miniMap.selectAll('.overlay').remove()
    
    this.childScatter
      .on('mouseover', (event)=>{
        if(event.ctrlKey){
          console.log('#' + event.target.id, event.target)
          console.log(this.k)
          d3.selectAll('#' + event.target.id).attr('r', ()=>{
            return 2*(1/this.k)*20 + 2
          })
        } else {
         //
        }
      })
      .on('mouseout', (event)=>{
        d3.selectAll('#' + event.target.id).attr('r', '1.5').attr('fill-opacity', 1)
        this.overlay.remove()
      })
      .on('click', (event)=>{
        if(event.ctrlKey){
          console.log(event.target.id, '장바구니로')
        }
      })
      document.onkeydown = (e)=>{
        let keypress = e.key
        console.log(keypress)
        if(keypress == 'Control') {
          this.scatter.style('opacity', 0.5)
          this.overlay.remove()
          this.gCam.lower()
        }
      }
  },

 
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
circle {
  fill: cadetblue;
  opacity: 0.1;
}
.outofBox {
  fill: red;
  opacity: 0.5;
}
.inbox {
  /* fill: rgb(0, 183, 255); */
  stroke: rgb(0, 183, 255);
  opacity: 0.5;
}
.clientGrey{
  fill: grey;
  opacity: 0.7;
}
.floorplan{
  fill: yellow;
  opacity: 0.9;
}
.inboxCircle{
  fill: rgb(0, 119, 255);
  opacity: 0.9;
}
.outboxCircle{
  fill: rgb(255, 0, 0);
  opacity: 0.9;
}
.presentCircle{
}
.previousCircle{
  display: none;
}
</style>
