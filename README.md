## Build: Ubuntu 16.04 LTS
- `sudo apt-get update && sudo apt-get upgrade --yes`
- `git clone https://github.com/bbscoin/bbscoun-wallet`
- `git clone https://github.com/bbscoin/bbscoin`
- `cd bbscoin`
- `ln -s ../bbscoin/ cryptonote`
- `mkdir build && cd build`
- `cmake ..`
- `make`
