---
title: Running Python Unbuffered
slug: running-python-unbuffered
description: >-
  Getting 'real-time' print statements in Docker container.
tags:
  - python
added: "January 25 2025"
---

# Why did my Docker container hang?

This was the question I struggled with for a few hours. Debugging the issue turned out to be pretty tricky. It did not help that my script was one giant `if` statement that included multiple `while` loops and in the middle of that had to wait roughly 20 minutes for a reply from an api.

> Where was the problem?  
> It worked in my local environment.  
> Oh Docker :/  


Was it:
- something wrongly indented?
- a loop not properly exited?
- some weird Docker thing that I have no idea about?

Nope!

It was Python buffering my print statements (default behavior in non interactive mode). If I was patient and had let the script run the normal amount of time it took to complete, I would have seen all my print statements show up when it exited. The script was running the whole time.

The thing that made troubleshooting difficult was that my print statements would show up until a `SQLAlchemy` debug print statement fired. This happened right before having to wait for a response from the API... then "nothing". So without knowing about Python's default buffering, it looked like the API portion of the code hanging.

So why were the print statement up until the `SQLAlchemy` showing up? I am guessing `SQLAlchemy` manually 'flushed' the buffer. Print has a few arguments that can be passed in and a flush boolean is one of them. The following is the defaults for print:
`print(*objects, sep=' ', end='\n', file=None, flush=False)`

So by setting `flush=True` that would have emptied out the buffer and make it look like the script was hanging at the API call.

## Troubleshooting Steps
- I ended up adding some logs where some of the print statements were in the API portion of the code. When running the code, the logs were being logged where the print statements were failing to print at the same spot in the code. This let me know it wasn't a script hanging issue at all, it was a print statements not printing issue.  

### Logging example
```python
# import module
import logging

# configure logger to append to file with debug level threshold
logging.basicConfig(
    filename="debugging.log",
    format='%(asctime)s %(message)s',
    filemode='a',
    level='DEBUG',
)

# create logger
logger = logging.getLogger()

# place debug messages in code where I think it is failing
logger.debug('still running before/after x area of code')
```  
- I read up on issues of Python print statements not printing which lead me to Python's default behavior of buffering them in non interactive environments. I could have went through and added `flush=True` to all my print statements or I could run the whole script unbuffered with the `-u` flag.

## Solution

I ended up using the `-u` flag when running the script for unbuffered print statements. This allowed me to read the print statement in real-time and monitor the status of the script and API call.
```bash
python -u main.py
```

## Final Thoughts
Pretty interesting learning about buffering in Python. Most of the time when writing and testing a script I do not witness that behavior because I am in interactive mode (REPL or hitting the play button in VSCode).  

From now on I will run my script with the `-u` flag when testing to get real- time print statements and monitor the script. Once confirmed everything is running correctly, I will run it without the `-u` flag. This will allow me to take advantage of any buffering performance gains.  


### Resources
#### Buffer
https://realpython.com/python-flush-print-output/
https://docs.python.org/3/library/functions.html#print
#### Logging
https://www.geeksforgeeks.org/logging-in-python/
https://docs.python.org/3/library/logging.html#logging.basicConfig
