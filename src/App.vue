<template>
  <div id="app">
    <ul>
      <li class="user" v-for="user in users" transition>
        <span>{{user.name}} - {{user.email}}</span>
        <button v-on:click="removeUser(user)">X</button>
      </li>
    </ul>
    <form id="form" v-on:submit.prevent="addUser">
      <input v-model="newUser.name">
      <input v-model="newUser.email">
      <input type="submit" value="Add User">
    </form>
    <ul class="errors">
      <li v-show="!validation.name">Name cannot be empty.</li>
      <li v-show="!validation.email">Please provide a valid email address.</li>
    </ul>
</div>
</template>
<script>
var firebase = require('firebase/app')
require('firebase/auth')
require('firebase/database')

var config = {
  apiKey: 'AIzaSyBvZQ5jYSY7hECHidvWd4F3btGgRkhOYdQ',
  authDomain: 'todos-50735.firebaseapp.com',
  databaseURL: 'https://todos-50735.firebaseio.com',
  storageBucket: 'todos-50735.appspot.com',
  messagingSenderId: '514818084106'
}
firebase.initializeApp(config)

// firebase.database().ref('users/1234').set({
//   username: 'name',
//   email: 'email'
// })
var Users = firebase.database().ref('users')
var emailRE = /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/
export default {
  data () {
    return {
      users: [],
      newUser: {
        name: '',
        email: ''
      }
    }
  },
  // computed property for form validation state
  computed: {
    validation: function () {
      return {
        name: !!this.newUser.name.trim(),
        email: emailRE.test(this.newUser.email)
      }
    },
    isValid: function () {
      var validation = this.validation
      return Object.keys(validation).every(function (key) {
        return validation[key]
      })
    }
  },
  // methods
  methods: {
    addUser: function () {
      if (this.isValid) {
        Users.push(this.newUser)
        this.newUser.name = ''
        this.newUser.email = ''
      }
    },
    removeUser: function (user) {
      // new Firebase(baseURL + 'users/' + user.id).remove()
      firebase.database().ref('users/' + user.id).remove()
    }
  },
  ready: function () {
    let vm = this
    Users.on('child_added', function (snapshot) {
      var item = snapshot.val()
      item.id = snapshot.key
      vm.users.push(item)
    })
    Users.on('child_removed', function (snapshot) {
      var id = snapshot.key
      vm.users.some(function (user) {
        if (user.id === id) {
          vm.users.$remove(user)
          return true
        }
      })
    })
  }
}
</script>

<style>

body {
  font-family: Helvetica, Arial, sans-serif;
}

ul {
  padding: 0;
}

.user {
  height: 30px;
  line-height: 30px;
  padding: 10px;
  border-top: 1px solid #eee;
  overflow: hidden;
  transition: all .25s ease;
}

.user:last-child {
  border-bottom: 1px solid #eee;
}

.v-enter, .v-leave {
  height: 0;
  padding-top: 0;
  padding-bottom: 0;
  border-top-width: 0;
  border-bottom-width: 0;
}

.errors {
  color: #f00;
}

</style>
