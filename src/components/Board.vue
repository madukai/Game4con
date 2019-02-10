<template>
    <div id="game">
        <div class="game_info">
          <span>Captain's Mistress Game</span>
        </div>
        <div class="container">
            <div class="column" @click="addPieceToColumn($event, 0)">
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
            </div>
            <div class="column" @click="addPieceToColumn($event, 1)">
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
            </div>
            <div class="column" @click="addPieceToColumn($event, 2)">
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
            </div>
            <div class="column" @click="addPieceToColumn($event, 3)">
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
            </div>
            <div class="column" @click="addPieceToColumn($event, 4)">
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
            </div>
            <div class="column" @click="addPieceToColumn($event, 5)">
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
            </div>
            <div class="column" @click="addPieceToColumn($event, 6)" >
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
                <div class="cell"><div class="piece"></div></div>
            </div>
        </div>
        <div class="alert">
          <span class="closebtn" @click="closeMSG($event)">New Game</span> 
          {{gameMsg}}
        </div>
    </div>
</template>

<script>
export default {
  name: "Board",
  data() {
    return {
      board: [],
      columnCount: 7,
      rowCount: 6,
      playerTurn: 1,
      winCount: 3,
      timeoutID: 0,
      gameMsg: '',
      gameEnd: 0 // flag to determine if the game already end
    };
  },
  beforeMount() {
    this.resetGame();
  },
  methods: {
    resetGame: function () {
      // Empty board
      this.board = [];

      let emptyColumn = [];
      for (let y = 0; y < this.rowCount; y++) {
        emptyColumn[y] = 0;
      }

      // Push columns with empty values
      for (let x = 0; x < this.columnCount; x++) {
        this.board.push(emptyColumn.slice(0));
      }

      let pieces = document.getElementsByClassName("piece");

      // Reset pieces
      for (let x = 0; x < pieces.length; x++) {
        pieces[x].className = "piece";
      }

      this.playerTurn = 1;

      this.gameEnd = 0;

      // clear timeout id
      clearTimeout(this.timeoutID);
    },
    addPieceToColumn(e, column) {

      // if the game already end prevent any actions, until the game resets
      if (this.gameEnd === 1) {
        return;
      }

      let cells = "";
      if (e.target.parentElement.className === "column") {
        cells = e.target.parentElement.children;
      } else {
        cells = e.target.parentElement.parentElement.children;
      }

      let spot = this.getOpenSpotInColumn(column);

      if (spot === null) {
        //unable to add piece to full column
        return;
      }

      this.board[spot.column][spot.row] = this.playerTurn;

      cells[spot.row].children[0].classList.add(
        this.playerTurn === 1 ? "red" : "blue"
      );

      // check for winner
      if (this.checkWinner()) { // If true
        // exit out of function to avoid switch
        return;
      }

      if (this.playerTurn === 1) {
        this.playerTurn = 2; //2nd player's turn next
        // call the timeout to give the AI 1.5 secs to think
        this.timeoutID = setTimeout(() => {
          this.aiPlayer();
        }, 1500);
      } else {
        this.playerTurn = 1; //1st player's turn next
      }
    },
    getOpenSpotInColumn(column) {
      for (let row = this.rowCount - 1; row >= 0; row--) {
        if (this.board[column][row] === 0) {
          //found an open spot
          return {
            column: column,
            row: row
          };
        }
      }

      //column is full
      return null;
    },
    aiPlayer() { // AI logic is different since it will not have the click event
      // Get a random number the will act as AI's choice
      let targetCol = this.getAIChoice(this.rowCount);
      // Get the DOM info for the target column chosen by AI
      let colDomData = document.getElementsByClassName("column")[targetCol];

      let spot = this.getOpenSpotInColumn(targetCol);

      if(this.gameEnd === 1) {
        return;
      }

      if (spot === null) {
        //unable to add piece to full column
        return;
      }

      this.board[spot.column][spot.row] = this.playerTurn;

      colDomData.children[spot.row].children[0].classList.add(
        this.playerTurn === 1 ? "red" : "blue"
      );

      // check for winner
      if (this.checkWinner()) {
        return true;
      }
      else {
        this.playerTurn = 1; //1st player's turn next
      }
    },
    getAIChoice(max) {
      return Math.floor(Math.random() * Math.floor(max));
    },
    checkWinner() {
      if (this.countVertical() || this.countHorizontal() || this.countDiagonal()) { // check if any of the functions returns true

        this.gameMsg = `Game ends, Player-${this.playerTurn} wins`;

        let alert = document.getElementsByClassName('alert')[0];

        alert.style.display = 'block';

        this.gameEnd = 1;

        return true;
      }
      return false;
    },
    countVertical() {
      let currentValue = null,
        previousValue = 0,
        tally = 0;

      // Scan each row in series, tallying the length of each series. If a series
      // ever reaches four, return true for a win.
      for (let col = 0; col < this.columnCount; col++) {
        for (let row = 0; row < this.rowCount; row++) {
          currentValue = this.board[col][row];
          if (currentValue === previousValue && currentValue !== 0) {
            tally += 1;
          } else {
            // Reset the tally if you find a gap.
            tally = 0;
          }
          if (tally === this.winCount) {
            return true;
          }
          previousValue = currentValue;
        }

        // After each row, reset the tally and previous value.
        tally = 0;
        previousValue = 0;
      }

      // No horizontal win was found.
      return false;
    },
    countHorizontal() {
      let currentValue = null,
        previousValue = 0,
        tally = 0;
        
      // Scan each row in series, tallying the length of each series. If a series
      // ever reaches four, return true for a win.
      for (let row = 0; row < this.rowCount; row++) {
        for (let col = 0; col < this.columnCount; col++) {
          currentValue = this.board[col][row];
          if (currentValue === previousValue && currentValue !== 0) {
            tally += 1;
          } else {
            // Reset the tally if you find a gap.
            tally = 0;
          }
          if (tally === this.winCount) {
            return true;
          }
          previousValue = currentValue;
        }

        // After each row, reset the tally and previous value.
        tally = 0;
        previousValue = 0;
      }

      // No horizontal win was found.
      return false;
    },
    countDiagonal() {
      let xtemp = null,
        ytemp = null,
        currentValue = null,
        previousValue = 0,
        tally = 0;

      // Test for down-right diagonals across the top.
      for (let row = 0; row < this.rowCount; row++) {
        xtemp = row;
        ytemp = 0;

        while (xtemp < this.rowCount && ytemp < this.columnCount) {
          currentValue = this.board[ytemp][xtemp];
          if (currentValue === previousValue && currentValue !== 0) {
            tally += 1;
          } else {
            // Reset the tally if you find a gap.
            tally = 0;
          }
          if (tally === this.winCount) {
            return true;
          }
          previousValue = currentValue;

          // Shift down-right one diagonal index.
          xtemp++;
          ytemp++;
        }
        // Reset the tally and previous value when changing diagonals.
        tally = 0;
        previousValue = 0;
      }

      // Test for down-left diagonals across the top.
      for (let row = 0; row < this.rowCount; row++) {
        xtemp = row;
        ytemp = 0;

        while (0 < xtemp && ytemp < this.columnCount) {
          currentValue = this.board[ytemp][xtemp];
          if (currentValue === previousValue && currentValue !== 0) {
            tally += 1;
          } else {
            // Reset the tally if you find a gap.
            tally = 0;
          }
          if (tally === this.winCount) {
            return true;
          }
          previousValue = currentValue;

          // Shift down-left one diagonal index.
          xtemp--;
          ytemp++;
        }
        // Reset the tally and previous value when changing diagonals.
        tally = 0;
        previousValue = 0;
      }

      // Test for down-right diagonals down the left side.
      for (let col = 0; col < this.columnCount; col++) {
        xtemp = 0;
        ytemp = col;

        while (xtemp < this.rowCount && ytemp < this.columnCount) {
          currentValue = this.board[ytemp][xtemp];
          if (currentValue === previousValue && currentValue !== 0) {
            tally += 1;
          } else {
            // Reset the tally if you find a gap.
            tally = 0;
          }
          if (tally === this.winCount) {
            return true;
          }
          previousValue = currentValue;

          // Shift down-right one diagonal index.
          xtemp++;
          ytemp++;
        }
        // Reset the tally and previous value when changing diagonals.
        tally = 0;
        previousValue = 0;
      }

      // Test for down-left diagonals down the right side.
      for (let col = 0; col < this.columnCount; col++) {
        xtemp = this.rowCount;
        ytemp = col;

        while (0 < xtemp && ytemp < this.columnCount) {
          currentValue = this.board[ytemp][xtemp];
          if (currentValue === previousValue && currentValue !== 0) {
            tally += 1;
          } else {
            // Reset the tally if you find a gap.
            tally = 0;
          }
          if (tally === this.winCount) {
            return true;
          }
          previousValue = currentValue;

          // Shift down-left one diagonal index.
          xtemp--;
          ytemp++;
        }
        // Reset the tally and previous value when changing diagonals.
        tally = 0;
        previousValue = 0;
      }

      // No diagonal wins found. Return false.
      return false;
    },
    closeMSG(e) {
      e.target.parentElement.style = 'none';

      // reset 
      this.resetGame();
    }
  }
};
</script>

<style scoped>
.game_info {
  margin-bottom: 10px;
}

.game_info > span {
  color: #0014ce;
  font-weight: bold;
  font-size: 2em;
}

.container {
  display: flex;
  margin: 0 auto 22px;
  max-width: 700px;
}
.column {
  background-color: #f0f0f0;
  flex: 1 1 auto;
  position: relative;
  z-index: 5;
}
.column:hover {
  background-color: #fff;
}

.cell {
  border: 1px solid #808080;
  min-width: 100px;
  min-height: 100px;
  position: relative;
  z-index: 2;
}

.piece {
  border: 4px solid transparent;
  border-radius: 50%;
  left: calc(50% - 44px);
  min-width: 80px;
  min-height: 80px;
  position: absolute;
  top: calc(50% - 44px);
  transform: translateY(-100vh);
  transition: transform 0.4s ease-in;
}
.piece.red, .piece.blue {
  transform: translateY(0vh);
}
.piece.red {
  background-image: url('../assets/SB1.png');
  background-position: center;
  background-repeat: no-repeat; 
  background-size: cover;
  border-color: black;
}
.piece.blue {
  background-image: url('../assets/SW1.png');
  background-position: center;
  background-repeat: no-repeat; 
  background-size: cover;
  border-color: black;
}

.alert {
  padding: 20px;
  background-color: #4CAF50;
  color: white;
  display: none;
}

.closebtn {
  margin-left: 15px;
  color: white;
  font-weight: bold;
  float: right;
  font-size: 22px;
  line-height: 20px;
  cursor: pointer;
  transition: 0.3s;
}

.closebtn:hover {
  color: black;
}
</style>

