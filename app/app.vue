<template>
  <div class="game">
    <h1>🐍 Color Snake</h1>
    <p class="score">Score: {{ score }}</p>

    <div class="field">
      <!-- Snake -->
      <div
        v-for="(segment, index) in snake"
        :key="index"
        class="snake-segment"
        :style="snakeStyle(index)"
      ></div>

      <!-- Food -->
      <div class="food" :style="foodStyle"></div>
    </div>

    <div v-if="gameOver" class="overlay">
      <h2>💀 Game Over</h2>
      <p>Final Score: {{ score }}</p>
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
      score: 0,
      gameOver: false,
      loop: null,
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
    },

    moveSnake() {
      if (this.gameOver) return;

      const head = { ...this.snake[this.snake.length - 1] };

      if (this.direction === "right") head.x++;
      if (this.direction === "left") head.x--;
      if (this.direction === "up") head.y--;
      if (this.direction === "down") head.y++;

      if (
        head.x < 0 ||
        head.x >= this.fieldSize ||
        head.y < 0 ||
        head.y >= this.fieldSize ||
        this.snake.some((s) => s.x === head.x && s.y === head.y)
      ) {
        this.gameOver = true;
        clearInterval(this.loop);
        return;
      }

      this.snake.push(head);

      if (head.x === this.food.x && head.y === this.food.y) {
        this.score++;
        this.spawnFood();
      } else {
        this.snake.shift();
      }
    },

    restart() {
      this.snake = [{ x: 7, y: 7 }];
      this.direction = "right";
      this.score = 0;
      this.gameOver = false;
      this.spawnFood();

      clearInterval(this.loop);
      this.loop = setInterval(this.moveSnake, 120);
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

    snakeStyle(index) {
      const segment = this.snake[index];
      const hue = (index * 25 + this.score * 10) % 360;

      return {
        top: segment.y * 20 + "px",
        left: segment.x * 20 + "px",
        background: `hsl(${hue}, 100%, 60%)`,
        boxShadow: `0 0 10px hsl(${hue}, 100%, 70%)`,
        borderRadius: "6px",
        transition: "all 0.1s linear",
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
        animation: "pulse 0.4s infinite alternate",
      };
    },
  },

  mounted() {
    window.addEventListener("keydown", this.handleKey);
    this.restart();
  },

  beforeUnmount() {
    window.removeEventListener("keydown", this.handleKey);
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
}

h1 {
  font-size: 2.5rem;
  background: linear-gradient(90deg, #00f5ff, #f00);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.score {
  font-size: 1.2rem;
  opacity: 0.8;
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
}

.snake-segment {
  position: absolute;
  width: 20px;
  height: 20px;
}

.food {
  position: absolute;
  width: 20px;
  height: 20px;
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
}

button:hover {
  transform: scale(1.05);
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
