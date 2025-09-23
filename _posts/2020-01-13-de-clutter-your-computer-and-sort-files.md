---
layout: post
title: De - clutter  your computer and sort  files smartly using Python
date: '2020-01-13T15:08:00.000+05:30'
author: Middlewarebytes
sitemap:
  lastmod: 2022-05-31
  priority: 0.7
  changefreq: 'monthly'
  exclude: 'no'
tags:
- sort files
- declutter
- sorting
- python
modified_time: '2020-01-13T15:10:11.151+05:30'
---

If your your computer filled with downloaded files that sometimes take hours, if not minutes, to search through for a specific file that you need to send to someone who has asked for it?

I tend to download a lot of tutorials, PDFs, libraries, executables, zips, etc., for my work. Somedays, when I go through my system, I don't even remember where certain files are kept. It feels great to have a clean, organized computer with all the files sorted based on their extension, but it is not easy and requires a lot of discipline and patience, at least for me, as you can see the before and after situation of running this program.

There are tools and software (both paid and free, some are trial versions which put a limit on the number of files that can be sorted) available in the market, but I am not looking for those. Hence, I have created this simple Python program to declutter your box and sort files based on their extension.

This requires Python to run, so you need to install Python on your machine first to execute this. I am still working on an executable version of this program so you won't need to install Python, but that's still a work in progress. I will definitely put up a link to the program once it is ready.

For the time being, if you want to use this program, you can download Python from the official site [Here](https://www.python.org/downloads/).

```
## Declutter Program
###############################################################################
##This program needs python to be installed on your computer in order to run ##          
##Modified Date: 10.01.2020                                                  ##
##File name : declutter.py                                                   ##
##Usage: py declutter.py                                                     ## 
###############################################################################
   

   ##   Import the important libraries required to perform the task 
   
   import os
   import shutil
   
   ## Write the name of the directory here that you want to sort and declutter, below is my desktop which was the most unorganized place in my laptop :). You can use your own
   
   path = "C:/Users/pinak.mazumdar/Desktop"
   
   ## This will create a list of all the files in the given path
   
   filelist_ = os.listdir(path)
   
   # This will go through each and every file 
   
   for file_ in filelist_:
   name, ext = os.path.splitext(file_)
   
   # This is going to store the extension type
   ext = ext[1:]
   
   # This forces the next iteration
   # if it is the directory
   
   if ext == '':
     continue
	 
	 # All the files will be moved to the directories , named with the extension
	 
	 if os.path.exists(path+'/'+ext):
	   shutil.move(path+'/'+file_, path+'/'+ext+'/'+file_)
	 # This will create a new directory , if it does not exist.
	     else:
		   os.makedirs(path+'/'+ext)
		   shutil.move(path+'/'+file_, path+'/'+ext+'/'+file_)
```