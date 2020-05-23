---
layout: post
title: A simple port scanner using Python
date: '2020-05-23 15:48:52'
tags: 
 - InfoSec
 - Python
 - Pentest
---

Port scanning is part of the first phase of a penetration test and allows you to find all network entry points available on a target system. It is analogous to knocking on doors to see if someone is home.

### 1. Overview

When it comes to scripting for security related stuffs, [Python](https://www.python.org/) is the go-to language. And, in the active [reconnaissance](https://en.wikipedia.org/wiki/Footprinting) stage of a [pentesting](https://en.wikipedia.org/wiki/Penetration_test), port scanning help us to understand the processes that are open and target them with the relevant attacking vectors.

We are going to take a look into a simple port scanner written in Python. The idea here is to understand how a port scanner functions, there are tried and tested tools like [Nmap](https://nmap.org/), which you shoud be using if you are looking for a robust port scanner. 

It takes less than ten lines of code to come up with a very basic version of a port scanner in Python. Once I show you how to do it, you should go through the **Refactoring** section to improve the quality of the program.

### 2. A very basic version of port scanner

Creat a file named *port_scanner.py*

Import [*sys*](https://docs.python.org/3/library/sys.html) package as we want to read some arguments from the command line. Another module which is the core to this is program is [*socket*](https://docs.python.org/3/library/socket.html), it helps to connect two nodes on a network, here it's our own system and the target node.
 
#### 2.1 Resolve a host name to an IP address
Say, you ran this script by providing google.com as the arugment, the output will show you the IP address of it.
```python
import sys
import socket

ip = socket.gethostbyname(sys.argv[1])
print("The target IP is %s"%ip)
```

#### 2.2 Scan for ports in a given range and see if it's open
The range of the ports to be scanned can be defined in the *for* loop, then we are cretaing a socket connection and trying to connect to the target host using it's IP and port.

```python
for port in range(20, 81):
	sock = socket.socket()
	open = sock.connect_ex((ip, port))
	if open == 0:
	    print("The port %d is open"%port) 
	sock.close()
``` 
#### 2.3 Execute the program
```shell
python3 port_scanner.py <host>
```
Your port scanner should be up and running.
#### 2.4 The missing pieces
a)  A network socket requires a socket family and a socket type. When we created the socket in section 2.2, we haven't mentioned these details, shouldn't we?

The [socket function](https://docs.python.org/3/library/socket.html#socket.socket) sets *AF_INET* (IPV4) as the address family and the *SOCK_STREAM* (TCP) as the type by default.

The modified socket creation code will look like this
```python
sock = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
```

b) The [connect_ex()](https://docs.python.org/3/library/socket.html#socket.socket.connect_ex) function takes a single argument - address - and connects to that address and returns error indicators, the value will be `0` if the operation succedded.

I said one agrument, but we are passing IP and port to the connect function, if you take a closer look, it's a tuple. [AF_INET](https://docs.python.org/3/library/socket.html#socket.AF_INET) address family is expecting a pair (host, port).

### 3. Refactoring
I will pose some questions and then will answer it with the solutions.

a) What will happen if I don't pass the host name while invoking the program?
The program will fail. To fix it we should add some validation before resolving the host name.
```python
if(len(sys.argv) < 2):
    print("Usage: python3 %s <host>"%sys.argv[0])
    sys.exit(1)
```
We are expecting two arguments in this program, one the file name and second is the host name, the file name can be accessed using the index `0`. We are stopping the execution of the program when the number of arguments are less than two.

b) What if the host name provided doesn't exist?
The program will exit by throwing [socket.gaierror](https://docs.python.org/3/library/socket.html#socket.gaierror) exception.
```
socket.gaierror: [Errno -3] Temporary failure in name resolution
```
We should capture the excetpion and show some user-friendly message.
```python
except socket.gaierror:
    print("Cannot resolve hostname!")
    sys.exit(1)
```
c) The scanning is taking too long to execute, when I interriupt the execution using `ctrl+c` what will happen?
Catch the built-in [KeyboardInterrupt](https://docs.python.org/3/library/exceptions.html) exception.
```python
except KeyboardInterrupt:
    print("I'm interrupted, may day, may day..")
    sys.exit(1)
```
d) The exceution is very slow, how can I improve the performance?
This is an I/O bound program. We can use threading to improve the performance.

You can read this [article](https://realpython.com/python-concurrency/) to add threading to the I/O task, I'm not going to explain about it in this article.

e) How do I know the execution time of the program?
We can make use of the timestamps. Import [datatime](https://docs.python.org/3/library/datetime.html) module and use [now()](https://docs.python.org/3/library/datetime.html#datetime.datetime.now) function to get the current timestamp.

```python
from datetime import datetime

print("Time is ",datetime.now())
```

### 4. Conclusion
We saw how to leverage in-built Python moudles to create a network port scanner from scratch. This program has lot of shortcomings, for doing real-world active footprinting use tools such as Nmap. **Don't reinvent the wheel!**

The improvments done as part of refactoring can be see [here](https://gist.github.com/pranavek/25f57ce24a2b2087772071f50b158b07) 



