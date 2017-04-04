---
title: "Multiple Processes"
teaching: 30
exercises: 20
questions:
objectives:
keypoints:
---
The multiprocessing module is easier to drop in than the threading module, as we don’t need to add a class like the threading example. The only 
changes we need to make are in the main function.

multiprocessing module

To use multiple processes we create a multiprocessing Pool. With the map method it provides, we will pass the list of URLs to the pool, which in 
turn will spawn 8 new processes and use each one to download the images in parallel. This is true parallelism, but it comes with a cost. The entire 
memory of the script is copied into each subprocess that is spawned. In this simple example it isn’t a big deal, but it can easily become serious 
overhead for non-trivial programs.

~~~
from functools import partial
from multiprocessing.pool import Pool

CLIENT_ID = #replace with your client id

def main():
   ts = time()
   download_dir = setup_download_dir()
   links = [l for l in get_links(CLIENT_ID) if l.endswith('.jpg')]
   download = partial(download_link, download_dir)
   with Pool(8) as p:
       p.map(download, links)
   print('Took {}s'.format(time() - ts))
~~~
{; .python}