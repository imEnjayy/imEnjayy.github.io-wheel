<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Prize Wheels</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #002C61;
            color: white;
            text-align: center;
        }

        .container {
            margin: 50px auto;
            max-width: 800px;
        }

        h1, h2 {
            color: #fff;
        }

        .wheel-container {
            margin-bottom: 50px;
        }

        .wheel {
            position: relative;
            width: 400px;
            height: 400px;
            border-radius: 50%;
            border: 10px solid #fff;
            overflow: hidden;
            margin: 0 auto;
            transform: rotate(0deg);
            transition: transform 4s ease-out;
        }

        /* Each segment will be positioned using transform and clip-path */
        .segment {
            position: absolute;
            width: 50%;
            height: 50%;
            clip-path: polygon(100% 0%, 100% 100%, 0 100%, 0 0%);
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 14px;
            font-weight: bold;
            color: white;
            text-align: center;
            transition: transform 0.4s ease-out;
        }

        .segment:nth-child(odd) {
            background-color: #FF5722;
        }

        .segment:nth-child(even) {
            background-color: #4CAF50;
        }

        button {
            background-color: #FF8C00;
            color: white;
            border: none;
            padding: 10px 20px;
            margin-top: 20px;
            cursor: pointer;
            font-size: 16px;
        }

        button:hover {
            background-color: #e67e22;
        }

        /* Popout Message */
        .popout {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: #000;
            color: #fff;
            padding: 20px;
            border-radius: 10px;
            display: none;
        }

        /* Switch styles */
        .switch {
            position: relative;
            display: inline-block;
            width: 34px;
            height: 20px;
        }

        .switch input {
            opacity: 0;
            width: 0;
            height: 0;
        }

        .slider {
            position: absolute;
            cursor: pointer;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background-color: #ccc;
            transition: 0.4s;
            border-radius: 50px;
        }

        .slider:before {
            position: absolute;
            content: "";
            height: 12px;
            width: 12px;
            border-radius: 50px;
            left: 4px;
            bottom: 4px;
            background-color: white;
            transition: 0.4s;
        }

        input:checked + .slider {
            background-color: #2196F3;
        }

        input:checked + .slider:before {
            transform: translateX(14px);
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Prize Wheels</h1>
        
        <!-- Toggle switch for doubling prizes -->
        <label class="switch">
            <input type="checkbox" id="togglePrize">
            <span class="slider"></span>
        </label>
        <p>Double prizes: <span id="status">OFF</span></p>

        <!-- Viewer Wheel -->
        <div class="wheel-container">
            <h2>Viewer Wheel</h2>
            <div class="wheel" id="viewerWheel"></div>
            <button onclick="spinWheel('viewer')">Spin Viewer Wheel</button>
        </div>

        <!-- Code User Wheel -->
        <div class="wheel-container">
            <h2>Code User Wheel</h2>
            <div class="wheel" id="codeUserWheel"></div>
            <button onclick="spinWheel('codeUser')">Spin Code User Wheel</button>
        </div>

        <!-- No Profit Wheel -->
        <div class="wheel-container">
            <h2>No Profit Wheel</h2>
            <div class="wheel" id="noProfitWheel"></div>
            <button onclick="spinWheel('noProfit')">Spin No Profit Wheel</button>
        </div>
    </div>

    <div id="popout" class="popout">
        <p id="popoutMessage"></p>
    </div>

    <script>
        // Wheel prize data
        const viewerWheelPrizes = [0, 0, 0, 0, 2, 2, 2, 2, 3, 3, 3, 3, 5, 5, 5, 5, 6, 6, 6, 6, 7, 7, 7, 7, 8, 8, 8, 8, 10, 10, 10, 10, 12, 12, 12, 12];
        const codeUserWheelPrizes = [3, 3, 3, 3, 5, 5, 5, 5, 7, 7, 7, 7, 8, 8, 8, 8, 9, 9, 9, 9, 10, 10, 10, 10, 12, 12, 12, 12, 15, 15, 15, 15, 20, 20, 20, 20];
        const noProfitWheelPrizes = [0, 0, 0, 0, 1, 1, 1, 1, 2, 2, 2, 2, 2.5, 2.5, 2.5, 2.5, 3, 3, 3, 3, 3.5, 3.5, 3.5, 3.5, 4, 4, 4, 4, 5, 5, 5, 5, 6, 6, 6, 6, 7, 7, 7, 7];

        // Shuffle function to randomize prize order
        function shuffleArray(array) {
            for (let i = array.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [array[i], array[j]] = [array[j], array[i]]; // Swap elements
            }
        }

        // Toggle Prize Multiplier
        let prizeMultiplier = 1;

        document.getElementById('togglePrize').addEventListener('change', (event) => {
            prizeMultiplier = event.target.checked ? 2 : 1;
            document.getElementById('status').textContent = event.target.checked ? 'ON' : 'OFF';
        });

        // Function to generate wheel with segments
        function createWheel(wheelType) {
            let prizes = [];
            if (wheelType === 'viewer') {
                prizes = viewerWheelPrizes;
            } else if (wheelType === 'codeUser') {
                prizes = codeUserWheelPrizes;
            } else if (wheelType === 'noProfit') {
                prizes = noProfitWheelPrizes;
            }

            shuffleArray(prizes); // Shuffle prizes to randomize positions

            const wheel = document.getElementById(wheelType + 'Wheel');
            const numberOfSegments = 36;

            // Create 36 segments for the wheel
            for (let i = 0; i < numberOfSegments; i++) {
                let segment = document.createElement('div');
                segment.classList.add('segment');
                segment.style.transform = `rotate(${(360 / numberOfSegments) * i}deg)`;
                segment.innerHTML = `$${prizes[i]}`;
                wheel.appendChild(segment);
            }
        }

        // Spin Wheel Function
        function spinWheel(wheelType) {
            const wheel = document.getElementById(wheelType + 'Wheel');
            const totalSpins = Math.floor(Math.random() * 3) + 4;  // Random number of spins

            // Add a random rotation for the spin
            const rotation = Math.floor(Math.random() * 360);
            wheel.style.transition = 'transform 4s ease-out';
            wheel.style.transform = `rotate(${rotation + 360 * totalSpins}deg)`;

            // Get the prize after the wheel stops spinning
            setTimeout(() => {
                let prizeData = [];
                if (wheelType === 'viewer') {
                    prizeData = viewerWheelPrizes;
                } else if (wheelType === 'codeUser') {
                    prizeData = codeUserWheelPrizes;
                } else if (wheelType === 'noProfit') {
                    prizeData = noProfitWheelPrizes;
                }

                // Determine the prize based on the final position
                const prizeIndex = Math.floor((rotation + 360 * totalSpins) % 360 / (360 / prizeData.length));
                const prizeAmount = prizeData[prizeIndex] * prizeMultiplier;

                // Show popout message with the prize
                document.getElementById('popoutMessage').textContent = `Congratulations! You won $${prizeAmount} from the ${wheelType.charAt(0).toUpperCase() + wheelType.slice(1)} Wheel!`;
                document.getElementById('popout').style.display = "block";

                // Hide popout after 3 seconds
                setTimeout(() => {
                    document.getElementById('popout').style.display = "none";
                }, 3000);
            }, 4000);  // Wait for the wheel to stop spinning
        }

        // Initialize wheels
        createWheel('viewer');
        createWheel('codeUser');
        createWheel('noProfit');
    </script>
</body>
</html>
