<template>
  <div
    ref="dragVerify"
    class="drag_verify"
    :style="dragVerifyStyle"
    @mousemove="dragMoving"
    @mouseup="dragFinish"
    @mouseleave="dragFinish"
    @touchmove="dragMoving"
    @touchend="dragFinish"
  >

    <div
      class="dv_progress_bar"
      :class="{goFirst2:isOk}"
      ref="progressBar"
      :style="progressBarStyle"
    >

    </div>
    <div
      class="dv_text"
      :style="textStyle"
      ref="message"
    >
      <slot
        name="textBefore"
        v-if="$slots.textBefore"
      ></slot>
      {{message}}
      <slot
        name="textAfter"
        v-if="$slots.textAfter"
      ></slot>
    </div>

    <div
      class="dv_handler dv_handler_bg"
      :class="{goFirst:isOk}"
      @mousedown="dragStart"
      @touchstart="dragStart"
      ref="handler"
      :style="handlerStyle"
    >
      <i :class="handlerIcon"></i>
    </div>

  </div>
</template>
<script>
export default {
  name: "dragVerify",
  props: {
    isPassing: {
      type: Boolean,
      default: false
    },
    width: {
      type: Number,
      default: 2.5
    },
    height: {
      type: Number,
      default: 0.4
    },
    text: {
      type: String,
      default: "swiping to the right side"
    },
    successText: {
      type: String,
      default: "success"
    },
    background: {
      type: String,
      default: "#eee"
    },
    progressBarBg: {
      type: String,
      default: "#76c61d"
    },
    completedBg: {
      type: String,
      default: "#76c61d"
    },
    circle: {
      type: Boolean,
      default: false
    },
    radius: {
      type: String,
      default: "4px"
    },
    handlerIcon: {
      type: String
    },
    successIcon: {
      type: String
    },
    handlerBg: {
      type: String,
      default: "#fff"
    },
    textSize: {
      type: String,
      default: "14px"
    },
    textColor: {
      type: String,
      default: "#333"
    }
  },
  mounted: function() {
    const dragEl = this.$refs.dragVerify;
    dragEl.style.setProperty("--textColor", this.textColor);
    dragEl.style.setProperty("--width", Math.floor(this.width / 2) + "rem");
    dragEl.style.setProperty("--pwidth", -Math.floor(this.width / 2) + "rem");
  },
  computed: {
    handlerStyle: function() {
      return {
        left: "0",
        width: this.height + "rem",
        height: this.height + "rem",
        background: this.handlerBg
      };
    },
    message: function() {
      return this.isPassing ? this.successText : this.text;
    },
    dragVerifyStyle: function() {
      return {
        width: this.width + "rem",
        height: this.height + "rem",
        lineHeight: this.height + "rem",
        background: this.background,
        borderRadius: this.circle ? this.height / 2 + "rem" : this.radius
      };
    },
    progressBarStyle: function() {
      return {
        background: this.progressBarBg,
        height: this.height + "rem",
        borderRadius: this.circle
          ? this.height / 2 + "rem 0 0 " + this.height / 2 + "rem"
          : this.radius
      };
    },
    textStyle: function() {
      return {
        height: this.height + "rem",
        width: this.width + "rem",
        fontSize: this.textSize
      };
    }
  },
  data() {
    return {
      isMoving: false,
      x: 0,
      isOk: false
    };
  },
  methods: {
    dragStart: function(e) {
      if (!this.isPassing) {
        this.isMoving = true;
        var handler = this.$refs.handler;
        this.x =
          (e.pageX || e.touches[0].pageX) -
          parseFloat(handler.style.left.replace("rem", ""), 10);
      }
      var that = this
      document.onmousemove = function(ev) {
        let el = ev || event;
        that.dragMoving(el);
        return false;
      }
      document.onmouseup = function(ev) {
        let el = ev || event;
        that.dragFinish(el);
        document.onmousemove = null;
        return false;
      }
      e.stopPropagation();
      e.preventDefault();
      this.$emit("handlerMove");
    },
    dragMoving: function(e) {
      if (this.isMoving && !this.isPassing) {
        var _x = (e.pageX || e.touches[0].pageX) - this.x;
        var handler = this.$refs.handler;
        if (_x > 0 && _x/100 <= this.width - this.height) {
          handler.style.left = _x/100 + "rem";
          console.log(this.width)
          this.$refs.progressBar.style.width = _x/100 + this.height / 2 + "rem";
        } else if (_x/100 > this.width - this.height) {
          handler.style.left = this.width - this.height + "rem";
          this.$refs.progressBar.style.width =
            this.width - this.height / 2 + "rem";
          this.passVerify();
        }
      }
    },
    dragFinish: function(e) {
      if (this.isMoving && !this.isPassing) {
        var _x = (e.pageX || e.changedTouches[0].pageX) - this.x;
        if (_x/100 < this.width - this.height) {
          this.isOk = true;
          var that = this;
          setTimeout(function() {
            that.$refs.handler.style.left = "0";
            that.$refs.progressBar.style.width = "0";
            that.isOk = false;
          }, 500);
        } else {
          var handler = this.$refs.handler;
          handler.style.left = this.width - this.height + "rem";
          this.$refs.progressBar.style.width =
            this.width - this.height / 2 + "rem";
          this.passVerify();
        }
        this.isMoving = false;
      }
    },
    passVerify: function() {
      this.$emit("update:isPassing", true);
      this.isMoving = false;
      var handler = this.$refs.handler;
      handler.children[0].className = this.successIcon;
      this.$refs.progressBar.style.background = this.completedBg;
      this.$refs.message.style["-webkit-text-fill-color"] = "unset";
      this.$refs.message.style.animation = "slidetounlock2 3s infinite";
      this.$refs.message.style.color = "#fff";
      this.$emit("passcallback");
    },
    reset: function() {
      const oriData = this.$options.data();
      for (const key in oriData) {
        if (oriData.hasOwnProperty(key)) {
          this.$set(this, key, oriData[key]);
        }
      }
      var handler = this.$refs.handler;
      var message = this.$refs.message;
      handler.style.left = "0";
      this.$refs.progressBar.style.width = "0";
      handler.children[0].className = this.handlerIcon;
      message.style["-webkit-text-fill-color"] = "transparent";
      message.style.animation = "slidetounlock 3s infinite";
      message.style.color = this.background;
    }
  }
};
</script>
<style scoped>
.drag_verify {
  position: relative;
  background-color: #e8e8e8;
  text-align: center;
  overflow: hidden;
}
.drag_verify .dv_handler {
  position: absolute;
  top: 0;
  left: 0;
  cursor: move;
}
.drag_verify .dv_handler i {
  color: #666;
  padding-left: 0;
  font-size: 0.16rem;
}
.drag_verify .dv_handler .el-icon-circle-check {
  color: #6c6;
  margin-top: 0.09rem;
}
.drag_verify .dv_progress_bar {
  position: absolute;
  height: 0.34rem;
  width: 0;
}
.drag_verify .dv_text {
  position: absolute;
  top: 0;
  color: transparent;
  -moz-user-select: none;
  -webkit-user-select: none;
  user-select: none;
  -o-user-select: none;
  -ms-user-select: none;
  background: -webkit-gradient(
    linear,
    left top,
    right top,
    color-stop(0, var(--textColor)),
    color-stop(0.4, var(--textColor)),
    color-stop(0.5, #fff),
    color-stop(0.6, var(--textColor)),
    color-stop(1, var(--textColor))
  );
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  -webkit-text-size-adjust: none;
  animation: slidetounlock 3s infinite;
}
.drag_verify .dv_text * {
  -webkit-text-fill-color: var(--textColor);
}
.goFirst {
  left: 0 !important;
  transition: left 0.5s;
}
.goFirst2 {
  width: 0 !important;
  transition: width 0.5s;
}
</style>
<style>
@-webkit-keyframes slidetounlock {
  0% {
    background-position: var(--pwidth) 0;
  }
  100% {
    background-position: var(--width) 0;
  }
}
@-webkit-keyframes slidetounlock2 {
  0% {
    background-position: var(--pwidth) 0;
  }
  100% {
    background-position: var(--pwidth) 0;
  }
}
</style>
