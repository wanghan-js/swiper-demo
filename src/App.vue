<template>
  <div class="app">
    <div
      class="swiper-wrapper"
      ref="wrapper"
    >
      <div
        class="swiper-container"
        ref="container"
        @touchstart="handleStart"
        @touchmove="handleMove"
        @touchend="handleEnd"
      >
        <div class="swiper-slide">1</div>
        <div class="swiper-slide">2</div>
        <div class="swiper-slide">3</div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Vue, Component } from 'vue-property-decorator';

@Component
export default class App extends Vue {
  private wrapper: HTMLDivElement = document.createElement('div');
  private container: HTMLDivElement = document.createElement('div');
  private slideWidth: number = 0;
  private slideCount: number = 0;
  private currentSlideIndex: number = 0;
  private timeStart: number = 0;
  private timeEnd: number = 0;
  private positionXStart: number = 0;
  private positionXMoving: number = 0;
  private positionXEnd: number = 0;
  private slideTurningSpeed: number = 0.5;

  // 注意 index 是负值
  get lastSlideIndex(): number {
    return -(this.slideCount - 1);
  }

  public mounted(): void {
    this.setContainerWidth();
  }

  private setContainerWidth(): void {
    this.wrapper = this.$refs.wrapper as HTMLDivElement;
    this.container = this.$refs.container as HTMLDivElement;
    this.slideWidth = this.wrapper.clientWidth;
    this.slideCount = this.container.childElementCount;
    this.container.style.width = `${this.slideWidth * this.slideCount}px`;
  }

  private handleStart(e: TouchEvent): void {
    this.timeStart = Date.now();
    this.positionXStart = e.changedTouches[0].clientX;
  }

  private handleMove(e: TouchEvent): void {
    this.positionXMoving = e.changedTouches[0].clientX;
    let distanceMoving: number = this.positionXMoving - this.positionXStart;
    if (
      (this.currentSlideIndex === 0 && distanceMoving > 0) ||
      (this.currentSlideIndex === this.lastSlideIndex && distanceMoving < 0)
      ) {
      // 在第一个 slide 和最后一个 slide 实现橡皮筋效果
      distanceMoving = distanceMoving / 3;
    }
    const distanceTotal: number = distanceMoving + this.currentSlideIndex * this.slideWidth;
    this.clearTransition();
    this.moveContainer(distanceTotal);
  }

  private handleEnd(e: TouchEvent): void {
    this.timeEnd = Date.now();
    this.positionXEnd = e.changedTouches[0].clientX;
    const distanceTotal: number = this.positionXEnd - this.positionXStart;
    const timeInterval: number = this.timeEnd - this.timeStart;
    const speed: number = Math.abs(distanceTotal / timeInterval);
    let slideCountMoved: number = Math.round(distanceTotal / this.slideWidth);
    if (slideCountMoved === 0 && speed > this.slideTurningSpeed) {
      if (distanceTotal > 0) {
        slideCountMoved += 1;
      } else if (distanceTotal < 0) {
        slideCountMoved -= 1;
      }
    }
    this.currentSlideIndex += slideCountMoved;
    this.restrictSlideIndex();
    this.addTransition();
    this.moveContainer(this.currentSlideIndex * this.slideWidth);
  }

  private moveContainer(distance: number): void {
    this.container.style.transform = `translate3d(${distance}px, 0, 0.001px)`;
  }

  private addTransition(): void {
    this.container.style.transition = '0.4s';
  }

  private clearTransition(): void {
    this.container.style.transition = 'none';
  }

  private restrictSlideIndex(): void {
    // 注意这个 index 是负值, 所以要这样判断
    if (this.currentSlideIndex > 0) {
      this.currentSlideIndex = 0;
    } else if (this.currentSlideIndex < this.lastSlideIndex) {
      this.currentSlideIndex = this.lastSlideIndex;
    }
  }
}
</script>

<style lang="less">
.app {
  position: relative;
  font-family: 微软雅黑;
  height: 100%;
  .swiper-wrapper {
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    width: 100%;
    overflow: hidden;
    .swiper-container {
      display: flex;
    }
    .swiper-slide {
      flex: 1;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 50vw;
      font-size: 10vw;
      color: #fff;
      &:nth-child(1) {
        background: cornflowerblue;
      }
      &:nth-child(2) {
        background: darkcyan;
      }
      &:nth-child(3) {
        background: greenyellow;
      }
    }
  }
}
</style>
