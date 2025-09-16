<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>حاسبة معدل التخصيص - جامعة الملك سعود</title>
    
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;500;700&display=swap" rel="stylesheet">
    
    <style>
        :root {
            --primary-color: #007bff;
            --primary-hover: #0056b3;
            --danger-color: #dc3545;
            --danger-hover: #c82333;
            --success-color: #28a745;
            --bg-color: #f0f2f5;
            --card-bg: #ffffff;
            --text-color: #2c3e50;
            --label-color: #8899a6;
            --border-color: #dee2e6;
            --input-bg: #f8f9fa;
            --shadow-color: rgba(0, 0, 0, 0.08);
            --mode-icon: '🌙';
        }

        body.dark-mode {
            --bg-color: #10171e;
            --card-bg: #1a2836;
            --text-color: #e0e0e0;
            --label-color: #8899a6;
            --border-color: #38444d;
            --input-bg: #213243;
            --shadow-color: rgba(0, 0, 0, 0.2);
            --mode-icon: '☀️';
        }

        body {
            font-family: 'Tajawal', sans-serif;
            background-color: var(--bg-color);
            color: var(--text-color);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            padding: 20px;
            box-sizing: border-box;
            transition: background-color 0.3s, color 0.3s;
        }
        .container-wrapper {
            display: flex;
            flex-direction: column;
            align-items: center;
            width: 100%;
        }
        .container {
            position: relative;
            background-color: var(--card-bg);
            padding: 30px;
            border-radius: 20px;
            box-shadow: 0 10px 30px var(--shadow-color);
            text-align: center;
            width: 100%;
            max-width: 400px; /* Default max-width */
            border-top: 5px solid var(--primary-color);
            transition: background-color 0.3s, max-width 0.3s ease-in-out; /* Smooth resizing */
            transform-origin: top center;
        }
        #theme-toggle {
            position: absolute;
            top: 20px;
            left: 20px;
            background: var(--input-bg);
            border: 1px solid var(--border-color);
            border-radius: 50%;
            width: 40px;
            height: 40px;
            cursor: pointer;
            font-size: 20px;
            display: flex;
            justify-content: center;
            align-items: center;
            transition: background-color 0.3s, border-color 0.3s;
            color: var(--text-color);
        }
        #theme-toggle::after { content: var(--mode-icon); }
        
        .header { margin-bottom: 20px; }
        .header h2 {
            font-size: 18px;
            font-weight: 500;
            color: var(--label-color);
            margin: 0 0 10px 0;
        }
        .header h1 {
            font-size: 22px;
            font-weight: 700;
            color: var(--text-color);
            margin: 0;
            line-height: 1.4;
        }
        hr {
            border: none;
            height: 1px;
            background-color: var(--border-color);
            margin: 25px auto;
        }
        
        .form-grid {
            display: grid;
            grid-template-columns: 2fr 1fr;
            gap: 20px 15px;
            margin-bottom: 25px;
        }
        .form-group {
            text-align: right;
        }
        .input-wrapper {
            position: relative;
        }
        label {
            display: block;
            margin-bottom: 8px;
            color: var(--label-color);
            font-weight: 500;
            font-size: 14px;
        }
        input {
            width: 100%;
            padding: 12px 15px;
            border: 1px solid var(--border-color);
            background-color: var(--input-bg);
            color: var(--text-color);
            border-radius: 10px;
            box-sizing: border-box;
            font-size: 16px;
            font-family: 'Tajawal', sans-serif;
            text-align: center;
            transition: border-color 0.3s, box-shadow 0.3s, padding 0.1s ease;
        }
        .input-wrapper input { padding-right: 35px; } /* Adjust padding for clear button */
        input:focus {
            border-color: var(--primary-color);
            outline: none;
            box-shadow: 0 0 0 4px rgba(0, 123, 255, 0.15);
        }
        .clear-btn {
            position: absolute;
            right: 5px;
            top: 50%;
            transform: translateY(-50%);
            background: transparent;
            border: none;
            color: var(--label-color);
            cursor: pointer;
            font-size: 22px;
            width: 30px;
            height: 100%;
            opacity: 0.7;
            transition: opacity 0.2s;
        }
        .clear-btn:hover {
            opacity: 1;
        }
        
        .weights-total {
            margin-bottom: 25px;
            font-weight: bold;
            color: var(--success-color);
        }
        .button-group {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
        }
        button {
            padding: 14px;
            color: white;
            border: none;
            border-radius: 10px;
            cursor: pointer;
            font-size: 16px;
            font-family: 'Tajawal', sans-serif;
            font-weight: 700;
            transition: all 0.3s ease;
        }
        button:active { transform: scale(0.97); }
        #calculateBtn { background-color: var(--primary-color); }
        #calculateBtn:hover { background-color: var(--primary-hover); }
        #clearBtn { background-color: var(--danger-color); }
        #clearBtn:hover { background-color: var(--danger-hover); }
        #result {
            margin-top: 25px;
            font-size: 24px;
            font-weight: 700;
            color: var(--success-color);
            min-height: 35px;
        }

        .footer {
            margin-top: 25px;
            text-align: center;
            width: 100%;
            max-width: 400px; /* Match container width for consistent layout */
            transition: max-width 0.3s ease-in-out;
        }
        .footer a {
            color: var(--text-color);
            text-decoration: none;
            font-weight: bold;
            display: inline-flex;
            align-items: center;
            gap: 8px;
            transition: color 0.3s ease;
        }
        .footer a:hover { color: var(--primary-color); }
        .footer svg { width: 20px; height: 20px; fill: currentColor; }
        .prayer-text {
            margin-top: 10px;
            font-size: 14px;
            color: var(--label-color);
        }

        /* Slider for resizing */
        .resizer-slider {
            margin-top: 30px;
            width: 100%;
            max-width: 500px; /* Max width for the slider itself */
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 10px;
        }
        .resizer-slider label {
            font-size: 14px;
            color: var(--label-color);
            font-weight: 500;
            margin-bottom: 5px;
        }
        .resizer-slider input[type="range"] {
            -webkit-appearance: none;
            appearance: none;
            width: 100%;
            height: 8px;
            background: var(--border-color);
            outline: none;
            border-radius: 5px;
            transition: background 0.2s;
            margin: 0;
            padding: 0;
        }
        .resizer-slider input[type="range"]::-webkit-slider-thumb {
            -webkit-appearance: none;
            appearance: none;
            width: 20px;
            height: 20px;
            background: var(--primary-color);
            border-radius: 50%;
            cursor: grab;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
        }
        .resizer-slider input[type="range"]::-moz-range-thumb {
            width: 20px;
            height: 20px;
            background: var(--primary-color);
            border-radius: 50%;
            cursor: grab;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
        }
    </style>
</head>
<body>

<div class="container-wrapper">
    <div class="container" id="calculatorContainer">
        <button id="theme-toggle" title="تبديل الوضع"></button>
        <div class="header">
            <h2>جامعة الملك سعود</h2>
            <h1>حاسبة معدل التخصيص للمسارين العلمي والصحي</h1>
        </div>
        <hr>
        <form id="calculatorForm" onsubmit="return false;">
            <div class="form-grid">
                <div class="form-group">
                    <label for="gpa">المعدل التراكمي (من 5)</label>
                    <div class="input-wrapper">
                        <input type="text" inputmode="decimal" id="gpa">
                        <button type="button" class="clear-btn" onclick="clearInput('gpa')">&times;</button>
                    </div>
                </div>
                <div class="form-group">
                    <label for="gpa-weight">الوزن %</label>
                    <input type="text" inputmode="numeric" id="gpa-weight" value="50" class="weight-input">
                </div>
                <div class="form-group">
                    <label for="qudrat">درجة القدرات</label>
                    <div class="input-wrapper">
                        <input type="text" inputmode="numeric" id="qudrat">
                        <button type="button" class="clear-btn" onclick="clearInput('qudrat')">&times;</button>
                    </div>
                </div>
                <div class="form-group">
                    <label for="qudrat-weight">الوزن %</label>
                    <input type="text" inputmode="numeric" id="qudrat-weight" value="25" class="weight-input">
                </div>
                <div class="form-group">
                    <label for="tahsili">درجة التحصيلي</label>
                    <div class="input-wrapper">
                        <input type="text" inputmode="numeric" id="tahsili">
                        <button type="button" class="clear-btn" onclick="clearInput('tahsili')">&times;</button>
                    </div>
                </div>
                <div class="form-group">
                    <label for="tahsili-weight">الوزن %</label>
                    <input type="text" inputmode="numeric" id="tahsili-weight" value="25" class="weight-input">
                </div>
            </div>
            
            <div class="weights-total" id="weights-total-text">
                مجموع الأوزان: 100%
            </div>

            <div class="button-group">
                <button id="clearBtn" type="button" onclick="clearData()">مسح الكل</button>
                <button id="calculateBtn" type="button" onclick="calculate()">احسب</button>
            </div>
        </form>
        <div id="result"></div>
    </div>

    <div class="footer" id="footer">
        <a href="https://x.com/K1alotaibi" target="_blank">
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16"><path d="M9.237 6.522L14.472.244h-1.25L8.73 5.688 4.734.244H.244l5.58 7.84L.244 15.756h1.25l4.74-5.83 4.28 5.83h4.49L9.237 6.522zm-1.13 1.623l-.74-1.04-4.23-5.9H5.03l3.22 4.5.74 1.04 4.5 6.26h-1.88l-3.5-4.88z"></path></svg>
            <span>@K1alotaibi</span>
        </a>
        <p class="prayer-text">دعواتكم لي ولوالدي وجميع المسلمين</p>
    </div>

    <div class="resizer-slider">
        <label for="sizeSlider">تعديل حجم الحاسبة:</label>
        <input type="range" id="sizeSlider" min="280" max="600" value="400">
    </div>

</div>

<script>
    // --- GENERAL ELEMENTS ---
    const calculatorContainer = document.getElementById('calculatorContainer');
    const footer = document.getElementById('footer');
    const resultDiv = document.getElementById('result');
    const dangerColor = getComputedStyle(document.documentElement).getPropertyValue('--danger-color').trim();
    const successColor = getComputedStyle(document.documentElement).getPropertyValue('--success-color').trim();

    // --- DARK MODE LOGIC ---
    const themeToggle = document.getElementById('theme-toggle');
    const body = document.body;
    themeToggle.addEventListener('click', () => {
        body.classList.toggle('dark-mode');
        localStorage.setItem('theme', body.classList.contains('dark-mode') ? 'dark' : 'light');
    });

    // --- WEIGHTS VALIDATION LOGIC ---
    const weightInputs = document.querySelectorAll('.weight-input');
    const weightsTotalText = document.getElementById('weights-total-text');
    function updateWeightsTotal() {
        let total = 0;
        weightInputs.forEach(input => {
            total += convertToEnglishNumerals(input.value) || 0;
        });
        weightsTotalText.textContent = `مجموع الأوزان: ${total}%`;
        if (Math.round(total) !== 100) {
            weightsTotalText.style.color = dangerColor;
        } else {
            weightsTotalText.style.color = successColor;
        }
    }
    weightInputs.forEach(input => input.addEventListener('input', updateWeightsTotal));

    // --- PAGE LOAD INITIALIZATION ---
    document.addEventListener('DOMContentLoaded', () => {
        // Default to dark mode unless user has explicitly chosen light mode
        if (localStorage.getItem('theme') !== 'light') {
            body.classList.add('dark-mode');
        }
        updateWeightsTotal();
    });

    // --- CALCULATOR LOGIC ---
    function convertToEnglishNumerals(str) {
        if (str === null || str === undefined || String(str).trim() === '') return NaN;
        const arabicEastern = /[\u0660-\u0669]/g;
        const arabicIndic = /[\u06F0-\u06F9]/g;
        let newStr = String(str)
            .replace(arabicEastern, d => d.charCodeAt(0) - 0x0660)
            .replace(arabicIndic, d => d.charCodeAt(0) - 0x06F0)
            .replace(/٫/g, '.');
        return parseFloat(newStr);
    }

    function calculate() {
        const gpa = convertToEnglishNumerals(document.getElementById('gpa').value);
        const qudrat = convertToEnglishNumerals(document.getElementById('qudrat').value);
        const tahsili = convertToEnglishNumerals(document.getElementById('tahsili').value);

        const gpaWeight = convertToEnglishNumerals(document.getElementById('gpa-weight').value);
        const qudratWeight = convertToEnglishNumerals(document.getElementById('qudrat-weight').value);
        const tahsiliWeight = convertToEnglishNumerals(document.getElementById('tahsili-weight').value);

        if (isNaN(gpa) || isNaN(qudrat) || isNaN(tahsili) || isNaN(gpaWeight) || isNaN(qudratWeight) || isNaN(tahsiliWeight)) {
            resultDiv.innerText = 'الرجاء إدخال جميع القيم.';
            resultDiv.style.color = dangerColor;
            return;
        }
        
        const totalWeight = gpaWeight + qudratWeight + tahsiliWeight;
        if (Math.round(totalWeight) !== 100) {
            resultDiv.innerText = 'مجموع الأوزان يجب أن يكون 100%.';
            resultDiv.style.color = dangerColor;
            return;
        }

        const gpaPercentage = (gpa / 5) * 100;
        const weightedScore = (gpaPercentage * (gpaWeight / 100)) + (qudrat * (qudratWeight / 100)) + (tahsili * (tahsiliWeight / 100));

        const arabicResult = new Intl.NumberFormat('ar-SA', { minimumFractionDigits: 2, maximumFractionDigits: 2 }).format(weightedScore);
        resultDiv.innerText = `النسبة النهائية: ${arabicResult}٪`;
        resultDiv.style.color = successColor;
    }
    
    // Function to clear a single input field
    function clearInput(inputId) {
        document.getElementById(inputId).value = '';
        document.getElementById(inputId).focus();
    }

    // Function to clear all form fields
    function clearData() {
        document.getElementById('calculatorForm').reset();
        resultDiv.innerText = '';
        updateWeightsTotal();
    }

    // --- RESIZER SLIDER LOGIC ---
    const sizeSlider = document.getElementById('sizeSlider');
    sizeSlider.addEventListener('input', (event) => {
        const newWidth = event.target.value; // Get value from slider (e.g., 300px, 500px)
        calculatorContainer.style.maxWidth = `${newWidth}px`;
        footer.style.maxWidth = `${newWidth}px`; // Keep footer width consistent
    });
</script>

</body>
</html>
