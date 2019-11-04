<template>
<div id="app">
    <div 
        class="term flex-col"
        tabindex="0"
        @focus.prevent="focusOnInput"
        id="term"
    >
        <div class="screen flex-col">
            <ul v-if="history.length" class="flex-col">
                <output-row v-for="entry in history" :key="entry.id" :entryText="entry.text" promptText="anon@ronniepyne.com" :entryType="entry.type"></output-row>
            </ul>

            <div id="prompt" class="flex-row-center input-row">
                <span class="prompt-icon">anon@ronniepyne.com&nbsp;$</span> 
                <input
                    ref="input" 
                    v-model="userInput" type="text"
                    autofocus autocomplete="off"
                    @keyup.enter="onReturnPressed"
                    @keyup.up="historyBackwards"
                    @keyup.down="historyForwards"
                />
            </div>
        </div>
    </div>
</div>
</template>

<script>

import OutputRow from './components/OutputRow.vue'
import CommandParser from './services/CommandParser'
import StorageService from './services/StorageService'

const cmdParser = new CommandParser()
const storage = new StorageService()

function sleep(ms) {
  return new Promise(async (resolve) => setTimeout(resolve, ms))
}

export default {
    components: { 'output-row': OutputRow },
    data () {
        return {
            history: [],
            userInput: '',
            currentHistoryIndex: 0
        } 
    },

    methods: {
        onReturnPressed() {
            this.addToHistory(
                'input',
                this.userInput.replace(/ /g, '&nbsp;') // replace spaces with nbsp
            )
            let [command, ...args] = this.userInput.trim().split(' ')

            let output = cmdParser.parse(command, args)
            command = command.toLowerCase()

            switch (command) {
                case 'clear':
                    this.history = []
                    storage.save(this.history)
                    break
                default:
                    if (command.length && output) this.doOutput(output)
                    break
            }

            this.userInput = ''
            this.currentHistoryIndex = this.history.length
        },

        async doOutput(output) {
            if (Array.isArray(output)) {
                for (let line in output) {
                    this.addToHistory('output', output[line])
                    await sleep(15)
                }
            } else {
                this.addToHistory('output', output)
            }
        },

        addToHistory(type, value) {
            this.history.push({
                id: this.history.length,
                text: value,
                timestamp: new Date(),
                type,
            })
            storage.save(this.history)
        },

        focusOnInput(e) {
            this.$refs.input.focus()
        },

        lastEntry() {
            return this.history[this.history.length - 1]
        },

        historyBackwards() {
            for (let index = this.currentHistoryIndex - 1; index >= 0; index--) {
                if (this.history[index].type == 'input') {
                    this.userInput = this.history[index].text.replace(/&nbsp;/g, ' ')
                    this.currentHistoryIndex = index

                    break
                }
            }
        },

        historyForwards() {
            for (let index = this.currentHistoryIndex + 1; index < this.history.length - 1; index++) {
                if (this.history[index].type == 'input') {
                    this.userInput = this.history[index].text.replace(/&nbsp;/g, ' ')
                    this.currentHistoryIndex = index

                    break
                }
            }
        }
    },

    updated() {
        // make sure we're scrolled to bottom after dom render

        this.$nextTick(function () {
            const term = document.getElementById('term')
            const isScrolledToBottom = term.scrollHeight - term.clientHeight <= term.scrollTop + 1
            term.scrollTop = term.scrollHeight - term.clientHeight
        })
    },

    mounted() {
        this.history = storage.fetch()

        if (!this.history.length) this.doOutput(cmdParser.parse('motd'))
        this.currentHistoryIndex = this.history.length
    },

    computed: {
        
    },
}
</script>

<style lang="scss">
@import './assets/normalize.scss';
@import './assets/app-style.scss';

</style>
