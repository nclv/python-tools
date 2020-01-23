From 
## Systematic and Random Errors

So what is wrong with the following way of measuring?
```python
time_start = time.perf_counter()
my_function()
time_end = time.perf_counter()
execution_time = time_end - time_start
```
First, there is a **systematic error**: by invoking `time.perf_counter()`, an unknown amount of time is added to the execution time of `my_function()`. How much time? This depends on the OS, the particular implementation and other uncontrollable factors.

Second, there is a **random error**: the execution time of the call to `my_function()` will vary to a certain degree.

We can combat the random error by just *performing multiple measurements and taking the average of those*. However, it is much more challenging to remove the systematic error.
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTgzMTQzNzkyOV19
-->