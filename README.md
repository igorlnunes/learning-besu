# Operando Hyperledger Besu em Redes Privadas

## 1. Introdução
Este repositório foi criado como uma introdução prática ao Hyperledger Besu. Primeiramente, utilizamos uma rede de desenvolvimento para testar a configuração básica. Em seguida, montamos uma rede privada com quatro nós Besu utilizando Docker, realizamos o deploy de um contrato inteligente, adicionamos carteiras no MetaMask e efetuamos transações.

> **Nota**: Este repositório é inspirado no trabalho de [Samuel Venzi](https://github.com/samuelvenzi). O repositório original está disponível [aqui](https://github.com/samuelvenzi/besu-workshop/). Para contexto adicional, veja o workshop do professor sobre [Como Operar e Usar Hyperledger Besu em Redes Públicas e Privadas](https://wiki.hyperledger.org/display/events/Como+Operar+e+Usar+Hyperledger+Besu+em+Redes+Publicas+e+Privadas).

## 2. Executando Besu com a rede de desenvolvimento (dev)
A rede de desenvolvimento (dev) é a forma mais básica de operar o Besu. Para iniciar um nó na rede dev, execute o seguinte comando:

```bash
besu --network=dev --rpc-http-enabled
```

Este comando inicia um nó Besu usando as configurações definidas no arquivo [`genesis.json`](https://github.com/hyperledger/besu/blob/main/config/src/main/resources/dev.json).

### 2.1 Verificando o saldo de uma conta
Para verificar o saldo de uma conta usando a API JSON-RPC, utilize o seguinte comando:

```bash
curl http://localhost:8545/ \
  -X POST \
  -H "Content-Type: application/json" \
  --data '{
          "method":"eth_getBalance",
          "params":["0x627306090abaB3A6e1400e9345bC60c78a8BEf57", "latest"],
          "id":1,
          "json-rpc":"2.0"
          }'
```

Esse comando faz uma requisição utilizando o método `eth_getBalance`, retornando o saldo da carteira em hexadecimal.

### 2.2 Conectando o MetaMask
Para conectar o MetaMask ao nó Besu, é necessário permitir CORS ao iniciar o nó:

```bash
besu --network=dev --rpc-http-enabled --rpc-http-cors-origins="all"
```

Em seguida, configure o MetaMask para se conectar à rede Besu em `http://localhost:8545` e importe a chave privada da conta `0x627306090abaB3A6e1400e9345bC60c78a8BEf57`, disponível no arquivo `genesis.json`. **Importante**: estas carteiras são apenas para testes e não devem ser usadas para valores reais.

## 3. Executando o Besu em uma Rede Privada
O Hyperledger Besu também pode ser configurado para operar em redes blockchain privadas, voltadas a aplicações empresariais que demandam segurança e alta performance. Uma rede privada não se conecta à Ethereum Mainnet ou Testnets. Essas redes utilizam IDs de cadeia (_chain IDs_) específicas e métodos de consenso como _Proof of Authority_ (QBFT, IBFT 2.0 ou Clique). Além disso, o Besu suporta soluções de privacidade e permissionamento.

### 3.1 Arquitetura de Rede Privada
Abaixo está uma representação da arquitetura recomendada para redes privadas no Besu:


<img alt="Arquitetura Besu para Redes Privadas" src="https://besu.hyperledger.org/assets/images/private-architecture-5a4d514abd93e693a77b25cacdfc9ef7.jpeg" width="680" height="550">

Para iniciar uma rede privada Besu com quatro nós, utilize o script de shell:

```bash
./startDev.sh
```

### 3.2 Explicação do [script de inicialização](./startDev.sh) da rede Besu


```bash
if ! [ -x "$(command -v besu)" ]; then
```
a. `if` - estrutura condicional;
b. `! [ -x "$(command -v besu)" ]` - verifica se o comando `besu` não está instalado;
c. `command -v` - busca o caminho do executável fornecido;

```bash
echo "Error: besu is not installed. Go to https://besu.hyperledger.org/private-networks/get-started/install/binary-distribution" >&2
exit 1
```
a. `>&2` - redireciona a saída para o erro padrão;
b. `exit 1` - encerra o script com código de erro 1;

```bash
docker rm -f besu-node-0 besu-node-1 besu-node-2 besu-node-3
```
a. `docker rm -f` - força a remoção de contêineres;
b. `besu-node-0` até `besu-node-3` - nomes dos contêineres a serem removidos;

```bash
sudo rm -rf node/besu-0/data
```
a. `sudo` - executa o comando como administrador;
b. `rm -rf` - remove diretórios e arquivos de forma recursiva e forçada;
c. `node/besu-0/data` - caminho do diretório a ser removido;

```bash
mkdir -p node/besu-0/data
```
a. `mkdir -p` - cria diretórios, ignorando erros se já existirem;

```bash
besu operator generate-blockchain-config --config-file=../config/qbftConfigFile.json --to=networkFiles --private-key-file-name=key
```
a. `besu` - executa o comando do cliente Besu;
b. `operator generate-blockchain-config` - comando para gerar a configuração da blockchain;

```bash
docker network create besu_network
```
a. `docker network create` - cria uma rede Docker;
b. `besu_network` - nome da rede;

```bash
docker-compose -f docker/docker-compose-bootnode.yaml up -d
```
a. `docker-compose` - gerenciador de contêineres em composição;
b. `-f` - especifica o arquivo YAML de configuração;
c. `up -d` - inicializa os contêineres em segundo plano;

```bash
export ENODE=$(curl -X POST --data '{"jsonrpc":"2.0","method":"net_enode","params":[],"id":1}' http://127.0.0.1:8545 | jq -r '.result')
```
a. `export` - define uma variável de ambiente;
b. `curl -X POST` - faz uma solicitação HTTP POST;
c. `jq -r '.result'` - filtra e extrai o valor do campo `result`;

```bash
sed "s/<ENODE>/enode:\/\/$E_ADDRESS/g" docker/templates/docker-compose-nodes.yaml > docker/docker-compose-nodes.yaml
```
a. `sed` - comando para substituição de texto;
b. `"s/<ENODE>/enode://$E_ADDRESS/g"` - substitui `<ENODE>` pelo valor de `$E_ADDRESS`;
c. `>` - redireciona a saída para o arquivo;

```bash
truffle migrate --f 1 --to 1 --network development
```
a. `truffle` - framework para desenvolvimento em Ethereum;
b. `migrate` - comando para migrar contratos;
c. `--network development` - define o ambiente de rede para `development`;
