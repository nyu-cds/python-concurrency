---
layout: lesson
root: .
---
Discussions criticizing Python often talk about how it is difficult to use Python for multithreaded work, pointing fingers at what is known as the 
global interpreter lock (affectionately referred to as the “GIL”) that prevents multiple threads of Python code from running simultaneously. 
Due to this, the threading module doesn’t quite behave the way you would expect it to if you’re not a Python developer and you are coming 
from other languages such as C++ or Java. It must be made clear that one can still write code in Python that runs concurrently or in parallel 
and make a stark difference resulting performance, as long as certain things are taken into consideration. If you haven’t read it yet, I suggest 
you take a look at Eqbal Quran’s [article on concurrency and parallelism](https://www.toptal.com/ruby/ruby-concurrency-and-parallelism-a-practical-primer)
in Ruby here on the Toptal blog.

In this Python concurrency tutorial, we will write a small Python script to download the top popular images from Imgur. We will start with a 
version that downloads images sequentially, or one at a time. As a prerequisite, you will have to register 
[an application on Imgur](https://api.imgur.com/oauth2/addclient). If you do 
not have an Imgur account already, please create one first.

The scripts in this tutorial has been tested with Python 3.4.2. With some changes, they should also run with Python 2 - urllib is what has 
changed the most between these two versions of Python.

