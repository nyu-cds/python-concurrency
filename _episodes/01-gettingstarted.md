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

The `get_links` function is used to obtain a list of available images.

The `download_link` function downloads the image given by the URL `link` into the download directory `directory`.

The `setup_download_dir` function creates a download destination directory if it doesn’t already exist.

Imgur’s API requires HTTP requests to bear the “Authorization” header with the client ID. You can find this client ID from the dashboard of the 
application that you have registered on Imgur. The response will be JSON encoded, and we can use Python’s standard JSON library to decode it.

This is what the `download.py` module looks like:

~~~
import json
import os
from pathlib import Path
from urllib.request import urlopen, Request

types = {'image/jpeg', 'image/png', 'image/gif'}

def get_links(client_id):
    headers = {'Authorization': 'Client-ID {}'.format(client_id)}
    req = Request('https://api.imgur.com/3/gallery/random/random/', headers=headers, method='GET')
    with urlopen(req) as resp:
        data = json.loads(resp.read().decode('utf-8'))
    return [item['link'] for item in data['data'] if 'type' in item and item['type'] in types]

def download_link(directory, link):
    download_path = directory / os.path.basename(link)
    with urlopen(link) as image, download_path.open('wb') as f:
        f.write(image.read())

def setup_download_dir():
    download_dir = Path('images')
    if not download_dir.exists():
        download_dir.mkdir()
    return download_dir
~~~
{: .python}

Next, we will need to write a module that will use these functions to download the images, one by one. We will name this program `single.py`. 

This program will contain the main function of our first, naive version of the Imgur image downloader. The module will use the Imgur 
client ID that you previously obtained. 

The program works by invoking the `setup_download_dir` to create the download destination directory. Then it will fetch a list of images using 
the `get_links` function, filter out all GIF and album URLs, and finally use `download_link` to 
download and save each of those images to the disk. 

Here is what `single.py` looks like:

~~~
from time import time

from download import setup_download_dir, get_links, download_link

CLIENT_ID = 'replace with your client ID'

def main():
   ts = time()
   download_dir = setup_download_dir()
   links = [l for l in get_links(CLIENT_ID)]
   for link in links:
       download_link(download_dir, link)
   print('Took {}s'.format(time() - ts))

if __name__ == '__main__':
   main()
~~~
{: .python}

> ## Challenge
>
> Create the `download.py` and `simple.py` files using the programs provided above. Replace the string `'replace with your client ID'` in 
> `simple.py` with the client ID you obtained from Imgur. Run `simple.py` and verify that you obtain a number of images in the `images`
> directory. Note how long it takes to download these images.
{: .challenge}

On my laptop, this script took 10.3 seconds to download 40 images. Please do note that these numbers may vary based on the network you are on. 
10.3 seconds isn’t terribly long, but what if we wanted to download more pictures? Perhaps 400 images, instead of 40. With an average of 0.25 
seconds per picture, 400 images would take approximately 1.5 minutes. For 4000 pictures it would take 15 minutes. The good news is that by 
introducing concurrency or parallelism, we can speed this up dramatically.
