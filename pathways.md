We no longer host the Pathway pages on this website, the content has moved to the `learn-web3-dapp` repository at <https://github.com/figment-networks/learn-web3-dapp#-get-started>.

The workflow for Pathways is easy to follow:

- Using the `learn-web3-dapp` code repository, you will be running a local development server (`yarn dev`)
- Each Pathway is accessible from the main page displayed when the development server is running
- Once you begin a Pathway, your completion progress is saved in the local storage of your web browser
- Each Pathway has a similar goal: To guide you through using various code libraries to work with different parts of the web3 stack
- Many of the individual steps of a Pathway provide you with coding challenges, so that you can learn by doing
- You will need to complete the code as explained in the instructions before moving on to the next step
- All of the coding challenges have solutions which are blurred until you click on them, to prevent you from being stuck
- Once you have completed a Pathway, you should have a working knowledge of that protocol which will help you tackle larger projects

For less experienced developers or for anybody who wants to dive in without needing to set up a development environment, **[we recommend using Gitpod](https://gitpod.io/#https://github.com/figment-networks/learn-web3-dapp)** to work with the `learn-web3-dapp` project (this requires you to have a GitHub account to sign in). Gitpod can save you a considerable amount of time, as you don't have to install dependencies or set up any software - everything just works! Gitpod provides you with a fully functional Visual Studio Code editor running in your browser, so you can jump right in. This is what it looks like: 

![Gitpod UI](https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/assets/gitpod-firstlook.png?raw=true)

If you are an experienced developer, or if you prefer to use a different code editor, [cloning the repo locally](https://github.com/figment-networks/learn-web3-dapp#-clone-locally) is still possible, however you might encounter various system configuration issues depending on how your local environment is set up. It will be more difficult to develop on a Windows system than on Linux or MacOS, and there are also some known issues with developing Solana programs on Windows. It is strongly recommended to use the [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install#manual-installation-steps) (WSL) if you are running Windows 10. 

If you are cloning the repo locally, a minimum of **NodeJS version 14.17 is required**. Some of the Pathways will require use of the [Rust toolchain](https://rustup.rs/). If you are developing on Linux, you will want to make sure you have the [build-essential](https://itsfoss.com/build-essential-ubuntu/) package installed, as the `cc` linker is required for compiling Solana programs.