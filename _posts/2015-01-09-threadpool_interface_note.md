---
layout: post
title: Python Threadpool Interface Demo
---

#### Introduction

threadpool: [https://pypi.python.org/pypi/threadpool](https://pypi.python.org/pypi/threadpool)

> Easy to use object-oriented thread pool framework
> 
> A thread pool is an object that maintains a pool of worker threads to perform time consuming operations in parallel. It assigns jobs to the threads by putting them in a work request queue, where they are picked up by the next available thread. This then performs the requested operation in the background and puts the results in another queue.
>
> The thread pool object can then collect the results from all threads from this queue as soon as they become available or after all threads have finished their work. It's also possible, to define callbacks to handle each result as it comes in.#### Transfer parameters

WTF! Shen me gui!

#### DEMO

    #!/usr/bin/env python
    # coding=utf8
    # author=ff0000team

    import threadpool as tp

    def test(url, passwd):
        print url, passwd

    if __name__ == '__main__':
        args = [
            (['http://xxx.com', '123'], {}),
            (['http://yyy.com', '213'], {}), 
            (['http://zzz.com', '321'], {})
        ]
        '''
        ``args_list`` contains the parameters for each invocation of callable.
        Each item in ``args_list`` should be either a 2-item tuple of the list of
        positional arguments and a dictionary of keyword arguments or a single,
        non-tuple argument.
        '''
        pool = tp.ThreadPool(2)
        reqs = tp.makeRequests(test, args)
        [pool.putRequest(req) for req in reqs]
        pool.wait()
