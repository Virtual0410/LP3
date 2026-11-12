# LP3 Final Cheat Sheet: Code + Viva

DAA, Blockchain and ML. Every practical has the final code, then the viva points for it.

# Part A: DAA

## A1. Fibonacci (non-recursive and recursive)

```python
def fib_iterative(n):          # non-recursive
    a, b = 0, 1
    for _ in range(n):
        a, b = b, a + b
    return a

def fib_recursive(n):
    if n <= 1: return n
    return fib_recursive(n - 1) + fib_recursive(n - 2)

n = int(input("n: "))
print("Non-recursive:", [fib_iterative(i) for i in range(n)])
print("Recursive    :", [fib_recursive(i) for i in range(n)])

# Non-recursive: time O(n),   space O(1)
# Recursive:     time O(2^n), space O(n)  (recursion depth)
```

**Viva points**

- **Non-recursive:** time **O(n)** (one loop), space **O(1)** (only `a` and `b` are stored).
- **Recursive:** time **O(2ⁿ)** (every call makes two more calls, so the call tree doubles each level), space **O(n)** (the call stack goes n levels deep).
- Why recursion is slow: it recomputes the same values again and again (fib(3) is calculated many times).
- Fix: memoization or DP brings the recursive version down to O(n).

## A2. Huffman encoding (greedy)

```python
from collections import Counter
import heapq

def huffman_encode(text):
    if not text:
        return "", None
    # 1. Count character frequencies
    heap = [[weight, [char, ""]] for char, weight in Counter(text).items()]
    heapq.heapify(heap)
    # 2. Build the Huffman tree greedily (merge the two lowest-frequency nodes)
    while len(heap) > 1:
        lo = heapq.heappop(heap)
        hi = heapq.heappop(heap)
        for pair in lo[1:]:
            pair[1] = '0' + pair[1]       # left branch = 0
        for pair in hi[1:]:
            pair[1] = '1' + pair[1]       # right branch = 1
        heapq.heappush(heap, [lo[0] + hi[0]] + lo[1:] + hi[1:])
    # 3. Dictionary of codes
    codes = dict(heapq.heappop(heap)[1:])
    # 4. Encode the text
    return "".join(codes[ch] for ch in text), codes

text = "hello"
encoded, codes = huffman_encode(text)
print(f"Original: {text}")
print(f"Codes: {codes}")
print(f"Encoded: {encoded}")
# Original: hello
# Codes: {'l': '0', 'o': '10', 'e': '110', 'h': '111'}
# Encoded: 1111100010
```

**Viva points**

- Huffman coding is a **greedy** compression algorithm: frequent characters get **short** binary codes, rare ones get **long** codes.
- Steps: count frequencies, put them in a **min-heap**, repeatedly pop the **two lowest** frequencies and merge them into one node (left = 0, right = 1), until one tree remains. The path from the root to a leaf is that character's code.
- It gives a **prefix code**: no code is the start of another, so the encoded bits decode without ambiguity.
- Why greedy is optimal: always merging the two smallest weights minimises the total weighted code length.
- Sample: "hello" has l = 2 and h, e, o = 1 each. Codes are l = 0, o = 10, e = 110, h = 111, so the output is 1111100010 (10 bits instead of 40 in 8-bit ASCII).
- Complexity: **O(n log n)** for n distinct characters (each heap operation is O(log n)).
- Used in: ZIP, JPEG and MP3 compression. It is lossless.

## A3. Fractional knapsack (greedy)

```python
def fractional_knapsack(w, p, cap):
    profit = 0
    for wt, pr in sorted(zip(w, p), key=lambda x: x[1] / x[0], reverse=True):
        take = min(wt, cap)
        profit += pr * take / wt
        cap -= take
    return profit

print(fractional_knapsack([10,15,20,10,16], [20,20,30,40,20], 50))   # 103.33
```

**Viva points**

- Greedy rule: highest **profit/weight ratio first**, and take a fraction of the last item.
- Complexity: **O(n log n)** because of the sort.
- Working for the sample: 40 + 20 + 30, then 10/15 of the (15, 20) item = 13.33. Total 103.33.
- Greedy is **optimal** here because items can be split.

## A4. 0/1 knapsack (dynamic programming)

```python
def knapsack(w, v, cap):
    dp = [0] * (cap + 1)
    for wt, val in zip(w, v):
        for c in range(cap, wt - 1, -1):          # backwards => each item used once
            dp[c] = max(dp[c], val + dp[c - wt])
    return dp[cap]

print(knapsack([2,3,4,5], [3,4,5,6], 5))          # 7
```

**Viva points**

- Recurrence: `dp[c] = max(skip item, value + dp[c - weight])`.
- Time **O(n·W)**, space **O(W)** with the 1-D table (O(n·W) for the 2-D table).
- It is **pseudo-polynomial**: polynomial in the value of W, not in the number of bits of W. The problem itself is NP-hard.
- The loop runs **backwards** so an item is not counted twice.
- Greedy fails here. Items (w,v) = (10,60), (20,100), (30,120), capacity 50: greedy by ratio gives 160, the optimum is 220.
- Alternative approach: branch and bound.

## A5. n-Queens with the first queen placed (backtracking)

```python
# q[r] = column of the queen in row r
def safe(q, c):
    r = len(q)
    return all(c != x and abs(c - x) != r - i for i, x in enumerate(q))

def solve(q, n):
    if len(q) == n: return True
    for c in range(n):
        if safe(q, c):
            q.append(c)
            if solve(q, n): return True
            q.pop()                                # backtrack
    return False

n = int(input("n: "))                              # e.g. 8
q = [int(input("Column of first queen (0 to n-1): "))]   # first queen is in row 0
if solve(q, n):
    for c in q: print([int(i == c) for i in range(n)])
else:
    print("No solution")
```

**Viva points**

- One queen per row, so only the **column** and the **two diagonals** need checking.
- Diagonal test: two queens clash if `|col1 - col2| == |row1 - row2|`.
- Backtracking = build step by step, **undo** (`pop`) when stuck, try the next option. It prunes invalid branches early, unlike brute force (about 4.4 billion placements).
- Worst case O(n!). For n = 8 the puzzle has **92 solutions** (12 distinct ignoring rotations and reflections).

**Greedy vs DP vs Backtracking in one line each:** greedy never reconsiders, DP stores and reuses subproblem answers, backtracking explores and undoes.

# Part B: Blockchain (BT)

## Deploy and observe gas (Remix)

1. Open Remix, paste the file, set compiler to 0.8.20+, compile.
2. Environment: **Injected Provider (MetaMask)** on a test network such as Sepolia (get test ETH from a faucet), or Remix VM for a quick demo.
3. Deploy, then expand each transaction in the console and note **gas used**, **transaction cost** and **execution cost**.
4. Fee = **gas used × gas price** (gwei). Deployment costs much more than a call. `view` functions cost no gas when called externally.

## B1. Bank account contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract BankAccount {
    uint public balance = 0;
    
    function setInitialBalance(uint initialBalance) public {
        balance = initialBalance;
    }

    function deposit(uint depositAmount) public returns(uint, string memory){
        balance = balance + depositAmount;
        return (balance, "SuccessFul");
    }

    function withdraw(uint withdrawAmount) public returns(uint){
        if(balance<=500){
            return balance;
        }
        else if(withdrawAmount>balance){
            return balance;
        }
        balance = balance - withdrawAmount;
        return balance;
    }

    function showBalance() public view returns(uint){
        return balance;
    }
}
```

Compiles cleanly on solc 0.8.26 (set the Remix compiler to 0.8.20 or higher).

**Viva points: Banking**

- **Smart contract**: code stored on the blockchain that executes automatically and cannot be changed once deployed.
- `uint public balance` is a **state variable** stored on-chain. `public` makes Solidity create a getter function `balance()` automatically. `uint` is the same as `uint256`.
- **Three operations asked:** `deposit` adds money, `withdraw` takes money out, `showBalance` shows it. `setInitialBalance` sets the starting amount.
- `deposit` returns **two values** (the new balance and a message string).
- **Withdraw rules in your code:** if the balance is 500 or less, nothing is withdrawn (a minimum-balance rule). If the amount is more than the balance, nothing is withdrawn (insufficient funds). Otherwise the amount is subtracted.
- `showBalance` is a `view` function: it only reads state, so it costs no gas when called externally. `deposit` and `withdraw` change state, so they are **transactions** that cost gas.
- Solidity 0.8 has **built-in overflow and underflow checks**, so `balance - withdrawAmount` can never silently wrap around.
- `string memory` is needed because strings are reference types and must say where they live (memory = temporary).
- **Known limitations (say these before they ask):** it keeps **one shared balance** for the whole contract, not one per customer (a real bank uses `mapping(address => uint)` with `msg.sender`). Anyone can call `setInitialBalance` (no access control). The numbers are plain values, not real ether (a real version uses `payable` and `msg.value`). Invalid withdrawals **return quietly instead of reverting**, so the transaction still succeeds and costs gas (`require` would revert). The 500 rule only checks the balance *before* withdrawing, so a withdrawal can still leave less than 500.
- Gas observation: fee = gas used × gas price. Deployment costs the most; `showBalance` called externally costs nothing.

## B2. Student data contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract StudentData{
    struct Student{
        uint256 id;
        string name;
        uint256 age;
        string major;
    }

    Student[] public students;

    function addStudent(
        uint256 _id,
        string memory _name,
        uint256 _age,
        string memory _major
    ) public{
        students.push(Student({
            id: _id,
            name: _name,
            age: _age,
            major: _major
        }));
    }

    function getStudentCount() public view returns (uint256) {
        return students.length;
    }
}
```

**Viva points: Student**

- **Structure** (`struct Student`) = a custom data type that groups id, name, age and major. **Array** (`Student[] public students`) = a dynamic list of those structs that grows with `push`.
- Because the array is `public`, Solidity creates a getter `students(index)` that returns the fields of the student at that index (index starts at 0).
- `addStudent` builds a `Student` using named fields and pushes it into the array. It changes state, so it is a **transaction that costs gas**. Writing to storage is the expensive part, and longer strings cost more.
- `getStudentCount` is a `view` function (reads `students.length`), so it costs no gas when called externally.
- `string memory` is needed for string parameters because strings are reference types. **storage** is permanent and expensive, **memory** is temporary, **calldata** is read-only input and the cheapest.
- **Fallback is not in your current code, but the question asks for it.** Add these two lines inside the contract before deploying: `fallback() external payable {}` and `receive() external payable {}`. The **fallback** function runs when a call matches no function (or ether arrives with data). **`receive()`** handles plain ether transfers with empty calldata.
- Limitations: no duplicate-ID check, and no search by ID (you look students up by array index). A mapping from ID to index would give O(1) lookup.
- Gas = unit of computation; fee = gas used × gas price. Deployment gas is high because the bytecode is stored on-chain.

# Part C: Machine Learning

## Shared imports (run once at the top)

```python
import numpy as np, pandas as pd
import matplotlib.pyplot as plt, seaborn as sns
from sklearn.model_selection import train_test_split, GridSearchCV
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import make_pipeline
from sklearn.impute import SimpleImputer
from sklearn.metrics import (confusion_matrix, accuracy_score, precision_score, recall_score,
                             classification_report, mean_absolute_error, mean_squared_error,
                             r2_score, silhouette_score)
```

## C1. Uber fare: Linear Regression vs Random Forest

```python
from sklearn.linear_model import LinearRegression
from sklearn.ensemble import RandomForestRegressor

df = pd.read_csv("uber.csv").dropna().drop(columns=["Unnamed: 0", "key"], errors="ignore")
dt = pd.to_datetime(df.pop("pickup_datetime"))
df = df.assign(year=dt.dt.year, month=dt.dt.month, day=dt.dt.day,
               hour=dt.dt.hour, day_of_week=dt.dt.dayofweek)

sns.boxplot(x=df["fare_amount"]); plt.show()                      # outliers (visual)

# outlier removal in one pass (NYC bounding box, sane fare, 1-6 passengers)
df = df[df.fare_amount.between(0.01, 200) & df.passenger_count.between(1, 6)
        & df.pickup_latitude.between(40, 42) & df.dropoff_latitude.between(40, 42)
        & df.pickup_longitude.between(-75, -72) & df.dropoff_longitude.between(-75, -72)].copy()

def haversine(lat1, lon1, lat2, lon2):                            # vectorised, km
    lat1, lon1, lat2, lon2 = map(np.radians, (lat1, lon1, lat2, lon2))
    a = np.sin((lat2-lat1)/2)**2 + np.cos(lat1)*np.cos(lat2)*np.sin((lon2-lon1)/2)**2
    return 6371 * 2 * np.arcsin(np.sqrt(a))

df["distance_km"] = haversine(df.pickup_latitude, df.pickup_longitude,
                              df.dropoff_latitude, df.dropoff_longitude)
df = df[df.distance_km > 0]

plt.figure(figsize=(10, 7)); sns.heatmap(df.corr(), annot=True, fmt=".2f", cmap="coolwarm"); plt.show()
print(df.corr()["fare_amount"].sort_values(ascending=False))

X, y = df.drop(columns="fare_amount"), df["fare_amount"]
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

models = {"Linear Regression": LinearRegression(),
          "Random Forest": RandomForestRegressor(n_estimators=100, n_jobs=-1, random_state=42)}
rows = []
for name, m in models.items():
    p = m.fit(X_train, y_train).predict(X_test)
    rows.append({"Model": name, "MAE": mean_absolute_error(y_test, p),
                 "RMSE": np.sqrt(mean_squared_error(y_test, p)), "R2": r2_score(y_test, p)})
print(pd.DataFrame(rows).set_index("Model"))
```

**Viva points**

- Pre-processing: drop nulls (1 row), drop useless columns (`key`, index), split datetime into year, month, day, hour, day of week.
- Outliers: boxplot/histogram to *see* them, then domain rules (fare ≤ 0 or > 200, passengers 0 or > 6, coordinates outside NYC). Alternative: the IQR rule.
- **Haversine** converts two lat/long pairs into great-circle distance in km (Earth radius 6371).
- Correlation: distance is the strongest predictor of fare (about 0.84 in the earlier run).
- **Random Forest beat Linear Regression** (earlier run: R² 0.857 vs 0.752, RMSE 3.62 vs 4.76) because the relationship is not perfectly linear and forests capture interactions.
- **RMSE** is in dollars and punishes big errors; **MAE** is the average absolute error; **R²** is the share of variance explained.
- Random Forest = many decision trees on bootstrap samples; predictions are averaged (bagging).

## C2. Email spam: KNN vs SVM

```python
from sklearn.neighbors import KNeighborsClassifier
from sklearn.svm import LinearSVC

df = pd.read_csv("emails.csv")
X, y = df.drop(columns=["Email No.", "Prediction"]), df["Prediction"]      # 1 = spam
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42, stratify=y)

for name, clf in {"KNN": KNeighborsClassifier(n_neighbors=5, n_jobs=-1),
                  "SVM": LinearSVC(max_iter=5000)}.items():            # LinearSVC = fast linear SVM
    pipe = make_pipeline(StandardScaler(), clf).fit(X_train, y_train)
    p = pipe.predict(X_test)
    print(f"\n{name}\n", confusion_matrix(y_test, p))
    print(classification_report(y_test, p, target_names=["Not spam", "Spam"]))
```

**Viva points**

- Binary classification: Normal state = not spam (0), Abnormal state = spam (1). Features are word counts.
- **KNN**: lazy learner (no training), classifies by majority vote of the k nearest points, uses distance so it needs **scaling**, and is slow on thousands of features (curse of dimensionality).
- **SVM**: finds the maximum-margin hyperplane; **support vectors** are the points on the margin; kernels (the kernel trick) handle non-linear data. A linear kernel suits text.
- Expect SVM to beat KNN. Compare with precision and recall: for spam, **precision** matters (don't send real mail to the spam folder).
- `stratify=y` keeps the spam ratio the same in train and test.

## C3. Bank churn: neural network

```python
from tensorflow import keras
from tensorflow.keras import layers

df = pd.read_csv("Churn_Modelling.csv")
X = df.drop(columns=["RowNumber", "CustomerId", "Surname", "Exited"])
y = df["Exited"]
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42, stratify=y)

prep = ColumnTransformer([("cat", OneHotEncoder(drop="first"), ["Geography", "Gender"])],
                         remainder=StandardScaler())                   # fit on TRAIN only
X_train = prep.fit_transform(X_train); X_test = prep.transform(X_test)

model = keras.Sequential([
    layers.Input(shape=(X_train.shape[1],)),
    layers.Dense(32, activation="relu"), layers.Dropout(0.2),
    layers.Dense(16, activation="relu"), layers.Dropout(0.2),
    layers.Dense(1, activation="sigmoid")])
model.compile(optimizer="adam", loss="binary_crossentropy", metrics=["accuracy"])

# improvements: dropout, early stopping, class weights (churners are only ~20% of data)
hist = model.fit(X_train, y_train, validation_split=0.2, epochs=100, batch_size=64, verbose=0,
                 class_weight={0: 1.0, 1: 2.0},
                 callbacks=[keras.callbacks.EarlyStopping(patience=10, restore_best_weights=True)])

y_pred = (model.predict(X_test, verbose=0) > 0.5).astype(int).ravel()
print("Accuracy:", accuracy_score(y_test, y_pred))
print(confusion_matrix(y_test, y_pred))
print(classification_report(y_test, y_pred))
pd.DataFrame(hist.history)[["loss", "val_loss"]].plot(); plt.show()
```

**Viva points**

- Features vs target: target = `Exited`; drop identifiers (`RowNumber`, `CustomerId`, `Surname`) because they carry no signal.
- Categorical columns (Geography, Gender) are one-hot encoded; `drop="first"` avoids the dummy-variable trap. Numeric columns are standardised, **fitted on train only** to avoid leakage.
- Architecture: **ReLU** in hidden layers (fast, avoids vanishing gradients), **sigmoid** at the output (probability for binary classes), **binary cross-entropy** loss, **Adam** optimiser.
- Improvements: **Dropout** (randomly switches off neurons, reduces overfitting), **EarlyStopping** (stops when validation loss stops improving and restores the best weights), **class weights** (churners are about 20%).
- Earlier run: accuracy 85.85% but churn **recall only 0.41**. Accuracy hides class imbalance. Other fixes: lower threshold than 0.5, SMOTE.
- Validation accuracy above training accuracy is normal with dropout (it is only active in training).
- Epoch = one pass over the data; batch = samples per weight update.

## C4. KNN on diabetes

```python
df = pd.read_csv("diabetes.csv")
X, y = df.drop(columns="Outcome"), df["Outcome"]

# zeros in these columns are really missing values
X = X.copy()
cols = ["Glucose", "BloodPressure", "SkinThickness", "Insulin", "BMI"]
X[cols] = X[cols].replace(0, np.nan)

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42, stratify=y)

pipe = make_pipeline(SimpleImputer(strategy="median"), StandardScaler(), KNeighborsClassifier())
gs = GridSearchCV(pipe, {"kneighborsclassifier__n_neighbors": range(1, 31, 2)}, cv=5).fit(X_train, y_train)
print("Best k:", gs.best_params_["kneighborsclassifier__n_neighbors"])

p = gs.predict(X_test)
acc = accuracy_score(y_test, p)
print("Confusion matrix:\n", confusion_matrix(y_test, p))
print(f"Accuracy {acc:.3f} | Error rate {1-acc:.3f} | "
      f"Precision {precision_score(y_test, p):.3f} | Recall {recall_score(y_test, p):.3f}")
```

**Viva points**

- Metrics from the confusion matrix `[[TN, FP], [FN, TP]]`: **accuracy** = (TP+TN)/total, **error rate** = 1 − accuracy, **precision** = TP/(TP+FP), **recall** = TP/(TP+FN).
- For diabetes, **recall** matters more: a missed patient (false negative) is the costly mistake.
- Why scale: KNN is distance-based, so Insulin (up to 846) would swamp Pregnancies (up to 17).
- Why `transform` only on test: fitting on test leaks information. The scaler and imputer live inside the pipeline so cross-validation stays clean.
- Choosing k: cross-validate odd values. Small k overfits (noisy), large k underfits (too smooth). Odd k avoids ties.
- Zeros in Glucose, Insulin, BMI etc. are impossible values, so they are treated as missing and imputed with the median.
- Earlier result for reference: k=5 gave confusion matrix \[\[79,20\],\[27,28\]\], accuracy 0.695, precision 0.583, recall 0.509. Your new numbers will differ.

## C5. K-Means on sales data

```python
from sklearn.cluster import KMeans

df = pd.read_csv("sales_data_sample.csv", encoding="latin1")
features = ["QUANTITYORDERED", "PRICEEACH", "SALES", "MSRP"]
Xs = StandardScaler().fit_transform(df[features].dropna())

ks = range(2, 11)
fits = {k: KMeans(n_clusters=k, n_init=10, random_state=42).fit(Xs) for k in ks}
wcss = [fits[k].inertia_ for k in ks]
sil = [silhouette_score(Xs, fits[k].labels_) for k in ks]

fig, ax = plt.subplots(1, 2, figsize=(11, 4))
ax[0].plot(ks, wcss, "o-"); ax[0].set(title="Elbow (WCSS)", xlabel="k")
ax[1].plot(ks, sil, "o-");  ax[1].set(title="Silhouette", xlabel="k")
plt.show()

k = 3                                                   # pick from the elbow
df_c = df[features].dropna().assign(Cluster=fits[k].labels_)
print(df_c["Cluster"].value_counts().sort_index())
print(df_c.groupby("Cluster").mean())                   # cluster profiles
plt.scatter(df_c.QUANTITYORDERED, df_c.SALES, c=df_c.Cluster); plt.show()
```

**Viva points**

- **Elbow method**: plot WCSS (within-cluster sum of squares, `inertia_`) against k and pick the bend where adding clusters stops helping much. Yours is about **k = 3**.
- **Silhouette score** (−1 to 1) measures how well points fit their own cluster vs the next one. Earlier run: 0.35, which is moderate (overlapping clusters).
- Algorithm: pick k centroids, assign each point to the nearest, recompute centroids, repeat until stable.
- Scale first, since distances drive everything.
- Weaknesses: you must choose k, results depend on initial centroids (hence `n_init=10`, `random_state`), assumes round clusters of similar size, sensitive to outliers.
- Here SALES = QUANTITY × PRICE, so the features are partly redundant.
- Unsupervised = no labels. Hierarchical clustering is the alternative (dendrogram, no need to fix k up front).

# Part D: One-liners for any viva

- **Supervised vs unsupervised**: with labels (regression, classification) vs without (clustering).
- **Overfitting**: great on training data, poor on new data. Cures: more data, dropout, early stopping, simpler model, cross-validation.
- **Underfitting**: model too simple to capture the pattern.
- **Train/test split**: test data must stay unseen until the final evaluation.
- **Regression vs classification**: predicts a number (fare) vs a class (spam, churn, diabetes).
- **Accuracy is not enough** when classes are imbalanced; use precision, recall, F1.
- **Gradient descent** (if asked): update x ← x − α·f′(x). For y = (x+3)² starting at x = 2 it converges to x = −3.

# 
