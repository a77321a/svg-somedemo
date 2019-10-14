<!--
 * @Descripttion: 
 * @Author: 辛顺宁
 * @Date: 2019-10-11 16:39:49
 * @LastEditors: 辛顺宁
 * @LastEditTime: 2019-10-12 18:23:02
 -->
<template>
  <div id="app">
    <div id="apps"></div>
    <ul class="list">
      <li @click="drawRect">rect</li>
      <li @click="drawText">text</li>
      <li @click="drawPencil">画笔</li>
      <li @click="drawElazer">橡皮擦</li>
      <li @click="removeAll">删除</li>
      <li @click="drawImage(half)">半对</li>
      <li @click="drawImage(right)">全对</li>
      <li @click="drawImage(wrong)">错误</li>
      <li @click="prev">回退</li>
      <li @click="saveImage">保存</li>
    </ul>
    <img :src="canvasSrc" alt />
    <canvas id="canvasId"></canvas>
  </div>
</template>

<script>
import canvg from 'canvg'
import saveSvgAsPng from 'save-svg-as-png'
import html2canvas from 'html2canvas'
import SVG from 'svg.js'
import './svg.for'
import img1 from './assets/logo.png'
import wrong from './assets/close.png'
import half from './assets/half.png'
import right from './assets/right.png'
export default {
  name: 'app',
  data() {
    return {
      img1,
      half,
      wrong,
      right,
      canvasSrc: '',
      draw: null,
      rect: false,
      text: false,
      pencil: false,
      imgaeU: false,
      elazer: false,
      imageUrl: ''
    }
  },
  mounted() {
    let imgurl = img1
    // let imgurl =
    //   'http://admin.dingcourse.com/uploads/images/20191012/4720a3ab8bff89245feceb8748fca5ff.jpg'
    this.draw = SVG('apps').size('100%', 400)
    let drawimage = this.draw.image(imgurl, '100%', '100%')
    drawimage.attr('class', 'item')
    drawimage.attr('crossOrigin', 'Anonymous')
    var fobj
    var isBlur = true

    this.draw.click(e => {
      if (!e.target.classList.contains('item') && this.elazer) {
        e.target.remove()
      }
      // if (e.target.classList.contains('item') && isBlur == false && this.text) {
      //   return
      // }
      if (this.imgaeU) {
        let min = this.draw.image(this.imageUrl, 20, 20)
        min.attr('x', e.clientX - 10)
        min.attr('y', e.clientY - 10)
        min.attr('class', 'itemhover')
      }
      if (this.text) {
        let odom = document.createElement('div')
        odom.setAttribute('contenteditable', 'true')
        let that = this
        odom.onblur = function(e) {
          setTimeout(() => {
            e.target.setAttribute('contenteditable', 'false')
            e.target.style.userSelect = 'none'
            e.target.style.pointerEvents = 'none'
            if (e.target.innerText == '') {
              that.draw.last().remove()
            }
            isBlur = true
          }, 500)
        }
        if (e.target.classList.contains('item') && isBlur == false) {
          return
        } else {
          fobj = this.draw
            .foreignObject()
            .attr({ transform: `translate(${e.offsetX},${e.offsetY})` })
          fobj.appendChild(odom)
          odom.focus()
          isBlur = false
        }
      }
    })
    let arr = [],
      isDown = false,
      polyLine = null,
      rectX = '',
      rectY = '',
      rectD = null,
      rectM = false,
      write = false
    this.draw.mousedown(e => {
      if (this.pencil) {
        arr = []
        let str = `${e.offsetX},${e.offsetY}`
        arr.push(str)
        polyLine = this.draw
          .polyline(arr.join(' '))
          .fill('none')
          .stroke({ width: 2, color: 'red' })
        isDown = true
      }
      if (this.rect) {
        rectM = true
        rectX = e.offsetX
        rectY = e.offsetY
        rectD = this.draw
          .rect(0, 0)
          .fill('none')
          .stroke({ width: 2, color: 'red' })
        rectD.attr('x', rectX), rectD.attr('y', rectY)
      }
    })
    this.draw.mousemove(e => {
      if (isDown) {
        let str = `${e.offsetX},${e.offsetY}`
        arr.push(str)
        polyLine.plot(arr.join(' '))
      }
      if (this.rect && rectM) {
        let width = e.offsetX - rectX
        let height = e.offsetY - rectY
        if (width < 0 || height < 0) {
          rectM = false
          return
        }
        rectD.attr('width', width), rectD.attr('height', height)
      }
    })
    this.draw.mouseup(e => {
      if (isDown) {
        let str = `${e.offsetX},${e.offsetY}`
        arr.push(str)
        polyLine.plot(arr.join(' '))
        polyLine.attr('class', 'itemhover')
        arr = []
        isDown = false
      }
      if (this.rect && rectM) {
        let width = e.offsetX - rectX
        let height = e.offsetY - rectY
        if (width < 0 || height < 0) {
          rectM = false
          return
        }
        rectD.attr('width', width), rectD.attr('height', height)
        rectM = false
      }
    })
  },
  methods: {
    drawImage(url) {
      if (this.imageUrl != url) {
        this.imageUrl = url
        this.imgaeU = true
      } else {
        this.imgaeU = !this.imgaeU
      }
      this.elazer = false
      this.pencil = false
      this.text = false
      this.rect = false
    },
    drawRect() {
      this.rect = !this.rect
      this.elazer = false
      this.pencil = false
      this.text = false
      this.imgaeU = false
    },
    drawText() {
      this.text = !this.text
      this.elazer = false
      this.pencil = false
      this.rect = false
      this.imgaeU = false
    },
    drawPencil() {
      this.pencil = !this.pencil
      this.elazer = false
      this.text = false
      this.rect = false
      this.imgaeU = false
    },
    drawElazer() {
      this.elazer = !this.elazer
      this.pencil = false
      this.text = false
      this.rect = false
      this.imgaeU = false
    },
    removeAll() {
      let that = this
      this.draw.each(function(i, children) {
        console.log(this.attr('x'))
        if (!this.node.classList.contains('item')) {
          this.remove()
        }
      })
    },
    prev() {
      if (!this.draw.last().node.classList.contains('item'))
        this.draw.last().remove()
    },
    saveImage() {
      let printbox = window.document.querySelector('svg')

      // saveSvgAsPng(printbox, 'diagram.png')
      let canva = canvg('canvasId', printbox)
      var data = 'data:image/svg+xml,' + this.draw.node.outerHtml
      this.canvasSrc = data
      setTimeout(() => {
        html2canvas(printbox, {
          dpi: 144,
          useCORS: true
        }).then(canvas => {
          this.canvasSrc = canvas.toDataURL('image/jpeg')
          // this.canvas = canvas
          // nodeListVideo.forEach(i => {
          //   i.style.display = 'block'
          // })
        })
      }, 0)
    }
  }
}
</script>

<style>
#app {
}
.item {
  /* cursor: pointer; */
}
.itemhover:hover {
  cursor: pointer;
  opacity: 0.7;
}
.list {
  list-style-type: none;
  margin-top: 30px;
}
.list li {
  float: left;
  padding: 10px;
  cursor: pointer;
  display: block;
  background-color: #aaa;
}
.mydiv {
  border: 1px solid red;
}
</style>
