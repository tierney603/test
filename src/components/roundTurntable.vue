<template>
  <div class="dial_plate" :style="{width: size+'px',height:size+'px'}">
    <div class="turntable" ref="turntable"  :style="{width: size+'px',height:size+'px'}">
      <div class="pointer">
        <h3>抽奖</h3>
      </div>
      <div class="pointer_needle"></div>

      <div
        class="myTurntable"
        :style="{ transform: rotateAngle, transition: rotateTransition }"
      >
        <canvas id="canvas" ref="canvas">
          当前浏览器版本过低，请使用其他浏览器尝试
        </canvas>

        <div class="prize-container">
          <div
            v-for="(item, index) in prizeData"
            :key="index"
            class="item"
            :style="getRotateAngle(index)"
          >
            <slot name="item" :item="item"></slot>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
  
<script>
export default {
  name: "round-turntable",
  mounted() {
    this.init();
  },
  props: {
    prizeData: {
      required: true,
    },
    rotateCircle: {
      default: 6,
    },
    turntableStyleOption: {
      default: () => {
        return {
          // 背景色
          prizeBgColors: [
            "#AE3EFF",
            "#4D3FFF",
            "#FC262C",
            "#3A8BFF",
            "#EE7602",
            "#FE339F",
          ],
          // 转盘的外边框颜色
          borderColor: "#199301",
        };
      },
    },
    duringTime: {
      default: 4.5,
    },
    size:{
      default:300,
    },

  },
  data() {
    return {
      // 开始转动的角度
      startRotateDegree: 0,
      rotateAngle: 0,
      rotateTransition: "",
    };
  },
  methods: {
    // 根据index计算每一格要旋转的角度的样式
    getRotateAngle(index) {
      const angle =
        (360 / this.prizeData.length) * index + 180 / this.prizeData.length;
      return {
        transform: `rotate(${angle}deg)`,
      };
    },
    // 初始化圆形转盘canvas
    init() {
      // 各种数据
      const data = this.turntableStyleOption;
      const prizeNum = this.prizeData.length;
      const { prizeBgColors, borderColor } = data;
      // 开始绘画
      const canvas = this.$refs.canvas;
      const ctx = canvas.getContext("2d");
      const canvasW = (this.$refs.canvas.width =
        this.$refs.turntable.clientWidth); // 画板的高度
      const canvasH = (this.$refs.canvas.height =
        this.$refs.turntable.clientHeight); // 画板的宽度
      // translate方法重新映射画布上的 (0,0) 位置
      ctx.translate(0, canvasH);
      // rotate方法旋转当前的绘图，因为文字适合当前扇形中心线垂直的！
      ctx.rotate((-90 * Math.PI) / 180);
      // 圆环的外圆的半径
      const outRadius = canvasW / 2;
      // 圆环的内圆的半径
      const innerRadius = 0;
      const baseAngle = (Math.PI * 2) / prizeNum; // 计算每个奖项所占角度数
      ctx.clearRect(0, 0, canvasW, canvasH); //去掉背景默认的黑色
      ctx.strokeStyle = borderColor; // 设置画图线的颜色
      for (let index = 0; index < prizeNum; index++) {
        const angle = index * baseAngle;
        ctx.fillStyle = prizeBgColors[index]; //设置每个扇形区域的颜色
        ctx.beginPath(); //开始绘制
        // 标准圆弧：arc(x,y,radius,startAngle,endAngle,anticlockwise)
        ctx.arc(
          canvasW * 0.5,
          canvasH * 0.5,
          outRadius,
          angle,
          angle + baseAngle,
          false
        );
        ctx.arc(
          canvasW * 0.5,
          canvasH * 0.5,
          innerRadius,
          angle + baseAngle,
          angle,
          true
        );
        ctx.stroke(); //开始链线
        ctx.fill(); //填充颜色
        ctx.save(); //保存当前环境的状态
      }
    },
    // 转动起来
    rotate(index) {
      // 运转时长
      const duringTime = this.duringTime;
      const rotateAngle =
        this.startRotateDegree +
        this.rotateCircle * 360 +
        360 -
        (180 / this.prizeData.length + (360 / this.prizeData.length) * index) -
        (this.startRotateDegree % 360);
      this.startRotateDegree = rotateAngle;
      this.rotateAngle = `rotate(${rotateAngle}deg)`;
      this.rotateTransition = `transform ${duringTime}s cubic-bezier(0.250, 0.460, 0.455, 0.995)`;
      setTimeout(() => {
        this.$emit("endRotation");
      }, duringTime * 1000 + 500);
    },
  },
};
</script>
  
<style scoped >
  /* 表盘 */
.dial_plate{
  background-color: rgb(241, 0, 0);
  position: relative;
  padding: 10px;
  border-radius: 50%;
  box-shadow: 10px 10px 10px rgb(255, 255, 255);
  border: 2px ;
}
.turntable {
  position: relative;
  /* position: absolute; */
  /* left: 10px; */
  /* top: 10px; */
  text-align: center;
  transform: translateZ(0);
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;

}
.canvas {
}
.myTurntable {
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
}
.prize-container {
  position: absolute;
  left: 25%;
  top: 0;
  width: 50%;
  height: 50%;
}
.item {
  /*background: pink;*/
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  transform-origin: center bottom;
}


/* 指针 */
.pointer {
  width: 15%;
  height: 15%;
  background-color: rgb(255, 0, 0);
  border-radius: 50%;
  border: 5px yellow solid;
  position: relative;
  /* top: 172px;
  left: 170px; */
  z-index: 10;
  margin: 0 auto;
  /* display: flex; */
  /* justify-content: center; */
  /* align-content: center; */
}
h3 {
  line-height: 50px;
  color: rgb(255, 255, 255);
}
.pointer_needle {
  /* position: relative; */
  width: 0px;
  height: 0px;
  border-bottom: 10rem solid rgb(255, 169, 169);
  border-left: 3rem solid transparent;
  border-right: 3rem solid transparent;
  position: absolute;
  /* background-color: rgb(255, 251, 251); */
  z-index: 1;
  top: 25%;
  /* left: 170px; */
}


.turntable {
      /* position: absolute;
      left: calc(50% - 200px);
      top: calc(50% - 200px);
      width: 400px;
      height: 400px; */
    }
    .turntable-name {
      /*background: pink;*/
      position: absolute;
      left: 10px;
      top: 20px;
      width: calc(100% - 20px);
      font-size: 26px;
      text-align: center;
      color: #fff;
    }
    .turntable-img {
      position: absolute;
      /*要居中就要50% - 宽度 / 2*/
      left: calc(50% - 80px / 2);
      top: 60px;
      width: 80px;
      height: 80px;
      
    }
    img {
        display: inline-block;
        width: 100%;
        height: 100%;
      }
    
</style>