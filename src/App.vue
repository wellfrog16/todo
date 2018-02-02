<template>
    <div id="app">
        <div class="container">
            <header><input type="text" placeholder="请输入待办事项" v-model="todo" @keyup.enter="save()"></header>
            <section>
                <ul>
                    <li v-for="item in list" v-bind:key="item.id">
                        <span class="finish">完成</span>
                        <div>{{ item.todo }}</div>
                        <span class="delete">删除</span>
                    </li>
                </ul>
            </section>
        </div>
    </div>
</template>

<script>
import $ from 'jquery';
import Hammer from 'hammer';

export default {
    name: 'app',
    data() {
        return {
            todo: '',
            list: []
        };
    },
    mounted() {
        $('li>div').each((index, item) => this.addHandle(item));
    },
    methods: {
        save() {
            if (!this.todo) { return; }
            this.list.unshift({ todo: this.todo, id: new Date().getTime() });
            this.todo = '';

            this.$nextTick(() => this.addHandle($('li:first-child>div')[0]));
        },
        addHandle(item) {
            item.status = 0;    // 初始状态

            let mc = new Hammer.Manager(item);
            let Pan = new Hammer.Pan({
                threshold: 0,
                direction: Hammer.DIRECTION_HORIZONTAL  // 仅水平方向
            });

            const spanWidth = $(item).parent().find('span.delete').width();
            let itemLeft = 0;
            let additionalEvent = '';

            mc.add(Pan);
            mc.on('panstart', function(e) {
                itemLeft = parseInt($(item).css('left'));
            });
            mc.on('panmove', function(e) {
                additionalEvent = e.additionalEvent;

                // 默认状态时，左滑，打开删除按钮
                if (e.additionalEvent === 'panleft' && item.status === 0) {
                    if (Math.abs(itemLeft + e.deltaX) >= spanWidth) { return; }
                    $(item).css({'left': (itemLeft + e.deltaX) + 'px'});
                }

                // 默认状态时，右滑，滑动到底完成，期间显示完成提示
                if (e.additionalEvent === 'panright' && item.status === 0) {
                    if (itemLeft + e.deltaX >= spanWidth) { return; }
                    $(item).css({'left': (itemLeft + e.deltaX) + 'px'});
                }

                // 状态1，删除按钮打开时，右滑，关闭删除按钮
                if (e.additionalEvent === 'panright' && item.status === 1) {
                    if (itemLeft + e.deltaX >= 0) { return; }
                    $(item).css({'left': (itemLeft + e.deltaX) + 'px'});
                }
            });
            mc.on('panend', function(e) {
                itemLeft = parseInt($(item).css('left'));

                console.log(item.status);

                // 默认状态，左滑结束时
                if (item.status === 0 && additionalEvent === 'panleft') {
                    if (itemLeft <= -10) {
                        $(item).stop().animate({'left': -spanWidth + 'px'}, () => {
                            item.status = 1;
                        });
                    } else {
                        $(item).stop().animate({'left': '0px'});
                    }
                }

                // 删除按钮打开，右滑结束时
                if (item.status === 1 && additionalEvent === 'panright') {
                    if (spanWidth + itemLeft >= 10) {
                        $(item).stop().animate({'left': '0px'}, () => {
                            item.status = 0;
                        });
                    } else {
                        $(item).stop().animate({'left': -spanWidth + 'px'});
                    }
                }

                // 默认状态，右滑结束时，确认完成动作
                if (item.status === 0 && additionalEvent === 'panright') {
                    if (spanWidth - itemLeft >= 10) {
                        $(item).stop().animate({'left': '0px'});
                    } else {
                        $(item).stop().animate({'left': spanWidth + 'px'}, () => {
                            item.status = 2;
                            // todo 完成操作
                        });
                    }
                }
            });
        }
    }
};
</script>

<style lang="less">
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
    padding: 50px;
    background-color:cornflowerblue;
}

.container {
    height: 100%;
    background-color: #fff;
    border-radius: 10px;
    // opacity: 0.5;
    display: flex;
    flex-direction: column;
    // align-items: 
}

header {
    flex-grow:0;
    padding: 30px;
    display: flex;
    flex-direction: column;
    justify-content: flex-end;

    input {
        width: 100%;
        height: 100%;
        display: block;
        border: 0;
        font-size: 40px;
        line-height: 40px;
        outline: none;
    }
}

section {
    flex-grow:1;
    overflow: hidden;

    ul {
        padding: 0;
        margin: 0;
        list-style: none;
        border-top: 1px solid #CCC; /*no*/

        li {
            height: 100px;
            line-height: 100px;
            font-size: 40px;
            position: relative;
            border-bottom: 1px solid #CCC; /*no*/

            &.finished div {
                background-color:rgb(225, 245, 206);
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
                    background-color: #67C23A;
                }

                &.delete {
                    right: 0px;
                    background-color: #F56C6C;
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

</style>