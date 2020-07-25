---
description: Learning how to perform actions which do not block the main thread
---

# Asynchronous operations

We had `shuffle_image` in the `__init__` method. Which in turn caused the application to not load until the image was fetched from the API. Let us now make another function called async\_shuffle\_image which does not block on the main thread. 

We will be using the `threading` library to achieve this. Go through [this](https://realpython.com/intro-to-python-threading/#what-is-a-thread) article to understand how the threading library should be used. The gist of it is that, thread is a separate flow of execution. But these threads are not actual OS threads. So things will not technically be running in parallel. This is okay since our operations are not CPU Intensive but IO intensive. If they are CPU intensive, then `multiprocessing` module should be preferred.

A thread can be instantiated by using the `threading.Thread(fn, args)` method.This method will return a thread which will start running once we call the `.start()` method on it. You can also pickup a thread by using the `.join()` function.

A daemon is a process running in the background in CS terms. The threading.Thread function takes in a parameter called daemon which is a boolean. If it is true, then the thread will be pushed to the background. Now let us use this knowledge to make our shuffle function async.



