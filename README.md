<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Sports Mystery</title>
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    <style>
        /* 1. BACKGROUND LAYER */
        #master-background {
            position: fixed;
            top: 0; left: 0;
            width: 100vw; height: 100vh;
            background-image: url('https://raw.githubusercontent.com/marionbunyi/sports-warm-up/4e541bf7ee299f2edd5aada8047548feaefd5746/backgroundphoto.jpg');
            background-size: cover;
            background-position: center;
            z-index: -100;
            opacity: 0.25;
            background-color: #000; 
        }

        html, body {
            margin: 0; padding: 0; width: 100%; height: 100%;
            overflow: hidden; 
            font-family: 'Arial Rounded MT Bold', sans-serif;
            background-color: #0f172a;
            color: white;
        }

        /* 2. TOP NAVIGATION */
        #top-controls {
            position: fixed;
            top: 15px; right: 20px;
            display: flex; align-items: center; gap: 15px;
            background: rgba(0,0,0,0.85);
            padding: 10px 15px;
            border: 3px solid #FBD000;
            border-radius: 15px;
            z-index: 1000;
        }
        #timer-box { font-size: 24px; color: #FBD000; font-weight: bold; min-width: 65px; text-align: center; }
        .nav-btn {
            background: #43ad2e; color: white; border: none;
            padding: 10px 18px; border-radius: 8px; font-weight: bold; cursor: pointer;
        }

        /* 3. MAIN LAYOUT (ADJUSTED FOR LONG WORDS) */
        #main-wrapper {
            display: grid;
            grid-template-columns: 1fr 450px; /* Widened right column for long words */
            width: 100vw; height: 100vh;
            padding-top: 85px; 
            box-sizing: border-box;
        }

        #left-col { padding: 15px; display: flex; align-items: center; justify-content: center; }
        
        #picture-box {
            position: relative;
            width: 100%; height: 100%;
            border: 8px solid #FBD000;
            border-radius: 25px;
            background-size: cover;
            background-position: center;
            background-color: #000;
        }

        /* 4. SIDE PANEL STABILITY */
        #right-col {
            padding: 15px;
            display: flex;
            flex-direction: column;
            gap: 12px;
        }

        .panel-section {
            background: rgba(0,0,0,0.85);
            border: 3px solid #FBD000;
            border-radius: 20px;
            padding: 15px;
            text-align: center;
            flex-shrink: 0;
        }

        /* Fixed Heights to prevent jumping when content changes */
        #clue-display { height: 230px; display: flex; flex-direction: column; align-items: center; justify-content: center; }
        #bank-section { height: 130px; display: flex; align-items: center; justify-content: center; }
        #answer-section { height: 110px; display: flex; align-items: center; justify-content: center; }

        #header { font-size: 24px; text-shadow: 2px 2px 0 #E52521; margin-bottom: 5px; text-align: center; }

        /* 5. LETTERS & SLOTS */
        .hidden-letter {
            position: absolute; width: 50px; height: 50px;
            background: rgba(255, 255, 255, 0.15);
            border: 2px solid rgba(255, 255, 255, 0.4);
            border-radius: 50%; display: flex; justify-content: center; align-items: center;
            color: rgba(255, 255, 255, 0.7); font-size: 28px; font-weight: bold;
            transform: translate(-50%, -50%); cursor: pointer;
        }

        .clickable-letter {
            width: 48px; height: 48px;
            background: #E52521; border: 2px solid #000;
            border-radius: 10px; display: flex; justify-content: center; align-items: center;
            font-size: 24px; font-weight: bold; cursor: pointer;
            box-shadow: 3px 3px 0 #000; flex-shrink: 0;
        }

        #answer-zone {
            display: flex; flex-direction: row; flex-wrap: nowrap; /* Forces one line */
            gap: 4px; justify-content: center; width: 100%;
        }

        .slot {
            width: 33px; height: 50px; /* Slimmer to fit 11-12 letters in one line */
            border: 2px solid #FBD000; border-radius: 6px;
            display: flex; justify-content: center; align-items: center;
            font-size: 24px; color: #FBD000; font-weight: bold;
            flex-shrink: 0; background: rgba(255,255,255,0.1);
        }

        #letter-bank { display: flex; flex-wrap: wrap; gap: 8px; justify-content: center; width: 100%; }

        #action-btn {
            width: 100%; padding: 18px; font-size: 24px; font-weight: bold;
            background: #43ad2e; color: white; border: none; border-radius: 15px;
            box-shadow: 0 5px 0 #2d7a1f; cursor: pointer; flex-shrink: 0;
        }

        #clue-img { 
            max-width: 95%; max-height: 160px; 
            display: none; border: 3px solid white; border-radius: 12px; margin-top: 10px;
        }

        /* OVERLAYS */
        #start-overlay, #victory-overlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.98); z-index: 5000; display: flex;
            flex-direction: column; align-items: center; justify-content: center;
        }
    </style>
</head>
<body>

<div id="master-background"></div>

<div id="start-overlay">
    <h1 style="font-size: clamp(40px, 10vw, 80px); text-shadow: 6px 6px 0 #E52521; margin-bottom: 20px;">SPORTS MYSTERY</h1>
    <button style="padding: 20px 60px; font-size: 30px; background: #43ad2e; border: 4px solid #FBD000; border-radius: 25px; color: white; cursor: pointer;" onclick="startGame()">PLAY NOW</button>
</div>

<div id="top-controls">
    <button class="nav-btn" onclick="prevRound()">BACK</button>
    <div id="timer-box">00:00</div>
    <button class="nav-btn" onclick="skipRound()">NEXT</button>
</div>

<div id="main-wrapper">
    <div id="left-col">
        <div id="picture-box"></div>
    </div>

    <div id="right-col">
        <div id="header">LETTERS TO FIND: <span id="find-count">0</span></div>
        
        <div class="panel-section" id="clue-display">
            <h3 style="margin:0; color:#FBD000;">CLUE</h3>
            <p id="lock-msg">Collect all letters to reveal the sport!</p>
            <img id="clue-img" src="">
        </div>

        <div class="panel-section" id="bank-section">
            <div id="letter-bank"></div>
        </div>

        <button id="action-btn" onclick="checkAnswer()">CHECK ANSWER</button>

        <div class="panel-section" id="answer-section">
            <div id="answer-zone"></div>
        </div>
    </div>
</div>

<div id="victory-overlay" style="display:none;">
    <h1 style="font-size: 70px; color: #FBD000;">STRIKE! 🏆</h1>
    <p id="time-flash" style="font-size: 30px; margin-bottom: 25px;"></p>
    <button style="padding: 20px 50px; font-size: 24px; background:#43ad2e; color:white; border-radius:15px; border:none; cursor:pointer;" onclick="handleNextClick()">NEXT LEVEL</button>
</div>

<audio id="bg-music" loop src="https://raw.githubusercontent.com/marionbunyi/sports-mario/main/Energetic%20Rock%20Background%20Music%20For%20Sports%20%26%20Workout%20Videos.mp3"></audio>
<audio id="snd-click" src="https://raw.githubusercontent.com/marionbunyi/sports-mario/main/Mouse%20Click%20-%20Sound%20Effect%20(HD).mp3"></audio>
<audio id="snd-win" src="https://raw.githubusercontent.com/marionbunyi/sports-mario/main/Correct%20answer%20sound%20effect%20_%20No%20copyright.mp3"></audio>

<script>
    const sports = [
        { name: "SWIMMING", img: "swimming" },
        { name: "BASKETBALL", img: "basketball" },
        { name: "SOCCER", img: "soccer" },
        { name: "TENNIS", img: "tennis" },
        { name: "BADMINTON", img: "badminton" },
        { name: "BASEBALL", img: "baseball" },
        { name: "TABLETENNIS", img: "table tennis" },
        { name: "VOLLEYBALL", img: "volleyball" }
    ];

    // Added more random positions for variety
    const safePos = [
        {t:15,l:15}, {t:20,l:85}, {t:85,l:15}, {t:80,l:80}, 
        {t:50,l:50}, {t:30,l:50}, {t:70,l:50}, {t:50,l:20}, 
        {t:50,l:80}, {t:10,l:50}, {t:90,l:50}, {t:30,l:30}, {t:70,l:70}
    ];

    let currentRoundIndex = 0;
    let foundLetters = [];
    let lettersToFind = 0;
    let timerInterval;
    let roundStartTime;

    function startGame() {
        document.getElementById('start-overlay').style.display = 'none';
        const music = document.getElementById('bg-music');
        music.volume = 0.1;
        music.play();
        initRound();
    }

    function initRound() {
        const sport = sports[currentRoundIndex];
        lettersToFind = sport.name.length;
        foundLetters = [];
        clearInterval(timerInterval);
        roundStartTime = Date.now();
        timerInterval = setInterval(updateTimer, 1000);

        updateCount();
        document.getElementById('picture-box').style.backgroundImage = `url('https://raw.githubusercontent.com/marionbunyi/sports-mario/main/hidden%20${sport.img}.png')`;
        document.getElementById('clue-img').src = `https://raw.githubusercontent.com/marionbunyi/sports-mario/main/${sport.img}%20clue.png`;
        document.getElementById('clue-img').style.display = 'none';
        document.getElementById('lock-msg').style.display = 'block';
        document.getElementById('victory-overlay').style.display = 'none';
        
        document.getElementById('letter-bank').innerHTML = '';
        const zone = document.getElementById('answer-zone');
        zone.innerHTML = '';

        sport.name.split('').forEach(() => {
            const s = document.createElement('div');
            s.className = 'slot';
            s.onclick = function() {
                if(this.innerText) {
                    const char = this.innerText;
                    const item = foundLetters.find(i => i.char === char && i.used);
                    if(item) { item.used = false; this.innerText = ""; renderBank(); }
                }
            };
            zone.appendChild(s);
        });

        const picBox = document.getElementById('picture-box');
        picBox.innerHTML = '';
        let pos = [...safePos].sort(() => 0.5 - Math.random());
        sport.name.split('').forEach((char, i) => {
            const l = document.createElement('div');
            l.className = 'hidden-letter';
            l.innerText = char;
            l.style.top = pos[i].t + "%";
            l.style.left = pos[i].l + "%";
            l.onclick = () => {
                document.getElementById('snd-click').play();
                l.style.display = 'none';
                foundLetters.push({char, used: false});
                lettersToFind--;
                updateCount();
                if(lettersToFind === 0) {
                    document.getElementById('lock-msg').style.display = 'none';
                    document.getElementById('clue-img').style.display = 'block';
                }
                renderBank();
            };
            picBox.appendChild(l);
        });
    }

    function updateCount() {
        document.getElementById('find-count').innerText = lettersToFind;
    }

    function renderBank() {
        const bank = document.getElementById('letter-bank');
        bank.innerHTML = '';
        foundLetters.forEach(item => {
            if(!item.used) {
                const d = document.createElement('div');
                d.className = 'clickable-letter';
                d.innerText = item.char;
                d.onclick = () => {
                    const empty = Array.from(document.querySelectorAll('.slot')).find(s => !s.innerText);
                    if(empty) { empty.innerText = item.char; item.used = true; renderBank(); }
                };
                bank.appendChild(d);
            }
        });
    }

    function updateTimer() {
        const diff = Date.now() - roundStartTime;
        const mins = Math.floor(diff / 60000).toString().padStart(2, '0');
        const secs = Math.floor((diff % 60000) / 1000).toString().padStart(2, '0');
        document.getElementById('timer-box').innerText = `${mins}:${secs}`;
    }

    function checkAnswer() {
        let guess = "";
        document.querySelectorAll('.slot').forEach(s => guess += s.innerText);
        if(guess === sports[currentRoundIndex].name) {
            clearInterval(timerInterval);
            document.getElementById('snd-win').play();
            document.getElementById('time-flash').innerText = `TIME: ${document.getElementById('timer-box').innerText}`;
            document.getElementById('victory-overlay').style.display = 'flex';
            confetti({ particleCount: 200, spread: 90, origin: { y: 0.6 } });
        }
    }

    function handleNextClick() { currentRoundIndex = (currentRoundIndex + 1) % sports.length; initRound(); }
    function skipRound() { currentRoundIndex = (currentRoundIndex + 1) % sports.length; initRound(); }
    function prevRound() { currentRoundIndex = (currentRoundIndex - 1 + sports.length) % sports.length; initRound(); }
</script>
</body>
</html>
