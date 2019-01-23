<template>
    <textarea id="markdownContent" :value="content" >
    </textarea>
</template>

<script>
import PubSub from 'pubsub-js'

export default {
    name: 'markdownContent',
    mounted () {
        PubSub.publish('contentChange', this.content)
        const input = document.getElementById('markdownContent')
        input.addEventListener('input', () => {
            // this.content = input.innerText
            // 当内容发生变化的时候发布contentChange的消息
            PubSub.publish('contentChange', input.value)
            // let content = input.innerHTML
            let content = input.value
            window.localStorage.setItem('article', content)
        })
    },
    data () {
        let content = 'Write Here Please'
        if (window.localStorage.getItem('article')) {
            content = window.localStorage.getItem('article')
        }
        PubSub.publish('contentChange', content)
        return {
            content: content
        }
    }
}
</script>

<style scoped>
div{
    white-space:normal;
    word-break:break-all;
    word-wrap:break-word
}

</style>
