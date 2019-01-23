<template>
    <div id="markdownPreview" v-html="content">
    </div>
</template>

<script>
import PubSub from 'pubsub-js'
import hljs from 'highlight.js'
import 'highlight.js/styles/atom-one-dark.css'
import showdown from 'showdown'
import $ from 'jquery'

var converter = new showdown.Converter()

const highlightCode = () => {
    const preEl = document.querySelectorAll('pre')
    preEl.forEach((element) => {
        hljs.highlightBlock(element)
    })
}

const updateHead = (elementArray) => {
    elementArray.forEach((element) => {
        element.id = escape(element.innerText)
    })
}

const indexHead = () => {
    let head1 = document.querySelectorAll('h1')
    updateHead(head1)
    let head2 = document.querySelectorAll('h2')
    updateHead(head2)
    let head3 = document.querySelectorAll('h3')
    updateHead(head3)
    let head4 = document.querySelectorAll('h4')
    updateHead(head4)
    let head5 = document.querySelectorAll('h5')
    updateHead(head5)
    let head6 = document.querySelectorAll('h6')
    updateHead(head6)
}

const generateCatlog = () => {
    // 代码区域高亮
    highlightCode()
    // 发布更新目录的消息
    // 更新 heads 的 id
    indexHead()
    const newPattern = /(<h\d id=".*?\/h\d>)/g
    const headInfo = $('#markdownPreview').html().match(newPattern)
    PubSub.publish('refreshCatlog', headInfo)
}

const contentChangeHandler = (vueComponent, content) => {
    PubSub.subscribe('contentChange', (msg, data) => {
        if (vueComponent) {
            vueComponent.content = data
            vueComponent.content = converter.makeHtml(data)
        } else {
            content = data
            content = converter.makeHtml(data)
        }
        if (vueComponent) {
            generateCatlog()
        }
    })
    return content
}

export default {
    name: 'markdownPreview',
    data () {
        let content = ''
        content = contentChangeHandler(false, content)
        // 模拟异步操作,在数据挂在之后在进行目录的更新
        setTimeout(generateCatlog, 100)
        return {
            content: content
        }
    },
    mounted () {
        contentChangeHandler(this)
    }
}
</script>

<style scoped>
</style>
