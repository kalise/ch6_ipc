# Share memory between containers
*Credits to the [Docker in Action](https://www.manning.com/books/docker-in-action) book.*

Linux provides a few tools for sharing memory between processes running on the same host machine. This form of inter-process communication (IPC) performs at system's memory speeds. It is usefull when the latency associated with network or a pipe is below the expectations, as for example, in the scientific calculation.

Docker creates a unique IPC namespace for each container by default. The IPC namespace prevents a process running in a container from accessing the memory on the host or in other containers. So, by default, two processes running in two different containers cannot share memory.

This image contains a simple C code implementing both a producer and a consumer talking each other via shared memory.

Clone the repo and build the image

    $ git clone https://github.com/kalise/shm
    $ cd shm
    $ docker build -t shm:latest .

Start a container as producer

    $ docker run -d --name producer shm:latest -producer

Start a second container as consumer

    $ docker run -d --name consumer shm:latest -consumer

The second should pull from the message queue and write the messages to the logs

    $ docker logs producer
    $ docker logs consumer

We can see the second container is not able to receive any message from the first one. This is because each of the two processes uses the same key to identify the shared memory but, thanks to the IPC isolation of the containers, they refer to a different memory areas.

If you need to run processes that communicate with shared memory in different containers, then you will need to join their IPC namespaces with the ``--ipc`` flag.

First, remove the old consumer

    $ docker rm -fv consumer

and create the consumer using the same IPC namespace of the producer

    $ docker run -d --name consumer --ipc container:producer shm:latest -consumer

The ``--ipc`` flag will create the consumer container in the same IPC namespace as the producer container.



