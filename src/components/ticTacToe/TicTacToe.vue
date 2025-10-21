<script setup lang="ts">
  import { computed, ref } from 'vue'

  const dimensions = [3, 3] as const
  const p1 = { name: 'Player 1', token: 'X' }
  const p2 = { name: 'Player 2', token: 'O' }
  const cellTotal = dimensions[0] * dimensions[1]
  const winnerMatrix = [
    [[1], [1], [1]],
    [[2], [2], [2]],
    [[3], [3], [3]],
    [[1], [2], [3]],
    [[3], [2], [1]],
    [[1, 2, 3], [], []],
    [[], [1, 2, 3], []],
    [[], [], [1, 2, 3]],
    // Flattened:
    // [1, 1, 1],
    // [2, 2, 2],
    // [3, 3, 3],
    // [3, 2, 1],
    // [1, 2, 3]
  ]
  const currentPlayer = ref(p1)
  const grid = ref<string[][]>([Array<string>(3), Array<string>(3), Array<string>(3)])
  const message = ref('')
  const hasWinner = ref(false)
  const winningSet = ref<number[][]>()
  const winningMessageClass = computed(() => ({ 'message-win': hasWinner.value }))
  const playedMatrix = computed(() =>
    grid.value.reduce((acc, current) => {
      const filled: number[] = current.reduce((cacc: number[], ccurrent, cindex) => {
        if (ccurrent === currentPlayer.value.token) {
          cacc.push(cindex + 1)
        }
        return cacc
      }, [])
      return [...acc, filled] as number[][]
    }, [] as number[][])
  )

  function cellClicked(rowIndex = 0, cellIndex = 0) {
    if (!grid.value[rowIndex]?.[cellIndex] && !hasWinner.value) {
      grid.value[rowIndex]![cellIndex] = currentPlayer.value.token      
      const cellsPlayed = grid.value.flat()
      const isDraw = cellsPlayed.length === cellTotal

      if (isWinner()) {
        hasWinner.value = true
        message.value = 'WON!!!'
        return
      } else {
        if (isDraw) {
          message.value = "It's a draw!"
          return
        }
      }

      if (!isDraw || !isWinner()) {
        currentPlayer.value = currentPlayer.value.token === 'X' ? p2 : p1
      }
    }
  }

  function isWinner() {
    if (playedMatrix.value.flat().length < 3) return false

    winningSet.value = winnerMatrix.filter((matrix) =>
      matrix.every((row, rowIndex) =>
        row.every((col) => playedMatrix.value?.[rowIndex]?.includes(col))
      )
    )[0]

    return !!winningSet.value
  }

  function isWinningCell(rowIndex: number, columnIndex: number): boolean {
    return winningSet.value?.[rowIndex]?.includes(columnIndex + 1) ?? false
  }

  function resetGrid() {
    grid.value = [Array(3), Array(3), Array(3)]
    currentPlayer.value = p1
    message.value = ''
    hasWinner.value = false
    winningSet.value = undefined
  }
</script>

<template>
  <header>
    <h1>Tic Tac Toe</h1>
  </header>
  <div class="gameboard">
    <div class="play-area">
      <button
        class="btn-reset"
        @click="resetGrid()"
        :disabled="!grid.flat().length"
      >
        Reset
      </button>
      <table cellspacing="0">
        <tr
        v-for="(row, rIndex) in dimensions[0]"
        :key="rIndex"
        >
          <template
          v-for="(_column, cIndex) in dimensions[1]"
          :key="cIndex"
          >
            <td
              :class="[
                'cell',
                {
                  filled: !!grid[rIndex]?.[cIndex],
                  winner: isWinningCell(rIndex, cIndex),
                },
              ]"
              @click="cellClicked(rIndex, cIndex)"
              role="button"
            >
              <span>{{ grid[rIndex]?.[cIndex] ?? '' }}</span>
            </td>
          </template>
        </tr>
      </table>
    </div>
    <div class="info">
      <h2>{{ currentPlayer.name }}</h2>
      <p
        class="message"
        :class="winningMessageClass"
      >
        {{ message }}
      </p>
    </div>
  </div>
</template>

<style scoped>
  h1,
  h2 {
    color: #ccc;
    font-weight: bold;
  }

  header {
    padding-bottom: 0.75rem;
  }

  h2 {
    padding-bottom: 0.5rem;
    transform: translateY(-0.375rem);
  }

  .play-area {
    display: flex;
    flex-direction: column;
    row-gap: 0.625rem;

    .btn-reset {
      align-self: flex-end;
    }
  }

  button {
    background-color: #444;
    color: #ccc;
    border: none;
    padding: 0.375rem 0.625rem;
    border-radius: 0.25rem;
    cursor: pointer;

    &:not(:disabled):hover {
      background-color: #666;
    }

    &:active {
      background-color: #888;
      box-shadow: 0 0 8px 2px rgba(255, 255, 255, 0.3);
      transition: box-shadow 0.25s ease;
    }

    &:disabled {
      opacity: 0.5;
      cursor: not-allowed;
    }
  }

  table {
    td {
      width: 5rem;
      height: 5rem;
      border: 1px solid #ccc;
      padding: 0;
      font-size: 3rem;
      text-align: center;
      vertical-align: middle;
    }
  }

  .gameboard {
    display: flex;
    column-gap: 2rem;
  }

  .cell {
    cursor: pointer;

    & > :first-child {
      opacity: 0;
      transition: 
        opacity 0.35s ease-in-out,
        box-shadow 0.75s ease;
    }

    &.filled {
      color: #ccc;
      cursor: not-allowed;
      box-shadow: 0 0 8px 2px rgba(255, 255, 255, 0.3);

      & > :first-child {
        /* fade in the content */
        opacity: 1;
      }
    }

    &.winner {
      box-shadow: inset 0px 0px 2px 1px lightgreen;
    }
    &.winner > :first-child {      
      display: inline-block;
      flex-basis: 100%;
      width: 100%;
      color: green;
      text-shadow:
        0px 0px 4px rgb(44, 168, 44),
        0 0 10px lightgreen,
        0 0 20px lightgreen,
        0 0 30px lightgreen,
        0 0 40px lightgreen,
        0 0 50px lightgreen;
      scale: 1.125;
      transition:
        transform 0.5s ease,
        scale 0.5s ease,
        box-shadow 0.5s ease,
        color 0.5s ease,
        text-shadow 0.5s ease,
        border-radius 0.5s ease;
      background-color: var(--color-background);
      border-radius: 3%;
    }
  }

  .message {
    color: orangered;
    font-size: 1.5rem;
    font-weight: bold;
    animation: fadeIn 0.375s ease;
    transition:
      opacity 0.6s ease,
      text-shadow 0.35s ease;

    &.message-win {
      text-shadow:
        0.375rem -1.4rem 2px rgba(144, 238, 144, 0.8),
        -0.375rem 1.4rem 2px rgba(144, 238, 144, 0.6),
        0px 0px 4px green;
      -webkit-text-stroke: 1px rgba(144, 238, 144, 0.6);
      color: green;
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
