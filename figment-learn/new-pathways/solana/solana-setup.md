The following software is required to set up and complete the Solana Pathway

* [Node.js v14.17.0 or higher installed](https://nodejs.org/)
* [yarn installed](https://yarnpkg.com/getting-started/install)
* [git installed](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

Start by cloning the repository and installing the dependencies with yarn :

```bash
git clone https://github.com/figment-networks/learn-web3-dapp.git
cd learn-web3-dapp
yarn
```

{% hint style="danger" %}
There are known compatibility issues with Microsoft Windows and also Apple Silicon products!
{% endhint %}

> Please **STOP** and read the following information carefully:
>
> **Windows Users:** The Rust BPF toolchain _is not available for Windows_. This means that compiling Solana programs cannot be performed in a Windows development environment. You must install [Docker Desktop](https://learn.figment.io/network-documentation/extra-guides/docker-setup-for-windows) and [WSL](https://docs.microsoft.com/en-us/windows/wsl/install-win10#manual-installation-steps) - then you will need to clone the `learn-web3-dapp` repository again and install Rust and the Solana CLI, all inside of the WSL filesystem.
>
> To access the filesystem of your [installed Linux distribution](https://docs.microsoft.com/en-us/windows/wsl/install-win10#step-6---install-your-linux-distribution-of-choice) for WSL :
> Run the command [`wsl`](https://docs.microsoft.com/en-us/windows/wsl/reference) from a `cmd.exe` or PowerShell terminal. It is also important to make sure your PATH in the Windows Subsystem for Linux environment includes the location of the Solana release you have installed, such as `:PATH="~/.local/share/solana/install/active_release/bin:$PATH"`.
> More information on viewing and setting the PATH in Linux is [available here](https://opensource.com/article/17/6/set-path-linux).
>
> macOS Users: If you are using any of the Apple Silicon products (M1 processor), you may need to build from source. [Refer to this GitHub PR for more information](https://github.com/solana-labs/solana/pull/16346/)

---------------------------

# Set your API key

Create an `.env.local` file at the root of the directory. Copy and paste the contents of the existing `.env.example` into the new file and save it to disk (alternatively, you can rename `.env.example`).

The value for `DATAHUB_SOLANA_API_KEY` can be found on the [DataHub Services Dashboard](https://datahub.figment.io/services/solana). Click on the Solana icon in the list of available protocols and then copy your key as shown below. You can now paste your personal API key into `.env.local` . This will authenticate you and enable you to make JSON-RPC requests to the Solana cluster via DataHub.

![](../../../.gitbook/assets/solana-setup-00.gif)

{% hint style="info" %}
[**Join us on Discord**, if you encounter any issues with the tutorial or have any questions!](https://discord.gg/fszyM7K)
{% endhint %}

---------------------------

# Start the server

With the API key in place, save the `.env.local` file and start the React interface with :

```bash
yarn dev
```

Once the development server loads and compiles the application, open your default browser and go to `http://localhost:3000` :

![](../../../.gitbook/assets/pathway-home.gif)

{% hint style="warning" %}
Did you know you can change the port? By default Next.js uses port 3000, but if you have another service already running on that port, use the `--port` flag, like `yarn dev --port 1122`.
{% endhint %}

---------------------------

# Next

You can now move ahead to the next step by clicking on the "Next" button below on the right. There are also links to the instructions for each step on the UI.
