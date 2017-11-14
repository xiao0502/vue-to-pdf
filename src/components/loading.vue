<template>
    <div class="mf-loading-container">
        <img :class="text?'':'center'" src="./loading.gif">{{text}}
    </div>
</template>
<script type="text/ecmascript-6">
    const COMPONENT_NAME = 'loading';
    export default {
        name: COMPONENT_NAME,
        props: {
            text: {
                type: String,
                default: '正在加载中...'
            }
        }
    }
</script>
<style lang="less" rel="stylesheet/less">
    .mf-loading-container {
        text-align: center;
        height: 50px;
        line-height: 50px;
        color: #a1a1a1;
        img {
            width: 20px;
            height: 20px;
            display: inline-block;
            position: relative;
            top: 4px;
            margin-right: 8px;
        }
        .center {
            margin-right: 0;
        }
    }

</style>
