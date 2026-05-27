# Sonar Radar Security System

Paste this code into **Windows PowerShell**. 

### 🚨 Prerequisites:
* Turn your device volume up to **50% or more**.
* Keep completely **still** while the radar calibrates.
* It only uses your speaker and microphone (no camera)—just like a plane's radar!
* Move your hand fast across where your microphone is located to trigger it.

### 💻 Code to Paste:

```powershell
\(Path = "\)env:TEMP\sonar_radar.html"
\$Code = @"
<!DOCTYPE html>
<html>
<head>
    <title>Instant Reset Sonar Radar</title>
    <style>
        body { margin: 0; background-color: black; color: #00ff00; font-family: monospace; display: flex; flex-direction: column; justify-content: center; align-items: center; height: 100vh; overflow: hidden; user-select: none; }
        #radar { font-size: 24px; text-align: center; display: block; }
        #stop { display: none; font-size: 150px; font-weight: bold; background-color: #cc0000; color: white; width: 100%; height: 100%; justify-content: center; align-items: center; flex-direction: column; }
        .data-box { font-size: 18px; color: #888; margin-top: 20px; background: #111; padding: 10px; border-radius: 5px; text-align: left; width: 320px; margin-left: auto; margin-right: auto; line-height: 1.6; }
    </style>
</head>
<body>
    <div id="radar">
        <h1>📢 HIGH-SPEED SONAR RADAR</h1>
        <p>👇 Click anywhere to start calibration...</p>
        <h2 id="status">Status: Offline</h2>
        <div class="data-box">
            Baseline Room Energy: <span id="baseNum">0</span><br>
            Live Room Energy: <span id="liveNum">0</span><br>
            Current Wave Shift: <span id="shiftNum">0</span> / Target: <span id="targetNum">400</span>
        </div>
    </div>
    <div id="stop">
        <div>🛑</div>
        <div>STOP</div>
    </div>
    <script>
        let audioCtx, analyser, dataArray;
        let baselineEnergy = 0;
        let triggerThreshold = 400; 
        let isArmed = false;

        document.body.onclick = async () => {
            if (audioCtx) return;
            document.getElementById('status').innerText = "Starting fast tracking engine...";
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ audio: { echoCancellation: false, noiseSuppression: false, autoGainControl: false } });
                audioCtx = new (window.AudioContext || window.webkitAudioContext)();
                analyser = audioCtx.createAnalyser();
                analyser.fftSize = 512;
                audioCtx.createMediaStreamSource(stream).connect(analyser);
                
                let osc = audioCtx.createOscillator();
                osc.frequency.value = 1000; 
                let gain = audioCtx.createGain();
                gain.gain.value = 0.05; 
                osc.connect(gain).connect(audioCtx.destination);
                osc.start();
                
                document.getElementById('status').innerText = "👂 CALIBRATING SPEED GRID... Keep still!";
                dataArray = new Uint8Array(analyser.frequencyBinCount);
                
                let initialScan = setInterval(() => {
                    analyser.getByteFrequencyData(dataArray);
                    let currentEnergy = dataArray.reduce((a, b) => a + b, 0);
                    document.getElementById('liveNum').innerText = currentEnergy;
                }, 15);

                setTimeout(() => {
                    clearInterval(initialScan);
                    
                    analyser.getByteFrequencyData(dataArray);
                    baselineEnergy = dataArray.reduce((a, b) => a + b, 0);
                    document.getElementById('baseNum').innerText = baselineEnergy;
                    document.getElementById('status').innerText = "🟢 RADAR SECURITY LOCKED. SYSTEM LIVE.";
                    isArmed = true; 
                    
                    setInterval(() => {
                        analyser.getByteFrequencyData(dataArray);
                        let currentEnergy = dataArray.reduce((a, b) => a + b, 0);
                        let soundVariance = Math.abs(currentEnergy - baselineEnergy);
                        
                        document.getElementById('liveNum').innerText = currentEnergy;
                        document.getElementById('shiftNum').innerText = soundVariance;
                        document.getElementById('targetNum').innerText = triggerThreshold;
                        
                        if (isArmed && soundVariance > triggerThreshold) {
                            document.getElementById('radar').style.display = 'none';
                            document.getElementById('stop').style.display = 'flex';
                        } else {
                            document.getElementById('stop').style.display = 'none';
                            document.getElementById('radar').style.display = 'block';
                            
                            baselineEnergy = (baselineEnergy * 0.90) + (currentEnergy * 0.10);
                            document.getElementById('baseNum').innerText = Math.round(baselineEnergy);
                        }
                    }, 15);
                }, 2500);
                
            } catch (e) { document.getElementById('status').innerText = "❌ Microphone access blocked."; }
        };
    </script>
</body>
</html>
"@
Out-File -FilePath \(Path -InputObject\)Code -Encoding utf8
& explorer.exe \$Path
```
