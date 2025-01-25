import React, { useState } from "react";

const wheels = {
  viewer: [
    { value: 0, probability: 15 },
    { value: 2, probability: 14 },
    { value: 3, probability: 12 },
    { value: 5, probability: 12 },
    { value: 6, probability: 11 },
    { value: 7, probability: 10 },
    { value: 8, probability: 9 },
    { value: 10, probability: 9 },
    { value: 12, probability: 8 },
  ],
  codeUser: [
    { value: 3, probability: 15 },
    { value: 5, probability: 14 },
    { value: 7, probability: 12 },
    { value: 8, probability: 12 },
    { value: 9, probability: 11 },
    { value: 10, probability: 10 },
    { value: 12, probability: 9 },
    { value: 15, probability: 9 },
    { value: 20, probability: 8 },
  ],
  mixed: [
    { value: 0, probability: 16 },
    { value: 1, probability: 16 },
    { value: 2, probability: 16 },
    { value: 2.5, probability: 12 },
    { value: 3, probability: 10 },
    { value: 3.5, probability: 8 },
    { value: 4, probability: 7 },
    { value: 5, probability: 6 },
    { value: 7, probability: 4 },
  ]
};

function getRandomPrize(wheel, isDouble) {
  const weightedPrizes = wheel.flatMap(({ value, probability }) =>
    Array(probability).fill(value)
  );
  let prize = weightedPrizes[Math.floor(Math.random() * weightedPrizes.length)];
  return isDouble ? prize * 2 : prize;
}

const PrizeWheel = () => {
  const [selectedWheel, setSelectedWheel] = useState("viewer");
  const [prize, setPrize] = useState(null);
  const [isDouble, setIsDouble] = useState(false);

  const spinWheel = () => {
    const selectedPrizes = wheels[selectedWheel];
    setPrize(getRandomPrize(selectedPrizes, isDouble));
    setTimeout(() => {
      alert(`You won: $${prize.toFixed(2)}`);
    }, 300);
  };

  return (
    <div className="flex flex-col items-center justify-center min-h-screen bg-gray-100">
      <h1 className="text-2xl font-bold mb-4">Spin the Prize Wheel!</h1>
      <div className="flex gap-4 mb-4">
        <button onClick={() => setSelectedWheel("viewer")} className="px-4 py-2 bg-gray-300 rounded">Viewer Wheel</button>
        <button onClick={() => setSelectedWheel("codeUser")} className="px-4 py-2 bg-gray-300 rounded">Code User Wheel</button>
        <button onClick={() => setSelectedWheel("mixed")} className="px-4 py-2 bg-gray-300 rounded">Mixed Wheel</button>
      </div>
      <button
        onClick={spinWheel}
        className="px-6 py-3 bg-blue-500 text-white font-semibold rounded-lg shadow-md hover:bg-blue-700"
      >
        Spin
      </button>
      <div className="mt-4">
        <label className="flex items-center space-x-2">
          <input type="checkbox" checked={isDouble} onChange={() => setIsDouble(!isDouble)} />
          <span>Double Prize</span>
        </label>
      </div>
      {prize !== null && (
        <p className="mt-4 text-xl font-semibold">You won: ${prize.toFixed(2)}</p>
      )}
    </div>
  );
};

export default PrizeWheel;
