<template>
  <body>
    <header>
      <h1>Monster Slayer</h1>
    </header>
    <div id="game">
      <section id="monster" class="container">
        <h2>Monster Health</h2>
        <div class="healthbar">
          <div class="healthbar__value" :style="monsterBarStyles"></div>
        </div>
      </section>
      <section id="player" class="container">
        <h2>Your Health</h2>
        <div class="healthbar">
          <div class="healthbar__value" :style="playerBarStyles"></div>
        </div>
      </section>
      <section v-if="winner" class="container">
        <h2>Game Over!</h2>
        <h3 v-if="winner === 'monster'">You Lost!</h3>
        <h3 v-else-if="winner === 'player'">You Won!</h3>
        <h3 v-else>It's a draw!</h3>
        <button @click="startGame">Start New Game</button>
      </section>
      <section v-else id="controls">
        <button @click="attackMonster">ATTACK</button>
        <button :disabled="mayUseSpecialAttack" @click="specialAttackMonster">SPECIAL ATTACK</button>
        <button @click="healPlayer">HEAL</button>
        <button @click="surrender">SURRENDER</button>
      </section>
      <section id="log" class="container">
        <h2>Battle Log</h2>
        <ul>
          <li v-for="(logMessage, index) in logMessages" :key="index">
            <!-- {{ logMessage.actionBy }} - {{ logMessage.actionType }} - {{ logMessage.actionValue }} -->
            <span :class="{'log--player': logMessage.actionBy === 'player', 'log--monster': logMessage.actionBy === 'monster'}">
              {{ logMessage.actionBy === "player" ? "Player" : "Monster" }}
            </span>
            <span v-if="logMessage.actionType === 'heal'"> heals himself for <span class="log--heal">{{ logMessage.actionValue }}</span></span>
            <span v-else> attacks and deals <span class="log--damage">{{ logMessage.actionValue }}</span></span>
          </li>
        </ul>
      </section>
    </div>
  </body>
</template>

<script>

export default {
  name: 'App',
  components: {
    
  },
  data() {
    return {
      playerHealth: 100,
      monsterHealth: 100,
      currentRound: 0,
      winner: null,
      logMessages: []
    }
  },
  watch: {
    playerHealth(value) {
      if(value <= 0 && this.monsterHealth <= 0) {
        // 무승부
        this.winner = "draw";
      } else if(value <= 0) {
        // 플레이어 패
        this.winner = "monster";
      }
    },
    monsterHealth(value) {
      if(value <= 0 && this.playerHealth <= 0) {
        // 무승부
        this.winner = "draw";
      } else if(value <= 0) {
        // 몬스터 패
        this.winner = "player";
      }
    }
  },
  methods: {
    attackMonster() { 
      this.currentRound++;

      // (플레이어 -> 몬스터) 공격시 몬스터 체력 피해량
      // 최솟값 5, 최댓값 12
      const attackValue = Math.floor(Math.random() * (12 - 5)) + 5;
      this.monsterHealth -= attackValue;

      this.addLogMessage("player", "attack", attackValue);
      this.attackPlayer();
    },
    attackPlayer() {
      // (몬스터 -> 플레이어) 공격시 플레이어 체력 피해량
      // 최솟값 8, 최댓값 15
      const attackValue = Math.floor(Math.random() * (15 - 8)) + 8;
      this.playerHealth -= attackValue;

      this.addLogMessage("monster", "attack", attackValue);
    },
    specialAttackMonster() { // currentRound가 3의 배수일 때만 사용 가능
      this.currentRound++;

      // (플레이어 -> 몬스터) 특수 공격시 몬스터 체력 피해량
      // 최솟값 10, 최댓값 25
      const attackValue = Math.floor(Math.random() * (25 - 10)) + 10;
      this.monsterHealth -= attackValue;

      this.addLogMessage("player", "special-attack", attackValue);
      this.attackPlayer();
    },
    healPlayer() {
      this.currentRound++;

      // 플레이어 체력 추가량
      // 최솟값 8, 최댓값 20
      const healValue = Math.floor(Math.random() * (20 - 8)) + 8;

      // 플레이어 체력 100 초과 불가능
      if(this.playerHealth + healValue > 100) {
        this.playerHealth = 100;
      } else {
        this.playerHealth += healValue;
      }

      this.addLogMessage("player", "heal", healValue);
      this.attackPlayer();
    },
    startGame() {
      // 게임 재시작 위해 모든 데이터값을 초기값으로 다시 셋팅
      this.playerHealth = 100,
      this.monsterHealth = 100,
      this.currentRound = 0,
      this.winner = null,
      this.logMessages = []
    },
    surrender() {
      // 플레이어가 항복하므로 승자는 몬스터
      this.winner = "monster";
    },
    addLogMessage(who, what, value) {
      // .push()는 배열의 맨 마지막에 새로운 항목 추가
      // .unshift()는 배열의 맨 처음에 새로운 항목 추가
      this.logMessages.unshift({
        actionBy: who,
        actionType: what,
        actionValue: value
      });
    }
  },
  computed: {
    monsterBarStyles() {
      if(this.monsterHealth < 0) {
        return {width: '0%'};
      }
      return {width: this.monsterHealth + '%'};
    },
    playerBarStyles() {
      if(this.playerHealth < 0) {
        return {width: '0%'};
      }
      return {width: this.playerHealth + '%'};
    },
    mayUseSpecialAttack() {
      return this.currentRound % 3 !== 0;
    }
  }
}
</script>

<style>
* {
  box-sizing: border-box;
}

html {
  font-family: 'Jost', sans-serif;
}

body {
  margin: 0;
}

header {
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.26);
  padding: 0.5rem;
  background-color: #880017;
  color: white;
  text-align: center;
  margin-bottom: 2rem;
}

section {
  width: 90%;
  max-width: 40rem;
  margin: auto;
}

.healthbar {
  width: 100%;
  height: 40px;
  border: 1px solid #575757;
  margin: 1rem 0;
  background: #fde5e5;
}

.healthbar__value {
  background-color: #00a876;
  width: 100%;
  height: 100%;
}

.container {
  text-align: center;
  padding: 0.5rem;
  margin: 1rem auto;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.26);
  border-radius: 12px;
}

#monster h2,
#player h2 {
  margin: 0.25rem;
}

#controls {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
}

button {
  font: inherit;
  border: 1px solid #88005b;
  background-color: #88005b;
  color: white;
  padding: 1rem 2rem;
  border-radius: 12px;
  margin: 1rem;
  width: 12rem;
  cursor: pointer;
  box-shadow: 1px 1px 4px rgba(0, 0, 0, 0.26);
}

button:focus {
  outline: none;
}

button:hover,
button:active {
  background-color: #af0a78;
  border-color: #af0a78;
  box-shadow: 1px 1px 8px rgba(0, 0, 0, 0.26);
}

button:disabled {
  background-color: #ccc;
  border-color: #ccc;
  box-shadow: none;
  color: #3f3f3f;
  cursor: not-allowed;
}

#log ul {
  list-style: none;
  margin: 0;
  padding: 0;
}

#log li {
  margin: 0.5rem 0;
}

.log--player {
  color: #7700ff;
}

.log--monster {
  color: #da8d00;
}

.log--damage {
  color: red;
}

.log--heal {
  color: green;
}
</style>
