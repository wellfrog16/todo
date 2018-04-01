<template>
    <div id="app">
        <div class="container">
            <header><input type="text" placeholder="请输入待办事项" v-model="todo" @keyup.enter="save()"></header>
            <section>
                <div>
                <transition-group name="list-complete" tag="ul">
                    <li v-for="item in list" v-bind:key="item.id" class="list-complete-item" :class="{finished: item.finished}">
                        <span class="finish">完成</span>
                        <div>{{ item.todo }}</div>
                        <span class="delete">删除</span>
                    </li>
                </transition-group>
                </div>
            </section>
        </div>
    </div>
</template>

<script>
import $ from 'jquery';
import Hammer from 'hammer';
import IScroll from 'iscroll';
import Shake from 'shake.js';

let myScroll = null;

export default {
    name: 'app',
    data() {
        return {
            currentEl: null,
            todo: '',
            list: []
        };
    },
    created() {
        this.readStorage();
    },
    mounted() {
        // 禁止拉动
        $('body').on('touchmove', e => e.preventDefault());

        // 事件绑定
        $('li>div').each((index, el) => this.addHandle(el));

        // 滚动条
        myScroll = new IScroll('section', {scrollbars: false});

        // 抖动事件
        new Shake({
            threshold: 15, // optional shake strength threshold
            timeout: 1000 // optional, determines the frequency of event generation
        }).start();
        window.addEventListener('shake', () => this.removeFinished(), false);
    },
    methods: {
        save() {
            if (!this.todo) { return; }
            this.list.unshift({ todo: this.todo, finished: false, id: new Date().getTime() });
            this.saveStorage();
            this.todo = '';

            this.$nextTick(() => this.addHandle($('li:first-child>div')[0]));
        },
        remove(index) {
            const item = this.list[index];
            this.list.splice(this.list.indexOf(item), 1);
            this.saveStorage();
        },
        removeFinished() {
            for (const item of this.list) {
                if (item.finished) {
                    this.list.splice(this.list.indexOf(item), this.list.length);
                    this.saveStorage();
                    return;
                }
            }
        },
        finish(index) {
            this.list[index].finished = true;
            this.saveStorage();
        },
        addHandle(el) {
            el.status = 0;    // 初始状态
            el.finished = false;

            let mc = new Hammer.Manager($(el).parent().find('span.delete')[0]);
            let Tap = new Hammer.Tap();

            mc.add(Tap);
            mc.on('tap', (e) => this.remove($(el).parent().index()));

            mc = new Hammer.Manager(el);
            let Pan = new Hammer.Pan({
                threshold: 0,
                direction: Hammer.DIRECTION_HORIZONTAL  // 仅水平方向
            });

            const spanWidth = $(el).parent().find('span.delete').width();
            let elLeft = 0;
            let additionalEvent = '';

            mc.add(Pan);
            mc.on('panstart', (e) => {
                if (!this.currentEl) {
                    this.currentEl = el;    // 设置当前el，用于复位
                } else {
                    if (this.currentEl !== el) {
                        $(this.currentEl).animate({'left': '0px'}, () => {
                            this.currentEl.status = 0;
                            this.currentEl = el;
                        });
                    }
                }
                elLeft = parseInt($(el).css('left'));
            });
            mc.on('panmove', (e) => {
                additionalEvent = e.additionalEvent;

                // 1、默认状态时，左滑，打开删除按钮
                if (e.additionalEvent === 'panleft' && el.status === 0) {
                    if (Math.abs(elLeft + e.deltaX) >= spanWidth) { return; }
                    $(el).css({'left': (elLeft + e.deltaX) + 'px'});
                }

                // 2、默认状态时，非完成，右滑，滑动到底完成，期间显示完成提示
                if (e.additionalEvent === 'panright' && el.status === 0 && !el.finished) {
                    if (elLeft + e.deltaX >= spanWidth) { $(el).css({'left': (spanWidth - 5) + 'px'}); return; }
                    $(el).css({'left': (elLeft + e.deltaX) + 'px'});
                    // console.log(`elleft:${elLeft}, deltaX:${e.deltaX}`);
                }

                // 3、删除按钮打开时，右滑，关闭删除按钮
                if (e.additionalEvent === 'panright' && el.status === 1) {
                    if (elLeft + e.deltaX >= 0) { return; }
                    $(el).css({'left': (elLeft + e.deltaX) + 'px'});
                }
            });
            mc.on('panend', (e) => {
                elLeft = parseInt($(el).css('left'));

                // console.log(el.status);

                // 1、默认状态，左滑结束时
                if (el.status === 0 && additionalEvent === 'panleft') {
                    if (elLeft <= -10) {
                        $(el).animate({'left': -spanWidth + 'px'}, () => {
                            el.status = 1;
                        });
                    } else {
                        $(el).animate({'left': '0px'});
                    }
                }

                // 2、默认状态，右滑结束时，确认完成动作
                if (el.status === 0 && !el.finished && additionalEvent === 'panright') {
                    if (spanWidth - elLeft >= 10) {
                        $(el).animate({'left': '0px'});
                    } else {
                        $(el).animate({'left': spanWidth + 'px'}, () => {
                            el.finished = true; // 设置完成
                            this.finish($(el).parent().index());
                            setTimeout(() => {
                                $(el).animate({'left': '0px'}, () => this.refresh());
                            }, 500);
                        });
                    }
                }

                // 3、删除按钮打开，右滑结束时
                if (el.status === 1 && additionalEvent === 'panright') {
                    if (spanWidth + elLeft >= 10) {
                        $(el).animate({'left': '0px'}, () => {
                            el.status = 0;
                            this.currentEl = null;
                        });
                    } else {
                        $(el).animate({'left': -spanWidth + 'px'});
                    }
                }

                if (['panup', 'pandown'].indexOf(additionalEvent) !== -1) {
                    $(el).animate({'left': '0px'});
                }
            });

            mc.on('pancancel', (e) => {
                console.log(e);
            });
        },
        refresh() {
            // this.list.sort((a, b) => a.finished !== b.finished ? a.finished : a.id < b.id);
            this.list.sort((a, b) => {
                if (a.finished !== b.finished) {
                    return a.finished ? 1 : -1;
                } else {
                    return a.id < b.id ? 1 : -1;
                }
            });
            this.saveStorage();
        },
        saveStorage() {
            localStorage.list = JSON.stringify(this.list);
            this.$nextTick(() => myScroll.refresh());
        },
        readStorage() {
            if (localStorage.list) { this.list = JSON.parse(localStorage.list); this.$nextTick(() => myScroll.refresh()); }
        }
    }
};
</script>

<style lang="less">
@import url('./assets/style/config.less');

html, body {
    padding: 0;
    margin: 0;
    width: 100%;
    height: 100%;
}

body {
    font-family: "Helvetica Neue",Helvetica,"PingFang SC","Hiragino Sans GB","Microsoft YaHei","微软雅黑",Arial,sans-serif;
    font-size: 14px;
}

* {
    box-sizing: border-box;
}

#app {
    width: 100%;
    height: 100%;
    padding: 10px;
    background-color:#111;
}

.container {
    height: 100%;
    background-color: #fff;
    border-radius: 10px;
    // opacity: 0.5;
    // display: flex;
    // flex-direction: column;
    // align-items: 
    padding-top: 110px;
    padding-bottom: 10px;
    position: relative;
}

header {
    // flex-grow:0;
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 110px;
    padding: 30px;
    display: flex;
    flex-direction: column;
    justify-content: flex-end;

    input {
        width: 100%;
        height: 50px;
        display: block;
        border: 0;
        padding: 0;
        font-size: 40px;
        line-height: 50px;
        outline: none;
    }
}

section {
    // flex-grow:1;
    height: 100%;
    overflow: hidden;

    ul {
        padding: 0;
        margin: 0;
        list-style: none;
        border-top: 1px solid @color-border1; /*no*/

        li {
            height: 100px;
            line-height: 100px;
            font-size: 40px;
            position: relative;
            border-bottom: 1px solid @color-border1; /*no*/
            

            &.finished div {
                // background-color:rgb(225, 245, 206);
                border-left: 5px solid @color-success; /*no*/
                text-decoration: line-through;
                color: @color-border1;
            }

            div {
                border-left: 5px solid @color-border1; /*no*/
            }

            span {
                position: absolute;
                top: 0;
                height: 100%;
                width: 140px;
                text-align: center;
                color: #FFF;
                
                &.finish {
                    left: 0px;
                    background-color: @color-success;
                }

                &.delete {
                    right: 0px;
                    background-color: @color-danger;
                }
            }

            div {
                position: absolute;
                z-index: 1;
                top: 0;
                left: 0;
                background-color: #fff;
                padding: 0 30px;
                width: 100%;
                height: 100%;
            }
        }
    }
}


.list-complete-item {
    transition: all 1s;
}
.list-complete-enter, .list-complete-leave-to
/* .list-complete-leave-active for below version 2.1.8 */ {
    opacity: 0;
}
.list-complete-leave-active {
    position: absolute;

    .finish {
        opacity: 0;
    }
}

</style>