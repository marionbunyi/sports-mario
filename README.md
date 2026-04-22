<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Sports Mystery</title>
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    <style>
        body, html {
            margin: 0; padding: 0; width: 100%; height: 100%;
            overflow: hidden; font-family: 'Arial Rounded MT Bold', sans-serif;
            background-color: #000;
        }

        /* 1. BACKGROUND PHOTO (FIXED & TRANSPARENT) */
        #bg-layer {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background-image: url('https://raw.githubusercontent.com/marionbunyi/sports-warm-up/main/backgroundphoto.jpg');
            background-size: cover; background-position: center;
            opacity: 0.25; /* Transparency level */
            z-index: -1;
        }

        /* 2. NAVIGATION TOP RIGHT */
        #top-right-nav {
            position: absolute; top: 15px; right: 20px;
            display: flex; align-items: center; gap: 10px;
            background: rgba(0,0,0,0.8); padding: 8px 15px;
            border: 2px solid #FBD000; border-radius: 12px; z-index: 100;
        }
        #timer { font-size: 22px; color: #FBD000; font-weight: bold; min-width: 60px; text-align: center; }
        .btn-nav { background: #43ad2e; color: white; border: none; padding: 8px 15px; border-radius: 6px; font-weight: bold; cursor: pointer; }

        /* 3. MAIN GAME LAYOUT */
        #game-wrapper {
            display: grid; grid-template-columns: 1fr 460px;
            width: 100vw; height: 100vh; padding: 80px 20px 20px 20px;
            box-sizing: border-box; gap: 20px;
        }

        #pic-container {
            width: 100%; height: 100%;
            border: 6px solid #FBD000; border-radius: 20px;
            background-size: cover; background-position: center;
            position: relative; background-color: #111;
        }

        /* 4. SIDE PANEL (STABILIZED) */
        #side-panel {
            display: flex; flex-direction: column; gap: 12px; height: 100%;
        }

        .section {
            background: rgba(0,0,0,0.85); border: 3px solid #FBD000;
            border-radius: 15px; text-align: center; padding: 10px;
            display: flex; flex-direction: column; align-items: center; justify-content: center;
            flex-shrink: 0;
        }

        #clue-box { height: 32%; }
        #bank-box { height: 22%; }
        #answer-box { height: 12%; }

        #find-msg { color: white; font-size: 20px; text-align: center; font-weight: bold; }
        #find-count { color: #FBD000; font-size: 24px; }

        #clue-img { max-height: 85%; max-width: 90%; border-radius: 10px; border: 2px solid white; display: none; }
        #lock-text { color: #888; font-size: 15px; }

        /* 5. TRANSPARENT HIDDEN LETTERS */
        .hidden-letter {
            position: absolute; width: 48px; height: 48px;
            background: rgba(255,255,255,0.08); /* Very transparent */
            border: 1px solid rgba(255,255,255,0.15); border-radius: 50%;
            display: flex; align-items: center; justify-content: center;
            color: rgba(255,255,255,0.4); /* Faded text */
            font-weight: bold; cursor: pointer; transform: translate(-50%, -50%);
            transition: opacity 0.3s;
        }

        .bank-letter {
            width: 44px; height: 44px; background: #E52521;
            color: white; border: 2px solid #000; border-radius: 8px;
            display: flex; align-items: center; justify-content: center;
            font-weight: bold; font-size: 20px; cursor: pointer; box-shadow: 2px 2px 0 #000;
        }

        /* 6. ONE-LINE SPELLING ZONE */
        #answer-zone {
            display: flex; flex-wrap: nowrap; gap: 3px; justify-content: center; width: 100%;
        }
        .slot {
            width: 33px; height: 44px; border: 2px solid #FBD000;
            border-radius: 6px; background: rgba(255,255,255,0.1);
            display: flex; align-items: center; justify-content: center;
            color: #FBD000; font-weight: bold; font-size: 20px; flex-shrink: 0;
        }

        #check-btn {
            width: 100%; padding: 18px; background: #43ad2e; color: white;
            border: none; border-radius: 12px; font-size: 22px; font-weight: bold;
            box-shadow: 0 5px 0 #2d7a1f; cursor: pointer; flex-shrink: 0;
        }

        #win-screen {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.96); z-index: 1000;
            display: none; flex-direction: column; align-items: center; justify-content: center;
        }
    </style>
</head>
<body>

<div id="bg-layer"></div>

<div id="top-right-nav">
    <button class="btn-nav" onclick="changeRound(-1)">BACK</button>
    <div id="timer">00:00</div>
    <button class="btn-nav" onclick="changeRound(1)">NEXT</button>
</div>

<div id="game-wrapper">
    <div id="pic-container"></div>

    <div id="side-panel">
        <div id="find-msg">LETTERS TO FIND: <span id="find-count">0</span></div>

        <div class="section" id="clue-box">
            <div id="lock-text">Find letters to unlock clue!</div>
            <img id="clue-img" src="">
        </div>

        <div class="section" id="bank-box">
            <div id="letter-bank" style="display:flex; flex-wrap:wrap; gap:6px; justify-content:center;"></div>
        </div>

        <button id="check-btn" onclick="checkAnswer()">CHECK ANSWER</button>

        <div class="section" id="answer-box">
            <div id="answer-zone"></div>
        </div>
    </div>
</div>

<div id="win-screen">
    <h1 style="font-size: 60px; color: #FBD000; margin: 10px;">STRIKE! 🏆</h1>
    <p id="final-time" style="color: white; font-size: 24px; margin-bottom: 20px;"></p>
    <button class="btn-nav" style="padding: 15px 50px; font-size: 24px;" onclick="changeRound(1)">NEXT ROUND</button>
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
        timerRef = setInterval(updTimer, 1000);

        document.getElementById('find-count').innerText = s.name.length;
        document.getElementById('pic-container').style.backgroundImage = `url('https://raw.githubusercontent.com/marionbunyi/sports-mario/main/hidden%20${s.img}.png')`;
        document.getElementById('clue-img').src = `https://raw.githubusercontent.com/marionbunyi/sports-mario/main/${s.img}%20clue.png`;
        document.getElementById('clue-img').style.display = 'none';
        document.getElementById('lock-text').style.display = 'block';
        document.getElementById('win-screen').style.display = 'none';
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

        const pic = document.getElementById('pic-container');
        pic.innerHTML = '';
        s.name.split('').forEach(char => {
            let l = document.createElement('div');
            l.className = 'hidden-letter';
            l.innerText = char;
            l.style.top = (Math.random() * 80 + 10) + "%";
            l.style.left = (Math.random() * 80 + 10) + "%";
            l.onclick = () => {
                l.style.display = 'none';
                found.push({char, used: false});
                let remain = s.name.length - found.length;
                document.getElementById('find-count').innerText = remain;
                if(remain === 0) {
                    document.getElementById('lock-text').style.display = 'none';
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

    function updTimer() {
        let d = Math.floor((Date.now() - startTime) / 1000);
        let m = Math.floor(d / 60).toString().padStart(2, '0');
        let s = (d % 60).toString().padStart(2, '0');
        document.getElementById('timer').innerText = `${m}:${s}`;
    }

    function checkAnswer() {
        let guess = [...document.querySelectorAll('.slot')].map(s => s.innerText).join('');
        if(guess === sports[currentIdx].name) {
            clearInterval(timerRef);
            document.getElementById('snd-win').play();
            document.getElementById('final-time').innerText = "Finished in: " + document.getElementById('timer').innerText;
            document.getElementById('win-screen').style.display = 'flex';
            confetti({ particleCount: 150, spread: 70, origin: { y: 0.6 } });
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
