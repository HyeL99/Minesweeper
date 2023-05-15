<template>
  <div id="app" :class="gameLv">
    <header-component @gameStart="gameStart" :isFailed="isFailed" :leftMines="leftMines" :time="time"/>
    <game-board :rows="rows" :cols="cols" :mines="mines" @gameOver="gameOver" :key="gameCount" @handleMines="handleMines" @startTime="startTime" @gameClear="gameClear"/>
  </div>
</template>

<script>

import HeaderComponent from "@/components/HeaderComponent.vue";
import GameBoard from "@/components/GameBoard.vue";

export default {
  name: 'App',
  components: {GameBoard, HeaderComponent},
  data(){
    return{
      rows: 9,
      cols: 9,
      mines: 10,
      isFailed: false,
      gameCount: 0,
      leftMines: 10,
      time: 0,
      timeId: null,
      gameLv: 'lv1'
    }
  },
  methods:{
    gameStart(lv){
      if(this.timeId) clearInterval(this.timeId);
      this.time = 0;
      this.timeId = null;
      this.isFailed = false;
      this.gameCount ++;
      switch (lv){
        case 1:
          this.rows = 9;
          this.cols = 9;
          this.mines = 10;
          this.leftMines = 10;
          this.gameLv = 'lv1';
          break;
        case 2:
          this.rows = 16;
          this.cols = 16;
          this.mines = 40;
          this.leftMines = 40;
          this.gameLv = 'lv2';
          break;
        case 3:
          this.rows = 16;
          this.cols = 30;
          this.mines = 99;
          this.leftMines = 99;
          this.gameLv = 'lv3';
          break;
      }
    },
    gameOver(){
      this.isFailed = true;
      clearInterval(this.timeId);
    },
    handleMines(num){
      this.leftMines += num;
    },
    startTime(){
      if(this.timeId) clearInterval(this.timeId);
      this.timeId = setInterval(() => {
        this.time++;
      }, 1000);
    },
    gameClear(){
      clearInterval(this.timeId);
    }
  }
}
</script>

<style lang="scss">
@font-face{
  font-family:'bitbit';
  src:url('https://cdn.df.nexon.com/img/common/font/DNFBitBit-Regular.woff'),url('https://cdn.df.nexon.com/img/common/font/DNFBitBit-Regular.woff2') ;
}

*{
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'bitbit';
}
html, body{
  width: 100%;
  height: 100%;
}
body{
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #0C134F;
}
#app{
  padding: 1rem;
  border-radius: 1rem;
  box-sizing: content-box;
  background-color: #EAA3FC;
  border: 5px solid #FFFF00;
  &.lv1{
    width: calc(2rem * 9);
  }
  &.lv2{
    width: calc(2rem * 16);
  }
  &.lv3{
    width: calc(2rem * 30);
  }
}
</style>
