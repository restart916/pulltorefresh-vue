
<template>
    <div class="wrap-pull" ref="wrap" @scroll="scrollEvent">
            <div :class="pullDownCls" ref="pullDownEl" v-show="showButton">
                <span class="icon"></span>
                <span :class="pullDownLabelCls">{{pullDownState}}</span>
            </div>
            <slot></slot>
            <div :class="pullUpCls" ref="pullUpEl" v-show="_showButton">
                <span class="icon"></span>
                <span :class="pullUpLabelCls">{{pullUpState}}</span>
            </div>
    </div>
</template>

<script>
const scrollCls = 'scroll',
  pullDownCls = 'pullDown',
  pullDownLabelCls = 'pullDownLabel',
  pullUpCls = 'pullUp',
  pullUpLabelCls = 'pullUpLabel';
const defaultLableUp = {
  initial: '上拉加载更多',
  suspend: '松开加载更多',
  loading: '加载中',
  complete: '加载完成',
};
const defaultLableDown = {
  initial: '下拉刷新',
  suspend: '松开刷新',
  loading: '刷新中',
  complete: '刷新完成',
};
export default {
  name: 'PullToRefresh',
  props: [
    'down',
    'up',
    'pullupOffset',
    'pulldownOffset',
    'addNew',
    'addMore',
    'propsLableUp',
    'propsLableDown'
  ],
  data() {
    return {
      msg: 'PullToRefresh',
      pullDownState: '',
      pullUpState: '',
      pullDownCls,
      pullDownLabelCls,
      pullUpCls,
      pullUpLabelCls,
      lableUp: {},
      lableDown: {},
      _showButton: true,
    };
  },
  components: {},
  methods: {
    scrollEvent(e) {
      this.$emit('scroll', e);
    },
    touchStart(ev) {
      console.log('touchStart');
      let touch = ev.touches[0];
      let scrollObj = this.scrollObj;
      this.pullFlag = 0;
      if (this.down) {
        this.pullDownEl.style.webkitTransitionDuration = '0s';
        this.pullDownCls = pullDownCls;
        this.pullDownState = this.lableDown.initial;
      }
      if (this.up && this.pullUpEl) {
        this.pullUpEl.style.webkitTransitionDuration = '0s';
      }
      this.startY = touch.screenY;
      this.startPageY = scrollObj.scrollTop;
      this.maxY = scrollObj.scrollHeight - scrollObj.clientHeight;
    },
    touchMove(ev) {
      let len = this.pulldownOffset || 80,
        offsetDefault = this.pullupOffset || 20;
      let offsetY, touch;
      touch = ev.touches[0];
      offsetY = (touch.screenY - this.startY) / 2;
      // console.log('pulldown', this.down, offsetY, len);
      if (this.down && this.startPageY == 0 && offsetY > 0) {
        // ev.preventDefault();
        if (offsetY > len) {
          offsetY = len;
          this.pullDownState = this.lableDown.suspend;
          this.pullDownCls = 'pullDown flip';
          this.pullFlag = 1;
        }
        this.pullDownEl.style.height = offsetY + 'px';
      } else if (
        this.up &&
        this.startPageY >= this.maxY - offsetDefault &&
        offsetY < 0
      ) {
        // ev.preventDefault();
        this.pullFlag = 2;
        this.pullUpCls = 'pullUp flip';
        this.pullUpState = this.lableUp.suspend;
      }
    },
    touchEnd(ev) {
      if (this.down && this.pullFlag == 1) {
        this.pullDownCls = 'pullDown loading';
        this.pullDownState = this.lableDown.loading;
        // console.log('loadData');
        this.addNew().then(() => {
          this.pullDownEl.style.webkitTransitionDuration = '0.5s';
          // this.pullDownEl.style.height = 0;
          this.pullDownCls = 'pullDown ok';
          this.pullDownState = this.lableDown.complete;
        });
      } else if (this.up && this.pullFlag == 2) {
        this.pullUpCls = 'pullUp loading';
        this.pullUpState = this.lableUp.loading;

        this.addMore().then(() => {
          this.pullUpCls = 'pullUp';
          this.pullUpState = this.lableUp.initial;
        });
      } else if (this.down) {
        this.pullDownEl.style.webkitTransitionDuration = '0.5s';
        // this.pullDownEl.style.height = 0;
      }
      this.pullFlag = 0;
    },
    showButton() {
      this._showButton = true
    },
    hideButton() {
      this._showButton = false
    },
  },

  mounted() {
    this.lableUp = Object.assign(defaultLableUp, this.propsLableUp);
    this.lableDown = Object.assign(defaultLableDown, this.propsLableDown);

    this.pullDownState = this.lableDown.initial;
    this.pullUpState = this.lableUp.initial;

    this.scrollObj = this.$refs.wrap;
    this.pullDownEl = this.$refs.pullDownEl;

    this.pullUpEl = this.$refs.pullUpEl;

    this.scrollObj.addEventListener('touchstart', this.touchStart.bind(this), {passive: false});
    this.scrollObj.addEventListener('touchmove', this.touchMove.bind(this)), {passive: false};
    this.scrollObj.addEventListener('touchend', this.touchEnd.bind(this)), {passive: false};
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
@import './PullToRefresh.css';
</style>
