<!--
     desc: 无限滚动/下拉刷新
     User: luozhong
     Date: 2017/7/26
     Time: 15:07
     email:luozhong0521@163.com / luozhong@huoyunren.com
-->
<template>
    <div class="v-scroller-content" :style="{height: height === '' ? '100%' : height + 'px'}" :class="{infinity: infinity}" ref="vScroller">
        <div class="v-scroller-box" :class="{show: activeDown}" ref="vScrollerContent">
            <div class="v-scroller-body" :style="vScrollerDownStyle" ref="vScrollerBody">
                <slot name="refresh">
                    <pulldownLayer v-if="loadingType === 'G7'" :status="status.downStatus" :gap="0 - offsetTop"></pulldownLayer>
                    <div v-else class="refresh-content" :class="{active: status.downStatus === 'loading' || status.downStatus === 'down'}">
                        <div>
                            <img src="./ic-refresh@2x.png">
                        </div>
                        <p>下拉刷新</p>
                    </div>
                </slot>
                <slot></slot>
            </div>
            <div class="v-scroller-up-loading" v-show="(activeUp || infinity) && usePullup" :style="vScrollerUpStyle">
                <div class="v-scroller-text" v-show="infinity && status.upStatus !== 'loading'">{{status.pullupText}}</div>
                <div class="v-scroller-text" v-show="!infinity && status.upStatus === 'up'">{{status.pullupText}}</div>
                <spinner v-show="status.upStatus === 'loading'" type="ios"></spinner>
            </div>
        </div>
    </div>
</template>

<style lang="less" type="text/less">
    .v-scroller-content {
        height: 100%;
        overflow: auto;
        position: relative;
        -webkit-overflow-scrolling: touch;
        .v-scroller-box {
            position: relative;
            overflow: hidden;
            &.show {
                overflow: visible;
            }
            .v-scroller-body {
                position: relative;
                z-index: 1;
            }
        }
        .v-pull-down-loading {
            position: absolute;
            left: 50%;
            margin-left: -14px;
            top: 50%;
            margin-top: -14px;
        }
        .refresh-content {
            position: absolute;
            width: 100%;
            // height: 60px;
            // line-height: 60px;
            padding: 10px 0;
            top: -62px;
            text-align: center;
            z-index: 0;
            .refresh-content-loading {
                width: 60px;
                height: 60px;
                margin: 0 auto;
                position: relative;
            }
            > div {
                line-height: 0;
            }
            img {
                width: 28px;
                display: inline-block;
            }
            &.active img{
                animation: rotate-loading 1000ms infinite linear;
            }
        }
        .v-scroller-up-loading {
            position: absolute;
            height: 50px;
            width: 100%;
            bottom: -50px;
            display: flex;
            align-items: center;
            justify-content: center;
            overflow: hidden;
            .v-scroller-text {
                color: #808080;
                line-height: 0;
            }
        }
        @keyframes rotate-loading {
            100% {
                transform: rotate(360deg);
            }
        }
        &.infinity {
            .v-scroller-up-loading {
                position: static;
                text-align: center;
            }
            .v-scroller-box {
                overflow: visible;
            }
        }
    }
</style>
<script>
    import { Spinner } from 'vux';
    import PulldownLayer from './PulldownLayer';

    export default{
        props: {
            value: { // 上拉 下拉 加载状态
                type: Object,
                default: () => {
                },
            },
            usePullup: { // 是否启用上拉加载 默认不启用
                type: Boolean,
                default: false,
            },
            usePulldown: { // 是否启用下拉加载 默认不启用
                type: Boolean,
                default: false,
            },
            scrollOffset: {
                type: Number,
                default: 60,
            },
            infinity: {
                type: Boolean,
                default: true,
            },
            height: {
                type: [Number, String],
                default: '',
            },
            loadingType: {
                type: String,
                default: '',
            },
        },
        activated() {
            if (this.scrollTop && this.vScroller) {
                this.vScroller.scrollTop = this.scrollTop;
            }
            this.fixScrollTouch(500);
        },
        computed: {
            angle() {
                const deg = Math.max(0, Math.abs(Math.min(-this.offsetTop / 1.3, 100))) / 100 * 360;
                return deg > 180 ? 180 : deg;
            },
        },
        data() {
            return {
                status: this.value,
                offsetNo: 0.3, // 偏移系数
                offsetTop: 0, // 容器偏移量
                scrollTop: 0,
                scrollContentHeight: 0, // 滚动容器的高度
                touchStartOffset: 0, // 手指按下时在屏幕上的位置
                activeDown: false, // 激活下拉
                activeUp: false, // 激活上拉
                pullupEnable: this.usePullup, // 是否启用上拉
                pulldownEnable: this.usePulldown, // 是否启用下拉
                vScrollerDownStyle: {
                    transform: '',
                    transition: '',
                },
                vScrollerUpStyle: {
                    transform: '',
                    transition: '',
                },
            };
        },
        mounted() {
            this.$nextTick(() => {
                this.vScroller = this.$refs.vScroller;
                this.vScrollerBody = this.$refs.vScrollerBody; // 下拉时变化容器
                this.vScrollerContent = this.$refs.vScrollerContent; // 列表内容的容器

                this.vScroller.addEventListener('touchstart', this.touchStart);
                this.vScroller.addEventListener('touchmove', this.touchMove);
                this.vScroller.addEventListener('touchend', this.touchEnd);

                if (this.checkiOS()) { // iOS
                    const body = document.body;
                    body.addEventListener('touchmove', (e) => {
                        if (e.pageY <= 0 || e.pageY > document.body.clientHeight) {
                            const events = document.createEvent('touchEvent');
                            events.initEvent('touchend', true, true);
                            this.vScroller.dispatchEvent(events);
                        }
                    });
                }

                // 无限滚动
                if (this.infinity) {
                    let nowScrollTop = 0; // 记录当前滚动位置 用于判断向上向下
                    this.vScroller.onscroll = () => {
                        if (!this.pullupEnable) {
                            return;
                        }
                        const bodyHeight = this.vScroller.clientHeight; // 滚动body高度
                        const scrollHeight = this.vScroller.scrollHeight; // 滚动内容高度
                        const scrollTop = this.vScroller.scrollTop; // 滚动高度
                        if (nowScrollTop > scrollTop) {
                            nowScrollTop = scrollTop;
                            return;
                        }
                        nowScrollTop = scrollTop;
                        if (scrollHeight !== bodyHeight && scrollHeight - bodyHeight - scrollTop <= 50 && this.status.upStatus === 'default') {
                            this.status.upStatus = 'loading';
                            this.$emit('on-pullup-loading');
                        }
                    };
                }
            });
        },
        methods: {
            // 检查是否是iOS 设备
            checkiOS() {
                return /\(i[^;]+;( U;)? CPU.+Mac OS X/.test(navigator.userAgent);
            },
            // 容器偏移
            offsetContent(top) {
                this.vScrollerDownStyle.transform = `translate3d(0px, ${top}px, 0px)`;
                this.vScrollerDownStyle.top = `translate3d(0px, ${top}px, 0px)`;
                if (this.status.upStatus === 'up' || this.status.upStatus === 'loading' || this.infinity) {
                    this.vScrollerUpStyle.transform = `translate3d(0px, ${top}px, 0px)`;
                }
            },
            // 复位
            resetOffset(offset) {
                if (this.infinity && offset < 0) { // 如果是无限滚动 则复位的时候直接为0
                    offset = 0;
                }
                const t = 300;
                this.offsetContent(offset || 0);
                this.vScrollerDownStyle.transition = `transform ${t}ms ease-in-out`;
                this.vScrollerUpStyle.transition = `transform ${t}ms ease-in-out`;
                setTimeout(() => {
                    this.vScrollerDownStyle.transition = '';
                    this.vScrollerUpStyle.transition = '';
                    this.offsetTop = 0;
                }, t);
            },
            // 手指按下
            touchStart(e) {
                this.touchStartOffset = e.targetTouches[0].clientY; // 当前按下的位置
                // this.listContentHeight = this.vScrollerContent.clientHeight; // 列表容器的高度
                this.listContentHeight = this.vScroller.scrollHeight; // 列表容器的高度
                this.scrollContentHeight = this.vScroller.clientHeight; // 滚动容器的高度
            },
            // 手指拖动
            touchMove(e) {
                if (this.status.downStatus === 'loading' || this.status.upStatus === 'loading') {
                    e.preventDefault();
                    return;
                }
                this.offsetTop = e.targetTouches[0].clientY - this.touchStartOffset; // 大于0时下拉，小于0上拉
                this.offsetTop *= this.offsetNo;
                const scrollTop = this.vScroller.scrollTop; // 容器滚动的高度
                this.scrollTop = scrollTop;
                if (this.usePulldown && this.pulldownEnable && this.offsetTop > 0 && scrollTop === 0) { // 下拉，并且容器滚动到顶部的时候
                    e.preventDefault();
                    this.offsetContent(this.offsetTop);
                    this.$emit('on-scroll', 0 - this.offsetTop); // 将滚动高度实时传到父组件
                    this.status.downStatus = 'down';
                    this.activeDown = true;
                } else {
                    this.activeDown = false;
                }
                const reduce = this.listContentHeight - this.scrollContentHeight; // 兼容部分安卓对于scrollTop四舍五入
                if (this.offsetTop < 0 && (((scrollTop - 1 < reduce || reduce < scrollTop + 1) && scrollTop >= reduce - 1) || this.listContentHeight <= this.scrollContentHeight)) { // 上拉，并且容器滚动到底部的时候
                    e.preventDefault();
                    if (this.usePullup && this.pullupEnable) {
                        this.activeUp = true;
                        this.status.upStatus = 'up';
                        this.offsetContent(this.offsetTop);
                    }
                } else {
                    this.activeUp = false;
                }
            },
            // 手指松开
            touchEnd() {
                this.isTouch = false;
                if (!this.activeUp && !this.activeDown) {
                    // this.resetOffset();
                    return;
                }
                if (this.offsetTop > 0 && this.pulldownEnable && this.status.downStatus !== 'loading') { // 下拉达到临界点，激活刷新
                    if (this.offsetTop > this.scrollOffset && this.activeDown) {
                        this.resetOffset(this.scrollOffset);
                        this.status.downStatus = 'loading';
                        this.$emit('on-pulldown-loading');
                    } else {
                        this.status.downStatus = 'default';
                        this.resetOffset();
                    }
                }
                if (this.offsetTop < 0 && this.pullupEnable && this.status.upStatus !== 'loading') { // 上拉达到临界点，激活刷新
                    if (Math.abs(this.offsetTop) > this.scrollOffset && this.activeUp) {
                        this.resetOffset(-this.scrollOffset);
                        this.status.upStatus = 'loading';
                        this.$emit('on-pullup-loading');
                    } else {
                        this.status.upStatus = 'default';
                        this.resetOffset();
                    }
                }
            },
            //  下拉加载完成 容器复位，并且将下拉状态还原
            donePulldown() {
                this.resetOffset();
                this.status.downStatus = 'default';
                this.activeDown = false;
            },
            // 上拉完成，容器复位，关闭loading
            donePullup() {
                this.resetOffset();
                this.status.upStatus = 'default';
                this.activeUp = false;
                // TODO 解决iOS初始状态没有一屏数据，后续加载更多数据后不能向上滚动的bug
                if (this.vScroller.scrollHeight <= this.vScroller.clientHeight) {
                    this.fixScrollTouch(0);
                }
            },
            disablePullup() {
                this.pullupEnable = false;
            },
            enablePullup() {
                this.pullupEnable = true;
            },
            disablePulldown() {
                this.pulldownEnable = false;
            },
            enablePulldown() {
                this.pulldownEnable = true;
            },
            // 解决ios 偶尔无法滚动bug
            fixScrollTouch(time) {
                this.$nextTick(() => {
                    if (this.checkiOS()) {
                        this.vScroller.style['-webkit-overflow-scrolling'] = 'auto';
                        setTimeout(() => {
                            this.vScroller.style['-webkit-overflow-scrolling'] = 'touch';
                        }, time);
                    }
                });
            },
        },
        components: {
            Spinner,
            PulldownLayer,
        },
    };
</script>
