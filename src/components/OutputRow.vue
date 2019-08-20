<template>
    <li class="entry flex-row-center" v-bind:class="{ 'input-row': entryType === 'input' }">
        <span v-show="entryType ==='input'" class="prompt-icon">{{promptText}}&nbsp;$</span>
        <span class="entry-text" v-html="entryText"></span>
    </li>
</template>

<script>

export default {
    name: 'OutputRow',
    props: ['promptText', 'entryText', 'entryType'],
    data () {
        return {
        } 
    },
}

</script>