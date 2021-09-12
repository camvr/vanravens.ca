<template>
  <v-app>
    <v-main>
      <v-container>
        <vue-command
          :commands="commands"
          :hide-bar="true"
          :hide-title="true"
          :prompt="prompt"
        />
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import VueCommand, { createStdout } from 'vue-command'
import 'vue-command/dist/vue-command.css'

const PROMPT = 'anon@vanravens.ca:~ $'

export default {
  components: {
    VueCommand
  },

  data: () => ({
    commands: {
      'hello-world': () => createStdout('Hello world')
    },

    // Terminal cursor position
    cursor: 0,
    history: [],
    options: {
      long: {
        loading: ['amount', 'timeout']
      }
    },
    prompt: PROMPT,
    stdin: ''
  })
}
</script>
