<template>
  <div class="toolsbox">
    <button @click="showOrHideTools" class="showOrHideBtn">
      <i class="iconfont icon-gongjuxiang"></i>
    </button>

    <transition name="fade">
      <div class="tools" v-if="isShowTools">
        <div class="tool">
          <div class="toolName">大小:</div>
          <el-slider v-model="lineWidth" :min="5" :max="50" class="toolBtn"></el-slider>
        </div>
        <div class="tool">
          <div class="toolName">颜色:</div>
          <el-color-picker v-model="color" class="toolBtn" :predefine="predefineColors"></el-color-picker>
        </div>
        <div class="tool">
          <div class="toolName">撤销:</div>
          <button class="toolBtn" @click="handleBack">
            <i class="iconfont icon-chexiao"></i>
          </button>
        </div>
        <div class="tool">
          <div class="toolName">清除:</div>
          <button class="toolBtn" @click="handleClear">
            <i class="iconfont icon-clear"></i>
          </button>
        </div>
        <div class="tool">
          <div class="toolName">保存:</div>
          <button class="toolBtn" @click="handleSave">
            <i class="iconfont icon-xiazai"></i>
          </button>
        </div>
        <div class="tool">
          <div class="toolName">重播:</div>
          <button class="toolBtn" @click="handleReplay">
            <i class="iconfont icon-bofang"></i>
          </button>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
export default {
  name: "tools",
  data() {
    return {
      isShowTools: true,
      lineWidth: 10,
      color: "#666",
      predefineColors: [
        "#ff4500",
        "#ff8c00",
        "#ffd700",
        "#90ee90",
        "#00ced1",
        "#1e90ff",
        "#c71585"
      ]
    };
  },
  methods: {
    showOrHideTools() {
      this.isShowTools = !this.isShowTools;
    },
    handleBack() {
      this.$emit("handleBack");
    },
    handleClear() {
      this.$emit("handleClear");
    },
    handleSave() {
      this.$emit("handleSave");
    },
    handleReplay() {
      this.$emit("handleReplay");
    }
  },
  watch: {
    lineWidth() {
      this.$emit("changeSize", this.lineWidth);
    },
    color() {
      this.$emit("changeColor", this.color);
    }
  }
};
</script>

<style scoped>
.toolsbox {
  position: fixed;
  background: #eee;
  border-radius: 10px;
  margin: 10px;
  width: 120px;
}
.showOrHideBtn {
  width: 120px;
  cursor: pointer;
}
.iconfont {
  font-size: 28px;
  cursor: pointer;
}
button {
  border: none;
  outline: none;
  background-color: transparent;
}
.tools {
  padding: 10px;
}
.tool {
  display: flex;
  align-items: center;
  margin: 5px 0;
}
.tool .toolName {
  width: 50px;
  font-size: 18px;
}
.tool .toolBtn {
  flex: 1;
  text-align: center;
}
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter,
.fade-leave-to {
  opacity: 0;
}
</style>
