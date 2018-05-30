## OSX
```
brew install qt5
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

Now, you should compile the BBSCoin dynamic libs without rocksdb first, comment "add_subdirectory(tests)" line in bbscoin/CMakeLists.txt, and compile the packages/rocksdb-rocksdb-4.11.2.zip dynamic libs.
Then copy these files to the libs dir.

Finally, use CMake and VS compile this project.

## Windows Pre-built

Following dependencies need to be installed before running the wallet:

Windows binary require VC2017 runtime.
Download from https://aka.ms/vs/15/release/vc_redist.x64.exe

Universal C Runtime
https://support.microsoft.com/en-us/help/2999226/update-for-universal-c-runtime-in-windows
