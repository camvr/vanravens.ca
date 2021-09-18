<template>
  <v-app>
    <v-main>
      <v-container fluid style="height: 100vh">
        <v-layout fill-height>
          <v-row>
            <v-col cols="12" sm="12" md="8" offset-md="2">
              <vue-command
                :autocompletion-resolver="autocompletionResolver"
                :commands="commands"
                :cursor.sync="cursor"
                :history.sync="history"
                :hide-bar="true"
                :hide-title="true"
                :intro="intro"
                :show-intro="false"
                :prompt="prompt"
                :stdin.sync="stdin"
                show-help
              />
            </v-col>
          </v-row>
        </v-layout>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import VueCommand, { createStdout, createDummyStdout } from 'vue-command'
import 'vue-command/dist/vue-command.css'

const PROMPT = 'guest@vanravens.ca $'

export default {
  components: {
    VueCommand
  },

  data: () => ({
    autocompleteResolver: () => undefined,

    commands: {
      about: () => createStdout(`Hello! My name is Cam Van Ravens.<br><br>
      I am a Software Developer from Canada, graduated from McMaster University 
      with a Bachelors Of Engineering in Software Development. I am also a hobby
      musician and music producer, and I enjoy extreme sports such as
      snowboarding and street unicycling.<br>
      `),

      // Clear terminals history
      clear: undefined,

      echo: _ => createStdout(JSON.stringify(_, null, 2)),

      'hello-world': () => createStdout('Hello world!'),

      welcome: () => createStdout(`Welcome to my homepage!<br><br>
      You can view available commands by typing 'help'.<br><br>
      This webpage is currently a work-in-progress. Stay tuned for future updates!
      `),

      // Show help
      help: () => createStdout(`Available commands:<br><br>
        &nbsp;about<br>
        &nbsp;clear<br>
        &nbsp;echo<br>
        &nbsp;welcome<br>
        &nbsp;hello-world<br><br>
      `)
    },

    intro: 'Welcome to my homepage! This site is a work-in-progress.',

    // Terminal cursor position
    cursor: 0,
    history: [],
    prompt: PROMPT,
    stdin: ''
  }),

  created () {
    this.commands.clear = () => {
      this.history = []
      // Push dummy Stdout to show Stdin
      return createDummyStdout()
    }

    this.autocompletionResolver = () => {
      // Preserve cursor position
      const cursor = this.cursor
      // Reverse concatenate autocompletable according to cursor
      let pointer = this.cursor
      let autocompleteableStdin = ''
      while (this.stdin[pointer - 1] !== ' ' && pointer - 1 > 0) {
        pointer--
        autocompleteableStdin = `${this.stdin[pointer]}${autocompleteableStdin}`
      }
      // Divide by arguments
      const command = this.stdin.split(' ')
      // Autocompleteable is program
      if (command.length === 1) {
        const autocompleteableProgram = command[0]
        // Collect all autocompletion candidates
        const candidates = []
        const programs = [...Object.keys(this.commands), 'reverse'].sort()
        programs.forEach((program) => {
          if (program.startsWith(autocompleteableProgram)) {
            candidates.push(program)
          }
        })
        // Autocompletion resolved into multiple results
        if (this.stdin !== '' && candidates.length > 1) {
          this.history.push({
            // Build table programmatically
            render: (createElement) => {
              const columns = candidates.length < 5 ? candidates.length : 4
              const rows = candidates.length < 5 ? 1 : Math.ceil(candidates.length / columns)
              let index = 0
              const table = []
              for (let i = 0; i < rows; i++) {
                const row = []
                for (let j = 0; j < columns; j++) {
                  row.push(createElement('td', candidates[index]))
                  index++
                }
                table.push(createElement('tr', [row]))
              }
              return createElement('table', { style: { width: '100%' } }, [table])
            }
          })
        }
        // Autocompletion resolved into one result
        if (candidates.length === 1) {
          // Mutating Stdin mutates the cursor, so we've to wait to push it to the end
          const unwatch = this.$watch(() => this.cursor, () => {
            this.cursor = cursor + (candidates[0].length - autocompleteableStdin.length + 0)
            unwatch()
          })
          this.stdin = candidates[0]
        }
        return
      }
      // Check if option might be completed already or option is last tokens
      if ((this.stdin[cursor] !== '' && this.stdin[cursor] !== ' ') && typeof this.stdin[cursor] !== 'undefined') {
        return
      }
      // Get the executable
      const program = command[0]
      // Check if any autocompleteable exists
      if (typeof this.options.long[program] === 'undefined' && typeof this.options.short[program] === 'undefined') {
        return
      }
      // Autocompleteable is long option
      if (autocompleteableStdin.substring(0, 2) === '--') {
        const candidates = []
        this.options.long[program].forEach((option) => {
          // If only dashes are presents, user requests all options
          if (`--${option}`.startsWith(autocompleteableStdin) || autocompleteableStdin === '--') {
            candidates.push(option)
          }
        })
        // Autocompletion resolved into one result
        if (candidates.length === 1) {
          const autocompleted = `${this.stdin.substring(0, pointer - 1)} --${candidates[0]}`
          const rest = `${this.stdin.substring(this.cursor)}`
          // Mutating Stdin mutates the cursor, so we've to wait to push it to the end
          const unwatch = this.$watch(() => this.cursor, () => {
            this.cursor = cursor + (candidates[0].length - autocompleteableStdin.length + 2)
            unwatch()
          })
          this.stdin = `${autocompleted}${rest}`
          return
        }
        // Autocompletion resolved into multiple result
        if (autocompleteableStdin === '--' || candidates.length > 1) {
          this.history.push({
            // Build table programmatically
            render: (createElement) => {
              const columns = candidates.length < 5 ? candidates.length : 4
              const rows = candidates.length < 5 ? 1 : Math.ceil(candidates.length / columns)
              let index = 0
              const table = []
              for (let i = 0; i < rows; i++) {
                const row = []
                for (let j = 0; j < columns; j++) {
                  row.push(createElement('td', `--${candidates[index]}`))
                  index++
                }
                table.push(createElement('tr', [row]))
              }
              return createElement('table', { style: { width: '100%' } }, [table])
            }
          })
        }
        return
      }
      // Autocompleteable is option
      if (autocompleteableStdin.substring(0, 1) === '-') {
        const candidates = []
        this.options.short[program].forEach((option) => {
          // If only one dash is present, user requests all options
          if (`-${option}`.startsWith(autocompleteableStdin) || autocompleteableStdin === '-') {
            candidates.push(option)
          }
        })
        // Autocompletion resolved into one result
        if (candidates.length === 1) {
          const autocompleted = `${this.stdin.substring(0, pointer - 1)} -${candidates[0]}`
          const rest = `${this.stdin.substring(this.cursor)}`
          // Mutating Stdin mutates the cursor, so we've to wait to push it to the end
          const unwatch = this.$watch(() => this.cursor, () => {
            this.cursor = cursor + (candidates[0].length - autocompleteableStdin.length + 1)
            unwatch()
          })
          this.stdin = `${autocompleted}${rest}`
          return
        }
        // Autocompletion resolved into multiple result
        if (autocompleteableStdin === '-' || candidates.length > 1) {
          this.history.push({
            // Build table programmatically
            render: (createElement) => {
              const columns = candidates.length < 5 ? candidates.length : 4
              const rows = candidates.length < 5 ? 1 : Math.ceil(candidates.length / columns)
              let index = 0
              const table = []
              for (let i = 0; i < rows; i++) {
                const row = []
                for (let j = 0; j < columns; j++) {
                  row.push(createElement('td', `-${candidates[index]}`))
                  index++
                }
                table.push(createElement('tr', [row]))
              }
              return createElement('table', { style: { width: '100%' } }, [table])
            }
          })
        }
      }
    }
  }
}
</script>

<style lang="scss">

html {
  overflow: hidden !important;

  .vue-command {
    ::-webkit-scrollbar {
      width: 6px;
    }
    ::-webkit-scrollbar-track {
      background: #252525;
    }
    ::-webkit-scrollbar-thumb {
      background: #f1f1f1;
    }
    ::-webkit-scrollbar-thumb:hover {
      background: #333;
    }
    .term-std {
      // Digusting, but it works lol
      min-height: 95vh;
      max-height: 95vh;
      overflow-y: scroll;
      background: var(--v-background-base);
    }
  }
}
</style>
