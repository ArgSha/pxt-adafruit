/**
 * MakeCode Memory Game
 * * This game creates a sequence of colors and challenges the player
 * to repeat it.
 * - Press Button A to start a new game.
 * - The Circuit Playground will show a growing sequence of colors.
 * - After the sequence, it's your turn.
 * - Press Button A to cycle through the available colors.
 * - Press Button B to select the color you think is next.
 * - If you're correct, the lights flash GREEN.
 * - If you're wrong, the lights flash RED and the game is over.
 * - The sequence gets one color longer each time you succeed!
 */

// --- Game Configuration ---
// The colors we will use for the game
const gameColors = [
    neopixel.colors(NeoPixelColors.Red),
    neopixel.colors(NeoPixelColors.Green),
    neopixel.colors(NeoPixelColors.Blue),
    neopixel.colors(NeoPixelColors.Yellow)
];

// How long to show each color in the sequence (in milliseconds)
const showTime = 700;
// How long to pause between colors (in milliseconds)
const pauseTime = 300;

// --- Game Variables ---
let strip: neopixel.Strip = null; // This will hold our light strip
let gameSequence: number[] = []; // Stores the computer's random sequence
let playerInputIndex = 0; // Tracks which step the player is on
let isPlayerTurn = false; // true if we are waiting for player input
let isGameStarted = false; // true if a game is in progress
let currentColorIndex = 0; // The color the player is currently cycling through

// --- Game Setup ---
// This code runs once when the device starts up
function onStart() {
    // Initialize the 10 NeoPixels on the Circuit Playground Express
    strip = neopixel.create(board.NEOPIXEL, 10, NeoPixelMode.RGB);
    strip.setBrightness(40); // Set brightness to a reasonable level
    showStartMessage();
}

// --- Main Game Functions ---

// Displays a message to prompt the user to start
function showStartMessage() {
    strip.clear();
    strip.show();
    isGameStarted = false;
    isPlayerTurn = false;
    gameSequence = [];
    playerInputIndex = 0;
    basic.showString("A?"); // Show "A?" on the 5x5 LED grid
}

// Starts a new game or the next level
function nextLevel() {
    isPlayerTurn = false;
    playerInputIndex = 0;
    basic.clearScreen(); // Clear the 5x5 grid

    // Add one new random color to the sequence
    let newColorIndex = Math.randomRange(0, gameColors.length - 1);
    gameSequence.push(newColorIndex);

    // Show the level number on the 5x5 grid
    basic.showNumber(gameSequence.length);

    // Play the full sequence back to the player
    showSequence();
}

// Plays the current sequence of colors on the lights
function showSequence() {
    for (let colorIndex of gameSequence) {
        let color = gameColors[colorIndex];
        strip.showColor(color);
        basic.pause(showTime);
        strip.clear();
        strip.show();
        basic.pause(pauseTime);
    }
    
    // Now it's the player's turn
    startPlayerTurn();
}

// Sets up the device to receive player input
function startPlayerTurn() {
    isPlayerTurn = true;
    currentColorIndex = 0; // Start cycling from the first color
    // Show the first color option
    strip.showColor(gameColors[currentColorIndex]);
    
    // Show a small icon to indicate it's the player's turn
    basic.showIcon(IconNames.SmallSquare);
}

// This function is called when the player selects a color
function checkPlayerInput(selectedColorIndex: number) {
    if (!isPlayerTurn) {
        return; // Do nothing if it's not the player's turn
    }

    let correctColorIndex = gameSequence[playerInputIndex];
    let selectedColor = gameColors[selectedColorIndex];
    
    if (selectedColorIndex == correctColorIndex) {
        // --- CORRECT ---
        playerInputIndex++; // Move to the next input
        
        // Flash green to show success
        flashColor(neopixel.colors(NeoPixelColors.Green));

        if (playerInputIndex >= gameSequence.length) {
            // --- LEVEL COMPLETE ---
            // Player finished the whole sequence
            basic.showIcon(IconNames.Happy);
            basic.pause(1000);
            // Start the next level
            nextLevel();
        } else {
            // --- WAITING FOR NEXT INPUT ---
            // Show the first color option again
            currentColorIndex = 0;
            strip.showColor(gameColors[currentColorIndex]);
            basic.showIcon(IconNames.SmallSquare);
        }

    } else {
        // --- INCORRECT / GAME OVER ---
        flashColor(neopixel.colors(NeoPixelColors.Red));
        basic.showIcon(IconNames.Sad);
        basic.pause(1000);
        
        // Show the correct color they missed
        strip.showColor(gameColors[correctColorIndex]);
        basic.pause(1500);

        // Game over, show start message
        showStartMessage();
    }
}

// Helper function to flash a color
function flashColor(color: number) {
    strip.showColor(color);
    basic.pause(300);
    strip.clear();
    strip.show();
    basic.pause(100);
}


// --- Button Inputs ---

// Button A: Cycle through colors OR Start a new game
input.onButtonEvent(Button.A, input.buttonEventClick(), function() {
    if (!isGameStarted) {
        // --- START NEW GAME ---
        isGameStarted = true;
        nextLevel();
    } else if (isPlayerTurn) {
        // --- CYCLE COLOR ---
        // Move to the next color, wrap around if at the end
        currentColorIndex++;
        if (currentColorIndex >= gameColors.length) {
            currentColorIndex = 0; // Wrap around to the start
        }
        // Show the new current color
        strip.showColor(gameColors[currentColorIndex]);
    }
});

// Button B: Select the current color
input.onButtonEvent(Button.B, input.buttonEventClick(), function() {
    if (isPlayerTurn) {
        // --- SELECT COLOR ---
        checkPlayerInput(currentColorIndex);
    }
});

// --- Run the onStart function ---
onStart();
