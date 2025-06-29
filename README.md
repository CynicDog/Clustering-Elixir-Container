# Communicate with an Elixir Node in Docker Container
This guide shows how to set up distributed Elixir nodes between your **host machine (macOS)** and a **Docker container** using node names and cookies, and how to call functions remotely.


## On Host Machine 🧑🏻‍💻 

### Start a BEAM node on the host machine

```bash
cynocdig@CynicDogs-MacBook % iex --name host@host.docker.internal --cookie mycookie
````

`host.docker.internal` is a special DNS name Docker provides
On macOS and Windows, Docker automatically sets `host.docker.internal` inside containers to point back to the host machine.

## On Docker Container 🐋

### Run the container with network configuration

```bash
cynocdig@CynicDogs-MacBook % docker run -it --rm \
  --add-host host.docker.internal:host-gateway \
  elixir bash
```

### Start a BEAM node inside the container
```bash
root@3e185f96415a:/# iex --name elixir_node@container --cookie mycookie
```

### Connect container node to host node
```bash
iex(elixir_node@container)1> Node.connect(:'host@host.docker.internal')
```

## Say Hello to the Host Node!

Define a module on both the host and container nodes:

```elixir
defmodule Greet do
  def say_hello do
    IO.puts("Hello 👋🏻 from #{node()}! 🐋")
  end
end
```
> Avoid sending anonymous functions between nodes; always use named module functions. Distributed BEAM nodes send function “closures” as a serialized object
For remote execution (e.g., Node.spawn/2), the function must be sent (serialized) over the network.

The BEAM tries to serialize the anonymous function, but the internal metadata and references are not portable or don’t match on the remote node.
 
Then, from the container node IEx shell, call the function remotely on the host node:

```bash
iex(elixir_node@container)2> Node.spawn(:"host@host.docker.internal", Greet, :say_hello, [])
```
