<template>
  <div class="game-block">
    <div class="panel" :class="panelSize">
      <div class="panel-block">{{ numberOfunflagedMines }}</div>
      <button class="restart-btn" @click="restart">{{ buttonText }}</button>
      <div class="panel-block">{{ time }}</div>
    </div>
    <div class="field" :class="fieldSize" oncontextmenu="return false">
      <div class="row" v-for="(row, index) in field" :key="index">
        <div class="cell"
             v-for="(cell, index) in row"
             :key="index"
             :class="{
             'closed': cell.opened === false && cell.flaged === false,
             'flaged': cell.flaged === true,
             'opened': cell.opened === true,
             'empty': cell.opened === true && cell.number === 0,
             'class1': cell.opened === true && cell.number === 1,
             'class2': cell.opened === true && cell.number === 2,
             'class3': cell.opened === true && cell.number === 3,
             'class4': cell.opened === true && cell.number === 4,
             'class5': cell.opened === true && cell.number === 5,
             'class6': cell.opened === true && cell.number === 6,
             'class7': cell.opened === true && cell.number === 7,
             'class8': cell.opened === true && cell.number === 8,
             'misflaged': cell.misflaged,
             'showed-mine': cell.mined && !cell.flaged && lost && !cell.exploded,
             'exploded': cell.exploded
           }"
             @mousedown="mouseDownHandler(cell, $event)"
             @mouseup="mouseUpHandler(cell, $event)"
        >{{ cell.number }}</div>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'Play',
    props: {
      difficulty: String
    },
    data: () => ({
      buttonText: "*_*",
      playing: true,
      lost: false,
      fieldSize: 'small-field',
      panelSize: 'small-panel',
      width: 9,
      height: 9,
      numberOfMines: 10,
      numberOfunflagedMines: 10,
      timer: null,
      time: 0,
      isFirstClick: true,
      field: []
    }),
    methods: {
      setParameters() {
        if (this.difficulty === 'beginner') {
          this.fieldSize = 'beginner-field'
          this.panelSize = 'beginner-panel'
          this.width = 9
          this.height = 9
          this.numberOfunflagedMines = this.numberOfMines = 10
        } else if (this.difficulty === 'intermediate') {
          this.fieldSize = 'intermediate-field'
          this.panelSize = 'intermediate-panel'
          this.width = 16
          this.height = 16
          this.numberOfunflagedMines = this.numberOfMines = 40
        } else if (this.difficulty === 'expert') {
          this.fieldSize = 'expert-field'
          this.panelSize = 'expert-panel'
          this.width = 30
          this.height = 16
          this.numberOfunflagedMines = this.numberOfMines = 99
        }
      },
      generateField() {
        this.field = []
        this.createField()
        this.setMines()
        this.setNumbers()
      },
      createField() {
        let idCounter = 0

        for (let y = 0; y < this.height; y++) {
          const row = []

          for (let x = 0; x < this.width; x++) {
            row.push({
              id: idCounter++,
              mined: false,
              number: 0,
              opened: false,
              flaged: false,
              marked: false,
              misflaged: false,
              exploded: false,
              x: x,
              y: y,
              leftPress: false,
              rightPress: false
            })
          }

          this.field.push(row)
        }
      },
      setMines() {
        for (let i = 0; i < this.numberOfMines; i++) {
          const cell = this.getRandomFreeCell()
          cell.mined = true
        }
      },
      getRandomFreeCell() {
        while (true) {
          const random = Math.floor(Math.random() * this.width * this.height - 1)
          for (let y = 0; y < this.height; y++) {
            for (let x = 0; x < this.width; x++) {
              const cell = this.field[y][x]
              if (cell.id === random && cell.mined === false) {
                return cell
              }
            }
          }
        }
      },
      setNumbers() {
        for (let y = 0; y < this.height; y++) {
          for (let x = 0; x < this.width; x++) {
            const cell = this.field[y][x]
            if (cell.mined === true) {
              const cells = this.getCellsAround(cell.x, cell.y)
              cells.forEach(cell => {
                cell.number++
              })
            }
          }
        }
      },
      getCellsAround(x, y) {
        const cells = []

        if (y !== 0) {
          cells.push(this.field[y - 1][x])

          if (x !== 0) {
            cells.push(this.field[y - 1][x - 1])
          }

          if (x !== this.width - 1) {
            cells.push(this.field[y - 1][x + 1])
          }
        }

        if (y !== this.height - 1) {
          cells.push(this.field[y + 1][x])

          if (x !== 0) {
            cells.push(this.field[y + 1][x - 1])
          }

          if (x !== this.width - 1) {
            cells.push(this.field[y + 1][x + 1])
          }
        }

        if (x !== 0) {
          cells.push(this.field[y][x - 1])
        }

        if (x !== this.width - 1) {
          cells.push(this.field[y][x + 1])
        }

        return cells
      },

      mouseDownHandler(cell, e) {
        if (e.button === 0) {
          cell.leftPress = true
        }

        if (e.button === 2) {
          cell.rightPress = true
        }
      },
      mouseUpHandler(cell, e) {
        if (!this.playing) {
          return
        }

        if ((e.button === 0 && cell.rightPress) || (e.button === 2 && cell.leftPress)) {
          this.bothClick(cell)
          cell.leftPress = false
          cell.rightPress = false
          return
        }

        if (e.button === 0 && cell.leftPress && !cell.rightPress) {
          this.openCell(cell)
          cell.leftPress = false
        }

        if (e.button === 2 && cell.rightPress && !cell.leftPress) {
          this.toggleFlag(cell)
          cell.rightPress = false
        }
      },
      bothClick(cell) {
        console.log('both click')
        const cells = this.getCellsAround(cell.x, cell.y)

        let mineCounter = 0
        let flagCounter = 0

        cells.forEach(cell => {
          if (cell.mined) {
            mineCounter++
          }

          if (cell.flaged) {
            flagCounter++
          }
        })

        if (mineCounter === flagCounter) {
          for (const item of cells) {
            this.openCell(item)
            if (!this.playing) {
              break
            }
          }
        }
      },
      toggleFlag(cell) {
        if (cell.opened === false) {
          cell.flaged = !cell.flaged
        }

        if (cell.flaged) {
          this.numberOfunflagedMines--
        } else {
          this.numberOfunflagedMines++
        }
      },
      openCell(cell) {
        if (cell.flaged === true) {
          return
        }

        if (cell.mined) {
          if (this.isFirstClick) {
            this.generateField()
            this.openCell(this.field[cell.y][cell.x])
            return
          }
          this.endGame(cell)
          return
        }

        cell.opened = true

        if (this.isFirstClick) {
          this.timer = setInterval(() => {
            this.time++
          }, 1000)
          this.isFirstClick = false
        }

        this.checkVictory()
        if (!this.playing) {
          return
        }

        if (cell.number === 0) {
          cell.marked = true
          this.openAround(cell)
        }
      },
      openAround(cell) {
        let done = false

        while (done === false) {
          done = true

          for (let y = 0; y < this.height; y++) {
            for (let x = 0; x < this.width; x++) {
              const cell = this.field[y][x]

              if (cell.marked && cell.number === 0) {
                const cells = this.getCellsAround(cell.x, cell.y)
                cells.forEach(cell => {
                  if (!cell.marked) {
                    cell.marked = true
                    done = false
                  }
                })
              }
            }
          }
        }

        for (let y = 0; y < this.height; y++) {
          for (let x = 0; x < this.width; x++) {
            const cell = this.field[y][x]

            if (cell.marked) {
              cell.opened = true
              cell.marked = false
              this.checkVictory()
              if (!this.playing) {
                return
              }
            }
          }
        }
      },

      checkVictory() {
        let closedCellsCounter = 0

        for (let y = 0; y < this.height; y++) {
          for (let x = 0; x < this.width; x++) {
            const cell = this.field[y][x]

            if (!cell.opened) {
              closedCellsCounter++
            }
          }
        }

        if (closedCellsCounter === this.numberOfMines) {
          this.victory()
        }
      },
      victory() {
        this.buttonText = '^_^'
        this.playing = false
        clearInterval(this.timer)
      },
      endGame(cell) {
        this.buttonText = 'X_X'
        this.playing = false
        clearInterval(this.timer)
        cell.exploded = true
        this.lost = true

        for (let y = 0; y < this.height; y++) {
          for (let x = 0; x < this.width; x++) {
            const cell = this.field[y][x]
            if (cell.flaged && !cell.mined) {
              cell.misflaged = true
            }
          }
        }
      },
      restart() {
        clearInterval(this.timer)
        this.generateField()
        this.playing = true
        this.numberOfunflagedMines = this.numberOfMines
        this.time = 0
        this.buttonText = '*_*'
        this.isFirstClick = true
        this.lost = false
      }
    },
    watch: {
      difficulty() {
        this.setParameters()
        this.generateField()
        this.restart()
      }
    },
    created() {
      this.setParameters()
      this.generateField()
    }
  }
</script>

<style scoped>
.game-block {
  display: flex;
  flex-direction: column;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
}

.panel {
  height: 30px;
  background-color: #344469;
  display: flex;
  justify-content: space-between;
}

.beginner-panel {
  width: 206px;
}

.intermediate-panel {
  width: 360px;
}

.expert-panel {
  width: 668px;
}

.panel-block {
  font-family: "Arial Black", sans-serif;
  margin: 5px;
  height: 25px;
  width: 40px;
  background-color: antiquewhite;
  text-align: right;
  padding-right: 5px;
  color: #bc3737;
}

.restart-btn {
  margin-top: 5px;
  height: 25px;
}

.restart-btn:hover {
  cursor: pointer;
}

.field {
  background-color: #344469;
}

.beginner-field {
  height: 206px;
  width: 206px;
}

.intermediate-field {
  height: 360px;
  width: 360px;
}

.expert-field {
  height: 360px;
  width: 668px;
}

.row {
  display: flex;
  margin-bottom: 2px;
}

.row:first-child {
  margin-top: 5px;
}

.cell {
  height: 20px;
  width: 20px;
  background-color: #faebd7;
  margin-right: 2px;
  font-size: 0;
  font-family: "Arial Black", sans-serif;
  text-align: center;
  cursor: default;
}

.cell:first-child {
  margin-left: 5px;
}

.cell:last-child {
  margin: 0;
}

.closed {
  background-image: url('../assets/square.png');
}

.flaged {
  background-image: url('../assets/flag.png');
}

.misflaged {
  background-image: url('../assets/misflag.png');
}

.exploded {
  background-image: url('../assets/opened-mine.png');
}

.showed-mine {
  background-image: url('../assets/mine.png');
}

.opened {
  font-size: 12px;
}

.empty {
  font-size: 0;
}

.class1 {
  color: #1a58a9;
}

.class2 {
  color: #248611;
}

.class3 {
  color: #bc3737;
}

.class4 {
  color: #001b8c;
}

.class5 {
  color: #863404;
}

.class6 {
  color: #5e03ac;
}

.class7 {
  color: #282525;
}

.class8 {
  color: #5d5555;
}
</style>