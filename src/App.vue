<template>
    <div id="app">
        <div class="container">
            <header><input type="text"></header>
            <section>
                <ul>
                    <li>1111111</li>
                    <li>2222222</li>
                    <li>33333</li>
                    <li>44444</li>
                </ul>
            </section>
        </div>
    </div>
</template>

<script>
import $ from 'jquery';

export default {
    name: 'app',
    created() {
        console.log(112233);
        console.log($);
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
        border-bottom: 1px solid #CCC;
        font-size: 40px;
        line-height: 40px;
        outline: none;
    }
}

section {
    flex-grow:1;

    ul {
        padding: 0;
        margin: 0;
        list-style: none;

        li {
            height: 100px;
            line-height: 100px;
            font-size: 40px;
            padding: 0 30px;
        }
    }
}

</style>