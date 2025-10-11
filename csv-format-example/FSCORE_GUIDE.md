# F-Score Implementation Guide

This guide explains how to implement F-score tracking in CSV format tests for classification tasks.

## What is F-Score?

F-score (F1 score) measures classification accuracy by balancing:
- **Precision**: "Of all the things I said were positive, how many actually were?" (avoiding false alarms)
- **Recall**: "Of all the things that were positive, how many did I catch?" (avoiding missed cases)
- **F1 Score**: Harmonic mean that balances both - high only when BOTH are good

## The Four Key Metrics

To calculate F-score, you need to track:

1. **True Positives (TP)**: Model says "positive" AND actual is "positive" ✅
2. **False Positives (FP)**: Model says "positive" BUT actual is "negative" ❌
3. **False Negatives (FN)**: Model says "negative" BUT actual is "positive" ❌
4. **True Negatives (TN)**: Model says "negative" AND actual is "negative" ✅

## CSV Implementation

### Step 1: Structure Your CSV

Your CSV needs:
- Input variables (e.g., `text`)
- Expected output (e.g., `sentiment`)
- Multiple `__expected` columns for tracking metrics

```csv
text,sentiment,__expected1,__expected2,__expected3,__expected4
"This is great!","positive","javascript: ...","javascript: ...","javascript: ...","javascript: ..."
```

### Step 2: Define Assertions in CSV

Each `__expected` column tracks a specific metric:

**__expected1: Accuracy**
```javascript
output.sentiment === context.vars.sentiment
```

**__expected2: True Positives**
```javascript
output.sentiment === 'positive' && context.vars.sentiment === 'positive' ? 1 : 0
```

**__expected3: False Positives**
```javascript
output.sentiment === 'positive' && context.vars.sentiment === 'negative' ? 1 : 0
```

**__expected4: False Negatives**
```javascript
output.sentiment === 'negative' && context.vars.sentiment === 'positive' ? 1 : 0
```

### Step 3: Configure Default Test (Alternative Approach)

Instead of putting assertions in CSV, you can use `defaultTest` in your config:

```yaml
defaultTest:
  assert:
    - type: javascript
      value: output.sentiment === context.vars.sentiment
      metric: accuracy

    - type: javascript
      value: "output.sentiment === 'positive' && context.vars.sentiment === 'positive' ? 1 : 0"
      metric: true_positives
      weight: 0  # Don't affect pass/fail

    - type: javascript
      value: "output.sentiment === 'positive' && context.vars.sentiment === 'negative' ? 1 : 0"
      metric: false_positives
      weight: 0

    - type: javascript
      value: "output.sentiment === 'negative' && context.vars.sentiment === 'positive' ? 1 : 0"
      metric: false_negatives
      weight: 0
```

**Benefits of defaultTest approach:**
- Cleaner CSV (just data, no complex JavaScript)
- Easier to maintain
- Reusable across multiple CSV files

### Step 4: Define Derived Metrics

In your `promptfooconfig.yaml`, add:

```yaml
derivedMetrics:
  # Precision = TP / (TP + FP)
  - name: precision
    value: true_positives / (true_positives + false_positives)

  # Recall = TP / (TP + FN)
  - name: recall
    value: true_positives / (true_positives + false_negatives)

  # F1 Score = 2 * (precision * recall) / (precision + recall)
  - name: f1_score
    value: 2 * true_positives / (2 * true_positives + false_positives + false_negatives)
```

## Complete Example

### CSV File (`fscore_simple.csv`)
```csv
text,sentiment,__description
"This product is amazing!","positive","Positive: enthusiastic review"
"Terrible experience.","negative","Negative: strong disappointment"
"Best purchase ever!","positive","Positive: strong recommendation"
```

### Config File (`promptfooconfig_fscore.yaml`)
```yaml
description: "F-Score Classification Example"

prompts:
  - file://prompts/sentiment_prompt.txt

providers:
  - id: openai:gpt-4o-mini
    config:
      temperature: 0
      response_format:
        type: json_object

tests: file://tests/fscore_simple.csv

derivedMetrics:
  - name: precision
    value: true_positives / (true_positives + false_positives)
  - name: recall
    value: true_positives / (true_positives + false_negatives)
  - name: f1_score
    value: 2 * true_positives / (2 * true_positives + false_positives + false_negatives)

defaultTest:
  assert:
    - type: javascript
      value: output.sentiment === context.vars.sentiment
      metric: accuracy
    - type: javascript
      value: "output.sentiment === 'positive' && context.vars.sentiment === 'positive' ? 1 : 0"
      metric: true_positives
      weight: 0
    - type: javascript
      value: "output.sentiment === 'positive' && context.vars.sentiment === 'negative' ? 1 : 0"
      metric: false_positives
      weight: 0
    - type: javascript
      value: "output.sentiment === 'negative' && context.vars.sentiment === 'positive' ? 1 : 0"
      metric: false_negatives
      weight: 0
```

## Understanding the Results

After running `promptfoo eval -c promptfooconfig_fscore.yaml`, you'll see:

- **Accuracy**: Overall correctness percentage
- **Precision**: How reliable your positive predictions are
- **Recall**: How well you catch all positive cases
- **F1 Score**: Balanced measure (0 to 1, higher is better)

### Interpreting Scores

- **High Precision, Low Recall**: Conservative - only flags things when very sure, but misses many
- **Low Precision, High Recall**: Aggressive - catches most cases, but many false alarms
- **High F1 Score**: Good balance between precision and recall

## When to Use F-Score

✅ **Use F-score when:**
- Doing classification tasks (sentiment, intent, categories)
- Working with imbalanced datasets
- False positives and false negatives have different costs
- You need both accuracy AND coverage

❌ **Don't use F-score when:**
- Not doing classification
- Simple pass/fail is sufficient
- You only care about one metric (precision OR recall)

## Tips

1. **Set weight: 0** for metric-tracking assertions so they don't affect pass/fail
2. **Use defaultTest** instead of CSV assertions for cleaner, more maintainable code
3. **Test with edge cases** including neutral/ambiguous examples
4. **Monitor all metrics** not just F1 - sometimes precision or recall alone is more important
5. **Use temperature: 0** for consistent classification results

## References

- [Promptfoo F-Score Documentation](https://www.promptfoo.dev/docs/configuration/expected-outputs/deterministic/#f-score)
- [Wikipedia: F-score](https://en.wikipedia.org/wiki/F-score)
- [Understanding Precision and Recall](https://en.wikipedia.org/wiki/Precision_and_recall)
