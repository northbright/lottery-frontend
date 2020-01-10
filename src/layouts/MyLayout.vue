<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-btn
          flat
          dense
          round
          @click="leftDrawerOpen = !leftDrawerOpen"
          icon="menu"
          aria-label="Menu"
        />

        <q-toolbar-title>
          Lottery
        </q-toolbar-title>

        <div>Quasar v{{ $q.version }}</div>
      </q-toolbar>
    </q-header>

    <q-footer bordered class="bg-white text-primary">
      <q-btn label="start/stop" @click="startStop()" />
    </q-footer>

    <q-drawer
      v-model="leftDrawerOpen"
      show-if-above
      bordered
      content-class="bg-grey-2"
    >
      <q-list>
        <q-item-label header>Essential Links</q-item-label>
        <q-item
          v-for="(prize, index) in prizes"
          :key="index"
          clickable
          @click="selectPrize(index)"
        >
          <q-item-section avatar>
            <q-icon name="school" />
          </q-item-section>
          <q-item-section>
            <q-item-label>{{ prize.name }}</q-item-label>
          </q-item-section>
        </q-item>
      </q-list>
    </q-drawer>

    <q-page-container> </q-page-container>
    <div class="q-gutter-md">
      <q-btn
        color="deep-orange"
        v-for="(winner, index) in this.winners"
        :label="winner.name"
        :key="index"
      ></q-btn>
    </div>
  </q-layout>
</template>

<script>
export default {
  name: "MyLayout",

  data() {
    return {
      leftDrawerOpen: false,
      prizes: [],
      prizeIndex: 0,
      winners: [],
      url: "ws://192.168.1.7:8081/ws",
      conn: {},
      started: false
    };
  },

  created() {
    console.log("created()");
    this.initWebSocket();
  },

  methods: {
    initWebSocket() {
      console.log("initWebSocket()");
      this.conn = new WebSocket(this.url);
      this.conn.onopen = this.webSocketOnOpen;
      this.conn.onmessage = this.webSocketOnMessage;
    },

    webSocketOnOpen() {
      console.log("WebSocket on open");
      var action = { name: "get_prizes" };
      this.conn.send(JSON.stringify(action));
    },

    webSocketOnMessage(msg) {
      var res = JSON.parse(msg.data);
      console.log(res);
      if (res.success === true) {
        this.prizes = res.prizes;
        if (res.action.name === "start") {
          this.started = true;
        } else if (res.action.name === "stop") {
          this.started = false;
        }
      }
    },

    selectPrize(index) {
      this.prizeIndex = index;
      console.log("prize: " + index + "selected");
    },

    startStop() {
      var action = {};
      if (!this.started) {
        action.name = "start";
      } else {
        action.name = "stop";
      }
      action.prize_index = this.prizeIndex;
      console.log(action);
      this.conn.send(JSON.stringify(action));
    }
  }
};
</script>
