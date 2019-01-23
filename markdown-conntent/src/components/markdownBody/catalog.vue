<template>
    <div class="pos-f-t text-left text-decoration-none fixed-top" id='markdownCatlog'>
        <div class="collapse" id="navbarToggleExternalContent">
            <div class="bg-dark p-4">
            </div>
        </div>
        <nav class="navbar navbar-dark bg-dark rounded-bottom">
            <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarToggleExternalContent" aria-controls="navbarToggleExternalContent" aria-expanded="false" aria-label="Toggle navigation">
            <span class="navbar-toggler-icon"></span>
            </button>
        </nav>
    </div>
</template>

<script>
import PubSub from 'pubsub-js'
import $ from 'jquery'

export default {
    name: 'markdownCatalog',
    mounted () {
        PubSub.subscribe('refreshCatlog', (msg, headInfo) => {
            if (!headInfo) {
                return false
            }
            let details = []
            headInfo.forEach((element, index) => {
                element.match(/<h(\d) id="(.*)?">(.*)?</g)
                details.push({
                    head: RegExp.$1,
                    id: RegExp.$2,
                    detail: RegExp.$3
                })
            }, this)
            let catlog = ''
            details.forEach((element) => {
                let textSize = parseInt(element.head) + 2
                if (textSize > 6) {
                    textSize = 6
                }
                let a = `<h${element.head} style='text-indent: ${element.head}em' class= "h${textSize}"
                ><a href="#${element.id}" class= 'text-white'>${element.detail}</a></h${element.head}>`
                catlog = catlog + a
            }, this)
            $('#navbarToggleExternalContent>div').html(catlog)
            $('#markdownCatlog').find('a').each((index, element) => {
                $(element).parent()[0].id = ''
                $(element).off('click').on('click', () => {
                    const id = element.href.split('/')[element.href.split('/').length - 1]
                    const target = document.getElementById(id.substr(1))
                    const scrollTop = target.offsetTop
                    document.documentElement.scrollTop = scrollTop
                    $('.navbar-toggler').trigger('click')
                })
            })
        })
    }
}
</script>

<style>
a,
a:link,
a:visited,
a:hover,
:active{
    text-decoration: none;
}
</style>
