<template>
  <div id="app">
    <h1>Color Flood</h1>
    <div class="status-bar">
      <div class="button" @click="restart()">NEW</div>
      <div class="steps">{{ step }} / {{ maxAttempts }}</div>
      <div class="button" @click="toggleSettings()">SETTINGS</div>
    </div>
    <div id="settings" v-if="openSettings">
      <form>
        <div class="categories">
          <div class="category">
            <span>Colors:</span><br>
            <input type="radio" id="3-colors" name="color-num" :value="3" v-model="newTotalColors"><label for="3-colors">3</label><br>
            <input type="radio" id="4-colors" name="color-num" :value="4" v-model="newTotalColors"><label for="4-colors">4</label><br>  
            <input type="radio" id="5-colors" name="color-num" :value="5" v-model="newTotalColors"><label for="5-colors">5</label><br>
          </div>
          <div class="category">
            <span>Grid size:</span><br>
            <input type="radio" id="12-grid" name="grid-size" :value="12" v-model="newGridWidth"><label for="12-grid">12 x 12</label><br>
            <input type="radio" id="16-grid" name="grid-size" :value="16" v-model="newGridWidth"><label for="16-grid">16 x 16</label><br>  
            <input type="radio" id="20-grid" name="grid-size" :value="20" v-model="newGridWidth"><label for="20-grid">20 x 20</label><br>
          </div>
        </div>
        <div class="button" @click="saveSettings()">
          SAVE
        </div>
      </form>
    </div>

    <div class="grid">
      <div 
        v-for="row in grid"
        :key="row.index"
        class="grid-row"
      >
        <div
          v-for="col in row"
          :key="col.index"
          class="grid-square"
          :style="squareStyle(col)"
        ></div>
      </div>
      <div class="message" :style="messageStyle">
        {{ message }}
      </div>
    </div>

    <div class="color-ui">
      <div 
        v-for="color in totalColors"
        :key="color.id"
        :style="{ backgroundColor: hexes[color - 1] }"
        class="clickable-color"
        @click="onColorClick(color - 1)"
      ></div>
    </div>
  </div>
</template>

<script>

export default {
  name: 'App',
  data() {
    return {
      gridWidth: 16,
      newGridWidth: 16,
      totalColors: 4,
      newTotalColors: 4,
      selectedColor: undefined,
      step: 0,
      grid: [],
      candidates: [],
      hexes: [
              '#2b6ae0', // blue
              '#f4fa52', // yellow
              '#32b814', // green
              '#e02b3d', // red
              '#ff4ffc', // pink
             ],
      message: "",
      openSettings: false
    }
  },
  created() {
    this.makeGrid();
    this.initClear();
  },
  methods: {
    makeGrid() {
      for (let i = 0; i < this.gridWidth; i++) {
        this.grid.push([]);

        for (let j = 0; j < this.gridWidth; j++) {
          const random = Math.floor(Math.random() * this.totalColors);
          // cleared means connected to the top left
          this.grid[i].push({ initColor: random, cleared : false });
        }
      }
    },
    initClear() {
      this.selectedColor = this.grid[0][0].initColor;
      this.grid[0][0].cleared = true;
      this.getCandidates();
      this.processCandidates();
    },
    onColorClick(i) {
      if (this.selectedColor === i) return;
      if (this.message !== "YOU WON!") this.step++;

      this.selectedColor = i;
      this.getCandidates();
      this.processCandidates();
      this.checkIfVictory();
    },
    getCandidates() {
      this.candidates = [];

      for (let y = 0; y < this.grid.length; y++) {
        for (let x = 0; x < this.grid[y].length; x++) {
          if (this.grid[y][x].cleared && this.hasAdjacent(y, x)) {
            this.candidates.push([y, x]);
          }
        }
      }
    },
    hasAdjacent(y, x, modify = false) {
      const color = this.selectedColor;

      // check if any of the adjacent squares (up, down, left, right) is the target color and not yet cleared (so new path)
      const directions = this.getDirections(y, x);

      for (let i = 0; i < directions.length; i++) {
        const d = directions[i];
        const target = d.condition ? this.grid[y + d.y][x + d.x] : null;

        if (target && !target.cleared && target.initColor === color) {

          if (modify) {
            this.grid[y + d.y][x + d.x].cleared = true;
            this.candidates.push([y + d.y, x + d.x]);
          }
          else {
            return true;
          }
        }
      }

      return false;
    },
    getDirections(y, x) {
      return [
        {
          name: "up",
          condition: y > 0,
          y: -1,
          x: 0
        },
        {
          name: "down",
          condition: y < this.gridWidth - 1,
          y: 1,
          x: 0
        },
        {
          name: "left",
          condition: x > 0,
          y: 0,
          x: -1
        },
        {
          name: "right",
          condition: x < this.gridWidth - 1,
          y: 0,
          x: 1
        }
      ];
    },
    processCandidates() {
      while (this.candidates.length > 0) {
        let startSquare = this.candidates[0];

        this.hasAdjacent(startSquare[0], startSquare[1], true);
        this.candidates.shift();
      }
    },
    checkIfVictory() {
      let canWin = true;

      for (let y = 0; y < this.grid.length; y++) {
        for (let x = 0; x < this.grid[y].length; x++) {
          if (!this.grid[y][x].cleared) {
            canWin = false;
          }
        }
      }

      if (canWin && this.step <= this.maxAttempts) {
        this.message = "YOU WON!";
      }
      else if (this.step >= this.maxAttempts) {
        this.message = "YOU LOST"
      }
    },
    toggleSettings() {
      this.openSettings = !this.openSettings;
    },
    restart() {
      this.step = 0;
      this.selectedColor = undefined;
      this.grid = [];
      this.message = "";
      this.makeGrid();
      this.initClear();
    },
    saveSettings() {
      this.gridWidth = this.newGridWidth;
      this.totalColors = this.newTotalColors;
      this.openSettings = false;
      this.restart();
    },
    squareStyle(square) {
      // if the square has been cleared (i.e. registered as being connected to the top left corner), it should automatically turn to whatever color the user selects. Otherwise it keeps the color designated at the start
      const color = square.cleared ? this.hexes[this.selectedColor] : this.hexes[square.initColor];

      return {
        backgroundColor: color
      }
    }
  },
  computed: {
    maxAttempts() {
      return Math.ceil(this.gridWidth * this.totalColors * 0.29);
    },
    messageStyle() {
      if (this.message === "") {
        return { display: 'none' }
      } 
      else {
        return { display: 'block' }
      }
    }
  }
}
</script>

<style lang="scss">

html {
  font-size: 16px;

  @media only screen and (max-width: 600px) {
    font-size: 0.8rem;
  }

  @media only screen and (max-width: 400px) {
    font-size: 0.6rem;
  }
}

body {
  background: #201f33;
  color: #fff;
  font-family: 'Fjalla One', sans-serif;
}

#app {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 4rem;

  h1 {
    font-size: 3rem;
  }

  .button {
    background: rgba(255, 255, 255, 1);
    color: #201f33;
    border-radius: 1.5rem;
    width: 6rem;
    height: 2.5rem;
    line-height: 2.5rem;
    font-size: 1.2rem;
    text-align: center;
    cursor: pointer;
    transition: background 0.2s;

    &:hover {
      background: rgba(255, 255, 255, 0.6);
      transition: background 0.2s;
    }
  }

  .status-bar {
    margin-bottom: 2rem;
    width: 30rem;
    display: flex;
    justify-content: space-around;
    align-items: center;

    .steps {
      font-size: 2rem;
      width: 8rem;
      text-align: center;
    }
  }

  form {

    .categories {
      display: flex;
      justify-content: space-around;
      width: 30rem;
      font-size: 1.5rem;
      font-family: 'Roboto', sans-serif;

      label {
        margin: 0.8rem;
      }
    }

    .button {
      margin: 20px auto;
    }
  }

  .grid {
    border: 4px solid #fff;
    width: 30rem;
    height: 30rem;
    display: flex;
    flex-direction: column;
    position: relative;

    .grid-row {
      display: flex;
      flex: 1;

      .grid-square {
        flex: 1;
      }
    }

    .message {
      position: absolute;
      width: 12rem;
      height: 4rem;
      left: 50%;
      top: 50%;
      margin-left: -6rem;
      margin-top: -2rem;
      background: #000;
      font-size: 2rem;
      text-align: center;
      line-height: 4rem;
    }
  }

  .color-ui {
    display: flex;
    justify-content: space-around;
    width: 20rem;
    margin-top: 3rem;

    @media only screen and (max-width: 600px) {
      width: 30rem;
    }

    .clickable-color {
      width: 3.5rem;
      height: 3.5rem;
      border-radius: 50%;
      box-sizing: border-box;
      cursor: pointer;
      border: 2px solid #201f33;

      &:hover {
        border: 2px solid transparent;
      }
    }
  }
}
</style>
