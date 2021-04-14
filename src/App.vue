<template>
  <div id="app">
    <template v-for="(symbol, code) in reactions">
      <button :key="code" @click="handleReactionClick(code)">{{ symbol }}</button>
    </template>

    <div>
      <template v-for="(symbol, index) in received">
        <span :key="index">{{ reactions[symbol] }}</span>
      </template>
    </div>
  </div>
</template>

<script>
import { io } from 'socket.io-client';

const REACTIONS = {
  like: '👍',
  dislike: '👎',
  heart: '❤️',
  happy:  '🙂',
  sad: '🙁',
};

const URL = 'http://localhost:3000';

const token = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkRtaXRyeSIsImlhdCI6MTUxNjIzOTAyMiwidXNlciI6InNpcm90ZW5rby1kIn0.wGEkWakFCDmC2PnfMUCd4gps0vD2cOp8veT7OhDYLN8';

const Socket = io(URL, {
  autoConnect: false,
  transports: ['websocket'],
  auth: (cb) => cb({ token }),
});

export default {
  data() {
    return {
      received: [],
    };
  },
  computed: {
    reactions: () => REACTIONS,
  },
  mounted() {
    Socket.connect();

    Socket.on('reaction:send', this.handleReactionMessage)
  },
  beforeDestroy() {
    Socket.disconnect();
  },
  destroyed() {
    Socket.off('reaction:send');
  },
  methods: {
    handleReactionClick(code) {
      this.updateReceived(code);

      Socket.emit('reaction:send', code);
    },
    handleReactionMessage(code) {
      this.updateReceived(code);
    },
    updateReceived(code) {
      this.received.push(code);
    },
  },
};
</script>
