---
title: "Multiple Processes"
teaching: 10
exercises: 20
questions:
- "What is a simple way to improve the performance of our image downloader?"
objectives:
- "Learn about the Python `multiprocessing` module."
- "Learn how to use the `Pool` class to manage multiple processes."
keypoints:
- "The `multiprocessing` module provides a rich set of features for manipluating processes."
- "The `Pool` class is an easy way to provide work to groups of processes."
---
One way to improve the performance of our image downloader would be to run multiple copies of the program at the same time. This works because
most computers these days have multiple CPU cores, each of which can execute a copy of the program. Each of these *processes* is able to issue the download 
requests, and because these happen simultaneously, the overall image downloading proceeds much faster. 

However in order to do this, we would need to know what images are available so that we could ensure that one process didn't download an image
that had already be downloaded by a different process. It would be more convenient if Python provided a simple way of managing these processes 
rather than the user having to be concerned with how to start them. Fortunately the `multiprocessing` module is available for this purpose.

To use multiple processes we create a multiprocessing [`Pool`](https://docs.python.org/2/library/multiprocessing.html#module-multiprocessing.pool). 
The `Pool` class provides a `map` method that will run a function as a separate process, passing arguments from a supplied iterable. The iterable
is divided into a number of chunks, so that each process gets roughly the same number of elements. We will pass the list of URLs to the pool, which 
in turn will start 8 new processes and use each one to download the images in parallel. 

~~~
from time import time
from functools import partial
from multiprocessing.pool import Pool

from download import setup_download_dir, get_links, download_link

CLIENT_ID = 'replace with your client ID'

def main():
   ts = time()
   download_dir = setup_download_dir()
   links = [l for l in get_links(CLIENT_ID)]
   download = partial(download_link, download_dir)
   with Pool(8) as p:
       p.map(download, links)
   print('Took {}s'.format(time() - ts))

if __name__ == '__main__':
   main()
~~~
{: .python}

> ## Challenge
>
> Create a `procs.py` file using the program provided above. Replace the string `'replace with your client ID'` in 
> `procs.py` with the client ID you obtained from Imgur. Run `procs.py` and verify that you obtain a number of images in the `images`
> directory. Compare how long it takes to download these images with the `simple.py` version.
{: .challenge}

This is true parallelism, but it comes with a cost. Creating new processes is a relatively expensive operation that requires system resources.
Also, when a new process is created, it contains a copy of the entire memory of the script. In this simple example it isn’t a big deal, but it 
can easily become serious overhead for non-trivial programs.