<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Kalkulator Biasa</title>
    <meta name="theme-color" content="#333333">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <style>
        /* Reset CSS */
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            -webkit-tap-highlight-color: transparent;
        }
        
        /* Tampilan HP */
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f0f0f0;
            height: 100vh;
            display: flex;
            flex-direction: column;
            touch-action: manipulation;
        }
        
        /* Mode Fullscreen */
        @media (display-mode: fullscreen) {
            body {
                background-color: #333;
            }
        }
        
        /* Container Kalkulator */
        .calculator {
            max-width: 100%;
            width: 100%;
            margin: auto;
            background-color: #333;
            border-radius: 20px 20px 0 0;
            box-shadow: 0 -5px 20px rgba(0,0,0,0.3);
            padding: 20px;
            position: fixed;
            bottom: 0;
        }
        
        /* Layar Display */
        .display {
            background-color: #222;
            color: white;
            padding: 20px;
            margin-bottom: 20px;
            text-align: right;
            font-size: 2.5rem;
            border-radius: 10px;
            min-height: 80px;
            overflow: hidden;
            word-break: break-all;
        }
        
        /* Tombol Kalkulator */
        .buttons {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 12px;
        }
        
        button {
            border: none;
            border-radius: 50%;
            height: 60px;
            font-size: 1.5rem;
            background-color: #505050;
            color: white;
            cursor: pointer;
            user-select: none;
            transition: all 0.2s;
        }
        
        button:active {
            transform: scale(0.95);
            opacity: 0.8;
        }
        
        /* Warna Tombol Khusus */
        .operator {
            background-color: #FF9500;
        }
        
        .function {
            background-color: #A5A5A5;
            color: black;
        }
        
        .equals {
            background-color: #FF9500;
            grid-column: span 2;
            border-radius: 30px;
        }
        
        /* Panel Rahasia */
        .secret-panel {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background-color: rgba(0,0,0,0.9);
            z-index: 100;
            padding: 20px;
            color: white;
            flex-direction: column;
            align-items: center;
            justify-content: center;
        }
        
        .secret-content {
            background-color: #333;
            padding: 20px;
            border-radius: 15px;
            max-width: 90%;
            width: 100%;
        }
        
        .secret-input {
            width: 100%;
            padding: 15px;
            margin: 10px 0;
            border-radius: 10px;
            border: none;
            font-size: 1rem;
        }
        
        /* PWA Install Prompt */
        .install-btn {
            position: fixed;
            top: 10px;
            right: 10px;
            background-color: #4CAF50;
            color: white;
            border: none;
            padding: 10px 15px;
            border-radius: 5px;
            z-index: 10;
            display: none;
        }
    </style>
</head>
<body>
    <!-- Tombol Install PWA -->
    <button id="installBtn" class="install-btn">Install App</button>
    
    <!-- Kalkulator -->
    <div class="calculator">
        <div class="display" id="display">0</div>
        <div class="buttons">
            <button class="function" onclick="clearDisplay()">AC</button>
            <button class="function" onclick="toggleSign()">+/-</button>
            <button class="function" onclick="percentage()">%</button>
            <button class="operator" onclick="appendOperator('/')">÷</button>
            
            <button onclick="appendNumber('7')">7</button>
            <button onclick="appendNumber('8')">8</button>
            <button onclick="appendNumber('9')">9</button>
            <button class="operator" onclick="appendOperator('*')">×</button>
            
            <button onclick="appendNumber('4')">4</button>
            <button onclick="appendNumber('5')">5</button>
            <button onclick="appendNumber('6')">6</button>
            <button class="operator" onclick="appendOperator('-')">−</button>
            
            <button onclick="appendNumber('1')">1</button>
            <button onclick="appendNumber('2')">2</button>
            <button onclick="appendNumber('3')">3</button>
            <button class="operator" onclick="appendOperator('+')">+</button>
            
            <button onclick="appendNumber('0')" style="grid-column: span 2; border-radius: 30px; text-align: left; padding-left: 25px;">0</button>
            <button onclick="appendDecimal()">.</button>
            <button class="operator" onclick="calculate()">=</button>
        </div>
    </div>
    
    <!-- Panel Rahasia -->
    <div class="secret-panel" id="secretPanel">
        <div class="secret-content">
            <h2 style="margin-bottom: 20px; text-align: center;">🔒 Mode Rahasia</h2>
            
            <div style="margin-bottom: 20px;">
                <label for="secretUrl">Masukkan URL Rahasia:</label>
                <input type="text" id="secretUrl" class="secret-input" placeholder="https://example.com">
            </div>
            
            <button onclick="openSecretLink()" style="background-color: #4CAF50; width: 100%; padding: 15px; border-radius: 10px; border: none; color: white; font-size: 1rem;">Buka Website</button>
            <button onclick="hideSecretPanel()" style="background-color: #f44336; width: 100%; padding: 15px; border-radius: 10px; border: none; color: white; font-size: 1rem; margin-top: 10px;">Tutup</button>
        </div>
    </div>
    
    <script>
        // Variabel Kalkulator
        let currentInput = '0';
        let previousInput = '';
        let operation = null;
        let resetScreen = false;
        
        // Variabel Rahasia
        const secretCode = '1234='; // PIN untuk membuka panel rahasia
        let inputSequence = '';
        
        // Elemen DOM
        const display = document.getElementById('display');
        const secretPanel = document.getElementById('secretPanel');
        const secretUrlInput = document.getElementById('secretUrl');
        const installBtn = document.getElementById('installBtn');
        
        // Fungsi Kalkulator
        function appendNumber(number) {
            if (currentInput === '0' || resetScreen) {
                currentInput = number;
                resetScreen = false;
            } else {
                currentInput += number;
            }
            updateDisplay();
            
            // Deteksi kode rahasia
            inputSequence += number;
            checkSecretCode();
        }
        
        function appendDecimal() {
            if (resetScreen) {
                currentInput = '0.';
                resetScreen = false;
            } else if (!currentInput.includes('.')) {
                currentInput += '.';
            }
            updateDisplay();
            
            // Deteksi kode rahasia
            inputSequence += '.';
            checkSecretCode();
        }
        
        function appendOperator(op) {
            if (operation !== null) calculate();
            previousInput = currentInput;
            operation = op;
            resetScreen = true;
            
            // Deteksi kode rahasia
            inputSequence += op;
            checkSecretCode();
        }
        
        function calculate() {
            let computation;
            const prev = parseFloat(previousInput);
            const current = parseFloat(currentInput);
            
            if (isNaN(prev) || isNaN(current)) return;
            
            switch (operation) {
                case '+':
                    computation = prev + current;
                    break;
                case '-':
                    computation = prev - current;
                    break;
                case '*':
                    computation = prev * current;
                    break;
                case '/':
                    computation = prev / current;
                    break;
                default:
                    return;
            }
            
            currentInput = computation.toString();
            operation = null;
            resetScreen = true;
            updateDisplay();
            
            // Deteksi kode rahasia (untuk tanda =)
            inputSequence += '=';
            checkSecretCode();
        }
        
        function clearDisplay() {
            currentInput = '0';
            previousInput = '';
            operation = null;
            updateDisplay();
            inputSequence = '';
        }
        
        function toggleSign() {
            currentInput = (parseFloat(currentInput) * -1).toString();
            updateDisplay();
        }
        
        function percentage() {
            currentInput = (parseFloat(currentInput) / 100).toString();
            updateDisplay();
        }
        
        function updateDisplay() {
            display.textContent = currentInput;
        }
        
        // Fungsi Rahasia
        function checkSecretCode() {
            if (inputSequence === secretCode) {
                showSecretPanel();
                inputSequence = '';
                clearDisplay();
            } else if (inputSequence.length >= secretCode.length) {
                inputSequence = '';
            }
        }
        
        function showSecretPanel() {
            // Ambil URL yang tersimpan
            const savedUrl = localStorage.getItem('secretUrl');
            if (savedUrl) {
                secretUrlInput.value = savedUrl;
            }
            
            secretPanel.style.display = 'flex';
        }
        
        function hideSecretPanel() {
            secretPanel.style.display = 'none';
        }
        
        function openSecretLink() {
            let url = secretUrlInput.value.trim();
            
            if (!url) {
                alert('Masukkan URL terlebih dahulu!');
                return;
            }
            
            // Tambahkan https:// jika tidak ada
            if (!url.startsWith('http://') && !url.startsWith('https://')) {
                url = 'https://' + url;
            }
            
            // Simpan URL
            localStorage.setItem('secretUrl', url);
            
            // Buka link
            window.open(url, '_blank');
        }
        
        // PWA Functionality
        let deferredPrompt;
        
        window.addEventListener('beforeinstallprompt', (e) => {
            e.preventDefault();
            deferredPrompt = e;
            installBtn.style.display = 'block';
        });
        
        installBtn.addEventListener('click', async () => {
            if (deferredPrompt) {
                deferredPrompt.prompt();
                const { outcome } = await deferredPrompt.userChoice;
                if (outcome === 'accepted') {
                    installBtn.style.display = 'none';
                }
                deferredPrompt = null;
            }
        });
        
        // Fullscreen pada mobile
        function requestFullscreen() {
            const elem = document.documentElement;
            if (elem.requestFullscreen) {
                elem.requestFullscreen();
            } else if (elem.webkitRequestFullscreen) { /* Safari */
                elem.webkitRequestFullscreen();
            } else if (elem.msRequestFullscreen) { /* IE11 */
                elem.msRequestFullscreen();
            }
        }
        
        // Long press untuk panel rahasia
        let pressTimer;
        const zeroBtn = document.querySelector('button[onclick="appendNumber(\'0\')"]');
        
        zeroBtn.addEventListener('touchstart', function(e) {
            pressTimer = setTimeout(function() {
                showSecretPanel();
            }, 1000);
            e.preventDefault();
        });
        
        zeroBtn.addEventListener('touchend', function(e) {
            clearTimeout(pressTimer);
            e.preventDefault();
        });
        
        // Inisialisasi
        updateDisplay();
    </script>
</body>
</html>
