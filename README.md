<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Wheel Spinner</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #002C61;
            color: white;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }

        .container {
            text-align: center;
        }

        #wheel-container {
            margin: 20px;
        }

        .controls {
            margin-top: 20px;
        }

        button {
            background-color: #4CAF50;
            color: white;
            border: none;
            padding: 10px 20px;
            margin: 10px;
            cursor: pointer;
            font-size: 16px;
        }

        button:hover {
            background-color: #45a049;
        }

        input {
            padding: 10px;
            font-size: 16px;
            margin: 10px;
            width: 200px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Wheel Spinner</h1>
        <div id="wheel-container">
            <canvas id="wheel" width="400" height="400"></canvas>
        </div>

        <div class="controls">
            <input type="text" id="segment-input" placeholder="Enter segment name" />
            <button id="add-segment-btn">Add Segment</button>
            <button id="spin-btn">Spin!</button>
            <button id="save-btn">Save Wheel</button>
            <button id="load-btn">Load Wheel</button>
        </div>
    </div>

    <script>
        // Initialize global variables
        let segments = [];  // Array to store the segments
        let angle = 0;      // To store the current rotation of the wheel

        // Get DOM elements
        const wheelCanvas = document.getElementById('wheel');
        const ctx = wheelCanvas.getContext('2d');
        const segmentInput = document.getElementById('segment-input');
        const addSegmentBtn = document.getElementById('add-segment-btn');
        const spinBtn = document.getElementById('spin-btn');
        const saveBtn = document.getElementById('save-btn');
        const loadBtn = document.getElementById('load-btn');

        // Function to draw the wheel
        function drawWheel() {
            const radius = wheelCanvas.width / 2;
            const segmentAngle = (2 * Math.PI) / segments.length;

            ctx.clearRect(0, 0, wheelCanvas.width, wheelCanvas.height);
            ctx.translate(radius, radius);

            segments.forEach((segment, i) => {
                ctx.beginPath();
                ctx.moveTo(0, 0);
                ctx.arc(0, 0, radius, i * segmentAngle, (i + 1) * segmentAngle);
                ctx.fillStyle = getRandomColor();
                ctx.fill();

                // Text
                ctx.save();
                ctx.rotate(i * segmentAngle + segmentAngle / 2);
                ctx.fillStyle = "white";
                ctx.font = "16px Arial";
                ctx.fillText(segment, radius / 2, 0);
                ctx.restore();
            });

            ctx.resetTransform();
        }

        // Function to spin the wheel
        function spinWheel() {
            const randomRotation = Math.random() * 360 + 3600; // Random between 3600 and 7200 degrees
            const duration = 5000;  // 5 seconds for spinning
            const start = Date.now();

            const spinInterval = setInterval(() => {
                const timeElapsed = Date.now() - start;
                if (timeElapsed < duration) {
                    angle = (timeElapsed / duration) * randomRotation;
                    drawWheel();
                } else {
                    clearInterval(spinInterval);
                }
            }, 1000 / 60); // 60 FPS
        }

        // Function to add a segment
        function addSegment() {
            const segmentName = segmentInput.value.trim();
            if (segmentName) {
                segments.push(segmentName);
                segmentInput.value = '';  // Clear input
                drawWheel();
            }
        }

        // Function to save the wheel to localStorage
        function saveWheel() {
            localStorage.setItem('wheelSegments', JSON.stringify(segments));
            alert('Wheel saved!');
        }

        // Function to load the saved wheel from localStorage
        function loadWheel() {
            const savedSegments = JSON.parse(localStorage.getItem('wheelSegments'));
            if (savedSegments) {
                segments = savedSegments;
                drawWheel();
            } else {
                alert('No saved wheel found.');
            }
        }

        // Add event listeners
        addSegmentBtn.addEventListener('click', addSegment);
        spinBtn.addEventListener('click', spinWheel);
        saveBtn.addEventListener('click', saveWheel);
        loadBtn.addEventListener('click', loadWheel);

        // Initial drawing of the wheel
        drawWheel();

        // Helper function to generate random colors for segments
        function getRandomColor() {
            const letters = '0123456789ABCDEF';
            let color = '#';
            for (let i = 0; i < 6; i++) {
                color += letters[Math.floor(Math.random() * 16)];
            }
            return color;
        }
    </script>
</body>
</html>
