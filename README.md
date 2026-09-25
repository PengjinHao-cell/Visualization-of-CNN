# Visualization of CNN

### LeNet · Neural Network Panorama

**See signals move through the entire network — then inspect the computation behind one neuron.**

[简体中文](README.zh-CN.md)

An interactive, browser-based CNN visualization for exploring handwritten digit classification. Follow a digit through convolution, pooling, fully connected layers, and classification, while keeping the whole network in view.

The English-language application is a **single self-contained HTML file**. Model weights, example digits, styles, and JavaScript are embedded; inference and activation-gradient computation run locally in the browser. No backend, build step, GPU, or external JavaScript library is required.

> This is a **LeNet-style teaching variant with ReLU and max pooling**, not an exact reproduction of the original LeNet-5. Backpropagation computes and displays gradients; it does **not** train the model or update its weights.

## Quick start

Download and extract the project, then open **`index.html`** in a browser with JavaScript enabled.

Alternatively, serve the project directory using Python 3:

```bash
cd Visualization-of-CNN
python -m http.server 8000 --bind 127.0.0.1
```

Open `http://127.0.0.1:8000`. Python is only needed for this optional local server, not for inference.

A desktop-sized window makes the network easier to inspect. On smaller screens, the local computation panel can be scrolled horizontally. Fullscreen support depends on the browser.

## Features

- **Network panorama:** animate forward propagation, pause, and advance one layer at a time.
- **Neuron and feature-map views:** switch between `Neuron Network` and `Feature Maps`, adjust connection density, pan, and zoom.
- **Convolution inspector:** inspect a 5 × 5 input patch, kernel weights, elementwise products, channel-wise sums, bias, and ReLU output. Scan across positions while the shared kernel stays fixed.
- **Pooling and fully connected layers:** inspect a 2 × 2 maximum or select a fully connected neuron to examine its computation.
- **Backward gradients:** choose a target label and visualize cross-entropy gradients propagating toward earlier activations and input pixels.
- **Digit inputs:** explore 40 embedded samples, four per digit class, or draw a digit on the handwriting pad.

## Suggested walkthrough

1. Select a digit, turn off `Auto sample`, and use `Next Layer →` to follow the forward pass.
2. Select `C1 · First Convolution`. Use `Next Position` or `Scan Convolution` to see how a shared kernel produces a feature map.
3. Select `C3 · Second Convolution` and change `Input channel`. One output combines all six input channels: 6 × 5 × 5 = 150 products, followed by one bias and ReLU.
4. Inspect S2 and S4 for pooling, then F5, F6, and OUT for classification.
5. Set `True label` and select `← Backpropagate Error`. Compare gradients for different target labels: the prediction itself does not change.

## Controls

| Control | Action |
| --- | --- |
| `Input Digit` / `Another Sample` | Select a class or cycle through its built-in examples. |
| `✎ Draw` | Draw a digit and run it through the network. |
| `Neuron Network` / `Feature Maps` | Change the main visualization mode. |
| `Play` / `Pause` | Resume or pause the propagation animation. |
| `↺ Restart Forward` | Restart the forward animation. |
| `Next Layer →` | Advance one layer in the current propagation direction. |
| `← Backpropagate Error` | Start the backward-gradient animation. |
| `True label` | Change the target used for loss and gradients, not the prediction. |
| Layer / channel / neuron selectors | Choose the unit to inspect. C3 also offers input-channel selection. |
| `Scan Convolution` / `Next Position` | Scan or step through spatial positions in convolution or pooling. |
| Click the inspector's output map | Select an output position. |
| Drag / scroll / `−` / `＋` / `Fit` | Pan, zoom, or reset the panorama. |
| `Space` / `→` | Toggle playback / advance a layer when not editing a form control or using the drawing dialog. |

## Network architecture

Spatial shapes use **channels × height × width**. Convolutions use 5 × 5 kernels, stride 1, and no convolution padding. Max pooling uses 2 × 2 windows with stride 2.

| Stage | Operation | Output |
| --- | --- | --- |
| INPUT | Grayscale input | 1 × 32 × 32 |
| C1 | Convolution 1 → 6 + ReLU | 6 × 28 × 28 |
| S2 | Max pooling | 6 × 14 × 14 |
| C3 | Convolution 6 → 16 + ReLU | 16 × 10 × 10 |
| S4 | Max pooling | 16 × 5 × 5 |
| Flatten | Flatten S4 | 400 |
| F5 | Fully connected 400 → 120 + ReLU | 120 |
| F6 | Fully connected 120 → 84 + ReLU | 84 |
| OUTPUT | Fully connected 84 → 10; softmax for probabilities | 10 classes |

The embedded weight and bias arrays contain **61,706 parameters**.

## Data and embedded model metadata

The application identifies its dataset as **scikit-learn `load_digits` / UCI optical handwritten digits**, **not MNIST**. Built-in 8 × 8 samples are normalized from the 0–16 intensity range, bilinearly enlarged to 28 × 28, and zero-padded to 32 × 32. Drawings follow a separate preprocessing path: crop the visible strokes, resize, center on a 32 × 32 canvas, and normalize.

These values are **reported by the embedded `modelData.meta` object**, not independently reproduced training results:

| Metadata | Reported value |
| --- | --- |
| Training examples | 1,437 |
| Test examples | 360 |
| Random seed | 42 |
| Training epochs | 35 |
| Test accuracy | 98.89% |
| Packaged demonstration samples | 40, four per class |

The page states that the demonstration samples are the first four test examples from each class, not samples selected for correct predictions. The training script, complete split indices, and full test set are not included, so this repository is not a complete reproduction package for the reported training result. That accuracy is not a guarantee for arbitrary handwriting.

## Reading the visualization

**Blue** represents forward activations, **red** represents gradient magnitude, and **gold** highlights the selected local receptive field.

Nodes and edges are visually sampled for readability, while numerical forward and backward calculations use the full network. Brightness is normalized within each layer, so it should not be compared directly across layers. Displayed matrix values are rounded; calculations use the underlying values.

Backpropagation starts from the softmax cross-entropy gradient `p − one_hot(target)` at the output logits and propagates to earlier activations and the input. Red does not represent a weight update, a signed gradient, or an image reconstruction. Animation speed is a presentation choice, not a measurement of inference time.

## Source navigation

All application code and data are in `index.html`:

| Area | Responsibility |
| --- | --- |
| Page markup and `<style>` | Layout, toolbars, canvases, and handwriting dialog. |
| `modelData` JSON | Base64-encoded float32 weights, digit samples, and metadata. |
| `NN.load` / `NN.prep` | Decode weights and prepare embedded inputs. |
| `NN.forward` / `NN.backward` | Compute activations, probabilities, loss, and activation gradients. |
| `drawNetwork` / `drawLocalConnections` | Render the panorama and local connections. |
| `updateDetail` / `drawMicro` | Render the selected unit's local calculation. |
| `window.__lab` | Local inspection and state-control hooks for testing or recording. |

The application contains no code to upload drawings, fetch a remote model, or send analytics. Its external links are reference links. A static hosting provider can still receive ordinary page requests; local inference is not a claim about that provider's logs.

## Project structure

```text
Visualization-of-CNN/
├── index.html          # Complete English-language application
├── README.md           # English documentation
├── README.zh-CN.md     # Chinese documentation
└── .gitignore
```

`index.html` preserves the supplied application's contents; only its entry-point filename has changed.

## References included in the application

- [PyTorch: Neural Networks tutorial](https://docs.pytorch.org/tutorials/beginner/blitz/neural_networks_tutorial.html)
- [scikit-learn: load_digits](https://scikit-learn.org/stable/modules/generated/sklearn.datasets.load_digits.html)

These references are not runtime dependencies. This README does not imply that an online deployment or cross-browser certification has been completed.
