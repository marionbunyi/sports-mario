<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Sports Mystery</title>
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    <style>
        * { box-sizing: border-box; }
        body, html {
            margin: 0; padding: 0; width: 100%; height: 100%;
            overflow-x: hidden; font-family: 'Arial Rounded MT Bold', sans-serif;
            background-color: #000; color: white;
        }

        /* 1. YOUR BACKGROUND PHOTO (TRANSPARENT) */
        #bg-layer {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background-image: url('https://raw.githubusercontent.com/marionbunyi/sports-warm-up/main/backgroundphoto.jpg');
            background-size: cover; background-position: center;
            opacity: 0.25; /* Adjusted transparency */
            z-index: -1;
        }

        /* 2. TOP NAV */
        #nav-bar {
            position: fixed; top: 0; width: 100%; height: 65px;
            display: flex; align-items: center; justify-content: space-between;
            padding: 0 20px; background: rgba(0,0,0,0.85);
            border-bottom: 3px solid #FBD000; z-index: 1000;
        }
        #timer-display { font-size: 22px; color: #FBD000; font-weight: bold; min-width: 80px; text-align: center; }
        .btn-nav { background: #43ad2e; color: white; border: none; padding: 10px 20px; border-radius: 10px; font-weight: bold; cursor: pointer; }

        /* 3. RESPONSIVE MAIN CONTAINER */
        #main-container {
            display: flex; flex-direction: row;
            padding: 80px 20px 20px 20px; width: 100vw; height: 100vh; gap: 20px;
        }

        #photo-section {
            flex: 1.2; border: 6px solid #FBD000; border-radius: 25px;
            position: relative; background-color: #111;
            background-size: cover; background-position: center;
        }

        #control-panel {
            flex: 0.8; max-width: 450px; display: flex; flex-direction: column; gap: 12px;
        }

        /* STABLE BOXES */
        .card {
            background: rgba(0,0,0,0.9); border: 3px solid #FBD000;
            border-radius: 20px; padding: 15px; text-align: center;
            display: flex; flex-direction: column; align-items: center; justify-content: center;
            flex-shrink: 0;
        }

        #clue-card { height: 35%; }
        #bank-card { height: 25%; }
        #answer-card { height: 15%; }

        /* MOBILE OVERRIDE */
        @media (max-width: 850px) {
            #main-container { flex-direction: column; height: auto; overflow-y: auto; }
            #photo-section { width: 100%; height: 400px; flex: none; }
            #control-panel { width: 100%; max-width: none; flex: none; padding-bottom: 40px; }
        }

        /* 4. GAME ELEMENTS */
        .hidden-letter {
            position: absolute; width: 46px; height: 46px;
            background: rgba(255,255,255,0.1); border: 1px solid rgba(255,255,255,0.2);
            border-radius: 50%; display: flex; align-items: center; justify-content: center;
            color: rgba(255,255,255,0.3); font-weight: bold; cursor: pointer; transform: translate(-50%, -50%);
        }

        .bank-letter {
            width: 46px; height: 46px; background: #E52521;
            border: 2px solid #000; border-radius: 10px;
            display: flex; align-items: center; justify-content: center;
            font-weight: bold; font-size: 22px; cursor: pointer; box-shadow: 2px 2px 0 #000; margin: 4px;
        }

        #answer-zone { display: flex; flex-wrap: nowrap; gap: 4px; justify-content: center; width: 100%; }
        
        .slot {
            width: 34px; height: 48px; border: 2px solid #FBD000;
            border-radius: 8px; background: rgba(255,255,255,0.1);
            display: flex; align-items: center; justify-content: center;
            color: #FBD000; font-weight: bold; font-size: 24px; flex-shrink: 0;
        }

        #check-btn {
            width: 100%; padding: 18px; background: #43ad2e; color: white;
            border: none; border-radius: 15px; font-size: 24px; font-weight: bold;
            box-shadow: 0 5px 0 #2d7a1f; cursor: pointer;
        }

        #clue-img { max-height: 90%; max-width: 90%; border-radius: 12px; display: none; border: 2px solid white; }

        /* 5. VICTORY OVERLAY WITH TIME */
        #win-overlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.97); z-index: 2000;
            display: none; flex-direction: column; align-items: center; justify-content: center;
            text-align: center;
        }
    </style>
</head>
<body>

<div id="bg-layer"></div>

<div id="nav-bar">
    <button class="btn-nav" onclick="changeRound(-1)">BACK</button>
    <div id="timer-display">00:00</div>
    <button class="btn-nav" onclick="changeRound(1)">NEXT</button>
</div>

<div id="main-container">
    <div id="photo-section"></div>

    <div id="control-panel">
        <div style="text-align: center; font-size: 18px; color: #FBD000; font-weight: bold;">
            LETTERS LEFT TO FIND: <span id="find-count">0</span>
        </div>

        <div class="card" id="clue-card">
            <div id="lock-msg" style="color: #666;">Find all letters to reveal the sport!</div>
            <img id="clue-img" src="">
        </div>

        <div class="card" id="bank-card">
            <div id="letter-bank" style="display:flex; flex-wrap:wrap; justify-content:center;"></div>
        </div>

        <button id="check-btn" onclick="checkAnswer()">CHECK ANSWER</button>

        <div class="card" id="answer-card">
            <div id="answer-zone"></div>
        </div>
    </div>
</div>

<div id="win-overlay">
    <h1 style="font-size: 70px; color: #FBD000; margin-bottom: 10px;">STRIKE! 🏆</h1>
    <h2 id="victory-time" style="font-size: 30px; color: white; margin-bottom: 30px;">Time: 00:00</h2>
    <button class="btn-nav" style="padding: 20px 60px; font-size: 26px;" onclick="changeRound(1)">NEXT LEVEL</button>
</div>

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

    let currentIdx = 0;
    let found = [];
    let startTime;
    let timerRef;

    function init() {
        const s = sports[currentIdx];
        found = [];
        clearInterval(timerRef);
        startTime = Date.now();
        timerRef = setInterval(updateTimer, 1000);

        document.getElementById('find-count').innerText = s.name.length;
        document.getElementById('photo-section').style.backgroundImage = `url('https://raw.githubusercontent.com/marionbunyi/sports-mario/main/hidden%20${s.img}.png')`;
        document.getElementById('clue-img').src = `https://raw.githubusercontent.com/marionbunyi/sports-mario/main/${s.img}%20clue.png`;
        document.getElementById('clue-img').style.display = 'none';
        document.getElementById('lock-msg').style.display = 'block';
        document.getElementById('win-overlay').style.display = 'none';
        document.getElementById('letter-bank').innerHTML = '';
        
        const zone = document.getElementById('answer-zone');
        zone.innerHTML = '';
        s.name.split('').forEach(() => {
            let sl = document.createElement('div');
            sl.className = 'slot';
            sl.onclick = function() {
                if(this.innerText) {
                    let item = found.find(f => f.char === this.innerText && f.used);
                    if(item) { item.used = false; this.innerText = ''; renderBank(); }
                }
            };
            zone.appendChild(sl);
        });

        const pic = document.getElementById('photo-section');
        pic.innerHTML = '';
        s.name.split('').forEach(char => {
            let l = document.createElement('div');
            l.className = 'hidden-letter';
            l.innerText = char;
            l.style.top = (Math.random() * 75 + 12.5) + "%";
            l.style.left = (Math.random() * 75 + 12.5) + "%";
            l.onclick = () => {
                l.style.display = 'none';
                found.push({char, used: false});
                let remain = s.name.length - found.length;
                document.getElementById('find-count').innerText = remain;
                if(remain === 0) {
                    document.getElementById('lock-msg').style.display = 'none';
                    document.getElementById('clue-img').style.display = 'block';
                }
                renderBank();
            };
            pic.appendChild(l);
        });
    }

    function renderBank() {
        const b = document.getElementById('letter-bank');
        b.innerHTML = '';
        found.forEach(item => {
            if(!item.used) {
                let div = document.createElement('div');
                div.className = 'bank-letter';
                div.innerText = item.char;
                div.onclick = () => {
                    let empty = [...document.querySelectorAll('.slot')].find(s => !s.innerText);
                    if(empty) { empty.innerText = item.char; item.used = true; renderBank(); }
                };
                b.appendChild(div);
            }
        });
    }

    function updateTimer() {
        let d = Math.floor((Date.now() - startTime) / 1000);
        let m = Math.floor(d / 60).toString().padStart(2, '0');
        let s = (d % 60).toString().padStart(2, '0');
        document.getElementById('timer-display').innerText = `${m}:${s}`;
    }

    function checkAnswer() {
        let guess = [...document.querySelectorAll('.slot')].map(s => s.innerText).join('');
        if(guess === sports[currentIdx].name) {
            clearInterval(timerRef);
            document.getElementById('snd-win').play();
            document.getElementById('victory-time').innerText = "FINISH TIME: " + document.getElementById('timer-display').innerText;
            document.getElementById('win-overlay').style.display = 'flex';
            confetti({ particleCount: 200, spread: 80, origin: { y: 0.6 } });
        }
    }

    function changeRound(dir) {
        currentIdx = (currentIdx + dir + sports.length) % sports.length;
        init();
    }

    window.onload = init;
</script>
</body>
</html>
