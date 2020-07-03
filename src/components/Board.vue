<template>
  <div class="Board container text-center">
    <ul class="Board__buckets-container">
      <li class="row">
        <Bucket v-bind:bucket="{stones: game.players[1].stonePoints, index: -1}"
                v-bind:playerIndex="1" />
      </li>
      <li v-for="(bucket, index) in game.players[0].buckets" v-bind:key="bucket.oponentBucketIndex"
          class="list-unstyled row">
          <Bucket v-bind:bucket="game.players[0].buckets[index]" v-bind:playerIndex="0" v-on="$listeners" :currentPlayer="currentPlayer" :stage="stage" />
          <Bucket v-bind:bucket="game.players[1].buckets[numberOfBuckets - index - 1]" :playerIndex="1" v-on="$listeners" :currentPlayer="currentPlayer" :stage="stage" />
      </li>
      <li class="row">
        <Bucket v-bind:bucket="{stones: game.players[0].stonePoints, index: -1}"       
                v-bind:playerIndex="0" />
      </li>
    </ul>
  </div>
</template>

<script>
import Bucket from './Bucket.vue';

export default {
  name: 'Board',
  components: { Bucket },
  props: {
    game: Object,
    currentPlayer: Number,
    stage: String
  },
  data() {
    return {
      numberOfBuckets: this.game.players[0].buckets.length
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .Board__buckets-container {
    max-width: 400px;
    margin: auto;
  }
</style>
