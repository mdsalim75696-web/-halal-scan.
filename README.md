# -halal-scan.
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Halal Scan Pro</title>
    <script src="https://unpkg.com/html5-qrcode"></script>
    <style>
        :root { --halal: #27ae60; --haram: #e74c3c; --mushbooh: #f39c12; }
        body { font-family: 'Segoe UI', sans-serif; background: #f4f7f6; margin: 0; display: flex; flex-direction: column; align-items: center; }
        .header { background: var(--halal); color: white; width: 100%; padding: 20px 0; text-align: center; border-bottom: 4px solid #1e8449; }
        .card { background: white; width: 92%; max-width: 420px; margin-top: 15px; border-radius: 20px; padding: 20px; box-shadow: 0 8px 25px rgba(0,0,0,0.1); box-sizing: border-box; }
        textarea { width: 100%; height: 100px; border: 2px solid #ddd; border-radius: 12px; padding: 12px; font-size: 15px; margin-bottom: 10px; resize: none; box-sizing: border-box; }
        button { width: 100%; padding: 16px; border: none; border-radius: 12px; font-size: 16px; font-weight: bold; margin-bottom: 10px; cursor: pointer; }
        .btn-scan { background: #34495e; color: white; }
        .btn-check { background: var(--halal); color: white; }
        #result, #country-box { padding: 15px; border-radius: 12px; margin-top: 10px; display: none; text-align: center; font-weight: bold; }
        #country-box { background: #ebf5fb; color: #2e86c1; border: 2px solid #3498db; }
        #reader { width: 100%; border-radius: 12px; overflow: hidden; display: none; margin-bottom: 10px; background: #000; }
    </style>
</head>
<body>

<div class="header">
    <h1 style="margin:0; font-size: 22px;">HALAL SCAN PRO</h1>
    <p style="margin:5px 0 0; font-size: 12px; opacity: 0.9;">Ingredient & Origin Checker</p>
</div>

<div class="card">
    <div id="reader"></div>
    <button class="btn-scan" onclick="startScanner()">📸 SCAN BARCODE</button>
    
    <textarea id="ingInput" placeholder="Paste ingredients list here..."></textarea>
    <button class="btn-check" onclick="processAnalysis()">ANALYZE PRODUCT</button>

    <div id="result"></div>
    <div id="country-box"></div>
</div>

<script>
    const haramList = ["gelatin", "carmine", "e120", "lard", "pork", "alcohol", "wine", "beer", "e904", "shellac", "rennet", "pepsin"];
    const mushboohList = ["e471", "e472", "glycerin", "whey", "emulsifier", "lecithin", "magnesium stearate"];

    function processAnalysis() {
        const inputField = document.getElementById('ingInput');
        const text = inputField.value.toLowerCase();
        
        if(!text.trim()) {
            alert("Please paste ingredients first!");
            return;
        }

        runHalalLogic(text);
        
        // Fix #2: Clear the input box so you can paste the next product
        inputField.value = ""; 
        inputField.placeholder = "Ready for next product...";
    }

    function runHalalLogic(input) {
        const res = document.getElementById('result');
        res.style.display = "block";
        
        let foundHaram = haramList.filter(h => input.includes(h));
        let foundMush = mushboohList.filter(m => input.includes(m));

        if (foundHaram.length > 0) {
            res.style.background = "#fadbd8"; res.style.color = "#c0392b";
            res.innerHTML = "❌ HARAM<br>Forbidden: " + foundHaram.join(", ");
        } else if (foundMush.length > 0) {
            res.style.background = "#fcf3cf"; res.style.color = "#d35400";
            res.innerHTML = "🤔 MUSHBOOH<br>Check Source: " + foundMush.join(", ");
        } else {
            res.style.background = "#d5f5e3"; res.style.color = "#27ae60";
            res.innerHTML = "✅ LIKELY HALAL<br>No Haram ingredients found.";
        }
    }

    // Fix #1: Improved Scanner with Permission Request
    function startScanner() {
        const readerEl = document.getElementById('reader');
        readerEl.style.display = "block";
        
        const html5QrCode = new Html5Qrcode("reader");
        
        const qrCodeSuccessCallback = (decodedText) => {
            html5QrCode.stop().then(() => {
                readerEl.style.display = "none";
                fetchProductData(decodedText);
            });
        };

        const config = { fps: 10, qrbox: { width: 250, height: 150 } };

        // This triggers the "Allow Camera" popup on Samsung
        html5QrCode.start({ facingMode: "environment" }, config, qrCodeSuccessCallback)
        .catch(err => {
            alert("Camera Error: Please ensure you are using Chrome/Samsung Internet and have allowed camera access in phone settings.");
            readerEl.style.display = "none";
        });
    }

    function fetchProductData(barcode) {
        const countryBox = document.getElementById('country-box');
        
        fetch(`https://world.openfoodfacts.org/api/v0/product/${barcode}.json`)
            .then(r => r.json())
            .then(data => {
                if(data.status === 1) {
                    // Show Origin Country (Like No Thanks app)
                    const country = data.product.countries || "Unknown Origin";
                    countryBox.style.display = "block";
                    countryBox.innerHTML = "🌍 ORIGIN: " + country;
                    
                    // Run Halal Logic
                    const ings = data.product.ingredients_text || "";
                    runHalalLogic(ings.toLowerCase());
                } else {
                    alert("Product not found in global database.");
                }
            })
            .catch(() => alert("Network error. Check your internet!"));
    }
</script>

</body>
</html>
