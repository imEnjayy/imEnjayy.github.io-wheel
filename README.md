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
            width: 200px;
            height: 200px;
            border-radius: 50%;
            background: conic-gradient(red, orange, yellow, green, blue, purple);
            margin: 0 auto;
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
        const viewerWheelPrizes = [
            { amount: 0, probability: 15 },
            { amount: 2, probability: 14 },
            { amount: 3, probability: 12 },
            { amount: 5, probability: 12 },
            { amount: 6, probability: 11 },
            { amount: 7, probability: 10 },
            { amount: 8, probability: 9 },
            { amount: 10, probability: 9 },
            { amount: 12, probability: 8 }
        ];

        const codeUserWheelPrizes = [
            { amount: 3, probability: 15 },
            { amount: 5, probability: 14 },
            { amount: 7, probability: 12 },
            { amount: 8, probability: 12 },
            { amount: 9, probability: 11 },
            { amount: 10, probability: 10 },
            { amount: 12, probability: 9 },
            { amount: 15, probability: 9 },
            { amount: 20, probability: 8 }
        ];

        const noProfitWheelPrizes = [
            { amount: 0, probability: 16 },
            { amount: 1, probability: 16 },
            { amount: 2, probability: 16 },
            { amount: 2.5, probability: 12 },
            { amount: 3, probability: 10 },
            { amount: 3.5, probability: 8 },
            { amount: 4, probability: 7 },
            { amount: 5, probability: 6 },
            { amount: 7, probability: 4 }
        ];

        // Toggle Prize Multiplier
        let prizeMultiplier = 1;

        document.getElementById('togglePrize').addEventListener('change', (event) => {
            prizeMultiplier = event.target.checked ? 2 : 1;
            document.getElementById('status').textContent = event.target.checked ? 'ON' : 'OFF';
        });

        // Spin Wheel Function
        function spinWheel(wheelType) {
            let prizeData = [];
            let wheelTitle = "";
            let prizeAmount = 0;

            if (wheelType === "viewer") {
                prizeData = viewerWheelPrizes;
                wheelTitle = "Viewer Wheel";
            } else if (wheelType === "codeUser") {
                prizeData = codeUserWheelPrizes;
                wheelTitle = "Code User Wheel";
            } else if (wheelType === "noProfit") {
                prizeData = noProfitWheelPrizes;
                wheelTitle = "No Profit Wheel";
            }

            // Randomly pick a prize based on probability
            const totalProbability = prizeData.reduce((sum, prize) => sum + prize.probability, 0);
            const random = Math.random() * totalProbability;
            let cumulativeProbability = 0;

            for (let prize of prizeData) {
                cumulativeProbability += prize.probability;
                if (random < cumulativeProbability) {
                    prizeAmount = prize.amount * prizeMultiplier;
                    break;
                }
            }

            // Show popout message with the prize
            document.getElementById('popoutMessage').textContent = `Congratulations! You won $${prizeAmount} from the ${wheelTitle}!`;
            document.getElementById('popout').style.display = "block";

            // Hide popout after 3 seconds
            setTimeout(() => {
                document.getElementById('popout').style.display = "none";
            }, 3000);
        }
    </script>
</body>
</html>
