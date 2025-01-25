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
            transition: transform 4s ease-out;
        }

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
            color: black;
            text-align: center;
            padding: 5px;
            transform-origin: 100% 100%;
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
    </style>
</head>
<body>
    <div class="container">
        <h1>Prize Wheels</h1>

        <div class="wheel-container">
            <h2>Viewer Wheel</h2>
            <div class="wheel" id="viewerWheel"></div>
            <button onclick="spinWheel('viewer')">Spin Viewer Wheel</button>
        </div>

        <div class="wheel-container">
            <h2>Code User Wheel</h2>
            <div class="wheel" id="codeUserWheel"></div>
            <button onclick="spinWheel('codeUser')">Spin Code User Wheel</button>
        </div>

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
        // Prize data with percentages for each wheel
        const viewerWheelPrizes = [
            { prize: 0, percentage: 15 },
            { prize: 2, percentage: 14 },
            { prize: 3, percentage: 12 },
            { prize: 5, percentage: 12 },
            { prize: 6, percentage: 11 },
            { prize: 7, percentage: 10 },
            { prize: 8, percentage: 9 },
            { prize: 10, percentage: 9 },
            { prize: 12, percentage: 8 }
        ];

        const codeUserWheelPrizes = [
            { prize: 3, percentage: 15 },
            { prize: 5, percentage: 14 },
            { prize: 7, percentage: 12 },
            { prize: 8, percentage: 12 },
            { prize: 9, percentage: 11 },
            { prize: 10, percentage: 10 },
            { prize: 12, percentage: 9 },
            { prize: 15, percentage: 9 },
            { prize: 20, percentage: 8 }
        ];

        const noProfitWheelPrizes = [
            { prize: 0, percentage: 16 },
            { prize: 1, percentage: 16 },
            { prize: 2, percentage: 16 },
            { prize: 2.5, percentage: 12 },
            { prize: 3, percentage: 10 },
            { prize: 3.5, percentage: 8 },
            { prize: 4, percentage: 7 },
            { prize: 5, percentage: 6 },
            { prize: 7, percentage: 4 }
        ];

        // Create the wheel by setting up the segments
        function createWheel(wheelType) {
            let prizeData = [];
            if (wheelType === 'viewer') {
                prizeData = viewerWheelPrizes;
            } else if (wheelType === 'codeUser') {
                prizeData = codeUserWheelPrizes;
            } else if (wheelType === 'noProfit') {
                prizeData = noProfitWheelPrizes;
            }

            const wheel = document.getElementById(wheelType + 'Wheel');
            const numberOfSegments = 36;
            let segments = [];

            // Calculate the number of segments for each prize
            prizeData.forEach(prize => {
                const segmentCount = Math.floor((prize.percentage / 100) * numberOfSegments);
                for (let i = 0; i < segmentCount; i++) {
                    segments.push(prize.prize);
                }
            });

            // Fill up remaining spots if there are fewer than 36 segments
            while (segments.length < numberOfSegments) {
                segments.push(segments[segments.length - 1]);  // Repeating the last prize if necessary
            }

            // Shuffle segments for randomness
            shuffleArray(segments);

            // Create 36 segments
            for (let i = 0; i < numberOfSegments; i++) {
                const segment = document.createElement('div');
                segment.classList.add('segment');
                segment.style.transform = `rotate(${(360 / numberOfSegments) * i}deg)`;
                segment.innerHTML = `$${segments[i]}`;
                wheel.appendChild(segment);
            }
        }

        // Shuffle function to randomize segments
        function shuffleArray(array) {
            for (let i = array.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [array[i], array[j]] = [array[j], array[i]];
            }
        }

        // Spin the wheel and determine the prize
        function spinWheel(wheelType) {
            const wheel = document.getElementById(wheelType + 'Wheel');
            const totalSpins = Math.floor(Math.random() * 3) + 4;  // Random number of spins
            const rotation = Math.floor(Math.random() * 360); // Random starting point

            // Spin the wheel
            wheel.style.transition = 'transform 4s ease-out';
            wheel.style.transform = `rotate(${rotation + 360 * totalSpins}deg)`;

            setTimeout(() => {
                let prizeData = [];
                if (wheelType === 'viewer') {
                    prizeData = viewerWheelPrizes;
                } else if (wheelType === 'codeUser') {
                    prizeData = codeUserWheelPrizes;
                } else if (wheelType === 'noProfit') {
                    prizeData = noProfitWheelPrizes;
                }

                // Calculate prize index based on wheel stop position
                const prizeIndex = Math.floor(((rotation + 360 * totalSpins) % 360) / (360 / 36));
                const prizeAmount = prizeData[prizeIndex].prize;

                // Show popout message with the prize
                document.getElementById('popoutMessage').textContent = `Congratulations! You won $${prizeAmount} from the ${wheelType.charAt(0).toUpperCase() + wheelType.slice(1)} Wheel!`;
                document.getElementById('popout').style.display = "block";

                setTimeout(() => {
                    document.getElementById('popout').style.display = "none";
                }, 3000);
            }, 4000);  // Adjust delay for smooth transition
        }

        // Initialize wheels
        createWheel('viewer');
        createWheel('codeUser');
        createWheel('noProfit');
    </script>
</body>
</html>
