## OSX
```
brew install qt5
brew link --force qt5
ln -s /usr/local/Cellar/qt/5.11.0/mkspecs /usr/local/mkspecs
ln -s /usr/local/Cellar/qt/5.11.0/plugins /usr/local/plugins
ln -s ../bbscoin/ cryptonote
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make
```

## Ubuntu
```
sudo apt-get install build-essential libboost-all-dev git cmake qtbase5-dev
git clone https://github.com/bbscoin/bbscoin.git
cd ..
git clone https://github.com/bbscoin/bbscoin-wallet.git
cd bbscoin-wallet
ln -s ../bbscoin/ cryptonote
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make
```

## Windows

Clone the bbscoin first,

```
git clone https://github.com/bbscoin/bbscoin.git bbscoin
```

Now, you should compile the BBSCoin dynamic libs first.
Then copy these files to the libs dir.

Finally, use CMake and VS compile this project.

## Windows Pre-built

Following dependencies need to be installed before running the wallet:

Windows binary require VC2017 runtime.
Download from https://aka.ms/vs/15/release/vc_redist.x64.exe

Universal C Runtime
https://support.microsoft.com/en-us/help/2999226/update-for-universal-c-runtime-in-windows
