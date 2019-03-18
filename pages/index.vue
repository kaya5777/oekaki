<template>
  <section class="container">
    <div class="CanvasArea">
      <canvas id="canvas"
        v-on:mousedown="handleMouseDown"
        v-on:mouseup="handleMouseUp"
        v-on:mousemove="handleMouseMove"
        v-on:touchstart="handleTouchDown"
        v-on:touchmove="handleTouchMove"
        v-on:touchend="handleTouchUp"
        width="300px" height="300px"></canvas>
      <div>
　      <input class="form-control" type="color" v-model="lineColor" />
        <select v-model="lineWidth">
          <option v-for="i in 10" v-bind:value="i">{{i}}</option>
        </select>
　　　　　<input type="checkbox" id="checkbox" v-model="eraseMode">
        <label for="checkbox">消しゴムモード</label>
      </div>
      <div>
        <button v-on:click="saveImage">保存</button>
        <button v-on:click="resetImage">やり直す</button>
        <a class="twitter-share-button"
          href="https://twitter.com/intent/tweet">
        Tweet</a>
      </div>
      <p>{{mouse}}</p>
    </div>
  </section>
</template>

<script>
export default {
  data() {
    return {
      mouse: {
        position: {
          x: 0,
          y: 0
        },
        down: false
      },
      canvas: null,
      context: null,
      lineWidth: 1,
      lineColor: "#000000",
      eraseMode: false,
    }
  },
  computed: {
    currentMouse: function () {
      const rect = this.canvas.getBoundingClientRect()
      return {
        x: this.mouse.position.x - rect.left,
        y: this.mouse.position.y - rect.top
      }
    }
  },
  methods: {
    draw(event) {
      if(this.mouse.down) {
        this.context.lineWidth = this.lineWidth
        if(this.eraseMode) {
          this.context.strokeStyle = "#ffffff"
        } else {
          this.context.strokeStyle = this.lineColor
        }

        this.context.lineTo(this.currentMouse.x, this.currentMouse.y)
        this.context.stroke()
      }
    },
    handleMouseDown(event) {
      this.mouse.down = true
      this.mouse.position = {
        x: event.pageX,
        y: event.pageY
      }
      this.context.beginPath()
      this.context.moveTo(this.currentMouse.x, this.currentMouse.y)
    },
    handleMouseUp() {
      this.mouse.down = false
      this.context.closePath()
    },
    handleMouseMove(event) {
      this.mouse.position = {
        x: event.pageX,
        y: event.pageY
      }
      this.draw(event)
    },
    handleTouchDown: function(event) {
      const t = event.changedTouches[0]
      this.mouse.down = true
      this.mouse.position = {
        x: t.pageX,
        y: t.pageY
      }

      this.context.beginPath()
      this.context.moveTo(this.currentMouse.x, this.currentMouse.y)
    },
    handleTouchMove: function(event) {
      const t = event.changedTouches[0]
      this.mouse.position = {
        x: t.pageX,
        y: t.pageY
      }
      this.draw(event)
    },
    handleTouchUp: function(event) {
      this.mouse.down = false
      this.context.closePath()
    },
    saveImage: function(event) {
      var png = this.canvas.toDataURL('image/png')
      png = png.replace(/^.*,/, '')

      var bin = atob(png)
      var buffer = new Uint8Array(bin.length)
      for(var i = 0; i < bin.length; i++){
        buffer[i] = bin.charCodeAt(i)
      }
      var blob = new Blob([buffer.buffer], {
        type: 'image/png'
      })

      var url = (window.URL || window.webkitURL)
      var dataUrl = url.createObjectURL(blob)
      var event = document.createEvent("MouseEvents")
      event.initMouseEvent("click", true, false, window,
        0, 0, 0, 0, 0, false, false, false, false, 0, null)
      var a = document.createElementNS("http://www.w3.org/1999/xhtml", "a")
      a.href = dataUrl
      a.download = "result.png"
      a.dispatchEvent(event)
    },
    resetImage: function(event) {
      this.context.clearRect(0, 0, 300, 300)
    },
  },
  mounted() {
    this.canvas = document.getElementById("canvas")
    this.context = this.canvas.getContext("2d")
  }
}
</script>

<style>
canvas {
  border-style: solid
}
</style>
