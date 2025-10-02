<template>
  <div class="container">
    <div class="logo-container">
      <img src="./assets/logo.png" alt="Tabooooosiness" class="game-logo" />
    </div>
    
    <!-- Player Setup -->
    <div v-if="gameState === 'setup'" class="player-setup">
      <h2>Configurazione Team</h2>
      
      <div class="player-count">
        <label>Numero di team:</label>
        <div class="player-count-buttons">
          <button 
            v-for="count in [2, 3, 4, 5]" 
            :key="count"
            class="player-count-btn"
            :class="{ active: playerCount == count }"
            @click="selectPlayerCount(count)"
          >
            <div class="player-count-icon">👥</div>
            <div class="player-count-number">{{ count }}</div>
          </button>
        </div>
      </div>
      
      <div class="player-inputs">
        <div v-for="i in parseInt(playerCount)" :key="i" class="player-input">
          <label :for="`player${i}`">Team {{ i }}:</label>
          <input 
            :id="`player${i}`"
            v-model="players[i-1].name" 
            type="text" 
            :placeholder="`Nome team ${i}`"
            @input="validatePlayers"
          />
        </div>
      </div>
      
      <div class="timer-setting">
        <label>Timer (secondi):</label>
        <div class="timer-options">
          <button 
            @click="setTimerDuration(0)"
            :class="['timer-btn', { active: initialTimerSeconds === 0 }]"
          >
            <div class="timer-icon">🚫</div>
            <div class="timer-text">Disabilitato</div>
          </button>
          <button 
            v-for="seconds in [15, 30, 45, 60]" 
            :key="seconds"
            @click="setTimerDuration(seconds)"
            :class="['timer-btn', { active: initialTimerSeconds === seconds }]"
          >
            <div class="timer-icon">⏱️</div>
            <div class="timer-text">{{ seconds }}s</div>
          </button>
        </div>
      </div>
      
      <button 
        class="btn" 
        @click="startGame" 
        :disabled="!canStartGame"
      >
        Inizia Partita
      </button>
    </div>
    
    <!-- Game -->
    <div v-else-if="gameState === 'playing'">
      <div class="current-player" :class="{ 'timer-critical': isTimerCritical }">
        <div class="player-info">
          <div class="player-name">Turno di: {{ currentPlayer.name }}</div>
          <div class="cards-progress">
            Carte indovinate: {{ cardsGuessedThisTurn }}/{{ cardsPerTurn }}
            <span class="cards-skipped"> • Saltate: {{ cardsNotGuessed }}</span>
          </div>
        </div>
        <div v-if="initialTimerSeconds > 0" class="timer-container">
          <div class="timer-display">{{ formatTime(timeLeft) }}</div>
          <div class="timer-label">Tempo rimanente</div>
        </div>
      </div>
      
      <div class="scoreboard">
        <div 
          v-for="(player, index) in players" 
          :key="index"
          class="player-score"
          :class="{ active: index === currentPlayerIndex }"
        >
          <h3>{{ player.name }}</h3>
          <div class="score">{{ player.score }}</div>
        </div>
      </div>
      
      <div v-if="currentCard" class="game-card-container">
        <div v-if="isCardBlurred" class="countdown-overlay">
          <div class="countdown-number">{{ countdown }}</div>
          <div class="countdown-text">Preparati...</div>
        </div>
        <div class="game-card" :class="{ 'card-blurred': isCardBlurred, 'card-flash': isCardFlashing }">
          <div class="target-word">{{ currentCard.targetWord }}</div>
          <div class="taboo-words">
            <span 
              v-for="(word, index) in currentCard.tabooWords" 
              :key="index"
              class="taboo-word"
            >
              {{ word }}
            </span>
          </div>
        </div>
      </div>
      
      <div class="game-controls">
        <button class="btn btn-secondary btn-square" @click="correctGuess" :disabled="isCardBlurred">
          <div class="btn-icon">✅</div>
          <div class="btn-text">Parola Indovinata (+10)</div>
        </button>
        <button class="btn btn-danger btn-square" @click="tabooWordMentioned" :disabled="isCardBlurred">
          <div class="btn-icon">❌</div>
          <div class="btn-text">Parola Taboo (-1)</div>
        </button>
        <button class="btn btn-square" @click="nextTurn" :disabled="isCardBlurred">
          <div class="btn-icon">⏭️</div>
          <div class="btn-text">Prossimo Turno</div>
        </button>
        <button class="btn btn-square" @click="skipCard" :disabled="isCardBlurred">
          <div class="btn-icon">🎴</div>
          <div class="btn-text">Nuova Carta</div>
        </button>
      </div>
      
      <div class="game-controls">
        <button class="btn btn-info" @click="showHelpDialog = true">
          ❓ Come Giocare
        </button>
      </div>
      
      <!-- Help Dialog -->
      <div v-if="showHelpDialog" class="dialog-overlay" @click="showHelpDialog = false">
        <div class="dialog" @click.stop>
          <div class="dialog-header">
            <h3>Come Giocare</h3>
            <button class="dialog-close" @click="showHelpDialog = false">×</button>
          </div>
          <div class="dialog-content">
            <p>
              <strong>{{ currentPlayer.name }}</strong> deve far indovinare la parola <strong>{{ currentCard.targetWord }}</strong> 
              senza usare le parole taboo mostrate sopra. Gli altri giocatori devono indovinare la parola.
            </p>
            <p>
              <strong>🎯 Turni:</strong> Ogni team deve indovinare <strong>3 carte</strong> prima che tocchi al team successivo.
            </p>
            <p>
              Clicca "Parola Indovinata" per +10 punti, "Parola Taboo" per -1 punto.
            </p>
            <p>
              <strong>⏱️ Timer:</strong> Se abilitato, ogni turno ha un tempo limite. Quando il tempo scade:
            </p>
            <ul>
              <li><strong>Prima volta:</strong> -2 punti, timer si riavvia con metà tempo</li>
              <li><strong>Seconda volta:</strong> -4 punti, timer si riavvia con 1/4 del tempo</li>
              <li><strong>Volte successive:</strong> -4 punti, timer continua con 1/4 del tempo</li>
            </ul>
            <p>
              <strong>🚨 Attenzione:</strong> Quando il timer è critico (ultima fase), la barra del turno diventa rossa e lampeggia!
            </p>
          </div>
        </div>
      </div>
    </div>
    
    <!-- Game Over -->
    <div v-else-if="gameState === 'finished'">
      <div class="winner">
        <h2>🎉 Partita Finita!</h2>
        <h3>Vincitore: {{ winner.name }}</h3>
        <p>Punteggio finale: {{ winner.score }}</p>
      </div>
      
      <div class="scoreboard">
        <div 
          v-for="(player, index) in sortedPlayers" 
          :key="index"
          class="player-score"
        >
          <h3>{{ player.name }}</h3>
          <div class="score">{{ player.score }}</div>
        </div>
      </div>
      
      <button class="btn" @click="resetGame">
        🔄 Nuova Partita
      </button>
    </div>
  </div>
</template>

<script>
import wordsData from '../words.json'

export default {
  name: 'App',
  data() {
    return {
      gameState: 'setup', // 'setup', 'playing', 'finished'
      playerCount: '2',
      players: [
        { name: '', score: 0 },
        { name: '', score: 0 }
      ],
      currentPlayerIndex: 0,
      currentCard: null,
      usedCards: new Set(),
      words: [],
      isCardBlurred: false,
      countdown: 5,
      countdownInterval: null,
      showHelpDialog: false,
      isCardFlashing: false,
      // Timer settings
      initialTimerSeconds: 30, // x seconds - can be configured, default 30
      timeLeft: 60,
      timerInterval: null,
      timerPhase: 1, // 1, 2, or 3
      // Turn management
      cardsGuessedThisTurn: 0,
      cardsPerTurn: 3,
      cardsNotGuessed: 0
    }
  },
  computed: {
    canStartGame() {
      return this.players.every(player => player.name.trim() !== '')
    },
    currentPlayer() {
      return this.players[this.currentPlayerIndex]
    },
    isTimerCritical() {
      // Critical when timer is in phase 3 (x/4 seconds) or when time is very low
      return this.timerPhase >= 3 || (this.initialTimerSeconds > 0 && this.timeLeft <= Math.floor(this.initialTimerSeconds / 4))
    },
    winner() {
      return this.players.reduce((prev, current) => 
        prev.score > current.score ? prev : current
      )
    },
    sortedPlayers() {
      return [...this.players].sort((a, b) => b.score - a.score)
    }
  },
  mounted() {
    this.loadWords()
  },
  methods: {
    loadWords() {
      // Extract words from the JSON structure
      this.words = wordsData.words || []
    },
    updatePlayerInputs() {
      const count = parseInt(this.playerCount)
      this.players = Array.from({ length: count }, (_, i) => ({
        name: this.players[i]?.name || '',
        score: 50
      }))
    },
    selectPlayerCount(count) {
      this.playerCount = count.toString()
      this.updatePlayerInputs()
    },
    setTimerDuration(seconds) {
      this.initialTimerSeconds = seconds
      this.timeLeft = seconds
    },
    validatePlayers() {
      // This method is called on input to ensure real-time validation
    },
    startGame() {
      if (!this.canStartGame) return
      
      this.gameState = 'playing'
      this.currentPlayerIndex = 0
      this.usedCards.clear()
      this.cardsGuessedThisTurn = 0
      this.cardsNotGuessed = 0
      this.newCard()
    },
    newCard() {
      if (this.words.length === 0) {
        alert('Nessuna carta disponibile!')
        return
      }
      
      // Clear any existing intervals before starting new card
      this.clearAllIntervals()
      
      // Get a random card that hasn't been used
      let availableCards = this.words.filter(word => !this.usedCards.has(word.id))
      
      // If all cards have been used, reset the used cards
      if (availableCards.length === 0) {
        this.usedCards.clear()
        availableCards = this.words
      }
      
      const randomIndex = Math.floor(Math.random() * availableCards.length)
      this.currentCard = availableCards[randomIndex]
      this.usedCards.add(this.currentCard.id)
      
      // Start countdown animation
      this.startCountdown()
    },
    startCountdown() {
      this.isCardBlurred = true
      this.countdown = 5
      
      this.countdownInterval = setInterval(() => {
        this.countdown--
        if (this.countdown <= 0) {
          this.isCardBlurred = false
          clearInterval(this.countdownInterval)
          this.countdownInterval = null
          // Start the turn timer after card is revealed
          this.startTurnTimer()
        }
      }, 1000)
    },
    startTurnTimer() {
      // Don't start timer if disabled
      if (this.initialTimerSeconds === 0) {
        return
      }
      
      // Reset timer for new turn
      this.timerPhase = 1
      this.timeLeft = this.initialTimerSeconds
      
      this.timerInterval = setInterval(() => {
        this.timeLeft--
        
        if (this.timeLeft <= 0) {
          this.handleTimerExpired()
        }
      }, 1000)
    },
    handleTimerExpired() {
      clearInterval(this.timerInterval)
      
      // Flash the card when timer expires
      this.flashCard()
      
      if (this.timerPhase === 1) {
        // First timeout: -2 points, restart with x/2 seconds
        this.currentPlayer.score -= 2
        this.timerPhase = 2
        this.timeLeft = Math.floor(this.initialTimerSeconds / 2)
        this.timerInterval = setInterval(() => {
          this.timeLeft--
          if (this.timeLeft <= 0) {
            this.handleTimerExpired()
          }
        }, 1000)
      } else if (this.timerPhase === 2) {
        // Second timeout: -4 points, restart with x/4 seconds
        this.currentPlayer.score -= 4
        this.timerPhase = 3
        this.timeLeft = Math.floor(this.initialTimerSeconds / 4)
        this.timerInterval = setInterval(() => {
          this.timeLeft--
          if (this.timeLeft <= 0) {
            this.handleTimerExpired()
          }
        }, 1000)
      } else {
        // Third timeout: -4 points, restart with x/4 seconds (continues)
        this.currentPlayer.score -= 4
        this.timeLeft = Math.floor(this.initialTimerSeconds / 4)
        this.timerInterval = setInterval(() => {
          this.timeLeft--
          if (this.timeLeft <= 0) {
            this.handleTimerExpired()
          }
        }, 1000)
      }
    },
    stopTurnTimer() {
      if (this.timerInterval) {
        clearInterval(this.timerInterval)
        this.timerInterval = null
      }
    },
    formatTime(seconds) {
      const mins = Math.floor(seconds / 60)
      const secs = seconds % 60
      return `${mins}:${secs.toString().padStart(2, '0')}`
    },
    flashCard() {
      this.isCardFlashing = true
      setTimeout(() => {
        this.isCardFlashing = false
      }, 500) // Flash for 500ms
    },
    clearAllIntervals() {
      if (this.countdownInterval) {
        clearInterval(this.countdownInterval)
        this.countdownInterval = null
      }
      if (this.timerInterval) {
        clearInterval(this.timerInterval)
        this.timerInterval = null
      }
    },
    skipCard() {
      this.cardsNotGuessed++
      this.stopTurnTimer()
      this.newCard()
    },
    correctGuess() {
      this.currentPlayer.score += 10
      this.cardsGuessedThisTurn++
      this.stopTurnTimer()
      
      // Check if team has guessed 3 cards
      if (this.cardsGuessedThisTurn >= this.cardsPerTurn) {
        // Turn complete, move to next team
        this.cardsGuessedThisTurn = 0
        this.currentPlayerIndex = (this.currentPlayerIndex + 1) % this.players.length
      }
      
      this.newCard()
    },
    tabooWordMentioned() {
      this.currentPlayer.score -= 1
      // Don't change the card - let the player try again with the same word
    },
    nextTurn() {
      this.stopTurnTimer()
      this.cardsGuessedThisTurn = 0
      this.currentPlayerIndex = (this.currentPlayerIndex + 1) % this.players.length
      this.newCard()
    },
    resetGame() {
      this.gameState = 'setup'
      this.players.forEach(player => {
        player.score = 50
      })
      this.currentPlayerIndex = 0
      this.currentCard = null
      this.usedCards.clear()
      this.isCardBlurred = false
      this.countdown = 5
      this.timeLeft = this.initialTimerSeconds
      this.timerPhase = 1
      this.cardsGuessedThisTurn = 0
      this.cardsNotGuessed = 0
      
      this.clearAllIntervals()
    }
  }
}
</script>

