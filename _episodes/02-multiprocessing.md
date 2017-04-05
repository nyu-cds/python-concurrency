---
title: "Multiple Processes"
teaching: 30
exercises: 20
questions:
objectives:
keypoints:
---
One way to improve the performance of our image downloader would be to run multiple copies of the program at the same time. This works because
most computers these days have multiple CPUs, each of which can execute a program. Each of these *processes* is able to issue the download 
requests, and because these happen simultaneously, the overall image downloading proceeds much faster. 

However it would be more convenient if Python provided a simple way of managing these processes rather than the user having to be concerned with
how to start them. Fortunately the `multiprocessing` module is available for this purpose.

To use multiple processes we create a multiprocessing `Pool`. With the `map` method it provides, we will pass the list of URLs to the pool, which in 
turn will start 8 new processes and use each one to download the images in parallel. 

~~~
from time import time
from functools import partial
from multiprocessing.pool import Pool

CLIENT_ID = 'replace with your client ID'

def main():
   ts = time()
   download_dir = setup_download_dir()
   links = [l for l in get_links(CLIENT_ID)]
   download = partial(download_link, download_dir)
   with Pool(8) as p:
       p.map(download, links)
   print('Took {}s'.format(time() - ts))
~~~
{: .python}

> ## Challenge
>
> Create a `procs.py` file using the program provided above. Replace the string `'replace with your client ID'` in 
> `procs.py` with the client ID you obtained from Imgur. Run the `procs.py` and verify that you obtain a number of images in the `images`
> directory. Compare how long it takes to download these images with the `simple.py` version.
{: .challenge}

This is true parallelism, but it comes with a cost. Creating new processes is a relatively expensive operation that requires system resources.
Also, when a new process is created, it contains a copy of the entire memory of the script. In this simple example it isn’t a big deal, but it 
can easily become serious overhead for non-trivial programs.