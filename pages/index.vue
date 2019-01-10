<template>
  <v-layout
    row
    fill-height
    justify-center
    align-end>
    <v-flex
      xs12
      sm6
    >
      <div v-if="!isAuthenticated()" class="text-xs-center">
        <img class="pen" position="center center" width="200px" :src="require('~/assets/pen.png')" />
      </div>
      <div v-if="!isAuthenticated()" class="text-xs-center">
        <v-btn color="info" round large @click="doLogin">Googleでログイン</v-btn>
      </div>
      <div  v-if="isAuthenticated()">
        <v-card>
          <v-list v-show="chat.length > 0" class="chat-list"> 
            <template v-for="(item, index) in chat">
              <v-divider :key="index" v-if="index != 0"></v-divider>
              <v-list-tile>
                <v-list-tile-avatar class="chat-avatar" size=32>
                  <img :src="item.image">
                </v-list-tile-avatar>
                <v-list-tile-content>
                  <v-list-tile-title class="chat-message" v-html="item.message" />
                </v-list-tile-content>
              </v-list-tile>
            </template>
          </v-list>
        </v-card>
        <form class="input-form">
          <v-text-field
            v-model="message"
            append-outer-icon="send"
            class="chat-input"
            label="メッセージ"
            hide-details
            required
            outline
            @click:append-outer="sendMessage"
            @click="sendMessage"
            @keyup.enter="sendMessage"
            ></v-text-field>
        </form>
      </div>
    </v-flex>
  </v-layout>
</template>

<script>
import firebase from '~/plugins/firebase'
import { mapActions, mapState, mapGetters  } from 'vuex'

export default {
  data () {
    return {
      message: "",
      chat: [],
      database: undefined,
    }
  },
  created: function () {
    this.database = firebase.database()
    this.chatRef = this.database.ref('chat')
    firebase.auth().onAuthStateChanged(user => {
      if (user) {
        this.chatRef.limitToLast(10).on('child_added', snapshot => {
          const ss = snapshot.val()
          this.chat.push({
            key: ss.key,
            message: ss.message,
            image: ss.image,
            name: ss.name,
          })
        this.scrollBottom()
        })
      } else {
        this.chat = []
      }
    })
  },
  computed: {
    ...mapState(['user'])
  },
  methods: {
    ...mapActions(['setUser']),
    ...mapGetters(['isAuthenticated']),
    doLogin() {
      const provider = new firebase.auth.GoogleAuthProvider()
      firebase.auth().signInWithPopup(provider)
    },
    clearMessage() {
      this.message = ""
    },
    sendMessage () {
      if (this.message.length) {
        this.chatRef.push({
          message: this.message,
          name: this.user.displayName,
          image: this.user.photoURL
        })
        this.clearMessage()
        this.scrollBottom()
      }
    },
    scrollBottom() {
     if (process.client) {
        this.$nextTick(() => {
          window.scrollTo(0, document.body.clientHeight)
        })
      }
    },
  },
}
</script>

<style>
.pen {
  width: 200px;
}

.chat-list {
  padding: 0;
}

.chat-avatar {
  min-width: 42px;
}

.chat-message {
  font-size: 14px;
}

.input-form {
  padding-top: 15px;
}
</style>
