# Why running a local graph node?

Developers are familiar with the idea of running a local webserver at the early stage of their work. Following the same principe we're going to do the same for the **THE GRAPH**. To sum up local graph node can be seen as a local webserver.

# How to setup a local graph node?

If you have follow up the setup, you should now have:

- A working installation of docker
- An API key for your alchemy application

Now we can:

- Run a containerized version of a graph node
- Connect it to Ethereum mainnet using alchemy as a provider

From the root directory of the project, run:

```text
cd docker
sudo ETHEREUM_RPC=mainnet:https://eth-mainnet.alchemyapi.io/v2/<API-KEY> docker-compose up
```

As for local webserver, local graph node are listening on **localhost**. Contrary to most of local webserver, by default the opening port is **8020**.

A graph node is deliver with two storages' solutions and one API endpoints.

- An ready to use IPFS swarm: A cutting-edge protocol emulating Distributed File's system.
- A Postgres database: A popular free and open-source **RDBMS**
- An GraphQL API endpoints (more on this later).

{% hint style="warning" %}
Linux's users have to run a script before all: `sudo ./script.sh`
{% endhint %}

# Time to verify your work

Now, it's time for you to verify if you have follow carefully our instruction, click on the button `Test local graph node` and see if magic happen.

{% hint style="info" %}
You can [**join us on Discord**](https://discord.gg/fszyM7K), if you have questions or want help completing the tutorial.
{% endhint %}

# Conclusion

Nice you made it, a local graph node is running on your computer. A graph node without subgraph sound like a forest without any trees it doesn't make sense. Click next to fill this gap.
