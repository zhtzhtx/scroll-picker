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
            rotateRate * i +
            'deg) translateZ(' +
            zDistance +
            'px)',
        }" v-for="(item, i) in currentData" :key="i">
          {{ item.label }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: ["data", "limit"],
  data() {
    return {
      // 触发懒加载的最小值
      min: 0,
      // 触发懒加载的最大值
      max: 0,
      // 当前选中的index
      index: 0,
      // 当前展示的数据
      currentData: [],
      // 当前展示的数据数组选中index
      currentIndex: 0,
      // 所有数据的开始index
      startIndex: 0,
      // 所有数据的结束index
      endIndex: 0,
      // 每个div高度
      itemHeight: 35,
      // 每次旋转角度
      rotateRate: 0,
      // 轴距
      zDistance: 0,
      // 当前旋转度数
      currentRotate: 0,
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
      this.data.forEach((item, index) => {
        this.$set(item, "_index", index);
      });
      // 展示数据
      this.currentData =
        this.data.length > this.limit ? this.data.slice(0, this.limit) : this.data;
      // 平均各个div的旋转角度
      this.rotateRate = 180 / this.currentData.length;
      // 计算轴距
      this.zDistance = this.itemHeight / (Math.tan((this.rotateRate / 360) * Math.PI) * 2);
      // 设置触发懒加载的最大值
      this.max = this.limit - 2
      // 设置所有数据的结束index
      this.endIndex = this.data.length - 1
    },
    mousedown(e) {
      // 记录点击的鼠标位置
      this.touchStart = e.clientY;
      // 初始化鼠标移动的距离
      this.moveDistance = 0;
      // 标记鼠标按下
      this.isTouch = true;
    },
    mousemove(e) {
      // 如果鼠标长按
      if (this.isTouch) {
        // 获取鼠标移动距离
        this.moveDistance = this.touchStart - e.clientY;
        // 获取周期
        const cycle = this.moveDistance / this.itemHeight;
        // 获取当前转动的index
        const turningIndex = cycle + this.index
        // 判断是否需要懒加载
        this.lazyLoad(turningIndex)
      }
    },
    mouseup() {
      // 判断当前转动角度
      let r = this.currentRotate % this.rotateRate;
      // 获取当前转到的index
      let currentIndex = parseInt(this.currentIndex);
      // 如果转动超过一半
      if (r > this.rotateRate / 2) {
        this.currentRotate = this.currentRotate + this.rotateRate - r;
        currentIndex++;
      } else {
        this.currentRotate = this.currentRotate - r;
      }
      // 获取当前选中的index
      this.index = currentIndex > this.currentData.length - 1
        ? this.currentData[this.currentData.length - 1]._index
        : this.currentData[currentIndex]._index;
      // 关闭鼠标按下标识
      this.isTouch = false;
      console.log(this.index)
    },
    wheel(e) {
      // 判断滚轮是上滑还是下滑
      const isTurnDown = e.deltaY > 0 ? 1 : -1;
      // 获取当前转动的index
      const turningIndex = this.index + isTurnDown
      // 判断是否需要懒加载
      this.lazyLoad(turningIndex)
      // 获取当前选中的index
      this.index = this.currentData[this.currentIndex]._index;
    },
    lazyLoad(turningIndex) {
      // 转动到0,不再转动
      if (turningIndex < 0) {
        this.currentIndex = 0;
        this.currentRotate = 0;
        return;
      }
      // 转动到最大值,不再转动
      if (turningIndex > this.endIndex) {
        this.currentIndex = this.currentData.length - 1;
        this.currentRotate = this.currentIndex * this.rotateRate;
        return;
      }
      // 判断是否需要懒加载
      // 判断当前展示数据数组是否包含第一位或者最后一位
      let isExceedsMax =
        // 如果当前数组没有转到底部，同时超过触发最大值
        (this.currentData[this.currentData.length - 1]._index !== this.endIndex)
        && (turningIndex > this.max)
      // 如果当前数组没有转到顶部，同时小于触发最小值
      let isExceedsMin = (this.currentData[0]._index !== this.startIndex) && (turningIndex < this.min)

      // 当转动超过限制区间时，加载新的数据数组
      if (isExceedsMax || isExceedsMin) {
        // 定义区间开始index和结束
        let startIndex, endIndex
        // 如果是超过最大值情况
        if (isExceedsMax) {
          // 获取区间开始index
          startIndex = turningIndex - 1 < 0 ? 0 : parseInt(turningIndex - 1);
          // 获取区间结束index
          endIndex =
            startIndex + this.limit > this.data.length
              ? this.data.length
              : startIndex + this.limit;
        } else {
          // 如果是小于最小值情况
          // 获取区间结束index
          endIndex =
            turningIndex + 2 > this.data.length
              ? this.data.length
              : parseInt(turningIndex + 2);
          // 获取区间开始index
          startIndex =
            endIndex - this.limit < 0
              ? 0
              : endIndex - this.limit;
        }
        // 截取当前展示的数组
        this.currentData = this.data.slice(startIndex, endIndex);
        // 获取当前转动对应当前展示数组中的index
        console.log(turningIndex, startIndex)
        this.currentIndex = turningIndex - startIndex;
        console.log(this.currentIndex)
        // 更新触发懒加载的最小值
        this.min = this.currentData[0]._index + 1
        // 更新触发懒加载的最大值
        this.max = this.currentData[this.currentData.length - 1]._index - 1
      } else {
        // 不需要懒加载
        const startIndex = this.currentData[0]._index
        this.currentIndex = turningIndex - startIndex
        console.log(this.currentIndex)
      }
      this.currentRotate = this.currentIndex * this.rotateRate;
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
  border: 1px solid #ccc;
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
  height: 35px;
  line-height: 35px;
  user-select: none;
}

.divide-bar-item.active {
  color: #3a72ed;
}
</style>