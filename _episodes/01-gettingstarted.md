---
title: "Getting Started"
teaching: 20
exercises: 0
questions:
objectives:
keypoints:
---
Let us start by creating a Python module, named `download.py`. This file will contain all the functions necessary to fetch the list of images and 
download them. We will split the functionality into three separate functions:

- `get_links`
- `download_link`
- `setup_download_dir`

The third function, `setup_download_dir`, will be used to create a download destination directory if it doesn’t already exist.

Imgur’s API requires HTTP requests to bear the “Authorization” header with the client ID. You can find this client ID from the dashboard of the 
application that you have registered on Imgur, and the response will be JSON encoded. We can use Python’s standard JSON library to decode it.
 
Downloading the image is an even simpler task, as all you have to do is fetch the image by its URL and write it to a file.

This is what the script looks like:

~~~
import json
import os
from pathlib import Path
from urllib.request import urlopen, Request

def get_links(client_id):
   headers = {'Authorization': 'Client-ID {}'.format(client_id)}
   req = Request('https://api.imgur.com/3/gallery/', headers=headers, method='GET')
   with urlopen(req) as resp:
       data = json.loads(resp.readall().decode('utf-8'))
   return map(lambda item: item['link'], data['data'])

def download_link(directory, link):
   print('Downloading %s', link)
   download_path = directory / os.path.basename(link)
   with urlopen(link) as image, download_path.open('wb') as f:
       f.write(image.readall())

def setup_download_dir():
   download_dir = Path('images')
   if not download_dir.exists():
       download_dir.mkdir()
   return download_dir
~~~
{: .python}

Next, we will need to write a module that will use these functions to download the images, one by one. We will name this `single.py`. 

This will contain the main function of our first, naive version of the Imgur image downloader. The module will use the Imgur 
client ID that you previously obtained. 

The program works by invoking the `setup_download_dir` to create the download destination directory. Then it will fetch a list of images using 
the `get_links` function, filter out all GIF and album URLs, and finally use `download_link` to 
download and save each of those images to the disk. 

Here is what `single.py` looks like:

~~~
import os
from time import time

from download import setup_download_dir, get_links, download_link

CLIENT_ID = #replace with your client id

def main():
   ts = time()
   download_dir = setup_download_dir()
   links = [l for l in get_links(CLIENT_ID) if l.endswith('.jpg')]
   for link in links:
       download_link(download_dir, link)
   print('Took {}s'.format(time() - ts))

if __name__ == '__main__':
   main()
~~~
{: .python}

On my laptop, this script took 19.4 seconds to download 91 images. Please do note that these numbers may vary based on the network you are on. 
19.4 seconds isn’t terribly long, but what if we wanted to download more pictures? Perhaps 900 images, instead of 90. With an average of 0.2 
seconds per picture, 900 images would take approximately 3 minutes. For 9000 pictures it would take 30 minutes. The good news is that by 
introducing concurrency or parallelism, we can speed this up dramatically.
