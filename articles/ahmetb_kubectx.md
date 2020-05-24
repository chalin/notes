ahmetb/kubectx

###    README.md

This repository provides both ` kubectx ` and ` kubens ` tools. Purpose of this project is to provide an utility and facilitate discussion about how ` kubectl `can manage contexts better.

# [(L)](https://github.com/ahmetb/kubectx#kubectx1)kubectx(1)

kubectx is an utility to manage and switch between kubectl(1) contexts.

	USAGE:
	  kubectx                   : list the contexts
	  kubectx <NAME>            : switch to context
	  kubectx -                 : switch to the previous context
	  kubectx <NEW_NAME>=<NAME> : create alias for context
	  kubectx -h,--help         : show this message

Purpose of this project is to provide an utility and facilitate discussion about how ` kubectl ` can manage contexts better.

### [(L)](https://github.com/ahmetb/kubectx#usage)Usage

$ kubectx minikube
Switched to context "minikube".
$ kubectx -
Switched to context "oregon".
$ kubectx -
Switched to context "minikube".
$ kubectx dublin=gke_ahmetb_europe-west1-b_dublin
Context "dublin" set.
Aliased "gke_ahmetb_europe-west1-b_dublin" as "dublin".

` kubectx ` supports `Tab` completion on bash/zsh shells to help with long context names. You don't have to remember full context names anymore.

* * *

# [(L)](https://github.com/ahmetb/kubectx#kubens1)kubens(1)

kubens is an utility to switch between Kubernetes namespaces.

	USAGE:
	  kubens                    : list the namespaces
	  kubens <NAME>             : change the active namespace
	  kubens -                  : switch to the previous namespace
	  kubens -h,--help          : show this message

### [(L)](https://github.com/ahmetb/kubectx#usage-1)Usage

$ kubens kube-system
Context "test" set.
Active namespace is "kube-system".
$ kubens -
Context "test" set.
Active namespace is "default".
` kubens ` also supports `Tab` completion on bash/zsh shells.

* * *

## [(L)](https://github.com/ahmetb/kubectx#installation)Installation

**For macOS:**
> Use > [> Homebrew](https://brew.sh/)>  package manager:

	 brew tap ahmetb/kubectx https://github.com/ahmetb/kubectx.git
	 brew install kubectx

> this will also set up bash/zsh completion scripts automatically.

Running ` brew install ` with ` --with-short-names ` will install tools with names` kctx ` and ` kns ` to prevent prefix collision with ` kubectl ` name.

**Other platforms:**

> Download the ` kubectx `>  script, make it executable and add it to your PATH. You can also install bash/zsh > [> completion scripts](https://github.com/ahmetb/kubectx/blob/master/completion)>  manually.

* * *

Disclaimer: This is not an official Google product.