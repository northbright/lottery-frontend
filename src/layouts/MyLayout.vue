<template>
  <q-layout view="lHh Lpr lff">
    <q-header elevated>
      <q-toolbar class="bg-pink-3 text-black">
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
      <div class="q-pa-md q-gutter-xl fit row wrap justify-around">
        <q-btn
          rounded
          color="pink-3"
          text-color="black"
          type="submit"
          size="20px"
          label="start/stop"
          @click="startStop()"
        />
      </div>
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

    <q-page-container>
      <div
        class="q-gutter-xl fit row wrap justify-around content-center bg-pink-1"
      >
        <q-btn
          color="deep-orange"
          v-for="(winner, index) in this.winners"
          :label="winner.name"
          :key="index"
          :size="fontSize"
          @click="selectWinner(index)"
        ></q-btn>
      </div>
    </q-page-container>
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
      oldWinnerIndexes: [],
      url: "ws://192.168.1.40:8081/ws",
      conn: {},
      started: false,
      fontSize: "35px"
    };
  },

  created() {
    console.log("created()");
    this.initWebSocket();
  },

  mounted() {
    var _this = this;
    document.onkeydown = function(e) {
      let key = e.keyCode;
      if (key == 13) {
        window.event.preventDefault();
        _this.startStop();
      }
    };
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
        var action = res.action;

        switch (action.name) {
          case "get_prizes":
            this.prizes = res.prizes;
            break;

          case "get_winners":
            if (res.winners.length === 0) {
              this.winners = [];
              var prizeNum = this.prizes[this.prizeIndex].num;

              for (var i = 0; i < this.prizes[this.prizeIndex].num; i++) {
                this.winners[i] = { id: "", name: "?" };
              }
            } else {
              this.winners = res.winners;
            }

            if (prizeNum >= 50) {
              this.fontSize = "50px";
            } else if (prizeNum >= 20) {
              this.fontSize = "80px";
            } else if (prizeNum >= 10) {
              this.fontSize = "90px";
            } else if (prizeNum >= 5) {
              this.fontSize = "100px";
            } else if (prizeNum >= 2) {
              this.fontSize = "110px";
            } else {
              this.fontSize = "120px";
            }

            break;

          case "start":
            this.started = true;
            this.winners = res.winners;
            break;

          case "stop":
            this.started = false;
            this.winners = res.winners;
            break;
        }
      }
    },

    selectPrize(index) {
      this.prizeIndex = index;
      console.log("prize: " + index + "selected");
      var action = { name: "get_winners", prize_index: index };
      this.conn.send(JSON.stringify(action));
    },

    /*
    selectWinner(index) {
      var idx = this.oldWinnerIndexes.indexOf(index);

      if (idx != -1) {
        delete this.oldWinnerIndexes[id];
      } else {
        this.oldWinnerIndexes.push(index);
      }
    },
*/
    startStop() {
      var action = {};

      if (!this.started) {
        action.name = "start";
      } else {
        action.name = "stop";
      }

      this.started = !this.started;

      action.prize_index = this.prizeIndex;
      console.log(action);
      this.conn.send(JSON.stringify(action));
    }
  }
};
</script>
