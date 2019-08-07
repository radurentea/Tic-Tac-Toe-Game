<template>
  <div>
    <div class="gameStatus" :class="gameStatusColor">{{ gameStatusMessage }}</div>
    <table class="grid">
      <tr>
        <cell name="1"></cell>
        <cell name="2"></cell>
        <cell name="3"></cell>
      </tr>
      <tr>
        <cell name="4"></cell>
        <cell name="5"></cell>
        <cell name="6"></cell>
      </tr>
      <tr>
        <cell name="7"></cell>
        <cell name="8"></cell>
        <cell name="9"></cell>
      </tr>
    </table>
  </div>
</template>

<script>
import Cell from "./Cell.vue";
export default {
  components: { Cell },
  data() {
    return {
      // can be O or X
      activePlayer: "O",
      // maintains the status of the game: turn or win or draw
      gameStatus: "turn",
      gameStatusMessage: `O's turn`,
      // status color is used as background color in the status bar
      // it can hold the name of either of the following CSS classes
      // statusTurn (default) is yellow for a turn
      // statusWin is green for a win
      // statusDraw is purple for a draw
      gameStatusColor: "statusTurn",
      // no. of moves played by both players in a single game (max = 9)
      moves: 0,
      // stores the placement of X and O in cells by their cell number
      cells: {
        1: "",
        2: "",
        3: "",
        4: "",
        5: "",
        6: "",
        7: "",
        8: "",
        9: ""
      },
      // contains all (8) possible winning conditions
      winConditions: [
        [1, 2, 3],
        [4, 5, 6],
        [7, 8, 9], // rows
        [1, 4, 7],
        [2, 5, 8],
        [3, 6, 9], // columns
        [1, 5, 9],
        [3, 5, 7] // diagonals
      ]
    };
  },
  computed: {
    // helper property to get the non-active player
    nonActivePlayer() {
      if (this.activePlayer === "O") {
        return "X";
      } else {
        return "O";
      }
    }
  },
  watch: {
    // watches for change in the value of gameStatus and changes the status
    // message and color accordingly
    gameStatus() {
      if (this.gameStatus === "win") {
        this.gameStatusColor = "statusWin";
        this.gameStatusMessage = `${this.nonActivePlayer} WON`;
        return;
      } else if (this.gameStatus === "draw") {
        this.gameStatusColor = "statusDraw";
        this.gameStatusMessage = "DRAW !";
        return;
      }
    }
  },
  methods: {
    // changes the active player to the non-active player with the help of the nonActivePlayer computed property
    changePlayer() {
      this.activePlayer = this.nonActivePlayer;
      this.gameStatusMessage = `${this.activePlayer}'s turn`;
    },
    // checks for possible win conditions from the data
    checkForWin() {
      //some breaks out of loop after first true is returned
      return this.winConditions.some(condition => {
        let count = 0;
        condition.forEach(cell => {
          if (this.cells[cell] === this.activePlayer) {
            count++;
          }
        });
        if (count === 3) {
          return true;
        }
        return false;
      });
    },
    gameIsWon() {
      // fires win event for the App component to change the score
      Event.$emit("win", this.activePlayer);
      // fires an event for the Cell to freeze
      Event.$emit("freeze");
      // sets the status to win
      return "win";
    },
    // returns the game status to the gameStatus property
    changeGameStatus() {
      if (this.checkForWin()) {
        return this.gameIsWon();
        // checks if the game is still not won and all cells are filled
      } else if (this.moves === 9) {
        // sets the status to draw
        return "draw";
      }
      // sets the status to turn
      return "turn";
    },
    // helper function for comparing cell values
    areEqual() {
      var len = arguments.length;
      // loops through each value and compares them with an empty string and
      // for inequality
      for (var i = 1; i < len; i++) {
        if (arguments[i] === "" || arguments[i] !== arguments[i - 1])
          return false;
      }
      return true;
    }
  },
  created() {
    // listens for a strike made by the user on cell
    // it is called by the Cell component
    Event.$on("strike", cellNumber => {
      // sets either X or O in the clicked cell of the cells array
      this.cells[cellNumber] = this.activePlayer;
      // increments the number of moves
      this.moves++;
      // stores the game status by calling the changeGameStatus method
      this.gameStatus = this.changeGameStatus();
      this.changePlayer();
    });
    // listens for a restart button press
    // the data of the component is reinitialized
    // it is called by the App component
    Event.$on("gridReset", () => {
      Object.assign(this.$data, this.$options.data.apply(this));
    });
  }
};
</script>

<style>
.grid {
  background-color: #34495e;
  color: #fff;
  width: 100%;
  border-collapse: collapse;
}
.gameStatus {
  margin: 0px;
  padding: 15px;
  border-top-left-radius: 20px;
  border-top-right-radius: 20px;
  background-color: #f1c40f;
  color: #fff;
  font-size: 1.4em;
  font-weight: bold;
}
.statusTurn {
  background-color: #35b6b4;
}
.statusWin {
  background-color: #2ecc71;
}
.statusDraw {
  background-color: #6c6968;
}
</style>