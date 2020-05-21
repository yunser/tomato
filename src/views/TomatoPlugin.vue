<template>
    <div class="box">
        <div class="time">{{ timeStr }}</div>
        <div class="btns">
            <ui-icon-button class="btn" icon="play_arrow"  @click="startRest(5)" v-if="!isRuning && isWorkTime" />
            <ui-icon-button class="btn" icon="play_arrow"  @click="startCount" v-if="!isRuning && !isWorkTime" />
            <ui-icon-button class="btn" icon="close"  @click="reset" v-if="isRuning" />
            <!-- <ui-raised-button class="btn" label="取消" @click="reset" v-if="isRuning" /> -->
        </div>
        <div class="label label-success" v-if="isRuning && isWorkTime">工作</div>
        <div class="label label-warning" v-if="isRuning && !isWorkTime">休息</div>
        <!-- 声音提醒 -->
        <audio id="tip">
            <source src="/static/res/tip.mp3">
            <source src="WhiteChristmas.ogg">
        </audio>
        <ui-drawer class="setting-box" right :open="settingVisible" :docked="false" @close="toggleSetting()">
            <ui-appbar title="设置">
                <ui-icon-button icon="close" slot="left" @click="toggleSetting" />
                <ui-icon-button icon="check" slot="right" @click="finish" />
            </ui-appbar>
            <div class="body">
                <ui-text-field v-model="formData.minute" label="工作时间（分钟）" />
            </div>
        </ui-drawer>
    </div>
</template>

<script>
    /* eslint-disable */
    export default {
        data () {
            return {
                showTip: false,
                timeStr: '00:00',
                second: 0,
                formData: {
                    minute: 25,
                    text: '' // 用户输入的待办事项
                },
                timerId: null,
                afterWork: false,
                isWorkTime: false, // 是否工作时间
                settingVisible: false,
                isRuning: false,
                progress: 0,
                page: {
                    menu: [
                        {
                            type: 'icon',
                            icon: 'settings',
                            click: this.toggleSetting,
                            title: '设置'
                        },
                        {
                            type: 'icon',
                            icon: 'help',
                            href: 'https://project.yunser.com/products/864d1ee065c311e9a7018922ed87a412',
                            target: '_blank',
                            title: '帮助'
                        }
                    ]
                }
            }
        },
        mounted() {
//            this.init()
            if (this.$route.query.start) {
                this.startCount()
            }
            this.formData.minute = this.$storage.get('workTime', 25)

            this.isRuning = this.$storage.get('isRuning', false)
            console.log('this.isRuning', this.isRuning)
            if (this.isRuning) {
                this.startTime = this.$storage.get('startTime', 0)
                this.afterWork = this.$storage.get('afterWork', false)
                this.totalTime = this.$storage.get('totalTime', 0)
                this.isWorkTime = this.$storage.get('isWorkTime', false)
                console.log('正字允许')
                this.startTimer()
            }
        },
        destroyed() {
            this.timerId && clearInterval(this.timerId)
        },
        methods: {
            reset() {
                this.timerId && clearInterval(this.timerId)
                this.afterWork = false
                this.isWorkTime = false
                this.isRuning = false
                this.progress = 0
                this.timeStr = '00:00'
                this.$storage.set('isRuning', false)
            },
            toggleSetting() {
                this.settingVisible = !this.settingVisible
            },
            finish() {
                this.$storage.set('workTime', this.formData.minute)
                this.settingVisible = false
            },
            // 点击开始计时按钮事件
            startCount() {
                // 如果输入的分钟不正确，默认25分钟
                if (isNaN(this.formData.minute) || this.formData.minute <= 0 ||
                        this.formData.minute > 24 * 60) {
                    this.formData.minute = 25
                }
                this.isWorkTime = true
                this.$storage.set('isWorkTime', true)
                this.countDown(this.formData.minute * 60)
                // this.countDown(1 * 60)
            },
            startRest(minute) {
                this.isWorkTime = false
                this.$storage.set('isWorkTime', false)
                this.countDown(minute * 60)
            },
            // 初始化
            init() {
                // 温馨提示
                let date = new Date()
                if (date.getHours() > 23 || date.getHours() < 4) {
                    this.showTip = true
                }
            },
            tip() {
                document.getElementById('tip').play()
            },
            startTimer() {
                let countTime = () => {
                    // this.time--
                    this.time = this.totalTime - (new Date().getTime() - this.startTime) / 1000 // 秒
                    if (this.time <= 0) {
                        if (this.isWorkTime) {
                            // this.time = 5 * 60
                            this.afterWork = true
                        } else {
                            // this.time = workTime + 1
                        }
                        // this.isWorkTime = !this.isWorkTime
                        
                        this.isRuning = false
                        this.$storage.set('isRuning', false)
                        this.tip()
                        clearInterval(this.timerId)
                        // alert('时间到了')
                        this.$message({
                            type: 'info',
                            text: '时间到了'
                        })
                        return
                    }

                    console.log('progress', this.time)
                    this.progress = ((new Date().getTime() - this.startTime) / 1000) / this.totalTime
    
                    let minute = Math.floor(this.time / 60)
                    let second = Math.floor(this.time % 60)
                    minute = (minute < 10) ? ('0' + minute) : minute
                    second = (second < 10) ? ('0' + second) : second
    
                    this.timeStr = minute + ':' + second
                    document.title = this.timeStr
                }
                countTime()
                this.timerId = setInterval(countTime, 1000)
            },
            /**
             * 倒计时
             * @param time 时间（秒）
             */
            countDown(time) {
                clearInterval(this.timerId)
                this.startTime = new Date().getTime()
                this.$storage.set('startTime', this.startTime)
                this.$storage.set('isRuning', true)
                this.$storage.set('afterWork', true)
                this.isRuning = true
                this.afterWork = false
                let workTime = time
                this.totalTime = time
                this.$storage.set('totalTime', time)
                // this.time = time + 1
                this.startTimer()
            }
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
    .time {
        font-size: 64px;
        margin-bottom: 16px;
    }
    .btns {
        margin-bottom: 16px;
        text-align: center;
        display: flex;
        align-items: center;
        .btn {
            margin-right: 16px;
            display: flex;
            align-self: start;
            justify-content: center;
            color: rgba(255, 255, 255, .5);
            border: 2px solid rgba(255, 255, 255, .12);
        }
    }
}

.label {
    position: fixed;
    right: 16px;
    top: 16px;
    display: inline;
    padding: .2em .6em .3em;
    font-size: 75%;
    font-weight: 700;
    line-height: 1;
    color: #fff;
    text-align: center;
    white-space: nowrap;
    vertical-align: baseline;
    border-radius: .25em;
    border: 1px solid #fff;
}
.label-warning {
    background-color: #f7a54a;
}
.label-success {
    background-color: #5cb85c;
}

.progressBox {
    height: 4px;
    width: 48px;
    background: #ccc;
    margin-bottom: 24px;
    .progressBar {
        height: 4px;
        width: 50%;
        background: #4285f4;
    }
}
.setting-box {
    width: 320px;
    max-width: 100%;
    .body {
        padding: 16px;
    }
}

</style>
