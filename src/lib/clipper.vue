<template>
  <div 
    id="screenshot" 
    class="screenshot-container"
    @mouseup="e => !disabled && mouseUp(e)"
    @mousedown="e => !disabled && mouseDown(e)"
    @mousemove="e => !disabled && move(e)"
    :style="{ width: `${width}px`, height: `${height}px` }"
  >
    <div class="slot-wrapper">
      <slot></slot>
    </div>

    <transition name="screenshot">
      <div class="flash" v-if="tookScreenshot"></div>
    </transition>
    
    <div
      v-if="!disabled"
      class="overlay"
      :class="{ 'highlighting': isDragging }"
      :style="{ borderWidth: borderWidth }"
    ></div>

    <div
      v-if="!disabled"
      class="crosshairs"
      :class="{ hidden: isDragging }"
      :style="{ left: crossHairsLeft + 'px', top: crossHairsTop + 'px' }"
    ></div>

    <div
      v-if="!disabled"
      class="borderedBox"
      :class="{ hidden: !isDragging }"
      :style="{ left: boxLeft + 'px', top: boxTop + 'px', width: boxEndWidth + 'px', height: boxEndHeight + 'px' }"
    ></div>

    <span
      v-if="!disabled"
      class="tooltip"
      :class="{ hidden: !isDragging }"
      :style="{ left: toolTipLeft + 'px', top: toolTipTop + 'px' }"
    >{{ boxEndWidth }} x {{ boxEndHeight }}px</span>
  </div>
</template>

<script>
import html2canvas from "html2canvas";

var parent, parentTop, parentLeft;
var TOOLTIP_MARGIN;
var pixelRatio;

var edgeSize = 200;
var timer = null;

export default {
  props: [
    'disabled',
    'width',
    'height',
  ],

  data() {
    return {
      mouseIsDown: false,
      isDragging: false,

      borderWidth: "",

      startX: 0,
      startY: 0,
      endX: 0,
      endY: 0,

      crossHairsLeft: 0,
      crossHairsTop: 0,

      windowHeight: 0,
      windowWidth: 0,

      boxTop: 0,
      boxLeft: 0,
      boxEndWidth: 0,
      boxEndHeight: 0,

      toolTipLeft: 0,
      toolTipTop: 0,
      toolTipWidth: 0,
      toolTipHeight: 0,

      imageUrl: undefined,
      tookScreenshot: false,
    };
  },

  mounted() {
    if (this.disabled) return;

    TOOLTIP_MARGIN = +window.getComputedStyle(document.querySelector(".tooltip")).margin.split("px")[0];
    pixelRatio = window.devicePixelRatio;

    parent = document.querySelector("#screenshot");

    this.windowWidth = parent.clientWidth;
    this.windowHeight = parent.clientHeight;

    const viewportOffset = parent.getBoundingClientRect();
    parentTop = viewportOffset.top;
    parentLeft = viewportOffset.left;

    this.toolTipWidth = 110;
    this.toolTipHeight = 30;
  },

  methods: {
    move(e) {
      if (!this.isInContainer(e)) return;

      this.crossHairsLeft = e.pageX - parentLeft;
      this.crossHairsTop = e.pageY - parentTop;

      if (this.mouseIsDown) {
        this.scroll(e);

        var endY = this.endY = e.pageY - parentTop,
            endX = this.endX = e.pageX - parentLeft,
            startX = this.startX,
            startY = this.startY,
            windowWidth = this.windowWidth,
            windowHeight = this.windowHeight;

        if (endX >= startX && endY >= startY) {
          this.isDragging = true;

          this.borderWidth = startY + "px " + (windowWidth - endX) + "px " + (windowHeight - endY) + "px " + startX + "px";

          this.boxTop = startY;
          this.boxLeft = startX;
          this.boxEndWidth = endX - startX;
          this.boxEndHeight = endY - startY;

          this.toolTipLeft = endX;
          this.toolTipTop = endY;

          if (endX + this.toolTipWidth >= windowWidth) {
            this.toolTipLeft = windowWidth - this.toolTipWidth - (TOOLTIP_MARGIN * 2);
          }
           
          if (endY + this.toolTipHeight + (TOOLTIP_MARGIN * 2) >= windowHeight) {
            this.toolTipTop = windowHeight - this.toolTipHeight - (TOOLTIP_MARGIN * 2);
          }
        } else if (endX <= startX && endY >= startY) {
          this.isDragging = true;

          this.borderWidth = startY + "px " + (windowWidth - startX) + "px " + (windowHeight - endY) + "px " + endX + "px";

          this.boxLeft = endX;
          this.boxTop = startY;
          this.boxEndWidth = startX - endX;
          this.boxEndHeight = endY - startY;

          this.toolTipLeft = endX - this.toolTipWidth;
          this.toolTipTop = endY;
          
          if (endX - this.toolTipWidth <= 0) {
            this.toolTipLeft = TOOLTIP_MARGIN;
          }
          
          if (endY + this.toolTipHeight + (TOOLTIP_MARGIN * 2) >= windowHeight) {
            this.toolTipTop = windowHeight - this.toolTipHeight - (TOOLTIP_MARGIN * 2);
          }
        } else if (endX >= startX && endY <= startY) {
          this.isDragging = true;

          this.boxLeft = startX;
          this.boxTop = endY;
          this.boxEndWidth = endX - startX;
          this.boxEndHeight = startY - endY;

          this.borderWidth = endY + "px " + (windowWidth - endX) + "px " + (windowHeight - startY) + "px " + startX + "px";

          this.toolTipLeft = endX;
          this.toolTipTop = endY - this.toolTipHeight;
          
          if (endX + this.toolTipWidth >= windowWidth) {
            this.toolTipLeft = windowWidth - this.toolTipWidth - (TOOLTIP_MARGIN * 2);
          }
          
          if (endY - this.toolTipHeight <= 0) {
            this.toolTipTop = TOOLTIP_MARGIN;
          }
        } else if (endX <= startX && endY <= startY) {
          this.isDragging = true;

          this.boxLeft = endX;
          this.boxTop = endY;
          this.boxEndWidth = startX - endX;
          this.boxEndHeight = startY - endY;

          this.borderWidth = endY + "px " + (windowWidth - startX) + "px " + (windowHeight - startY) + "px " + endX + "px";

          this.toolTipLeft = endX - this.toolTipWidth;
          this.toolTipTop = endY - this.toolTipHeight;
          
          if (endX - this.toolTipWidth <= 0) {
            this.toolTipLeft = TOOLTIP_MARGIN;
          }
          
          if (endY - this.toolTipHeight <= 0) {
            this.toolTipTop = TOOLTIP_MARGIN;
          }
        } else {
          this.isDragging = false;
        }
      }
    },

    scroll(event) {
      let viewportX = event.clientX;
      let viewportY = event.clientY;

      let viewportWidth = document.documentElement.clientWidth;
      let viewportHeight = document.documentElement.clientHeight;

      let edgeTop = edgeSize;
      let edgeLeft = edgeSize;
      let edgeBottom = viewportHeight - edgeSize;
      let edgeRight = viewportWidth - edgeSize;

      let isInLeftEdge = viewportX < edgeLeft;
      let isInRightEdge = viewportX > edgeRight;
      let isInTopEdge = viewportY < edgeTop;
      let isInBottomEdge = viewportY > edgeBottom;

      if (!(isInLeftEdge || isInRightEdge || isInTopEdge || isInBottomEdge)) {
        clearTimeout(timer);
        return;
      }

      let documentWidth = Math.max(
				document.body.scrollWidth,
				document.body.offsetWidth,
				document.body.clientWidth,
				document.documentElement.scrollWidth,
				document.documentElement.offsetWidth,
				document.documentElement.clientWidth
      );
      
			let documentHeight = Math.max(
				document.body.scrollHeight,
				document.body.offsetHeight,
				document.body.clientHeight,
				document.documentElement.scrollHeight,
				document.documentElement.offsetHeight,
				document.documentElement.clientHeight
      );
      
      let maxScrollX = documentWidth - viewportWidth;
      let maxScrollY = documentHeight - viewportHeight;

      (function checkForWindowScroll() {
        clearTimeout(timer);
        if (adjustWindowScroll()) {
          timer = setTimeout(checkForWindowScroll, 30);
        }
      })();

      function adjustWindowScroll() {
        let currentScrollX = window.pageXOffset;
        let currentScrollY = window.pageYOffset;

        let canScrollUp = currentScrollY > 0;
        let canScrollDown = currentScrollY < maxScrollY;
        let canScrollLeft = currentScrollX > 0;
        let canScrollRight = currentScrollX < maxScrollX;

        let nextScrollX = currentScrollX;
        let nextScrollY = currentScrollY;

        let maxStep = 50;

        if (isInLeftEdge && canScrollLeft) {
          let intensity = (edgeLeft - viewportX) / edgeSize;
          nextScrollX = nextScrollX - (maxStep * intensity);
        } else if (isInRightEdge && canScrollRight) {
          let intensity = (viewportX - edgeRight) / edgeSize;
          nextScrollX = nextScrollX + (maxStep * intensity);
        }

        if (isInTopEdge && canScrollUp) {
          let intensity = (edgeTop - viewportY) / edgeSize;
          nextScrollY = nextScrollY - (maxStep * intensity);
        } else if (isInBottomEdge && canScrollDown) {
          let intensity = (viewportY - edgeBottom) / edgeSize;
          nextScrollY = nextScrollY + (maxStep * intensity);
        }

        nextScrollX = Math.max(0, Math.min(maxScrollX, nextScrollX));
        nextScrollY = Math.max(0, Math.min(maxScrollY, nextScrollY));

        if (
          nextScrollX !== currentScrollX ||
          nextScrollY !== currentScrollY
        ) {
          window.scrollTo(nextScrollX, nextScrollY);
          return true;
        } else {
          return false;
        }
      }
    },

    mouseDown(e) {
      if (!this.isInContainer(e)) return;

      this.borderWidth = "";

      this.startX = e.pageX - parentLeft;
      this.startY = e.pageY - parentTop;

      this.mouseIsDown = true;
      this.tookScreenshot = false;
    },

    mouseUp(e) {
      if (!this.isInContainer(e)) return;

      this.borderWidth = 0;

      if (this.isDragging) {
        this.tookScreenshot = true;
      }

      this.isDragging = false;
      this.mouseIsDown = false;

      this.takeScreenshot();
    },

    takeScreenshot() {
      html2canvas(document.body, { useCORS: true }).then(canvas => {
        let croppedCanvas = document.createElement('canvas'),
            croppedCanvasContext = croppedCanvas.getContext('2d');

        croppedCanvas.width = this.boxEndWidth;
        croppedCanvas.height = this.boxEndHeight;

        const x = this.startX < this.endX ? this.startX : this.endX;
        const y = this.startY < this.endY ? this.startY : this.endY;

        croppedCanvasContext.drawImage(canvas,
          (x + parentLeft) * pixelRatio, (y + parentTop) * pixelRatio, 
          this.boxEndWidth * pixelRatio, this.boxEndHeight * pixelRatio,
          0, 0,
          this.boxEndWidth, this.boxEndHeight);

        this.imageUrl = croppedCanvas.toDataURL("image/png");

        croppedCanvas.toBlob(blob => {
          this.$emit("image-cropped", { url: this.imageUrl, blob });
        }, "image/png");
      });
    },

    isInContainer(e) {
      const containerXStart = parentLeft;
      const containerYStart = parentTop;
      const containerXEnd = containerXStart + this.width;
      const containerYEnd = containerYStart + this.height;

      const cursorPosX = e.pageX;
      const cursorPosY = e.pageY;

      return !(cursorPosX < containerXStart ||
            cursorPosX > containerXEnd ||
            cursorPosY < containerYStart ||
            cursorPosY > containerYEnd);
    }
  },
}
</script>

<style scoped>
*,
*:before,
*:after {
    box-sizing: border-box;
}

html, body {
  padding: 0;
  margin: 0;
  height: 100%;
}

.overlay,
.crosshairs,
.tooltip,
.borderedBox {
  user-select: none;
}

.screenshot-container {
  clear: both;
  position: relative;
}

.overlay {
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  position: relative;
}

.overlay.highlighting {
  background: none;
  border-color: rgba(0, 0, 0, 0.5);
  border-style: solid;
}

.crosshairs {
  height: 15px;
  width: 15px;
  position: absolute;
  z-index: 100;
}

.crosshairs.hidden {
  display: none;
}

.crosshairs::before,
.crosshairs::after {
  content: "";
  height: 100%;
  width: 100%;
  position: absolute;
}

.crosshairs::before {
  left: -100%;
  top: -100%;
  border-right: 1px solid rgba(255, 255, 255, 0.7) !important;
  border-bottom: 1px solid rgba(255, 255, 255, 0.7) !important;
}

.crosshairs::after {
  left: -1px;
  top: -1px;
  border-top: 1px solid rgba(255, 255, 255, 0.7) !important;
  border-left: 1px solid rgba(255, 255, 255, 0.7) !important;
}

.borderedBox {
  border: 1px solid #fff;
  position: absolute;
}

.borderedBox.hidden {
  display: none;
}

.tooltip {
  display: inline-block;
  position: absolute;
  
  color: #fff;

  border-radius: 4px;
  
  font-size: 12px;
  font-family: monospace;
  
  padding: 6px;
  margin: 6px;
  white-space: nowrap;
}

.tooltip.hidden {
  display: none;
}

.slot-wrapper {
  position: absolute;
}

/* flash when taking screenshot */
.flash {
  position: absolute;
  width: 100%;
  height: 100%;
  
  top: 0;
  left: 0;
  
  background-color: #fff;
  z-index: 100;
  
  opacity: 1;
  
  animation-delay: 0.2s;
  animation-name: fade-out;
  animation-duration: 1s;
  animation-iteration-count: 1;
  animation-fill-mode: forwards;
}

.screenshot-enter-active, .screenshot-leave-active {
  transition: opacity .2s;
}

.screenshot-enter, .screenshot-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}

@keyframes fade-out {
  from {
    opacity: 1;
  }

  to {
    opacity: 0;
  }
}
</style>