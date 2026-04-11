# NumPy Cheat Sheet for Data Engineering

A practical reference for NumPy operations commonly used in data processing and pipeline work.

---

## Setup

```python
import numpy as np
```

---

## Creating Arrays

### From a list
```python
arr = np.array([1, 2, 3, 4, 5])
arr = np.array([[1, 2, 3], [4, 5, 6]])   # 2D array
```

### Preset arrays
```python
np.zeros((3, 4))          # 3x4 array of zeros
np.ones((3, 4))           # 3x4 array of ones
np.full((3, 4), 99)       # 3x4 array filled with 99
np.eye(3)                 # 3x3 identity matrix
np.empty((3, 4))          # uninitialized array (fast, values are garbage)
```

### Range arrays
```python
np.arange(0, 10, 2)       # [0, 2, 4, 6, 8]
np.linspace(0, 1, 5)      # 5 evenly spaced values between 0 and 1
```

### Random arrays
```python
np.random.rand(3, 4)         # uniform random floats between 0 and 1
np.random.randn(3, 4)        # standard normal distribution
np.random.randint(0, 100, (3, 4))   # random integers between 0 and 100
np.random.seed(42)           # set seed for reproducibility
```

---

## Array Properties

```python
arr.shape       # dimensions as a tuple e.g. (3, 4)
arr.ndim        # number of dimensions
arr.size        # total number of elements
arr.dtype       # data type of elements
```

---

## Indexing & Slicing

### 1D array
```python
arr[0]          # first element
arr[-1]         # last element
arr[1:4]        # elements at index 1, 2, 3
arr[::2]        # every second element
```

### 2D array
```python
arr[0, 1]       # row 0, column 1
arr[0, :]       # entire first row
arr[:, 1]       # entire second column
arr[0:2, 1:3]   # submatrix
```

### Boolean indexing
```python
arr[arr > 5]                  # elements greater than 5
arr[(arr > 2) & (arr < 8)]    # elements between 2 and 8
```

---

## Reshaping

### Reshape an array
```python
arr.reshape(2, 6)       # reshape to 2 rows, 6 columns
arr.reshape(-1, 3)      # auto-calculate rows, 3 columns
arr.flatten()           # collapse to 1D
arr.ravel()             # flatten (returns a view when possible)
```

### Transpose
```python
arr.T
np.transpose(arr)
```

### Add a dimension
```python
arr[:, np.newaxis]      # add a column dimension
arr[np.newaxis, :]      # add a row dimension
```

---

## Math Operations

### Element-wise operations
```python
arr + 10
arr * 2
arr ** 2
np.sqrt(arr)
np.log(arr)
np.exp(arr)
np.abs(arr)
```

### Operations between arrays
```python
a + b       # element-wise addition
a * b       # element-wise multiplication
np.dot(a, b)    # dot product / matrix multiplication
a @ b           # same as np.dot for 2D
```

---

## Aggregations

```python
arr.sum()
arr.mean()
arr.std()
arr.var()
arr.min()
arr.max()
arr.cumsum()        # cumulative sum
np.median(arr)
np.percentile(arr, 75)
```

### Aggregate along an axis
```python
arr.sum(axis=0)     # sum each column
arr.sum(axis=1)     # sum each row
arr.mean(axis=0)    # mean of each column
```

---

## Sorting & Searching

```python
np.sort(arr)                  # sorted copy
np.argsort(arr)               # indices that would sort the array
np.argmin(arr)                # index of the minimum value
np.argmax(arr)                # index of the maximum value
np.where(arr > 5, 1, 0)       # replace values based on condition
np.unique(arr)                # unique values
np.unique(arr, return_counts=True)   # unique values and their counts
```

---

## Combining Arrays

```python
np.concatenate([a, b])            # join 1D arrays
np.concatenate([a, b], axis=0)    # stack rows
np.concatenate([a, b], axis=1)    # stack columns
np.vstack([a, b])                 # vertical stack (row-wise)
np.hstack([a, b])                 # horizontal stack (column-wise)
np.stack([a, b], axis=0)          # stack along a new axis
```

---

## Handling NaN

```python
np.isnan(arr)               # boolean mask of NaN positions
np.nansum(arr)              # sum ignoring NaN
np.nanmean(arr)             # mean ignoring NaN
np.nanmax(arr)              # max ignoring NaN
arr[np.isnan(arr)] = 0      # replace NaN with 0
```

---

## Type Conversion

```python
arr.astype(float)
arr.astype(int)
arr.astype(np.float32)      # useful for reducing memory usage
```

---

## Saving & Loading

```python
np.save("array.npy", arr)           # save a single array
np.load("array.npy")                # load it back

np.savetxt("array.csv", arr, delimiter=",")     # save as CSV
np.loadtxt("array.csv", delimiter=",")          # load from CSV

np.savez("arrays.npz", a=arr1, b=arr2)         # save multiple arrays
data = np.load("arrays.npz")
data["a"], data["b"]
```

---

## ETL Patterns

### Normalize a column to 0–1 range
```python
arr = (arr - arr.min()) / (arr.max() - arr.min())
```

### Standardize a column (z-score)
```python
arr = (arr - arr.mean()) / arr.std()
```

### Replace outliers with the column mean
```python
mean = np.nanmean(arr)
std = np.nanstd(arr)
arr = np.where(np.abs(arr - mean) > 3 * std, mean, arr)
```

### Count null values in a numpy array
```python
np.isnan(arr).sum()
```

### Bin values into categories
```python
bins = [0, 100, 500, 1000, np.inf]
labels = np.digitize(arr, bins)
```

### Apply a threshold mask
```python
arr[arr < 0] = 0    # clip negatives to zero
```

### Compute a correlation matrix
```python
np.corrcoef(matrix.T)
```

### Efficient row-wise operations on a 2D array
```python
row_means = arr.mean(axis=1, keepdims=True)
normalized = arr - row_means
```

---

*Last updated: 2026*
