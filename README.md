<!DOCTYPE html>
<html lang="vi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Game Dò Mìn</title>
<style>
body {
    font-family: Arial, sans-serif;
    background: #222;
    color: white;
    text-align: center;
}
#controls {
    margin: 10px;
}
#board {
    display: grid;
    justify-content: center;
    margin-top: 10px;
}
.cell {
    width: 32px;
    height: 32px;
    background: #555;
    border: 1px solid #333;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    font-weight: bold;
    user-select: none;
}
.cell.open {
    background: #aaa;
    cursor: default;
}
.cell.mine {
    background: red;
}
.cell.flag {
    background: orange;
}
</style>
</head>

<body>
<h2>💣 GAME DÒ MÌN</h2>

<div id="controls">
    Hàng: <input id="rows" type="number" value="10" min="5" max="20">
    Cột: <input id="cols" type="number" value="10" min="5" max="20">
    Mìn: <input id="mines" type="number" value="15" min="1">
    <button onclick="startGame()">Chơi mới</button>
</div>

<div id="board"></div>

<script>
let rows, cols, mineCount;
let board = [];
let gameOver = false;

function startGame() {
    rows = +rowsInput.value;
    cols = +colsInput.value;
    mineCount = +minesInput.value;

    gameOver = false;
    board = [];
    boardDiv.innerHTML = "";
    boardDiv.style.gridTemplateColumns = `repeat(${cols}, 32px)`;

    for (let r = 0; r < rows; r++) {
        board[r] = [];
        for (let c = 0; c < cols; c++) {
            const cell = {
                mine: false,
                open: false,
                flag: false,
                count: 0,
                el: document.createElement("div")
            };

            cell.el.className = "cell";
            cell.el.oncontextmenu = e => {
                e.preventDefault();
                toggleFlag(r, c);
            };
            cell.el.onclick = () => openCell(r, c);

            boardDiv.appendChild(cell.el);
            board[r][c] = cell;
        }
    }

    placeMines();
    calculateNumbers();
}

function placeMines() {
    let placed = 0;
    while (placed < mineCount) {
        let r = Math.floor(Math.random() * rows);
        let c = Math.floor(Math.random() * cols);
        if (!board[r][c].mine) {
            board[r][c].mine = true;
            placed++;
        }
    }
}

function calculateNumbers() {
    for (let r = 0; r < rows; r++) {
        for (let c = 0; c < cols; c++) {
            if (board[r][c].mine) continue;
            let count = 0;
            for (let dr = -1; dr <= 1; dr++) {
                for (let dc = -1; dc <= 1; dc++) {
                    let nr = r + dr, nc = c + dc;
                    if (nr >= 0 && nc >= 0 && nr < rows && nc < cols) {
                        if (board[nr][nc].mine) count++;
                    }
                }
            }
            board[r][c].count = count;
        }
    }
}

function openCell(r, c) {
    if (gameOver) return;
    const cell = board[r][c];
    if (cell.open || cell.flag) return;

    cell.open = true;
    cell.el.classList.add("open");

    if (cell.mine) {
        cell.el.classList.add("mine");
        cell.el.textContent = "💣";
        endGame(false);
        return;
    }

    if (cell.count > 0) {
        cell.el.textContent = cell.count;
    } else {
        for (let dr = -1; dr <= 1; dr++) {
            for (let dc = -1; dc <= 1; dc++) {
                let nr = r + dr, nc = c + dc;
                if (nr >= 0 && nc >= 0 && nr < rows && nc < cols) {
                    openCell(nr, nc);
                }
            }
        }
    }

    checkWin();
}

function toggleFlag(r, c) {
    if (gameOver) return;
    const cell = board[r][c];
    if (cell.open) return;

    cell.flag = !cell.flag;
    cell.el.classList.toggle("flag");
    cell.el.textContent = cell.flag ? "🚩" : "";
}

function endGame(win) {
    gameOver = true;
    for (let r = 0; r < rows; r++) {
        for (let c = 0; c < cols; c++) {
            if (board[r][c].mine) {
                board[r][c].el.textContent = "💣";
            }
        }
    }
    alert(win ? "🎉 BẠN THẮNG!" : "💥 THUA RỒI!");
}

function checkWin() {
    let opened = 0;
    for (let r = 0; r < rows; r++) {
        for (let c = 0; c < cols; c++) {
            if (board[r][c].open) opened++;
        }
    }
    if (opened === rows * cols - mineCount) {
        endGame(true);
    }
}

const rowsInput = document.getElementById("rows");
const colsInput = document.getElementById("cols");
const minesInput = document.getElementById("mines");
const boardDiv = document.getElementById("board");

startGame();
</script>
</body>
</html>
