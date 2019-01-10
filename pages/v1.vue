<template>
  <v-layout
    row
    fill-height
    justify-center
    align-end>
    <v-flex
      xs12
      sm6
      offset-sm3
    >
      <v-card>
        <v-list v-show="existsChat"> 
          <template v-for="(item, index) in chat">
            <v-divider :key="index" v-if="index != 0"></v-divider>
            <v-list-tile>
              <v-list-tile-avatar>
                <v-icon>{{ item.user.icon }}</v-icon>
              </v-list-tile-avatar>
              <v-list-tile-content>
                <v-list-tile-title v-html="item.message" />
              </v-list-tile-content>
            </v-list-tile>
          </template>
        </v-list>
      </v-card>
      <form class="input-form">
        <v-text-field
          v-model="message"
          :prepend-icon="user.icon"
          append-outer-icon="send"
          class="chat-input"
          label="メッセージ"
          required
          outline
          @click:append-outer="sendMessage"
          @click="sendMessage"
          ></v-text-field>
      </form>
    </v-flex>
  </v-layout>
</template>

<script>
import firebase from '~/plugins/firebase'

export default {
  data () {
    return {
      user: {
        icon: "face",
      },
      message: "",
      chat: [],
      database: undefined,
      iconIndex: 0,
      icons: [
        "face",
        "tag_faces",
      ]
    }
  },
  created: function () {
    this.database = firebase.database()
    this.chatRef = this.database.ref('chat')
    this.chatRef.limitToLast(10).on('child_added', snapshot => {
      console.log(this.chat.length)
      const ss = snapshot.val()
      this.chat.push({
        key: ss.key,
        message: ss.message,
        user: ss.user
      })
    })
    this.scrollBottom()
  },
  computed: {
    existsChat () {
      return this.chat.length > 0
    },
    nextIcon () {
      this.iconIndex += 1
      if (this.iconIndex >= this.icons.length) {
        this.iconIndex = 0
      }
      this.user.icon = this.icons[this.iconIndex]
    },
  },
  methods: {
    clearMessage() {
      this.message = ""
    },
    sendMessage () {
      if (this.message.length) {
        this.chatRef.push({message: this.message, user: this.user})
        this.clearMessage()
        this.scrollBottom()
      }
    },
    scrollBottom() {
      this.$nextTick(() => {
        window.scrollTo(0, document.body.clientHeight)
      })
    },
  },
}
</script>

<style>
.input-form {
  padding-top: 30px;
}
</style>
