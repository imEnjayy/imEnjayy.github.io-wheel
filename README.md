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
            flex-direction: column;
        }

        .container {
            text-align: center;
            margin-bottom: 20px;
        }

        #wheel-container {
            margin: 20px;
            display: flex;
            justify-content: center;
        }

        .wheel-and-description {
            display: flex;
            justify-content: center;
            align-items: center;
        }

        .wheel-description {
            margin-left: 20px;
            background-color: #444;
            padding: 15px;
            border-radius: 10px;
            color: white;
            width: 250px;
            max-height: 400px;
            overflow-y: scroll;
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

        #saved-wheels {
            margin-top: 20px;
            list-style: none;
            padding: 0;
        }

        .saved-wheel {
            cursor: pointer;
            margin-bottom: 10px;
            padding: 10px;
            background-color: #444;
            border-radius: 5px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .saved-wheel:hover {
            background-color: #555;
        }

        .popup {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background: rgba(0, 0, 0, 0.5);
            color: white;
            font-size: 20px;
            justify-content: center;
            align-items: center;
            z-index: 999;
        }

        .popup-content {
            background-color: #222;
            padding: 20px;
            border-radius: 10px;
            text-align: center;
            border: 5px solid #d50f25;
        }

        .close-btn {
            margin-top: 20px;
            padding: 10px 20px;
            background-color: #d50f25;
            border: none;
            color: white;
            font-size: 16px;
            cursor: pointer;
        }

        .close-btn:hover {
            background-color: #c00e1d;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1 id="wheel-title">Wheel Spinner</h1>
        <div id="wheel-container" class="wheel-and-description">
            <canvas id="wheel" width="400" height="400"></canvas>
            <div id="wheel-description" class="wheel-description"></div>
        </div>

        <div class="controls">
            <input type="text" id="segment-input" placeholder="Enter segment name" />
            <input type="number" id="percentage-input" placeholder="Enter percentage" min="1" max="100" />
            <button id="add-segment-btn">Add Segment</button>

            <!-- Spin Time Slider -->
            <label for="spin-duration-slider">Spin Duration (seconds): </label>
            <input type="range" id="spin-duration-slider" min="1" max="10" value="5">
            <span id="spin-duration-label">5</span> seconds

            <button id="spin-btn">Spin!</button>
            <button id="save-btn">Save Wheel</button>
            <button id="new-wheel-btn">Create New Wheel</button>
            <button id="shuffle-btn">Shuffle Segments</button>
        </div>

        <h3>Saved Wheels</h3>
        <ul id="saved-wheels"></ul>
    </div>

    <div class="popup" id="popup">
        <div class="popup-content" id="popup-content">
            <div id="popup-message"></div>
            <button class="close-btn" id="close-popup-btn">Close</button>
        </div>
    </div>

    <script>
        // Initialize global variables
        let segments = [];
        let totalPercentage = 0;
        let angle = 0;
        let spinning = false;
        let spinAngle = 0;
        let spinTimeout = null;

        // Get DOM elements
        const wheelCanvas = document.getElementById('wheel');
        const ctx = wheelCanvas.getContext('2d');
        const segmentInput = document.getElementById('segment-input');
        const percentageInput = document.getElementById('percentage-input');
        const addSegmentBtn = document.getElementById('add-segment-btn');
        const spinBtn = document.getElementById('spin-btn');
        const saveBtn = document.getElementById('save-btn');
        const newWheelBtn = document.getElementById('new-wheel-btn');
        const shuffleBtn = document.getElementById('shuffle-btn');
        const savedWheelsList = document.getElementById('saved-wheels');
        const popup = document.getElementById('popup');
        const popupMessage = document.getElementById('popup-message');
        const closePopupBtn = document.getElementById('close-popup-btn');
        const wheelDescription = document.getElementById('wheel-description');
        const spinDurationSlider = document.getElementById('spin-duration-slider');
        const spinDurationLabel = document.getElementById('spin-duration-label');
        const wheelTitle = document.getElementById('wheel-title');

        // Function to draw the wheel
        function drawWheel() {
            const radius = wheelCanvas.width / 2;
            const segmentAngle = (2 * Math.PI) / 100;
            let offsetAngle = -Math.PI / 2 + spinAngle;

            ctx.clearRect(0, 0, wheelCanvas.width, wheelCanvas.height);
            ctx.translate(radius, radius);

            segments.forEach((segment, i) => {
                const anglePercentage = segment.percentage / 100 * 2 * Math.PI;

                ctx.beginPath();
                ctx.moveTo(0, 0);
                ctx.arc(0, 0, radius, offsetAngle, offsetAngle + anglePercentage);
                
                const colors = ['#3369e8', '#d50f25', '#eeb211', '#009925', '#5e02e9', '#3c70ef', '#30d800', '#e7e200', '#fd8b00', '#f20800'];
                ctx.fillStyle = colors[i % colors.length];
                ctx.fill();
                
                ctx.lineWidth = 2;
                ctx.strokeStyle = 'black';
                ctx.stroke();

                ctx.save();
                ctx.rotate(offsetAngle + anglePercentage / 2);
                ctx.fillStyle = "white";
                ctx.font = "24px Arial";
                ctx.lineWidth = 4;
                ctx.strokeStyle = 'black';
                ctx.strokeText(segment.name, radius / 2, 0);
                ctx.fillText(segment.name, radius / 2, 0);
                ctx.restore();

                offsetAngle += anglePercentage;
            });

            drawPointer();
            ctx.resetTransform();
        }

        // Function to draw the triangle pointer
        function drawPointer() {
            const radius = wheelCanvas.width / 2;
            const pointerHeight = 20; 
            const pointerX = radius;
            const pointerY = -radius - pointerHeight;

            ctx.beginPath();
            ctx.moveTo(pointerX - pointerHeight / 2, pointerY);
            ctx.lineTo(pointerX + pointerHeight / 2, pointerY);
            ctx.lineTo(pointerX, pointerY - pointerHeight);
            ctx.closePath();
            ctx.fillStyle = '#d50f25'; 
            ctx.fill();
        }

        // Function to update the prize description area
        function updateWheelDescription() {
            wheelDescription.innerHTML = '';
            segments.forEach(segment => {
                const prizeInfo = document.createElement('p');
                prizeInfo.textContent = `${segment.name} - ${segment.percentage}%`;
                wheelDescription.appendChild(prizeInfo);
            });
        }

        // Function to spin the wheel
        function spinWheel() {
            if (spinning) return;

            spinning = true;
            spinAngle = 0;

            const spinDuration = spinDurationSlider.value * 1000;
            const spinTarget = Math.random() * 7200 + 3600;

            let startTime = null;

            function animate(time) {
                if (!startTime) startTime = time;
                const elapsedTime = time - startTime;

                const progress = Math.min(elapsedTime / spinDuration, 1);
                spinAngle = (progress * spinTarget);

                drawWheel();

                if (elapsedTime < spinDuration) {
                    requestAnimationFrame(animate);
                } else {
                    spinning = false;
                    const winner = getWinningSegment();
                    showPopup(winner);
                }
            }

            requestAnimationFrame(animate);
        }

        // Function to get the winning segment
        function getWinningSegment() {
            const totalRotation = spinAngle % 360;
            const anglePerSegment = 360 / segments.length;
            const winningIndex = Math.floor(totalRotation / anglePerSegment);
            return segments[winningIndex];
        }

        // Function to show the pop-up with a "Close" button
        function showPopup(winner) {
            popupMessage.textContent = `Congratulations! You have won ${winner.name}`;
            popup.style.display = 'flex';
        }

        closePopupBtn.addEventListener('click', () => {
            popup.style.display = 'none';
        });

        // Function to add a segment
        function addSegment() {
            const segmentName = segmentInput.value.trim();
            const segmentPercentage = parseFloat(percentageInput.value);

            if (segmentName && segmentPercentage && totalPercentage + segmentPercentage <= 100) {
                segments.push({ name: segmentName, percentage: segmentPercentage });
                totalPercentage += segmentPercentage;

                segmentInput.value = '';
                percentageInput.value = '';
                drawWheel();
                updateWheelDescription();
            } else {
                alert('Please enter a valid segment name and percentage (total percentage must equal 100)');
            }
        }

        // Function to save the wheel
        function saveWheel() {
            const wheelName = prompt('Enter a name for this wheel:');
            if (wheelName) {
                const savedWheels = JSON.parse(localStorage.getItem('savedWheels')) || [];
                savedWheels.push({ name: wheelName, segments: segments });
                localStorage.setItem('savedWheels', JSON.stringify(savedWheels));
                updateSavedWheelsList();
            }
        }

        // Function to load a saved wheel
        function loadWheel(loadedSegments, wheelName) {
            segments = loadedSegments;
            drawWheel();
            updateWheelDescription();
            wheelTitle.textContent = wheelName;
        }

        // Function to delete a saved wheel
        function deleteSavedWheel(wheelIndex) {
            const savedWheels = JSON.parse(localStorage.getItem('savedWheels')) || [];
            savedWheels.splice(wheelIndex, 1);
            localStorage.setItem('savedWheels', JSON.stringify(savedWheels));
            updateSavedWheelsList();
        }

        // Function to update saved wheels list
        function updateSavedWheelsList() {
            savedWheelsList.innerHTML = '';
            const savedWheels = JSON.parse(localStorage.getItem('savedWheels')) || [];
            savedWheels.forEach((wheel, index) => {
                const listItem = document.createElement('li');
                listItem.classList.add('saved-wheel');
                listItem.textContent = wheel.name;

                const deleteBtn = document.createElement('button');
                deleteBtn.textContent = 'Delete';
                deleteBtn.style.marginLeft = '10px';
                deleteBtn.addEventListener('click', () => deleteSavedWheel(index));

                listItem.appendChild(deleteBtn);
                listItem.addEventListener('click', () => loadWheel(wheel.segments, wheel.name));
                savedWheelsList.appendChild(listItem);
            });
        }

        // Update spin duration label when slider changes
        spinDurationSlider.addEventListener('input', () => {
            spinDurationLabel.textContent = spinDurationSlider.value;
        });

        // Event listeners
        addSegmentBtn.addEventListener('click', addSegment);
        spinBtn.addEventListener('click', spinWheel);
        saveBtn.addEventListener('click', saveWheel);
        newWheelBtn.addEventListener('click', () => {
            segments = [];
            totalPercentage = 0;
            drawWheel();
            updateWheelDescription();
            wheelTitle.textContent = 'Wheel Spinner';
        });
        shuffleBtn.addEventListener('click', shuffleSegments);

        // Function to shuffle the segments
        function shuffleSegments() {
            for (let i = segments.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [segments[i], segments[j]] = [segments[j], segments[i]]; 
            }
            drawWheel();
            updateWheelDescription();
        }

        // Initial draw of the wheel
        drawWheel();
        updateSavedWheelsList();
    </script>
</body>
</html>
