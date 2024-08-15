## Set up

Install ipython3

`$ sudo apt install ipython3`

Get documention of a function at ipython console:

`> help(np.argmax)`



## tricks

### unpack a dictionary as function argument
The dictionary keys must match function parameters.

```python
dic = {"x": 100, "y": 200}
def add(x, y):
    return(x + y)
    
add(**dic)
    # 300
```

## Packages

### numpy

#### np.argmax()
Index of the largest value along a axis

```python
import numpy as np
arr = np.array([[1,3,2,1], [2,5,8,9], [9,4,4,3]])
    # array([[1, 3, 2, 1],
    #        [2, 5, 8, 9],
    #        [9, 4, 4, 3]])

# axis=0 means which row index has the largest value (in each column)
np.argmax(arr, axis=0)
    # array([2, 1, 1, 1])

# axis=1 returns column index of the largest value in each row
np.argmax(arr, axis=1)
    # array([1, 3, 0])
    
# axis=-1 where -1 is the index of the last axis, which is column for 2-D arrays
np.argmax(arr, axis=-1)
    # array([1, 3, 0])
```
