<template>
    <div id="app">
        <div class="page-pdf" >
            <scroll @scroll="handleScroll" :freeScroll="freeScroll" :scrollX="scrollX" ref="scroll" class="scroll-content">
                <div class="scroll-content-info" :style="handleChangeStyle">
                    <canvas id="the-canvas"></canvas>
                </div>
                <loading :text="text" v-show="isLoading"></loading>
            </scroll>
            <div class="showPage">
                <span class="currentPage">{{pageNum}}</span>
                <span class="pageTotal">/{{pageTotal}}</span>
            </div>
            <div class="wrapper-todo">
                <div class="btn" @click="handlePrev">上一页<i></i></div>
                <div class="btn" @click="handleNext">下一页<i></i></div> &nbsp;
                <div class="btn" @click="handleAddscale">放大<i></i></div>
                <div class="btn" @click="handleMinus">缩小<i></i></div>
            </div>
        </div>
    </div>
</template>
<script type="es6">
    import Scroll from '@/components/scroll'
    import Loading from '@/components/loading'

    export default {
        created() {
            this.$nextTick(() => {
                this.handleInitPdf(this.pageNum);
            })
        },
        data() {
            return {
                url: 'https://mozilla.github.io/pdf.js/web/compressed.tracemonkey-pldi-09.pdf',
                pdfDoc: {},
                pageNum: 1,
                pageTotal: 1,
                scale: 1,
                maxScale: 3,
                minScale: 0,
                initFlag: true,
                scrollX: true,
                freeScroll: true,
                isLoading: true,
                text: ''
            }
        },
        methods: {
            handleScroll(pos) {
                console.log(pos);
            },
            /*
            *   新增canvas
            * */
            createCanvas() {
                let canvas = document.createElement('canvas');
                canvas.setAttribute('id', 'the-canvas')
                document.querySelector('.scroll-content-info').appendChild(canvas)
            },
            /*
            *   删除canvas
            * */
            deleteCanvas() {
                document.querySelector('.scroll-content-info').removeChild(document.querySelector('#the-canvas'))
            },

            /*
            *   初始化PDF
            * */
            handleInitPdf(num) {
                let pdfJS = require('pdfjs-dist').PDFJS;
                let vm = this;
                pdfJS.workerSrc = require('pdfjs-dist/build/pdf.worker.min');
                var canvas = document.getElementById('the-canvas');
                pdfJS.getDocument(this.url).then(function getPdfHelloWorld(pdf) {
                    pdf.getPage(num).then(function getPageHelloWorld(page) {
                        if (vm.initFlag) {
                            vm.scale = document.body.getBoundingClientRect().width / page.view[2];
                            vm.minScale = vm.scale;
                        }
                        vm.initFlag = false;
                        var viewport = page.getViewport(vm.scale);
                        var context = canvas.getContext('2d');
                        canvas.height = viewport.height;
                        canvas.width = viewport.width;
                        var renderContext = {
                            canvasContext: context,
                            viewport: viewport
                        };
                        page.render(renderContext);
                        document.querySelector('.scroll-content-info').style.width = document.querySelector('#the-canvas').getBoundingClientRect().width + 'px';
                        vm.$refs.scroll.refresh();
                        vm.isLoading = false;
                    });
                });
                pdfJS.getDocument(this.url).then(function (pdfDoc_) { //初始化pdf
                    vm.pdfDoc = pdfDoc_;
                    vm.pageTotal = vm.pdfDoc.numPages;
                }).catch(function (err) {
                    if (err) {
                        console.log(err)
                        vm.throwerr(vm.pdfurl)
                    }
                })
            },
            /*
            *   放大
            * */
            handleAddscale() {
                if (this.scale >= this.maxScale) {
                    return
                }
                this.deleteCanvas();
                this.createCanvas();
                this.scale += 0.1;
                this.isLoading = true;
                this.handleQueueRenderPage(this.pageNum)
            },
            /*
            *   缩小
            * */
            handleMinus() {
                if (this.scale <= this.minScale) {
                    return
                }
                this.deleteCanvas();
                this.createCanvas();
                this.scale -= 0.1;
                this.isLoading = true;
                this.handleQueueRenderPage(this.pageNum)
            },
            /*
            *   上一页
            * */
            handlePrev() {
                let vm = this
                if (vm.pageNum <= 1) {
                    return;
                }
                vm.pageNum--;
                vm.$refs.scroll.scrollTo(0,0);
                vm.handleQueueRenderPage(vm.pageNum);
            },
            /*
            *   下一页
            * */
            handleNext() {
                let vm = this
                if (vm.pageNum >= vm.pageTotal) {
                    return;
                }
                vm.pageNum++;
                vm.$refs.scroll.scrollTo(0,0);
                vm.handleQueueRenderPage(vm.pageNum);
            },
            /*
            *   重新渲染
            * */
            handleQueueRenderPage(num) {
                this.handleInitPdf(num);
            }
        },
        components: {
            Scroll, Loading
        },
        computed: {
            handleChangeStyle() {

            }
        }
    };
</script>
<style lang="less" rel="stylesheet/less">
    @import "./common/less/define";
    @import "./common/less/mixin";

    .page-pdf {
        position: relative;
        width: 100%;
        height: 100vh;
        overflow: hidden;
        .scroll-content {
            position: absolute;
            width: 100%;
            top: 0;
            bottom: 50px;
            left: 0;
        }
        .showPage {
            position: absolute;
            right: 10px;
            bottom: 60px;
            width: 40px;
            height: 40px;
            border-radius: 50%;
            background-color: rgba(0,0,0,0.3);
            color: #fff;
            text-align: center;
            line-height: 40px;
            font-size: 12px;
            .currentPage {
                font-size: 17px;
            }
        }
        .wrapper-todo {
            background-color: #fff;
            z-index: 10;
            position: absolute;
            left: 0;
            bottom: 0;
            width: 100%;
            height: 50px;
            display: flex;
            box-shadow: 0px -2px 18px 2px rgba(0, 0, 0, 0.1);
            .btn {
                line-height: 80px;
                display: block;
                flex: 1;
                text-align: center;
                font-size: 11px;
                color: #666;
                position: relative;
                i {
                    width: 25px;
                    height: 25px;
                    display: block;
                    border-radius: 50%;
                    background-color: #FFB700;
                    position: absolute;
                    left: 50%;
                    margin-left: -12px;
                    top: 4px;
                }
                &:nth-child(1) {
                    i:before {
                        content: '';
                        width: 25px;
                        height: 25px;
                        display: block;
                        .bg-image('icon-arrow-prev');
                        background-size: 7px;
                    }
                }
                &:nth-child(2) {
                    i:before {
                        content: '';
                        width: 25px;
                        height: 25px;
                        display: block;
                        .bg-image('icon-arrow-next');
                        background-size: 7px;
                    }
                }
                &:nth-child(3) {
                    i:before {
                        content: '';
                        width: 25px;
                        height: 25px;
                        display: block;
                        .bg-image('icon-arrow-bigger');
                        background-size: 11px;
                    }
                }
                &:nth-child(4) {
                    i:before {
                        content: '';
                        width: 25px;
                        height: 25px;
                        display: block;
                        .bg-image('icon-arrow-smaller');
                        background-size: 11px;
                    }
                }
            }
        }
    }
</style>
