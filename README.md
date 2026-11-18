# Workshop setup — local Docker or Podman

Use `README.md` and `workshop.ipynb` in the same directory. The notebook and native
OpenShell run on the host; Docker/Podman runs the isolated containers. No GPU is
required.

## 1. Install prerequisites and choose one engine

Linux commands target Ubuntu 22.04+; macOS requires Apple Silicon and
[Homebrew](https://brew.sh/). Install `curl` first if absent. Run the following in
one terminal and keep its engine variables set.

<details>
<summary>Docker on local Linux — installation and selection</summary>

Install Docker Engine 28+ using the [official Docker setup](https://docs.docker.com/engine/install/ubuntu/)
and complete its [non-root setup](https://docs.docker.com/engine/install/linux-postinstall/).
Then verify access and select it:

```sh
docker info
export CONTAINER_ENGINE=docker
unset CONTAINER_CONNECTION
```

</details>

<details>
<summary>Podman on local macOS or Linux — installation and selection</summary>

Follow the [official Podman setup](https://podman.io/docs/installation).

**macOS:** install the CLI with its matching VM helpers. For a Homebrew install,
use `brew install podman`. Create the machine once, then start it if stopped:

```sh
podman machine inspect bts26-cli >/dev/null 2>&1 || \
  CONTAINERS_MACHINE_PROVIDER=applehv podman machine init --cpus 4 --memory 6144 --disk-size 30 bts26-cli
if [ "$(podman machine inspect bts26-cli --format '{{.State}}')" != "running" ]; then
  CONTAINERS_MACHINE_PROVIDER=applehv podman machine start bts26-cli
fi
export CONTAINER_ENGINE=podman
export CONTAINER_CONNECTION=bts26-cli
podman --connection bts26-cli info
```

The headless Linux VM is required for macOS containers; no desktop GUI is needed.

**Native Linux:** install Podman 5+ from the official distribution guide, enable
its rootless API socket once, then select it:

```sh
systemctl --user enable --now podman.socket
export CONTAINER_ENGINE=podman
unset CONTAINER_CONNECTION
podman info
```

If your distribution only supplies Podman <5, choose Docker or upgrade Podman first.

</details>

The selected engine's `info` command must work as your normal user before continuing.

## 2. Install uv and OpenShell

```sh
curl -LsSf https://astral.sh/uv/0.13.8/install.sh -o uv-install.sh
sh uv-install.sh
source ~/.local/bin/env

curl -fsSL https://raw.githubusercontent.com/NVIDIA/OpenShell/v0.1.8/install.sh -o openshell-install.sh
OPENSHELL_VERSION=v0.1.8 sh openshell-install.sh
openshell --version
openshell status
openshell gateway info
```

Skip installers already present at these versions. The native OpenShell installer
starts and registers the gateway. Check that its detected driver matches the
engine you selected. See the [official OpenShell installation guide](https://docs.nvidia.com/openshell/v0.1.8/about/installation).

## 3. Create the uv environment

Enter the directory containing this README and `workshop.ipynb`, then run:

```sh
uv venv btsworkshop --python 3.13.18
cd btsworkshop
source bin/activate
cd ..
uv pip install \
  jupyterlab==4.5.0 ipykernel==7.1.0 nbconvert==7.16.6 \
  nbformat==5.10.4 nbclient==0.11.0 PyYAML==6.0.3 \
  nemo-gym==0.7.0 openshell==0.1.8 openai==2.44.0
uv pip check
uv run --active python -m ipykernel install --prefix "$VIRTUAL_ENV" --name btsworkshop --display-name "Python (btsworkshop)"
```

## 4. Open the notebook

```sh
uv run --active jupyter lab workshop.ipynb --no-browser --ip=127.0.0.1 --port=8888
```
