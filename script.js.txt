// متن‌های نمونه
const texts = [
    "The quick brown fox jumps over the lazy dog",
    "Python is a great programming language for beginners",
    "Artificial intelligence is changing the world rapidly",
    "Practice makes perfect in typing and programming",
    "Every great developer started as a beginner coder",
    "Learning to code opens doors to amazing opportunities",
    "The best way to predict the future is to create it",
    "Code is like humor when you have to explain it is bad",
    "First solve the problem then write the code",
    "Simplicity is the soul of efficiency in programming",
    "JavaScript is the language of the web",
    "HTML and CSS are the building blocks of websites",
    "Typing fast is a valuable skill in today world",
    "Consistent practice leads to mastery",
    "Every expert was once a beginner"
];

// متغیرهای بازی
let currentText = "";
let userInput = "";
let startTime = 0;
let timerInterval = null;
let isPlaying = false;
let records = JSON.parse(localStorage.getItem('typingRecords')) || [];

// المان‌های DOM
const menuScreen = document.getElementById('menu');
const gameScreen = document.getElementById('game');
const resultScreen = document.getElementById('result');
const textDisplay = document.getElementById('textDisplay');
const userInputArea = document.getElementById('userInput');
const timerDisplay = document.getElementById('timer');
const currentWPMDisplay = document.getElementById('currentWPM');
const currentAccuracyDisplay = document.getElementById('currentAccuracy');
const progress = document.getElementById('progress');
const progressText = document.getElementById('progressText');
const finalWPMDisplay = document.getElementById('finalWPM');
const finalAccuracyDisplay = document.getElementById('finalAccuracy');
const finalTimeDisplay = document.getElementById('finalTime');
const finalCharsDisplay = document.getElementById('finalChars');
const resultMessage = document.getElementById('resultMessage');
const bestRecordsDisplay = document.getElementById('bestRecords');
const leaderboardDisplay = document.getElementById('leaderboard');

// دکمه‌ها
const startBtn = document.getElementById('startBtn');
const backBtn = document.getElementById('backBtn');
const restartBtn = document.getElementById('restartBtn');
const retryBtn = document.getElementById('retryBtn');
const menuBtn = document.getElementById('menuBtn');

// رویدادها
startBtn.addEventListener('click', startGame);
backBtn.addEventListener('click', showMenu);
restartBtn.addEventListener('click', startGame);
retryBtn.addEventListener('click', startGame);
menuBtn.addEventListener('click', showMenu);
userInputArea.addEventListener('input', handleInput);
userInputArea.addEventListener('keydown', handleKeydown);

// شروع بازی
function startGame() {
    currentText = texts[Math.floor(Math.random() * texts.length)];
    userInput = "";
    isPlaying = false;
    
    if (timerInterval) {
        clearInterval(timerInterval);
        timerInterval = null;
    }
    
    showScreen('game');
    displayText();
    userInputArea.value = "";
    userInputArea.disabled = false;
    userInputArea.focus();
    
    timerDisplay.textContent = "0.0";
    currentWPMDisplay.textContent = "0";
    currentAccuracyDisplay.textContent = "100";
    progress.style.width = "0%";
    progressText.textContent = "0%";
}

// نمایش متن با رنگ‌بندی
function displayText() {
    textDisplay.innerHTML = "";
    
    for (let i = 0; i < currentText.length; i++) {
        const span = document.createElement('span');
        span.className = 'char';
        span.textContent = currentText[i];
        
        if (i < userInput.length) {
            if (userInput[i] === currentText[i]) {
                span.classList.add('correct');
            } else {
                span.classList.add('incorrect');
            }
        } else if (i === userInput.length) {
            span.classList.add('current');
        }
        
        textDisplay.appendChild(span);
    }
}

// مدیریت ورودی
function handleInput(e) {
    if (!isPlaying && userInputArea.value.length > 0) {
        startTimer();
        isPlaying = true;
    }
    
    userInput = userInputArea.value;
    displayText();
    updateStats();
    
    // بررسی پایان
    if (userInput.length >= currentText.length) {
        finishGame();
    }
}

// مدیریت کلیدها
function handleKeydown(e) {
    if (e.key === 'Tab') {
        e.preventDefault();
    }
}

// شروع تایمر
function startTimer() {
    startTime = Date.now();
    timerInterval = setInterval(updateTimer, 100);
}

// بروزرسانی تایمر
function updateTimer() {
    const elapsed = (Date.now() - startTime) / 1000;
    timerDisplay.textContent = elapsed.toFixed(1);
}

// بروزرسانی آمار
function updateStats() {
    // محاسبه WPM
    const elapsed = (Date.now() - startTime) / 1000;
    if (elapsed > 0) {
        const words = userInput.split(' ').length;
        const wpm = Math.round((words / elapsed) * 60);
        currentWPMDisplay.textContent = wpm;
    }
    
    // محاسبه دقت
    let correct = 0;
    for (let i = 0; i < userInput.length; i++) {
        if (userInput[i] === currentText[i]) {
            correct++;
        }
    }
    const accuracy = userInput.length > 0 ? Math.round((correct / userInput.length) * 100) : 100;
    currentAccuracyDisplay.textContent = accuracy;
    
    // نوار پیشرفت
    const progressPercent = Math.round((userInput.length / currentText.length) * 100);
    progress.style.width = progressPercent + "%";
    progressText.textContent = progressPercent + "%";
}

// پایان بازی
function finishGame() {
    clearInterval(timerInterval);
    userInputArea.disabled = true;
    
    const elapsed = (Date.now() - startTime) / 1000;
    const words = currentText.split(' ').length;
    const wpm = Math.round((words / elapsed) * 60);
    
    let correct = 0;
    for (let i = 0; i < userInput.length; i++) {
        if (userInput[i] === currentText[i]) {
            correct++;
        }
    }
    const accuracy = Math.round((correct / currentText.length) * 100);
    
    // ذخیره رکورد
    const record = {
        wpm: wpm,
        accuracy: accuracy,
        time: elapsed.toFixed(1),
        date: new Date().toLocaleDateString()
    };
    
    records.push(record);
    records.sort((a, b) => b.wpm - a.wpm);
    if (records.length > 10) records = records.slice(0, 10);
    
    localStorage.setItem('typingRecords', JSON.stringify(records));
    
    // نمایش نتیجه
    showResult(record);
}

// نمایش نتیجه
function showResult(record) {
    finalWPMDisplay.textContent = record.wpm;
    finalAccuracyDisplay.textContent = record.accuracy + "%";
    finalTimeDisplay.textContent = record.time + "s";
    finalCharsDisplay.textContent = currentText.length;
    
    // پیام نتیجه
    if (record.wpm > 60) {
        resultMessage.textContent = "Amazing! You are a typing master!";
    } else if (record.wpm > 40) {
        resultMessage.textContent = "Great job! Keep practicing!";
    } else if (record.wpm > 25) {
        resultMessage.textContent = "Good start! Practice more!";
    } else {
        resultMessage.textContent = "Keep trying! Practice makes perfect!";
    }
    
    // نمایش لیدربورد
    updateLeaderboard();
    
    showScreen('result');
}

// بروزرسانی لیدربورد
function updateLeaderboard() {
    leaderboardDisplay.innerHTML = "";
    
    records.slice(0, 5).forEach((record, index) => {
        const item = document.createElement('div');
        item.className = 'leaderboard-item';
        
        item.innerHTML = `
            <span class="leaderboard-rank">#${index + 1}</span>
            <span class="leaderboard-name">${record.date}</span>
            <span class="leaderboard-score">${record.wpm} WPM | ${record.accuracy}%</span>
        `;
        
        leaderboardDisplay.appendChild(item);
    });
}

// نمایش رکوردهای بهترین
function updateBestRecords() {
    if (records.length === 0) {
        bestRecordsDisplay.innerHTML = "<p>No records yet. Play to set your first record!</p>";
        return;
    }
    
    const bestWPM = records[0].wpm;
    const bestAccuracy = Math.max(...records.map(r => r.accuracy));
    const bestTime = Math.min(...records.map(r => parseFloat(r.time)));
    
    bestRecordsDisplay.innerHTML = `
        <div style="display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; text-align: center;">
            <div>
                <div style="font-size: 1.5rem; font-weight: bold; color: #2ecc71;">${bestWPM}</div>
                <div style="font-size: 0.9rem; color: #7f8c8d;">Best WPM</div>
            </div>
            <div>
                <div style="font-size: 1.5rem; font-weight: bold; color: #3498db;">${bestAccuracy}%</div>
                <div style="font-size: 0.9rem; color: #7f8c8d;">Best Accuracy</div>
            </div>
            <div>
                <div style="font-size: 1.5rem; font-weight: bold; color: #e74c3c;">${bestTime}s</div>
                <div style="font-size: 0.9rem; color: #7f8c8d;">Best Time</div>
            </div>
        </div>
    `;
}

// نمایش صفحه
function showScreen(screen) {
    menuScreen.classList.remove('active');
    gameScreen.classList.remove('active');
    resultScreen.classList.remove('active');
    
    switch(screen) {
        case 'menu':
            menuScreen.classList.add('active');
            updateBestRecords();
            break;
        case 'game':
            gameScreen.classList.add('active');
            break;
        case 'result':
            resultScreen.classList.add('active');
            break;
    }
}

// نمایش منو
function showMenu() {
    if (timerInterval) {
        clearInterval(timerInterval);
        timerInterval = null;
    }
    showScreen('menu');
}

// شروع اولیه
updateBestRecords();
showScreen('menu');