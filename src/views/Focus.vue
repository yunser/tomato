<template>
    <my-page title="专注" :page="page">
        <div class="common-container container">
            <div :class="['box', isFocus ? 'full' : '']">
                <div class="state" v-if="isFocus">专注中...</div>
                <div class="state" v-if="!isFocus">休息中...</div>
                <div class="result" v-if="result">你专注了 {{ result }} 分钟</div>
                <div class="btns">
                    <ui-raised-button class="btn" primary label="开始专注" @click="start" v-if="!isFocus" />
                    <ui-raised-button class="btn" secondary label="结束" @click="finish" v-if="isFocus" />
                </div>
            </div>
        </div>
    </my-page>
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
    padding-top: 160px;
    text-align: center;
    &.full {
        position: fixed;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        z-index: 100000;
        background: #fff;
    }
}
.container {
}
.state {
    margin-bottom: 80px;
    font-size: 32px;
    font-weight: bold;
}
.result {
    margin-bottom: 80px;
    font-size: 32px;
    font-weight: bold;
}
</style>
