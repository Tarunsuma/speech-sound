# speech-sound
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Speech Therapy App</title>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background: #1e1e2f;
      color: #f0f0f0;
      margin: 0; padding: 0;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
    }
    .container {
      background: #2c2c3e;
      padding: 30px;
      border-radius: 20px;
      box-shadow: 0 10px 30px rgba(0,0,0,0.4);
      text-align: center;
      width: 450px;
      max-width: 95%;
    }
    h1 { color: #ffda79; margin-bottom: 10px; }
    h2 { font-size: 22px; color: #9cdcfe; margin: 20px 0; }
    select, button {
      margin: 8px 4px;
      padding: 10px;
      font-size: 16px;
      border-radius: 8px;
      border: none;
      outline: none;
    }
    select {
      background: #3a3a50;
      color: #fff;
    }
    button {
      background: #4a90e2;
      color: #fff;
      cursor: pointer;
      transition: 0.3s;
    }
    button:hover { background: #357ABD; transform: scale(1.05); }
    #result {
      margin-top: 20px;
      font-size: 18px;
      padding: 12px;
      border-radius: 12px;
      display: inline-block;
      min-width: 200px;
    }
    .correct { background: #2d6a4f; color: #d9fdd3; border: 2px solid #28a745; animation: pop 0.4s ease; }
    .almost  { background: #735d32; color: #ffe6a7; border: 2px solid #ffb703; animation: pop 0.4s ease; }
    .wrong { background: #6a1e1e; color: #ffd6d6; border: 2px solid #dc3545; animation: shake 0.4s ease; }
    @keyframes pop { from { transform: scale(0.9); } to { transform: scale(1); } }
    @keyframes shake {
      0%,100% { transform: translateX(0); }
      25% { transform: translateX(-5px); }
      50% { transform: translateX(5px); }
      75% { transform: translateX(-5px); }
    }
    .score-box {
      margin-top: 15px;
      font-size: 18px;
      color: #ffda79;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>Speech Therapy App</h1>

    <label for="mode">Choose Training:</label>
    <select id="mode" onchange="setTarget()">
      <option value="english">English Words</option>
      <option value="hindi">Hindi Words</option>
      <option value="twister">Tongue Twisters</option>
    </select>

    <h2 id="targetWord">Select a mode and click "New"</h2>
    <button onclick="newTarget()">🔄 New</button>
    <button onclick="speakSample()">🎧 Hear Sample</button>
    <button onclick="startListening()">🎤 Speak</button>

    <p id="result">Say what's shown above!</p>
    <div class="score-box">⭐ Score: <span id="score">0</span></div>
  </div>

  <script>
    // English words
    const englishWords = [
      "Hello", "Good morning", "Thank you", "How are you",
      "I am fine", "What is your name", "Nice to meet you",
      "Please help me", "I love learning", "Good night"
    ];

    // Hindi words with Roman support
    const hindiWords = [
      "नमस्ते (Namaste)",
      "कैसे हो (Kaise ho)",
      "धन्यवाद (Dhanyavaad)",
      "मैं ठीक हूँ (Main theek hoon)",
      "क्या कर रहे हो (Kya kar rahe ho)",
      "आपका नाम क्या है (Aapka naam kya hai)",
      "मुझे हिंदी पसंद है (Mujhe Hindi pasand hai)"
    ];

    // Tongue twisters
    const twisters = [
      "She sells seashells by the seashore",
      "Red lorry yellow lorry",
      "Peter Piper picked a peck of pickled peppers",
      "Chandu ke chacha ne chandu ki chachi ko chandi ke chamach se chatni chatai",
      "Kaccha papita pakta nahi, pakka papita kaccha nahi"
    ];

    let target = "";
    let recognitionLang = "en-US";
    let score = localStorage.getItem("therapyScore") ? parseInt(localStorage.getItem("therapyScore")) : 0;
    document.getElementById("score").innerText = score;

    function setTarget() { newTarget(); }

    function newTarget() {
      const mode = document.getElementById("mode").value;
      let list;

      if (mode === "english") {
        list = englishWords;
        recognitionLang = "en-US";
      } else if (mode === "hindi") {
        list = hindiWords;
        recognitionLang = "hi-IN";
      } else {
        list = twisters;
        recognitionLang = "en-US";
      }

      target = list[Math.floor(Math.random() * list.length)];
      document.getElementById("targetWord").innerText = target;
      document.getElementById("result").innerText = "Say what's shown above!";
      document.getElementById("result").className = "";
    }

    // Text-to-speech sample
    function speakSample() {
      if (!target) { alert("Please click New first!"); return; }
      let utter = new SpeechSynthesisUtterance(target.split("(")[0].trim());
      utter.lang = recognitionLang;
      speechSynthesis.speak(utter);
    }

    // Edit distance for fuzzy matching
    function similarity(a, b) {
      let longer = a.length > b.length ? a : b;
      let shorter = a.length > b.length ? b : a;
      let longerLength = longer.length;
      if (longerLength === 0) return 1.0;
      return (longerLength - editDistance(longer, shorter)) / parseFloat(longerLength);
    }

    function editDistance(s1, s2) {
      s1 = s1.toLowerCase();
      s2 = s2.toLowerCase();
      let costs = [];
      for (let i = 0; i <= s1.length; i++) {
        let lastValue = i;
        for (let j = 0; j <= s2.length; j++) {
          if (i === 0)
            costs[j] = j;
          else {
            if (j > 0) {
              let newValue = costs[j - 1];
              if (s1.charAt(i - 1) !== s2.charAt(j - 1))
                newValue = Math.min(Math.min(newValue, lastValue), costs[j]) + 1;
              costs[j - 1] = lastValue;
              lastValue = newValue;
            }
          }
        }
        if (i > 0) costs[s2.length] = lastValue;
      }
      return costs[s2.length];
    }

    function startListening() {
      if (!target) { alert("Please select a mode and click New!"); return; }

      const recognition = new (window.SpeechRecognition || window.webkitSpeechRecognition)();
      recognition.lang = recognitionLang;
      recognition.start();

      recognition.onresult = function(event) {
        const spoken = event.results[0][0].transcript.trim();
        const resultEl = document.getElementById("result");

        let compareTarget = target.split("(")[0].trim();
        let scoreVal = similarity(spoken, compareTarget);

        if (scoreVal > 0.8) {
          resultEl.innerHTML = "✅ Great! You said: '" + spoken + "'";
          resultEl.className = "correct";
          score++;
        } else if (scoreVal > 0.5) {
          resultEl.innerHTML = "⚠️ Almost correct: '" + spoken + "'";
          resultEl.className = "almost";
        } else {
          resultEl.innerHTML = "❌ Try Again: You said '" + spoken + "'";
          resultEl.className = "wrong";
          if (score > 0) score--;
        }

        document.getElementById("score").innerText = score;
        localStorage.setItem("therapyScore", score);
      };

      recognition.onerror = function() {
        const resultEl = document.getElementById("result");
        resultEl.innerHTML = "⚠️ Error occurred. Please try again.";
        resultEl.className = "wrong";
      };
    }
  </script>
</body>
</html>
