<template>
  <div id="app">
    <section class="hero is-fullheight">
      <div class="hero-head">
        <nav class="navbar">
          <div class="container">
            <div>
              <a href="/">
                <p class="title">Chess Common positions</p>
                <p class="subtitle">Show the most common positions from a pgn file.</p>
              </a>
            </div>
            <div id="navbarMenu" class="navbar-menu">
              <div class="navbar-end"></div>
            </div>
          </div>
        </nav>
      </div>
      <div class="hero-body">
        <div class="container">
          <b-input v-model="pgnText" rows="5" type="textarea"></b-input>
          <div class="has-text-centered">Paste the pgn file content and then generate the most common positions. </div>

          <p class="control has-text-centered">
            <progress class="progress" :value="progress" :max="fens.length">15%</progress>
            <button @click="generatePositions" class="button is-large is-primary">Generate positions</button>
          </p>
          <br>
          <div class="columns is-multiline"> 
            <div v-for="f in fens.slice(50,100)" class="column is-one-third">
              <div class="card">
                <div class="card-content">
                  <chessboard :fen="f"/>
                  <div style="font-size:0.6rem">{{f}}</div>
                  <div class="content is-size-7">Count: {{counter[f]}}</div>
                </div>
              </div>
            </div>
          </div>
        </div>

      </div>

      <div class="hero-foot">
        <div class="container">
          <div class="has-text-centered">
            Made by <a href="http://vitomd.com">@vitomd</a> with <a href="https://vuejs.org/">Vue.js</a> | Check all my <a href="http://vitomd.com/blog/projects/">chess related projects</a>
          </div>
        </div>
      </div>
    </section>
    <a href="https://github.com/vitogit/lichess-tactics-generator"><img src="https://camo.githubusercontent.com/38ef81f8aca64bb9a64448d0d70f1308ef5341ab/68747470733a2f2f73332e616d617a6f6e6177732e636f6d2f6769746875622f726962626f6e732f666f726b6d655f72696768745f6461726b626c75655f3132313632312e706e67" alt="Fork me on GitHub" data-canonical-src="https://s3.amazonaws.com/github/ribbons/forkme_right_darkblue_121621.png" style="position: absolute; top: 0px; right: 0px; border: 0px;z-index:1000"></a>    
  </div>

</template>

<script>
import chessboard from './components/chessboard'
import './components/style/theme.css'

import Chess from 'chess.js'

export default {
  name: 'app',
  components: {
    chessboard
  },
  data () {
    return {
      pgnText: '',
      fens: [],
      counter: [],
      progress: 0
    }
  },
  methods: {
    downloadLichess() {
      let loading = this.$loading.open({
        canCancel: true
      })
      let toast = this.$toast.open('Downloading and processing games. This will take a minute')
      let url = `https://lichess.org/games/export/${this.username}?max=50&analysed=true&evals=true&moves=true&perfType=blitz,rapid,classical`
      this.$http.get(url, {headers: {'Accept': 'application/x-ndjson'}}).then(result => {
        this.tactics = []
        let games = result.bodyText.split("\n").filter(x => x !== "").map(x => JSON.parse(x))
        this.generatePositions()
        loading.close()
      }, error => {
        console.error(error);
      });
    },
    generatePositions() {
      this.pgnText = this.pgnText || this.pgnExample
      let games = this.pgnText.split("\n\n")
      // if (games.length > 200) {
      //   alert('Pgn too big')
      //   return
      // }
      let positions = []
      for (let game of games ) {
        this.progress++
        let tempChess = new Chess()
        if ( tempChess.load_pgn(game)) {
          let history = tempChess.history()
          if (history) {
            let chess = new Chess()
            for (let move of history) {
              chess.move(move)
              let fen = chess.fen().split(' ')[0] + ' w KQkq - 0 1'
              positions.push(fen)
            }
          }
        }
        

      }
      console.log("positions________",positions)
      var arr = positions
      var cnts = arr.reduce( function (obj, val) {
          obj[val] = (obj[val] || 0) + 1;
          return obj;
      }, {} );
      console.log("cnts________",cnts)
      //Use the keys of the object to get all the values of the array
      //and sort those keys by their counts
      var sorted = Object.keys(cnts).sort( function(a,b) {
          return cnts[b] - cnts[a];
      });
      console.log(sorted);
      console.log("sorted[0]________",sorted[0])
      console.log("cnts(sorted[0])________",cnts[sorted[0]])
      console.log("this.fens________",this.fens)
      this.fens = sorted
      this.counter = cnts
    }
  },
  created() {
    this.pgnExample = `
1. e4 e5 2. f4 d6 3. Nf3 Be7 4. Bc4 Nc6 5. fxe5 Bh4+ 6. g3 Be7 7. O-O Bg4 8. Bxf7+ Kf8 9. d3 Nxe5 10. Bxg8 Kxg8 11. Nbd2 Bg5 12. Kg2 Qe8 13. d4 Bxd2 14. Bxd2 Qh5 15. dxe5 Rf8 16. Bf4 dxe5 17. Qd5+ Rf7 18. Qxe5 Bh3+ 19. Kh1 Qxe5 20. Bxe5 Bxf1 21. Rxf1 Kf8 22. Kg2 Ke7 23. Bxc7 Rc8 24. Ba5 Rxc2+ 25. Rf2 Rc4 26. Re2 Ke6 27. Ng5+ 1-0


1. e4 c5 2. d4 cxd4 3. c3 e5 4. Nf3 Nc6 5. Bc4 h6 6. O-O a6 7. cxd4 exd4 8. Nxd4 Nf6 9. Nc3 Bc5 10. Nxc6 bxc6 11. e5 Nh7 12. Qf3 O-O 13. Qe4 g6 14. Bxh6 Re8 15. Bxf7+ Kxf7 16. Qc4+ Re6 17. Qxc5 Rb8 18. f4 Qb6 19. Qf2 Qxb2 20. Qxb2 Rxb2 21. Ne4 a5 22. Nd6+ Ke7 23. Nxc8+ Kd8 24. Nd6 Rd2 25. a4 Rdxd6 26. exd6 Rxd6 27. Rad1 Re6 28. Rfe1 Re8 29. Rxe8+ Kxe8 30. Bg5 d5 31. Rc1 Kd7 32. Rc5 Kd6 33. Rxa5 Nf8 34. Ra7 Nd7 35. h4 c5 36. f5 gxf5 37. Bf4+ Kc6 38. h5 Nf8 39. h6 c4 40. h7 Nxh7 41. Rxh7 Kc5 42. Rc7+ Kb4 43. Rb7+ Kxa4 44. Bc1 d4 45. Kf2 Ka5 46. Kf3 Ka6 47. Rb8 Ka7 48. Rb4 c3 49. Rxd4 Kb6 50. Rc4 Kb5 51. Rxc3 Kb6 52. Kf4 Kb5 53. Kxf5 1-0
    
    `
  }
}
</script>

<style>
  .hero-head {
    padding-top: 10px;
  }
  .hero-foot {
    padding-bottom: 10px;
  }
  pre, code {
      white-space: pre-line;
      padding: 10px
  }
</style>
