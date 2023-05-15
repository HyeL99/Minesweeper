<template>
<div id="mainBoard">
  <template v-if="mainBoard">
    <div v-for="y in rows" class="rows">
      <div v-for="x in cols" class="cols">
        <button @click="clickBoard(y-1, x-1, $event)" @contextmenu="clickBoard(y-1, x-1, $event)" :disabled="checkBoard[y - 1][x - 1].isChecked" :key="checkBoard[y-1][x-1].board">{{ checkBoard[y - 1][x - 1].board }}</button>
      </div>
    </div>
  </template>
</div>
</template>

<script>
export default {
  name: "GameBoard",
  props:{
    rows: Number,
    cols: Number,
    mines: Number
  },
  data(){
    return{
      mainBoard: null,
      checkBoard: null,
      isStart: false,
    }
  },
  methods:{
    resetBoard(){
      this.isStart = false;
      let list = new Array(this.rows).fill();
      this.mainBoard = JSON.parse(JSON.stringify(list.map(_ => new Array(this.cols).fill(0))));
      this.checkBoard = JSON.parse(JSON.stringify(list.map(_ => new Array(this.cols).fill({board: "", isChecked: false}))));
      const randX = Math.floor(Math.random() * this.cols);
      const randY = Math.floor(Math.random() * this.rows);
      for(let i = 0; i < this.mines; i ++){
        const randX = Math.floor(Math.random() * this.cols);
        const randY = Math.floor(Math.random() * this.rows);

        if(this.mainBoard[randY][randX] !== '💣'){
          this.mainBoard[randY][randX] = '💣'

          if(randY - 1 >= 0 && randX - 1 >= 0 && Number.isInteger(this.mainBoard[randY - 1][randX - 1])) this.mainBoard[randY - 1][randX - 1] += 1
          if(randY - 1 >= 0 && Number.isInteger(this.mainBoard[randY - 1][randX])) this.mainBoard[randY - 1][randX] += 1
          if(randY - 1 >= 0 && randX + 1 < this.cols && Number.isInteger(this.mainBoard[randY - 1][randX + 1])) this.mainBoard[randY - 1][randX + 1] += 1

          if(randX - 1 >= 0 && Number.isInteger(this.mainBoard[randY][randX - 1])) this.mainBoard[randY][randX - 1] += 1
          if(randX + 1 < this.cols && Number.isInteger(this.mainBoard[randY][randX + 1])) this.mainBoard[randY][randX + 1] += 1

          if(randY + 1 < this.rows && randX - 1 >= 0 && Number.isInteger(this.mainBoard[randY + 1][randX - 1])) this.mainBoard[randY + 1][randX - 1] += 1
          if(randY + 1 < this.rows && Number.isInteger(this.mainBoard[randY + 1][randX])) this.mainBoard[randY + 1][randX] += 1
          if(randY + 1 < this.rows && randX + 1 < this.cols && Number.isInteger(this.mainBoard[randY + 1][randX + 1])) this.mainBoard[randY + 1][randX + 1] += 1
        } else {
          i --;
        }
      }
    },
    clickBoard(y, x, event){
      if(this.isStart === false){
        this.isStart = true;
        this.$emit('startTime')
      }
      event?.preventDefault();
      if(event && event.button === 2){ //좌클릭
        if(this.checkBoard[y][x].board === '🚩') {
          this.checkBoard[y].splice(x, 1, {board: "", isChecked: false})
          this.$emit('handleMines', 1)
        }
        else {
          this.checkBoard[y].splice(x, 1, {board: "🚩", isChecked: false})
          this.$emit('handleMines', -1)
        }
      } else {  //우클릭
        if(this.checkBoard[y][x].board === "🚩"){
          return;
        }
        if(this.mainBoard[y][x] === '💣') {
          document.querySelector(`#mainBoard div:nth-of-type(${y + 1}) div:nth-of-type(${x + 1}) button`).classList.add('fail')
          this.gameOver();
        }
        else {
          this.checkBoard[y].splice(x, 1, {board: this.mainBoard[y][x] === 0 ? "" : this.mainBoard[y][x], isChecked: true});
          if(this.mainBoard[y][x] === 0){
            this.openAroundBoard(y, x);
          }
        }
      }
      let isDone = true;
      this.checkBoard.forEach((row, y) => {
        row.forEach((col, x) => {
          if(this.checkBoard[y][x].board === "" && !this.checkBoard[y][x].isChecked){
            isDone = false;
          }
        })
      })
      if(isDone){
        this.$emit('gameClear')
      }
    },
    openAroundBoard(y, x){
      for(let i of [-1, 0, 1]){
        for(let j of [-1, 0, 1]){
          if(i === 0 && j === 0) continue;
          if(y + i >= 0 && y + i < this.rows && x + j >= 0 && x + j < this.cols && !this.checkBoard[y + i][x + j].isChecked){
            this.checkBoard[y + i].splice(x + j, 1, {board: this.mainBoard[y + i][x + j] === 0 ? "" : this.mainBoard[y + i][x + j], isChecked: true})
            if(this.mainBoard[y + i][x + j] === 0){
              this.openAroundBoard(y + i, x + j);
            }
          }
        }
      }
    },
    gameOver(){
      for(let y = 0; y < this.rows; y++){
        for(let x = 0; x < this.cols; x++){
          // if(this.mainBoard[y][x] === '💣') this.checkBoard[y].splice(x, 1, {board: this.mainBoard[y][x] === 0 ? "" : this.mainBoard[y][x], isChecked: true})
          this.checkBoard[y].splice(x, 1, {board: this.mainBoard[y][x] === 0 ? "" : this.mainBoard[y][x], isChecked: true})
        }
      }
      this.$emit('gameOver')
    }
  },
  created(){
    this.resetBoard()
  }
}
</script>

<style lang="scss">
.rows{
  display: flex;
  .cols{
    width: 2rem;
    height: 2rem;
    button{
      width: 2rem;
      height: 2rem;
      border: 1px solid #49a6a1;
      border-radius: 0.2rem;
      background-color: #9af5f0;
    }
    button.fail:disabled{
      opacity: 1;
    }
    button:disabled{
      opacity: 0.5;
      border: 1px solid gray;

    }
  }

}
</style>
