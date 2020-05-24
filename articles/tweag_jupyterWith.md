tweag/jupyterWith

# [![](data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' class='octicon octicon-link js-evernote-checked' viewBox='0 0 16 16' version='1.1' width='16' height='16' aria-hidden='true' data-evernote-id='246'%3e%3cpath fill-rule='evenodd' d='M4 9h1v1H4c-1.5 0-3-1.69-3-3.5S2.55 3 4 3h4c1.45 0 3 1.69 3 3.5 0 1.41-.91 2.72-2 3.25V8.59c.58-.45 1-1.27 1-2.09C10 5.22 8.98 4 8 4H4c-.98 0-2 1.22-2 2.5S3 9 4 9zm9-3h-1v1h1c1 0 2 1.22 2 2.5S13.98 12 13 12H9c-.98 0-2-1.22-2-2.5 0-.83.42-1.64 1-2.09V6.25c-1.09.53-2 1.84-2 3.25C6 11.31 7.55 13 9 13h4c1.45 0 3-1.69 3-3.5S14.5 6 13 6z' data-evernote-id='1024' class='js-evernote-checked'%3e%3c/path%3e%3c/svg%3e)](https://github.com/tweag/jupyterWith#jupyterwith)JupyterWith

[![kernels.png](../_resources/70cf697cb88fc721c1935a95db4130b5.png)](https://github.com/tweag/jupyterWith/blob/master/kernels.png)

This repository provides a Nix-based framework for the definition of declarative and reproducible Jupyter environments. These environments include JupyterLab - configurable with extensions - the classic notebook, and configurable Jupyter kernels.

In practice, a Jupyter environment is defined in a single `shell.nix` file which can be distributed together with a notebook as a self-contained reproducible package.

These kernels are currently included by default:

- [IPython](https://github.com/ipython/ipykernel)
- [IHaskell](https://github.com/gibiansky/IHaskell) (long build time)
- [CKernel](https://github.com/brendan-rius/jupyter-c-kernel)
- [IRuby](https://github.com/SciRuby/iruby)
- [Juniper RKernel](https://github.com/JuniperKernel/JuniperKernel) (limited jupyterlab support)
- [Ansible Kernel](https://github.com/ansible/ansible-jupyter-kernel)
- [Xeus-Cling CPP](https://github.com/QuantStack/xeus-cling) (experimental, not yet configurable with packages, long build time)
- [IJavascript](https://github.com/n-riesco/ijavascript) (not yet configurable with packages)
- [gophernotes](https://github.com/gopherdata/gophernotes) (not yet configurable with packages)

Example notebooks are [here](https://github.com/tweag/jupyterWith/blob/master/example).

## [![](data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' class='octicon octicon-link js-evernote-checked' viewBox='0 0 16 16' version='1.1' width='16' height='16' aria-hidden='true' data-evernote-id='247'%3e%3cpath fill-rule='evenodd' d='M4 9h1v1H4c-1.5 0-3-1.69-3-3.5S2.55 3 4 3h4c1.45 0 3 1.69 3 3.5 0 1.41-.91 2.72-2 3.25V8.59c.58-.45 1-1.27 1-2.09C10 5.22 8.98 4 8 4H4c-.98 0-2 1.22-2 2.5S3 9 4 9zm9-3h-1v1h1c1 0 2 1.22 2 2.5S13.98 12 13 12H9c-.98 0-2-1.22-2-2.5 0-.83.42-1.64 1-2.09V6.25c-1.09.53-2 1.84-2 3.25C6 11.31 7.55 13 9 13h4c1.45 0 3-1.69 3-3.5S14.5 6 13 6z' data-evernote-id='1041' class='js-evernote-checked'%3e%3c/path%3e%3c/svg%3e)](https://github.com/tweag/jupyterWith#getting-started)Getting started

[Nix](https://nixos.org/nix/) must be installed in order to use JupyterWith. A simple JupyterLab environment with kernels can be defined in a `shell.nix` file such as:

let  jupyter  =  import (builtins.fetchGit { url  =  https://github.com/tweag/jupyterWith; rev  =  "";

}) {}; iPython  =  jupyter.kernels.iPythonWith { name  =  "python"; packages  =  p: with  p; [ numpy ];

}; iHaskell  =  jupyter.kernels.iHaskellWith { name  =  "haskell"; packages  =  p: with  p; [ hvega  formatting ];

}; jupyterEnvironment  =  jupyter.jupyterlabWith { kernels  = [ iPython  iHaskell ];

};in  jupyterEnvironment.env
JupyterLab can then be started by running:

	nix-shell --command "jupyter lab"

This can take a while, especially when it is run for the first time because all dependencies of JupyterLab have to be downloaded, built and installed. Subsequent runs are instantaneous for the same environment, or much faster even when some packages or kernels are changed, since a lot will already be cached.

### [![](data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' class='octicon octicon-link js-evernote-checked' viewBox='0 0 16 16' version='1.1' width='16' height='16' aria-hidden='true' data-evernote-id='248'%3e%3cpath fill-rule='evenodd' d='M4 9h1v1H4c-1.5 0-3-1.69-3-3.5S2.55 3 4 3h4c1.45 0 3 1.69 3 3.5 0 1.41-.91 2.72-2 3.25V8.59c.58-.45 1-1.27 1-2.09C10 5.22 8.98 4 8 4H4c-.98 0-2 1.22-2 2.5S3 9 4 9zm9-3h-1v1h1c1 0 2 1.22 2 2.5S13.98 12 13 12H9c-.98 0-2-1.22-2-2.5 0-.83.42-1.64 1-2.09V6.25c-1.09.53-2 1.84-2 3.25C6 11.31 7.55 13 9 13h4c1.45 0 3-1.69 3-3.5S14.5 6 13 6z' data-evernote-id='1047' class='js-evernote-checked'%3e%3c/path%3e%3c/svg%3e)](https://github.com/tweag/jupyterWith#using-jupyterlab-extensions)Using JupyterLab extensions

Lab extensions can be added by generating a JupyterLab frontend directory. This can be done by running `nix-shell` from the folder with the `shell.nix`file and then using the `generate-directory` executable that is available from inside the shell.

$ generate-directory [EXTENSIONS]
$ generate-directory jupyterlab-ihaskell jupyterlab_bokeh

This will generate a folder called `jupyterlab` that can then be passed to`jupyterWith`. With extensions, the example above becomes:

let  jupyter  =  import (builtins.fetchGit { url  =  https://github.com/tweag/jupyterWith; rev  =  "";

}); iPython  =  jupyter.kernels.iPythonWith { name  =  "python kernel name"; packages  =  p: with  p; [ numpy ];

}; iHaskell  =  jupyter.kernels.iHaskellWith { name  =  "haskell kernel name"; packages  =  p: with  p; [ hvega  formatting ];

}; jupyterEnvironment  =  jupyter.jupyterlabWith { kernels  = [ iPython  iHaskell ]; ## The generated directory goes here  directory  =  ./jupyterlab;

};in  jupyterEnvironment.env

Another option is to use the impure `mkDirectoryWith` Nix function that comes with this repo:

{ jupyterEnvironment  =  jupyter.jupyterlabWith { kernels  = [ iPython  iHaskell ]; ## The directory is generated here  directory  =  mkDirectoryWith { extensions  = [ "jupyterlab-ihaskell"  "jupyterlab_bokeh" ];

};
};
}

In this case, you must make sure that sandboxing is disabled in your Nix configuration. Newer Nix versions have it enabled by default. Sandboxing can be disabled:

- either by running `nix-shell --option build-use-sandbox false`; or
- by setting `build-use-sandbox = false` in `/etc/nix/nix.conf`.

The first option may require using `sudo`, depending on the version of Nix.

### [![](data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' class='octicon octicon-link js-evernote-checked' viewBox='0 0 16 16' version='1.1' width='16' height='16' aria-hidden='true' data-evernote-id='249'%3e%3cpath fill-rule='evenodd' d='M4 9h1v1H4c-1.5 0-3-1.69-3-3.5S2.55 3 4 3h4c1.45 0 3 1.69 3 3.5 0 1.41-.91 2.72-2 3.25V8.59c.58-.45 1-1.27 1-2.09C10 5.22 8.98 4 8 4H4c-.98 0-2 1.22-2 2.5S3 9 4 9zm9-3h-1v1h1c1 0 2 1.22 2 2.5S13.98 12 13 12H9c-.98 0-2-1.22-2-2.5 0-.83.42-1.64 1-2.09V6.25c-1.09.53-2 1.84-2 3.25C6 11.31 7.55 13 9 13h4c1.45 0 3-1.69 3-3.5S14.5 6 13 6z' data-evernote-id='1060' class='js-evernote-checked'%3e%3c/path%3e%3c/svg%3e)](https://github.com/tweag/jupyterWith#changes-to-the-default-package-sets)Changes to the default package sets

The kernel environments rely on the default package sets that are provided by the Nixpkgs repository that is defined in the [nix folder](https://github.com/tweag/jupyterWith/blob/master/nix). These package sets can be modified using overlays, for example to add a new Python package from PIP. You can see examples of this in the[`./nix/python-overlay.nix`](https://github.com/tweag/jupyterWith/blob/master/nix/python-overlay.nix) and[`./nix/haskell-overlay.nix`](https://github.com/tweag/jupyterWith/blob/master/nix/haskell-overlay.nix) files. You can also modify the package set directly in the `shell.nix` file, as demonstrated in [this](https://github.com/tweag/jupyterWith/blob/master/example/Haskell/bayesMonad/shell.nix)example that adds a new Haskell package to the package set.

### [![](data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' class='octicon octicon-link js-evernote-checked' viewBox='0 0 16 16' version='1.1' width='16' height='16' aria-hidden='true' data-evernote-id='250'%3e%3cpath fill-rule='evenodd' d='M4 9h1v1H4c-1.5 0-3-1.69-3-3.5S2.55 3 4 3h4c1.45 0 3 1.69 3 3.5 0 1.41-.91 2.72-2 3.25V8.59c.58-.45 1-1.27 1-2.09C10 5.22 8.98 4 8 4H4c-.98 0-2 1.22-2 2.5S3 9 4 9zm9-3h-1v1h1c1 0 2 1.22 2 2.5S13.98 12 13 12H9c-.98 0-2-1.22-2-2.5 0-.83.42-1.64 1-2.09V6.25c-1.09.53-2 1.84-2 3.25C6 11.31 7.55 13 9 13h4c1.45 0 3-1.69 3-3.5S14.5 6 13 6z' data-evernote-id='1063' class='js-evernote-checked'%3e%3c/path%3e%3c/svg%3e)](https://github.com/tweag/jupyterWith#building-the-docker-images)Building the Docker images

One can easily generate Docker images from Jupyter environments defined with JupyterWith with a `docker.nix` file:

let  jupyter  =  import (builtins.fetchGit { url  =  https://github.com/tweag/jupyterWith; rev  =  "";

}); jupyterEnvironment  =  jupyter.jupyterlabWith {

};in  jupyter.mkDockerImage { name  =  "jupyter-image"; jupyterlab  =  jupyterEnvironment;

}
`nix-build docker.nix` builds the image and it can be passed to Docker with:

	$ cat result | docker load
	$ docker run -v $(pwd)/example:/data -p 8888:8888 jupyter-image:latest

## [![](data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' class='octicon octicon-link js-evernote-checked' viewBox='0 0 16 16' version='1.1' width='16' height='16' aria-hidden='true' data-evernote-id='251'%3e%3cpath fill-rule='evenodd' d='M4 9h1v1H4c-1.5 0-3-1.69-3-3.5S2.55 3 4 3h4c1.45 0 3 1.69 3 3.5 0 1.41-.91 2.72-2 3.25V8.59c.58-.45 1-1.27 1-2.09C10 5.22 8.98 4 8 4H4c-.98 0-2 1.22-2 2.5S3 9 4 9zm9-3h-1v1h1c1 0 2 1.22 2 2.5S13.98 12 13 12H9c-.98 0-2-1.22-2-2.5 0-.83.42-1.64 1-2.09V6.25c-1.09.53-2 1.84-2 3.25C6 11.31 7.55 13 9 13h4c1.45 0 3-1.69 3-3.5S14.5 6 13 6z' data-evernote-id='1068' class='js-evernote-checked'%3e%3c/path%3e%3c/svg%3e)](https://github.com/tweag/jupyterWith#contributing)Contributing

### [![](data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' class='octicon octicon-link js-evernote-checked' viewBox='0 0 16 16' version='1.1' width='16' height='16' aria-hidden='true' data-evernote-id='252'%3e%3cpath fill-rule='evenodd' d='M4 9h1v1H4c-1.5 0-3-1.69-3-3.5S2.55 3 4 3h4c1.45 0 3 1.69 3 3.5 0 1.41-.91 2.72-2 3.25V8.59c.58-.45 1-1.27 1-2.09C10 5.22 8.98 4 8 4H4c-.98 0-2 1.22-2 2.5S3 9 4 9zm9-3h-1v1h1c1 0 2 1.22 2 2.5S13.98 12 13 12H9c-.98 0-2-1.22-2-2.5 0-.83.42-1.64 1-2.09V6.25c-1.09.53-2 1.84-2 3.25C6 11.31 7.55 13 9 13h4c1.45 0 3-1.69 3-3.5S14.5 6 13 6z' data-evernote-id='1070' class='js-evernote-checked'%3e%3c/path%3e%3c/svg%3e)](https://github.com/tweag/jupyterWith#kernels)Kernels

New kernels are easy to add to `jupyterWith`. Kernels are derivations that expose a `kernel.json` file with all information that is required to run a kernel to the main Jupyter derivation. Examples can be found in the [kernels](https://github.com/tweag/jupyterWith/blob/master/kernels) folder.

### [![](data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' class='octicon octicon-link js-evernote-checked' viewBox='0 0 16 16' version='1.1' width='16' height='16' aria-hidden='true' data-evernote-id='253'%3e%3cpath fill-rule='evenodd' d='M4 9h1v1H4c-1.5 0-3-1.69-3-3.5S2.55 3 4 3h4c1.45 0 3 1.69 3 3.5 0 1.41-.91 2.72-2 3.25V8.59c.58-.45 1-1.27 1-2.09C10 5.22 8.98 4 8 4H4c-.98 0-2 1.22-2 2.5S3 9 4 9zm9-3h-1v1h1c1 0 2 1.22 2 2.5S13.98 12 13 12H9c-.98 0-2-1.22-2-2.5 0-.83.42-1.64 1-2.09V6.25c-1.09.53-2 1.84-2 3.25C6 11.31 7.55 13 9 13h4c1.45 0 3-1.69 3-3.5S14.5 6 13 6z' data-evernote-id='1073' class='js-evernote-checked'%3e%3c/path%3e%3c/svg%3e)](https://github.com/tweag/jupyterWith#about-extensions)About extensions

In order to install extensions, JupyterLab runs `yarn` to resolve the precise compatible versions of the JupyterLab core modules, extensions, and all of their dependencies. This resolution process is difficult to replicate with Nix. We therefore decided to use the JupyterLab build system for now to prebuild a custom JupyterLab version with extensions.

If you have ideas on how to make this process more declarative, feel free to create an issue or PR.

### [![](data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' class='octicon octicon-link js-evernote-checked' viewBox='0 0 16 16' version='1.1' width='16' height='16' aria-hidden='true' data-evernote-id='254'%3e%3cpath fill-rule='evenodd' d='M4 9h1v1H4c-1.5 0-3-1.69-3-3.5S2.55 3 4 3h4c1.45 0 3 1.69 3 3.5 0 1.41-.91 2.72-2 3.25V8.59c.58-.45 1-1.27 1-2.09C10 5.22 8.98 4 8 4H4c-.98 0-2 1.22-2 2.5S3 9 4 9zm9-3h-1v1h1c1 0 2 1.22 2 2.5S13.98 12 13 12H9c-.98 0-2-1.22-2-2.5 0-.83.42-1.64 1-2.09V6.25c-1.09.53-2 1.84-2 3.25C6 11.31 7.55 13 9 13h4c1.45 0 3-1.69 3-3.5S14.5 6 13 6z' data-evernote-id='1077' class='js-evernote-checked'%3e%3c/path%3e%3c/svg%3e)](https://github.com/tweag/jupyterWith#nixpkgs)Nixpkgs

The final goal of this project is to be completely integrated into Nixpkgs eventually. However, the migration path, in part due to extensions, is not completely clear.

If you have ideas, feel free to create an issue so that we can discuss.

## [![](data:image/svg+xml,%3csvg xmlns='http://www.w3.org/2000/svg' class='octicon octicon-link js-evernote-checked' viewBox='0 0 16 16' version='1.1' width='16' height='16' aria-hidden='true' data-evernote-id='255'%3e%3cpath fill-rule='evenodd' d='M4 9h1v1H4c-1.5 0-3-1.69-3-3.5S2.55 3 4 3h4c1.45 0 3 1.69 3 3.5 0 1.41-.91 2.72-2 3.25V8.59c.58-.45 1-1.27 1-2.09C10 5.22 8.98 4 8 4H4c-.98 0-2 1.22-2 2.5S3 9 4 9zm9-3h-1v1h1c1 0 2 1.22 2 2.5S13.98 12 13 12H9c-.98 0-2-1.22-2-2.5 0-.83.42-1.64 1-2.09V6.25c-1.09.53-2 1.84-2 3.25C6 11.31 7.55 13 9 13h4c1.45 0 3-1.69 3-3.5S14.5 6 13 6z' data-evernote-id='1081' class='js-evernote-checked'%3e%3c/path%3e%3c/svg%3e)](https://github.com/tweag/jupyterWith#license)License

This project is licensed under the MIT License. See the [LICENSE](https://github.com/tweag/jupyterWith/blob/master/LICENSE)file for details.