<template>
  <div id="app">
    <div v-for="(item, row) in grid" :key="row" class="row">
      <div
        v-for="(n, col) in item"
        :key="col"
        class="col"
        :style="computeBg(row, col)"
      >
        <input
          v-show="customStatus"
          v-model="grid[row][col]"
          :style="computeBg(row, col)"
          @input="customInput($event, row, col)"
          @focus="customFocus"
        />
        <span v-show="!customStatus">{{ n }}</span>
      </div>
    </div>
    <p>
      <button @click="start">开始</button>
      <button @click="stop">暂停</button>
      <button @click="step">单步</button>
      <button @click="clear">清空</button>
      <button @click="reset">重置</button>
      <button @click="custom">{{ customStatus ? "取消" : "自定义" }}</button>
    </p>
  </div>
</template>

<script>
const VOID_NUM = undefined;
function isValidNum(val) {
  if (val === "" || val == null) {
    return false;
  }
  if (!isNaN(val)) {
    //对于空数组和只有一个数值成员的数组或全是数字组成的字符串，isNaN返回false，例如：'123'、[]、[2]、['123'],isNaN返回false,
    //所以如果不需要val包含这些特殊情况，则这个判断改写为if(!isNaN(val) && typeof val === 'number' )
    return true;
  } else {
    return false;
  }
}
export default {
  name: "App",
  data() {
    return {
      grid: [],
      customStatus: false,
    };
  },
  beforeMount() {
    this.timer = null;
    this.initNums = {};
    this.wrong = {};
    this.reset();
  },
  methods: {
    setGrid(row, col, val) {
      this.grid[row][col] = val;
      this.$set(this.grid, row, this.grid[row]);
    },
    customFocus(e) {
      e.target.select();
    },
    customInput(e, row, col) {
      let value = e.target.value;
      if (isValidNum(value)) {
        value *= 1;
        if (value > 9) {
          value = 9;
        } else if (value < 1) {
          value = VOID_NUM;
        }
      } else {
        value = VOID_NUM;
      }
      this.grid[row][col] = value;
      this.checkAll();
      e.target.select();
      this.$set(this.grid, row, this.grid[row]);
    },
    customBlur(e, row, col) {
      let value = e.target.value;
      if (isValidNum(value)) {
        value *= 1;
        if (value > 9) {
          this.initNums[`${row}-${col}`] = true;
          value = 9;
        } else if (value < 1) {
          value = VOID_NUM;
        } else {
          this.initNums[`${row}-${col}`] = true;
        }
      } else {
        value = VOID_NUM;
      }
      this.setGrid(row, col, value);
      setTimeout(() => this.checkAll());
    },
    custom() {
      this.customStatus = !this.customStatus;
    },
    computeBg(row, col) {
      const res = {
        fontWeight: this.initNums[`${row}-${col}`] ? "700" : "normal",
        color: this.initNums[`${row}-${col}`] ? "red" : "",
      };
      if (
        ([0, 1, 2, 6, 7, 8].indexOf(row) > -1 && [3, 4, 5].indexOf(col) > -1) ||
        ([3, 4, 5].indexOf(row) > -1 && [0, 1, 2, 6, 7, 8].indexOf(col) > -1)
      ) {
        res.background = "none";
      }
      if (this.wrong[`${row}-${col}`]) {
        res.background = "pink";
      }
      return res;
    },
    computeWord(row, col) {
      return {
        fontWeight: this.initNums[`${row}-${col}`] ? "700" : "normal",
        color: this.initNums[`${row}-${col}`] ? "red" : "",
      };
    },
    initRandomValue() {
      const random8 = () => Math.floor(Math.random() * 8);

      this.initNums = {};
      const nums = (random8() + 1) * (random8() + 1);
      for (let i = 0; i < nums; i++) {
        const randomRow = random8();
        const randomCol = random8();
        const randomVal = random8() + 1;

        if (!this.hasUsed(randomRow, randomCol, randomVal)) {
          this.setGrid(randomRow, randomCol, randomVal);
          this.initNums[`${randomRow}-${randomCol}`] = true;
        }
      }
    },
    intervalResolve() {
      this.timer = setTimeout(() => {
        this.intervalResolve();
        this.step();
      }, 10);
    },
    checkAll() {
      this.wrong = {};
      for (let row = 0; row < 9; row++) {
        for (let col = 0; col < 9; col++) {
          if (this.grid[row][col] !== VOID_NUM) {
            let used = this.hasUsed(row, col, this.grid[row][col]);
            if (used) {
              this.wrong[`${used[0]}-${used[1]}`] = true;
              this.wrong[`${row}-${col}`] = true;
              this.setGrid(row, col, this.grid[row][col]);
            }
          }
        }
      }
      console.log(this.wrong);
      return Object.keys(this.wrong).length > 0 ? false : true;
    },
    usedInBox(r, c, temp) {
      const rowStart = Math.floor(r / 3) * 3;
      const colStart = Math.floor(c / 3) * 3;
      for (let row = rowStart; row < rowStart + 3; row++) {
        for (let col = colStart; col < colStart + 3; col++) {
          if (r !== row && c !== col && this.grid[row][col] === temp) {
            // console.log("usedInBox", { row, col, temp });
            return [row, col];
          }
        }
      }
      return false;
    },
    usedInCol(r, col, temp) {
      for (let row = 0; row < 9; row++) {
        if (r !== row && this.grid[row][col] === temp) {
          // console.log("usedInCol", { row, col, temp });
          return [row, col];
        }
      }
      return false;
    },
    usedInRow(row, c, temp) {
      for (let col = 0; col < 9; col++) {
        if (c !== col && this.grid[row][col] === temp) {
          // console.log("usedInRow", { row, col, temp });
          return [row, col];
        }
      }
      return false;
    },
    hasUsed(row, col, temp) {
      return (
        this.usedInRow(row, col, temp) ||
        this.usedInCol(row, col, temp) ||
        this.usedInBox(row, col, temp)
      );
    },
    recallOneData() {
      for (let row = 8; row > -1; row--) {
        for (let col = 8; col > -1; col--) {
          if (
            this.grid[row][col] !== VOID_NUM &&
            !this.initNums[`${row}-${col}`]
          ) {
            let temp = this.grid[row][col] + 1;
            while (this.hasUsed(row, col, temp) && temp <= 9) {
              temp++;
            }
            if (temp === 10) {
              if (row === 0 && col === 0) {
                alert("无解");
                this.stop();
                return;
              }
              // console.log("1-9都不可行，回溯");
              this.setGrid(row, col, VOID_NUM);
              this.recall = true;
              return;
            }

            this.recall = false;
            this.setGrid(row, col, temp);
            return;
          }
        }
      }
      alert("无解");
      this.stop();
    },
    addOneData() {
      for (let row = 0; row < 9; row++) {
        for (let col = 0; col < 9; col++) {
          if (this.grid[row][col] === VOID_NUM) {
            let temp = 1;
            while (this.hasUsed(row, col, temp) && temp <= 9) {
              temp++;
            }
            if (temp === 10) {
              // console.log("1-9都不可行，回溯");
              this.recall = true;
              return;
            }

            this.recall = false;
            this.setGrid(row, col, temp);
            return;
          }
        }
      }
      this.stop();
      alert(this.checkAll() ? "成功" : "无解");
    },
    start() {
      this.recall = false;
      this.stop();
      this.intervalResolve();
    },
    stop() {
      if (this.timer) {
        clearTimeout(this.timer);
        this.timer = null;
      }
    },
    step() {
      if (this.recall) {
        this.recallOneData();
      } else {
        this.addOneData();
      }
    },
    reset() {
      this.initNums = {};
      this.wrong = {};
      this.clear();
      this.initRandomValue();
    },
    clear() {
      this.wrong = {};
      for (let row = 0; row < 9; row++) {
        if (!this.grid[row]) {
          this.grid[row] = [];
        }
        for (let col = 0; col < 9; col++) {
          if (!this.initNums[`${row}-${col}`]) {
            this.setGrid(row, col, VOID_NUM);
          }
        }
      }
    },
  },
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.row {
  overflow: hidden;
  display: flex;
  justify-content: center;
}
.col,
.col input {
  font-size: 24px;
  height: 40px;
  width: 40px;
  text-align: center;
  line-height: 40px;
  float: left;
  border-top: 1px solid black;
  border-left: 1px solid black;
  background: rgba(0, 0, 0, 0.3);
}
.col:nth-of-type(9) {
  border-right: 1px solid black;
}
.row:nth-of-type(9) .col {
  border-bottom: 1px solid black;
}
.col input {
  width: 100%;
  height: 100%;
  background: none;
  border: none;
  outline: none;
  padding: 0;
}
</style>
