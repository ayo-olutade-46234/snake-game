# Snake Game

A classic Snake game implementation using HTML5 Canvas and JavaScript.

## Features

- Classic snake gameplay
- Score tracking
- Game over detection
- Responsive controls using arrow keys
- Clean, modern UI

## How to Play

1. Open `index.html` in your web browser
2. Press any arrow key to start the game
3. Use arrow keys to control the snake:
   - ↑ Up Arrow - Move up
   - ↓ Down Arrow - Move down
   - ← Left Arrow - Move left
   - → Right Arrow - Move right
4. Eat the red food to grow and increase your score
5. Avoid hitting the walls or your own tail

## Installation

### Option 1: Direct Download
1. Download all files (`index.html`, `style.css`, `game.js`)
2. Place them in the same folder
3. Open `index.html` in your web browser

### Option 2: Clone Repository
```bash
git clone <repository-url>
cd snake-game
```

Then open `index.html` in your browser.

## Files

- **index.html** - Main HTML structure
- **style.css** - Styling and layout
- **game.js** - Game logic and mechanics

## Game Code

### index.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Snake Game</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div class="game-container">
        <h1>Snake Game</h1>
        <div class="score-board">
            <p>Score: <span id="score">0</span></p>
            <p>High Score: <span id="highScore">0</span></p>
        </div>
        <canvas id="gameCanvas" width="400" height="400"></canvas>
        <div class="controls">
            <p>Use Arrow Keys to Control</p>
            <button id="restartBtn">Restart Game</button>
        </div>
    </div>
    <script src="game.js"></script>
</body>
</html>
```

### style.css
```css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Arial', sans-serif;
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.game-container {
    text-align: center;
    background: white;
    padding: 30px;
    border-radius: 20px;
    box-shadow: 0 10px 40px rgba(0, 0, 0, 0.3);
}

h1 {
    color: #333;
    margin-bottom: 20px;
    font-size: 2.5em;
}

.score-board {
    display: flex;
    justify-content: space-around;
    margin-bottom: 20px;
    font-size: 1.2em;
    color: #555;
}

.score-board span {
    font-weight: bold;
    color: #667eea;
}

#gameCanvas {
    border: 3px solid #333;
    border-radius: 10px;
    background-color: #f0f0f0;
    display: block;
    margin: 0 auto;
}

.controls {
    margin-top: 20px;
}

.controls p {
    color: #666;
    margin-bottom: 10px;
}

#restartBtn {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    border: none;
    padding: 12px 30px;
    font-size: 1em;
    border-radius: 25px;
    cursor: pointer;
    transition: transform 0.2s, box-shadow 0.2s;
}

#restartBtn:hover {
    transform: translateY(-2px);
    box-shadow: 0 5px 15px rgba(102, 126, 234, 0.4);
}

#restartBtn:active {
    transform: translateY(0);
}
```

### game.js
```javascript
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const scoreElement = document.getElementById('score');
const highScoreElement = document.getElementById('highScore');
const restartBtn = document.getElementById('restartBtn');

// Game constants
const GRID_SIZE = 20;
const TILE_COUNT = canvas.width / GRID_SIZE;
const GAME_SPEED = 100; // milliseconds

// Game state
let snake = [{ x: 10, y: 10 }];
let food = { x: 15, y: 15 };
let dx = 0;
let dy = 0;
let score = 0;
let highScore = localStorage.getItem('snakeHighScore') || 0;
let gameLoop;
let gameStarted = false;

highScoreElement.textContent = highScore;

// Initialize game
function init() {
    snake = [{ x: 10, y: 10 }];
    food = generateFood();
    dx = 0;
    dy = 0;
    score = 0;
    gameStarted = false;
    scoreElement.textContent = score;
    
    if (gameLoop) {
        clearInterval(gameLoop);
    }
    
    draw();
}

// Generate random food position
function generateFood() {
    let newFood;
    do {
        newFood = {
            x: Math.floor(Math.random() * TILE_COUNT),
            y: Math.floor(Math.random() * TILE_COUNT)
        };
    } while (snake.some(segment => segment.x === newFood.x && segment.y === newFood.y));
    
    return newFood;
}

// Draw game elements
function draw() {
    // Clear canvas
    ctx.fillStyle = '#f0f0f0';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    
    // Draw snake
    snake.forEach((segment, index) => {
        ctx.fillStyle = index === 0 ? '#4CAF50' : '#66BB6A';
        ctx.fillRect(
            segment.x * GRID_SIZE,
            segment.y * GRID_SIZE,
            GRID_SIZE - 2,
            GRID_SIZE - 2
        );
    });
    
    // Draw food
    ctx.fillStyle = '#F44336';
    ctx.beginPath();
    ctx.arc(
        food.x * GRID_SIZE + GRID_SIZE / 2,
        food.y * GRID_SIZE + GRID_SIZE / 2,
        GRID_SIZE / 2 - 2,
        0,
        Math.PI * 2
    );
    ctx.fill();
}

// Update game state
function update() {
    if (dx === 0 && dy === 0) return;
    
    // Calculate new head position
    const head = { x: snake[0].x + dx, y: snake[0].y + dy };
    
    // Check wall collision
    if (head.x < 0 || head.x >= TILE_COUNT || head.y < 0 || head.y >= TILE_COUNT) {
        gameOver();
        return;
    }
    
    // Check self collision
    if (snake.some(segment => segment.x === head.x && segment.y === head.y)) {
        gameOver();
        return;
    }
    
    // Add new head
    snake.unshift(head);
    
    // Check food collision
    if (head.x === food.x && head.y === food.y) {
        score++;
        scoreElement.textContent = score;
        
        if (score > highScore) {
            highScore = score;
            highScoreElement.textContent = highScore;
            localStorage.setItem('snakeHighScore', highScore);
        }
        
        food = generateFood();
    } else {
        // Remove tail if no food eaten
        snake.pop();
    }
    
    draw();
}

// Game over
function gameOver() {
    clearInterval(gameLoop);
    ctx.fillStyle = 'rgba(0, 0, 0, 0.7)';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    
    ctx.fillStyle = 'white';
    ctx.font = '30px Arial';
    ctx.textAlign = 'center';
    ctx.fillText('Game Over!', canvas.width / 2, canvas.height / 2 - 20);
    ctx.font = '20px Arial';
    ctx.fillText(`Score: ${score}`, canvas.width / 2, canvas.height / 2 + 20);
}

// Handle keyboard input
document.addEventListener('keydown', (e) => {
    if (!gameStarted && (e.key.startsWith('Arrow'))) {
        gameStarted = true;
        gameLoop = setInterval(update, GAME_SPEED);
    }
    
    switch (e.key) {
        case 'ArrowUp':
            if (dy === 0) {
                dx = 0;
                dy = -1;
            }
            break;
        case 'ArrowDown':
            if (dy === 0) {
                dx = 0;
                dy = 1;
            }
            break;
        case 'ArrowLeft':
            if (dx === 0) {
                dx = -1;
                dy = 0;
            }
            break;
        case 'ArrowRight':
            if (dx === 0) {
                dx = 1;
                dy = 0;
            }
            break;
    }
});

// Restart button
restartBtn.addEventListener('click', init);

// Start game
init();
```

## Game Mechanics

### Snake Movement
- The snake moves continuously in the current direction
- Direction changes are made using arrow keys
- The snake cannot reverse direction (e.g., can't go left if moving right)

### Scoring
- Each food item eaten increases the score by 1
- The snake grows by one segment with each food eaten
- High score is saved in browser's local storage

### Game Over Conditions
- Hitting any wall
- Colliding with the snake's own body

## Customization

You can modify these constants in `game.js` to customize the game:

- `GRID_SIZE` - Size of each grid cell (default: 20)
- `GAME_SPEED` - Game update speed in milliseconds (default: 100, lower = faster)
- Colors can be changed in the `draw()` function

## Browser Compatibility

This game works in all modern browsers that support HTML5 Canvas:
- Chrome
- Firefox
- Safari
- Edge

## License

This project is open source and available for personal and educational use.

## Credits

Created as a classic arcade game implementation using vanilla JavaScript.

---

Enjoy playing! 🐍
