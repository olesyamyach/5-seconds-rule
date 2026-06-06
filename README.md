<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>5 Second Rule — Centered Card</title>
    <style>
        * {
            user-select: none;
            -webkit-tap-highlight-color: transparent;
        }
        body {
            margin: 0;
            min-height: 100vh;
            background: linear-gradient(135deg, #2AB7CA, #1a8b9b);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            font-family: 'Segoe UI', 'Arial', sans-serif;
        }
        .game-container {
            text-align: center;
            padding: 20px;
            width: 100%;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        .timer {
            font-size: 5.2rem;
            font-weight: 800;
            color: white;
            background: rgba(0,0,0,0.2);
            padding: 0.2rem 1.4rem;
            border-radius: 80px;
            backdrop-filter: blur(5px);
            font-family: monospace;
            letter-spacing: 6px;
            margin: 0 auto 40px auto;
            display: inline-block;
            transition: font-size 0.2s;
        }
        .timer-timeup {
            font-size: 3.2rem;
        }
        .card-row {
            display: flex;
            align-items: center;
            justify-content: space-between;
            width: 100%;
            max-width: 1000px;
            margin: 0 auto;
            gap: 20px;
        }
        .restart-btn {
            background: #4CAF50;
            color: white;
        }
        .flip-card {
            width: 420px;
            height: 320px;
            perspective: 1200px;
            cursor: pointer;
            flex-shrink: 0;
        }
        .next-btn {
            background: #FFB347;
            color: #2d2f36;
        }
        .side-btn {
            padding: 18px 31px;
            font-size: 25px;
            font-weight: bold;
            border: none;
            border-radius: 60px;
            cursor: pointer;
            transition: transform 0.1s;
            font-family: inherit;
            box-shadow: 0 12px 28px rgba(0,0,0,0.3);
            white-space: nowrap;
            flex-shrink: 0;
        }
        .side-btn:active { transform: scale(0.94); }
        .hidden { display: none !important; }

        .flip-inner {
            position: relative;
            width: 100%;
            height: 100%;
            transition: transform 0.55s cubic-bezier(0.23, 1, 0.32, 1);
            transform-style: preserve-3d;
            border-radius: 30px;
            box-shadow: 0 20px 35px rgba(0,0,0,0.2);
        }
        .flip-card.flipped .flip-inner {
            transform: rotateY(180deg);
        }
        .front, .back {
            position: absolute;
            width: 100%;
            height: 100%;
            backface-visibility: hidden;
            border-radius: 30px;
            display: flex;
            align-items: center;
            justify-content: center;
            text-align: center;
            padding: 30px;
            font-size: 48px;
            font-weight: bold;
            background: white;
            word-wrap: break-word;
            line-height: 1.3;
        }
        .front {
            background: white;
            color: #1f4e5a;
            background-image: radial-gradient(circle at 10% 30%, rgba(42,183,202,0.08) 2%, transparent 2.5%);
            background-size: 30px 30px;
            transform: rotateY(0deg);
            flex-direction: column;
            gap: 12px;
        }
        .front::before {
            content: "🎤";
            font-size: 3rem;
            display: block;
            margin-bottom: 8px;
        }
        .back {
            background: #FDF8E7;
            color: #1E2F36;
            transform: rotateY(180deg);
            border: 3px solid #FFD966;
            flex-direction: column;
            justify-content: center;
            font-size: 48px;
        }

        @media (max-width: 800px) {
            .timer {
                font-size: 4rem;
            }
            .timer-timeup {
                font-size: 2.8rem;
            }
            .flip-card {
                width: 320px;
                height: 260px;
            }
            .front, .back { font-size: 32px; padding: 20px; }
            .front::before { font-size: 2.5rem; }
            .side-btn { padding: 14px 24px; font-size: 20px; }
            .card-row { gap: 15px; }
        }
        @media (max-width: 600px) {
            .timer {
                font-size: 3rem;
            }
            .timer-timeup {
                font-size: 2.2rem;
            }
            .flip-card {
                width: 260px;
                height: 220px;
            }
            .front, .back { font-size: 24px; padding: 16px; }
            .front::before { font-size: 2rem; }
            .side-btn { padding: 10px 18px; font-size: 16px; }
            .card-row { gap: 8px; }
        }
        @keyframes pulseUp {
            0% { transform: scale(1); background: rgba(0,0,0,0.2);}
            50% { transform: scale(1.05); background: #e67e22; text-shadow: 0 0 6px white;}
            100% { transform: scale(1); background: rgba(0,0,0,0.2);}
        }
        .timer-up { animation: pulseUp 0.5s ease-in-out 2; }
    </style>
</head>
<body>
<div class="game-container">
    <div class="timer" id="timer">5</div>
    <div class="card-row">
        <button class="side-btn restart-btn hidden" id="restartBtn">🔄 RESTART</button>
        <div class="flip-card" id="flipCard">
            <div class="flip-inner">
                <div class="front">Tap to start</div>
                <div class="back" id="questionBack"></div>
            </div>
        </div>
        <button class="side-btn next-btn hidden" id="nextBtn">⏩ NEXT</button>
    </div>
</div>
<script>
    (function(){
        const allQuestions = [
            "Name three things you can do in summer",
            "Name three things you can do in fall",
            "Name three things you can do in winter",
            "Name three things you can do in spring",
            "Name three things you can do in a car"
        ];
        let remaining = [];
        function shuffle(arr) {
            for (let i = arr.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [arr[i], arr[j]] = [arr[j], arr[i]];
            }
            return arr;
        }
        function resetRemaining() { remaining = shuffle([...allQuestions]); }
        function getNextQuestion() {
            if (remaining.length === 0) resetRemaining();
            return remaining.pop();
        }

        const flipCard = document.getElementById('flipCard');
        const questionBack = document.getElementById('questionBack');
        const timerDiv = document.getElementById('timer');
        const restartBtn = document.getElementById('restartBtn');
        const nextBtn = document.getElementById('nextBtn');

        let isActive = false;
        let waitingChoice = false;
        let timerTimeout = null;
        let preloadedQuestion = null;
        let isFlipped = false;
        let countdownStarted = false;

        // Звуки
        let tickAudio = null;
        let audioCtx = null;
        let soundsReady = false;

        function createWav(freq, duration, volume) {
            const sampleRate = 44100;
            const numSamples = Math.floor(duration * sampleRate);
            const buffer = new ArrayBuffer(44 + numSamples * 2);
            const view = new DataView(buffer);
            function writeString(view, offset, str) {
                for (let i = 0; i < str.length; i++) view.setUint8(offset + i, str.charCodeAt(i));
            }
            writeString(view, 0, 'RIFF');
            view.setUint32(4, 36 + numSamples * 2, true);
            writeString(view, 8, 'WAVE');
            writeString(view, 12, 'fmt ');
            view.setUint32(16, 16, true);
            view.setUint16(20, 1, true);
            view.setUint16(22, 1, true);
            view.setUint32(24, sampleRate, true);
            view.setUint32(28, sampleRate * 2, true);
            view.setUint16(32, 2, true);
            view.setUint16(34, 16, true);
            writeString(view, 36, 'data');
            view.setUint32(40, numSamples * 2, true);
            let offset = 44;
            for (let i = 0; i < numSamples; i++) {
                const t = i / sampleRate;
                let sample = Math.sin(2 * Math.PI * freq * t) * volume;
                if (t > duration * 0.8) sample *= (1 - (t - duration * 0.8) / (duration * 0.2));
                sample = Math.max(-1, Math.min(1, sample));
                const intSample = Math.floor(sample * 32767);
                view.setInt16(offset, intSample, true);
                offset += 2;
            }
            const blob = new Blob([buffer], { type: 'audio/wav' });
            return new Audio(URL.createObjectURL(blob));
        }

        function initSounds() {
            if (soundsReady) return;
            tickAudio = createWav(1500, 0.12, 0.6);
            try {
                audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            } catch(e) {}
            soundsReady = true;
        }

        function playTick() {
            if (!soundsReady) initSounds();
            if (tickAudio) {
                try { tickAudio.currentTime = 0; tickAudio.play().catch(e=>{}); } catch(e) {}
            }
        }

        function playBuzzOnce() {
            if (!soundsReady) initSounds();
            if (!audioCtx) return;
            try {
                if (audioCtx.state === 'suspended') audioCtx.resume();
                const now = audioCtx.currentTime;
                const osc = audioCtx.createOscillator();
                const gain = audioCtx.createGain();
                osc.type = "sawtooth";
                osc.connect(gain);
                gain.connect(audioCtx.destination);
                gain.gain.setValueAtTime(0.55, now);
                osc.frequency.setValueAtTime(120, now);
                osc.frequency.linearRampToValueAtTime(90, now + 0.15);
                osc.frequency.linearRampToValueAtTime(120, now + 0.30);
                osc.frequency.linearRampToValueAtTime(90, now + 0.45);
                osc.frequency.linearRampToValueAtTime(120, now + 0.60);
                gain.gain.exponentialRampToValueAtTime(0.0001, now + 0.8);
                osc.start(now);
                osc.stop(now + 0.8);
            } catch(e) { console.error("Buzz error:", e); }
        }

        function playDoubleBuzzer() {
            playBuzzOnce();
            setTimeout(() => playBuzzOnce(), 300);
        }

        function stopTimer() {
            if (timerTimeout) {
                clearTimeout(timerTimeout);
                timerTimeout = null;
            }
        }

        function setTimerDisplay(value, isUp = false) {
            timerDiv.textContent = value;
            if (value === "Time is up!") {
                timerDiv.classList.add('timer-timeup');
            } else {
                timerDiv.classList.remove('timer-timeup');
            }
            if (isUp) {
                timerDiv.classList.add('timer-up');
                setTimeout(() => timerDiv.classList.remove('timer-up'), 1000);
            }
        }

        function startCountdown(onComplete) {
            if (countdownStarted) return;
            countdownStarted = true;
            stopTimer();
            let seconds = 5;
            setTimerDisplay(seconds);
            playTick();

            function step() {
                seconds--;
                if (seconds > 0) {
                    setTimerDisplay(seconds);
                    playTick();
                    timerTimeout = setTimeout(step, 1000);
                    return;
                }
                setTimerDisplay(0);
                timerTimeout = setTimeout(() => {
                    setTimerDisplay("Time is up!", true);
                    playDoubleBuzzer();
                    countdownStarted = false;
                    if (onComplete) onComplete();
                }, 1000);
            }
            timerTimeout = setTimeout(step, 1000);
        }

        function speakQuestion(text, callback) {
            if (!window.speechSynthesis) { callback(); return; }
            try { window.speechSynthesis.cancel(); } catch(e) {}
            let called = false;
            const done = () => { if(called) return; called=true; callback(); };
            const u = new SpeechSynthesisUtterance(text);
            u.lang = 'en-US'; u.rate = 0.9;
            u.onend = done; u.onerror = done;
            window.speechSynthesis.speak(u);
            setTimeout(done, 7000);
        }

        function showButtons(show) {
            if (show) {
                restartBtn.classList.remove('hidden');
                nextBtn.classList.remove('hidden');
            } else {
                restartBtn.classList.add('hidden');
                nextBtn.classList.add('hidden');
            }
        }

        function openCard(question) {
            if (isActive) return;
            isActive = true;
            waitingChoice = false;
            countdownStarted = false;
            showButtons(false);
            questionBack.textContent = question;
            flipCard.classList.add('flipped');
            isFlipped = true;

            speakQuestion(question, () => {
                if (isActive) {
                    startCountdown(() => {
                        isActive = false;
                        waitingChoice = true;
                        showButtons(true);
                    });
                }
            });
        }

        function closeCard() {
            stopTimer();
            countdownStarted = false;
            isActive = false;
            waitingChoice = false;
            showButtons(false);
            flipCard.classList.remove('flipped');
            isFlipped = false;
            setTimerDisplay("5");
        }

        function onCardClick() {
            if (waitingChoice && isFlipped) return;
            if (isActive) return;
            initSounds();
            if (audioCtx && audioCtx.state === 'suspended') audioCtx.resume();

            if (preloadedQuestion && !isFlipped) {
                const q = preloadedQuestion;
                preloadedQuestion = null;
                openCard(q);
                return;
            }
            if (!isFlipped && !isActive && !waitingChoice) {
                const q = getNextQuestion();
                openCard(q);
            }
        }

        function onRestart() {
            if (!waitingChoice) return;
            if (isActive) return;
            stopTimer();
            countdownStarted = false;
            waitingChoice = false;
            setTimerDisplay("5");
            isActive = true;
            showButtons(false);
            startCountdown(() => {
                isActive = false;
                waitingChoice = true;
                showButtons(true);
            });
        }

        function onNext() {
            if (!waitingChoice) return;
            if (isActive) return;
            closeCard();
            preloadedQuestion = getNextQuestion();
        }

        function endGame() {
            stopTimer();
            countdownStarted = false;
            if(window.speechSynthesis) window.speechSynthesis.cancel();
            isActive = false;
            waitingChoice = false;
            preloadedQuestion = null;
            showButtons(false);
            flipCard.classList.remove('flipped');
            isFlipped = false;
            setTimerDisplay("5");
        }

        function resetGame() {
            endGame();
            resetRemaining();
        }

        flipCard.addEventListener('click', onCardClick);
        restartBtn.addEventListener('click', onRestart);
        nextBtn.addEventListener('click', onNext);
        window.addEventListener('keydown', (e) => { if(e.key === 'Escape') endGame(); });
        window.addEventListener('load', () => {
            resetRemaining();
            resetGame();
            initSounds();
        });
    })();
</script>
</body>
</html>
