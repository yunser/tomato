<template>
    <ui-page :title="title || page.title" :page="page" ref="page">
        <div slot="drawer">
            <div class="header">
                <img class="logo" src="/static/img/todo.svg">
            </div>
            <ui-list @itemClick="toggle()">
                <!-- <ui-list-item title="待办" to="/">
                    <ui-icon slot="left" value="grade"/>
                </ui-list-item> -->
                <ui-list-item title="番茄钟" to="/tomato">
                    <ui-icon slot="left" value="grade"/>
                </ui-list-item>
                <ui-list-item title="番茄钟插件" to="/tomato/plugin">
                    <ui-icon slot="left" value="grade"/>
                </ui-list-item>
                <ui-list-item title="专注" to="/focus">
                    <ui-icon slot="left" value="grade"/>
                </ui-list-item>
                <ui-list-item title="专注插件" to="/focus/plugin">
                    <ui-icon slot="left" value="grade"/>
                </ui-list-item>
                <ui-list-item title="帮助" href="https://project.yunser.com/products/864d1ee065c311e9a7018922ed87a412" target="_blank">
                    <ui-icon slot="left" value="info"/>
                </ui-list-item>
            </ui-list>
        </div>
        <slot></slot>
    </ui-page>
</template>

<script>
    export default {
        data() {
            return {
                categories: [],
                dialog: false,
                categoryName: ''
            }
        },
        props: {
            title: {
                type: String,
                default: ''
            },
            name: {
                type: String,
                default: 'default'
            },
            page: {
                type: Object,
                default: function () {
                    return {
                        title: '云设'
                    }
                }
            },
            backable: {
                type: Boolean,
                default: false
            }
        },
        mounted() {
            this.init()
        },
        methods: {
            init() {
                this.categories = this.$storage.get('categories')
                if (!this.categories) {
                    this.categories = []
                }
            },
            save() {
                this.dialog = false

                this.categories.push({
                    id: '' + new Date().getTime(),
                    name: this.categoryName
                })
                this.$storage.set('categories', this.categories)
                this.categoryName = ''
            },
            openDialog () {
                this.dialog = true
            },
            close () {
                this.dialog = false
            },
            toggle(flag) {
                this.open = !this.open
                this.docked = !flag
            }
        }
    }
</script>

<style lang="scss" scoped>
    .header {
        padding: 40px 0;
        border: 1px solid rgba(0,0,0,.12);
    }
    .logo {
        display: block;
        width: 80px;
        margin: 0 auto;
    }
    .version {
        position: absolute;
        bottom: 16px;
        left: 16px;
    }
</style>
