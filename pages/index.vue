<template lang="pug">
  v-layout.dpt-numbers-wrapper(row wrap align-center fill-height justify-center)
    v-flex.dpt-numbers(xs3)
      v-layout(row nowrap justify-center)
        v-flex(xs2 v-for="column in columns" :key="column"): v-layout(column nowrap)
          v-flex(xs1 v-for="item in items[column]" :key="item.text").dpt-number-wrapper
            v-card.dpt-number(v-bind:class="{ 'dpt-called': item.called }" @click="onClickNumber(item)")
              v-card-text.headline.text-xs-center {{ item.text }}
    v-flex(xs7)
      v-layout(column nowrap)
        v-flex(xs12): .dpt-display.text-xs-center(:class="displayClass") {{ display }}
        v-flex(xs12)
          v-layout(row nowrap)
            v-spacer
            v-flex(xs2): v-btn.dpt-button(color="secondary" @click="showResetDialog" large block) RESET
            v-flex(xs8): v-btn.dpt-button(color="primary" @click="draw(false)" large block) DRAW
            v-flex(xs2): v-btn.dpt-button(color="accent" @click="draw(true)" large block) DRAW QUICK
    v-dialog(v-model="dialog" persistent="" max-width="290")
      v-card
        v-card-title.headline RESET
        v-card-text
          | Are you sure you want to reset?
        v-card-actions
          v-spacer
          v-btn(color="primary" flat="" @click="dismiss") NO
          v-btn(color="primary" flat="" @click="reset") RESET
</template>

<style lang="stylus">
.dpt-display {
  color: #0078b5
  font-family: Coda, sans-serif !important
  font-size: 45em
  opacity: 1
  text-shadow: 0 0 64px #0078b5
  transition: opacity .5s ease-out
  &.dpt-display-fade-out {
    opacity: 0
  }
  &.dpt-display-fade-in {
    opacity: 1
    transition: opacity 1s ease-in
  }
}
.dpt-numbers-wrapper .dpt-numbers {
  .dpt-number-wrapper {
    padding: 2px !important
  }
  .dpt-number {
    background: #1a3c45
    border: 1px solid #4597ad
    color: rgba(255, 255, 255, .3)
    transition: all .5s ease-out
    &.dpt-called {
      background: #136978
      border: 1px solid #8cdaf0
      color: rgba(255, 255, 255, .8)
    }
    .headline {
      font-family: Coda, sans-serif !important
      padding: 15px
    }
  }
}
.dpt-button {
  font-family: Coda, sans-serif !important
}
</style>

<script>
function generate(min, max) {
  let numbers = {}
  for (let i = min; i <= max; ++i) {
    numbers[i] = { text: i < 10 ? '0' + i : i, called: false }
  }
  return numbers
}

function range(min, max) {
  return Array.from(Array(max - min + 1), (v, k) => k + min)
}

function random(array) {
  return array[Math.floor(Math.random() * array.length)]
}

export default {
  data: () => ({
    dialog: false,
    display: 0,
    displayClass: {
      'dpt-display-fade-in': false,
      'dpt-display-fade-out': false,
    },
    columns: ['B', 'I', 'N', 'G', 'O'],
    items: { B: [], I: [], N: [], G: [], O: [] },
    numbersMap: {},
    numbersArray: [],
    drawing: false
  }),
  mounted () {
    this.load()
    window.addEventListener('keyup', (event) => {
      event.code === 'Space' && this.draw()
    })
  },
  methods: {
    draw (quick) {
      if (!this.drawing) {
        let counter = quick ? 15 : 50
        const numbers = this.numbersArray.filter((n) => !n.called)
        const callback = () => {
          const number = random(numbers)
          --counter !== 0 ? this.progress(number, counter, callback) : this.done(number)
        }
        callback()
        this.drawing = true
      }
    },
    progress (number, counter, callback) {
      counter === 10 && this.fadeOut()
      this.display = number.text
      setTimeout(callback, 50)
    },
    done (number) {
      number.called = true
      this.display = number.text
      this.drawing = false
      this.fadeIn()
      this.save()
    },
    fadeIn () {
      this.displayClass['dpt-display-fade-out'] = false
      this.displayClass['dpt-display-fade-in'] = true
    },
    fadeOut () {
      this.displayClass['dpt-display-fade-in'] = false
      this.displayClass['dpt-display-fade-out'] = true
    },
    onClickNumber (item) {
      item.called = !item.called
      this.save()
    },
    save () {
      localStorage.numbersMap = JSON.stringify(this.numbersMap)
    },
    load () {
      this.numbersMap = localStorage.numbersMap ? JSON.parse(localStorage.numbersMap) : generate(1, 75)
      this.numbersArray = range(1, 75).map((i) => this.numbersMap[i])
      this.items = {
        B: range(1, 15).map((i) => this.numbersMap[i]),
        I: range(16, 30).map((i) => this.numbersMap[i]),
        N: range(31, 45).map((i) => this.numbersMap[i]),
        G: range(46, 60).map((i) => this.numbersMap[i]),
        O: range(61, 75).map((i) => this.numbersMap[i]),
      }
    },
    showResetDialog () {
      this.dialog = true
    },
    dismiss () {
      this.dialog = false
    },
    reset () {
      localStorage.removeItem('numbersMap')
      this.load()
      this.dialog = false
    }
  }
}
</script>
