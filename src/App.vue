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
  smile:  '🙂',
  sad: '🙁',
};

const URL = 'http://localhost:3000';

const Socket = io(URL, { autoConnect: false });

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
    Socket.on('reaction', this.handleReactionMessage)
  },
  beforeDestroy() {
    Socket.disconnect();
  },
  destroyed() {
    Socket.off('reaction');
  },
  methods: {
    handleReactionClick(code) {
      this.updateReceived(code);

      Socket.emit('reaction', code);
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
