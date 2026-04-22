<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Sports Mystery</title>
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    <style>
        /* BACKGROUND SETTINGS */
        #master-background {
            position: fixed;
            top: 0; left: 0;
            width: 100vw; height: 100vh;
            /* Updated to your new repo link */
            background-image: url('https://raw.githubusercontent.com/marionbunyi/sports-warm-up/4e541bf7ee299f2edd5aada8047548feaefd5746/backgroundphoto.jpg');
            background-size: cover;
            background-position: center;
            background-repeat: no-repeat;
            z-index: -100;
            opacity: 0.25; /* Makes the background faint */
            background-color: #000; 
        }

        html, body {
            margin: 0; padding: 0; width: 100%; height: 100%;
            overflow: hidden; 
            font-family: 'Arial Rounded MT Bold', 'Helvetica', sans-serif; user-select: none;
            background-color: #0f172a; 
        }

        body { display: flex; flex-direction: column; height: 100dvh; position: relative; }

        #top-bar {
            position: absolute; top: 10px; left: 10px;
            background: rgba(0,0,0,0.8); border: 2px solid #FBD000;
            border-radius: 10px; padding: 5px; display: flex; gap: 8px; z-index: 100;
        }

        #timer-box {
            position: absolute; top: 10px; right: 10px;
            background: rgba(0,0,0,0.8); border: 2px solid #FBD000;
            border-radius: 10px; padding: 5px 15px; font-size: 18px; color: #FBD000;
            z-index: 100;
        }
        
        .nav-btn {
            background: #43ad2e; color: white; border: none;
            padding: 8px 12px; border-radius: 5px; cursor: pointer; font-weight: bold;
        }

        #header {
            font-size: clamp(28px, 5vh, 45px); text-align: center; 
            margin-top: 65px; margin-bottom: 10px; 
            text-shadow: 3px 3px 0 #E52521;
            color: white; flex-shrink: 0;
        }

        #game-container {
            display: grid; grid-template-columns: 1.6fr 1.4fr;
            gap: 15px; width: 95%; max-width: 1300px;
            margin: 0 auto; flex-grow: 1; padding-bottom: 20px; min-height: 0;
        }
        
        #picture-box {
            position: relative; border: 8px solid #FBD000;
            background-size: cover; background-position: center;
            box-shadow: 0 0 30px rgba(0,0,0,0.8); border-radius: 20px;
            background-color: #000; height: 100%;
        }

        #side-panel { display: flex; flex-direction: column; gap: 10px; height: 100%; }
        
        .panel-section {
            background: rgba(0, 0, 0, 0.8); border: 3px solid #FBD000; 
            border-radius: 15px; padding: 10px; text-align: center; color: white;
        }

        #clue-display { flex: 2; display: flex; flex-direction: column; align-items: center; justify-content: center; } 
        #bank-section { flex: 0.8; display: flex; align-items: center; justify-content: center; }
        #answer-section { flex: 0.8; display: flex; align-items: center; justify-content: center; }

        #clue-img { 
            max-width: 85%; max-height: 200px; 
            display: none; border: 3px solid white; border-radius: 10px; margin-top: 10px;
        }

        /* Letters floating on the main picture */
        .hidden-letter {
            position: absolute; width: 45px; height: 45px;
            background: rgba(255, 255, 255, 0.15); 
            border: 1px solid rgba(255, 255, 255, 0.4);
            border-radius: 50%; display: flex; justify-content: center; align-items: center;
            color: rgba(255, 255, 255, 0.6); 
            font-size: 24px; font-weight: bold;
            cursor: pointer; z-index: 5; transform: translate(-50%, -50%);
            transition: 0.2s;
        }
        .hidden-letter:hover { background: #FBD000; color: #000; transform: translate(-50%, -50%) scale(1.1); }

        /* Letters in the red bank */
        .clickable-letter {
            width: 42px; height: 42px; background: #E52521; color: white; border: 2px solid #000;
            display: flex; justify-content: center; align-items: center; font-size: 20px;
            cursor: pointer; border-radius: 8px; box-shadow: 2px 2px 0 #000; font-weight: bold;
        }

        #answer-zone { display: flex; gap: 5px; justify-content: center; flex-wrap: wrap; }

        .slot {
            width: 35px; height: 45px; border: 2px solid #FBD000; background: rgba(255,255,255,0.05);
            color: #FBD000; display: flex; justify-content: center; align-items: center;
            font-size: 22px; font-weight: bold; border-radius: 6px; cursor: pointer;
        }

        #action-btn {
            width: 100%; padding: 12px; font-size: 20px; font-weight: bold; cursor: pointer;
            border-radius: 10px; border: none; background: #43ad2e; color: white;
            box-shadow: 0 4px 0 #2d7a1f;
        }

        #start-overlay, #victory-overlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.95); z-index: 2000; display: flex;
            flex-direction: column; align-items: center; justify-content: center;
        }
    </style>
</head>
<body>

<div id="master-background"></div>

<div id="start-overlay">
    <h1 style="color:white; font-size: clamp(40px, 8vw, 80px); text-shadow: 5px 5px 0 #E52521;">SPORTS MYSTERY</h1>
    <button style="padding: 15px 50px; font-size: 25px; background: #43ad2e; color: white; border: 4px solid #FBD000; border-radius: 15px; cursor: pointer;" onclick="startGame()">PLAY GAME</button>
</div>

<div id="top-bar">
    <button class="nav-btn" onclick="prevRound()">⬅️</button>
    <button class="nav-btn" onclick="skipRound()">➡️</button>
</div>

<div id="timer-box">00:00</div>
<div id="header">LETTERS TO FIND: 0</div>

<div id="game-container">
    <div id="picture-box"></div>
    <div id="side-panel">
        <div class="panel-section" id="clue-display">
            <h3 style="margin: 0; color: #FBD000; font-size: 22px;">CLUE</h3>
            <div id="lock-msg" style="margin-top:10px;">Find the hidden letters!</div>
            <img id="clue-img" src="" alt="Clue">
        </div>
        
        <div class="panel-section" id="bank-section">
            <div id="letter-bank" style="display:flex; flex-wrap:wrap; gap:8px; justify-content:center;"></div>
        </div>

        <button id="action-btn" onclick="checkAnswer()">CHECK ANSWER</button>

        <div class="panel-section" id="answer-section">
            <div id="answer-zone"></div>
        </div>
    </div>
</div>

<div id="victory-overlay" style="display:none;">
    <h1 style="font-size: 50px; color: #FBD000;">STRIKE! 🏆</h1>
    <p id="time-flash" style="font-size: 25px; color: white; margin-bottom: 20px;"></p>
    <button style="padding: 15px 40px; font-size: 20px; background:#43ad2e; color:white; border:none; border-radius:12px; cursor:pointer;" onclick="handleNextClick()">NEXT ROUND</button>
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

    const safePos = [{t:15,l:20}, {t:25,l:75}, {t:75,l:25}, {t:85,l:80}, {t:50,l:50}, {t:35,l:45}, {t:65,l:55}, {t:45,l:15}, {t:55,l:85}, {t:10,l:60}, {t:90,l:40}];

    let currentRoundIndex = 0;
    let foundLetters = [];
    let lettersLeft = 0;
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
        lettersLeft = sport.name.length;
        foundLetters = [];
        clearInterval(timerInterval);
        roundStartTime = Date.now();
        timerInterval = setInterval(updateTimer, 1000);

        document.getElementById('header').innerText = `LETTERS TO FIND: ${lettersLeft}`;
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
                lettersLeft--;
                document.getElementById('header').innerText = `LETTERS TO FIND: ${lettersLeft}`;
                if(lettersLeft === 0) {
                    document.getElementById('lock-msg').style.display = 'none';
                    document.getElementById('clue-img').style.display = 'block';
                }
                renderBank();
            };
            picBox.appendChild(l);
        });
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
