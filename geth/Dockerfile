FROM ethereum/client-go:latest

RUN mkdir /var/ethereum
ADD genesis.json /

ENTRYPOINT ["geth", "--datadir", "/var/ethereum", "--password", "/var/ethereum/passwd"]
