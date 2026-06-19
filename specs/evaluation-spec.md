# Evaluation Spec — Pod Classifier

Complete this spec **before** writing any code for Milestone 3.

Use Plan or Ask mode to think through each blank field. When you're done,
your answers here become the blueprint for `compute_accuracy()` and
`compute_per_class_accuracy()` in `evaluate.py`.

---

## Background: What is evaluation?

After building a classifier, we need to know how well it works. Evaluation answers:
- **Overall:** What fraction of episodes did we classify correctly?
- **Per-class:** Are we better at some labels than others?

Both functions take the same inputs: a list of predicted labels and a list of
ground-truth labels, in the same order.

---

## compute_accuracy(predictions, ground_truth)

### What it does
Returns the fraction of predictions that exactly match the ground truth.

### Inputs

| Parameter | Type | Description |
|---|---|---|
| `predictions` | `list[str]` | Labels predicted by `classify_episode()`, one per episode. |
| `ground_truth` | `list[str]` | The correct labels, in the same order as `predictions`. |

### Output

| Return value | Type | Description |
|---|---|---|
| accuracy | `float` | A value between 0.0 and 1.0. |

---

### Spec fields — fill these in before writing code

**Formula:**

```
A prediction is "correct" when it exactly matches the ground-truth label at
the same position (e.g. predictions[i] == ground_truth[i], "interview" ==
"interview"). Accuracy is the number of correct predictions divided by the
total number of predictions.

    accuracy = (# positions where prediction == ground_truth) / (total predictions)
```

---

**Step-by-step logic:**

```
 1. If predictions is empty, return 0.0 (avoid dividing by zero).
 2. Pair up predictions and ground_truth position-by-position (zip).
 3. Count the pairs where predicted == truth.
 4. Return that count divided by len(predictions), as a float.
```

---

**Edge case — what if both lists are empty?**

```
Return 0.0. With no predictions there are zero correct answers, and the
ratio 0/0 is undefined — so 0.0 is the safe convention. It also avoids a
ZeroDivisionError and stays consistent with the "0.0 when total is 0" rule
used in per-class accuracy below.
```

---

**Worked example:**

```
predictions  = ["interview", "solo", "panel", "interview"]
ground_truth = ["interview", "solo", "solo",  "narrative"]

Compare position by position:
  interview == interview   ✓
  solo      == solo        ✓
  panel     != solo        ✗
  interview != narrative   ✗
2 correct out of 4  →  2 / 4 = 0.5
```

---

## compute_per_class_accuracy(predictions, ground_truth)

### What it does
Returns accuracy broken down by each label. For each label in `VALID_LABELS`,
reports how many episodes with that ground-truth label were classified correctly.

### Inputs

| Parameter | Type | Description |
|---|---|---|
| `predictions` | `list[str]` | Labels predicted by `classify_episode()`. |
| `ground_truth` | `list[str]` | Correct labels, in the same order. |

### Output

A `dict` keyed by label. Each value is a dict with three keys:

```python
{
    "interview": {"correct": int, "total": int, "accuracy": float},
    "solo":      {"correct": int, "total": int, "accuracy": float},
    "panel":     {"correct": int, "total": int, "accuracy": float},
    "narrative": {"correct": int, "total": int, "accuracy": float},
}
```

---

### Spec fields — fill these in before writing code

**What does "correct" mean for a given class?**

```
For class C, a prediction is correct when the ground truth is C AND the
prediction is also C. Example for "interview": the episode counts as correct
for the interview class only when ground_truth == "interview" and
predicted == "interview".
```

---

**What does "total" mean for a given class?**

```
"total" is the number of episodes whose GROUND TRUTH equals that class — not
the total number of predictions, and not the number of times the class was
predicted. (This makes per-class accuracy a recall-style metric: of all the
true interviews, how many did we classify correctly?)
```

---

**Step-by-step logic:**

```
 1. Initialize a result dict: for every label in VALID_LABELS, set
    {"correct": 0, "total": 0, "accuracy": 0.0}.
 2. Loop over the (predicted, truth) pairs with zip(predictions, ground_truth).
 3. For each pair: increment result[truth]["total"]; if predicted == truth,
    also increment result[truth]["correct"].
 4. After the loop, for each label set "accuracy" = correct / total, or 0.0
    if total == 0.
 5. Return the result dict.
```

---

**Edge case — what if a class has no examples in ground_truth (total == 0)?**

```
Set accuracy to 0.0. There are no episodes of that class to score, so the
ratio is undefined; 0.0 avoids a ZeroDivisionError and matches the docstring
in evaluate.py ("accuracy" : correct / total (0.0 if total is 0)).
```

---

**Worked example:**

```
predictions  = ["interview", "interview", "solo", "panel", "panel"]
ground_truth = ["interview", "solo",      "solo", "panel", "narrative"]

label       correct  total  accuracy
----------  -------  -----  --------
interview      1       1      1.0
solo           1       2      0.5
panel          1       1      1.0
narrative      0       1      0.0

Work:
- interview: truth at position 0 only (total 1); predicted interview there → correct 1 → 1.0
- solo:      truth at positions 1 and 2 (total 2); position 1 predicted interview ✗,
             position 2 predicted solo ✓ → correct 1 → 0.5
- panel:     truth at position 3 (total 1); predicted panel ✓ → correct 1 → 1.0
- narrative: truth at position 4 (total 1); predicted panel ✗ → correct 0 → 0.0
```

---

## Reflection questions (discuss at the checkpoint)

1. Your overall accuracy might be decent even if one class has very low accuracy.
   Why is per-class accuracy a more informative metric than overall accuracy alone?

2. If `panel` episodes consistently get misclassified as `interview`, what does
   that tell you about your training labels or your prompt?

3. You labeled 20 training episodes and evaluated on 20 test episodes (5 per class).
   How might the evaluation results change if you had labeled 100 training episodes?
   What if you had 200 test episodes?
