<template>
    <my-page title="番茄钟" :page="page">
        <div class="common-container center container">
            <div class="tip" v-if="showTip">夜深了，注意休息</div>
            <!-- <div class="input-minute input-group">
                        <input class="form-control" placeholder="工作时间" v-model="formData.minute">
                        <div class="input-group-addon">分钟</div>
                </div> -->
            <!-- {{ progress }} -->
            <ui-float-button
                @click="toggleCount(isRunning)"
                class="demo-float-button tomato-box"
            >
                <div class="statu">
                    {{ isRunning ? (isWorkTime ? "工作中" : "点击延长休息时间") : "" }}
                </div>
                <div class="tt-countdown">{{ timeStr }}</div>
                <svg
                    style="z-index: -1"
                    class="circle-progress"
                    width="250"
                    height="250"
                    xmlns="http://www.w3.org/2000/svg"
                    viewBox="15 15 70 70"
                >
                    <circle
                        cx="50"
                        cy="50"
                        r="33.5"
                        stroke="#fff"
                        stroke-width="2.5"
                        fill="transparent"
                    />
                    <path
                        v-bind:d="path"
                        stroke="#4285f4"
                        fill="transparent"
                        stroke-width="2.8"
                    ></path>
                </svg>
            </ui-float-button>
            <br /><br />
            <ui-raised-button
                class="btn"
                primary
                label="跳过休息"
                @click="startCount"
                v-if="isRunning && !isWorkTime"
            />
            <ui-raised-button
                class="btn"
                label="重置"
                @click="reset"
                v-if="isRunning && isWorkTime"
            />
        </div>
        <!-- 声音提醒 -->
        <audio id="tip">
            <source src="/static/res/tip.mp3" />
            <source src="WhiteChristmas.ogg" />
        </audio>
        <ui-drawer
            class="setting-box"
            right
            :open="settingVisible"
            :docked="false"
            @close="toggleSetting()"
        >
            <ui-appbar title="设置">
                <ui-icon-button
                    icon="close"
                    slot="left"
                    @click="toggleSetting"
                />
                <ui-icon-button icon="check" slot="right" @click="finish" />
            </ui-appbar>
            <div class="body">
                <ui-text-field
                    v-model="formData.minute"
                    label="工作时间（分钟）"
                />
            </div>
        </ui-drawer>
    </my-page>
</template>

<script>
// 角度转弧度
function d2a(n) {
    return (n * Math.PI) / 180;
}
/* eslint-disable */
export default {
    data() {
        return {
            showTip: new Date().getHours() >= 23,
            timeStr: "00:00",
            second: 0,
            formData: {
                minute: 25,
                text: "" // 用户输入的待办事项
            },
            timerId: null,
            afterWork: false,
            isWorkTime: false, // 是否工作时间
            settingVisible: false,
            isRunning: false,
            progress: 0,
            path: "", // svg绘制路径
            page: {
                menu: [
                    {
                        type: "icon",
                        icon: "settings",
                        click: this.toggleSetting,
                        title: "设置"
                    },
                    {
                        type: "icon",
                        icon: "help",
                        href:
                            "https://project.yunser.com/products/864d1ee065c311e9a7018922ed87a412",
                        target: "_blank",
                        title: "帮助"
                    }
                ]
            }
        };
    },
    watch: {
        progress: function(newProgress, oldProgress) {
            const r = 33.5;
            const ang1 = newProgress * 360;
            const ang2 = 360;
            this.path = `M50 50 
                        L${50 + Math.sin(d2a(ang1)) * r} ${50 - Math.cos(d2a(ang1)) * r}
                        A${r} ${r} 0 ${ang2 - ang1 >= 180 ? 1 : 0} 1 ${50 +
                        Math.sin(d2a(ang2)) * r} ${50 - Math.cos(d2a(ang2)) * r} Z`;
        }
    },
    mounted() {
        //            this.init()
        if (this.$route.query.start) {
            this.startCount();
        }
        this.formData.minute = this.$storage.get("workTime", 25);

        this.isRunning = this.$storage.get("isRunning", false);
        console.log("this.isRunning", this.isRunning);
        if (this.isRunning) {
            this.startTime = this.$storage.get("startTime", 0);
            this.afterWork = this.$storage.get("afterWork", false);
            this.totalTime = this.$storage.get("totalTime", 0);
            this.isWorkTime = this.$storage.get("isWorkTime", false);
            console.log("正字允许");
            this.startTimer();
        }
    },
    destroyed() {
        this.timerId && clearInterval(this.timerId);
    },
    methods: {
        /**
         * 切换倒计时状态
         * @param { boolean } isRunning 是否在运行
         */
        toggleCount(isRunning, isWorkTime) {
            !isRunning && this.startCount();
            isRunning && !isWorkTime && this.delayRest(5)
        },
        /**
         * 延长休息时间
         * @param { number } minute
         */
        delayRest(minute){
            this.totalTime = this.totalTime + minute * 60;
        },
        reset() {
            this.timerId && clearInterval(this.timerId);
            this.afterWork = false;
            this.isWorkTime = false;
            this.isRunning = false;
            this.progress = 0;
            this.timeStr = "00:00";
            this.$storage.set("isRunning", false);
            document.title = '番茄钟'
        },
        toggleSetting() {
            this.settingVisible = !this.settingVisible;
        },
        finish() {
            this.$storage.set("workTime", this.formData.minute);
            this.settingVisible = false;
        },
        // 点击开始计时按钮事件
        startCount() {
            // 如果输入的分钟不正确，默认25分钟
            if (
                isNaN(this.formData.minute) ||
                this.formData.minute <= 0 ||
                this.formData.minute > 24 * 60
            ) {
                this.formData.minute = 25;
            }
            this.isWorkTime = true;
            this.$storage.set("isWorkTime", true);
            this.countDown(this.formData.minute * 60);
            // this.countDown(1 * 60)
        },
        startRest(minute) {
            this.isWorkTime = false;
            this.$storage.set("isWorkTime", false);
            this.countDown(minute * 60);
        },
        // 初始化
        init() {
            // 温馨提示
            let date = new Date();
            if (date.getHours() > 23 || date.getHours() < 4) {
                this.showTip = true;
            }
        },
        tip() {
            document.getElementById("tip").play();
        },
        startTimer() {
            let countTime = () => {
                // this.time--
                this.time =
                    this.totalTime -
                    (new Date().getTime() - this.startTime) / 1000; // 秒
                if (this.time <= 0) {
                    if (this.isWorkTime) {
                        // this.time = 5 * 60
                        this.afterWork = true;
                    } else {
                        // this.time = workTime + 1
                    }
                    // this.isWorkTime = !this.isWorkTime

                    this.isRunning = false;
                    this.$storage.set("isRunning", false);
                    this.tip();
                    clearInterval(this.timerId);
                    if(this.isWorkTime){
                        this.startRest(0.5)
                    }else{
                        this.startCount()
                    }
                    // alert('时间到了')
                    this.$message({
                        type: "info",
                        text: "时间到了"
                    });
                    return;
                }

                // console.log("progress", this.time);
                this.progress =
                    (new Date().getTime() - this.startTime) /
                    1000 /
                    this.totalTime;

                let minute = Math.floor(this.time / 60);
                let second = Math.floor(this.time % 60);
                minute = minute < 10 ? "0" + minute : minute;
                second = second < 10 ? "0" + second : second;

                this.timeStr = minute + ":" + second;
                document.title = `${this.timeStr} - 番茄钟`;
            };
            countTime();
            this.timerId = setInterval(countTime, 1000);
        },
        /**
         * 倒计时
         * @param time 时间（秒）
         */
        countDown(time) {
            clearInterval(this.timerId);
            this.startTime = new Date().getTime();
            this.$storage.set("startTime", this.startTime);
            this.$storage.set("isRunning", true);
            this.$storage.set("afterWork", true);
            this.isRunning = true;
            this.afterWork = false;
            let workTime = time;
            this.totalTime = time;
            this.$storage.set("totalTime", time);
            // this.time = time + 1
            this.startTimer();
        }
    }
};
</script>

<style lang="scss" scoped>
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
.container {
    width: 400px;
}
.btn {
    margin-right: 8px;
}
.label {
    display: inline;
    padding: 0.2em 0.6em 0.3em;
    font-size: 75%;
    font-weight: 700;
    line-height: 1;
    color: #fff;
    text-align: center;
    white-space: nowrap;
    vertical-align: baseline;
    border-radius: 0.25em;
}
.label-warning {
    background-color: #f7a54a;
}
.label-success {
    background-color: #5cb85c;
}
.circle-progress {
    margin: 0 auto;
}
</style>
