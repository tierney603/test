<template>
  <div id="app">
    <h1>剩余抽奖次数：{{ num }}</h1>
    <round-turntable
      ref="roundTurntable"
      :prizeData="prizeData"
      :rotateCircle="rotateCircle"
      :duringTime="duringTime"
      :turntableStyleOption="turntableStyleOption"
      @endRotation="endRotation"
      :size="turntableSize"
>
      <template slot="item" slot-scope="scope">
        <div class="turntable-name">{{ scope.item.prizeName }}</div>
        <div class="turntable-img">
          <img :src="scope.item.prizeImg" :style="{width:'30px',height:'30px'}">
        </div>
      </template>
    </round-turntable>

    大小：<input type="text" v-model="turntableSize">
    <button @click="initRoundTurntable">生成</button>

    <button @click="startRotation">抽奖</button>
  </div>
</template>

<script>
  import roundTurntable from '../components/roundTurntable.vue';
  export default {
    name: 'App',
    components: {
      roundTurntable
    },
    data() {
      return {
        // 转盘上的奖品数据
        prizeData: [
          {
            id: 1,
            prizeName: '2000元京东券',
            prizeImg: 'https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg.jj20.com%2Fup%2Fallimg%2F4k%2Fs%2F02%2F2109242332225H9-0-lp.jpg&refer=http%3A%2F%2Fimg.jj20.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1665239287&t=a10a5e7dff3cf09e426eb8b90b1b2694',
          },
          {
            id: 2,
            prizeName: '300元京东券',
            prizeImg: 'https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg.jj20.com%2Fup%2Fallimg%2F4k%2Fs%2F02%2F2109242332225H9-0-lp.jpg&refer=http%3A%2F%2Fimg.jj20.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1665239287&t=a10a5e7dff3cf09e426eb8b90b1b2694',
          },
          {
            id: 3,
            prizeName: '50个比特币',
            prizeImg: 'https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg.jj20.com%2Fup%2Fallimg%2F4k%2Fs%2F02%2F2109242332225H9-0-lp.jpg&refer=http%3A%2F%2Fimg.jj20.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1665239287&t=a10a5e7dff3cf09e426eb8b90b1b2694',
          },
          {
            id: 4,
            prizeName: '50元话费券',
            prizeImg: 'https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg.jj20.com%2Fup%2Fallimg%2F4k%2Fs%2F02%2F2109242332225H9-0-lp.jpg&refer=http%3A%2F%2Fimg.jj20.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1665239287&t=a10a5e7dff3cf09e426eb8b90b1b2694',
          },
          {
            id: 5,
            prizeName: '100元话费券',
            prizeImg: 'https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg.jj20.com%2Fup%2Fallimg%2F4k%2Fs%2F02%2F2109242332225H9-0-lp.jpg&refer=http%3A%2F%2Fimg.jj20.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1665239287&t=a10a5e7dff3cf09e426eb8b90b1b2694',
          },
          {
            id: 6,
            prizeName: '100个比特币',
            prizeImg: 'https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg.jj20.com%2Fup%2Fallimg%2F4k%2Fs%2F02%2F2109242332225H9-0-lp.jpg&refer=http%3A%2F%2Fimg.jj20.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1665239287&t=a10a5e7dff3cf09e426eb8b90b1b2694',
          },
          {
            id: 7,
            prizeName: '10个比特币',
            prizeImg: 'https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg.jj20.com%2Fup%2Fallimg%2F4k%2Fs%2F02%2F2109242332225H9-0-lp.jpg&refer=http%3A%2F%2Fimg.jj20.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1665239287&t=a10a5e7dff3cf09e426eb8b90b1b2694',
          }
        ],
        // 转动的圈数
        rotateCircle: 10,
        // 转动需要持续的时间（s）
        duringTime: 7,
        // 转盘样式的选项
        turntableStyleOption: {
          // 背景色
          prizeBgColors: ['#AE3EFF', '#4D3FFF', '#FC262C', '#3A8BFF', '#EE7602', '#FE339F'],
          // 转盘的外边框颜色
          borderColor: '#199301',
        },
        // 中奖的奖品的index
        prizeIndex: -1,
        // 用来锁定转盘，避免同时多次点击转动
        isLocking: false,
        // 剩余抽奖次数
        num: 100,
        turntableSize:300,

      }
    },
    methods: {
      initRoundTurntable(){

        this.$refs.roundTurntable.init()
      },
      startRotation() {
        // 如果还不可以转动
        if (!this.canBeRotated()) {
          return false;
        }

        // 开始转动
        // 先上锁
        this.isLocking = true;
        // 设置在哪里停下，应该与后台交互，这里随机抽取0~5
        const index = Math.floor(Math.random() * this.prizeData.length);
        // 成功后次数减少一次
        this.num--;
        this.prizeIndex = index;
        // 告诉子组件，开始转动了
        this.$refs.roundTurntable.rotate(index);
      },
      // 已经转动完转盘触发的函数
      endRotation() {
        // 提示中奖
        alert(`恭喜您获奖啦,您的奖品是：${this.prizeData[this.prizeIndex].prizeName}`);
        // 解锁
        this.isLocking = false;
      },
      // 判断是否可以转动
      canBeRotated() {
        if (this.num <= 0) {
          alert('已经木有次数了！');
          return false;
        }
        if (this.isLocking) {
          return false;
        }
        return true;
      },
    }
  }
</script>

<style scoped>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin: 0;
    padding: 0;
    background-image: url();
  }
  
</style>