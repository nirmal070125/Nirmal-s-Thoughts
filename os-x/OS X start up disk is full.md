OS X start up disk is full?
=========

I have faced a situation where my OS X start up disk reached 100% usage and machine become drastically slow. I realized that I need to find the culprit files for this growth and delete/back-up them.

I used following command on my terminal to find out the files which are greater than 1GB on my start up disk.

```sh
sudo find -x / -type f -size +1G
```

From the above command I found that some of my VirtualBox images caused this growth and then backed them up to some other partition.

Next, I need to find a way to make my VirtualBox to not use my start-up disk. :-)
