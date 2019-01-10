<template>
  <v-app light>
    <v-toolbar
      fixed
      app
      color="teal lighten-3"
    >
    <v-toolbar-title class="toolbar-title" v-text="title"/>
    <v-spacer></v-spacer>
    <v-toolbar-items class="toolbar-items">
      <v-btn v-if="!isAuthenticated()" @click="doLogin" flat>LOGIN</v-btn>
      <v-btn v-if="isAuthenticated()" @click="doLogout" flat>LOGOUT</v-btn>
      <v-avatar v-if="isAuthenticated()" class="toolbar-avator" size=38>
        <img :src="$store.state.user.photoURL">
      </v-avatar>
    </v-toolbar-items>
    </v-toolbar>
    <v-content>
      <v-container fill-height>
        <nuxt />
      </v-container>
    </v-content>
  </v-app>
</template>

<script>
import { mapActions, mapState, mapGetters  } from 'vuex'
import firebase from '~/plugins/firebase'

export default {
  data() {
    return {
      title: 'Easy Chat'
    }
  },
  created: function () {
    this.setUser(null)
    firebase.auth().onAuthStateChanged(user => {
      this.setUser(user ? user : null)
    })
  },
  computed: {
    ...mapState(['user']),
  },
  methods: {
    ...mapActions(['setUser']),
    ...mapGetters(['isAuthenticated']),
    doLogin() {
      const provider = new firebase.auth.GoogleAuthProvider()
      firebase.auth().signInWithPopup(provider)
    },
    doLogout() {
      firebase.auth().signOut()
    },
  }
}
</script>

<style>
.toolbar-title {
  display        : inline-block;
  color          : #ffffff;            /* 文字の色 */
  font-size      : 20pt;               /* 文字のサイズ */
  letter-spacing : 4px;                /* 文字間 */
  text-shadow    : 
       2px  2px 1px #003366,
      -2px  2px 1px #003366,
       2px -2px 1px #003366,
      -2px -2px 1px #003366,
       2px  0px 1px #003366,
       0px  2px 1px #003366,
      -2px  0px 1px #003366,
       0px -2px 1px #003366;        /* 文字の影 */
}

.toolbar-items {
  align-items: center;
}

.toolbar-avator {
}
</style>
