<template>
  <div class="game" tabindex="0" @keydown="handleKey">
    <h1>🐍 Color Snake</h1>
    <p>Score: {{ score }}</p>
    <p>🏆 Highscore: {{ highScore }}</p>
    <p class="lives">❤️ Lives: {{ lives }}</p>

    <div
      class="field"
      @touchstart="handleTouchStart"
      @touchend="handleTouchEnd"
      @touchmove.prevent
    >
      <!-- Snake -->
      <div
        v-for="(segment, index) in snake"
        :key="index"
        class="snake-segment"
        :style="snakeStyle(index)"
      ></div>

      <!-- Normal Food -->
      <div class="food" :style="foodStyle"></div>

      <!-- Special Food -->
      <div
        v-if="specialFood"
        class="food special"
        :style="{
          top: specialFood.y * 20 + 'px',
          left: specialFood.x * 20 + 'px',
        }"
      ></div>
    </div>

    <!-- Mobile Controls -->
    <div class="controls">
      <button @click="setDirection('up')">⬆️</button>
      <div>
        <button @click="setDirection('left')">⬅️</button>
        <button @click="setDirection('down')">⬇️</button>
        <button @click="setDirection('right')">➡️</button>
      </div>
    </div>

    <!-- Game Over -->
    <div v-if="gameOver" class="overlay">
      <h2>💀 Game Over</h2>
      <p>Final Score: {{ score }}</p>
      <p>🏆 Highscore: {{ highScore }}</p>
      <button @click="restart">🔁 Restart</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      fieldSize: 15,
      snake: [{ x: 7, y: 7 }],
      direction: "right",
      food: { x: 10, y: 10 },
      specialFood: null,
      score: 0,
      highScore: 0,
      combo: 0,
      lives: 3,
      speed: 120,
      minSpeed: 60,
      gameOver: false,
      loop: null,
      touchStartX: 0,
      touchStartY: 0,
    };
  },

  methods: {
    spawnFood() {
      let x, y;
      do {
        x = Math.floor(Math.random() * this.fieldSize);
        y = Math.floor(Math.random() * this.fieldSize);
      } while (this.snake.some((s) => s.x === x && s.y === y));
      this.food = { x, y };

      // 20% Chance Special Food
      if (Math.random() < 0.2) {
        let sx, sy;
        do {
          sx = Math.floor(Math.random() * this.fieldSize);
          sy = Math.floor(Math.random() * this.fieldSize);
        } while (
          this.snake.some((s) => s.x === sx && s.y === sy) ||
          (sx === x && sy === y)
        );
        this.specialFood = { x: sx, y: sy };
      } else this.specialFood = null;
    },

    moveSnake() {
      if (this.gameOver) return;

      const head = { ...this.snake[this.snake.length - 1] };

      if (this.direction === "right") head.x++;
      if (this.direction === "left") head.x--;
      if (this.direction === "up") head.y--;
      if (this.direction === "down") head.y++;

      // Collision
      if (
        head.x < 0 ||
        head.x >= this.fieldSize ||
        head.y < 0 ||
        head.y >= this.fieldSize ||
        this.snake.some((s) => s.x === head.x && s.y === head.y)
      ) {
        this.lives--;
        if (this.lives <= 0) {
          this.gameOver = true;
          clearInterval(this.loop);
          if (this.score > this.highScore) {
            this.highScore = this.score;
            localStorage.setItem("colorSnakeHighscore", this.highScore);
          }
          return;
        }
        // Respawn
        this.snake = [{ x: 7, y: 7 }];
        this.direction = "right";
        this.combo = 0;
        return;
      }

      this.snake.push(head);

      let ateFood = false;
      if (head.x === this.food.x && head.y === this.food.y) {
        ateFood = true;
        this.combo++;
        this.score += 1 + this.combo;
      }

      if (
        this.specialFood &&
        head.x === this.specialFood.x &&
        head.y === this.specialFood.y
      ) {
        this.score += 5;
        this.specialFood = null;
        ateFood = true;
      }

      if (ateFood) {
        if (this.speed > this.minSpeed) {
          this.speed -= 3;
          clearInterval(this.loop);
          this.loop = setInterval(() => this.moveSnake(), this.speed);
        }
        this.spawnFood();
      } else {
        this.combo = 0;
        this.snake.shift();
      }
    },

    restart() {
      this.snake = [{ x: 7, y: 7 }];
      this.direction = "right";
      this.score = 0;
      this.combo = 0;
      this.speed = 120;
      this.lives = 3;
      this.gameOver = false;

      const savedHighscore = localStorage.getItem("colorSnakeHighscore");
      if (savedHighscore) this.highScore = parseInt(savedHighscore);

      this.spawnFood();

      clearInterval(this.loop);
      this.loop = setInterval(() => this.moveSnake(), this.speed);
      this.$el.focus();
    },

    handleKey(e) {
      if (e.key === "ArrowUp" && this.direction !== "down")
        this.direction = "up";
      if (e.key === "ArrowDown" && this.direction !== "up")
        this.direction = "down";
      if (e.key === "ArrowLeft" && this.direction !== "right")
        this.direction = "left";
      if (e.key === "ArrowRight" && this.direction !== "left")
        this.direction = "right";
    },

    handleTouchStart(e) {
      this.touchStartX = e.touches[0].clientX;
      this.touchStartY = e.touches[0].clientY;
    },

    handleTouchEnd(e) {
      const dx = e.changedTouches[0].clientX - this.touchStartX;
      const dy = e.changedTouches[0].clientY - this.touchStartY;
      const threshold = 20;

      if (Math.abs(dx) > Math.abs(dy) && Math.abs(dx) > threshold) {
        if (dx > 0 && this.direction !== "left") this.direction = "right";
        if (dx < 0 && this.direction !== "right") this.direction = "left";
      } else if (Math.abs(dy) > threshold) {
        if (dy > 0 && this.direction !== "up") this.direction = "down";
        if (dy < 0 && this.direction !== "down") this.direction = "up";
      }
    },

    setDirection(dir) {
      if (
        (dir === "up" && this.direction !== "down") ||
        (dir === "down" && this.direction !== "up") ||
        (dir === "left" && this.direction !== "right") ||
        (dir === "right" && this.direction !== "left")
      ) {
        this.direction = dir;
      }
    },

    snakeStyle(index) {
      const segment = this.snake[index];
      const hue = (index * 25 + this.score * 10) % 360;
      return {
        top: segment.y * 20 + "px",
        left: segment.x * 20 + "px",
        background: `hsl(${hue},100%,60%)`,
        boxShadow: `0 0 10px hsl(${hue},100%,70%)`,
        borderRadius: "6px",
        transition: "all 0.1s linear",
        position: "absolute",
        width: "20px",
        height: "20px",
      };
    },
  },

  computed: {
    foodStyle() {
      const hue = (this.score * 80) % 360;
      return {
        top: this.food.y * 20 + "px",
        left: this.food.x * 20 + "px",
        background: `radial-gradient(circle, hsl(${hue},100%,70%), hsl(${hue},100%,40%))`,
        boxShadow: `0 0 15px hsl(${hue},100%,60%)`,
        borderRadius: "50%",
        position: "absolute",
        width: "20px",
        height: "20px",
        animation: "pulse 0.4s infinite alternate",
      };
    },
  },

  mounted() {
    this.restart();
  },

  beforeUnmount() {
    clearInterval(this.loop);
  },
};
</script>

<style>
body {
  margin: 0;
  font-family: Arial, sans-serif;
  background: radial-gradient(circle at top, #1a1a2e, #090916);
  color: white;
}

.game {
  text-align: center;
  margin-top: 30px;
  outline: none;
}

h1 {
  font-size: 2.5rem;
  background: linear-gradient(90deg, #00f5ff, #f00);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.field {
  position: relative;
  width: 300px;
  height: 300px;
  margin: 20px auto;
  border-radius: 12px;
  background:
    linear-gradient(#111 1px, transparent 1px),
    linear-gradient(90deg, #111 1px, transparent 1px), #000;
  background-size: 20px 20px;
  box-shadow: 0 0 25px #00f5ff44;
  overflow: hidden;
  touch-action: none;
}

.food.special {
  background: radial-gradient(circle, gold, orange);
  box-shadow: 0 0 20px gold;
  animation: pulse 0.2s infinite alternate;
}

.controls {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 15px;
}

.controls > div {
  display: flex;
  justify-content: center;
}

.overlay {
  margin-top: 20px;
  animation: fadeIn 0.5s ease;
}

button {
  padding: 10px 20px;
  border: none;
  border-radius: 8px;
  background: linear-gradient(90deg, #00f5ff, #ff00c8);
  color: white;
  font-weight: bold;
  cursor: pointer;
  box-shadow: 0 0 10px #ff00c8aa;
  margin: 5px;
}

@keyframes pulse {
  from {
    transform: scale(1);
  }
  to {
    transform: scale(1.3);
  }
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
</style>
