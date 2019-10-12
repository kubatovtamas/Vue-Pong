<template>
  <div id="app">
      <div class="menu" v-bind:class="mainIsActive ? 'showElement' : 'hideElement' ">
        <div class="kozep">
          <h1>PONG</h1>

          <button v-on:click="setPlay()" id="playButton">Play</button>

          <div class="text">
            Score to:
          </div>
          <button v-on:click="setWinningScore(5)" class="fixwbutton" id="winc-5">5</button>
          <button v-on:click="setWinningScore(10)" class="fixwbutton" id="winc-10">10</button>
          <button v-on:click="setWinningScore(1000)" class="fixwbutton" id="winc-free">Infinite</button>
          
          <div class="text">
            Difficulty:
          </div>
          <button v-on:click="setDifficulty(3)" class="fixwbutton" id="ai-speed-1">Easy</button>
          <button v-on:click="setDifficulty(5)" class="fixwbutton" id="ai-speed-2">Hard</button>
        </div>
      </div>
      <canvas class="game" id="gameCanvas" v-bind:class="play ? 'showElement' : 'hideElement' " width="800" height="600"></canvas>
  </div>
</template>

<script>

const PADDLE_HEIGHT = 100;
const PADDLE_THICKNESS = 10;
const PADDLE_SPEED_PLAYER = 20;
const PADDLE_BUFFER = 20;
const PADDLE_OFFSET = 40;
const FRAMES_PER_SECOND = 50;

export default {
  name: 'app',
  data() {
    return {
      canvas : "",
      canvasContext : "",
      framesPerSecond : FRAMES_PER_SECOND,

      ballX : 395,
      ballY : 295,
      ballSpeedX : 5,
      ballSpeedY : 3,
      deltaY : 0,
      modY : 0.15,

      mainIsActive : true,
      play : false,
      showingPause : true,
      winningScore : 5,
      paddleSA : 3,
      paddleSP : PADDLE_SPEED_PLAYER,
      paddleH : PADDLE_HEIGHT,
      paddleT : PADDLE_THICKNESS,
      paddleB : PADDLE_BUFFER,
      paddleO : PADDLE_OFFSET,
      paddle1Y : 250,
      paddle2Y : 250,

      player1Score : 0,
      player2Score : 0,
    }
  },

  methods: {
    // Setters
    setWinningScore(newScore) {
      this.winningScore = newScore;
    },

    setDifficulty(newAISpeed) {
      this.paddleSA = newAISpeed;
    },

    setPlay() {
      this.play = true;
      this.mainIsActive = false;
    },

    setMain() {
      this.play = false;
      this.mainIsActive = true;
      this.setPause();
    },

    setPause() {
      this.showingPause = true;
      this.paddle1Y = 250;
      this.paddle2Y = 250;
      this.player1Score = 0;
      this.player2Score = 0;
      this.ballReset();
    },

    setResume() {
      this.showingPause = false;
    },

    //Drawers
    colorRect(leftX, topY, width, height, drawColor) {
      this.canvasContext.fillStyle = drawColor;
      this.canvasContext.fillRect(leftX, topY, width, height);
    },

    drawNet() {
      for (var i = -10; i < this.canvas.height + 40; i+=40) {
        this.colorRect(this.canvas.width/2-1, i, 2, 20, 'white');
      }
    },

    drawEverything() {
      this.colorRect(0,0,this.canvas.width, this.canvas.height, 'black'); // Background
      this.colorRect(this.paddleO, this.paddle1Y, this.paddleT, this.paddleH, 'white'); // Left paddle
      this.colorRect(this.canvas.width - this.paddleT - this.paddleO, this.paddle2Y, this.paddleT, this.paddleH, 'white'); // Right paddle
      this.colorRect(this.ballX, this.ballY, 10, 10, 'white'); // Ball
      this.canvasContext.fillText(this.player1Score, 100, 100); // Score 1
      this.canvasContext.fillText(this.player2Score, this.canvas.width - 100, 100); // Score 2
      this.drawNet();
      if (this.showingPause) { 
        if(this.player1Score >= this.winningScore) {
          this.canvasContext.fillText("Left player won!", 360, 150);
        } else if (this.player2Score >= this.winningScore) {
          this.canvasContext.fillText("Right player won!", 360, 150);
        }
        this.canvasContext.fillText("Press Y to play!", 360, 250);
      }
    },

    // Movers
    ballReset() {
      if (this.player1Score >= this.winningScore || this.player2Score >= this.winningScore) {
        this.setPause()
      }
      this.ballX = this.canvas.width / 2 - 5;
      this.ballY = this.canvas.height / 2 - 5;
      this.ballSpeedX = -this.ballSpeedX;
      this.ballSpeedY = -this.ballSpeedY * 0.5;
    },

    computerMovement() {
      var paddle2Center = this.paddle2Y + (this.paddleH / 2);
      if (paddle2Center < this.ballY - this.paddleB) {
        this.paddle2Y += this.paddleSA;
      } else if (paddle2Center > this.ballY + this.paddleB) {
        this.paddle2Y -= this.paddleSA;
      }
    },

    movePaddleUp() {
      if (this.paddle1Y >= 0) {
        this.paddle1Y = this.paddle1Y - this.paddleSP;
      }
    },

    movePaddleDown() {
      if (this.paddle1Y + this.paddleH <= this.canvas.height) {
        this.paddle1Y = this.paddle1Y + this.paddleSP;
      }
    },
    
    moveEverything() {
      if (this.showingPause) {
        return;
      }

      this.computerMovement();
      this.ballX += this.ballSpeedX;
      this.ballY += this.ballSpeedY;

      // Left side
      if (this.ballX <= this.paddleO + this.paddleT &&
          this.ballY >= this.paddle1Y &&
          this.ballY <= this.paddle1Y + this.paddleH) {
            // Paddle hit
            this.ballSpeedX = -this.ballSpeedX;
            this.deltaY = this.ballY-(this.paddle1Y + this.paddleH / 2);
            this.ballSpeedY = this.deltaY * this.modY;}
      if (this.ballX <= 0){
            // Score
            this.player2Score++;
            this.ballReset();
      }
      
      // Right side
      if (this.ballX >= this.canvas.width - this.paddleO - this.paddleT &&
          this.ballY >= this.paddle2Y &&
          this.ballY <= this.paddle2Y + this.paddleH) {
            // Paddle hit
            this.ballSpeedX = -this.ballSpeedX;
            this.deltaY = this.ballY-(this.paddle2Y + this.paddleH / 2);
            this.ballSpeedY = this.deltaY * this.modY;}
      if (this.ballX >= this.canvas.width) {
            // Score
            this.player1Score++;
            this.ballReset();
        }
      

      // Bottom
      if(this.ballY < 0) {
        this.ballSpeedY = -this.ballSpeedY
      }

      // Top
      if(this.ballY > this.canvas.height) {
        this.ballSpeedY = -this.ballSpeedY
      }
    },

    //Drivers
    callBoth() {
      this.moveEverything();
      this.drawEverything();
    },

    Control(e) {
      let cmd = String.fromCharCode(e.keyCode).toLowerCase();
      if (cmd == "s") {
        this.movePaddleDown();
      }
      if (cmd == "w") {
        this.movePaddleUp();
      }
      if (cmd == "y"){
        this.setResume();
      }
      if (cmd == "m") {
        this.setMain(); 
      }
      if (cmd == "p") {
        this.setPlay();
      }
    },
  },
  mounted() {
    this.canvas = document.getElementById('gameCanvas');
    this.canvasContext = this.canvas.getContext('2d');
    setInterval(this.callBoth, 1000 / this.framesPerSecond);
  },

  created() {
    window.addEventListener('keypress', this.Control);
  },

  destroyed() {
    window.removeEventListener('keypress', this.Control);
  }
}
</script>

<style scoped>
#app {
  display: block;
  font-family: 'Aldrich', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: black;
  margin-top: 60px;
}
.menu {
  background-color: black;
  color: white;
  width: 800px;
  height: 600px;
  margin: auto;
}
.kozep {
	padding: 50px 0;
	text-align: center;
}
button {
	padding: 0 25px;
	background: black;
  color: white;
	border: none;
	border-radius: 0px;
	height: 50px;
	line-height: 50px;
  font-family: 'Aldrich', sans-serif;
  font-size: 15px;
}
.fixwbutton {
	width: 100px;
}
.text {
	margin: 50px 0;
	font-size: 25px;
}
.showElement {
  display: block;
}
.hideElement {
  display: none;
}
.game {
  margin: auto;
}
</style>

