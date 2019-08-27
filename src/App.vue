<template>
  <div id="app">
    <Tools
      class="tools"
      @changeSize="changeSize"
      @changeColor="changeColor"
      @handleBack="back"
      @handleClear="clear"
      @handleSave="save"
      @handleReplay="replay"
    ></Tools>
    <canvas id="canvas"></canvas>
  </div>
</template>

<script>
import Tools from "./components/Tools.vue";
export default {
  name: "app",
  components: {
    Tools
  },
  data() {
    return {
      lineWidth: 10,
      color: "#666",
      canvas: null,
      context: null,
      drawing: false,
      myTimeStamp: undefined,
      minTimeDiff: 8, // 值越大 记录的点越少
      minPointDistance: 8,
      stack: [],
      path: []
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    changeSize(newSize) {
      this.lineWidth = newSize;
    },
    changeColor(newColor) {
      this.color = newColor;
    },
    init() {
      this.canvas = document.getElementById("canvas");
      this.context = this.canvas.getContext("2d");
      this.setCanvasSize();
      this.context.lineCap = "round";
      this.context.lineJoin = "round";
      this.context.fillStyle = "white";
      this.context.fillRect(0, 0, this.canvas.width, this.canvas.height);
      window.onresize = this.setCanvasSize;
      this.canvas.addEventListener("mousedown", this.handleMousedown);
      this.canvas.addEventListener("touchstart", this.handleTouchstart);
    },
    setCanvasSize() {
      this.canvas.width = document.documentElement.clientWidth;
      this.canvas.height = document.documentElement.clientHeight;
    },
    handleMousedown(e) {
      this.drawing = true;
      this.myTimeStamp = new Date().getTime();
      let x = e.clientX,
        y = e.clientY;
      this.path.push({ width: this.lineWidth, color: this.color });
      this.path.push({ x, y });
      this.stack.push(this.path);
      this.drawLine();
      this.canvas.addEventListener("mousemove", this.handleMousemove);
      this.canvas.addEventListener("mouseup", this.handleMouseup);
    },
    handleMousemove(e) {
      // 类似函数节流 控制笔迹数量
      if (new Date().getTime() - this.myTimeStamp < this.minTimeDiff) {
        return;
      }
      let x = e.clientX,
        y = e.clientY;
      // 判断两个点是否太靠近 太近的点不要
      let x0 = this.path[this.path.length - 1].x,
        y0 = this.path[this.path.length - 1].y;
      if (
        Math.abs(x - x0) < this.minPointDistance &&
        Math.abs(y - y0) < this.minPointDistance
      ) {
        return;
      }
      this.path.push({ x, y });
      this.stack.pop();
      this.stack.push(this.path);
      this.drawLine();
      this.myTimeStamp = new Date().getTime();
    },
    handleMouseup() {
      this.drawing = false;
      this.path = [];
      this.canvas.removeEventListener("mousemove", this.handleMousemove);
      this.canvas.removeEventListener("mouseup", this.handleMouseup);
    },
    handleTouchstart(e) {
      e.preventDefault();
      this.drawing = true;
      this.myTimeStamp = new Date().getTime();
      let x = e.touches[0].clientX,
        y = e.touches[0].clientY;
      this.path.push({ width: this.lineWidth, color: this.color });
      this.path.push({ x, y });
      this.stack.push(this.path);
      this.drawLine();
      this.canvas.addEventListener("touchmove", this.handleTouchmove);
      this.canvas.addEventListener("touchend", this.handleTouchend);
    },
    handleTouchmove(e) {
      e.preventDefault();
      // 类似函数节流 控制笔迹数量
      if (new Date().getTime() - this.myTimeStamp < this.minTimeDiff) {
        return;
      }
      let x = e.touches[0].clientX,
        y = e.touches[0].clientY;
      // 判断两个点是否太靠近 太近的点不要
      let x0 = this.path[this.path.length - 1].x,
        y0 = this.path[this.path.length - 1].y;
      if (
        Math.abs(x - x0) < this.minPointDistance &&
        Math.abs(y - y0) < this.minPointDistance
      ) {
        return;
      }
      this.path.push({ x, y });
      this.stack.pop();
      this.stack.push(this.path);
      this.drawLine();
      this.myTimeStamp = new Date().getTime();
    },
    handleTouchend(e) {
      e.preventDefault();
      this.drawing = false;
      this.path = [];
      document.removeEventListener("mousemove", this.handleTouchmove);
      document.removeEventListener("mouseup", this.handleTouchend);
    },
    drawLine() {
      this.context.fillStyle = "white";
      this.context.fillRect(0, 0, this.canvas.width, this.canvas.height);
      this.stack.forEach(path => {
        let length = path.length;
        path.forEach((value, index, array) => {
          if (index === 0) {
            // 这条线的样式
            this.context.lineWidth = value.width;
            this.context.strokeStyle = value.color;
          } else if (index === 1) {
            // 第一个点
            this.context.beginPath();
            this.context.moveTo(value.x, value.y);
            this.context.lineTo(value.x, value.y);
          } else {
            // 贝塞尔优化
            let x1 = array[index - 1].x,
              y1 = array[index - 1].y,
              x2 = value.x,
              y2 = value.y;
            let x3 = x1 / 2 + x2 / 2,
              y3 = y1 / 2 + y2 / 2;
            this.context.quadraticCurveTo(x1, y1, x3, y3);
          }
          if (index === length - 1) {
            this.context.lineTo(value.x, value.y);
            this.context.stroke();
          }
        });
      });
    },
    back() {
      this.stack.splice(this.stack.length - 1, 1);
      this.path.length = 0;
      this.drawLine();
    },
    clear() {
      this.path.length = 0;
      this.stack.length = 0;
      this.drawLine();
    },
    save() {
      let a = document.createElement("a");
      a.href = this.canvas.toDataURL("image/png");
      let ua = window.navigator.userAgent.toLowerCase();
      if (ua.match(/iphone|android|ipad/)) {
        a.target = "_blank";
      } else {
        a.download = "myCanvas";
      }
      a.click();
      document.body.removeChild(a);
    },
    replay() {
      let array = this.stack.reduce((reducer, value) => {
        return reducer.concat(value);
      });
      let maxLength = array.length;
      let length = 2;
      let draw = () => {
        this.context.fillStyle = "white";
        this.context.fillRect(0, 0, this.canvas.width, this.canvas.height);
        if (length <= maxLength) {
          for (let i = 0; i < length; i++) {
            let value = array[i];
            if (value.width) {
              this.context.lineWidth = value.width;
              this.context.strokeStyle = value.color;
            } else {
              if (array[i - 1].width) {
                // 第一个点
                this.context.beginPath();
                this.context.moveTo(value.x, value.y);
                this.context.lineTo(value.x, value.y);
              } else {
                let x1 = array[i - 1].x,
                  y1 = array[i - 1].y,
                  x2 = value.x,
                  y2 = value.y;
                let x3 = x1 / 2 + x2 / 2,
                  y3 = y1 / 2 + y2 / 2;
                this.context.quadraticCurveTo(x1, y1, x3, y3);
              }
              if (i === length - 1 || array[i + 1].width) {
                // 最后一个点
                this.context.lineTo(value.x, value.y);
                this.context.stroke();
              }
            }
          }
        }
        length += 1;
        if (length > maxLength) {
          cancelAnimationFrame(raf);
          return;
        }
        if (this.drawing) {
          cancelAnimationFrame(raf);
          this.drawLine();
          return;
        }
        requestAnimationFrame(draw);
      };
      let raf = requestAnimationFrame(draw);
    }
  }
};
</script>

<style>
body {
  margin: 0;
  padding: 0;
  position: relative;
  overflow: hidden;
}
canvas {
  cursor: crosshair;
}
</style>
