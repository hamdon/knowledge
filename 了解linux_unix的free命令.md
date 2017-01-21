free is a command which can give us valuable information on available RAM in Linux machine. But many new Linux users and admins misinterpret it’s output. In this post we will walk through it’s output format and show you actual free RAM.

###**FREE COMMAND EXAMPLES**
**Example 1**: Display RAM details in Linux machine
> free

Output:
```
    	       total        used      free   shared   buffers   cached
      	Mem:    8027952    4377300   3650652   0      103648    1630364
    	-/+ buffers/cache: 2643288   5384664
       Swap:  15624188    608948   15015240
``` 
 Let us see what this table for you.

**Line 1**: Indicates Memory details like total available RAM, used RAM, Shared RAM, RAM used for buffers, RAM used of caching content.
**Line 2**: Indicates total buffers/Cache used and free.
**Line 3**: Indicates total swap memory available, used swap and free swap memory size available.

Let us dig more in to these lines to better understand it as Linux user prospective.

**Line 1:Mem: 8027952 4377300 3650652 0 103648 1630364**

**8027952** : Indicates memory/physical RAM available for your machine. These numbers are in KB’s
**4377300** : Indicates memory/RAM used by system. This include even buffers and cached data size as well.
**3650652** : Indicates Total RAM free and available for new process to run.
**0** :       Indicates shared memory. This column is obsolete and may be removed in future releases of free.
**103648** : Indicates total RAM buffered by different applications in Linux
**1630364** : Indicates total RAM used for Caching of data for future purpose.
Puzzled with buffers and cache?
###**WHAT IS THE DIFFERENCE BETWEEN BUFFERS AND CACHE?**
A buffer is a temporary location to store data for a particular application and this data is not used by any other application. This is similar to bandwidth concept. When you try to send burst of data through network, if your network card is capable of sending less data, it will keep these huge amounts of data in buffer so that it can send data constantly in lesser speeds. In other hand Cache is a memory location to store frequently used data for faster access. Other difference between a buffer and a cache is that cache can be used multiple times where as buffer is used single time. And both are temporary store for your data processing.
**Line 2: -/+ buffers/cache: 2643288 5384664**
```
2643288 : This is actual size of used RAM which we get from RAM used -(buffers + cache)
	A bit of mathematical calculation
	Used RAM = +4377300
	Used Buffers = -103648
	Used Cache = -1630364
```
>Actual Total used RAM is 4377300 -(103648+1630364)= 2643288

Then why my Linux machine is showing 4377300 as used RAM. This is because Linux counts cached RAM, Buffered RAM to this used RAM. But in future if any application want to use these buffers/cache, Linux will free it for you. To know more about this, visit [this site](http://www.linuxatemyram.com/).

**5384664** : Indicates actual total RAM available, we get to this number by subtracting actual RAM used from total RAM available in the system.
```
    Total RAM = +8027952
	actual used RAM = -2643288
	Total actual available RAM = 5384664
```
So from today on words don’t complain that Linux ate your RAM, it’s our understanding of free command output which is the culprit and the teacher who thought us Linux. If any one asks what is the free RAM available, we have to give this number(**5384664**) instead of first line number(**4377300**) for free RAM available in your machine.
**Line 3: Swap: 15624188 608948 15015240**
This line indicates swap details like total SWAP size, used as well as free SWAP.

Swap is a virtual memory created on HDD to increase RAM size virtually. To know more about swap click here for creating swap partition and here for swap file creation.

**Example2**: Display RAM in human readable formats like in KB’s, MB’s, GB’s, TB’s
```
	free -k
	free -m
	free -g
	free --tera
```
Output:
```
root@linuxnix.com:/home/surendra# free -k
	total used free shared buffers cached
	Mem: 8027952 5323952 2704000 0 116876 1626940
	-/+ buffers/cache: 3580136 4447816
	Swap: 15624188 603792 15020396
	root@linuxnix.com:/home/surendra# free -m
	total used free shared buffers cached
	Mem: 7839 5197 2642 0 114 1588
	-/+ buffers/cache: 3495 4344
	Swap: 15257 589 14668
	root@linuxnix.com:/home/surendra# free -g
	total used free shared buffers cached
	Mem: 7 5 2 0 0 1
	-/+ buffers/cache: 3 4
	Swap: 14 0 14
	root@linuxnix.com:/home/surendra# free --tera
	total used free shared buffers cached
	Mem: 0 0 0 0 0 0
	-/+ buffers/cache: 0 0
	Swap: 0 0 0
```
**Note**: If you observe with option –tera, the RAM size shows as 0, this is because my RAM is just 8GB and the output of free command will not support floating point numbers.

**Example3**: My actual RAM is 8GB but it is showing 7GB when using -g option, whats wrong with my RAM. This is because free command by default counts 1024 as power instead of 1000. To see difference use –si

With out –si
>free -g
Output:
```
	total used free shared buffers cached
	Mem: 7 5 2 0 0 1
	-/+ buffers/cache: 3 4
	Swap: 14 0 14
```
with –si
  >free --si -g

Output:
```
	total used free shared buffers cached
	Mem: 8 5 2 0 0 1
	-/+ buffers/cache: 3 4
	Swap: 15 0 15
```
**Example 4**: Want to see combined size of both swap as well as RAM use -t option
>free -t

Output:
```
	total used free shared buffers cached
	Mem: 8027952 5369012 2658940 0 117228 1634396
	-/+ buffers/cache: 3617388 4410564
	Swap: 15624188 603788 15020400
	Total: 23652140 5972800 17679340
```
**Example 5**: I want to see continuous varying of RAM usage for every second, use -s option to mention number of seconds
>free -s 1
 
 Output:
```
total used free shared buffers cached
	Mem: 8027952 5370220 2657732 0 117376 1635144
	-/+ buffers/cache: 3617700 4410252
	Swap: 15624188 603788 15020400 total used free shared buffers cached
	Mem: 8027952 5367244 2660708 0 117392 1635272
	-/+ buffers/cache: 3614580 4413372
	Swap: 15624188 603788 15020400 total used free shared buffers cached
	Mem: 8027952 5367556 2660396 0 117392 1635272
	-/+ buffers/cache: 3614892 4413060
	Swap: 15624188 603788 15020400 total used free shared buffers cached
	Mem: 8027952 5367388 2660564 0 117392 1635272
	-/+ buffers/cache: 3614724 4413228
	Swap: 15624188 603788 15020400
```
This is not a good way to display continuous output, we can use watch command to do iterations
>	watch free

In out next post we will see other RAM details.