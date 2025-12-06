<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Tile Board</title>
  <style>
    body {
      background: #395094;
      display: flex;
      flex-direction: column;
      align-items: center;
      font-family: Arial, sans-serif;
      margin: 20px;
    }

    .board {
      display: grid;
      gap: 2px;
    }

    .main-board {
      grid-template-columns: repeat(8, 40px);
      grid-template-rows: repeat(8, 40px);
      margin-bottom: 20px;
      transition: background 0.3s;
    }

    .main-board.grey {
      background-color: grey;
    }

    .small-boards {
      display: flex;
      gap: 20px;
      margin-bottom: 20px;
    }

    .small-board {
      display: grid;
      grid-template-columns: repeat(3, 30px);
      grid-template-rows: repeat(3, 30px);
      gap: 2px;
      cursor: move;
    }

    .tile {
      width: 100%;
      height: 100%;
      background: url('emptytile.png') no-repeat center center;
      background-size: cover;
      cursor: pointer;
    }

    .palette {
      display: flex;
      gap: 10px;
      margin-top: 20px;
      align-items: center;
    }

    .color {
      width: 30px;
      height: 30px;
      border-radius: 50%;
      cursor: pointer;
      border: 2px solid transparent;
    }

    .selected {
      border: 3px solid black;
    }

    .eraser {
      width: 30px;
      height: 30px;
      border-radius: 6px;
      background: lightgray;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      font-size: 20px;
      border: 2px solid transparent;
    }

    .eraser.selected {
      border: 3px solid black;
    }

    .drag-toggle {
      width: 30px;
      height: 30px;
      border-radius: 6px;
      background: lightgray;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      font-size: 18px;
      border: 2px solid transparent;
    }

.save-reset-buttons {
  margin-top: 20px;
  display: flex;
  gap: 20px;
  justify-content: center; /* Center buttons horizontally */
  align-items: center; /* Ensure all buttons are aligned vertically in the center */
}

.save-reset-button {
  padding: 0;  /* Remove padding to fit the image */
  background-color: transparent; /* Make the background transparent */
  border: none; /* Remove border */
  cursor: pointer;
  display: flex;
  justify-content: center;
  align-items: center;
}

.save-reset-button:hover {
  background-color: #dcdcdc; /* Hover effect */
}

    }

/* Opaque shape clone style */
.opaque-shape {
  position: absolute;
  pointer-events: none;  /* Prevent it from interfering with the rest of the interaction */
  opacity: 0.5;  /* Semi-transparent */
  background-size: cover;
  background-repeat: no-repeat;
  z-index: 999;  /* Ensure it’s above other elements */
}

/* Modal styles */
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: none; /* Hidden by default */
  justify-content: center;
  align-items: center;
}

.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 8px;
  width: 400px;
  max-width: 90%;
  text-align: center;
}

textarea {
  width: 100%;
  margin-bottom: 10px;
}

button {
  margin-top: 10px;
}
  </style>
</head>
<body>

  <div id="mainBoard" class="board main-board"></div>

  <div class="small-boards">
    <div id="smallBoard1" class="small-board" draggable="true"></div>
    <div id="smallBoard2" class="small-board" draggable="true"></div>
    <div id="smallBoard3" class="small-board" draggable="true"></div>
  </div>

  <div class="palette">
    <div class="color" data-color="white" style="background:url('whitetile.png') no-repeat center center; background-size: cover;"></div>
    <div class="color" data-color="black" style="background:url('blacktile.png') no-repeat center center; background-size: cover;"></div>
    <div class="color" data-color="blue" style="background:url('bluetile.png') no-repeat center center; background-size: cover;"></div>
    <div class="color" data-color="green" style="background:url('greentile.png') no-repeat center center; background-size: cover;"></div>
    <div class="color" data-color="orange" style="background:url('orangetile.png') no-repeat center center; background-size: cover;"></div>
    <div class="color" data-color="yellow" style="background:url('yellowtile.png') no-repeat center center; background-size: cover;"></div>
    <div class="color" data-color="red" style="background:url('redtile.png') no-repeat center center; background-size: cover;"></div>
    <div class="color" data-color="lightblue" style="background:url('lightbluetile.png') no-repeat center center; background-size: cover;"></div>
    <div class="eraser" data-color="erase">🩹</div>
    <div class="drag-toggle" id="dragToggle" style="width:30px;height:30px;border-radius:6px;background:lightgray;display:flex;align-items:center;justify-content:center;cursor:pointer;font-size:18px;border:2px solid transparent;">🔁</div>
  </div>

<!-- Add this HTML for the popup modal to import the save string -->
<div id="importModal" class="modal" style="display: none;">
  <div class="modal-content">
    <h2>Import Board</h2>
    <textarea id="saveText" rows="5" cols="50" placeholder="Paste your save string here"></textarea>
    <button id="importButtonModal" class="save-reset-button">Import</button>
    <button id="closeImportModal" class="save-reset-button">Close</button>
  </div>
</div>

<div class="save-reset-buttons">
  <button id="saveButton" class="save-reset-button">
    <img src="save.png" alt="Save" style="width: 100px; height: 50px;">
  </button>
  <button id="openImportModalButton" class="save-reset-button">
    <img src="import.png" alt="Import" style="width: 100px; height: 50px;">
  </button>
  <button id="checkpointButton" class="save-reset-button">
    <img src="checkpoint.png" alt="Checkpoint" style="width: 100px; height: 50px;">
  </button>
  <button id="backButton" class="save-reset-button">
    <img src="back.png" alt="Back" style="width: 100px; height: 50px;">
  </button>
</div>

<!-- Message for save feedback -->
<div id="saveFeedback" style="display:none; color: green;">Save copied to clipboard!</div>

  <script>
    let selectedColor = null;
    let dragMode = false;
    let draggedBoard = null;
    let draggedBoardId = null;
    let savedState = null;  // Variable to store the saved state of the board and small boards
    let checkpointState = null; // Variable to store the checkpoint state


function saveBoardState() {
  const mainBoardTiles = Array.from(document.getElementById('mainBoard').children).map(tile => tile.style.background);
  const smallBoard1Tiles = Array.from(document.getElementById('smallBoard1').children).map(tile => tile.style.background);
  const smallBoard2Tiles = Array.from(document.getElementById('smallBoard2').children).map(tile => tile.style.background);
  const smallBoard3Tiles = Array.from(document.getElementById('smallBoard3').children).map(tile => tile.style.background);

  const savedState = {
    mainBoard: mainBoardTiles,
    smallBoard1: smallBoard1Tiles,
    smallBoard2: smallBoard2Tiles,
    smallBoard3: smallBoard3Tiles
  };

  // Convert the saved state to a JSON string
  const savedStateString = JSON.stringify(savedState);

  // Copy to clipboard
  navigator.clipboard.writeText(savedStateString).then(() => {
    console.log('Board saved and copied to clipboard!');
    // Show feedback message
    const saveFeedback = document.getElementById('saveFeedback');
    saveFeedback.style.display = 'block';

    // Fade out the message after 3 seconds
    setTimeout(() => {
      saveFeedback.style.display = 'none';
    }, 3000);
  }).catch(err => {
    console.error('Error copying to clipboard: ', err);
  });
}

// Open the import modal when the Import button is clicked
document.getElementById('openImportModalButton').addEventListener('click', () => {
  document.getElementById('importModal').style.display = 'flex';
});

// Close the import modal
document.getElementById('closeImportModal').addEventListener('click', () => {
  document.getElementById('importModal').style.display = 'none';
});

// Import the board state from the pasted JSON string
function importBoardState() {
  const savedStateString = document.getElementById('saveText').value;

  if (!savedStateString) {
    alert('No saved state to import!');
    return;
  }

  try {
    // Parse the saved state string into an object
    const savedState = JSON.parse(savedStateString);

    // Apply the saved state to the main board and small boards
    const mainBoardTiles = Array.from(document.getElementById('mainBoard').children);
    savedState.mainBoard.forEach((background, index) => {
      mainBoardTiles[index].style.background = background;
      mainBoardTiles[index].style.backgroundSize = "cover";
    });

    const smallBoard1Tiles = Array.from(document.getElementById('smallBoard1').children);
    savedState.smallBoard1.forEach((background, index) => {
      smallBoard1Tiles[index].style.background = background;
      smallBoard1Tiles[index].style.backgroundSize = "cover";
    });

    const smallBoard2Tiles = Array.from(document.getElementById('smallBoard2').children);
    savedState.smallBoard2.forEach((background, index) => {
      smallBoard2Tiles[index].style.background = background;
      smallBoard2Tiles[index].style.backgroundSize = "cover";
    });

    const smallBoard3Tiles = Array.from(document.getElementById('smallBoard3').children);
    savedState.smallBoard3.forEach((background, index) => {
      smallBoard3Tiles[index].style.background = background;
      smallBoard3Tiles[index].style.backgroundSize = "cover";
    });

    console.log('Board imported successfully!');
    document.getElementById('importModal').style.display = 'none'; // Close the modal after import
  } catch (e) {
    alert('Failed to import the save state. Please make sure the string is correct.');
    console.error('Error parsing saved state: ', e);
  }
}

// Link the import button inside the modal to the import function
document.getElementById('importButtonModal').addEventListener('click', importBoardState);


    // Reset the board to its saved state
    function resetBoardState() {
      if (savedState) {
        const mainBoardTiles = Array.from(document.getElementById('mainBoard').children);
        savedState.mainBoard.forEach((background, index) => {
          mainBoardTiles[index].style.background = background;
          mainBoardTiles[index].style.backgroundSize = "cover";
        });

        const smallBoard1Tiles = Array.from(document.getElementById('smallBoard1').children);
        savedState.smallBoard1.forEach((background, index) => {
          smallBoard1Tiles[index].style.background = background;
          smallBoard1Tiles[index].style.backgroundSize = "cover";
        });

        const smallBoard2Tiles = Array.from(document.getElementById('smallBoard2').children);
        savedState.smallBoard2.forEach((background, index) => {
          smallBoard2Tiles[index].style.background = background;
          smallBoard2Tiles[index].style.backgroundSize = "cover";
        });

        const smallBoard3Tiles = Array.from(document.getElementById('smallBoard3').children);
        savedState.smallBoard3.forEach((background, index) => {
          smallBoard3Tiles[index].style.background = background;
          smallBoard3Tiles[index].style.backgroundSize = "cover";
        });
      }
    }

function importBoardState() {
  const savedStateString = document.getElementById('saveText').value;

  if (!savedStateString) {
    alert('No saved state to import!');
    return;
  }

  try {
    // Parse the saved state string into an object
    const savedState = JSON.parse(savedStateString);

    // Apply the saved state to the main board and small boards
    const mainBoardTiles = Array.from(document.getElementById('mainBoard').children);
    savedState.mainBoard.forEach((background, index) => {
      mainBoardTiles[index].style.background = background;
      mainBoardTiles[index].style.backgroundSize = "cover";
    });

    const smallBoard1Tiles = Array.from(document.getElementById('smallBoard1').children);
    savedState.smallBoard1.forEach((background, index) => {
      smallBoard1Tiles[index].style.background = background;
      smallBoard1Tiles[index].style.backgroundSize = "cover";
    });

    const smallBoard2Tiles = Array.from(document.getElementById('smallBoard2').children);
    savedState.smallBoard2.forEach((background, index) => {
      smallBoard2Tiles[index].style.background = background;
      smallBoard2Tiles[index].style.backgroundSize = "cover";
    });

    const smallBoard3Tiles = Array.from(document.getElementById('smallBoard3').children);
    savedState.smallBoard3.forEach((background, index) => {
      smallBoard3Tiles[index].style.background = background;
      smallBoard3Tiles[index].style.backgroundSize = "cover";
    });

    console.log('Board imported successfully!');

// Automatically save the imported board as a checkpoint
    saveCheckpoint();

  } catch (e) {
    alert('Failed to import the save state. Please make sure the string is correct.');
    console.error('Error parsing saved state: ', e);
  }
}

function saveCheckpoint() {
  // Save the current state of the main board and small boards
  const mainBoardTiles = Array.from(document.getElementById('mainBoard').children).map(tile => tile.style.background);
  const smallBoard1Tiles = Array.from(document.getElementById('smallBoard1').children).map(tile => tile.style.background);
  const smallBoard2Tiles = Array.from(document.getElementById('smallBoard2').children).map(tile => tile.style.background);
  const smallBoard3Tiles = Array.from(document.getElementById('smallBoard3').children).map(tile => tile.style.background);

  checkpointState = {
    mainBoard: mainBoardTiles,
    smallBoard1: smallBoard1Tiles,
    smallBoard2: smallBoard2Tiles,
    smallBoard3: smallBoard3Tiles
  };

  console.log('Checkpoint saved!');
}

function loadCheckpoint() {
  if (!checkpointState) {
    alert('No checkpoint saved!');
    return;
  }

  // Restore the state of the main board and small boards from the checkpoint
  const mainBoardTiles = Array.from(document.getElementById('mainBoard').children);
  checkpointState.mainBoard.forEach((background, index) => {
    mainBoardTiles[index].style.background = background;
    mainBoardTiles[index].style.backgroundSize = "cover";
  });

  const smallBoard1Tiles = Array.from(document.getElementById('smallBoard1').children);
  checkpointState.smallBoard1.forEach((background, index) => {
    smallBoard1Tiles[index].style.background = background;
    smallBoard1Tiles[index].style.backgroundSize = "cover";
  });

  const smallBoard2Tiles = Array.from(document.getElementById('smallBoard2').children);
  checkpointState.smallBoard2.forEach((background, index) => {
    smallBoard2Tiles[index].style.background = background;
    smallBoard2Tiles[index].style.backgroundSize = "cover";
  });

  const smallBoard3Tiles = Array.from(document.getElementById('smallBoard3').children);
  checkpointState.smallBoard3.forEach((background, index) => {
    smallBoard3Tiles[index].style.background = background;
    smallBoard3Tiles[index].style.backgroundSize = "cover";
  });

  console.log('Board restored from checkpoint!');
}

    function createBoard(elementId, size) {
      const board = document.getElementById(elementId);
      for (let i = 0; i < size * size; i++) {
        const tile = document.createElement("div");
        tile.className = "tile";
        tile.dataset.original = "emptytile";
        tile.style.background = "url('emptytile.png') no-repeat center center";
        tile.style.backgroundSize = "cover";
        tile.addEventListener("click", () => {
          if (selectedColor === "erase") {
            tile.style.background = "url('emptytile.png') no-repeat center center";
            tile.style.backgroundSize = "cover";
          } else if (selectedColor) {
            tile.style.background = `url('${selectedColor}tile.png') no-repeat center center`;
            tile.style.backgroundSize = "cover";
          }
          checkAndClearLine(); // Check after color change
        });
        board.appendChild(tile);
      }
    }

    createBoard("mainBoard", 8);
    createBoard("smallBoard1", 3);
    createBoard("smallBoard2", 3);
    createBoard("smallBoard3", 3);

    const paletteItems = document.querySelectorAll(".color, .eraser");
    const dragToggle = document.getElementById("dragToggle");

    // Handle tile selection and eraser
    paletteItems.forEach(item => {
      item.addEventListener("click", () => {
        // Disable drag mode when a tile or eraser is selected
        if (dragMode) {
          dragMode = false;
          dragToggle.style.border = "2px solid transparent";
        }

        // Enable tile mode
        paletteItems.forEach(i => i.classList.remove("selected"));
        item.classList.add("selected");
        selectedColor = item.dataset.color;
      });
    });

    // Handle drag mode toggle
    dragToggle.addEventListener("click", () => {
      // Disable tile mode when drag mode is activated
      if (selectedColor) {
        selectedColor = null;
        paletteItems.forEach(i => i.classList.remove("selected"));
      }

      // Toggle drag mode
      dragMode = !dragMode;
      dragToggle.style.border = dragMode ? "3px solid black" : "2px solid transparent";
    });

    // Drag & Drop Logic
    const smallBoards = document.querySelectorAll('.small-board');
    smallBoards.forEach(board => {
      board.addEventListener("dragstart", (event) => {
        if (!dragMode) return; // Allow dragging only if dragMode is active
        draggedBoard = board;
        draggedBoardId = board.id;
        board.style.opacity = "0.5";
      });

      board.addEventListener("dragend", () => {
        if (draggedBoard) {
          draggedBoard.style.opacity = "1";
          draggedBoard = null;
          draggedBoardId = null;
        }
      });
    });

// Link the "Checkpoint" button to the saveCheckpoint function
document.getElementById('checkpointButton').addEventListener('click', saveCheckpoint);

// Link the "Back" button to the loadCheckpoint function
document.getElementById('backButton').addEventListener('click', loadCheckpoint);

    const mainBoard = document.getElementById('mainBoard');

    // Enable dropping on the main board
    mainBoard.addEventListener('dragover', event => {
      event.preventDefault();
    });

    mainBoard.addEventListener('drop', event => {
      event.preventDefault();
      if (!draggedBoard) return;

      // Add grey background to the main board when a shape is dropped
      mainBoard.classList.add('grey');

      const tiles = Array.from(mainBoard.children);
      const data = Array.from(draggedBoard.children).map(tile => tile.style.background || "emptytile");

      const rect = mainBoard.getBoundingClientRect();
      const x = event.clientX - rect.left;
      const y = event.clientY - rect.top;

      const tileSize = rect.width / 8;
      const row = Math.floor(y / tileSize);
      const col = Math.floor(x / tileSize);

for (let r = 0; r < 3; r++) {
  for (let c = 0; c < 3; c++) {
    const index = (row + r) * 8 + (col + c);
    if (tiles[index]) {
      const tileColor = data[r * 3 + c];
      // Only update the tile if it's empty
      if (tileColor && tileColor !== "emptytile" && tiles[index].style.background.includes('emptytile')) {
        tiles[index].style.background = tileColor;
        tiles[index].style.backgroundSize = "cover";
      }
    }
  }
}


      // Clear the small board after drop
      clearSmallBoard(draggedBoardId);

      checkAndClearLine(); // Check after drop
      draggedBoard.style.opacity = "1"; // Reset opacity after dropping
      draggedBoard = null; // Clear the dragged board
      draggedBoardId = null; // Reset the dragged board ID

      // Reset the grey background after a short delay to simulate the action
      setTimeout(() => {
        mainBoard.classList.remove('grey');
      }, 1000); // 1 second delay before returning the board to normal
    });

    // Function to clear the small board after dropping
    function clearSmallBoard(boardId) {
      const board = document.getElementById(boardId);
      const tiles = board.children;
      for (let tile of tiles) {
        tile.style.background = "url('emptytile.png') no-repeat center center";
        tile.style.backgroundSize = "cover";
      }
    }

    // Check if there are 8 colored tiles in a row or column
    function checkAndClearLine() {
      const rows = 8;
      const cols = 8;
      const tiles = Array.from(document.getElementById('mainBoard').children);

      // Check rows for 8 colored tiles
      for (let row = 0; row < rows; row++) {
        let allColored = true;
        for (let col = 0; col < cols; col++) {
          const tile = tiles[row * cols + col];
          if (tile.style.background.includes('emptytile')) {
            allColored = false;
            break;
          }
        }
        if (allColored) {
          for (let col = 0; col < cols; col++) {
            tiles[row * cols + col].style.background = "url('emptytile.png') no-repeat center center";
            tiles[row * cols + col].style.backgroundSize = "cover";
          }
        }
      }

      // Check columns for 8 colored tiles
      for (let col = 0; col < cols; col++) {
        let allColored = true;
        for (let row = 0; row < rows; row++) {
          const tile = tiles[row * cols + col];
          if (tile.style.background.includes('emptytile')) {
            allColored = false;
            break;
          }
        }
        if (allColored) {
          for (let row = 0; row < rows; row++) {
            tiles[row * cols + col].style.background = "url('emptytile.png') no-repeat center center";
            tiles[row * cols + col].style.backgroundSize = "cover";
          }
        }
      }
    }

    // Save and Reset button functionality
    document.getElementById('saveButton').addEventListener('click', saveBoardState);
    document.getElementById('resetButton').addEventListener('click', resetBoardState);
    document.getElementById('importButton').addEventListener('click', importBoardState);

  </script>
</body>
</html>
