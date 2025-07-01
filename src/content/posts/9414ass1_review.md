---
title: 9414ass1_review
date: 2025-07-01
category: 'unsw'
---

# learning rate

In deep learning (and other iterative optimization algorithms), the **learning rate** controls how big a step you take on each parameter update. Choosing it poorly leads to:

---

### 1. If the learning rate is **too large**

- **Divergence**发散: You overshoot the minimum and the loss blows up instead of going down.
- **Oscillation**: Parameters jump back and forth around the optimum, so the loss keeps bouncing and never settles.

### 2. If the learning rate is **too small**

- **Very slow convergence**收敛速度慢: Each update barely moves the parameters—training takes a very long time.
- **Getting stuck in local minima** 陷入局部最优: Small steps can’t escape shallow “valleys” in the loss landscape.
- **Apparent stagnation** 早期停滞: Early on, the loss hardly decreases, giving the false impression that the model isn’t learning.

---

## How to find a good learning rate

1. **Warm-up**
   Start with a small learning rate for a few epochs, then increase it to your target. This prevents large, unstable updates at the very beginning.
2. **Decay schedules**
   Gradually reduce the learning rate as training progresses (e.g. step decay, exponential decay, cosine annealing) so you can fine-tune near the optimum.
3. **Adaptive optimizers**
   Use methods like Adam, RMSProp, or AdaGrad, which adjust each parameter’s step size automatically based on past gradients—making your choice of base learning rate more forgiving.
4. **Learning-rate search**
   Run a short experiment that cycles through values (e.g. 1e-4 → 1e-3 → 1e-2 → 1e-1) and plot loss vs. learning rate. Pick the value where loss decreases fastest without diverging.

---

**Bottom line**:

- **Too large** → unstable or divergent training
- **Too small** → painfully slow and prone to poor local optima
- Use warm-up, decay, adaptive optimizers, or systematic search to zero in on the “sweet spot.”

# Hyper parameter settings

Below is an English translation of the rationale for choosing these hyperparameters:

1. **`learning_rate=0.001` (initial learning rate for Adam)**
   - **Adam’s default** recommendation is 1e–3, which tends to give stable convergence on most tasks.
   - This value is large enough to make meaningful progress each step, yet small enough to avoid divergence.
   - If training is unstable or too slow at first, you can use a warm-up phase, a decay scheduler, or fine-tune within the range 1e–4 to 1e–2.
2. **`loss_fn=BinaryCrossentropy()`**
   - This is the most common loss for binary classification, equivalent to maximizing the Bernoulli log-likelihood for each label.
   - Compared to mean squared error (MSE), binary cross-entropy penalizes misclassifications more strongly, which accelerates learning of the decision boundary.
3. **`batch_size=16`**
   - Smaller mini-batches introduce more “noise” into the gradient estimate, which can improve generalization.
   - They also consume less memory, making them suitable for mid-range GPUs or CPUs.
   - To speed up convergence or better utilize hardware, you might try larger sizes like 32 or 64—just remember to adjust your learning rate and any batch-norm settings accordingly.
4. **`epochs=500`**
   - Provides the model with plenty of passes over the data to fully learn its patterns.
   - It’s a good idea to combine this with an **EarlyStopping** callback (e.g. monitoring validation loss with `patience=20`) to automatically halt training and avoid overfitting.

---

**How to further improve?**

- **Learning-rate scheduling**: Use cosine annealing, step decay, or ReduceLROnPlateau to fine-tune the step size later in training.
- **Batch-size experiments**: Apply the linear scaling rule (scale learning rate by batch size) with a warm-up period when increasing batch size.
- **Metric monitoring**: In addition to loss, track accuracy, ROC-AUC, etc., and adjust hyperparameters based on those signals.

This setup generally balances speed and accuracy for a binary classifier, and you can further refine it based on your validation results.

# nn design

1. **Input layer `Input(shape=(len(input_features),))`**
   - Its dimensionality exactly matches the number of input features, ensuring every feature is received and used by the model.
   - Explicitly declaring the input shape lets Keras automatically infer the correct weight matrix sizes.
2. **Hidden layer `Dense(5, activation='relu')`**
   - **Five neurons** gives the model enough capacity to **learn nonlinear relationships** between features without being so large that it overfits easily.
   - **ReLU activation**:
     - Very cheap to compute (just a max(0, x)), which speeds up training.
     - Mitigates the **vanishing-gradient problem** that plagues sigmoid/tanh, keeping deeper networks stable during forward/backward passes.
3. **Output layer `Dense(1, activation='sigmoid')`**
   - **One neuron** produces the single output needed for binary classification: the probability of the positive class.
   - **Sigmoid activation** squashes any real-valued input into the $(0,1)$ range, directly interpretable as a probability and a natural match for the `BinaryCrossentropy` loss.
4. **Overall architecture: shallow, compact, easy to train**
   - With just one hidden layer, capacity is moderate. On small-to-medium datasets, this **shallow network** can learn effective decision boundaries while training quickly.
   - If performance later proves insufficient, you can deepen (add more layers), widen (add more neurons), or introduce Dropout/regularization to control overfitting.

---

**Summary**

> This model uses the simplest “Input → small ReLU hidden layer → Sigmoid output” pattern. It can capture nonlinearity while remaining interpretable and efficient—ideal for entry-level or smaller-scale binary-classification tasks.

# why we fit a separate scaler on the training‐set targets

Here’s an English explanation of why we fit a separate scaler on the training‐set targets and then apply it to the validation and test targets:

1. **Faster, more stable training.**
   When the target values y span a wide range, gradient‐based optimizers (especially in neural networks) can struggle with very large or very small gradients. Scaling $y$ (for example to zero mean and unit variance) makes the loss landscape smoother and helps the model converge faster and more reliably.
2. **Consistent feature–target scaling.**
   If your input features $X$ are normalized but the targets remain on a very different scale, the model may “overemphasize” errors on large targets (because loss functions like MSE weight large absolute errors more heavily). Bringing both inputs and outputs to comparable scales leads to more balanced learning.
3. **Numerical stability.**
   Extremely large or tiny values can cause floating‐point overflow/underflow or exacerbate gradient explosion/vanishing. Scaling $y$ keeps all computations within a safe numeric range.
4. **Avoiding data leakage.**
   By fitting the scaler only on the **training** targets, we ensure that no information from the validation or test sets “leaks” into the training process. We then **apply** (but do not re‐fit) that same transformation to validation and test $y$, so that all evaluations use the exact same scale without ever peeking at their true distributions.
