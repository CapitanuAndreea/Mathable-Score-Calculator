# Mathable Game - Automatic Score Calculator

##  Project Overview
This project implements a computer vision system for automatically detecting newly placed pieces and calculating scores in the **Mathable** board game.  

The system takes sequential images of the board, detects the new piece, recognizes the digits on it using **template matching**, and updates player scores based on the game rules, including multipliers and equation constraints.  

## How It Works

### Task 1 – Board Detection & Piece Placement
- **Background removal:** converted the board image to **HSV color space** and tuned the `Lower Hue` (LH=14) parameter to filter out the brown table background.  
- **Blue border cropping:** manually cropped fixed margins (255px on each side).  
- **Resizing & grid:** standardized the image to `1960×1960` so each cell is `140×140`.  
- **Change detection:** used **frame differencing** (pixel-wise difference between two consecutive board images).  
  - Summed pixel differences per cell.  
  - The cell with the highest sum corresponds to the newly placed piece.  

### Task 2 – Digit Recognition
- Chose **digit-level template matching** instead of full-number matching.  
  - Only 10 templates (0–9) needed, instead of 46 possible numbers.  
- **Template preparation:** manually created `35×55 px` digit templates with minimal white space.  
- **Preprocessing:** converted the detected piece region to **grayscale**, then **binary thresholding** for better contrast.  
- **Contour detection:** used `cv.findContours(..., RETR_EXTERNAL)` to locate digit regions.  
  - Single contour = one-digit number  
  - Two contours = two-digit number  
- **Size filtering:** ignored small noisy contours.  
- **Template matching:** applied **normalized cross-correlation (TM_CCOEFF_NORMED)** between contours and digit templates.  
  - Dynamically resized contours to match template dimensions.  
- **Multi-digit handling:** sorted contours left-to-right and concatenated digits to form numbers (e.g., `4` + `2` → `42`).  

### Task 3 – Score Calculation
- Represented the game board with **3 matrices**:  
  1. **Piece values** (numbers already placed)  
  2. **Constraint squares** (enforcing specific operations)  
  3. **Bonus multipliers** (`3x`, `2x`, `1x`)  

- For each new piece at `(i, j)`:
  - Scanned in 4 directions (**up, down, left, right**) to find possible equations.  
  - Validated equations using **addition, subtraction (absolute), multiplication, and division (excluding ÷0)**.  
  - If the square contained a constraint, only that operation was checked.  
  - Applied **bonus multipliers** (2×, 3×).  
  - If multiple valid equations were completed at once, multiplied the score by the number of equations.  
  - Score = Piece Value × Bonus Multiplier × Number of Valid Equations
  - Updated the board state and accumulated scores per player across rounds. 

