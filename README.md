# TVM

TVM (Terraform Version Manager) is a CLI tool to manage Terraform installations.

It supports both MacOS and Linux, and both arm64 and amd64 architectures.

## Usage

- Install the required libraries

```bash
python3 -m pip install --user -r requirements.txt
```

- Install specific Terraform version

```bash
./tvm install 1.6.4

# You can feed multiple versions to the command
./tvm install 1.5.2 1.5.3
```

- Use specific Terraform version

```bash
./tvm use 1.6.4
```

You can use the "use" command to symlink specific version without the "install" command. "use" command symlinks the installed Terraform versions from `~/.tvm/` path.

- List Terraform versions on the system. Active version is marked on the output.

```bash
./tvm list
```

- Remove unused or all Terraform versions

```bash
./tvm remove unused # Removes all versions except the active version
./tvm remove all
```

- Get available commands and their descriptions

```bash
./tvm --help
```
