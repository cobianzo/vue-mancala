<template>
  <a  class="Bucket" 
      v-bind:class="{ 'col-6': bucket.index >= 0, 
                      'col-12': bucket.index < 0 ,
                      'active': canMove,
                      'pl-1': playerIndex === 1,
                      'pl-0': playerIndex === 0
                      }"
      v-on="canMove ? { click: handleClickOnBucket } : null">

        {{ bucket.stones }} 

    <small>
      ix: {{ bucket.index }}
    </small>
  </a>
</template>

<script>
export default {
  name: 'Bucket',
  props: {
    currentPlayer: Number,
    playerIndex: Number,
    stage: String,
    bucket: Object
  },
  data() {
    return {
      dataStage: this.stage
    }
  },
  computed: {
    canMove: function() {
      return ( this.currentPlayer === this.playerIndex && 'move' === this.stage );
    }
  },
  methods: {
    handleClickOnBucket(e) {
      e.preventDefault();
      if (! this.canMove ) return;
      console.log('can Move, ' + this.playerIndex + ' ' + this.currentPlayer + ' ' + this.stage + '. ' + this.dataStage, this.canMove );
      this.$emit('move-stones', this.playerIndex, this.bucket);
    }
    // dontHandle(e) {
    //   e.preventDefault();
    //   console.log('jander');
    // }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .Bucket {
    border: 1px solid gray;
  }
  .Bucket.active {
    border: 1px solid red;
  }
  .Bucket.active:hover {
    background: rgb(255, 181, 181);
  }
  .Bucket.pl-0 {
    background: #beffb5;
  }
  .Bucket.pl-1 {
    background: #f0d3a8;
  }
</style>
