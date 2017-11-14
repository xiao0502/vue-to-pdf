<template>
    <div ref="wrapper">
        <slot></slot>
        <slot name="pulldown"
              :pullDownRefresh="pullDownRefresh"
              :pullDownStyle="pullDownStyle"
              :beforePullDown="beforePullDown"
              :isPullingDown="isPullingDown"
              :bubbleY="bubbleY">
            <div ref="pulldown" class="pulldown-wrapper" :style="pullDownStyle" v-show="initPullDownRefresh">
                <div class="before-trigger" v-if="beforePullDown">
                    <bubble :y="bubbleY"></bubble>
                </div>
                <div class="after-trigger" v-else>
                    <div v-if="isPullingDown" class="loading">
                        <loading></loading>
                    </div>
                    <div v-else><span>{{refreshTxt}}</span></div>
                </div>
            </div>
        </slot>
    </div>
</template>

<script type="es6">
    import BScroll from 'better-scroll'
    import Loading from './loading.vue'
    import Bubble from './bubble.vue'

    export default {
        props: {
            probeType: {
                type: Number,
                default: 3
            },
            click: {
                type: Boolean,
                default: true
            },
            listenScroll: {
                type: Boolean,
                default: false
            },
            data: {
                type: Array,
                default: null
            },
            pullup: {
                type: Boolean,
                default: false
            },
            pullUpLoad: {
                type: null,
                default: false
            },
            beforeScroll: {
                type: Boolean,
                default: false
            },
            scrollDisable: {
                type: Boolean,
                default: false
            },
            pullDownRefresh: {
                type: null,
                default: false
            },
            initPullDownRefresh: {
                type: Boolean,
                default: false
            },
            freeScroll: {
                type: Boolean,
                default: false
            },
            scrollX: {
                type: Boolean,
                default: false
            }
        },
        created() {
            this.pullDownInitTop = -50
        },
        components: {
            Loading, Bubble
        },
        data() {
            return {
                isPullingDown: false,
                isRebounding: false,
                beforePullDown: true,
                isPullUpLoad: false,
                pullDownStyle: '',
                bubbleY: 0
            }
        },
        mounted() {
            setTimeout(() => {
                this._initScroll()
            }, 20)
        },
        computed: {
            refreshTxt() {
                return this.pullDownRefresh && this.pullDownRefresh.txt || '刷新完成！'
            }
        },
        methods: {
            _initScroll() {
                if (!this.$refs.wrapper) {
                    return
                }

                this.scroll = new BScroll(this.$refs.wrapper, {
                    probeType: this.probeType,
                    click: this.click,
                    pullDownRefresh: this.pullDownRefresh,
                    pullUpLoad: this.pullUpLoad,
                    freeScroll: this.freeScroll,
                    scrollX: this.scrollX
                })

                if (this.listenScroll) {
                    this.scroll.on('scroll', (pos) => {
                        if (this.beforePullDown) {
                            this.bubbleY = Math.max(0, pos.y + this.pullDownInitTop)
                            this.pullDownStyle = `top:${Math.min(pos.y + this.pullDownInitTop, 10)}px`
                        } else {
                            this.bubbleY = 0
                        }
                        if (this.isRebounding) {
                            this.pullDownStyle = `top:${10 - (this.pullDownRefresh.stop - pos.y)}px`
                        }
                        this.$emit('scroll',pos)
                    })
                }

                if (this.pullDownRefresh) {
                    this._initPullDownRefresh()
                }
                if (this.pullUpLoad) {
                    this._initPullUpLoad()
                }

            },
            disable() {
                this.scroll && this.scroll.disable()
            },
            enable() {
                this.scroll && this.scroll.enable()
            },
            refresh() {
                this.scroll && this.scroll.refresh()
            },
            scrollTo() {
                this.scroll && this.scroll.scrollTo.apply(this.scroll, arguments)
            },
            scrollToElement() {
                this.scroll && this.scroll.scrollToElement.apply(this.scroll, arguments)
            },
            // 重新计算高度
            forceUpdate(dirty) {
                if (this.pullDownRefresh && this.isPullingDown) {
                    this.isPullingDown = false
                    this._reboundPullDown().then(() => {
                        this._afterPullDown()
                    })
                } else if (this.pullUpLoad && this.isPullUpLoad) {
                    this.isPullUpLoad = false
                    this.scroll.finishPullUp()
                    this.pullUpDirty = dirty
                    this.refresh()
                } else {
                    this.refresh()
                }
            },
            // 下拉刷新
            _initPullDownRefresh() {
                this.scroll.on('pullingDown', () => {
                    this.beforePullDown = false
                    this.isPullingDown = true
                    this.$emit('pullingDown')
                })
                this.scroll.on('scroll', (pos) => {
                    if (this.beforePullDown) {
                        this.bubbleY = Math.max(0, pos.y + this.pullDownInitTop)
                        this.pullDownStyle = `top:${Math.min(pos.y + this.pullDownInitTop, 0)}px`
                    } else {
                        this.bubbleY = 0
                    }
                    if (this.isRebounding) {
                        this.pullDownStyle = `top:${10 - (this.pullDownRefresh.stop - pos.y)}px`
                    }
                    this.$emit('scroll', pos)
                })
            },
            _reboundPullDown() {
                const {stopTime = 600} = this.pullDownRefresh
                return new Promise((resolve) => {
                    setTimeout(() => {
                        this.isRebounding = true
                        this.scroll.finishPullDown()
                        resolve()
                    }, stopTime)
                })
            },
            // 上拉加载
            _initPullUpLoad() {
                this.scroll.on('pullingUp', () => {
                    this.isPullUpLoad = true
                    this.$emit('pullingUp')
                })
            },
            // 上拉加载完后
            _afterPullDown() {
                setTimeout(() => {
                    this.pullDownStyle = `top:${this.pullDownInitTop}px`
                    this.beforePullDown = true
                    this.isRebounding = false
                    this.refresh()
                }, this.scroll.options.bounceTime)
            }
        },
        watch: {
            data() {
                setTimeout(() => {
                    this.forceUpdate(true)
                }, this.refreshDelay)
//                setTimeout(() => {
//                    this.refresh()
//                }, this.refreshDelay)
            },
            scrollDisable(newBoolean) {
                if (newBoolean) {
                    this.disable();
                } else {
                    this.enable();
                }
            }
        }
    }
</script>

<style lang="less" rel="stylesheet/less">
    .list-wrapper {
        position: absolute;
        left: 0;
        top: 0;
        right: 0;
        bottom: 0;
        overflow: hidden;
        background: #fff;
        .scroll-content {
            position: relative;
            z-index: 1;
        }

        .list-content {
            position: relative;
            z-index: 10;
            background: #fff;
            .list-item {
                height: 60px;
                line-height: 60px;
                font-size: 18px;
                padding-left: 20px;
                border-bottom: 1px solid #e5e5e5;
            }

        }
    }

    .pulldown-wrapper {
        position: absolute;
        width: 100%;
        left: 0;
        display: flex;
        justify-content: center;
        align-items: center;
        transition: all;
        .after-trigger {
            line-height: 50px;
            font-size: 15px;
        }
    }

    .pullup-wrapper {
        width: 100%;
        display: flex;
        justify-content: center;
        align-items: center;
        padding: 16px 0;
    }
</style>
