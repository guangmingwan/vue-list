<template lang="html">
  <div class="vue-list" @scroll="handleScroll">
    <ul :style="{
      paddingTop: lineTopHeight +'px',
      paddingBottom: lineBottomHeight +'px'
    }">
      <!--<li class="line-top" :style="{height: lineTopHeight +'px'}"></li>-->
      <li v-for="item in previewList">{{item.title}}<img draggable="true" src="/static/img/test.png" width="32"></li>
      <!--<li class="line-bottom" :style="{height: lineBottomHeight +'px'}"></li>-->
    </ul>
    
  </div>
</template>

<script>

export default {
  name: 'vue-list',
  props: {
    list: {
      type: Array,
      required: true,
      default: [],
      twoWays: true
    },
    height: {
      type: Number,
      default: 144
    },
    canScroll: {
      type: Boolean,
      default: true
    },
    dispatchData: {
      type: Function
    }
  },
  data() {
    return {
      lastScrollTop: null,
      distance: 44,
      lineTopHeight: 0,
      lineBottomHeight: 0,
      canLoadmore: true,
      previewList: [],
      displayCount: 0
    }
  },
  mounted () {
    this.initData();
    this.handleScroll();
  },
  methods: {
    initData() {
      // init all data
      console.log("initdata")
      this._cols = 2;
      this._rowsInWindow = Math.ceil(this.$el.offsetHeight / this.height) * this._cols,
      this._above = this._rowsInWindow * 2,
      this._below = this._rowsInWindow,
      this._max = this._rowsInWindow * this.height;
      
    },
    handleScroll() {
      console.log("handleScroll")
      let _scrollTop = this.$el.scrollTop,
          _height = this.$el.querySelectorAll('ul')[0].offsetHeight,
          _contentHeight = this.$el.offsetHeight;

      // Counts the number of rows on the current screen
      if (_scrollTop / this.height - Math.floor(_scrollTop / this.height) > 0.5) {
        this.displayCount = Math.ceil(_scrollTop / this.height)* this._cols;
      } else {
        this.displayCount = Math.floor(_scrollTop / this.height)* this._cols;
      }

      // if the maximum height is exceeded, reset the previewList
      if (this.lastScrollTop === null || Math.abs(_scrollTop - this.lastScrollTop) > this._max) {
          this.lastScrollTop = _scrollTop;
      } else {
          if (this.to === this.list.length && _height - _scrollTop - _contentHeight < this.distance) {
            this.canScroll && this.loadmore(this.from, this.to);
          }
          return;
      }

      // get from and to count
      let _from = parseInt(_scrollTop / this.height) * this._cols - this._above;
      if (_from < 0) {
          _from = 0;
      }
      let _to = _from + this._above + this._below + this._rowsInWindow;
      if (_to > this.list.length) {
          _to = this.list.length;
      }
      this.from = _from;
      this.to = _to;

      // set top height and bottom height
      this.lineTopHeight = _from / this._cols * this.height;
      this.lineBottomHeight = (this.list.length - _to) /this._cols * this.height;

      // dispatch data
      if (typeof this.dispatchData === 'function') {
        this.dispatchData(this)
      }

      this.resetPreviewList(_from, _to);

      this.$nextTick(() => {
        let _scrollTop = this.$el.scrollTop,
            _height = this.$el.querySelectorAll('ul')[0].offsetHeight,
            _contentHeight = this.$el.offsetHeight;

        if (_to === this.list.length && _height - _scrollTop - _contentHeight < 0) {
            this.canScroll && this.loadmore(this.from, this.to);
        }
      });
    },
    loadmore(from, to) {
      if (!this.canLoadmore) return;
      this.canLoadmore = false;
      // fetch mock
      setTimeout(() => {
        
        let _from = from, _to = to + this._below;
        this.resetPreviewList(_from, _to);
        this.lineBottomHeight = (this.list.length - _to) / this._cols * this.height;
        this.handleScroll();

        this.canLoadmore = true;
      }, 2000)
    },
    resetPreviewList(from, to) {
      console.log("resetPreviewList",from,to)
      // reset previewList
      this.previewList = [];
      for (; from < to; from++) {
          this.previewList.push(this.list[from])
      }
    }
  },
  components: {}
}
</script>

<style lang="less">
.vue-list{
  width: 100%;
  height: 100%;
  overflow-y: auto;
  &::scroll-bar{
    width: 0;
  }
  ul {
    margin: 0;
    padding: 0;
    li{
      width: 100px;
      float: left;
      text-decoration: none;
      height: 144px;
      font-size: 14px;
      line-height: 3;
      text-align: left;
      padding-left: 15px;
      border-bottom: 1px solid #ddd;
      box-sizing: border-box;
      background: #fff;
      &.line-top, &.line-bottom{
        border-bottom: 0;
      }
    }
  }
  .load-more-gif{
    width: 100%;
    height: 144px;
    text-align: center;
    line-height: 144px;
    background: #fff;
    border-top: none;
    color: #48B884;
  }
}
</style>
