---
description: Learn how to build a nameservice application (continued)
---

# go.mod and Makefile

## go.mod and Makefile <a id="go-mod-and-makefile"></a>

### `Makefile` <a id="makefile"></a>

Help users build your application by writing a `./Makefile` in the root directory that includes common commands. The scaffolding tool has created a generic makefile that you will be able to use:

> _NOTE_: The below Makefile contains some of same commands as the Cosmos SDK and Tendermint Makefiles.

```javascript
PACKAGES=$(shell go list ./... | grep -v '/simulation')

VERSION := $(shell echo $(shell git describe --tags) | sed 's/^v//')
COMMIT := $(shell git log -1 --format='%H')

ldflags = -X github.com/cosmos/cosmos-sdk/version.Name=NameService \
    -X github.com/cosmos/cosmos-sdk/version.ServerName=nsd \
    -X github.com/cosmos/cosmos-sdk/version.ClientName=nscli \
    -X github.com/cosmos/cosmos-sdk/version.Version=$(VERSION) \
    -X github.com/cosmos/cosmos-sdk/version.Commit=$(COMMIT) 

BUILD_FLAGS := -ldflags '$(ldflags)'

include Makefile.ledger
all: install

install: go.sum
        @echo "--> Installing nsd & nscli"
        @go install -mod=readonly $(BUILD_FLAGS) ./cmd/nsd
        @go install -mod=readonly $(BUILD_FLAGS) ./cmd/nscli

go.sum: go.mod
        @echo "--> Ensure dependencies have not been modified"
        GO111MODULE=on go mod verify

test:
    @go test -mod=readonly $(PACKAGES)
```

#### How about including Ledger Nano S support? <a id="how-about-including-ledger-nano-s-support"></a>

This requires a few small changes:

* Create a file `Makefile.ledger` with the following content:

```javascript
LEDGER_ENABLED ?= true

build_tags =
ifeq ($(LEDGER_ENABLED),true)
  ifeq ($(OS),Windows_NT)
    GCCEXE = $(shell where gcc.exe 2> NUL)
    ifeq ($(GCCEXE),)
      $(error gcc.exe not installed for ledger support, please install or set LEDGER_ENABLED=false)
    else
      build_tags += ledger
    endif
  else
    UNAME_S = $(shell uname -s)
    ifeq ($(UNAME_S),OpenBSD)
      $(warning OpenBSD detected, disabling ledger support (https://github.com/cosmos/cosmos-sdk/issues/1988))
    else
      GCC = $(shell command -v gcc 2> /dev/null)
      ifeq ($(GCC),)
        $(error gcc not installed for ledger support, please install or set LEDGER_ENABLED=false)
      else
        build_tags += ledger
      endif
    endif
  endif
endif
```

* Add `include Makefile.ledger` at the beginning of the Makefile:

```javascript
LEDGER_ENABLED ?= true

build_tags =
ifeq ($(LEDGER_ENABLED),true)
  ifeq ($(OS),Windows_NT)
    GCCEXE = $(shell where gcc.exe 2> NUL)
    ifeq ($(GCCEXE),)
      $(error gcc.exe not installed for ledger support, please install or set LEDGER_ENABLED=false)
    else
      build_tags += ledger
    endif
  else
    UNAME_S = $(shell uname -s)
    ifeq ($(UNAME_S),OpenBSD)
      $(warning OpenBSD detected, disabling ledger support (https://github.com/cosmos/cosmos-sdk/issues/1988))
    else
      GCC = $(shell command -v gcc 2> /dev/null)
      ifeq ($(GCC),)
        $(error gcc not installed for ledger support, please install or set LEDGER_ENABLED=false)
      else
        build_tags += ledger
      endif
    endif
  endif
endif
```

### `go.mod` <a id="go-mod"></a>

Golang has a few dependency management tools. In this tutorial you will be using [`Go Modules`](https://github.com/golang/go/wiki/Modules). `Go Modules` uses a `go.mod` file in the root of the repository to define what dependencies the application needs. Cosmos SDK apps currently depend on specific versions of some libraries. The below manifest contains all the necessary versions. To get started replace the contents of the `./go.mod` file with the `constraints` and `overrides` below:

> _NOTE_: If you are following along in your own repo you will need to change the module path to reflect that \(`github.com/{ .Username }/{ .Project.Repo }`\).

* You will have to run `go get ./...` to get all the modules the application is using. This command will get the dependency version stated in the `go.mod` file.
* If you would like to use a specific version of a dependency then you have to run `go get github.com/<github_org>/<repo_name>@<version>`

### Building the app <a id="building-the-app"></a>

```javascript
# Install the app into your $GOBIN
make install

# Now you should be able to run the following commands:
nsd help
nscli help
```

#### Congratulations, you have finished your nameservice application! Try running and interacting with it! <a id="congratulations-you-have-finished-your-nameservice-application-try-running-and-interacting-with-it"></a>

If you had any difficulties following this tutorial or simply want to discuss Cosmos tech with us you can [**join our community today**](https://discord.gg/fszyM7K)!

