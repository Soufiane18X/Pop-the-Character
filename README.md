# Pop-the-Character
<!DOCTYPE html>
<html>
<head>
    <title>Pop the Character!</title>
    <style>
        body {
            margin: 0;
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            align-items: center;
            background: #2c3e50;
            font-family: 'Arial', sans-serif;
            color: white;
        }

        .game-title {
            color: #fff;
            font-size: 2.5em;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
            margin: 20px 0;
        }

        .game-stats {
            display: flex;
            gap: 30px;
            margin-bottom: 20px;
            font-size: 1.5em;
        }

        .stat {
            background: #34495e;
            padding: 10px 20px;
            border-radius: 10px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }

        #game-container {
            width: 600px;
            height: 400px;
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            grid-template-rows: repeat(3, 1fr);
            gap: 10px;
            padding: 20px;
            background: #34495e;
            border-radius: 15px;
            box-shadow: 0 8px 16px rgba(0,0,0,0.2);
        }

        .hole {
            background: #2c3e50;
            border-radius: 50%;
            position: relative;
            overflow: hidden;
            cursor: pointer;
        }

        .character {
            width: 100%;
            height: 100%;
            position: absolute;
            bottom: -100%;
            transition: bottom 0.1s;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 3em;
            user-select: none;
        }

        .character.show {
            bottom: 0;
        }

        .character.hit {
            animation: hit 0.2s ease-out;
        }

        @keyframes hit {
            0% { transform: scale(1); }
            50% { transform: scale(1.2); }
            100% { transform: scale(0); }
        }

        .controls {
            margin-top: 20px;
            display: flex;
            gap: 20px;
        }

        .btn {
            padding: 12px 24px;
            font-size: 1.2em;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            background: #e74c3c;
            color: white;
            transition: transform 0.1s, background 0.3s;
        }

        .btn:hover {
            background: #c0392b;
            transform: translateY(-2px);
        }

        #level-display {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            font-size: 5em;
            color: white;
            text-shadow: 2px 2px 8px rgba(0,0,0,0.5);
            opacity: 0;
            transition: opacity 0.3s;
        }

        .level-animation {
            animation: levelUp 1.5s ease-out;
        }

        @keyframes levelUp {
            0% { transform: translate(-50%, -50%) scale(0); opacity: 0; }
            50% { transform: translate(-50%, -50%) scale(1.2); opacity: 1; }
            100% { transform: translate(-50%, -50%) scale(1); opacity: 0; }
        }
    </style>
</head>
<body>
    <h1 class="game-title">Pop the Character!</h1>
    
    <div class="game-stats">
        <div class="stat">Score: <span id="score">0</span></div>
        <div class="stat">Niveau: <span id="level">1</span></div>
        <div class="stat">Temps: <span id="timer">30</span>s</div>
    </div>

    <div id="game-container"></div>
    <div id="level-display"></div>

    <div class="controls">
        <button class="btn" onclick="startGame()">Démarrer</button>
        <button class="btn" onclick="showRules()">Règles</button>
    </div>

    <script>
        const characters = ['🦸‍♂️', '🧙‍♂️', '🦹‍♀️', '🥷', '🦊', '🐲'];
        const gameContainer = document.getElementById('game-container');
        const scoreDisplay = document.getElementById('score');
        const levelDisplay = document.getElementById('level');
        const timerDisplay = document.getElementById('timer');
        const levelAnimation = document.getElementById('level-display');
        
        let score = 0;
        let level = 1;
        let timeLeft = 30;
        let gameInterval;
        let timerInterval;
        let isPlaying = false;
        
        // Create holes
        for (let i = 0; i < 12; i++) {
            const hole = document.createElement('div');
            hole.className = 'hole';
            const character = document.createElement('div');
            character.className = 'character';
            character.textContent = characters[Math.floor(Math.random() * characters.length)];
            hole.appendChild(character);
            hole.addEventListener('click', () => whack(character));
            gameContainer.appendChild(hole);
        }

        function showRules() {
            alert(`Règles du jeu:
1. Cliquez sur les personnages quand ils apparaissent
2. Chaque personnage touché = 10 points
3. Atteignez 100 points pour passer au niveau suivant
4. La vitesse augmente à chaque niveau
5. Vous avez 30 secondes par partie!
            `);
        }

        function startGame() {
            if (isPlaying) return;
            
            // Reset game state
            isPlaying = true;
            score = 0;
            level = 1;
            timeLeft = 30;
            scoreDisplay.textContent = score;
            levelDisplay.textContent = level;
            timerDisplay.textContent = timeLeft;

            // Start character appearances
            gameInterval = setInterval(showRandomCharacter, 1000 / level);

            // Start timer
            timerInterval = setInterval(() => {
                timeLeft--;
                timerDisplay.textContent = timeLeft;
                if (timeLeft <= 0) endGame();
            }, 1000);
        }

        function showRandomCharacter() {
            const holes = document.querySelectorAll('.hole');
            const randomHole = holes[Math.floor(Math.random() * holes.length)];
            const character = randomHole.querySelector('.character');
            
            character.textContent = characters[Math.floor(Math.random() * characters.length)];
            character.classList.add('show');

            setTimeout(() => {
                character.classList.remove('show');
            }, 800 / level);
        }

        function whack(character) {
            if (!character.classList.contains('show') || character.classList.contains('hit')) return;
            
            score += 10;
            scoreDisplay.textContent = score;
            
            character.classList.add('hit');
            character.classList.remove('show');
            
            // Level up check
            if (score >= level * 100) {
                levelUp();
            }
        }

        function levelUp() {
            level++;
            levelDisplay.textContent = level;
            
            // Show level animation
            levelAnimation.textContent = `Niveau ${level}!`;
            levelAnimation.classList.add('level-animation');
            
            // Clear and restart intervals with new speed
            clearInterval(gameInterval);
            gameInterval = setInterval(showRandomCharacter, 1000 / level);
            
            setTimeout(() => {
                levelAnimation.classList.remove('level-animation');
            }, 1500);
        }

        function endGame() {
            isPlaying = false;
            clearInterval(gameInterval);
            clearInterval(timerInterval);
            
            alert(`Partie terminée!\nScore final: ${score}\nNiveau atteint: ${level}`);
            
            // Remove all showing characters
            document.querySelectorAll('.character').forEach(char => {
                char.classList.remove('show', 'hit');
            });
        }
    </script>
</body>
</html>
