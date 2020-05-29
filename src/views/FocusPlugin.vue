<template>
    <div>
        <div :class="['box', isFocus ? 'full' : '']">
            <div class="state" v-if="isFocus">专注中...</div>
            <div class="state" v-if="!isFocus">休息中...</div>
            <!-- <div class="result" v-if="result">你专注了 {{ result }} 分钟</div> -->
            <div class="btns">
                <ui-icon-button class="btn" icon="play_arrow"  @click="start" v-if="!isFocus" />
                <ui-icon-button class="btn" icon="close"  @click="finish" v-if="isFocus" />
            </div>
        </div>
    </div>
</template>

<script>
    /* eslint-disable */
    export default {
        data () {
            return {
                isFocus: false,
                result: '',
                page: {
                    menu: [
                        {
                            type: 'icon',
                            icon: 'help',
                            href: 'https://project.yunser.com/products/86053a007ee911e9b0ea55c98e108afb',
                            target: '_blank',
                            title: '帮助'
                        }
                    ]
                }
            }
        },
        mounted() {
            document.title = '专注（插件版）'
        },
        destroyed() {
        },
        methods: {
            start() {
                this.startTime = new Date().getTime()
                this.isFocus = true
                this.result = ''
            },
            finish() {
                this.isFocus = false
                this.result = Math.ceil((new Date().getTime() - this.startTime) / (60 * 1000))
            },
        }
    }
</script>

<style lang="scss" scoped>
.box {
    height: 100vh;
    color: #fff;
    background: #000;
    padding: 32px;
    line-height: 1;
    // padding-top: 160px;
    // text-align: center;
    &.full {
        // position: fixed;
        // top: 0;
        // left: 0;
        // right: 0;
        // bottom: 0;
        // z-index: 100000;
        // background: #fff;
    }
}
.container {
}
.state {
    margin-bottom: 16px;
    font-size: 56px;
    font-weight: bold;
}
.result {
    margin-bottom: 16px;
    font-size: 16px;
    font-weight: bold;
}
.btns {
    margin-bottom: 16px;
    text-align: center;
    display: flex;
    align-items: center;
    // justify-content: center;
    .btn {
        margin-right: 16px;
        display: flex;
        align-self: start;
        justify-content: center;
        color: rgba(255, 255, 255, .5);
        border: 2px solid rgba(255, 255, 255, .12);
    }
}
</style>
