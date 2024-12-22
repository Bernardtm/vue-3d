<template>
  <q-page>
    <div ref="canvasContainer" class="canvas-container">
      <canvas ref="canvas"></canvas>
      <canvas ref="canvas2"></canvas>
      <div class="bg-black text-green matrix-cli flex flex-center">
        <q-card flat bordered class="cli-container q-pa-md">
          <div v-for="(line, index) in output" :key="index" class="output-line">
            {{ line }}
          </div>
          <div class="input-line">
            <span class="prompt">></span>
            <input
              v-model="currentInput"
              @keyup.enter="processCommand"
              class="cli-input"
              autofocus
            />
          </div>
        </q-card>
      </div>
    </div>
  </q-page>
</template>

<script>
import { openURL } from 'quasar'
export default {
  data () {
    return {
      currentInput: '',
      output: [],
      charArr: 'abcdefghijklmnopqrstuvwxyz'.split(''),
      maxCharCount: 100,
      fallingCharArr: [],
      fontSize: 10,
      maxColumns: 0,
      cw: window.innerWidth,
      ch: window.innerHeight
    }
  },
  mounted () {
    this.initializeCanvas()
    this.startAnimation()
    window.addEventListener('resize', this.onResize)
  },
  beforeUnmount () {
    window.removeEventListener('resize', this.onResize)
  },
  methods: {
    processCommand () {
      const command = this.currentInput.trim().toLowerCase()
      this.output.push(`> ${command}`)

      let response
      switch (command) {
        case 'help':
          response = `whoami - show name
          uptime - show how long is alive
          uname - show OS name
          github - open GitHub link
          clear - clear screen
          `
          break
        case 'whoami':
          response = 'Bernard'
          break
        case 'uptime':
          response = '33 years'
          break
        case 'uname':
          response = 'Linux'
          break
        case 'github':
          response = 'https://github.com/Bernardtm'
          this.goTo(response)
          break
        case 'linkedin':
          response = 'https://www.linkedin.com/in/bernard-mendes-dev/'
          this.goTo(response)
          break
        case 'clear':
          this.clear()
          break
        default:
          response = `Command not found: ${command}`
      }
      this.output.push(response)
      this.currentInput = ''

      this.$nextTick(() => {
        const cliContainer = this.$refs.cliContainer
        if (cliContainer) {
          cliContainer.scrollTop = cliContainer.scrollHeight
        }
      })
    },
    goTo (url) {
      openURL(url)
    },
    clear () {
      this.output = []
    },
    initializeCanvas () {
      const canvas = this.$refs.canvas
      const canvas2 = this.$refs.canvas2
      this.ctx = canvas.getContext('2d')
      this.ctx2 = canvas2.getContext('2d')

      // Set canvas dimensions
      this.cw = window.innerWidth
      this.ch = window.innerHeight
      canvas.width = canvas2.width = this.cw
      canvas.height = canvas2.height = this.ch

      this.maxColumns = this.cw / this.fontSize
      this.fallingCharArr = Array.from({ length: this.maxColumns }, (_, i) =>
        this.createPoint(i * this.fontSize, this.randomFloat(-500, 0))
      )
    },
    randomInt (min, max) {
      return Math.floor(Math.random() * (max - min) + min)
    },
    randomFloat (min, max) {
      return Math.random() * (max - min) + min
    },
    createPoint (x, y) {
      return { x, y, value: '', speed: this.randomFloat(1, 5) }
    },
    drawPoint (point) {
      point.value = this.charArr[this.randomInt(0, this.charArr.length)].toUpperCase()
      point.speed = this.randomFloat(1, 5)

      // Drawing on the first canvas
      this.ctx2.fillStyle = 'rgba(255,255,255,0.8)'
      this.ctx2.font = `${this.fontSize}px sans-serif`
      this.ctx2.fillText(point.value, point.x, point.y)

      // Drawing on the second canvas with green color
      this.ctx.fillStyle = '#0F0'
      this.ctx.font = `${this.fontSize}px sans-serif`
      this.ctx.fillText(point.value, point.x, point.y)

      // Update the y position
      point.y += point.speed
      if (point.y > this.ch) {
        point.y = this.randomFloat(-100, 0)
        point.speed = this.randomFloat(2, 5)
      }
    },
    startAnimation () {
      const update = () => {
        // Semi-transparent background for trail effect
        this.ctx.fillStyle = 'rgba(0, 0, 0, 0.05)'
        this.ctx.fillRect(0, 0, this.cw, this.ch)
        this.ctx2.clearRect(0, 0, this.cw, this.ch)

        // Draw all falling characters
        this.fallingCharArr.forEach((point) => this.drawPoint(point))

        // Loop the animation
        requestAnimationFrame(update)
      }
      update()
    },
    onResize () {
      this.initializeCanvas()
    }
  }
}
</script>

<style scoped>
body {
  margin: 0;
}

.canvas-container {
  position: relative;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}
canvas {
  position: absolute;
  top: 0;
  left: 0;
}
input {
  background: transparent;
  border: none;
  color: #00ff00;
  font-family: monospace;
  outline: none;
  flex: 1;
}
.matrix-cli {
  position: absolute;
  z-index: 9999;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-family: monospace;
}

.cli-container {
  max-width: 600px;
  margin: auto;
  background-color: rgba(0, 0, 0, 0.8);
  color: #00ff00;
  border: 1px solid #00ff00;
}

.output-line {
  white-space: pre-line;
  color: #00ff00;
  margin-bottom: 4px;
}

.input-line {
  display: flex;
  align-items: center;
}

.prompt {
  color: #00ff00;
  margin-right: 5px;
}

.cli-input .q-field__native {
  background: transparent !important;
  color: #00ff00;
  font-family: monospace;
}

.q-field__native, .q-field__prefix, .q-field__suffix, .q-field__input {
  color: #00ff00 !important;
}
</style>
