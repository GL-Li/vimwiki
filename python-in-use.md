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
