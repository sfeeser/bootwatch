
1. Download the Latest Go Binary from https://go.dev/dl/

    > Find the latest stable version for Linux (e.g., go1.23.2.linux-amd64.tar.gz for 64-bit systems).

    `wget https://go.dev/dl/go1.24.2.linux-amd64.tar.gz`
   
2. Remove any existing Go installation (if necessary)
 
    `sudo rm -rf /usr/local/go`
   
3. Extract the downloaded tarball to /usr/local:

    `sudo tar -C /usr/local -xzf go1.24.2.linux-amd64.tar.gz`
   
4. Add Go’s binary path to your PATH and set the GOPATH environment variable by editing your shell configuration file (e.g., ~/.bashrc or ~/.zshrc):

    `echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc`

    `echo 'export GOPATH=$HOME/go' >> ~/.bashrc`
   
5. Apply the changes:

    `source ~/.bashrc`

6. Check the installed Go version:

    `go version`

    ```
    go version go1.24.2 linux/amd64
    ```

7. Remove the downloaded tarball:

   `rm go1.23.2.linux-amd64.tar.gz`
   
8. Create directories for your Go workspace:

    `mkdir -p $HOME/go/{bin,src,pkg}
