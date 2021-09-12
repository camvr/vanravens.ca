<template>
  <v-app>
    <v-content>
      <v-container fill-height>
        <v-row align="center" justify="center">
          <v-col>
            <vue-command
              :commands="commands"
              :cursor.sync="cursor"
              :history.sync="history"
              :hide-bar="true"
              :hide-title="true"
              :intro="intro"
              :show-intro="true"
              :prompt="prompt"
              :stdin.sync="stdin"
            />
          </v-col>
        </v-row>
      </v-container>
    </v-content>
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
    commands: {
      // Clear terminals history
      clear: undefined,

      echo: _ => createStdout(JSON.stringify(_, null, 2)),

      'hello-world': () => createStdout('Hello world!'),

      // Show help
      help: () => createStdout(`Available commands:<br><br>
        &nbsp;clear<br>
        &nbsp;echo<br>
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
  }
}
</script>

<style>
html {
  overflow: hidden !important;
}

.v-card {
  display: flex !important;
  flex-direction: column;
}

.v-card__text {
  flex-grow: 1;
  overflow: auto;
}
</style>
