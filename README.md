## OSX
```
brew install qt5
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_PREFIX_PATH=/usr/local/Cellar/qt/5.10.0_1
make
```

## Ubuntu
```
ln -s ../bbscoin/ cryptonote
mkdir build && cd build
cmake ..
make
```
