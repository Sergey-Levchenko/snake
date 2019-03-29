<template>
  <div id="app" @touchstart="touchstartFunc" @touchend="touchendFunc">
    <transition v-on:after-enter="transitionFunc" name="fade" mode="out-in">
      <div
        key="field"
        class="field"
        v-if="statedGame"
        :style="`width:${fieldSize}px; height:${fieldSize}px;`"
      >
        <!-- <div class="nav nav_1"></div>
        <div class="nav nav_2"></div>
        <div class="nav nav_3"></div>
        <div class="nav nav_4"></div>-->
        <div
          v-for="(item,i) in size*size"
          :style="`width:${itemSize}px; height:${itemSize}px;`"
          :key="`field__item_${i}`"
          class="field__item"
        ></div>
      </div>
      <div key="board" class="board" v-if="!statedGame">
        <div class="board__title"></div>
        <div class="board__line board__speed">
          <div class="board__subtitle">
            <i class="fas fa-tachometer-alt"></i>
          </div>
          <button
            class="board__button board__button_speed"
            :class="{active: speed==1}"
            @click="speed=1"
          >1</button>
          <button
            class="board__button board__button_speed"
            :class="{active: speed==2}"
            @click="speed=2"
          >2</button>
          <button
            class="board__button board__button_speed"
            :class="{active: speed==3}"
            @click="speed=3"
          >3</button>
        </div>
        <div class="board__line board__size">
          <div class="board__subtitle">
            <i class="fas fa-expand-arrows-alt"></i>
          </div>
          <button
            class="board__button board__button_speed"
            :class="{active: size==10}"
            @click="size=10"
          >10</button>
          <button
            class="board__button board__button_speed"
            :class="{active: size==15}"
            @click="size=15"
          >15</button>
          <button
            class="board__button board__button_speed"
            :class="{active: size==20}"
            @click="size=20"
          >20</button>
        </div>
        <button
          class="board__button board__button_play"
          @click.prevent="startGame"
          v-if="!statedGame"
        >
          <i class="fas fa-play"></i>
        </button>
      </div>
    </transition>
    <!-- <transition name="fade"></transition>
    <transition name="fade"></transition>
    <transition name="fade"></transition>
    <transition name="fade"></transition>-->
    <div class="score"></div>
    <div :class="[`button-block`,{'right':windowDirection}]">
      <button class="button button_2" @click.prevent="stopGame" v-if="statedGame">
        <i class="fas fa-stop"></i>
      </button>
      <button class="button button_3" @click.prevent="pauseGame" v-if="statedGame &&! paused">
        <i class="fas fa-pause"></i>
      </button>
      <button class="button button_4" @click.prevent="resumeGame" v-if="statedGame && paused">
        <i class="fas fa-play"></i>
      </button>
    </div>
  </div>
</template>

<script>
import { setInterval, setTimeout, clearInterval } from "timers";
export default {
  data() {
    return {
      windowDirection: false,
      statedGame: false,
      snake: [[5, 5], [5, 6], [5, 7]],
      target: [1, 1],
      size: 15,
      speed: 2,
      direction: "up",
      action: false,
      crash: false,
      paused: false,
      clientX: null,
      clientY: null
    };
  },
  computed: {
    itemSize: function() {
      return this.fieldSize / this.size;
    },
    fieldSize: function() {
      let size;
      if (this.windowDirection) size = (window.innerHeight / 100) * 80;
      else size = (window.innerWidth / 100) * 90;
      return size - (size % this.size);
    },
    end: function() {
      if (this.crash && !this.paused) this.endFunc();

      return this.crash && !this.paused;
    }
  },
  methods: {
    touchstartFunc(event) {
      this.clientX = event.changedTouches["0"].clientX;
      this.clientY = event.changedTouches["0"].clientY;
    },
    touchendFunc(event) {
      let x = event.changedTouches["0"].clientX;
      let y = event.changedTouches["0"].clientY;
      if (this.statedGame &&
          this.action == false) {
        if (
          x - this.clientX > 100 &&
          Math.abs(this.clientY - y) < 70 &&
          this.action == false
        ) {
          this.direction = "right";
        } else if (
          x - this.clientX < -100 &&
          Math.abs(this.clientY - y) < 70 &&
          this.action == false
        ) {
          this.direction = "left";
        } else if (
          y - this.clientY < -100 &&
          Math.abs(this.clientX - x) < 70 &&
          this.action == false
        ) {
          this.direction = "up";
        } else if (
          y - this.clientY > 100 &&
          Math.abs(this.clientX - x) < 70 &&
          this.action == false
        ) {
          this.direction = "down";
        }
        this.action = true;
      }
    },
    transitionFunc() {
      if (this.statedGame)
        this.$nextTick(() => {
          this.startGame();
        });
    },
    startGame() {
      this.crash = false;
      this.statedGame = true;
      this.snake = [[5, 5], [5, 6], [5, 7]];
      this.direction = "up";
      this.paused = false;
      this.$nextTick(() => {
        this.markedField();
        this.reDrawSnake();
        this.newTarget();
        this.$nextTick(() => {
          this.playGame();
        });
      });
    },
    resumeGame() {
      this.crash = false;
      this.playGame();
      this.paused = false;
    },
    pauseGame() {
      this.crash = true;
      this.paused = true;
    },
    stopGame() {
      this.statedGame = false;
      this.crash = true;
      this.paused = true;
    },
    playGame() {
      this.crash = false;
      const interval = setInterval(() => {
        this.collisionCheck();
        if (this.crash) clearInterval(interval);
        else {
          this.action = false;
          this.moveSnake();
          this.clearField();
          this.reDrawSnake();
        }
      }, 500 / this.speed);
    },
    endFunc() {
      let items = document.querySelectorAll(`.field__item`);
      let i = 0;
      let endIntrval = setInterval(function() {
        if (i > this.size * this.size) {
          clearInterval(endIntrval);
        }
        items[i++].classList.add("end-game");
      }, 10);
    },
    markedField() {
      let x = 1,
        y = 1;
      let fieldDom = document.getElementsByClassName("field__item");
      for (let i = 0; i < fieldDom.length; i++) {
        fieldDom[i].dataset.posX = x++;
        fieldDom[i].dataset.posY = y;
        if (x > this.size) x = 1;
        if (x % this.size == 1 && y < this.size) y++;
      }
    },
    reDrawSnake() {
      let head = document.querySelector(
        `.field__item[data-pos-x = '${this.snake[0][0]}'][data-pos-y = '${
          this.snake[0][1]
        }']`
      );
      head.classList.add("snake-head");

      for (let i = 1; i < this.snake.length; i++) {
        let body = document.querySelector(
          `.field__item[data-pos-x = '${this.snake[i][0]}'][data-pos-y = '${
            this.snake[i][1]
          }']`
        );
        body.classList.add("snake-body");
      }
    },
    clearField() {
      let items = document.querySelectorAll(`.field__item`);

      items.forEach(element => {
        element.classList.remove("snake-body", "snake-head");
      });
    },
    moveSnake() {
      let tmp = [
        this.snake[this.snake.length - 1][0],
        this.snake[this.snake.length - 1][1]
      ];
      if (this.direction == "up") {
        this.snake.unshift([
          this.snake[0][0],
          this.snake[0][1] - 1 > 0 ? this.snake[0][1] - 1 : this.size
        ]);
      } else if (this.direction == "down") {
        this.snake.unshift([
          this.snake[0][0],
          this.snake[0][1] + 1 <= this.size ? this.snake[0][1] + 1 : 1
        ]);
      } else if (this.direction == "right") {
        this.snake.unshift([
          this.snake[0][0] + 1 <= this.size ? this.snake[0][0] + 1 : 1,
          this.snake[0][1]
        ]);
      } else if (this.direction == "left") {
        this.snake.unshift([
          this.snake[0][0] - 1 > 0 ? this.snake[0][0] - 1 : this.size,
          this.snake[0][1]
        ]);
      }
      if (
        this.snake[0][0] === this.target[0] &&
        this.snake[0][1] === this.target[1]
      ) {
        this.newTarget();
      } else {
        this.snake.pop();
      }
    },
    newTarget() {
      let posX = Math.floor(Math.random() * (this.size - 1) + 1);
      let posY = Math.floor(Math.random() * (this.size - 1) + 1);
      var positiveArr = this.snake.filter(item => {
        return item[0] === posX && item[1] === posY;
      });
      if (positiveArr.length == 0) {
        let oldTarget = document.querySelector(
          `.field__item[data-pos-x = '${this.target[0]}'][data-pos-y = '${
            this.target[1]
          }']`
        );
        oldTarget.classList.remove("target");
        let target = document.querySelector(
          `.field__item[data-pos-x = '${posX}'][data-pos-y = '${posY}']`
        );
        target.classList.add("target");
        this.target = [posX, posY];
      } else {
        this.newTarget();
      }
    },
    collisionCheck() {
      for (let i = 2; i < this.snake.length; i++) {
        if (
          this.snake[i][0] === this.snake[0][0] &&
          this.snake[i][1] === this.snake[0][1]
        )
          this.crash = true;
      }
    }
  },
  watch: {
    action: elem => {
      console.log("test - ", elem);
    }
  },
  beforeCreate() {},
  mounted() {
    this.windowDirection = window.innerWidth > window.innerHeight;
    document.addEventListener("keyup", e => {
      if (e.keyCode == 37 && this.direction != "right" && this.action == false)
        this.direction = "left";
      else if (
        e.keyCode == 38 &&
        this.direction != "down" &&
        this.action == false
      )
        this.direction = "up";
      else if (
        e.keyCode == 39 &&
        this.direction != "left" &&
        this.action == false
      )
        this.direction = "right";
      else if (
        e.keyCode == 40 &&
        this.direction != "up" &&
        this.action == false
      )
        this.direction = "down";

      this.action = true;
    });
  }
};
</script>


<style lang="scss">
body {
  margin: 0;
}
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  position: absolute;
  width: 100%;
  height: 100%;
  background-color: #333333;
  display: flex;
  justify-content: center;
  align-items: center;
}
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter,
.fade-leave-active {
  opacity: 0;
}
.field {
  display: flex;
  flex-wrap: wrap;
  align-items: flex-start;
  justify-content: flex-start;
  box-sizing: content-box;
  border: 1px solid darkorange;
  &__item {
    border: 1px solid rgba(0, 0, 0, 0.5);
    box-sizing: border-box;
    &.target {
      background-color: darkmagenta;
    }
    &.snake-body {
      background-color: aquamarine;
    }
    &.snake-head {
      background-color: tomato;
    }
    &.end-game {
      background-color: crimson;
    }
  }
}
.button-block {
  display: flex;
  position: absolute;

  right: 20px;
  top: 20px;
  &.right {
  }
}
.button {
  width: 50px;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  background: 0;
  border: 3px solid #ffffff;
  color: #ffffff;
  border-radius: 50%;
  cursor: pointer;
}
.board {
  width: 300px;
  height: 400px;
  border-radius: 30px;
  border: 3px solid #ffffff;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  padding: 20px;
  box-sizing: border-box;
  &__title {
    font-size: 3px;
    color: #ffffff;
    width: 80px;
    height: 80px;
    background: url("assets/images/snake.png") no-repeat center center / 100%;
    margin: 20px;
    align-self: center;
  }
  &__line {
    display: flex;
    align-items: center;
    margin-bottom: 15px;
  }
  &__subtitle {
    font-size: 40px;
    color: #ffffff;
    margin-right: 20px;
    text-align: center;
    width: 45px;
  }
  &__button {
    width: 50px;
    height: 30px;
    display: flex;
    justify-content: center;
    align-items: center;
    background: 0;
    border: 3px solid #ffffff;
    color: #ffffff;
    border-radius: 15px;
    cursor: pointer;
    margin: 0 5px;
    transition: all 0.3s ease;
    font-weight: 700;
    &_play {
      width: 140px;
      height: 70px;
      font-size: 30px;
      align-self: center;
      margin-top: 10px;
      border-radius: 40px;
      &:focus {
        background-color: #fff;
        color: #2c3e50;
      }
    }
    &.active {
      background-color: #fff;
      color: #2c3e50;
    }
  }
}
</style>
