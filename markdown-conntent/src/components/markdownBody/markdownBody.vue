<template>
    <div id="markdownBody">
        <div class="row text-left">
            <markdown-catlog class='w-100 rounded' />
            <div class="w-100"></div>
            <markdown-content class="col-6 px-3" :content="content"/>
            <markdown-preview class="col-6 px-3" />
        </div>
    </div>
</template>

<script>
import markdownCatlog from './catalog'
import markdownContent from './content'
import markdownPreview from './preview'

export default {
    name: 'markdownMain',
    components: {
        markdownCatlog,
        markdownContent,
        markdownPreview
    },
    data () {
        return {
            content: ''
        }
    }
}
</script>

<style scoped>
</style>
