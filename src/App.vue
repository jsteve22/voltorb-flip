<script lang="ts">
import { defineComponent } from 'vue';

// Each board tile will have three different components
// 1. Actual value of tile    (1,2,3,bomb) [integer] (bomb == 0)
// 2. Current displayed value (covered, marked value, actual value) [string] 
// 3. Flipped or not flipped  (whether it has been flipped) [boolean]

const levels = [
  [ 
    { level: 0, twos: 3, threes: 1, bombs: 6 }
  ],
  [ 
    { level: 1, twos: 1, threes: 3, bombs: 7 }
  ],
  [ 
    { level: 2, twos: 2, threes: 3, bombs: 8 }
  ],
  [ 
    { level: 3, twos: 3, threes: 3, bombs: 8 }
  ],
  [ 
    { level: 4, twos: 7, threes: 1, bombs: 10 }
  ],
  [ 
    { level: 5, twos: 3, threes: 4, bombs: 10 }
  ],
  [ 
    { level: 6, twos: 7, threes: 2, bombs: 10 }
  ],
  [ 
    { level: 7, twos: 0, threes: 7, bombs: 10 }
  ]
];

class boardTile {
  value:   number;
  disp:    string;
  flipped: boolean;

  constructor() {
    this.value   = 1;
    this.disp    = 'n';
    this.flipped = false;
  }
}

function getRandomValue() {
  const array = new Uint32Array(1);
  crypto.getRandomValues(array);
  return array[0];
}

function resetBoard(board:boardTile[][]) {
  for (let i=0; i < board.length; i++) {
    for (let j=0; j < board[i].length; j++) {
      // reset the value, display, and flipped
      board[i][j].value   = 1;  // one 
      board[i][j].disp    = 'n'; // covered
      board[i][j].flipped = false; // not flipped
    }
  }
}

function addRandomTiles(board:boardTile[][], tile:number, count:number) {
  // TODO: Rewrite this function

  // this function will add random values to the board
  let added = 0;
  let position = getRandomValue() % 25;
  while (added < count) {
    // try to add value to board
    let attemptPosition = position;
    let x = Math.floor(attemptPosition / 5);
    let y = attemptPosition % 5;
    while ( board[x][y].value !== 1 ) {
      attemptPosition++;
      attemptPosition = attemptPosition % 25;
      x = Math.floor(attemptPosition / 5);
      y = attemptPosition % 5;
      if ( attemptPosition === position )
        return;
    }

    board[x][y].value = tile;
    added++;
    position = getRandomValue() % 25;
  }
}

var board: boardTile[][] = [];

// initialize the board with nothing
for (let i = 0; i < 5; i++) {
  var row: boardTile[] = [];
  for (let j = 0; j < 5; j++) {
    row.push( new boardTile );
  }
  board.push( row );
}

let boardInterface: string[][] = [['n','n','n','n','n'],['n','n','n','n','n'],['n','n','n','n','n'],['n','n','n','n','n'],['n','n','n','n','n']];

export default defineComponent({
  name: 'App',
  data() {
    return {
      board: board,
      boardInterface: boardInterface,
      cursor: 'x',
      level: 0,
      playingGame: false,
      betweenGame: false,
      won:  false,
      lost: false
    }
  },
  methods: {
    outputHello() {
      console.log('Hello World!');
    },
    printBoard() {
      for (let i = 0; i < 5; i++) {
        for (let j = 0; j < 5; j++) {
          // let x = board[i][j];
          // console.log( x.disp );
          this.boardInterface[i][j] = board[i][j].disp;
        }
      }
      // this.boardInterface = boardInterface;
    },
    updateBoard(x:number,y:number) {
      if (this.playingGame == false)
        return;

      this.boardInterface[y][x] = this.cursor;
      // this will hold the logic for updating the board
      if (this.cursor === 'x') {
        this.board[y][x].flipped = true;
      }
      this.checkWin();
      this.checkLost();
      console.log("updateBoard");
    },
    randomizeBoard() {
      // reset board
      this.reset();
      // resetBoard(this.board);

      if (this.level >= levels.length)
        this.level = 0;

      // get level
      const level = this.level;

      // get level configuration
      const config = levels[level][0];

      // randomize each value
      addRandomTiles(this.board,2,config.twos);
      addRandomTiles(this.board,3,config.threes);
      addRandomTiles(this.board,0,config.bombs);
    },
    reset() {
      // reset the board
      resetBoard(this.board);
      // reset the game state variables
      this.won = false;
      this.lost = false;
      this.playingGame = true;
    },
    rowSumCoins(row:number){
      let coins = 0;
      this.board[row].forEach( tile => {
        coins += tile.value;
      });
      return coins;
    },
    rowSumBombs(row:number){
      let bombs = 0;
      this.board[row].forEach( tile => {
        if (tile.value === 0)
          bombs++;
      });
      return bombs;
    },
    colSumCoins(col:number){
      let coins = 0;
      this.board.forEach( row => {
        coins += row[col].value;
      });
      return coins;
    },
    colSumBombs(col:number){
      let bombs = 0;
      this.board.forEach( row => {
        if (row[col].value === 0)
          bombs++;
      });
      return bombs;
    },
    checkWin() {
      let win = true;
      this.board.forEach( row => {
        row.forEach( tile => {
          if ( (tile.value == 2 || tile.value == 3) && tile.flipped == false ) {
            win = false;
          }
        });
      });
      if (win) {
        this.won = true;
        this.playingGame = false;
        // also update level
        // this.level++;
      }
    },
    checkLost() {
      this.board.forEach( row => {
        row.forEach( tile => {
          if ( tile.flipped && tile.value == 0 ) {
            this.lost = true;
            this.playingGame = false;
          }
        })
      })
    },
    updateGame() {
      if (this.playingGame)
        return;
        
      if (this.betweenGame) {
        this.betweenGame = false;
        return;
      }

      if (this.lost) {
        this.level--;
        this.level = (this.level < 0) ? 0 : this.level;
      }

      if (this.won) {
        this.level++;
        this.level = this.level % levels.length;
      }

      this.betweenGame = true;
      this.randomizeBoard();
      console.log('updateGame');
    }
  }
});
</script>

<template>
  <h1>
    This is the empty website
  </h1>
  <button @click="printBoard" style="display: block;">This is a button</button>
  <button @click="randomizeBoard" style="display: block;">Random board</button>
  <button @click="reset" style="display: block;">Reset board</button>
  <!-- <input v-model="cursor"> -->
  <select v-model="cursor">
    <option value="1">1</option>
    <option value="2">2</option>
    <option value="3">3</option>
    <option value="b">b</option>
    <option value="x">x</option>
    <option value="n">n</option>
  </select>
  <p> cursor = {{ cursor }} </p>
  <p> level = {{ level }} </p>
  <p> playingGame = {{ playingGame }} </p>
  <p v-if="won"> You WON! </p>
  <p v-if="lost"> You LOST </p>
  <br>
  <div @click="updateGame">
    <div v-for="(row, iy) in board" :key="iy">
      <span v-for="(tile, ix) in row" :key=ix>
        <span v-if="tile.flipped">
          <span @click="updateBoard(ix,iy)" style="font-family: monospace; font-size: large; padding-left: 2px;">{{tile.value}}</span>
        </span>
        <span v-else>
          <span @click="updateBoard(ix,iy)" style="font-family: monospace; font-size: large; padding-left: 2px;">{{tile.disp}}</span>
        </span>
      </span>
      <span style="font-family: monospace; padding-left: 2px; color: red;">
        {{ rowSumCoins(iy) }}
      </span>
      <span style="font-family: monospace; padding-left: 2px; color: darkblue;">
        {{ rowSumBombs(iy) }}
      </span>
    </div>

    <div style="color: red;">
      <span v-for="(row, iy) in board" :key="iy" style="font-family: monospace; font-size: large; padding-left: 2px;">
        {{ colSumCoins(iy) }}
      </span>
      <span style="font-family: monospace; padding-left: 2px;">_</span>
      <span style="font-family: monospace; padding-left: 2px;">_</span>
    </div>

    <div style="color: darkblue;">
      <span v-for="(row, iy) in board" :key="iy" style="font-family: monospace; font-size: large; padding-left: 2px;">
        {{ colSumBombs(iy) }}
      </span>
      <span style="font-family: monospace; padding-left: 2px;">_</span>
      <span style="font-family: monospace; padding-left: 2px;">_</span>
    </div>
  </div>

  <div v-for="(row, iy) in board" :key="iy">
    <span v-for="(tile, ix) in row" :key=ix>
      <span @click="updateBoard(ix,iy)" style="font-family: monospace; font-size: large; padding-left: 2px;">{{tile.value}}</span>
    </span>
  </div>


</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
