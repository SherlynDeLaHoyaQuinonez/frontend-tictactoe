<template>
  <div id="app">
    <h1>Tic Tac Toe</h1>
    <div class="board">
      <div v-for="(row, rowIndex) in board" :key="rowIndex" class="row">
        <div
          v-for="(cell, colIndex) in row"
          :key="colIndex"
          class="cell"
          @click="makeMove(rowIndex, colIndex)"
        >
          {{ cell }}
        </div>
      </div>
    </div>
    <div class="message">
      <p :class="{ 'alert': message !== '' }">{{ message }}</p>
      <button @click="startGame">Nuevo Juego</button>
      <button @click="restartGame">Reiniciar Juego</button>
    </div>
  </div>
</template>

<script>
import axios from "axios";
export default {
  data() {
    return {
      board: [
        ["", "", ""],
        ["", "", ""],
        ["", "", ""],
      ],
      ID_Game: null,
      Player: null,
      message: "",
      apiBaseUrl: "https://46i3aj7hn3.execute-api.us-east-1.amazonaws.com", // Cambia a la URL de tu API
    };
  },
  methods: {
    makeMove(rowIndex, colIndex) {
      if (this.ID_Game != null) {
        if (this.board[rowIndex][colIndex] === "") {
          axios
            .get(`${this.apiBaseUrl}/game/${this.ID_Game}`)
            .then((response) => {
              console.log(response.data["Game"]);
            });

          const move = {
            row: rowIndex.toString(),
            col: colIndex.toString(),
            player: this.Player, // Enviar el jugador actual con la solicitud
          };

          axios
            .put(`${this.apiBaseUrl}/makeMove/${this.ID_Game}`, move)
            .then((response) => {
              console.log(response.data);
              if (response.data["Valido"] === "Movimiento valido.") {
                this.board[rowIndex][colIndex] = this.Player;

                if (this.checkWinner(this.Player)) {
                  this.message = `¡El jugador ${this.Player} ha ganado!`;
                } else if (this.checkDraw()) {
                  this.message = "¡Es un empate!";
                } else {
                  this.Player = this.Player === "X" ? "O" : "X"; // Cambiar al siguiente jugador
                }
              }
            });
        } else {
          alert("La celda ya está ocupada");
        }
      } else {
        alert("Inicia un juego primero");
      }
    },

    checkWinner(player) {
  const winPatterns = [
    // Filas
    [[0, 0], [0, 1], [0, 2]],
    [[1, 0], [1, 1], [1, 2]],
    [[2, 0], [2, 1], [2, 2]],
    // Columnas
    [[0, 0], [1, 0], [2, 0]],
    [[0, 1], [1, 1], [2, 1]],
    [[0, 2], [1, 2], [2, 2]],
    // Diagonales
    [[0, 0], [1, 1], [2, 2]],
    [[0, 2], [1, 1], [2, 0]],
  ];

  for (const pattern of winPatterns) {
    const [a, b, c] = pattern;
    if (
      this.board[a[0]][a[1]] === player &&
      this.board[b[0]][b[1]] === player &&
      this.board[c[0]][c[1]] === player
    ) {
      return true; // Jugador actual ha ganado
    }
  }

  return false; // Nadie ha ganado todavía
},

    checkDraw() {
      for (let i = 0; i < 3; i++) {
        for (let j = 0; j < 3; j++) {
          if (this.board[i][j] === "") {
            return false; // Todavía hay celdas vacías, no es un empate
          }
        }
      }
      return true; // Todas las celdas están ocupadas, es un empate
    },

    startGame() {
      axios
        .get(`${this.apiBaseUrl}/get-id`)
        .then((response) => {
          this.ID_Game = response.data["New Game"];
          this.Player = "X"; // Inicializar el jugador actual como "X"
        })
        .then(() => {
          axios
            .put(`${this.apiBaseUrl}/game/${this.ID_Game}`)
            .then((response) => {
              console.log(response.data);
              alert(`Juego iniciado.`);
            });
        });
    },

    restartGame() {
      axios
        .put(`${this.apiBaseUrl}/resetGame/${this.ID_Game}`)
        .then((response) => {
          if (response.data["Valido"] == "Juego reiniciado") {
            this.message = '';
            this.board = [
              ["", "", ""],
              ["", "", ""],
              ["", "", ""],
            ];
            alert ("Juego reiniciado");
          }
        });
    },
  },
};
</script>

<style scoped>
#app {
  background: linear-gradient(135deg, #752aa3, #be90db);
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
}

#app h1 {
  font-size: 5em; /* Ajusta el tamaño de la fuente según tus preferencias */
  color: #ccc;
}

.board {
  display: inline-block;
  background: #f8f1f1;
  border-radius: 20px;
  width: 430px;
  height: 400px;
  flex-shrink: 0;
  box-shadow: 0px 4px 4px 0px rgba(0, 0, 0, 0.25);
  content: attr(center);
}

.row {
  display: flex;
}

.cell {
  width: 141px;
  height: 132px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 2em;
  cursor: pointer;
  border: 1px solid #ccc;
  border-radius: 20px;
  background: #d9d9d9;
  box-shadow: 0px 2px 4px 0px rgba(0, 0, 0, 0.25);
  flex-shrink: 0;

}

.message {
  margin-top: 20px;
  font-size: 1.5em;
}

button {
  font-size: 1em;
  padding: 10px 20px;
  background-color: black;
  color: white;
  border: none;
  cursor: pointer;
  margin-top: 0px;
  margin-block-end: 50px;
  margin-left: 8px;
  border-radius: 5px;
}

button:hover {
  background-color: #333;
}

</style>