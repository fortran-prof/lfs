# LFS
[Linux From Scratch (LFS)](https://www.linuxfromscratch.org/)

# Обход блокировок

Из-за двусторонних блокировок некоторые пакеты, необходимые для LFS, не качаются.
Поэтому нужно использовать VPN, а лучше Tor, но не tor browser, а tor network.

В windows для этого есть замечательное приложение [OnionFruit](https://dragonfruit.network/onionfruit), которое позволяет использовать tor сеть в чистом виде и через мосты obfs4.

В linux все немного сложнее. Нужно использовать утилиту torctl, которая к сожалению, есть только в дистрибутиве BlackArch.
В моем случае был Kali Linux и инструкцию пришлось собирать по кусочкам на просторах интернета.
С одной стороны есть статья https://kali.tools/?p=5043, но она не поясняет как в torctl использовать мосты obfs4.
С другой стороны есть статья https://zalinux.ru/?p=6049, она поясняет как настроить мосты, но не в torctl, а в tor, что не совсем то.
Итогом поисков стало создание следующей инструкции для полноценной работы с tor network в Kali Linux (в других дистрибутивах не проверялось)

```shell
sudo apt install tor obfs4proxy macchanger secure-delete
git clone https://github.com/BlackArch/torctl
cd torctl
sudo cp service/* /etc/systemd/system/
sudo cp bash-completion/torctl /usr/share/bash-completion/completions/torctl

cat > torrc-obfs4 << EOF
ClientTransportPlugin obfs4 exec /usr/bin/obfs4proxy
Bridge obfs4 87.197.128.98:9292 AE6BA15179AEEF64A0F543BEB7FA1BE2E91A93CF cert=t36bkZYMFJyJzTN1HYyh5LzAK/R81GAAH2Q7N3P9qLt5Jd0g7qDvWPIKTurxO2t7XzxSQA iat-mode=0
Bridge obfs4 5.181.25.98:9443 3CD15854FD000C837B17DCD05885470F1A3FF748 cert=3ATT47ZVy0f7ep85pSErGp026P2pFpb+EEvRXC8nnLplc0pcdPRFT0fRJbR3C42C2s/BAw iat-mode=0
UseBridges 1
EOF

sed -i 's/TOR_UID="tor"/TOR_UID="debian-tor"/' torctl
sed -i 's/chmod 644 ${TORRC}/cat torrc-obfs4 >> ${TORRC}\n    chmod 644 ${TORRC}/' torctl

sudo ln -s $(pwd)/torctl /usr/bin/torctl

sudo torctl start
sudo torctl ip
...
sudo torctl stop
```