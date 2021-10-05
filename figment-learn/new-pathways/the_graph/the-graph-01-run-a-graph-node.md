# Why run a local graph node?

Many Developers are familiar with the idea of running a local webserver in the early stages of their project. This helps them to rapidly design, write and test their code without needing to rely on external servers. Following this principle, we're going to do the same for the **The Graph Protocol**. A local graph node can be seen as loosely equivalent to a webserver, it will listen for requests and respond to the clients which connect to it.

# How to setup a local graph node?

If you have followed the setup instructions, you should now have:

- A working installation of docker
- An API key for your Alchemy application

Now we can:

- Run a containerized version of a Graph node
- Connect it to Ethereum mainnet using Alchemy as a provider

{% hint style="warning" %}
Linux and macOS users can run a script to ensure the necessary software is installed: `sudo ./script.sh`
{% endhint %}

From the root directory of the project, run:

```text
cd docker
ETHEREUM_RPC=mainnet:https://eth-mainnet.alchemyapi.io/v2/<API-KEY> docker compose up
```

If you encounter an error such as:

```text
FileNotFoundError: [Errno 2] No such file or directory
```

Make sure that you have the latest version of Docker Desktop and that it is currently running on your system.

Similarly to a local webserver, our local Graph node is listening for connections on **localhost**, however the default port for the 
Graph node is **8020**. `docker compose` will output logging information to the terminal such as the following:

```text
Starting docker_ipfs_1     ... done
Starting docker_postgres_1 ... done
Starting docker_graph-node_1 ... done
Attaching to docker_postgres_1, docker_ipfs_1, docker_graph-node_1
```

A Graph node comes with two storage solutions and one API endpoint.

- A ready to use IPFS swarm: IPFS is a distributed system for storing and accessing files, websites, applications, and data.
- A Postgres database: A popular free and open-source **RDBMS** (**R**elational **D**ata**B**ase **M**anagement **S**ystem)
- A GraphQL API endpoint (more on this later).

# Time to verify your work

Now, it's time for you to verify that you have followed the instructions carefully, click on the button **Test local graph node** to detect the presence of a local graph node listening on **port 8020**!

If the node is not running, you will see the following error modal:

![](../assets/the_graph-no-local-node.png)


{% hint style="info" %}
You can [**join us on Discord**](https://discord.gg/fszyM7K), if you have questions or want help completing the tutorial.
{% endhint %}

# Conclusion

Nice! You made it. A local Graph node is running on your computer. A Graph node without a subgraph to index sounds like a forest without any trees, it doesn't make sense! Click Next Step to fill this gap by creating a subgraph.
