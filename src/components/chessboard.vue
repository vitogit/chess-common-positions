<template>
  <div class="blue merida">
    <div ref="board" class="cg-board-wrap"></div> </br>
    {{this.board}}
  </div>
</template>

<script>
import Chess from 'chess.js'
import {Chessground} from 'chessground'

export default {
  name: 'chessboard',
  props: {
    shapes: {
      type: Array,
      default: () => [],
    },
    fen: {
      type: String,
      default: '',
    },
    onPromotion: {
      type: Function,
      default: () => 'q',
    },
  },
  data () {
    return {
      takingback: false
    }
  },
  watch: {
    fen: function (newFen) {
      this.fen = newFen
      this.loadPosition()
    },
    shapes: function (newShapes) {
      this.board.setShapes(newShapes)
    }
  },
  methods: {
    takeBack() {
      this.takingback = true
      this.game.undo()
      this.game.undo()

      this.scores.pop()
      this.board.set({
        fen: this.game.fen()
      })
      this.sendInfo()
    },

    possibleMoves () {
      const dests = {}
      this.game.SQUARES.forEach(s => {
        const ms = this.game.moves({square: s, verbose: true})
        if (ms.length) dests[s] = ms.map(m => m.to)
      })
      return dests
    },
    opponentMoves () {
      let originalPGN = this.game.pgn()
      let tokens = this.game.fen().split(' ')
      tokens[1] = tokens[1] === 'w' ? 'b' : 'w'
      tokens = tokens.join(' ')
      let valid = this.game.load(tokens)
      if (valid) {
        let moves = this.game.moves({verbose: true})
        this.game.load_pgn(originalPGN)
        return moves
      } else {
        return []
      }
    },
    toColor () {
      return (this.game.turn() === 'w') ? 'white' : 'black'
    },
    calculatePromotions () {
      let moves = this.game.moves({verbose: true})
      this.promotions = []
      for (let move of moves) {
        if (move.promotion) {
          this.promotions.push(move)
        }
      }
    },
    isPromotion   (orig, dest) {
      let filteredPromotions = this.promotions.filter(move => move.from === orig && move.to === dest)
      return filteredPromotions.length > 0 // The current movement is a promotion
    },
    userPlay() {
      return (orig, dest) => {
        if (this.isPromotion(orig, dest)) {
          this.promoteTo = this.onPromotion()
        }
        this.game.move({from: orig, to: dest, promotion: this.promoteTo}) // promote to queen for simplicity
        this.board.set({
          fen: this.game.fen(),
          turnColor: this.toColor(),
          movable: {
            color: this.toColor(),
            dests: this.possibleMoves(),
          }
        })
        this.calculatePromotions()
        this.afterMove()
      };
    },
    afterMove () {
      if (this.isFinished() ) {
        return
      }
      this.sendInfo()
    },
    sendInfo() {
      let info = []
      info['history'] = this.game.history()
      info['fen'] = this.game.fen()
      info['pgn'] = this.game.pgn()
      info['shapes'] = this.board.state.drawable.shapes
      this.$emit('onMove', info)
    },
    isFinished() {
      if (this.game.in_checkmate()) {
        if (this.toColor() == 'black') {
          this.$emit('onEnd', "YOU WIN")
        } else {
          this.$emit('onEnd', "YOU LOSE")
        }
        return true
      } else if (this.game.in_draw() ) {
        this.$emit('onEnd', "DRAW")
        return true
      } else if (this.game.in_stalemate() ) {
        this.$emit('onEnd', "STALEMATE")
        return true
      }
      return false

    },
    loadPosition () { // set a default value for the configuration object itself to allow call to loadPosition()
      this.game.load(this.fen)
      this.board = Chessground(this.$refs.board, {
        fen: this.game.fen(),
        turnColor: this.toColor(),
        movable: {
          color: this.toColor(),
          dests: this.possibleMoves(),
        },
        drawable: {
          enabled: true
        } 
      })
      this.board.set({
        movable: { events: { after: this.userPlay()} },
      })
      this.board.setShapes(this.shapes)
    },
  },
  mounted () {
    this.loadPosition()
  },
  created () {
    this.game = new Chess()
    this.board = null
    this.promotions = []
    this.promoteTo = 'q'
  },
}
</script>
