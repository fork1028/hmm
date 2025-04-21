<template>
  <div class="game-container">
    <audio ref="bgm" src="/hmm/audio/bgm.mp3" loop autoplay></audio>
    <audio ref="moveSound" src="/hmm/audio/move.mp3"></audio>

    <div class="game-inner-container">
      <v-btn
        v-if="gameStarted"
        @click="toggleMode"
        color="teal-accent-4"
        class="mt-4 mb-4"
        variant="tonal"
        >{{ winScore == 2048 ? "Easy Mode 🍕" : "Normal Mode 🔥" }}</v-btn
      >

      <div v-if="gameStarted" class="grid">
        <div v-for="(row, rowIndex) in board" :key="rowIndex" class="row">
          <div v-for="(cell, colIndex) in row" :key="colIndex" class="cell">
            <div class="tile-wrapper" v-if="cell">
              <img
                :src="getImage(cell)"
                :key="`${cell}-${rowIndex}-${colIndex}`"
                class="tile-image"
              />
              <div class="tile-score">{{ cell }}</div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div v-if="!gameStarted" style="color: #a8a8a8">Guess what game it is?</div>
    <v-btn
      v-if="!gameStarted"
      @click="startGame"
      color="teal-accent-4"
      class="mt-4"
      variant="tonal"
      >Start Game</v-btn
    >

    <v-dialog v-model="win" max-width="500">
      <v-card class="pa-2">
        <v-card-title>🎉 You win!</v-card-title>
        <v-card-text>
          <div style="font-size: 14px">
            Annnnnnnnnnd.... Happy Birthday!
            <br />
            Hope you like this simple game made by 90% of our helper (well you
            know who) and me, not a great writer but wish you all the best for
            this coming 365 days before your next birthday 😉

            <br />
            <br />
            Also hope you get over with any awful feelings with the powder
            blush, I hope dressing yourself up could cheer you up a bit 💄

            <br />
            <br />
            You are already doing sooo amazing, don't be to hard at work, you
            are the best!

            <br />
            <br />
            Love,
            <br />
            Yumika & Grace
          </div>
        </v-card-text>
        <v-card-actions>
          <v-btn color="teal-accent-4" @click="startGame">Play Again</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="lose" max-width="500">
      <v-card>
        <v-card-title>🤔 Game over!</v-card-title>
        <v-card-actions>
          <v-btn color="teal-accent-4" @click="startGame">Try Again</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script lang="ts" setup>
import { ref, onMounted } from "vue";
import {
  VBtn,
  VDialog,
  VCard,
  VCardTitle,
  VCardActions,
} from "vuetify/components";

const size = 4;
const board = ref<number[][]>([]);
const win = ref(false);
const lose = ref(false);
const gameStarted = ref(false);
const bgm = ref<HTMLAudioElement | null>(null);
const moveSound = ref<HTMLAudioElement | null>(null);
const winScore = ref(2048);

const startGame = () => {
  gameStarted.value = true;
  win.value = false;
  lose.value = false;
  board.value = Array(size)
    .fill(null)
    .map(() => Array(size).fill(0));
  addRandomTile();
  addRandomTile();
  bgm.value?.play();
};

const addRandomTile = () => {
  const emptyCells: { x: number; y: number }[] = [];
  for (let i = 0; i < size; i++) {
    for (let j = 0; j < size; j++) {
      if (board.value[i][j] === 0) emptyCells.push({ x: i, y: j });
    }
  }
  if (emptyCells.length === 0) return;
  const { x, y } = emptyCells[Math.floor(Math.random() * emptyCells.length)];
  board.value[x][y] = Math.random() < 0.9 ? 2 : 4;
};

const getImage = (value: number): string => {
  return `/hmm/images/${value}.png`;
};

const preloadImages = () => {
  const values = [2, 4, 8, 16, 32, 64, 128, 256, 512, 1024, 2048];
  values.forEach((val) => {
    const img = new Image();
    img.src = getImage(val);
  });
};

onMounted(() => {
  preloadImages();
  window.addEventListener("keydown", handleKey);
});

const handleKey = (e: KeyboardEvent) => {
  if (win.value || lose.value) return;
  let moved = false;
  switch (e.key) {
    case "ArrowUp":
      moved = move("up");
      break;
    case "ArrowDown":
      moved = move("down");
      break;
    case "ArrowLeft":
      moved = move("left");
      break;
    case "ArrowRight":
      moved = move("right");
      break;
  }
  if (moved) {
    moveSound.value?.play();
    addRandomTile();
    checkWin();
    checkLose();
  }
};

const move = (
  direction: "up" | "down" | "left" | "right",
  inputBoard: number[][] = board.value
): boolean => {
  const newBoard = inputBoard.map((row) => [...row]);
  let moved = false;

  const merge = (line: number[]): number[] => {
    const newLine = line.filter((val) => val !== 0);
    for (let i = 0; i < newLine.length - 1; i++) {
      if (newLine[i] === newLine[i + 1]) {
        newLine[i] *= 2;
        newLine[i + 1] = 0;
      }
    }
    return newLine
      .filter((val) => val !== 0)
      .concat(Array(size).fill(0))
      .slice(0, size);
  };

  for (let i = 0; i < size; i++) {
    let line: number[];
    switch (direction) {
      case "left":
        line = newBoard[i];
        const mergedLeft = merge(line);
        if (JSON.stringify(mergedLeft) !== JSON.stringify(newBoard[i]))
          moved = true;
        newBoard[i] = mergedLeft;
        break;
      case "right":
        line = [...newBoard[i]].reverse();
        const mergedRight = merge(line).reverse();
        if (JSON.stringify(mergedRight) !== JSON.stringify(newBoard[i]))
          moved = true;
        newBoard[i] = mergedRight;
        break;
      case "up":
        line = newBoard.map((row) => row[i]);
        const mergedUp = merge(line);
        if (!moved)
          moved = mergedUp.some((val, idx) => val !== newBoard[idx][i]);
        for (let j = 0; j < size; j++) newBoard[j][i] = mergedUp[j];
        break;
      case "down":
        line = newBoard.map((row) => row[i]).reverse();
        const mergedDown = merge(line).reverse();
        if (!moved)
          moved = mergedDown.some(
            (val, idx) => val !== newBoard[size - 1 - idx][i]
          );
        for (let j = 0; j < size; j++) newBoard[j][i] = mergedDown[j];
        break;
    }
  }

  const isSimulated = inputBoard !== board.value;
  if (!isSimulated) {
    board.value = newBoard;
  }
  return JSON.stringify(newBoard) !== JSON.stringify(inputBoard);
};

const checkWin = () => {
  for (const row of board.value) {
    for (const cell of row) {
      if (cell === winScore.value) {
        win.value = true;
        return;
      }
    }
  }
};

const toggleMode = () => {
  if (winScore.value === 128) {
    winScore.value = 2048;
  } else {
    winScore.value = 128;
  }
};

const checkLose = () => {
  const directions: ("up" | "down" | "left" | "right")[] = [
    "up",
    "down",
    "left",
    "right",
  ];
  for (const dir of directions) {
    const testBoard = board.value.map((row) => [...row]);
    if (move(dir, testBoard)) return;
  }
  lose.value = true;
};
</script>

<style scoped>
.game-container {
  height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background-color: rgb(232, 254, 247);
}
.game-inner-container {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}
.grid {
  display: grid;
  grid-template-rows: repeat(4, 100px);
  gap: 10px;
}
.row {
  display: grid;
  grid-template-columns: repeat(4, 100px);
  gap: 10px;
}
.cell {
  width: 100px;
  height: 100px;
  background: #ccc;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  border-radius: 5px;
}
.tile-wrapper {
  position: relative;
  width: 100%;
  height: 100%;
}
.tile-image {
  max-width: 100%;
  max-height: 100%;
  display: block;
  border-radius: 5px;
}
.tile-score {
  position: absolute;
  bottom: 4px;
  left: 6px;
  font-size: 20px;
  color: white;
  text-shadow: 0 0 2px rgb(6, 4, 28);
  font-weight: bold;
}
</style>
