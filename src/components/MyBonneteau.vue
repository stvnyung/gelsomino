<template>
  <div>
    <div v-show="status" :class="{win: isWin, lost: isLost}" class="msg">
      {{ endGameMsg }}
    </div>

    <div id="shells">
      <div v-for="shell in shells" :key="shell.id" class="shell">
        <shell
          @display="display(...arguments)"
          @endGame="finishGame(...arguments)"
          :id="shell.id"
          :has-pearl="shell.id === perlPosition"
          :display-shell="shell.displayShell"
        >
        </shell>
      </div>
    </div>

    <button @click="tryAgain" v-show="this.isEndGame">Try Again</button>
  </div>
</template>

<script>
import axios from 'axios';
import Shell from './Shell.vue';

const PLAYING = 0;
const ENDGAME = 1;

export default {
  name: 'MyBonneteau',
  mounted() {
    this.shuffle();
  },
  data() {
    return {
      perlPosition: 0,
      shells: [
        { id: 0, displayShell: true },
        { id: 1, displayShell: true },
        { id: 2, displayShell: true },
      ],
      status: PLAYING,
      endGameStatus: ''
    };
  },
  methods: {
    shuffle() {
      axios.get('https://www.random.org/integers/?num=1&min=0&max=2&col=1&base=10&format=plain&rnd=new')
        .then((response) => {
          this.perlPosition = response.data;
        })
        .catch((error) => {
          console.error(error);
        });
    },
    finishGame(endGameStatus) {
      if (this.status != PLAYING) {
        return;
      }

      this.status = ENDGAME;
      this.endGameStatus = endGameStatus;
    },
    displayAllShells() {
      this.shells.forEach((shell) => {
        shell.displayShell = true;
      });
    },
    tryAgain() {
      this.status = PLAYING;
      this.shuffle();
      this.displayAllShells();
    },
    display(id) {
      this.shells.forEach((shell) => {
        if (shell.id === id) {
          shell.displayShell = false;
        }
      });
    }
  },
  computed: {
    isEndGame() {
      return this.status === ENDGAME;
    },
    isLost() {
      return this.isEndGame && this.endGameStatus === 'lost';
    },
    isWin() {
      return this.isEndGame && this.endGameStatus === 'win';
    },
    endGameMsg() {
      return this.isLost ? 'LOST :(' : 'WIN !!!!!!!';
    }
  },
  components: { Shell }
}
</script>

<style scoped>
#shells {
  width: 100%;
}
.shell {
  width: 33%;
  display: inline-block;
}
.msg {
  width: 100%;
  height: 150px;
}
.win {
  background-color: green;
}
.lost {
  background-color: red;
}
</style>
