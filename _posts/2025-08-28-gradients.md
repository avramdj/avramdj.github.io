---
title: "Exploring the gradient noise scale"
description: "An expansion of \"Why Momentum Really Works\" by Gabriel Goh"
date: 2025-08-28
image: 
    path: assets/img/gradient.png
    alt: "Gradient descent"
    hide: true
comments: false
toc: false
categories: [Machine Learning]
tags: [math, optimization, visualization]
math: true
---

This is heavily inspired by ["Why Momentum Really Works"](https://distill.pub/2017/momentum) by Gabriel Goh and [An Empirical Model of Large-Batch Training
](https://arxiv.org/abs/1812.06162).


<style>
  .gradient-container {
    display: flex;
    justify-content: center;
    align-items: flex-start;
    margin-top: 20px;
    gap: 20px;
  }
  .controls-panel {
    display: flex;
    flex-direction: column;
    gap: 15px;
    width: 220px;
    text-align: left;
    font-family: sans-serif;
  }
  .control-group {
    display: flex;
    flex-direction: column;
  }
  .control-group label {
    margin-bottom: 5px;
    font-weight: bold;
    font-size: 0.9em;
    color: var(--text-color);
  }
  .input-slider-group {
    display: flex;
    align-items: center;
    gap: 10px;
  }
  .input-slider-group input[type="number"] {
    width: 60px;
    text-align: right;
    background-color: var(--main-bg-color);
    color: var(--text-color);
    border: 1px solid var(--border-color);
    border-radius: 5px;
  }
  /* Custom Range Slider Styles */
    input[type="range"] {
        -webkit-appearance: none;
        appearance: none;
        width: 100%;
        height: 4px;
        background: #e2e8f0;
        border-radius: 5px;
        outline: none;
    }

    input[type="range"]::-webkit-slider-thumb {
        -webkit-appearance: none;
        appearance: none;
        width: 16px;
        height: 16px;
        background: var(--text-color);
        cursor: pointer;
        border-radius: 50%;
    }

    input[type="range"]::-moz-range-thumb {
        width: 16px;
        height: 16px;
        background: var(--text-color);
        cursor: pointer;
        border-radius: 50%;
        border: none;
    }
  #gradient-tooltip {
    position: fixed;
    display: none;
    background: rgba(0, 0, 0, 0.4);
    color: white;
    padding: 5px 10px;
    border-radius: 5px;
    font-family: sans-serif;
    pointer-events: none;
  }
  #reset-gradient-button {
      width: 100%;
      padding: 8px;
      font-weight: bold;
      background-color: var(--button-bg);
      border: 1px solid var(--button-border-color);
      color: var(--button-text-color);
      border-radius: 5px;
      cursor: pointer;
  }
  #reset-gradient-button:hover {
      background-color: var(--button-hover-bg);
  }

  /* Simple checkbox styling */
  .checkbox-label {
    display: flex;
    align-items: center;
    cursor: pointer;
    font-weight: 500;
    color: var(--text-color);
    font-size: 0.9em;
  }

  .checkbox-label input[type="checkbox"] {
    margin-right: 8px;
    width: 16px;
    height: 16px;
    cursor: pointer;
  }

  .debug-button {
    background: var(--button-bg);
    border: 1px solid var(--border-color);
    color: var(--text-color);
    font-size: 11px;
    padding: 6px 12px;
    border-radius: 6px;
    cursor: pointer;
    transition: all 0.2s ease;
    font-weight: 500;
    letter-spacing: 0.025em;
    opacity: 0.8;
  }

  .debug-button:hover {
    opacity: 1;
    transform: translateY(-1px);
  }

  .debug-button:active {
    transform: translateY(0);
  }

  .lr-decay-buttons {
    display: flex;
    border-radius: 6px;
    overflow: hidden;
    border: 1px solid var(--border-color);
  }

  .decay-button {
    flex: 1;
    padding: 6px 8px;
    font-size: 11px;
    font-weight: 500;
    border: none;
    background: var(--button-bg);
    color: var(--text-color);
    cursor: pointer;
    transition: all 0.2s ease;
    border-right: 1px solid var(--border-color);
    opacity: 0.7;
  }

  .decay-button:last-child {
    border-right: none;
  }

  .decay-button:hover {
    opacity: 0.9;
  }

  /* Active LR decay button: high-contrast inversion per theme */
  .decay-button.active {
    /* Light mode (default): black bg, white text */
    background: #000000;
    color: #ffffff;
    opacity: 1;
  }

  @media (prefers-color-scheme: dark) {
    .decay-button.active {
      /* Dark mode: white bg, black text */
      background: #ffffff;
      color: #000000;
    }
  }

  /* Disabled state styling - only affects the input-slider-group, not checkboxes */
  .control-group.disabled .input-slider-group {
    opacity: 0.5;
    pointer-events: none;
  }

  .control-group.disabled .input-slider-group input {
    background-color: var(--main-bg-color);
    opacity: 0.6;
  }

  .control-group.disabled > label:first-child {
    opacity: 0.6;
  }

  /* Beta2 disabled look */
  #beta2-control-group.disabled {
    opacity: 0.5;
  }

  /* Learning rate scale styling */
  .lr-scale-container {
    width: 600px;
    margin: 15px 0;
    font-family: sans-serif;
  }

  .lr-scale-label {
    font-weight: 600;
    font-size: 0.9em;
    color: var(--text-color);
    margin-bottom: 8px;
    text-align: center;
  }

  .lr-scale-wrapper {
    position: relative;
    background: #e2e8f0;
    border-radius: 6px;
    height: 8px;
    margin-bottom: 5px;
  }

  .lr-scale-slider {
    -webkit-appearance: none;
    appearance: none;
    width: 100%;
    height: 8px;
    background: transparent;
    border-radius: 6px;
    outline: none;
    position: absolute;
    top: 0;
    left: 0;
  }

  .lr-scale-slider::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    width: 20px;
    height: 20px;
    background: var(--text-color);
    cursor: pointer;
    border-radius: 50%;
    box-shadow: 0 2px 4px rgba(0,0,0,0.2);
  }

  .lr-scale-slider::-moz-range-thumb {
    width: 20px;
    height: 20px;
    background: var(--text-color);
    cursor: pointer;
    border-radius: 50%;
    border: none;
    box-shadow: 0 2px 4px rgba(0,0,0,0.2);
  }

  .lr-scale-value {
    text-align: center;
    font-size: 0.85em;
    color: var(--text-color);
    font-weight: 500;
    margin-top: 5px;
  }

  .lr-scale-ticks {
    display: flex;
    justify-content: space-between;
    font-size: 0.7em;
    color: var(--text-color);
    opacity: 0.7;
    margin-top: 5px;
  }

  @media (max-width: 768px) {
      .gradient-container {
          flex-direction: column;
          align-items: center;
      }
      .controls-panel {
          width: 100%;
          max-width: 400px;
      }
      .lr-scale-container {
        width: 100%;
        max-width: 400px;
      }
      #gradientCanvas {
          width: 100%;
          max-width: 400px;
          height: auto;
      }
  }
</style>

<div class="gradient-container">
  <div style="display: flex; flex-direction: column; align-items: center;">
    <!-- Objective selector (moved above canvas) -->
    <div class="control-group" style="width: 600px; margin-bottom: 10px;">
      <label>Objective</label>
      <div class="lr-decay-buttons">
        <button id="obj-rosen" class="decay-button active">Rosenbrock</button>
        <button id="obj-himmel" class="decay-button">Himmelblau</button>
        <button id="obj-camel" class="decay-button">Three-hump camel</button>
      </div>
    </div>
    <canvas id="gradientCanvas" width="1200" height="1200" style="width: 600px; height: 600px; border:1px solid var(--border-color); border-radius: 5px; background-color: white; cursor: grab;"></canvas>
    <!-- Learning Rate Scale -->
    <div class="lr-scale-container">
      <div class="lr-scale-label">Learning Rate</div>
      <div class="lr-scale-wrapper">
        <input type="range" id="lr-scale-slider" class="lr-scale-slider" min="1" max="10000" value="7000">
      </div>
      <div class="lr-scale-value" id="lr-scale-value">0.0050</div>
      <div class="lr-scale-ticks">
        <span>0.001</span>
        <span>0.01</span>
        <span>0.1</span>
        <span>1.0</span>
      </div>
    </div>

    
  </div>
  <div class="controls-panel">
    <div class="control-group" id="steps-control-group">
      <label for="steps-input">Steps</label>
      <div class="input-slider-group">
        <input type="number" id="steps-input" min="1" max="400" value="200" step="1">
        <input type="range" id="steps-slider" min="1" max="400" value="200" step="1">
      </div>
      <label for="until-converge-toggle" class="checkbox-label" style="margin-top: 8px;">
        <input type="checkbox" id="until-converge-toggle">
        Run until convergence
      </label>
    </div>
    <div class="control-group">
              <label for="momentum-input">Beta1 (Momentum)</label>
        <div class="input-slider-group">
          <input type="number" id="momentum-input" min="0" max="0.999" value="0.9" step="0.001">
          <input type="range" id="momentum-slider" min="0" max="999" value="900">
        </div>
    </div>
    <div class="control-group" id="beta2-control-group">
              <label for="beta2-input">Beta2 (RMSprop)</label>
        <div class="input-slider-group">
          <input type="number" id="beta2-input" min="0" max="1" value="0.999" step="0.001">
          <input type="range" id="beta2-slider" min="0" max="1000" value="999">
        </div>
    </div>
    <div class="control-group">
      <label for="warmup-input">LR Warmup Steps</label>
      <div class="input-slider-group">
        <input type="number" id="warmup-input" min="0" max="200" value="0" step="1">
        <input type="range" id="warmup-slider" min="0" max="200" value="0">
      </div>
    </div>
    <div class="control-group">
      <label for="noise-input">Additive Noise</label>
      <div class="input-slider-group">
        <input type="number" id="noise-input" min="0" max="1" value="0" step="0.01">
        <input type="range" id="noise-slider" min="0" max="100" value="0">
      </div>
    </div>
    <div class="control-group">
      <label for="pct-noise-input">% Noise</label>
      <div class="input-slider-group">
        <input type="number" id="pct-noise-input" min="0" max="1" value="0.2" step="0.01">
        <input type="range" id="pct-noise-slider" min="0" max="100" value="20">
      </div>
    </div>
    <div class="control-group">
        <button id="reset-gradient-button">Reset</button>
    </div>
    <div class="control-group">
        <label>Optimizer</label>
        <div class="lr-decay-buttons">
            <button id="opt-sgd" class="decay-button active">SGD</button>
            <button id="opt-adam" class="decay-button">Adam</button>
        </div>
    </div>
    <div class="control-group">
        <label>LR Decay</label>
        <div class="lr-decay-buttons">
            <button id="decay-none" class="decay-button active">None</button>
            <button id="decay-linear" class="decay-button">Linear</button>
            <button id="decay-cosine" class="decay-button">Cosine</button>
        </div>
    </div>
    <div class="control-group">
        <button id="debug-toggle" class="debug-button">Debug ▼</button>
    </div>
    <div id="debug-section" style="display: none;">
        <div class="control-group">
            <label for="tooltip-toggle" class="checkbox-label">
                <input type="checkbox" id="tooltip-toggle">
                Show Tooltip
            </label>
        </div>
        <div class="control-group">
          <label for="noise-seed-input">Noise Seed</label>
          <input type="number" id="noise-seed-input" min="0" max="999" value="42" step="1" style="width: 80px;">
        </div>
        <div class="control-group">
          <label for="converge-threshold-input">Convergence Threshold</label>
          <input type="number" id="converge-threshold-input" min="0.001" max="1" value="0.0015" step="0.001" style="width: 80px;">
        </div>
        <div class="control-group">
          <label for="max-steps-input">Max Steps (Convergence)</label>
          <input type="number" id="max-steps-input" min="50" max="1000" value="300" step="10" style="width: 80px;">
        </div>
    </div>
  </div>
  <div id="gradient-tooltip"></div>
</div>

<script>
document.addEventListener('DOMContentLoaded', function () {
    const canvas = document.getElementById('gradientCanvas');
    if (!canvas) return;
    
    const ctx = canvas.getContext('2d');
    
    /* Enable high-DPI rendering */
    const dpr = window.devicePixelRatio || 1;
    const displayWidth = 600;
    const displayHeight = 600;
    
    canvas.width = displayWidth * 1.5; /* 1.5x resolution for crisp rendering */
    canvas.height = displayHeight * 1.5;
    canvas.style.width = displayWidth + 'px';
    canvas.style.height = displayHeight + 'px';
    
    ctx.scale(1.5, 1.5); /* Scale context to match resolution */
    ctx.imageSmoothingEnabled = true;
    ctx.imageSmoothingQuality = 'high';
    
    /* Get control elements */
    const stepsSlider = document.getElementById('steps-slider');
    const lrScaleSlider = document.getElementById('lr-scale-slider');
    const lrScaleValue = document.getElementById('lr-scale-value');
    const momentumSlider = document.getElementById('momentum-slider');
    const beta2Slider = document.getElementById('beta2-slider');
    const warmupSlider = document.getElementById('warmup-slider');
    const noiseSlider = document.getElementById('noise-slider');
    const pctNoiseSlider = document.getElementById('pct-noise-slider');
    const resetButton = document.getElementById('reset-gradient-button');
    const decayNoneBtn = document.getElementById('decay-none');
    const decayLinearBtn = document.getElementById('decay-linear');
    const decayCosineBtn = document.getElementById('decay-cosine');
    const optSgdBtn = document.getElementById('opt-sgd');
    const optAdamBtn = document.getElementById('opt-adam');
    const debugToggle = document.getElementById('debug-toggle');
    const debugSection = document.getElementById('debug-section');
    const untilConvergeToggle = document.getElementById('until-converge-toggle');
    const objRosenBtn = document.getElementById('obj-rosen');
    const objHimmelBtn = document.getElementById('obj-himmel');
    const objCamelBtn = document.getElementById('obj-camel');
    
    let lrDecayMode = 'none'; /* 'none', 'linear', 'cosine' */
    let optimizerMode = 'sgd'; /* 'sgd' | 'adam' */
    let objectiveMode = 'rosen'; /* 'rosen' | 'himmel' | 'camel' */
    
    const stepsInput = document.getElementById('steps-input');
    const momentumInput = document.getElementById('momentum-input');
    const beta2Input = document.getElementById('beta2-input');
    const warmupInput = document.getElementById('warmup-input');
    const noiseInput = document.getElementById('noise-input');
    const pctNoiseInput = document.getElementById('pct-noise-input');
    const noiseSeedInput = document.getElementById('noise-seed-input');
    const convergeThresholdInput = document.getElementById('converge-threshold-input');
    const maxStepsInput = document.getElementById('max-steps-input');
    
    /* Starting position (controlled by click/drag) */
    let startX = 1.3;
    let startY = 1.3;
    
    /* Visualization parameters (will be adjusted per objective) */
    let xMin = -0.4, xMax = 1.8, yMin = -0.6, yMax = 1.8;
    const width = displayWidth, height = displayHeight;
    
    /* Simple seeded random number generator (Linear Congruential Generator) */
    function SeededRandom(seed) {
        this.seed = seed % 2147483647;
        if (this.seed <= 0) this.seed += 2147483646;
    }
    
    SeededRandom.prototype.next = function() {
        this.seed = this.seed * 16807 % 2147483647;
        return (this.seed - 1) / 2147483646;
    };
    
    /* Function and gradient - Scaled and rotated Rosenbrock function */
    function f(x, y, a, b) {
        /* Select objective function */
        if (objectiveMode === 'rosen') {
            /* Rotated/scaled Rosenbrock */
            const angle = Math.PI / 12; /* 15 degrees */
            const cos_a = Math.cos(angle);
            const sin_a = Math.sin(angle);
            const x_rot = cos_a * x - sin_a * y;
            const y_rot = sin_a * x + cos_a * y;
            const x_scaled = x_rot * 1.5 + 0.5;
            const y_scaled = y_rot * 1.5 + 0.5;
            return (1 - x_scaled) * (1 - x_scaled) + 3 * (y_scaled - x_scaled * x_scaled) * (y_scaled - x_scaled * x_scaled);
        } else if (objectiveMode === 'himmel') {
            /* Original Himmelblau's function (no scaling) */
            const X = x, Y = y;
            const term1 = (X * X + Y - 11);
            const term2 = (X + Y * Y - 7);
            return term1 * term1 + term2 * term2;
        } else if (objectiveMode === 'camel') {
            /* Three-hump camel function (no scaling) */
            const X = x, Y = y;
            return 2 * X * X - 1.05 * Math.pow(X, 4) + Math.pow(X, 6) / 6 + X * Y + Y * Y;
        }
        return 0;
    }
    
    function grad(x, y, a, b) {
        /* Compute gradient using finite differences for general functions */
        const h = 1e-6;
        const df_dx = (f(x + h, y, a, b) - f(x - h, y, a, b)) / (2 * h);
        const df_dy = (f(x, y + h, a, b) - f(x, y - h, a, b)) / (2 * h);
        return [df_dx, df_dy];
    }
    
    /* Coordinate transforms */
    function toCanvas(x, y) {
        return [(x - xMin) / (xMax - xMin) * width, height - (y - yMin) / (yMax - yMin) * height];
    }
    
    function toWorld(cx, cy) {
        return [(cx / width) * (xMax - xMin) + xMin, yMax - (cy / height) * (yMax - yMin)];
    }
    
    /* Adam optimizer with learning rate warmup, two types of gradient noise, decay options, and convergence */
    function gradientDescent(startX, startY, steps, lr, beta1, beta2, warmupStepsInput, additiveNoise, pctNoise, lrDecayMode, noiseSeed, useConvergence, convergeThreshold, maxSteps, optimizerMode) {
        const path = [{x: startX, y: startY}];
        let x = startX, y = startY;
        let mx = 0, my = 0; /* First moment estimates */
        let vx = 0, vy = 0; /* Second moment estimates */
        const actualSteps = useConvergence ? maxSteps : steps;
        const warmupSteps = Math.max(0, Math.min(actualSteps, Math.floor(warmupStepsInput)));
        const rngAdditive = new SeededRandom(noiseSeed);
        const rngPct = new SeededRandom(noiseSeed + 1);
        const epsilon = 1e-8; /* Small constant for numerical stability */
        
        for (let i = 0; i < actualSteps; i++) {
            const [gx, gy] = grad(x, y, 1, 1); /* Fixed parameters since we removed curvature controls */
            
            /* Check convergence condition if enabled */
            if (useConvergence) {
                const fval = f(x, y, 1, 1);
                if (fval <= convergeThreshold) {
                    break;
                }
            }
            
            /* Apply separate additive and percentage noise */
            let noisyGx = gx, noisyGy = gy;
            
            /* Additive noise: fixed magnitude */
            if (additiveNoise > 0) {
                const additiveScale = 5.0; /* 10x bigger than before (was 0.5) */
                const noiseX = (rngAdditive.next() - 0.5) * 2;
                const noiseY = (rngAdditive.next() - 0.5) * 2;
                noisyGx += additiveNoise * additiveScale * noiseX;
                noisyGy += additiveNoise * additiveScale * noiseY;
            }
            
            /* Percentage noise: scales with gradient magnitude */
            if (pctNoise > 0) {
                const gradMagnitude = Math.sqrt(gx*gx + gy*gy);
                const noiseX = (rngPct.next() - 0.5) * 2;
                const noiseY = (rngPct.next() - 0.5) * 2;
                noisyGx += pctNoise * gradMagnitude * noiseX;
                noisyGy += pctNoise * gradMagnitude * noiseY;
            }
            
            /* Apply learning rate warmup and decay */
            let currentLr = lr;
            if (i < warmupSteps && warmupSteps > 0) {
                currentLr = lr * (i + 1) / warmupSteps;
            } else if (lrDecayMode === 'linear') {
                /* Linear decay after warmup: linearly decrease from lr to 0 */
                const decaySteps = actualSteps - warmupSteps;
                const decayProgress = (i - warmupSteps) / decaySteps;
                currentLr = lr * (1 - decayProgress);
            } else if (lrDecayMode === 'cosine') {
                /* Cosine decay after warmup: smooth cosine curve from lr to 0 */
                const decaySteps = actualSteps - warmupSteps;
                const decayProgress = (i - warmupSteps) / decaySteps;
                currentLr = lr * 0.5 * (1 + Math.cos(Math.PI * decayProgress));
            }
            /* else: lrDecayMode === 'none', currentLr stays at lr */
            
            if (optimizerMode === 'adam') {
                /* Adam optimizer updates */
                mx = beta1 * mx + (1 - beta1) * noisyGx;
                my = beta1 * my + (1 - beta1) * noisyGy;

                const useSecondMoment = beta2 < 1 - 1e-12; /* allow beta2 = 1.0 to disable RMS term */
                let denomX = 1.0;
                let denomY = 1.0;

                if (useSecondMoment) {
                    vx = beta2 * vx + (1 - beta2) * noisyGx * noisyGx;
                    vy = beta2 * vy + (1 - beta2) * noisyGy * noisyGy;
                }
                
                /* Bias correction */
                const mxHat = mx / (1 - Math.pow(beta1, i + 1));
                const myHat = my / (1 - Math.pow(beta1, i + 1));
                if (useSecondMoment) {
                    const vxHat = vx / (1 - Math.pow(beta2, i + 1));
                    const vyHat = vy / (1 - Math.pow(beta2, i + 1));
                    denomX = Math.sqrt(vxHat) + epsilon;
                    denomY = Math.sqrt(vyHat) + epsilon;
                } else {
                    denomX = 1.0;
                    denomY = 1.0;
                }
                
                /* Parameter updates */
                x -= currentLr * mxHat / denomX;
                y -= currentLr * myHat / denomY;
            } else {
                /* Classic SGD with momentum (Nesterov not used) */
                mx = beta1 * mx + (1 - beta1) * noisyGx;
                my = beta1 * my + (1 - beta1) * noisyGy;
                x -= currentLr * mx;
                y -= currentLr * my;
            }
            path.push({x, y});
            
            if (Math.sqrt(gx*gx + gy*gy) < 1e-6) break;
        }
        return path;
    }
    
    /* Learning rate scale functions */
    function updateLrFromScale() {
        /* High precision logarithmic scale: 0.001 to 1.0 with 10000 steps */
        const lrValue = Math.pow(10, -3 + (lrScaleSlider.value / 10000) * 3);
        lrScaleValue.textContent = lrValue.toFixed(4);
        drawVisualization();
    }
    
    function setLrScale(lr) {
        const lrLog = Math.log10(lr);
        lrScaleSlider.value = Math.round(((lrLog + 3) / 3) * 10000);
        lrScaleValue.textContent = lr.toFixed(4);
    }
    
    function getLrFromScale() {
        return Math.pow(10, -3 + (lrScaleSlider.value / 10000) * 3);
    }

    /* Drawing functions */
    function drawVisualization() {
        const steps = parseInt(stepsInput.value);
        const lr = getLrFromScale();
        const beta1 = parseFloat(momentumInput.value);
        const beta2 = parseFloat(beta2Input.value);
        const warmupStepsInput = parseInt(warmupInput.value);
        const noiseLevel = parseFloat(noiseInput.value);
        
        /* Adjust world bounds based on objective */
        if (objectiveMode === 'himmel') {
            xMin = -6; xMax = 6; yMin = -6; yMax = 6;
        } else if (objectiveMode === 'camel') {
            xMin = -2; xMax = 2; yMin = -2; yMax = 2;
        } else {
            xMin = -0.4; xMax = 1.8; yMin = -0.6; yMax = 1.8;
        }
        
        /* Clear canvas */
        ctx.fillStyle = '#ffffff';
        ctx.fillRect(0, 0, width, height);
        
        /* Define more contour levels for better granularity */
        const contourLevels = [0.1, 0.3, 0.6, 1.0, 1.8, 3.0, 5.0, 8.0, 14.0, 25.0, 45.0, 80.0];
        
        /* Create contour-based color regions at 1.5x resolution */
        const imageData = ctx.createImageData(width * 1.5, height * 1.5);
        for (let i = 0; i < width * 1.5; i++) {
            for (let j = 0; j < height * 1.5; j++) {
                const [x, y] = toWorld(i / 1.5, j / 1.5);
                const fval = f(x, y, 1, 1);
                
                /* Find which contour level this point belongs to */
                let levelIndex = 0;
                for (let k = 0; k < contourLevels.length; k++) {
                    if (fval <= contourLevels[k]) {
                        levelIndex = k;
                        break;
                    }
                }
                if (fval > contourLevels[contourLevels.length - 1]) {
                    levelIndex = contourLevels.length;
                }
                
                /* Color based on contour level - brighter blue (min) to white (max) */
                const t = levelIndex / contourLevels.length;
                const r = Math.floor(255 * (0.5 + t * 0.5));      /* ~128 to 255 */
                const g = Math.floor(255 * (0.6 + t * 0.4));      /* ~153 to 255 */
                const b_color = Math.floor(255 * (0.85 + t * 0.15)); /* ~217 to 255 */
                
                const idx = (j * width * 1.5 + i) * 4;
                imageData.data[idx] = r;
                imageData.data[idx + 1] = g;
                imageData.data[idx + 2] = b_color;
                imageData.data[idx + 3] = 255;
            }
        }
        ctx.putImageData(imageData, 0, 0);
        
        /* Draw contour lines using the same levels as color regions */
        contourLevels.forEach((level, i) => {
            const alpha = Math.max(0.2, 0.6 - i * 0.03);
            const isImportant = [1.0, 3.0, 14.0].includes(level);
            ctx.strokeStyle = isImportant ? `rgba(75, 85, 99, ${alpha})` : `rgba(107, 114, 128, ${alpha})`;
            ctx.lineWidth = isImportant ? 0.7 : 0.4;
            
            /* Grid-based contour detection */
            const gridSize = 80;
            const stepX = (xMax - xMin) / gridSize;
            const stepY = (yMax - yMin) / gridSize;
            
            ctx.beginPath();
            
            for (let i = 0; i < gridSize - 1; i++) {
                for (let j = 0; j < gridSize - 1; j++) {
                    const x1 = xMin + i * stepX;
                    const y1 = yMin + j * stepY;
                    const x2 = x1 + stepX;
                    const y2 = y1 + stepY;
                    
                    /* Sample the four corners */
                    const val00 = f(x1, y1, 1, 1);
                    const val10 = f(x2, y1, 1, 1);
                    const val01 = f(x1, y2, 1, 1);
                    const val11 = f(x2, y2, 1, 1);
                    
                    /* Check if contour passes through this cell */
                    const vals = [val00, val10, val11, val01];
                    const minVal = Math.min(...vals);
                    const maxVal = Math.max(...vals);
                    
                    if (minVal <= level && level <= maxVal) {
                        /* Find intersection points on edges */
                        const points = [];
                        
                        /* Bottom edge */
                        if ((val00 <= level && level <= val10) || (val10 <= level && level <= val00)) {
                            const t = (level - val00) / (val10 - val00);
                            if (t >= 0 && t <= 1) {
                                points.push([x1 + t * stepX, y1]);
                            }
                        }
                        
                        /* Right edge */
                        if ((val10 <= level && level <= val11) || (val11 <= level && level <= val10)) {
                            const t = (level - val10) / (val11 - val10);
                            if (t >= 0 && t <= 1) {
                                points.push([x2, y1 + t * stepY]);
                            }
                        }
                        
                        /* Top edge */
                        if ((val11 <= level && level <= val01) || (val01 <= level && level <= val11)) {
                            const t = (level - val11) / (val01 - val11);
                            if (t >= 0 && t <= 1) {
                                points.push([x2 - t * stepX, y2]);
                            }
                        }
                        
                        /* Left edge */
                        if ((val01 <= level && level <= val00) || (val00 <= level && level <= val01)) {
                            const t = (level - val01) / (val00 - val01);
                            if (t >= 0 && t <= 1) {
                                points.push([x1, y2 - t * stepY]);
                            }
                        }
                        
                        /* Draw line segments between intersection points */
                        if (points.length >= 2) {
                            const [p1x, p1y] = toCanvas(points[0][0], points[0][1]);
                            const [p2x, p2y] = toCanvas(points[1][0], points[1][1]);
                            
                            ctx.moveTo(p1x, p1y);
                            ctx.lineTo(p2x, p2y);
                        }
                    }
                }
            }
            
            ctx.stroke();
        });
        
        /* Run gradient descent and draw path */
        const noiseSeed = parseInt(noiseSeedInput.value);
        const additiveNoise = parseFloat(noiseInput.value);
        const pctNoise = parseFloat(pctNoiseInput.value);
        const useConvergence = untilConvergeToggle.checked;
        const convergeThreshold = parseFloat(convergeThresholdInput.value);
        const maxSteps = parseInt(maxStepsInput.value);
        const path = gradientDescent(startX, startY, steps, lr, beta1, beta2, warmupStepsInput, additiveNoise, pctNoise, lrDecayMode, noiseSeed, useConvergence, convergeThreshold, maxSteps, optimizerMode);
        
        if (path.length > 1) {
            /* Draw individual line segments with transparency */
            for (let i = 0; i < path.length - 1; i++) {
                const [x1, y1] = toCanvas(path[i].x, path[i].y);
                const [x2, y2] = toCanvas(path[i + 1].x, path[i + 1].y);
                
                /* Color progression in darker pink-orange tones - bright pink to darker orange */
                const t = i / (path.length - 1);
                const r = Math.floor(220 + t * (234 - 220));
                const g = Math.floor(85 + t * (88 - 85));
                const b = Math.floor(100 + t * (42 - 100));
                
                ctx.strokeStyle = `rgba(${r}, ${g}, ${b}, 0.9)`;
                ctx.lineWidth = 1.5;
                ctx.lineCap = 'round';
                
                ctx.beginPath();
                ctx.moveTo(x1, y1);
                ctx.lineTo(x2, y2);
                ctx.stroke();
            }
            
            /* Draw step points with transparency */
            path.forEach((p, i) => {
                const [px, py] = toCanvas(p.x, p.y);
                
                if (i === 0) {
                    /* Start point - darker pink, bigger white stroke */
                    ctx.fillStyle = 'rgba(220, 85, 100, 0.95)';
                    ctx.strokeStyle = 'rgba(255, 255, 255, 0.6)';
                    ctx.lineWidth = 1.2;
                    ctx.beginPath();
                    ctx.arc(px, py, 6, 0, 2*Math.PI);
                    ctx.fill();
                    ctx.stroke();
                } else if (i === path.length - 1) {
                    /* End point - darker orange, bigger white stroke */
                    ctx.fillStyle = 'rgba(234, 88, 42, 0.95)';
                    ctx.strokeStyle = 'rgba(255, 255, 255, 0.6)';
                    ctx.lineWidth = 1.2;
                    ctx.beginPath();
                    ctx.arc(px, py, 5.5, 0, 2*Math.PI);
                    ctx.fill();
                    ctx.stroke();
                } else {
                    /* Every intermediate point - darker gradient colors, bigger white stroke */
                    const t = i / (path.length - 1);
                    const r = Math.floor(220 + t * (234 - 220));
                    const g = Math.floor(85 + t * (88 - 85));
                    const b = Math.floor(100 + t * (42 - 100));
                    
                    ctx.fillStyle = `rgba(${r}, ${g}, ${b}, 0.8)`;
                    ctx.strokeStyle = 'rgba(255, 255, 255, 0.5)';
                    ctx.lineWidth = 0.6;
                    ctx.beginPath();
                    ctx.arc(px, py, 3, 0, 2*Math.PI);
                    ctx.fill();
                    ctx.stroke();
                }
            });
        }
        
        /* Compute and draw minimum point per objective */
        let minPoint;
        if (objectiveMode === 'rosen') {
            /* Inverse transform of Rosenbrock minimum (x'=1,y'=1) */
            const angle = Math.PI / 12; /* 15 degrees */
            const cos_a = Math.cos(angle);
            const sin_a = Math.sin(angle);
            const x_rot_min = 1/3; /* (1 - 0.5)/1.5 */
            const y_rot_min = 1/3;
            const x_min = cos_a * x_rot_min + sin_a * y_rot_min;
            const y_min = -sin_a * x_rot_min + cos_a * y_rot_min;
            minPoint = [x_min, y_min];
        } else if (objectiveMode === 'himmel') {
            /* Use a canonical minimum (3,2) */
            minPoint = [3, 2];
        } else if (objectiveMode === 'camel') {
            /* Three-hump camel minimum at (0,0) */
            minPoint = [0, 0];
        } else {
            minPoint = [0, 0];
        }
        
        const [mx, my] = toCanvas(minPoint[0], minPoint[1]);
        
        /* Outer glow - soft blue */
        ctx.fillStyle = 'rgba(99, 102, 241, 0.2)';
        ctx.beginPath();
        ctx.arc(mx, my, 12, 0, 2*Math.PI);
        ctx.fill();
        
        /* Main point - light blue with 75% opacity */
        ctx.fillStyle = 'rgba(99, 102, 241, 0.75)';
        ctx.strokeStyle = 'rgba(255, 255, 255, 0.75)';
        ctx.lineWidth = 2;
        ctx.beginPath();
        ctx.arc(mx, my, 8, 0, 2*Math.PI);
        ctx.fill();
        ctx.stroke();
        
        /* Center dot */
        ctx.fillStyle = 'rgba(255, 255, 255, 0.75)';
        ctx.beginPath();
        ctx.arc(mx, my, 2, 0, 2*Math.PI);
        ctx.fill();
    }
    
    /* Slider synchronization functions */
    function updateFromSliders() {
        stepsInput.value = stepsSlider.value;
        momentumInput.value = (momentumSlider.value / 1000).toFixed(3);
        beta2Input.value = (beta2Slider.value / 1000).toFixed(3);
        /* keep warmup in steps, clamp max to steps */
        const maxWarm = parseInt(stepsInput.value);
        warmupSlider.max = maxWarm;
        if (parseInt(warmupInput.value) > maxWarm) warmupInput.value = maxWarm;
        if (parseInt(warmupSlider.value) > maxWarm) warmupSlider.value = maxWarm;
        warmupInput.value = warmupSlider.value;
        noiseInput.value = (noiseSlider.value / 100).toFixed(2);
        pctNoiseInput.value = (pctNoiseSlider.value / 100).toFixed(2);
        drawVisualization();
    }
    
    function updateFromInputs() {
        stepsSlider.value = stepsInput.value;
        momentumSlider.value = Math.round(parseFloat(momentumInput.value) * 1000);
        beta2Slider.value = Math.round(parseFloat(beta2Input.value) * 1000);
        const maxWarm = parseInt(stepsInput.value);
        warmupSlider.max = maxWarm;
        warmupInput.max = maxWarm;
        if (parseInt(warmupInput.value) > maxWarm) warmupInput.value = maxWarm;
        warmupSlider.value = warmupInput.value;
        noiseSlider.value = Math.round(parseFloat(noiseInput.value) * 100);
        pctNoiseSlider.value = Math.round(parseFloat(pctNoiseInput.value) * 100);
        drawVisualization();
    }
    
    function setInitialValues() {
        startX = 1.3;
        startY = 1.3;
        stepsInput.value = 200;
        /* Default optimizer SGD with LR 0.03 and Beta2 disabled */
        setLrScale(0.03);
        setOptimizerMode('sgd');
        momentumInput.value = 0.9;
        beta2Input.value = 0.999;
        warmupInput.value = 0;
        noiseInput.value = 0;
        pctNoiseInput.value = 0.2;
        noiseSeedInput.value = 42;
        convergeThresholdInput.value = 0.0015;
        maxStepsInput.value = 300;
        untilConvergeToggle.checked = false;
        toggleStepsControls();
        updateFromInputs();
    }
    
    /* Function to toggle steps control state */
    function toggleStepsControls() {
        const stepsControlGroup = document.getElementById('steps-control-group');
        const isConvergeMode = untilConvergeToggle.checked;
        
        if (isConvergeMode) {
            stepsControlGroup.classList.add('disabled');
        } else {
            stepsControlGroup.classList.remove('disabled');
        }
        
        drawVisualization();
    }
    
    /* Event listeners */
    [stepsSlider, momentumSlider, beta2Slider, warmupSlider, noiseSlider, pctNoiseSlider].forEach(s => 
        s.addEventListener('input', updateFromSliders)
    );
    [stepsInput, momentumInput, beta2Input, warmupInput, noiseInput, pctNoiseInput, noiseSeedInput, convergeThresholdInput, maxStepsInput].forEach(i => 
        i.addEventListener('change', updateFromInputs)
    );
    
    /* Learning rate scale listener */
    lrScaleSlider.addEventListener('input', updateLrFromScale);
    
    /* Convergence toggle listener */
    untilConvergeToggle.addEventListener('change', toggleStepsControls);
    
    resetButton.addEventListener('click', setInitialValues);
    
    /* LR Decay button handling */
    function setDecayMode(mode) {
        lrDecayMode = mode;
        [decayNoneBtn, decayLinearBtn, decayCosineBtn].forEach(btn => btn.classList.remove('active'));
        
        if (mode === 'none') decayNoneBtn.classList.add('active');
        else if (mode === 'linear') decayLinearBtn.classList.add('active');
        else if (mode === 'cosine') decayCosineBtn.classList.add('active');
        
        drawVisualization();
    }
    
    decayNoneBtn.addEventListener('click', () => setDecayMode('none'));
    decayLinearBtn.addEventListener('click', () => setDecayMode('linear'));
    decayCosineBtn.addEventListener('click', () => setDecayMode('cosine'));

    /* Optimizer toggle */
    function setOptimizerMode(mode) {
        optimizerMode = mode;
        [optSgdBtn, optAdamBtn].forEach(btn => btn.classList.remove('active'));
        if (mode === 'sgd') {
            optSgdBtn.classList.add('active');
            /* Set LR to 0.03 when switching to SGD */
            setLrScale(0.03);
        } else {
            optAdamBtn.classList.add('active');
            /* Set LR to 0.3 when switching to Adam */
            setLrScale(0.3);
        }

        /* Enable/disable Beta2 controls when switching optimizers */
        const beta2Group = document.getElementById('beta2-control-group');
        const beta2Num = document.getElementById('beta2-input');
        const beta2Sld = document.getElementById('beta2-slider');
        const isAdam = mode === 'adam';
        if (!isAdam) {
            beta2Group.classList.add('disabled');
            beta2Num.disabled = true;
            beta2Sld.disabled = true;
        } else {
            beta2Group.classList.remove('disabled');
            beta2Num.disabled = false;
            beta2Sld.disabled = false;
        }
        drawVisualization();
    }
    optSgdBtn.addEventListener('click', () => setOptimizerMode('sgd'));
    optAdamBtn.addEventListener('click', () => setOptimizerMode('adam'));

    /* Objective toggle */
    function setObjectiveMode(mode) {
        objectiveMode = mode;
        [objRosenBtn, objHimmelBtn, objCamelBtn].forEach(btn => btn.classList.remove('active'));
        if (mode === 'rosen') objRosenBtn.classList.add('active');
        else if (mode === 'himmel') objHimmelBtn.classList.add('active');
        else if (mode === 'camel') objCamelBtn.classList.add('active');
        drawVisualization();
    }
    objRosenBtn.addEventListener('click', () => setObjectiveMode('rosen'));
    objHimmelBtn.addEventListener('click', () => setObjectiveMode('himmel'));
    objCamelBtn.addEventListener('click', () => setObjectiveMode('camel'));
    
    /* Debug section toggle */
    debugToggle.addEventListener('click', function() {
        const isVisible = debugSection.style.display !== 'none';
        debugSection.style.display = isVisible ? 'none' : 'block';
        debugToggle.innerHTML = isVisible ? 'Debug ▼' : 'Debug ▲';
    });
    
    /* Live click and drag functionality */
    let isDragging = false;
    
    function updateStartPosition(e) {
        const rect = canvas.getBoundingClientRect();
        const [wx, wy] = toWorld(e.clientX - rect.left, e.clientY - rect.top);
        
        startX = Math.max(xMin, Math.min(xMax, wx));
        startY = Math.max(yMin, Math.min(yMax, wy));
        
        drawVisualization();
    }
    
        canvas.addEventListener('mousedown', function(e) {
        isDragging = true;
        updateStartPosition(e);
        canvas.style.cursor = 'grabbing';
    });

    canvas.addEventListener('mousemove', function(e) {
        if (isDragging) {
            updateStartPosition(e);
        }
    });

    canvas.addEventListener('mouseup', function() {
        isDragging = false;
        canvas.style.cursor = 'grab';
    });

    canvas.addEventListener('mouseleave', function() {
        isDragging = false;
        canvas.style.cursor = 'grab';
    });
    
    /* Also handle single clicks */
    canvas.addEventListener('click', function(e) {
        if (!isDragging) {
            updateStartPosition(e);
        }
    });
    
    /* Tooltip functionality with toggle control */
    const tooltip = document.getElementById('gradient-tooltip');
    const tooltipToggle = document.getElementById('tooltip-toggle');
    
    canvas.addEventListener("mousemove", (e) => {
        if (!tooltipToggle.checked) {
            tooltip.style.display = "none";
            return;
        }
        
        const rect = canvas.getBoundingClientRect();
        const [wx, wy] = toWorld(e.clientX - rect.left, e.clientY - rect.top);
        const val = f(wx, wy, 1, 1);
        
        tooltip.style.left = `${e.clientX + 10}px`;
        tooltip.style.top = `${e.clientY + 10}px`;
        tooltip.style.display = "block";
        tooltip.textContent = `(${wx.toFixed(2)}, ${wy.toFixed(2)}): f=${val.toFixed(3)} | Start: (${startX.toFixed(1)}, ${startY.toFixed(1)})`;
    });
    
    canvas.addEventListener("mouseleave", () => tooltip.style.display = "none");
    
    /* Hide tooltip when toggle is unchecked */
    tooltipToggle.addEventListener("change", () => {
        if (!tooltipToggle.checked) {
            tooltip.style.display = "none";
        }
    });
    
    /* Initialize */
    setInitialValues();
});
</script>


