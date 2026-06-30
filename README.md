<!DOCTYPE html>
<html lang="hi">
<head>
    <meta charset="UTF-8">
    <title>रीवा (दुबगवां) भू-नक्शा कस्टमाइज्ड बटवारा टूल</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; background-color: #f4f7f6; color: #333; }
        .container { max-width: 100%; margin: auto; background: white; padding: 25px; border-radius: 8px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); }
        h2 { color: #2c3e50; text-align: center; margin-bottom: 5px; }
        .upload-area { border: 2px dashed #27ae60; background: #f9fbf9; padding: 20px; text-align: center; border-radius: 6px; cursor: pointer; }
        #status { font-weight: bold; color: #2980b9; margin: 15px 0; padding: 12px; background: #e8f4fd; border-left: 5px solid #3498db; border-radius: 4px; }
        
        /* वर्कस्पेस - नक्शे को ऑरिजनल बड़े आकार में स्क्रॉल के साथ दिखाने के लिए */
        .workspace { display: flex; gap: 20px; margin-top: 20px; align-items: flex-start; }
        .map-container-scroll { flex-grow: 1; border: 2px solid #bdc3c7; background: #eef0f2; overflow: auto; max-height: 80vh; min-height: 550px; }
        .map-wrapper { position: relative; display: inline-block; background: #fff; }
        
        /* इमेज को मूल बड़े आकार में रखने के लिए */
        #map-img { display: block; max-width: none; height: auto; pointer-events: none; }
        #highlight-canvas { position: absolute; top: 0; left: 0; cursor: crosshair; }
        
        /* कंट्रोल पैनल */
        .control-panel { width: 340px; background: #f8f9fa; padding: 15px; border: 1px solid #ddd; border-radius: 6px; flex-shrink: 0; position: sticky; top: 20px; display: flex; flex-direction: column; gap: 10px; }
        .panel-title { font-weight: bold; color: #2c3e50; border-bottom: 2px solid #34495e; padding-bottom: 5px; margin-bottom: 10px; }
        .btn { padding: 11px; border: none; border-radius: 4px; cursor: pointer; font-weight: bold; font-size: 14px; text-align: center; color: white; display: block; width: 100%; box-sizing: border-box; }
        .btn-field { background-color: #2980b9; }
        .btn-split { background-color: #e67e22; }
        .btn-success { background-color: #27ae60; display: none; }
        .btn-print { background-color: #2c3e50; display: none; }
        .btn-reset { background-color: #95a5a6; }
        
        .info-card { background: #fff; border: 1px solid #e0e0e0; padding: 12px; border-radius: 4px; box-shadow: 0 2px 4px rgba(0,0,0,0.05); }
        .info-card h4 { margin: 0 0 8px 0; color: #16a085; }
        .data-val { font-size: 16px; font-weight: bold; color: #2c3e50; }
        
        @media print {
            .upload-area, .control-panel, #status, h2, p { display: none !important; }
            .map-container-scroll { border: none; max-height: none; overflow: visible; }
            .print-report-table { display: table !important; width: 100%; border-collapse: collapse; margin-top: 30px; }
            .print-report-table th, .print-report-table td { border: 1px solid #333; padding: 12px; text-align: left; }
        }
        .print-report-table { display: none; }
    </style>
</head>
<body>

<div class="container">
    <h2>🚜 भू-नक्शा कस्टमाइज्ड Z-बटवारा टूल (मापांक 1:4000)</h2>
    <p style="text-align: center; color: #666; margin: 0 0 15px 0;">ग्राम: दुबगवां, जिला: रीवा (म.प्र.)। इमेज (JPG/PNG) आधारित ऑफलाइन सुरक्षित मोड।</p>
    
    <!-- इमेज अपलोड करने का बटन (फिक्स आईडी: image-input) -->
    <div class="upload-area" onclick="document.getElementById('image-input').click()">
        <p style="font-size: 15px; margin: 0; color: #27ae60;">📁 <b>यहाँ क्लिक करके नक्शे की बनाई गई IMAGE (फोटो) फाइल लोड करें</b></p>
        <input type="file" id="image-input" accept="image/*" style="display: none;" />
    </div>

    <div id="status">सिस्टम ऑफलाइन तैयार है। कृपया अपने नक्शे की फोटो फाइल (JPG या PNG) चुनें।</div>

    <div class="workspace">
        <div class="map-container-scroll">
            <div class="map-wrapper" id="mapWrapper">
                <!-- मूल बड़े आकार का नक्शा दिखाने के लिए सीधा इमेज टैग -->
                <img id="map-img" alt="" style="display:none;" />
                <canvas id="highlight-canvas"></canvas>
            </div>
        </div>

        <div class="control-panel">
            <div class="panel-title">🛠️ बटवारा कंट्रोल पैनल</div>
            <button class="btn btn-field" id="fieldBtn" onclick="startFieldMode()">📐 1. खेत की बाउंड्री लॉक करें</button>
            <button class="btn btn-split" id="modeBtn" onclick="startSplitMode()" style="display:none;">✂️ 2. नया मेढ़ (Z-बटवारा) शुरू करें</button>
            <button class="btn btn-success" id="finalBtn" onclick="calculatePartition()">✅ 3. मेढ़ निर्माण फाइनल करें</button>
            <button class="btn btn-print" id="printBtn" onclick="window.print()">🖨️ बंटवारा रिपोर्ट प्रिंट करें</button>
            <button class="btn btn-reset" onclick="resetAll()">🔄 पूरा पेज रीसेट करें</button>
            
            <div class="info-card">
                <h4>📏 प्रस्तावित मेढ़ की कुल लंबाई:</h4>
                <div id="live-line-length" class="data-val">0.00 मीटर</div>
            </div>
            <div class="info-card" style="border-left: 4px solid #e67e22;">
                <h4>🌾 बटवारे का लाइव परिणाम:</h4>
                <div id="split-results">
                    <p style="color: #7f8c8d; font-size: 13px; margin: 0;">बाउंड्री सेट करने और मेढ़ खींचने के बाद यहाँ स्वतंत्र हिस्सों का एरिया दिखेगा।</p>
                </div>
            </div>
        </div>
    </div>

    <table class="print-report-table" id="printTable">
        <thead>
            <tr><th colspan="2" style="text-align: center; font-size: 18px; padding: 15px;">🚜 डिजिटल भू-अभिलेख बटवारा एवं नाप रिपोर्ट</th></tr>
            <tr><th>विवरण (Description)</th><th>माप मूल्य (Calculated Value)</th></tr>
        </thead>
        <tbody id="printTableBody"></tbody>
    </table>
</div>

<script>
    const PERIMETER_MULTIPLIER = 0.711;    

    let currentMode = "NONE"; 
    let fieldPoints = []; 
    let cutPoints = []; 
    let isMapActive = false;

    // जावास्क्रिप्ट बाइंडिंग को पूरी तरह आईडी (image-input) से फिक्स किया गया
    const fileInput = document.getElementById('image-input');
    const mapImg = document.getElementById('map-img');
    const canvas = document.getElementById('highlight-canvas');
    const ctx = canvas.getContext('2d');

    fileInput.addEventListener('change', handleImage);

    function handleImage(event) {
        const file = event.target.files[0]; // पहली सिलेक्टेड फाइल को ठीक से उठाना
        if (!file) return;

        document.getElementById('status').innerText = "नक्शे की फोटो लोड की जा रही है...";
        
        const fileReader = new FileReader();
        fileReader.onload = function(e) {
            mapImg.src = e.target.result;
            mapImg.style.display = "block";
            
            mapImg.onload = function() {
                // कैनवास का आकार हुबहू मूल बड़ी फोटो के बराबर लॉक करना
                canvas.width = mapImg.clientWidth || mapImg.naturalWidth;
                canvas.height = mapImg.clientHeight || mapImg.naturalHeight;
                
                document.getElementById('status').innerText = `✅ फुल साइज नक्शा पूरी तरह लोड हो गया! साइज: ${canvas.width}x${canvas.height}। बटन '1' दबाएं।`;
                isMapActive = true;
                resetAll();
            };
        };
        fileReader.readAsDataURL(file);
    }

    function startFieldMode() {
        if(!isMapActive) { alert("कृपया पहले नक्शे की फोटो फाइल चुनें!"); return; }
        currentMode = "FIELD"; fieldPoints = [];
        document.getElementById('status').innerText = "📍 मोड सक्रिय: नक्शे पर खेत (जैसे 294S) के कोनों पर क्लिक करके बाउंड्री घेरें।";
    }

    function startSplitMode() {
        if(fieldPoints.length < 3) { alert("कृपया पहले खेत की पूरी बाउंड्री लॉक करें!"); return; }
        currentMode = "SPLIT"; cutPoints = [];
        document.getElementById('status').innerText = "✂️ मोड सक्रिय: अब लॉक किए गए खेत के अंदर क्लिक करके टेढ़ी-मेढ़ी (Z) बटवारा लाइन बनाएं।";
        document.getElementById('finalBtn').style.display = "block";
    }

    function resetAll() {
        currentMode = "NONE"; fieldPoints = []; cutPoints = [];
        document.getElementById('live-line-length').innerText = "0.00 मीटर";
        document.getElementById('split-results').innerHTML = `<p style="color: #7f8c8d; font-size: 13px; margin: 0;">परिणाम यहाँ दिखेगा.</p>`;
        document.getElementById('finalBtn').style.display = "none";
        document.getElementById('printBtn').style.display = "none";
        document.getElementById('modeBtn').style.display = "none";
        ctx.clearRect(0, 0, canvas.width, canvas.height);
    }

    canvas.addEventListener('click', function(e) {
        if (currentMode === "NONE") return;
        const rect = canvas.getBoundingClientRect();
        const mouseCoords = { x: e.clientX - rect.left, y: e.clientY - rect.top };

        if (currentMode === "FIELD") {
            fieldPoints.push(mouseCoords);
            document.getElementById('status').innerText = `📍 बाउंड्री पॉइंट #${fieldPoints.length} सेट। खेत घेरना पूरा होने पर बटन '2' दबाएं।`;
            document.getElementById('modeBtn').style.display = "block";
            redrawAll();
        } else if (currentMode === "SPLIT") {
            cutPoints.push(mouseCoords);
            document.getElementById('status').innerText = `✂️ बटवारा पॉइंट #${cutPoints.length} सेट। मेढ़ पूरी होने पर बटन '3' दबाएं।`;
            redrawAll();
        }
    });

    canvas.addEventListener('mousemove', function(e) {
        if (currentMode !== "SPLIT" || cutPoints.length === 0) return;
        const rect = canvas.getBoundingClientRect();
        const mouseCoords = { x: e.clientX - rect.left, y: e.clientY - rect.top };
        redrawAll();

        ctx.strokeStyle = "#e67e22"; ctx.lineWidth = 3;
