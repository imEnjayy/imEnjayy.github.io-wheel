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
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Wheel Spinner</h1>
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
            <input type="range" id="spin-duration-slider" min="1" max="10" value="3">
            <span id="spin-duration-label">3</span> seconds

            <button id="spin-btn">Spin!</button>
            <button id="save-btn">Save Wheel</button>
            <button id="new-wheel-btn">Create New Wheel</button>
            <button id="shuffle-btn">Shuffle Segments</button>
        </div>

        <h3>Saved Wheels</h3>
        <ul id="saved-wheels"></ul>
    </div>

    <div class="popup" id="popup">
        <div class="popup-content" id="popup-content"></div>
    </div>

    <script>
        // Initialize global variables
        let segments = [];  // Array to store the segments
        let totalPercentage = 0;
        let angle = 0;
        let spinning = false;  // Flag to track if the wheel is spinning
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
        const popupContent = document.getElementById('popup-content');
        const wheelDescription = document.getElementById('wheel-description');
        const spinDurationSlider = document.getElementById('spin-duration-slider');
        const spinDurationLabel = document.getElementById('spin-duration-label');

        // Function to draw the wheel
        function drawWheel() {
            const radius = wheelCanvas.width / 2;
            const segmentAngle = (2 * Math.PI) / 100;
            let offsetAngle = -Math.PI / 2 + spinAngle; // Start from top

            ctx.clearRect(0, 0, wheelCanvas.width, wheelCanvas.height);
            ctx.translate(radius, radius);

            segments.forEach((segment, i) => {
                const anglePercentage = segment.percentage / 100 * 2 * Math.PI;

                ctx.beginPath();
                ctx.moveTo(0, 0);
                ctx.arc(0, 0, radius, offsetAngle, offsetAngle + anglePercentage);
                ctx.fillStyle = segment.color;
                ctx.fill();

                // Text
                ctx.save();
                ctx.rotate(offsetAngle + anglePercentage / 2);
                ctx.fillStyle = "white";
                ctx.font = "16px Arial";
                ctx.fillText(segment.name, radius / 2, 0);
                ctx.restore();

                offsetAngle += anglePercentage;
            });

            ctx.resetTransform();
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
            if (spinning) return;  // Prevent multiple spins at once

            spinning = true;
            spinAngle = 0;

            // Get the spin duration from the slider and convert to milliseconds
            const spinDuration = spinDurationSlider.value * 1000; // In milliseconds
            const spinTarget = Math.random() * 7200 + 3600; // Randomize final spin target between 3600 and 10800 degrees

            let startTime = null;
            
            function animate(time) {
                if (!startTime) startTime = time; // Store the start time of the animation
                const elapsedTime = time - startTime;

                if (elapsedTime < spinDuration) {
                    const progress = elapsedTime / spinDuration;
                    spinAngle = progress * spinTarget; // Gradually increase the spin angle
                    drawWheel();
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

        // Function to show the pop-up
        function showPopup(winner) {
            popupContent.textContent = `Congratulations! You have won ${winner.name}`;
            popup.style.display = 'flex';
            setTimeout(() => {
                popup.style.display = 'none';
            }, 3000); // Hide the popup after 3 seconds
        }

        // Function to add a segment
        function addSegment() {
            const segmentName = segmentInput.value.trim();
            const segmentPercentage = parseFloat(percentageInput.value);

            if (segmentName && segmentPercentage && totalPercentage + segmentPercentage <= 100) {
                segments.push({ name: segmentName, percentage: segmentPercentage, color: getRandomColor() });
                totalPercentage += segmentPercentage;

                segmentInput.value = '';  // Clear input fields
                percentageInput.value = '';
                drawWheel();
                updateWheelDescription();
            } else {
                alert('Please enter a valid segment name and percentage (total percentage must equal 100)');
            }
        }

        // Function to save the wheel to localStorage
        function saveWheel() {
            const wheelName = prompt('Enter a name for this wheel:');
            if (wheelName) {
                const savedWheels = JSON.parse(localStorage.getItem('savedWheels')) || [];
                savedWheels.push({ name: wheelName, segments: segments });
                localStorage.setItem('savedWheels', JSON.stringify(savedWheels));
                updateSavedWheelsList();
            }
        }

        // Function to load a saved wheel from localStorage
        function loadWheel(loadedSegments) {
            segments = loadedSegments;
            drawWheel();
            updateWheelDescription();
        }

        // Function to delete a saved wheel from localStorage
        function deleteSavedWheel(wheelIndex) {
            const savedWheels = JSON.parse(localStorage.getItem('savedWheels')) || [];
            savedWheels.splice(wheelIndex, 1);
            localStorage.setItem('savedWheels', JSON.stringify(savedWheels));
            updateSavedWheelsList();
        }

        // Function to update the list of saved wheels
        function updateSavedWheelsList() {
            savedWheelsList.innerHTML = '';
            const savedWheels = JSON.parse(localStorage.getItem('savedWheels')) || [];
            savedWheels.forEach((wheel, index) => {
                const listItem = document.createElement('li');
                listItem.classList.add('saved-wheel');
                listItem.textContent = wheel.name;

                // Delete button for each saved wheel
                const deleteBtn = document.createElement('button');
                deleteBtn.textContent = 'Delete';
                deleteBtn.style.marginLeft = '10px';
                deleteBtn.addEventListener('click', () => deleteSavedWheel(index));

                listItem.appendChild(deleteBtn);
                listItem.addEventListener('click', () => loadWheel(wheel.segments));
                savedWheelsList.appendChild(listItem);
            });
        }

        // Helper function to generate random colors for segments
        function getRandomColor() {
            const letters = '0123456789ABCDEF';
            let color = '#';
            for (let i = 0; i < 6; i++) {
                color += letters[Math.floor(Math.random() * 16)];
            }
            return color;
        }

        // Function to shuffle the segments
        function shuffleSegments() {
            for (let i = segments.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [segments[i], segments[j]] = [segments[j], segments[i]]; // Swap elements
            }
            drawWheel();
            updateWheelDescription();
        }

        // Function to create a new wheel
        function createNewWheel() {
            segments = [];
            totalPercentage = 0;
            drawWheel();
            updateWheelDescription();
        }

        // Update the spin duration label when the slider changes
        spinDurationSlider.addEventListener('input', () => {
            spinDurationLabel.textContent = spinDurationSlider.value;
        });

        // Add event listeners
        addSegmentBtn.addEventListener('click', addSegment);
        spinBtn.addEventListener('click', spinWheel);
        saveBtn.addEventListener('click', saveWheel);
        newWheelBtn.addEventListener('click', createNewWheel);
        shuffleBtn.addEventListener('click', shuffleSegments);

        // Initial draw of the wheel
        drawWheel();
        updateSavedWheelsList();
    </script>
</body>
</html>
