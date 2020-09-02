<template>
  <div class="main">
    <div>x: {{mousex}}, y: {{mousey}}</div>
    <div class="header">
      <div class="top">
        <div class="title">Freecell</div>
        <div class="countAndTime">count | time</div>
        <div class="btn">
          <img src="./assets/images/Menu.svg" width="30" style="margin-top:13px;">
        </div>
      </div>
      <div class="putBox_block">
        <div ref="tempPut_rect" style="margin-right: 88px;">
          <div v-for="(item, index) in putCard.tempPut" :key="index"
            :class="{'putBox': !item, 'tempPut': item}"
          >
            <img v-if="item" :src="require('./assets/images/'+ numTransformIcon(item.num) +'.png')" width="100" heigth="154">
          </div>
          <!-- <div class="putBox tempPut"></div>
          <div class="putBox tempPut"></div>
          <div class="putBox tempPut"></div>
          <div class="putBox tempPut"></div> -->
        </div>
        <div ref="sortPut_rect">
          <div v-for="(item, index) in putCard.sort" :key="index"
            :class="{'putBox': !item[0], 'sort': item[0]}">
            <img v-if="item[0]" :src="require('./assets/images/'+ numTransformIcon(item[item.length-1].num) +'.png')" width="100" heigth="154">
            <span v-else>A</span>
          </div>
          <!-- <div class="putBox sort">A</div>
          <div class="putBox sort">A</div>
          <div class="putBox sort">A</div>
          <div class="putBox sort">A</div> -->
        </div>
      </div>
      <div class="clearboth"></div>
    </div>

    <div class="card_block">
      <div v-for="(base, col) in putCard.base" :key="col" class="base_row">
        <div class="base_card" 
          v-for="(card, row) in base" 
          :ref="'base_card'+row+col"
          :key="row"
          :style="{top:card.posTop+'px', left:card.posLeft+'px'}"
          @mousedown="(e)=>{mounsedownEvent(e, card, col, row)}"
        >
          <img :src="require('./assets/images/'+ numTransformIcon(card.num) +'.png')" width="100" heigth="154">
        </div>
      </div>
    </div>

    <div class="drag_block"
      :style="{top:dragItem_test.posTop+'px', left:dragItem_test.posLeft+'px'}"
      :class="{'transition': dragItem_test.returning, 'zindex': false}"
    >
      <div class="drag_card"
        v-for="(card, index) in dragItem_test.cards"
        :key="index"
        :style="{top:card.posTop+'px', left:card.posLeft+'px'}"
      >
        <img :src="require('./assets/images/'+ numTransformIcon(card.num) +'.png')" width="100" heigth="154">
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      pokerCards: [],
      putCard: {
        base: [[],[],[],[],[],[],[],[]],
        tempPut: ['','','',''],
        sort: [[],[],[],[]]
      },
      dragItem: {},
      dragItem_test: {
        posTop: 0,
        posLeft: 0,
        cards: [],
      },
      definAry: [1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52],
      mousex: '',
      mousey: '',
      mouseStartX: '',
      mouseStartY: '',
      targetTop: '',
      targetLeft: '',
      purpose: '',

      presetPutBoxXY: {
        tempPut: [],
        sort: []
      },
      firstCardXY:{
        x: '',
        y: ''
      },
      lastTimeStemp: ''
    }
  },
  created() {
    this.getRandomNum(52, this.definAry);
    document.documentElement.addEventListener('mousemove', (e)=>{
      this.mousex = e.clientX;
      this.mousey = e.clientY;
    });
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      // get tempPut rect x,y
      let firstTempPutX = this.$refs.tempPut_rect.getBoundingClientRect().x + 7;
      let firstTempPutY = this.$refs.tempPut_rect.getBoundingClientRect().y + 2;

      // get sortput rect x,y
      let firstSortPutX = this.$refs.sortPut_rect.getBoundingClientRect().x + 7;
      let firstSortPutY = this.$refs.sortPut_rect.getBoundingClientRect().y + 2;

      for(let i=0; i<4; i++) {
        let tempObj = {
          x: firstTempPutX + i*114,
          y: firstTempPutY
        }
        let sortObj = {
          x: firstSortPutX + i*114,
          y: firstSortPutY
        }

        this.presetPutBoxXY.tempPut.push(tempObj);
        this.presetPutBoxXY.sort.push(sortObj);
      }

      // console.log(this.$refs['base_card00'][0].getBoundingClientRect());
      this.firstCardXY.x = this.$refs['base_card00'][0].getBoundingClientRect().x;
      this.firstCardXY.y = this.$refs['base_card00'][0].getBoundingClientRect().y;
    },
    // random number
    getRandomNum(num, arr) {
      // concat definAry
      let theAry = [].concat(arr); 
      let baseIndex = 0, row = 1;

      for(let i=0;i<num;i++){
        let n=Math.floor(Math.random()*theAry.length);
        let obj = {
          num: theAry[n],
          posTop: this.calY(row),
          posLeft: this.calX(baseIndex)
        }

        this.putCard.base[baseIndex].push(obj);
        row++;
        if(this.putCard.base[baseIndex].length>6 && baseIndex<4) {
          row = 1;
          baseIndex++;
        } else if(this.putCard.base[baseIndex].length>5 && baseIndex>3) {
          row = 1;
          baseIndex++;
        }
        theAry.splice(n,1);
      }
    },
    // num to path
    numTransformIcon(num) {
      // 梅花: Clubs 1~13, 方塊: Diamonds 14~26, 愛心: Hearts 27~39, 黑桃: Spades 40~52
      let ary = [], iconPath, typeText;
        if(num<14) {
          iconPath = num + '_Clubs';
        } else if(13<num && num<27) {
          iconPath = num-13 + '_Diamonds';
        } else if(26<num && num<40) {
          iconPath = num-26 + '_Hearts';
        } else {
          iconPath = num-39 + '_Spades';
        }
        return iconPath;
    },
    // mouse Event
    mounsedownEvent(e, card, col, row) {
      // 避免返回未完成就執行
      if(this.lastTimeStemp && this.lastTimeStemp + 500 > e.timeStamp) return;
      
      // let lastNum = this.putCard.base[col].length - 1;
      // if(type != 'tempPut' && card.num != this.putCard.base[row][lastNum].num) {
      //   console.log('不是最後一個');
      //   return;
      // }

      // ----------------- drag分離
      if(!this.isCanDrag(card, col, row)) {
        return;
      }      

      let lestNum = this.putCard.base[col].length;
      this.dragItem_test.col = col;
      this.dragItem_test.cards = this.putCard.base[col].splice(row, lestNum - row);
      this.dragItem_test.returning = false;
      this.dragItem_test.draging = true;
      // -----------------

      this.dragItem = card;
      this.dragItem.col = col;

      this.mouseStartX = e.clientX;
      this.mouseStartY = e.clientY;

	    e.stopPropagation();
      e.preventDefault();
      
      document.documentElement.addEventListener('mousemove', this.mousemoveEvent);
      document.documentElement.addEventListener('mouseup', this.mouseupEvent);
      
    },
    mousemoveEvent(e) {
      let moveLeft = this.mouseStartX - e.clientX;
      let moveTop = this.mouseStartY - e.clientY;

      this.dragItem_test.posTop = 0 - moveTop;
      this.dragItem_test.posLeft = 0 - moveLeft;
    },
    mouseupEvent(e) {
      let me = this;
      this.lastTimeStemp = e.timeStamp;
      let isSucceedDrag = false;
      // let dragItemBaseCol = this.dragItem.col;
      // let dragBaseLastIndex = this.putCard.base[dragItemBaseCol].length - 1;

      let dragItem_quantity = this.dragItem_test.cards.length;
      // 判斷拖曳方塊 tempPut or sort
      if(dragItem_quantity == 1) {
        isSucceedDrag = this.putTempOrSort(e);
      }

      // stack
      if(!isSucceedDrag) {
        for(let col=0; col<8; col++) {
          let lastIndex = this.putCard.base[col].length-1;
          let baseBoundX = this.firstCardXY.x + 150*col;
          let baseBoundY_last = this.firstCardXY.y + lastIndex*35;
          // 位置
          if(baseBoundX < e.clientX &&  e.clientX < baseBoundX + 100 && baseBoundY_last < e.clientY && e.clientY < baseBoundY_last + 153) {
            // 數值
            if(this.isStack(col, lastIndex)) {
              this.putCard.base[col].push(this.dragItem);
              // this.putCard.base[dragItemBaseCol].splice(dragBaseLastIndex, 1);
              isSucceedDrag = true;
              type = 'stack'
            }
            break;
          }
        }
      }

      // 返回拖曳前位置
      if(!isSucceedDrag) {
        this.dragItem_test.returning = true;
        this.dragItem_test.posTop = 0;
        this.dragItem_test.posLeft = 0;
        setTimeout(()=>{
          me.putCard.base[me.dragItem_test.col]= me.putCard.base[me.dragItem_test.col].concat(me.dragItem_test.cards);
          me.dragItem_test.cards = [];
        }, 500);
      }

      document.documentElement.removeEventListener('mousemove', this.mousemoveEvent);
      document.documentElement.removeEventListener('mouseup', this.mouseupEvent);

      this.dragItem_test.draging = false;
    },
    putTempOrSort(e) {
      let isSucceedDrag = false;
      for(let i=0; i<4; i++) {
        let tempPutX = this.presetPutBoxXY.tempPut[i].x;
        let tempPutY = this.presetPutBoxXY.tempPut[i].y;
        let sortX = this.presetPutBoxXY.sort[i].x;
        let sortY = this.presetPutBoxXY.sort[i].y;

        if(tempPutX < e.clientX && e.clientX < tempPutX + 100 && tempPutY < e.clientY && e.clientY < tempPutY + 153) {
          if(!this.putCard.tempPut[i]) {
            this.putCard.tempPut[i] = this.dragItem_test.cards[0];
            isSucceedDrag = true;
          }
          break;
        } else if(sortX < e.clientX && e.clientX < sortX + 100 && sortY < e.clientY && e.clientY < sortY + 153) {
          if(this.isSameSuit(this.dragItem_test.cards[0].num, i)) {
            this.putCard.sort[i].push(this.dragItem_test.cards[0]);
            isSucceedDrag = true;
          }
          break;
        }
      }
      if(isSucceedDrag) {
        this.dragItem_test.cards = [];
        this.dragItem_test.posTop = 0;
        this.dragItem_test.posLeft = 0;
        this.lastTimeStemp = '';
        return true;
      } else {
        return false;
      }
    },
    isSameSuit(num, i) {
      let dragAry = this.numTransformIcon(num).split('_');
      if(this.putCard.sort[i].length > 0) {
        let lastNum = this.putCard.sort[i][this.putCard.sort[i].length-1].num;
        let lastAry = this.numTransformIcon(lastNum).split('_');

        if(lastAry[1] == dragAry [1] && lastAry[0] == dragAry[0]-1) {
          return true;
        } else {
          return false;
        }
      } else {
        if(dragAry[0] == 1) return true;
        else return false;
      }
    },
    isCanDrag(card, col, row) {
      let result = false;
      let totalLength = this.putCard.base[col].length;

      if(card.num == this.putCard.base[col][row].num) return true;
      
      for(let r=row; row<totalLength; row++) {
        if(!this.putCard.base[col][r+1]) break;
        let card = this.numTransformIcon(this.putCard.base[col][r].num).split('_');
        let cardColor = (card[1] == 'Clubs' || card[1] == 'Spades') ? 'black' : 'red';
        let nextCard = this.numTransformIcon(this.putCard.base[col][r+1].num).split('_');
        let nextColor = (nextCard[1] == 'Clubs' || nextCard[1] == 'Spades') ? 'black' : 'red';

        if(card[0] == nextCard[0]-1 && cardColor != nextColor) {
          result = true;
        } else {
          result = false;
        }
      }

      return result;
    },
    isStack(col, lastIndex) {
      let dragAry = this.numTransformIcon(this.dragItem.num).split('_');
      let dragColor = (dragAry[1] == 'Clubs' || dragAry[1] == 'Spades') ? 'black' : 'red';
      let ppCardAry = this.numTransformIcon(this.putCard.base[col][lastIndex].num).split('_');
      let ppCardColor = (ppCardAry[1] == 'Clubs' || ppCardAry[1] == 'Spades') ? 'black' : 'red';

      if(dragColor != ppCardColor && dragAry[0] == ppCardAry[0]-1) {
        this.dragItem.posLeft = this.putCard.base[col][lastIndex].posLeft;
        this.dragItem.posTop = this.putCard.base[col][lastIndex].posTop + 35;
        return true;
      } else {
        return false;
      }      
    },
    // cal x, y
    calY(row) {
      return row*35 + 220;
    },
    calX(col) {
      return col*150 + 25;
    }
  },
  computed: {
  }
}
</script>

<style scope lang="scss">
body {
  margin: 0;
  background-color: #0a141d;
  color: #fff;
}
.main {
  margin: 0 auto;
  width: 1200px;
  height: 937px;
  position: relative;
}

.header {
  width: 100%;
  margin-bottom: 20px;
  .top {
    display: flex;
    width: 100%;
    height: 50px;
    line-height: 50px;
    padding: 0px 10px;
    box-sizing: border-box;
    // z-index: 4;
  }
  .title{
    font-size: 20px;
    letter-spacing: 0.05em;
    color: #ffac4e;
    text-align: left;
  }

  .countAndTime {
    width: 87%;
    text-align: center;
    height: 50px;
    line-height: 50px;
    font-size: 16px;
  }
  
  .putBox_block{
    width: 1000px;
    margin: 0 auto;
    div {
      float: left;
    }
  }
  .putBox{
    margin: 0 5px;
    border: 2px solid #ffac4e;
    width: 100px;
    height: 153px;
    border-radius: 10px;
    line-height: 157px;
    text-align: center;
  }
  .tempPut, .sort {
    margin: 0px 5px;
    border: 2px solid rgba(255, 172, 78, 0);
    width: 100px;
    height: 153px;
    border-radius: 10px;
    line-height: 157px;
    text-align: center;
  }
}
.clearboth {
  clear: both;
}

.card_block {

  .base_row {
    width:100px;
    float: left;
  }

  .base_card{
    position: absolute;
  }
}
.drag_block {
  position: absolute;
  .drag_card {
    position: absolute;
  }
}
.transition {
  transition: all 0.5s;
  z-index: 1;
}
.zindex {
  z-index: 999;
}
</style>
