```html
<!DOCTYPE html>
<html dir="rtl" lang="he">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ארץ עיר - מסך פתיחה</title>
    <style>
        body {
            font-family: 'Heebo', Arial, sans-serif;
            background-color: #f0f8ff;
            margin: 0;
            padding: 20px;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }
        .container {
            background-color: white;
            border-radius: 15px;
            padding: 30px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            max-width: 500px;
            width: 100%;
        }
        h1 {
            color: #3498db;
            text-align: center;
            font-size: 2.5em;
            margin-bottom: 30px;
        }
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        input, select {
            width: 100%;
            padding: 10px;
            margin-bottom: 20px;
            border: 1px solid #ddd;
            border-radius: 5px;
            font-size: 16px;
        }
        button {
            width: 100%;
            padding: 12px;
            background-color: #3498db;
            color: white;
            border: none;
            border-radius: 5px;
            font-size: 18px;
            cursor: pointer;
            transition: background-color 0.3s;
        }
        button:hover {
            background-color: #2980b9;
        }
        #playerInputs {
            margin-top: 20px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>ברוכים הבאים למשחק ארץ עיר</h1>
        <form id="setupForm">
            <label for="playerCount">מספר שחקנים:</label>
            <select id="playerCount" onchange="generatePlayerInputs()">
                <option value="2">2</option>
                <option value="3">3</option>
                <option value="4">4</option>
            </select>
            
            <div id="playerInputs"></div>
            
            <button type="submit">התחל משחק</button>
        </form>
    </div>

    <script>
        function generatePlayerInputs() {
            const playerCount = document.getElementById('playerCount').value;
            const playerInputs = document.getElementById('playerInputs');
            playerInputs.innerHTML = '';
            
            for (let i = 1; i <= playerCount; i++) {
                playerInputs.innerHTML += `
                    <label for="player${i}">שם שחקן ${i}:</label>
                    <input type="text" id="player${i}" name="player${i}" required>
                `;
            }
        }

        document.getElementById('setupForm').addEventListener('submit', function(e) {
            e.preventDefault();
            const playerCount = document.getElementById('playerCount').value;
            const players = [];
            
            for (let i = 1; i <= playerCount; i++) {
                const playerName = document.getElementById(`player${i}`).value;
                players.push(playerName);
            }
            
            localStorage.setItem('players', JSON.stringify(players));
            window.location.href = 'game.html'; // עבור למסך המשחק
        });

        // יצירת שדות קלט לשחקנים בטעינת הדף
        generatePlayerInputs();
    </script>
</body>
</html>
```

```html
<!DOCTYPE html>
<html dir="rtl" lang="he">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ארץ עיר - משחק</title>
    <style>
        body {
            font-family: 'Heebo', Arial, sans-serif;
            background-color: #f0f8ff;
            margin: 0;
            padding: 20px;
        }
        .game-container {
            background-color: white;
            border-radius: 15px;
            padding: 20px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            max-width: 800px;
            margin: 0 auto;
        }
        h1, h2 {
            color: #3498db;
            text-align: center;
        }
        .letter-display {
            font-size: 48px;
            text-align: center;
            margin: 20px 0;
            padding: 10px;
            background-color: #3498db;
            color: white;
            border-radius: 10px;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 12px;
            text-align: center;
        }
        th {
            background-color: #3498db;
            color: white;
        }
        input[type="text"] {
            width: 90%;
            padding: 8px;
            border: 1px solid #ddd;
            border-radius: 5px;
            font-size: 16px;
        }
        button {
            padding: 10px 20px;
            font-size: 16px;
            background-color: #3498db;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            transition: background-color 0.3s;
            display: block;
            margin: 20px auto;
        }
        button:hover {
            background-color: #2980b9;
        }
        button:disabled {
            background-color: #bdc3c7;
            cursor: not-allowed;
        }
        #timer {
            font-size: 24px;
            font-weight: bold;
            text-align: center;
            margin: 20px 0;
            color: #e74c3c;
        }
        .hidden {
            display: none;
        }
        #results {
            margin-top: 30px;
        }
        .fade-in {
            animation: fadeIn 0.5s;
        }
        @keyframes fadeIn {
            from { opacity: 0; }
            to { opacity: 1; }
        }
        #scoreboard {
            margin-top: 30px;
        }
    </style>
</head>
<body>
    <div class="game-container">
        <h1>משחק ארץ עיר</h1>
        <div class="letter-display" id="randomLetter">א</div>
        <div id="timer" class="hidden">10</div>
        <div id="playerForm">
            <h2 id="playerName"></h2>
            <form id="gameForm">
                <table>
                    <tr>
                        <th>קטגוריה</th>
                        <th>תשובה</th>
                    </tr>
                    <tr>
                        <td>ארץ</td>
                        <td><input type="text" name="country" required></td>
                    </tr>
                    <tr>
                        <td>עיר</td>
                        <td><input type="text" name="city" required></td>
                    </tr>
                    <tr>
                        <td>חי</td>
                        <td><input type="text" name="animal" required></td>
                    </tr>
                    <tr>
                        <td>צומח</td>
                        <td><input type="text" name="plant" required></td>
                    </tr>
                    <tr>
                        <td>דומם</td>
                        <td><input type="text" name="object" required></td>
                    </tr>
                    <tr>
                        <td>שם של בת</td>
                        <td><input type="text" name="girlName" required></td>
                    </tr>
                    <tr>
                        <td>שם של בן</td>
                        <td><input type="text" name="boyName" required></td>
                    </tr>
                    <tr>
                        <td>מקצוע</td>
                        <td><input type="text" name="profession" required></td>
                    </tr>
                </table>
                <button type="submit" id="submitButton" disabled>סיימתי</button>
            </form>
        </div>
        <div id="results" class="hidden">
            <h2>תוצאות הסיבוב</h2>
            <table id="resultsTable"></table>
            <button onclick="startNewRound()">סיבוב חדש</button>
        </div>
        <div id="scoreboard" class="hidden">
            <h2>טבלת ניקוד</h2>
            <table id="scoreTable"></table>
        </div>
    </div>

    <script>
        const players = JSON.parse(localStorage.getItem('players')) || ['שחקן 1', 'שחקן 2'];
        let currentPlayerIndex = 0;
        let currentLetter = '';
        let answers = {};
        let scores = players.reduce((acc, player) => ({ ...acc, [player]: 0 }), {});
        const categories = ['country', 'city', 'animal', 'plant', 'object', 'girlName', 'boyName', 'profession'];
        const hebrewCategories = ['ארץ', 'עיר', 'חי', 'צומח', 'דומם', 'שם של בת', 'שם של בן', 'מקצוע'];

        function startGame() {
            chooseLetter();
            showPlayerForm();
            updateScoreboard();
        }

        function chooseLetter() {
            const hebrewLetters = 'אבגדהוזחטיכלמנסעפצקרשת';
            const randomIndex = Math.floor(Math.random() * hebrewLetters.length);
            currentLetter = hebrewLetters[randomIndex];
            document.getElementById('randomLetter').textContent = currentLetter;
        }

        function showPlayerForm() {
            document.getElementById('playerName').textContent = players[currentPlayerIndex];
            document.getElementById('playerForm').classList.remove('hidden');
            document.getElementById('results').classList.add('hidden');
            document.getElementById('scoreboard').classList.add('hidden');
            document.getElementById('submitButton').disabled = true;
            document.getElementById('gameForm').reset();
        }

        document.getElementById('gameForm').addEventListener('input', function() {
            const allFilled = [...this.elements].every(element => element.value.trim() !== '');
            document.getElementById('submitButton').disabled = !allFilled;
        });

        document.getElementById('gameForm').addEventListener('submit', function(e) {
            e.preventDefault();
            const formData = new FormData(e.target);
            answers[players[currentPlayerIndex]] = Object.fromEntries(formData.entries());
            
            if (currentPlayerIndex < players.length - 1) {
                currentPlayerIndex++;
                showPlayerForm();
            } else {
                showResults();
            }
        });

        function showResults() {
            document.getElementById('playerForm').classList.add('hidden');
            document.getElementById('results').classList.remove('hidden');
            
            const resultsTable = document.getElementById('resultsTable');
            resultsTable.innerHTML = `
                <tr>
                    <th>קטגוריה</th>
                    ${players.map(player => `<th>${player}</th>`).join('')}
                </tr>
            `;
            
            categories.forEach((category, index) => {
                let row = `<tr><td>${hebrewCategories[index]}</td>`;
                players.forEach(player => {
                    const answer = answers[player][category] || '-';
                    const score = calculateScore(category, player);
                    row += `<td class="fade-in" style="animation-delay: ${index * 0.2}s">
                                ${answer} (${score})
                            </td>`;
                    scores[player] += score;
                });
                row += '</tr>';
                resultsTable.innerHTML += row;
            });

            updateScoreboard();
        }

        function calculateScore(category, player) {
            const playerAnswer = answers[player][category].toLowerCase();
            if (!playerAnswer || playerAnswer[0] !== currentLetter) return 0;
