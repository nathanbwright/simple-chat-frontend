<template>
  <div id="app">
    <h1>Simple chat</h1>
    <div v-if="!joined">
      <label>
        Please enter a username:
        <input
          v-model="username"
          v-on:keyup.enter="selectUserName">
      </label>
        <button
          v-show="username"
          v-on:click="selectUserName">
          Join chat
        </button>
    </div>
    <div v-else>
      <ul>
        <chat-message
          v-for="(message, i) in chatMessages"
          :key="i"
          :message="message"/>
      </ul>
      <div>
        <label>Say something:
          <input
            v-model="newMsg"
            v-on:keyup.enter="message">
        </label>
      </div>
    </div>
    <div v-if="joined && username">
      <button
        v-on:click="leaveRoom">
        Leave chat
      </button>
    </div>
  </div>
</template>

<script>
const ActionCable = require('actioncable');

import ChatMessage from './components/ChatMessage'

export default {
  name: 'app',
  data () {
    return {
      room: null,
      newMsg: '',
      chatMessages: [],
      joined: false,
      username: null
    }
  },
  created: function() {
    var self = this;

    const cable = ActionCable.createConsumer('ws://localhost:3000/cable');
    this.room = cable.subscriptions.create('RoomChannel', {
      connected: () => {},
      disconnected: () => {},
      received: (data) => {
        this.chatMessages.push(data["message"])
      },
      speak: (message) => {
        self.room.perform("speak", {message: message})
      },
      joinRoom: (username) => {
        self.room.perform("join_room", {username: username})
      },
      leaveRoom: (username) => {
        self.room.perform("leave_room", {username: username})
      }
    });
  },
  methods: {
    message: function() {
      if (this.newMsg.length === 0) { return };
      this.room.speak(this.newMsg);
      this.newMsg = "";
    },
    selectUserName: function() {
      this.chatMessages = [];
      this.room.joinRoom(this.username);
      this.joined = true;
    },
    leaveRoom: function() {
      this.room.leaveRoom(this.username);
      this.joined = false;
      this.username = null;
      this.$children = [];
      this.chatMessages = [];
    }
  },
  components: {
    ChatMessage
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  margin: 0 10px;
}
</style>
