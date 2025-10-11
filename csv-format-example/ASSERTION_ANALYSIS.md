# Why Assertions "Fail" and Do We Need All of Them?

## The Problem: Duplicate Assertions

Looking at your configuration, you have **TWO SETS** of the same assertions:

### Set 1: In the CSV file (`fscore_simple.csv`)
```csv
__expected1, __expected2, __expected3, __expected4
```

### Set 2: In the config file (`promptfooconfig_fscore.yaml`)
```yaml
defaultTest:
  assert:
    - assertion 1
    - assertion 2
    - assertion 3
    - assertion 4
```

**This means EVERY test runs 8 assertions total (4 from CSV + 4 from defaultTest)!**

---

## Why They "Fail" (But Actually Don't)

### Understanding the Assertions

For EACH test, only ONE assertion should return `1` (true), the others return `0`:

**Example: Positive sentiment test**
```
Text: "This is amazing!"
Expected: positive
Output: positive
```

**Results:**
- ✅ Assertion 1 (Accuracy): `true` - PASSES
- ✅ Assertion 2 (True Positive): `1` - Returns 1 (this IS a true positive)
- ✅ Assertion 3 (False Positive): `0` - Returns 0 (this is NOT a false positive)
- ✅ Assertion 4 (False Negative): `0` - Returns 0 (this is NOT a false negative)

**The "failures" are actually correct!** They're supposed to return 0 when that condition doesn't apply.

---

## The Confusion: Pass vs. Metric Tracking

### Assertions 2, 3, 4 are NOT pass/fail checks!

They're **counters** that track metrics:

```yaml
- type: javascript
  value: "output.sentiment === 'positive' && context.vars.sentiment === 'positive' ? 1 : 0"
  metric: true_positives
  weight: 0  # ← This means "don't affect pass/fail"
```

**`weight: 0`** means:
- ❌ NOT a pass/fail test
- ✅ JUST counting for statistics
- The test can still PASS even if this returns 0

---

## Do We Need All These Assertions?

### Short Answer: **NO, you're duplicating them!**

### Two Options:

### **Option 1: Use CSV Assertions Only** (Remove defaultTest)

**Pros:**
- Everything in one place (CSV)
- Can see assertions per test

**Cons:**
- Repetitive (same assertions in every row)
- Hard to read CSV
- Hard to maintain

**When to use:** Small test sets where you want everything visible in the CSV

---

### **Option 2: Use defaultTest Only** (Clean CSV) ⭐ **RECOMMENDED**

**Pros:**
- Clean, readable CSV (just test data)
- Define assertions once
- Easy to maintain
- Can reuse across multiple CSV files

**Cons:**
- Assertions separated from test data

**When to use:** Larger test sets, production use

---

## The Right Way: Clean CSV + defaultTest

### Step 1: Simplify the CSV

Remove all the `__expected` columns:

```csv
text,sentiment,__description
"This product is amazing!","positive","Positive: enthusiastic review"
"Terrible experience.","negative","Negative: strong disappointment"
```

**That's it!** Just your test data.

### Step 2: Keep defaultTest in Config

The config already has the assertions with `weight: 0`:

```yaml
defaultTest:
  assert:
    - type: javascript
      value: output.sentiment === context.vars.sentiment
      metric: accuracy
      # This one affects pass/fail (no weight specified = weight: 1)

    - type: javascript
      value: "output.sentiment === 'positive' && context.vars.sentiment === 'positive' ? 1 : 0"
      metric: true_positives
      weight: 0  # Just counting, doesn't affect pass/fail

    - type: javascript
      value: "output.sentiment === 'positive' && context.vars.sentiment === 'negative' ? 1 : 0"
      metric: false_positives
      weight: 0  # Just counting

    - type: javascript
      value: "output.sentiment === 'negative' && context.vars.sentiment === 'positive' ? 1 : 0"
      metric: false_negatives
      weight: 0  # Just counting
```

---

## What Each Assertion Does

### Assertion 1: Accuracy (The Only "Real" Test)
```javascript
output.sentiment === context.vars.sentiment
```
- **Purpose:** Check if the answer is correct
- **Weight:** 1 (default) - affects pass/fail
- **Returns:** true or false
- **This is the ONLY one that determines if the test passes!**

### Assertions 2-4: Metric Counters (Not Tests)
```javascript
// True Positives
output.sentiment === 'positive' && context.vars.sentiment === 'positive' ? 1 : 0

// False Positives  
output.sentiment === 'positive' && context.vars.sentiment === 'negative' ? 1 : 0

// False Negatives
output.sentiment === 'negative' && context.vars.sentiment === 'positive' ? 1 : 0
```
- **Purpose:** Count occurrences for F-score calculation
- **Weight:** 0 - do NOT affect pass/fail
- **Returns:** 1 or 0 (just counting)
- **These are statistics, not tests!**

---

## Why You See "Failures"

When you look at results, you might see:

```
Test 1: "This is amazing!" (positive)
✅ Assertion 1: PASS (accuracy)
✅ Assertion 2: 1 (true positive - counted)
❓ Assertion 3: 0 (false positive - not applicable)
❓ Assertion 4: 0 (false negative - not applicable)
```

**Those 0s are NOT failures!** They're correct counts. The assertion is saying:
- "Is this a false positive? No (0)"
- "Is this a false negative? No (0)"

---

## The Bottom Line

### ❌ Current Setup (Redundant)
- 4 assertions in CSV
- 4 assertions in defaultTest
- **= 8 total assertions per test** (duplicates!)

### ✅ Recommended Setup
- 0 assertions in CSV (just data)
- 4 assertions in defaultTest
- **= 4 total assertions per test**

### 🎯 What Actually Matters for Pass/Fail
- **Only 1 assertion** (accuracy) determines pass/fail
- **The other 3 assertions** just count metrics for F-score
- **All 4 are necessary** for F-score calculation

---

## Summary

1. **You're not seeing failures** - you're seeing counters returning 0 (which is correct)
2. **You ARE duplicating assertions** - remove them from CSV, keep in defaultTest
3. **You DO need all 4 assertions** - but only ONE affects pass/fail
4. **The others (weight: 0)** are just counting for statistics

**Think of it like a sports game:**
- Assertion 1 (accuracy) = Did you win? (Pass/Fail)
- Assertions 2-4 = Game statistics (points, rebounds, assists)

You need the statistics to understand performance, but only the win/loss matters for the final score!
