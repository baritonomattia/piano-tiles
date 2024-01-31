<script>
    
  let rows = [];
  let score = 0;
  let gameover = false;
  let timeRemaining = 120; // 2 minuti
  let timerInterval;
  let keydownListener; // Aggiunto un riferimento all'ascoltatore per la rimozione successiva

  function generateRow() {
    let row = new Array(4).fill("white");
    let pos = Math.trunc(Math.random() * 4);
    row[pos] = "black";
    return row;
  }

  function fillRows() {
    for (let i = 0; i < 4; i++) {
      rows.push(generateRow());
    }
  }

  function tapped(i, j) {
    if (i !== rows.length - 1 || rows[i][j] === "white") {
      gameover = true;
      rows[i][j] = "red";
      endGame();
    } else {
      rows.splice(i);
      rows = [generateRow(), ...rows];
      score++;
      if (!timerInterval) {
        startTimer();
      }
    }
  }

  function handleKeyDown(event) {
    if (gameover) {
      // Se il gioco è già finito, ignora gli input da tastiera
      return;
    }

    const keyMap = { KeyA: 0, KeyS: 1, KeyD: 2, KeyF: 3 };
    const rowIndex = rows.length - 1;
    const columnIndex = keyMap[event.code];
    if (columnIndex !== undefined) {
      tapped(rowIndex, columnIndex);
    }
  }

  function updateTimerDisplay() {
    const timerDisplay = document.getElementById("timer");
    timerDisplay.textContent = `Time: ${formatTime(timeRemaining)}`;
  }

  function formatTime(seconds) {
    const minutes = Math.floor(seconds / 60);
    const remainingSeconds = seconds % 60;
    return `${minutes}:${remainingSeconds < 10 ? "0" : ""}${remainingSeconds}`;
  }

  function startTimer() {
    timerInterval = setInterval(function () {
      timeRemaining--;
      updateTimerDisplay();

      if (timeRemaining <= 0) {
        endGame();
      }
    }, 1000);
  }

  function endGame() {
    if (keydownListener) {
      // Rimuovi l'ascoltatore di eventi keydown solo se è stato precedentemente registrato
      document.removeEventListener('keydown', handleKeyDown);
      keydownListener = null;
    }

    gameover = true;
    clearInterval(timerInterval);
    timerInterval = null; // Resetta il timerInterval

    const resultMessage = document.getElementById('result-message');
    const resultScore = document.getElementById('result-score');

    resultMessage.textContent = `Tempo finito`;
    resultScore.textContent = `Punteggio: ${score}`;
  }

  function restart() {
    score = 0;
    gameover = false;
    rows = [];
    fillRows();
    timeRemaining = 120; // 2 minuti
    updateTimerDisplay();

    // Rimuovi l'ascoltatore di eventi keydown se è stato precedentemente registrato
    if (keydownListener) {
      document.removeEventListener('keydown', handleKeyDown);
      keydownListener = null;
    }

    // Aggiungi nuovamente l'ascoltatore di eventi keydown
    keydownListener = document.addEventListener('keydown', handleKeyDown);
  }
  fillRows();

  // Aggiungi il gestore dell'evento keydown al documento solo se non è già stato registrato
  if (!keydownListener) {
    keydownListener = document.addEventListener('keydown', handleKeyDown);
  }
</script>



<style>
  .app {
    position: fixed;
    top: 0px;
    left: 50%;
    transform: translateX(-50%);
    width: 100%;
    height: 100%;
    max-width: 400px;
  }

  .app .header {
    position: sticky;
    top: 0px;
    left: 0px;
    width: 100%;
    height: 50px;
    background: #0badea;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0px 20px;
    color: #fff;
  }

  .app .game {
    width: 110%;
    height: calc(100% - 50px);
    background: #fff;
  }

  .app .game .row {
    display: flex;
    width: 100%;
    height: calc(100% / 4);
  }

  .app .game .row .box {
    flex: 1;
    border: 1px solid #555;
    cursor: pointer;
  }

  .app .game .row .box.black {
    background: #111;
  }

  .app .game .row .box.red {
    animation: blinkRed 500ms ease-in-out infinite;
  }

  @keyframes blinkRed {
    0%, 100% {
        background: #fff;
    }
    50% {
        background: tomato;
    }
  }

  .result {
    position: absolute;
    top: 0px;
    left: 0px;
    width: 110%;
    height: 100%;
    background: rgb(0, 0, 0, 0.8);
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    gap: 10px;
  }

  .result h2 {
    font-size: 40px;
    color: #fff;
  }

  .result p {
    font-size: 20px;
    margin-bottom: 10px;
    color: #eee;
  }

  .result button {
    padding: 10px 20px;
    cursor: pointer;
    font-size: 16px;
    border: 2px solid #fff;
    color: #fff;
    outline: none;
    font-weight: 600;
    background: transparent;
  }
</style>

<main class="app">
  <div class="header">
    <h4>PianoTiles</h4>
    <p id="timer">Time: 2:00</p>
    <p>Score: {score}</p>
  </div>
  <div class="game">
    {#each rows as row, i}
    <div class="row">
      {#each row as box, j}
      <div class={"box " + box} on:click={() => tapped(i, j)}></div>
      {/each}
    </div>
    {/each}
  </div>
  {#if gameover}
  <div class="result">
    <h2 id="result-message">Tempo finito</h2>
    <p id="result-score">Punteggio: {score}</p>
    <button on:click={restart}>Restart</button>
  </div>
  {/if}
</main>
