# Local LLM Solution: Connecting MSTY on Local Machine to Remote Ollama Server on ALCF Supercomputers

[Ollama](https://ollama.ai/) is currently the go-to solution for running local LLM. however, the GPU memory requirement severely limits the type of models that can be run locally.
In most cases, local LLMs are limited to 7b parameter size or less and high quantization like q4 or even q2. Such small and quantized models may leave users unsatisfied.
Running the Ollama server remotely on ALCF supercomputers (Polaris or Sophia) is a great solution to this problem since those supercomputers are equipped with A100 gpus.
This guide provides step-by-step instructions to connect the [MSTY](https://msty.app/) app to a remote Ollama server running on Sophia. MSTY is a great platform to run Ollama since it provides a beautiful GUI that is designed for Ollama.



## Prerequisites

- **ALCF Account**: Ensure you have an active account with access to ALCF supercomputers (in this case, Sophia)
- **Ollama**: Installed on one of your project directories on ALCF. Installation directions are provided [here](https://github.com/fbhuiyan2/Remote-Ollama-server-on-ALCF-HPC-for-local-LLM/blob/main/Install%20Ollama%20on%20ALCF%20HPC.md)
- **MSTY app**: Installed on your local machine

## Steps

- Log in to Sophia and submit an interactive job using the following command:

```bash
qsub -I -l select=2 -q by-gpu -l filesystems=home:eagle -l walltime=2:00:00 -A projectname
```

  - `-I`: Starts an interactive session.
  - `-l select=2`: Requests 2 nodes.
  - `-q by-gpu`: Specifies the GPU queue.
  - `-A projectname`: Charges the job to the `projectname` project allocation.

Wait until the job is scheduled and you are placed into an interactive session on the compute node.


- Once on the compute node, start the Ollama server in the background:

```bash
OLLAMA_HOST=0.0.0.0:11434 OLLAMA_KEEP_ALIVE=-1 OLLAMA_CONTEXT_LENGTH=16000 ollama serve &
```

`OLLAMA_HOST=0.0.0.0:11434` Configures Ollama to listen on all network interfaces on port `11434`. This overrides the default 127.0.0.0 which does not listen to outside networks and thus, cannot be used for remote Ollama purposes.

`OLLAMA_KEEP_ALIVE=-1` is used to keep the server running indefinitely (default is like 10 minutes or so). 

`OLLAMA_CONTEXT_LENGTH=16000` is used to increase the context window to 16k tokens (current default is 4k tokens).


- Get the fully qualified domain name (FQDN) of the compute node (i.e. Compute Node Hostname):

```bash
HOSTNAME=$(hostname -f)
echo -e " ssh -N -L 8888:${HOSTNAME}:11434 username@sophia.alcf.anl.gov"
```

Copy the printed line. Note, 8888 is the port number on your pc and username is your username for ALCF.

- On your **local machine**, paste the copied line in your terminal to establish an SSH tunnel to forward a local port to the Ollama server on the compute node. You may be prompted for ALCF password.

  - **Example line**

```bash
ssh -N -L 8888:sophianode01:11434 username@sophia.alcf.anl.gov
```

Leave this SSH session running; it maintains the tunnel.

- Test the connection. In a different terminal tab, run:

```bash
curl http://localhost:8888/api/tags
```

You should get JSON output.

- Open MSTY app, set up a new remote model provide.
  - Use `http://localhost:8888` as the remote endpoint and fetch models

And you are all set!


## Additional Notes

- **Firewall Considerations**: Ensure that your local firewall allows connections on port `8888`.
- **SSH Session**: The SSH tunnel must remain open. Do not close the SSH session established in Step 4.
- **Session Duration**: The interactive job has a walltime limit. Be mindful of the job's duration.

## Troubleshooting

- **Connection Refused**: If you receive a connection refused error, check that:
  - The Ollama server is running on the compute node.
  - The SSH tunnel is active.
  - The ports are correctly specified and not blocked or not already in use.
- **SSH Tunnel Errors**: Ensure that you have the proper permissions and that your SSH keys are correctly set up for passwordless login if desired.


---
