<template>
  <div id="app">
    <div class="container">
      <div class="row border justify-content-center">
        <header class="app__nav nav text-center d-flex align-items-center">          
          <button v-if="stage === 'not-started'" v-on:click="startGame"
                  class="btn btn-primary m-auto" >
            Start game
          </button> 
          
          <button v-if="gameFinished" v-on:click="resetBoard"
                  class="btn btn-primary m-auto" >
            Start over
          </button> 

          <span class="ml-3">
            {{ this.game.players[this.currentPlayer].name }} {{ this.stage }}

            {{ gameFinished ? 'FINISH!' : '' }}
            {{ (typeof winner === 'number' && (0 > winner)) ? 'Draw!' : '' }}
            {{ typeof winner === 'number' && winner >= 0 ? 'Won ' + game.players[winner].name  : '' }}

          </span>
        </header>

        <Board v-bind:game="game" 
              :currentPlayer="currentPlayer"
              :stage="stage"
        @move-stones="moveStones" />

        <footer class="position-absolute bottom-0"></footer>
      </div>
    </div>
    <img alt="Vue logo" src="./assets/logo.png">
  </div>
</template>

<script>
import Board from './components/Board.vue'

export default {
  name: 'App',
  components: {
    Board
  },
  data() {
    return {
      game: {
        players: [ { 
                    name: 'Alvaro',
                    stonePoints: 0,
                    buckets: [
                      {
                        stones: 4,
                        index: 0,
                        oponentBucketIndex: 5
                      },
                      {
                        stones: 4,
                        index: 1,
                        oponentBucketIndex: 4
                      },
                      {
                        stones: 4,
                        index: 2,
                        oponentBucketIndex: 3
                      },
                      {
                        stones: 4,
                        index: 3,
                        oponentBucketIndex: 2
                      },
                      {
                        stones: 4,
                        index: 4,
                        oponentBucketIndex: 1
                      },
                      {
                        stones: 4,
                        index: 5,
                        oponentBucketIndex: 0
                      },
                    ]
                  },
                  { 
                    name: 'Player2 ',
                    stonePoints: 0,
                    buckets: [
                      {
                        stones: 4,
                        index: 0,
                        oponentBucketIndex: 5
                      },
                      {
                        stones: 4,
                        index: 1,
                        oponentBucketIndex: 4
                      },
                      {
                        stones: 4,
                        index: 2,
                        oponentBucketIndex: 3
                      },
                      {
                        stones: 4,
                        index: 3,
                        oponentBucketIndex: 2
                      },
                      {
                        stones: 4,
                        index: 4,
                        oponentBucketIndex: 1
                      },
                      {
                        stones: 4,
                        index: 5,
                        oponentBucketIndex: 0
                      },
                    ]
                }, ],
      },
      currentPlayer: 0,
      stage: 'not-started',
      winner: null,
      stageParams: {}
    }
  },
  computed: { // we dont need a computed for this. A method would be enough. Because in the computed we can't tun code to change other data.
    gameFinished: function() {
      const stones = this.getTotalStonesInBuckets(this.currentPlayer);
      console.log('calculating game finished', stones);
      return !stones ? true : false;
    }
  },
  methods: {    
    startGame(e) {      
      e.preventDefault();
      this.stage = 'move';
    },
    getCurrentPlayer() { return this.game.players[this.currentPlayer]; },
  
    // when clicking on a bucket
    moveStones( playerIndex, bucket ) { // { stones, index }
      console.log( 'player index before acting' , playerIndex );
      console.log( 'bucket ba ', bucket );
      if (! bucket.stones ) return;
      let numberOfStones = bucket.stones;
      let bucketDeposit = bucket.index;
      let playerDeposit = playerIndex;
      this.game.players[playerIndex].buckets[bucketDeposit].stones = 0;
      while (numberOfStones) {        
        bucketDeposit = (0 > bucketDeposit)? 0 : bucketDeposit + 1; // index where to place the stone
        // If arrived to the bucket of counting
        if ( bucketDeposit === this.game.players[playerDeposit].buckets.length ) {
          if (playerDeposit === playerIndex) { // got the counter bucket
            this.game.players[playerIndex].stonePoints++;
            bucketDeposit = -1;
            playerDeposit = this.getOponentPlayerIndex(playerDeposit);
          } else { // got to the end of oponents bucket: back to 0 in our buckets
            playerDeposit = playerIndex;
            bucketDeposit = 0;
            this.game.players[playerDeposit].buckets[bucketDeposit].stones++;
          }
        } else {
          this.game.players[playerDeposit].buckets[bucketDeposit].stones++;
        }
        numberOfStones--;
      }
      
      /*** the stones where moved. The last stone was deposited in the bucketDeposit of playerDeposit. */
      
      // check if steals from oponent player
      const lastBucketDeposit = this.game.players[playerDeposit].buckets[bucketDeposit];
      if ( this.currentPlayer === playerDeposit && bucketDeposit >= 0 )
        if ( lastBucketDeposit.stones === 1 )  {
          // conditions approved! get oposite bucket
          const oponentBucket = this.game.players[this.getOponentPlayerIndex(playerDeposit)].buckets[lastBucketDeposit.oponentBucketIndex];
          this.game.players[playerIndex].stonePoints += oponentBucket.stones;
          oponentBucket.stones = 0;
        }

      // check change of player. If bucketDeposit is -1, is because last stone was in the point counter
      if ( bucketDeposit === -1 ) {
        console.log('still turn of  ', this.game.players[playerIndex].name)
      } else {
        this.currentPlayer = this.getOponentPlayerIndex(playerIndex); // turn of the other player
      }

      // check if finish game
      console.log(`Game finished: ${this.gameFinished}`);
      if (this.gameFinished) {
        this.stage = 'game-finished'; // see watch to run the finish game actions
      }

    },
    resetBoard() {
      this.game.players[0].buckets = this.game.players[0].buckets
            .map( (buc) => Object.assign({ ... buc }, { stones: 4 } ) );
      this.game.players[1].buckets = this.game.players[1].buckets
            .map( (buc) => Object.assign({ ... buc }, { stones: 4 } ) );
      this.game.players[0].stonePoints = 0;
      this.game.players[1].stonePoints = 0;
      this.stage = 'move';
      this.currentPlayer = 0;
    },
    // helper
    getOponentPlayerIndex(playerIndex) {
      return playerIndex === 1 ? 0 : 1;
    },
    getTotalStonesInBuckets(playerIndex) {
      return this.game.players[playerIndex].buckets.reduce( (acc, bucket ) => acc + bucket.stones, 0 );
    }
  }, // end methods
  // Watch
  watch: {
    stage: function (val) {
      console.log(`stage ${val}`);
      if ('game-finished' === this.stage) {
        console.log('stage is finsihed? ', this.gameFinished);
        const oponentIndex = this.getOponentPlayerIndex(this.currentPlayer);
        const stonesCurrent = this.getTotalStonesInBuckets(this.currentPlayer);
        const stonesOponent = this.getTotalStonesInBuckets(oponentIndex);
        if (stonesOponent && (0 === stonesCurrent)) {
          console.log(`updating stones and decide who wons`);
          this.game.players[oponentIndex].stonePoints += stonesOponent;
          this.game.players[oponentIndex].buckets = this.game.players[oponentIndex].buckets
            .map( (buc) => Object.assign({ ... buc }, { stones: 0 } ) );
          if (this.game.players[oponentIndex].stonePoints > this.game.players[this.currentPlayer].stonePoints)
            this.winner = oponentIndex;
          else if (this.game.players[oponentIndex].stonePoints < this.game.players[this.currentPlayer].stonePoints)
            this.winner = this.currentPlayer;
          else this.winner = -1;
        }
      }
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
