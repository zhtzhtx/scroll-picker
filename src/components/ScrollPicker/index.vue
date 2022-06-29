<template>
  <div class="divide-bar" @mousedown="mousedown" @mousemove="mousemove" @mouseup="mouseup">
    <div class="divide-bar-stage" @wheel="wheel">
      <div class="divide-bar-container" :style="{
        height: itemHeight + 'px',
        transform: 'rotateX(' + currentRotate + 'deg)'
      }">
        <div class="divide-bar-item" :style="{
          height: itemHeight + 'px',
          transform:
            'rotateX(-' +
            rotateRate * index +
            'deg) translateZ(' +
            zDistance +
            'px)',
        }" v-for="(item, index) in data" :key="index">
          {{ item.label }}
        </div>
      </div>
    </div>
  </div>
</template>


<script>
export default {
  props: ["data"],
  data() {
    return {
      // 数据总个数
      total: 0,
      // 每个div高度
      itemHeight: 35,
      // 每次旋转角度
      rotateRate: 0,
      // 轴距
      zDistance: 0,
      // 当前旋转度数
      currentRotate: 0,
      // 最后一次记录旋转度数
      lastRotate: 0,
      // 移动距离
      eachOffset: 0,
      // 记录点击的鼠标位置
      touchStart: 0,
      // 记录鼠标移动的距离
      moveDistance: 0,
      // 鼠标长按标识
      isTouch: false,
    };
  },
  mounted() {
    // 初始化数据
    this.init();
  },
  methods: {
    init() {
      // 记录数据总个数
      this.total = this.data.length;
      // 平均各个div的旋转角度
      this.rotateRate = 180 / this.total;
      // 计算轴距
      this.zDistance =
        this.itemHeight / (Math.tan((this.rotateRate / 360) * Math.PI) * 2);
    },
    mousedown(e) {
      // 记录点击的鼠标位置
      this.touchStart = e.clientY;
      // 初始化鼠标移动的距离
      this.moveDistance = 0;
      // 记录当前转动角度
      this.lastRotate = this.currentRotate;
      // 标记鼠标按下
      this.isTouch = true;
    },
    mousemove(e) {
      // 如果鼠标长按
      if (this.isTouch) {
        // 获取鼠标移动距离
        this.moveDistance = this.touchStart - e.clientY;
        // 获取应当旋转的角度
        this.eachOffset = (this.moveDistance / this.itemHeight) * this.rotateRate;
        // 获取总旋转角度
        const totalOffset = (this.total - 1) * this.rotateRate;
        // 如果超出总旋转角度或小于0，则不旋转
        if (this.lastRotate + this.eachOffset > totalOffset) {
          this.currentRotate = totalOffset;
        } else if (this.lastRotate + this.eachOffset < 0) {
          this.currentRotate = 0;
        } else {
          this.currentRotate = this.lastRotate + this.eachOffset;
        }
      }
    },
    mouseup() {
      // 判断当前转动角度
      let r = this.currentRotate % this.rotateRate;
      // 获取当前转到的index
      let index = parseInt(this.currentRotate / this.rotateRate);
      // 如果转动超过一半，则前进一格
      if (r > this.rotateRate / 2) {
        this.currentRotate = this.currentRotate + this.rotateRate - r;
        index++;
      } else {
        this.currentRotate = this.currentRotate - r;
      }
      // 关闭鼠标按下标识
      this.isTouch = false;
      console.log(index);
    },
    wheel(e) {
      // 判断鼠标滚轮是向上滑还是向下滑
      const isTurnDown = e.deltaY > 0
      // 获取总旋转角度
      const totalOffset = (this.total - 1) * this.rotateRate;
      // 如果向下滑
      if (isTurnDown) {
        const turnOffset = this.currentRotate + this.rotateRate
        // 判断是否超过总旋转角度，如果超过则固定为总旋转角度
        this.currentRotate = turnOffset > totalOffset ? totalOffset : turnOffset;
      } else {
        // 如果向上滑
        const turnOffset = this.currentRotate - this.rotateRate
        // 判断旋转角度是否小于0，如果小于0则固定为0
        this.currentRotate = turnOffset < 0 ? 0 : turnOffset;
      }
    }
  },
};
</script>
<style lang='scss' scoped>
.divide-bar-stage {
  position: relative;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
}

.divide-bar-container {
  position: relative;
  width: 100%;
  transform-style: preserve-3d;
}

.divide-bar-container.easein {
  transition: transform 0.5s ease;
}

.divide-bar-item {
  position: absolute;
  width: 100%;
  border-bottom: 1px solid #d3d3d3;
  text-align: center;
  backface-visibility: hidden;
  color: #a4a4a4;
  line-height: 35px;
  user-select: none;
}

.divide-bar-item.active {
  color: #3a72ed;
}
</style>