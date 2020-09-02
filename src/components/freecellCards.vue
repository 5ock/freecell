<template>
  <div>
    <!-- <dragCard /> -->
    <div v-for="(item, index) in cardIcon_list" :key="index" style="display:inline-block">
      <img :src="require('../assets/images/'+ item.path +'.png')" width="100">
    </div>
  </div>
</template>

<script>
import dragCard from './dragCard'
// 梅花: Clubs 1~13, 方塊: Diamonds 14~26, 愛心: Hearts 27~39, 黑桃: Spades 40~52
export default {
  name: 'freecell',
  components: {
    dragCard,
  },
  props:['randomNum'],
  data() {
    return {
      
    }
  },
  methods: {
    numTransformIcon(nums) {
      let ary = [], iconPath, typeText;
      for(let i=0; i<nums.length; i++) {
        if(nums[i]<14) {
          typeText = 'Clubs';
          iconPath = nums[i] + '_Clubs';
        } else if(13<nums[i] && nums[i]<27) {
          typeText = 'Diamonds';
          iconPath = nums[i]-13 + '_Diamonds';
        } else if(26<nums[i] && nums[i]<40) {
          typeText = 'Hearts';
          iconPath = nums[i]-26 + '_Hearts';
        } else {
          typeText = 'Spades';
          iconPath = nums[i]-39 + '_Spades';
        }
        ary.push({type:typeText, path:iconPath});
      }

      return ary;
    }
  },
  computed: {
    cardIcon_list() {
      return this.numTransformIcon(this.randomNum);
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
</style>
