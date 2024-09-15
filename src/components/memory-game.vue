<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'
import rawData from '@/data/data.json'

const getRawData = () => JSON.parse(JSON.stringify(rawData))

type Box = {
  id: string
  value: string
  show: boolean
  matched: boolean
}

const boxList = ref<Box[]>(getRawData())
const userSelection = ref<Box[]>([])
const userCanOpenBox = ref(true)

const shuffleBoxes = () => {
  let currentIndex = boxList.value.length

  // While there remain elements to shuffle...
  while (currentIndex != 0) {
    // Pick a remaining element...
    let randomIndex = Math.floor(Math.random() * currentIndex)
    currentIndex--

    // And swap it with the current element.
    ;[boxList.value[currentIndex], boxList.value[randomIndex]] = [
      boxList.value[randomIndex],
      boxList.value[currentIndex]
    ]
  }
}

const restartGame = () => {
  console.log('rowData: ', rawData)
  boxList.value = getRawData()
  shuffleBoxes()
}

const openBox = (boxIndex: number, openedBox: Box) => {
  if (userCanOpenBox.value) {
    boxList.value[boxIndex].show = true

    if (openedBox.matched) return
    if (userSelection.value[0]) {
      if (
        userSelection.value[0].id === openedBox.id &&
        userSelection.value[0].value === openedBox.value
      ) {
        return
      } else {
        userSelection.value[1] = openedBox
      }
    } else {
      userSelection.value[0] = openedBox
    }
  }
}

watch(
  userSelection,
  (currentValue) => {
    if (currentValue.length === 2) {
      const cardOne = currentValue[0]
      const cardTwo = currentValue[1]

      const box1Position = boxList.value.findIndex((box) => box.id === cardOne.id)
      const box2Position = boxList.value.findIndex((box) => box.id === cardTwo.id)

      // Disable ability to open boxes
      userCanOpenBox.value = false

      if (cardOne.value === cardTwo.value) {
        boxList.value[box1Position].matched = true
        boxList.value[box2Position].matched = true
        userCanOpenBox.value = true
      } else {
        // Disable ability to open boxes
        setTimeout(() => {
          boxList.value[box1Position].show = false
          boxList.value[box2Position].show = false
          // Allow user to open an new box
          userCanOpenBox.value = true
        }, 1500)
      }

      userSelection.value.length = 0
    }
  },
  { deep: true }
)
</script>

<template>
  <main>
    <header>
      <button class="restartBtn" @click="restartGame()">Restart Game</button>
    </header>

    <section class="gameBoard">
      <div
        class="box"
        v-for="(box, index) in boxList"
        :key="box.id"
        @click="openBox(index, box)"
        :class="box.show ? 'openedBox' : ''"
      >
        <Transition name="baseFade" mode="out-in">
          <span class="emoji" v-if="box.show">{{ box.value }}</span>
        </Transition>
        <Transition name="baseFade" mode="out-in">
          <span class="tickMark" v-if="box.matched">✅</span>
        </Transition>
      </div>
    </section>
  </main>
</template>

<style scoped lang="scss">
* {
  box-sizing: border-box;
}

main {
  margin: 30px auto;
}

header {
  display: flex;
  justify-content: center;
  .restartBtn {
    padding: 12px 16px;
    outline: none;
    color: white;
    background: orange;
    font-weight: 700;
    font-size: 16px;
    border-radius: 8px;
  }
}
.gameBoard {
  margin-top: 32px;
  display: grid;
  grid-template-columns: repeat(4, 60px);
  grid-template-rows: repeat(4, 60px);
  grid-column-gap: 12px;
  grid-row-gap: 12px;
  justify-content: center;

  .box {
    display: flex;
    flex-direction: column;
    text-align: center;
    background: yellow;
    position: relative;

    &.openedBox {
      background: whitesmoke;
    }
    .emoji {
      font-size: 80px;
    }

    .tickMark {
      position: absolute;
      right: 4px;
      font-size: 16px;
    }
  }

  @media screen and (min-width: 500px) {
    grid-template-columns: repeat(4, 90px);
    grid-template-rows: repeat(4, 90px);

    .box {
      width: 90px;
    }
  }

  @media screen and (min-width: 600px) {
    grid-template-columns: repeat(4, 120px);
    grid-template-rows: repeat(4, 120px);

    .box {
      width: 120px;
    }
  }
}
</style>
