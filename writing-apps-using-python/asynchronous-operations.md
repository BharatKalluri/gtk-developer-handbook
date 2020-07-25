---
description: Learning how to perform actions which do not block the main thread
---

# Asynchronous operations

We had shuffle\_image in the \_\_init\_\_ method. Which in turn caused the application to not load until the image was fetched from the API. Let us now make another function called async\_shuffle\_image which does not block on the main thread. 

