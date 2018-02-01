<template>
    <div id="app">
        <div class="container">
            <header><input type="text" placeholder="请输入待办事项"></header>
            <section>
                <ul>
                    <li>
                        <span class="finish">完成</span>
                        <div>11111</div>
                        <span class="delete">删除</span>
                    </li>
                    <li class="finished">
                        <span class="finish">完成</span>
                        <div>22222</div>
                        <span class="delete">删除</span>
                    </li>
                    <li>
                        <span class="finish">完成</span>
                        <div>33333</div>
                        <span class="delete">删除</span>
                    </li>
                    <li>
                        <span class="finish">完成</span>
                        <div>44444</div>
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
    mounted() {
        const li = $('li>div');
        li.each((index, item) => {
            this.addHandle(item);
        });
    },
    methods: {
        addHandle(item) {
            let mc = new Hammer.Manager(item);
            let Pan = new Hammer.Pan();

            mc.add(Pan);
            mc.on('panleft', function(e) {
                // 删除按钮宽度
                // const spanWidth = $(item).parent().find('span.delete').width();

                $(item).css({'left': e.deltaX + 'px'});
                console.log(e.deltaX);
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
        border-bottom: 1px solid #CCC; /*no*/

        li + li {
            border-top: 1px solid #CCC; /*no*/
        }

        li {
            height: 100px;
            line-height: 100px;
            font-size: 40px;
            position: relative;

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
                    left: 30px;
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