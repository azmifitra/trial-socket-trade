<template>
  <!-- <Tutorial /> -->
  <div class="row">
    <div class="col-md-6 offset-md-3 col-sm-12">
      <h1 class="text-center">{{ title }}</h1>
      <br />
      <div id="status"></div>
      <div id="chat">
        <label for="room">Select room:</label>

        <select class="form-control" id="room" v-model="selected">
          <option v-for="(room, idx) of listRooms" :key="idx">
            {{ room }}
          </option>
        </select>

        <br />
        <label for="username">Your Name:</label>
        <input
          type="text"
          v-model="name"
          id="username"
          class="form-control"
          placeholder="Enter name..."
        />
        <br />
        <label for="messages">Chat:</label>
        <div class="card">
          <div id="messages" class="card-block">
            <ul>
              <li v-for="(message, idx) in messages" :key="idx">
                {{ message.name }}: {{ message.text }}
              </li>
            </ul>
          </div>
        </div>
        <br />
        <label for="textarea">Message:</label>
        <textarea
          id="textarea"
          class="form-control"
          v-model="text"
          placeholder="Enter message..."
        ></textarea>
        <br />
        <button id="send" class="btn" @click.prevent="sendMessage">Send</button>
      </div>
    </div>
  </div>
</template>

<script>
import io from 'socket.io-client'
export default {
  name: 'IndexPage',
  data: () => ({
    title: 'NestJS Chat Real Time',
    name: '',
    text: '',
    selected: 'general',
    messages: [],
    socket: null,
    activeRoom: '',
    rooms: {
      general: false,
      roomA: false,
      roomB: false,
      roomC: false,
      roomD: false,
    },
    listRooms: ['general', 'roomA', 'roomB', 'roomC', 'roomD'],
  }),

  computed: {
    isMemberOfActiveRoom() {
      return this.rooms[this.activeRoom]
    },
  },

  created() {
    this.activeRoom = this.selected
    this.socket = io('http://localhost:3004/chat')
    this.socket.on('msgToClient', (message) => {
      console.log(message)
      this.receivedMessage(message)
    })

    this.socket.on('connect', () => {
      this.check()
    })

    this.socket.on('joinedRoom', (room) => {
      this.rooms[room] = true
    })

    this.socket.on('leftRoom', (room) => {
      this.rooms[room] = false
    })
  },
  methods: {
    onChange(event) {
      this.socket.emit('leaveRoom', this.activeRoom)
      this.activeRoom = event.target.value
      this.socket.emit('joinRoom', this.activeRoom)
    },

    sendMessage() {
      if (this.validateInput()) {
        const message = {
          name: this.name,
          text: this.text,
          room: this.activeRoom,
        }
        this.socket.emit('msgToServer', message)
        this.text = ''
      }
    },
    receivedMessage(message) {
      this.messages.push(message)
    },
    validateInput() {
      return this.name.length > 0 && this.text.length > 0
    },
    check() {
      if (this.isMemberOfActiveRoom) {
        this.socket.emit('leaveRoom', this.activeRoom)
      } else {
        this.socket.emit('joinRoom', this.activeRoom)
      }
    },
  },
}
</script>
