# Install Ollama on ALCF HPC

### Link to latest release 

Get the latest release of Ollama from [here](https://github.com/ollama/ollama/releases/). At the time of writing, the latest version is 0.5.13

### Installation directions

```bash
mkdir ollama
pushd ollama
curl -L https://github.com/ollama/ollama/releases/download/v0.5.13/ollama-linux-amd64.tgz -o ollama-linux-amd64.tgz
mkdir usr
tar -C usr -xvf ollama-linux-amd64.tgz 
popd
```
### Set environment variables

There are a couple of environment variables that are needed to run with this custom install.

  * `PATH` - Needs to point to `/ollama_install/usr/bin`.
    * export PATH=${PATH}:path-to-/ollama/usr/bin
  * `LD_LIBRARY_PATH` - Needs to point to `/ollama_install/usr/lib`.
    * export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:path-to-/ollama/usr/lib
  * `OLLAMA_MODELS` - Needs to point to `/ollama_install/models`. This is where Ollama installs the models and they are large so make sure that they are in your project folder.
    * export OLLAMA_MODELS=path-to-/ollama/models
  * `OLLAMA_HOST` - This is the IP address the server should watch this needs to be set as if multiple Ollama servers are running at the same time at the same port they will interfere.
    * Note, by default, for one server, setting this is not necessary

Adding these environment settings in the .bashrc or .bash_profile and sourcing the file is recommended so you do not have to define the environment everytime.

