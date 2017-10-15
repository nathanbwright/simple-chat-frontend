<template>
  <div id="app">
    <h1>Simple chat</h1>
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
      chatMessages: []
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
      }
    });
  },
  methods: {
    message: function() {
      this.room.speak(this.newMsg);
      this.newMsg = "";
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
