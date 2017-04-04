---
title: "Using Threads"
teaching: 20
exercises: 0
questions:
objectives:
keypoints:
---
Threading is one of the most well known approaches to attaining Python concurrency and parallelism. Threading is a feature usually provided by the 
operating system. Threads are lighter than processes, and share the same memory space.

In our Python thread tutorial, we will write a new module to replace `single.py`. This module will create a pool of 8 threads, making a total of 9 
threads including the main thread. I chose 8 worker threads, because my computer has 8 CPU cores and one worker thread per core seemed a good number 
for how many threads to run at once. In practice, this number is chosen much more carefully based on other factors, such as other applications and 
services running on the same machine.

This is almost the same as the previous one, with the exception that we now have a new class, `DownloadWorker`, that inherits from the Thread class.
 The run method has been overridden, which runs an infinite loop. On every iteration, it calls `self.queue.get()` to try and fetch an URL to from a 
thread-safe queue. It blocks until there is an item in the queue for the worker to process. Once the worker receives an item from the queue, it then 
calls the same `download_link` method that was used in the previous script to download the image to the images directory. After the download is 
finished, the worker signals the queue that that task is done. This is very important, because the Queue keeps track of how many tasks were enqueued. 
The call to `queue.join()` would block the main thread forever if the workers did not signal that they completed a task.

~~~
from queue import Queue
from threading import Thread

CLIENT_ID = #replace with your client id

class DownloadWorker(Thread):
   def __init__(self, queue):
       Thread.__init__(self)
       self.queue = queue

   def run(self):
       while True:
           # Get the work from the queue and expand the tuple
           directory, link = self.queue.get()
           download_link(directory, link)
           self.queue.task_done()

def main():
   ts = time()
   download_dir = setup_download_dir()
   links = [l for l in get_links(CLIENT_ID) if l.endswith('.jpg')]
   # Create a queue to communicate with the worker threads
   queue = Queue()
   # Create 8 worker threads
   for x in range(8):
       worker = DownloadWorker(queue)
       # Setting daemon to True will let the main thread exit even though the workers are blocking
       worker.daemon = True
       worker.start()
   # Put the tasks into the queue as a tuple
   for link in links:
       logger.info('Queueing {}'.format(link))
       queue.put((download_dir, link))
   # Causes the main thread to wait for the queue to finish processing all the tasks
   queue.join()
   print('Took {}'.format(time() - ts))
~~~
{: .python}

Running this script on the same machine used earlier results in a download time of 4.1 seconds! Thats 4.7 times faster than the previous example. 
While this is much faster, it is worth mentioning that only one thread was executing at a time throughout this process due to the GIL. 
Therefore, this code is concurrent but not parallel. The reason it is still faster is because this is an IO bound task. The processor is hardly 
breaking a sweat while downloading these images, and the majority of the time is spent waiting for the network. This is why threading can provide 
a large speed increase. The processor can switch between the threads whenever one of them is ready to do some work. Using the threading module in 
Python or any other interpreted language with a GIL can actually result in reduced performance. If your code is performing a CPU bound task, 
such as decompressing gzip files, using the threading module will result in a slower execution time. For CPU bound tasks and truly parallel 
execution, we can use the multiprocessing module.

While the de facto reference Python implementation - CPython - has a GIL, this is not true of all Python implementations. For example, IronPython, a 
Python implementation using the .NET framework does not have a GIL, and neither does Jython, the Java based implementation. You can find a list of 
working Python implementations here.

