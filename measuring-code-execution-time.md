From <https://knasmueller.net/measure-code-execution-time-accurately-in-python>.
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

We can combat the random error by just *performing multiple measurements and taking the average of those*. 

However, it is much more challenging to remove the systematic error.

**The idea is to first measure the time of _one_ function call, then the time of _two_, then the time of three, and so on....**
You can then fit a straight line through the measurements. The overall execution time can then be obtained by taking the slope `a` from the straight line `y = a x + b`. 
This type of measurement is very robust against occasional measurements with large errors. This can be visualized by artificially changing one measurement and rerunning the line fitting process.

## Code Samples
Gist [bench.py](https://gist.github.com/NicovincX2/58bf32e1555a1f617191ea578a5ec6d3)
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE4NDM4MDM0OTMsLTE1Nzk0NjU0MTAsLT
E3MDk1MTI4MDFdfQ==
-->