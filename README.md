<!DOCTYPE html>


<html lang="ru">

<head>

    <meta charset="UTF-8">

   
    <title>Сапёр</title>

    
    <style>

       

        /* Таблица игрового поля */
        #board {

            /* Убираем двойные границы между ячейками */
            border-collapse: collapse;

            
            margin: 10px 0;
        }

        /* Все клетки игрового поля */
        #board td {


            width: 30px;

            
            height: 30px;

            
            text-align: center;

  
            vertical-align: middle;

            border: 1px solid #ccc;

            cursor: pointer;

            
            font-size: 18px;
        }

        
        #board td.covered {

            
            background-color: #ddd;
        }

        
        #board td.uncovered {

            
            background-color: #474444;
        }

        
        #board td.mine {

            
            background-color: #f88;
        }

        
        #board td.flag {

            
            background-color: rgb(105, 223, 105);
        }

        
        body {

            
            font-family: Arial, sans-serif;
        }

        
        #controls {

            
            margin: 10px 0;
        }

        
        #records {

            
            margin-top: 10px;
        }

    </style>

</head>

<body>

    
    <h1>Игра «Сапёр»</h1>

    
    <div id="controls">

        
        <button id="newGame">
            Новая игра
        </button>

        
        <label>

            Имя:

            
            <input
                type="text"
                id="playerName"
                placeholder="Введите имя">

        </label>

    </div>

   
    <div id="gameInfo"></div>

    
    <table id="board"></table>

    
    <h2>Рекорды</h2>

    
    <table id="records"></table>

    
    <script>


const ROWS = 3;
const COLS = 3;
const MINES_COUNT = 1;



let board = [];
let uncoveredCount = 0;
let gameOver = false;
let startTime = null;


let records = JSON.parse(localStorage.getItem("records")) || [];
function addRecord(name, time) {
    records.push({
        name: name,
        time: time
    });
    records.sort((a,b) => a.time - b.time);

    // Сохраняем массив рекордов в localStorage
    localStorage.setItem(

        // Ключ хранения
        "records",

        // Превращаем массив в строку
        JSON.stringify(records)
    );
}


function showRecords() {
    let table = document.getElementById("records");
    table.innerHTML = "";
    let header = table.insertRow();
    header.innerHTML = "<th>Имя</th><th>Время (с)</th>";

    // Перебираем все рекорды
    records.forEach(record => {

        // Создаём новую строку таблицы
        let row = table.insertRow();

        // Создаём первую ячейку и записываем имя
        row.insertCell().innerText =
            record.name;

        // Создаём вторую ячейку и записываем время
        row.insertCell().innerText =
            record.time;
    });
}


function initBoard() {
    board = [];
    uncoveredCount = 0;
    gameOver = false;
    startTime = Date.now();
    document.getElementById('gameInfo').innerText = '';
    let table = document.getElementById('board');
    table.innerHTML = '';

    // Создаём строки игрового поля
    for (let i = 0; i < ROWS; i++) {

        // Создаём строку в массиве board
        board[i] = [];

        // Добавляем строку в HTML-таблицу
        let row = table.insertRow();

        // Создаём клетки в текущей строке
        for (let j = 0; j < COLS; j++) {

            // Создаём объект клетки
            board[i][j] = {

                // Есть ли мина
                mine: false,

                // Открыта ли клетка
                uncovered: false,

                // Стоит ли флажок
                flagged: false,

                // Количество мин вокруг
                near: 0
            };

            // Создаём HTML-ячейку
            let cell = row.insertCell();

            // Назначаем класс закрытой клетки
            cell.className = 'covered';

            // Сохраняем номер строки в data-атрибуте
            cell.dataset.row = i;

            // Сохраняем номер столбца в data-атрибуте
            cell.dataset.col = j;

            // Обработчик левого клика
            cell.addEventListener('click', () =>

                // Открываем клетку
                openCell(i, j)
            );

            // Обработчик правого клика
            cell.addEventListener('contextmenu', (e) => {

                // Отключаем стандартное меню браузера
                e.preventDefault();

                // Ставим или снимаем флажок
                toggleFlag(i, j);
            });
        }
    }
    // Размещаем мины случайным образом
    let minesPlaced = 0;
    while (minesPlaced < MINES_COUNT) {
        let r = Math.floor(Math.random() * ROWS);
        let c = Math.floor(Math.random() * COLS);
        if (!board[r][c].mine) {
            board[r][c].mine = true;
            minesPlaced++;
        }
    }

    // Вычисляем значение near (число мин вокруг) для каждой клетки
    for (let i = 0; i < ROWS; i++) {
        for (let j = 0; j < COLS; j++) {
            if (!board[i][j].mine) {
                let count = 0;
                for (let dx = -1; dx <= 1; dx++) {
                    for (let dy = -1; dy <= 1; dy++) {
                        let ni = i + dx, nj = j + dy;
                        if (ni >= 0 && ni < ROWS && nj >= 0 && nj < COLS) {
                            if (board[ni][nj].mine) count++;
                        }
                    }
                }
                board[i][j].near = count;
            }
        }
    }
}

// Открывает клетку (row, col). Если мины нет – показывает число или открывает соседние.
function openCell(row, col) {
    if (gameOver) return;
    let cellData = board[row][col];
    // Нельзя открыть уже открытую или помеченную флагом клетку
    if (cellData.uncovered || cellData.flagged) return;
    let table = document.getElementById('board');
    let cell = table.rows[row].cells[col];
    cellData.uncovered = true;
    cell.className = 'uncovered';
    uncoveredCount++;
    // Если мина – проигрываем
    if (cellData.mine) {
        cell.className = 'mine';
        cell.innerText = 'm';
        endGame(false);
        return;
    }
    // Показываем число мин вокруг, если оно > 0
    if (cellData.near > 0) {
        cell.innerText = cellData.near;
    } else {
        // Если число 0, то автоматически открываем все соседние клетки
        for (let dx = -1; dx <= 1; dx++) {
            for (let dy = -1; dy <= 1; dy++) {
                let ni = row + dx, nj = col + dy;
                if (ni >= 0 && ni < ROWS && nj >= 0 && nj < COLS) {
                    openCell(ni, nj);
                }
            }
        }
    }
    // Проверка на победу: открыты все клетки, кроме мин
    if (uncoveredCount === ROWS * COLS - MINES_COUNT) {
        endGame(true);
    }
}

// Ставим или снимаем флажок (правый клик)
function toggleFlag(row, col) {
    if (gameOver) return;
    let data = board[row][col];
    // Нельзя ставить флаг на уже открытую клетку
    if (data.uncovered) return;
    data.flagged = !data.flagged;
    let cell = document.getElementById('board').rows[row].cells[col];
    if (data.flagged) {
        cell.className = 'flag';
        cell.innerText = 'f';
    } else {
        cell.className = 'covered';
        cell.innerText = '';
    }
}

// Завершение игры: win = true при победе, иначе поражение
function endGame(win) {

    if (gameOver) return;

    gameOver = true;

    let message = win ? 'Вы выиграли!' : 'Вы проиграли!';
    document.getElementById('gameInfo').innerText = message;

    if (win) {
        let time = Math.floor((Date.now() - startTime) / 1000);
        let name = document.getElementById('playerName').value || 'Без имени';

        addRecord(name, time);
        showRecords();
    }
}
// Обработчик кнопки «Новая игра»
document.getElementById('newGame').addEventListener('click', () => {
    initBoard();
});

// При загрузке страницы инициализируем игру и показываем рекорды
window.onload = () => {
    initBoard();
    showRecords();
};
</script>

</body>
</html>
