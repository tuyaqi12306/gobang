<template>
  <div id="app">
    <div class="main">
      <table>
        <tr v-for="(row, x) in chessBoard" :key="x">
          <td
            v-for="(cell, y) in row"
            :key="y"
            @click="playChess($event, x, y)"
            :class="{
              square: true,
              blackBgc: chessBoard[x][y] === 1,
              blueBgc: chessBoard[x][y] === 2
            }"
          >
            <!-- {{ x }}{{ y }} -->
          </td>
        </tr>
      </table>
      <div class="user-defined">
        <div>
          自定义棋盘大小(px)：
          <el-radio-group v-model.number="size" @change="initBoard">
            <el-radio-button label="10"></el-radio-button>
            <el-radio-button label="15"></el-radio-button>
            <el-radio-button label="20"></el-radio-button>
          </el-radio-group>
        </div>
        <div style="margin-top: 20px">
          自定义获胜条件(数量)：
          <el-radio-group v-model="winCount">
            <el-radio-button label="4"></el-radio-button>
            <el-radio-button label="5"></el-radio-button>
            <el-radio-button label="6"></el-radio-button>
          </el-radio-group>
        </div>
        <span>当前时间：{{ date }}</span>
        <br />
        <el-button type="text" @click="showHistory">
          点击查看历史棋局 {{ this.history.winner }}
        </el-button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      // bug1:计算属性返回的数组不是响应式的，数组改变之后组件不会重新刷新。这样的话直接在data里面创建数据就行。
      size: 10, // 棋盘大小
      chessBoard: null, // 棋盘
      date: new Date(), //开局时间
      timer: "",
      over: false, // 此处是否有棋子
      currentColor: 1, // 黑棋先落子
      winCount: 5, // 赢法数量
      // black: 0, // 黑棋
      // blue: 0, // 蓝旗
      isOver: false, // 比赛是否结束
      history: {
        checkerBoard: [], // 上一场历史布局
        winner: ""
      }, // 历史记录
      wins: [], // AI赢的统计数组
      myWin: []
    };
  },
  mounted() {
    this.initBoard();
    this.timer = setInterval(() => {
      this.date = new Date().toLocaleTimeString();
    });
    this.getStorageValue();
  },
  beforeDestroy() {
    if (this.timer) {
      clearInterval(this.timer);
    }
  },
  methods: {
    initBoard() {
      // bug2: 直接在data函数调用的话，this.size为undefined，因为data函数还没运行完
      const board = new Array(this.size).fill(0).map(() => {
        return new Array(this.size).fill(0);
      });
      this.chessBoard = board;
    },
    initWins() {
      let wins = [];
      //三维数组
      for (let i = 0; i < this.size; i++) {
        wins[i] = [];
        for (let j = 0; j < this.size; j++) {
          wins[i][j] = [];
        }
      }
      return wins;
    },
    // 落子
    playChess(event, x, y) {
      if (this.isOver) {
        event.preventDefault(); // 阻止默认行为
        event.stopPropagation(); // 阻止事件冒泡
        return;
      }
      if (this.chessBoard[x][y] !== 0) {
        return;
      } else if (this.currentColor === 1) {
        // bug3:数组的下标没有变为getter/setter，直接赋值不会触发渲染。对象的所有属性变为了getter/setter。
        // 数组上的一些方法是可以触发更新的。1.可以使用splice方法，vue会劫持这个方法然后更新 2. 调用this.$forceUpdate()函数
        // this.chessBoard[x][y] = 1;

        this.chessBoard[x].splice(y, 1, 1); // 添加黑色棋子
        this.currentColor = 2; // 当前棋子切换为蓝色
      } else if (this.currentColor === 2) {
        this.chessBoard[x].splice(y, 1, 2);
        this.currentColor = 1;
      }
      this.history.checkerBoard = this.chessBoard; // 每一次布局都添加进历史记录
      this.maxContinuousCount(x, y);
      this.saveValueToLocal();
    },
    // 从某个点出发，从上往下，从左往右，从左上方往斜下方。从某个点儿往他的左下方。
    maxContinuousCount(x, y) {
      // 横向统计
      let blackMax = 0;
      let blueMax = 0;
      let blackCount = 0;
      let blueCount = 0;
      for (let i = 0; i < this.size; i++) {
        if (this.chessBoard[x][i] === 1) {
          blackCount++;
          blackMax = blackCount > blackMax ? blackCount : blackMax;
          if (blackMax >= this.winCount) {
            alert("黑方胜！");
            this.isOver = true;
            this.history.winner = "黑方";
            return;
          }
        } else {
          blackCount = 0;
        }
        if (this.chessBoard[x][i] === 2) {
          blueCount++;
          blueMax = blueCount > blueMax ? blueCount : blueMax;
          if (blueMax >= this.winCount) {
            alert("蓝方胜！");
            this.isOver = true;
            this.history.winner = "蓝方";
            return;
          }
        } else {
          blueCount = 0;
        }
      }
      // 纵向统计
      blackMax = 0;
      blueMax = 0;
      blackCount = 0;
      blueCount = 0;
      for (let i = 0; i < this.size; i++) {
        if (this.chessBoard[i][y] === 1) {
          blackCount++;
          blackMax = blackCount > blackMax ? blackCount : blackMax;
          if (blackMax >= this.winCount) {
            alert("黑方胜！");
            this.isOver = true;
            this.history.winner = "黑方";
            return;
          }
        } else {
          blackCount = 0;
        }
        if (this.chessBoard[i][y] === 2) {
          blueCount++;
          blueMax = blueCount > blueMax ? blueCount : blueMax;
          if (blueMax >= this.winCount) {
            alert("蓝方胜！");
            this.isOver = true;
            this.history.winner = "蓝方";
            return;
          }
        } else {
          blueCount = 0;
        }
      }
      // 正斜统计, x++, y++
      blackMax = 0;
      blueMax = 0;
      blackCount = 0;
      blueCount = 0;
      let lineX1 = x;
      let liney1 = y;
      let lineX2 = x;
      let liney2 = y;
      while (lineX1 >= 0 && liney1 >= 0) {
        if (this.chessBoard[lineX1][liney1] === 1) {
          blackCount++;
          blackMax = blackCount > blackMax ? blackCount : blackMax;
          if (blackMax >= this.winCount) {
            alert("黑方胜！");
            this.isOver = true;
            this.history.winner = "黑方";
            return;
          }
        } else {
          blackCount = 0;
        }
        if (this.chessBoard[lineX1][liney1] === 2) {
          blueCount++;
          blueMax = blueCount > blueMax ? blueCount : blueMax;
          if (blueMax >= this.winCount) {
            alert("蓝方胜！");
            this.isOver = true;
            this.history.winner = "蓝方";
            return;
          }
        } else {
          blueCount = 0;
        }
        lineX1--;
        liney1--;
      }
      while (lineX2 <= 9 && liney2 <= 9) {
        if (this.chessBoard[lineX2][liney2] === 1) {
          blackCount++;
          blackMax = blackCount > blackMax ? blackCount : blackMax;
          if (blackMax >= this.winCount) {
            alert("黑方胜！");
            this.isOver = true;
            this.history.winner = "黑方";
            return;
          }
        } else {
          blackCount = 0;
        }
        if (this.chessBoard[lineX2][liney2] === 2) {
          blueCount++;
          blueMax = blueCount > blueMax ? blueCount : blueMax;
          if (blueMax >= this.winCount) {
            alert("蓝方胜！");
            this.isOver = true;
            this.history.winner = "蓝方";
            return;
          }
        } else {
          blueCount = 0;
        }
        lineX2++;
        liney2++;
      }
      // 反斜统计 x++, y--
      blackMax = 0;
      blueMax = 0;
      blackCount = 0;
      blueCount = 0;
      let lineX3 = x;
      let liney3 = y;
      let lineX4 = x;
      let liney4 = y;
      while (lineX3 >= 0 && liney3 <= 9) {
        if (this.chessBoard[lineX3][liney3] === 1) {
          blackCount++;
          blackMax = blackCount > blackMax ? blackCount : blackMax;
          if (blackMax >= this.winCount) {
            alert("黑方胜！");
            this.isOver = true;
            this.history.winner = "黑方";
            return;
          }
        } else {
          blackCount = 0;
        }
        if (this.chessBoard[lineX3][liney3] === 2) {
          blueCount++;
          blueMax = blueCount > blueMax ? blueCount : blueMax;
          if (blueMax >= this.winCount) {
            alert("蓝方胜！");
            this.isOver = true;
            this.history.winner = "蓝方";
            return;
          }
        } else {
          blueCount = 0;
        }
        lineX3--;
        liney3++;
      }
      while (lineX4 <= 9 && liney4 >= 0) {
        if (this.chessBoard[lineX4][liney4] === 1) {
          blackCount++;
          blackMax = blackCount > blackMax ? blackCount : blackMax;
          if (blackMax >= this.winCount) {
            alert("黑方胜！");
            this.isOver = true;
            this.history.winner = "黑方";
            return;
          }
        } else {
          blackCount = 0;
        }
        if (this.chessBoard[lineX4][liney4] === 2) {
          blueCount++;
          blueMax = blueCount > blueMax ? blueCount : blueMax;
          if (blueMax >= this.winCount) {
            alert("蓝方胜！");
            this.isOver = true;
            this.history.winner = "蓝方";
            return;
          }
        } else {
          blueCount = 0;
        }
        lineX4++;
        liney4--;
      }
    },
    getStorageValue() {
      // 读取 localStorage 中 Json 格式数据
      let value = localStorage.getItem("boardHistory");
      if (value) {
        let jsonValue = JSON.parse(value);
        this.history = jsonValue;
      }
    },
    saveValueToLocal() {
      // 存储 localStorage 数据为 Json 格式
      let history = JSON.stringify(this.history);
      localStorage.setItem("boardHistory", history);
    },
    showHistory() {
      debugger;
      this.chessBoard = this.history.checkerBoard;
    }
    // ,
      // AI落子赢的算法
      // omputerWinSum() {
      //  // 横向,需要连续棋子的数量
      //  let count = 0;
      //  for (let i = 0; i < 15; i++) {
      //    for(let j=0; j<11; j++){
      //      for(let k=0; k<5; k++){
      //        //此循环结束，就是横向的第一种赢法
      //        this.wins[i][j+k][count] = true;
      //      }
      //      count++ ;
      //    }
      //  }
      //  //纵线
      //  for(let i=0; i<15; i++){
      //    for(let j=0; j<11; j++){
      //      for(let k=0; k<5; k++){
      //        this.wins[j+k][i][count] = true;
      //      }
      //      count++ ;
      //    }
      //  }
      //  //正斜线
      //  for(let i=0; i<11; i++){
      //    for(let j=0; j<11; j++){
      //      for(let k=0; k<5; k++){
      //        this.wins[i+k][j+k][count] = true;
      //      }
      //      count++ ;
      //    }
      //  }
      //  //反斜线
      //  for(let i=0; i<11; i++){
      //    for(let j=14; j>3; j--){
      //      for(let k=0; k<5; k++){
      //        this.wins[i+k][j-k][count] = true;
      //      }
      //      count++ ;
      //    }
      //  }
      //  // console.log(count) // 572
    // }
  },
  watch: {
    // history: this.saveValueToLocal
    // history: function() {
    //   // 存储 localStorage 数据为 Json 格式
    //   let history = JSON.stringify(this.history);
    //   localStorage.setItem("boardHistory", history);
    // }
  }
};
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
.main {
  width: 700px;
  padding: 10px;
  box-sizing: border-box;
  margin: auto;
  white-space: nowrap; // 让表格不折行
}
table {
  border-collapse: collapse;
}
.square {
  display: inline-block;
  width: 35px;
  height: 35px;
  padding: 0;
  box-sizing: border-box;
  position: relative;
  background:
    linear-gradient(
      to bottom, transparent 48%,
      #4c4c4c 48%,
      #4c4c4c 52%,
      transparent 52%
    ),
    linear-gradient(
      to right, transparent 48%,
      #4c4c4c 48%,
      #4c4c4c 52%,
      transparent 52%
    );
    // 从0-48%是透明的，从48%-52%颜色为黑，从50%-100%也是透明的
}
// .blackBgc {
//   background-color: black;
// }
// .blueBgc {
//   background-color: blue;
// }
.blackBgc::after {
  content: "";
  display: inline-block;
  width: 80%;
  height: 80%;
  border-radius: 80%;
  background: #000;
  position: absolute;
  transform: translate(-50%, -50%);
  top: 50%;
  left: 50%;
}
.blueBgc::after {
  content: "";
  display: inline-block;
  width: 80%;
  height: 80%;
  border-radius: 80%;
  background: rgb(24, 118, 156);
  position: absolute;
  transform: translate(-50%, -50%);
  top: 50%;
  left: 50%;
}
.user-defined {
  margin-top: 20px;
  text-align: left;
}
// 改进：给每个格子一个十字背景,再把表格的边框去掉
</style>
