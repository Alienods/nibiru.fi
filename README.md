### Nibiru — это независимый PoS блокчейн, который объединяет торговлю деривативами с кредитным плечом, спотовую торговлю, стейкинг и предоставление облигационной ликвидности в единый пользовательский интерфейс, позволяющий пользователям более 40 блокчейнов торговать с кредитным плечом, используя набор компонуемых децентрализованных приложений.



### Эксплоер

Тестнет эксплорер - http://explorer.nodera.org/nibiru/staking

### Discord

Дискорд проект - https://discord.com/invite/nibiru

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet-  |   4|  8GB | 150GB    |

Аренда сервера под ноду  https://pq.hosting/?from=533917

# Обновляем пакеты
```python
sudo apt update && sudo apt upgrade -y
```
#### Устанавливаем инструменты разработчика и необходимые пакеты
```python
sudo apt install curl build-essential pkg-config libssl-dev git wget jq make gcc tmux chrony -y
```
#### Устанавливаем GO
```python
wget https://go.dev/dl/go1.18.4.linux-amd64.tar.gz; \
rm -rv /usr/local/go; \
tar -C /usr/local -xzf go1.18.4.linux-amd64.tar.gz && \
rm -v go1.18.4.linux-amd64.tar.gz && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
source ~/.bash_profile && \
go version
```
#### установим Rust (выберем 1 и y )
```python
curl https://sh.rustup.rs -sSf | sh
```
# Установка ноды

#### Клонируем репозиторий проекта с нодой, переходим в папку с проектом и собираем бинарные файлы
```python
cd $HOME 
git clone https://github.com/NibiruChain/nibiru.git
cd nibiru
git checkout v0.16.3
make install
```
Проверяем версию
```
nibid version
```
Создаем переменные
```
MONIKER_NIBIRU=вводим свое имя
CHAIN_ID_NIBIRU=nibiru-testnet-2
PORT_NIBIRU=36
```

Сохраняем переменные, перезагружаем .bash_profile и проверяем значения переменных
```
echo "export MONIKER_NIBIRU="${MONIKER_NIBIRU}"" >> $HOME/.bash_profile
echo "export CHAIN_ID_NIBIRU="${CHAIN_ID_NIBIRU}"" >> $HOME/.bash_profile
echo "export PORT_NIBIRU="${PORT_NIBIRU}"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
```
echo -e "\nmoniker_NIBIRU > ${MONIKER_NIBIRU}.\n"
echo -e "\nchain_id_NIBIRU > ${CHAIN_ID_NIBIRU}.\n"
echo -e "\nport_NIBIRU > ${PORT_NIBIRU}.\n"
```
Настраиваем конфиг
```
nibid config chain-id $CHAIN_ID_NIBIRU
nibid config keyring-backend test
nibid config node tcp://localhost:${PORT_NIBIRU}657
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.025unibi\"/" $HOME/.nibid/config/app.toml
```
Инициализируем ноду
```
nibid init $MONIKER_NIBIRU --chain-id $CHAIN_ID_NIBIRU
```
Загружаем генезис файл и адресбук
```
curl -s https://networks.testnet.nibiru.fi/$CHAIN_ID_NIBIRU/genesis > $HOME/.nibid/config/genesis.json
```
Проверим генезис
```
sha256sum ~/.nibid/config/genesis.json
5cedb9237c6d807a89468268071647649e90b40ac8cd6d1ded8a72323144880d
```
Проверяем, состояние валидатора на начальном этапе
```
cd && cat .nibid/data/priv_validator_state.json
{
  "height": "0",
  "round": 0,
  "step": 0
}

 если нет, то выполняем команду
nibid tendermint unsafe-reset-all --home $HOME/.nibid
```
Добавляем сиды и пиры
```
sed -i 's|seeds =.*|seeds = "'$(curl -s https://networks.testnet.nibiru.fi/$CHAIN_ID_NIBIRU/seeds)'"|g' $HOME/.nibid/config/config.toml
```
Изменяем порты для возможности дальнейшего подселения других нод проектов экосистемы Космос на один сервер
```
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT_NIBIRU}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT_NIBIRU}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT_NIBIRU}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT_NIBIRU}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT_NIBIRU}660\"%" $HOME/.nibid/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PORT_NIBIRU}317\"%; s%^address = \":8080\"%address = \":${PORT_NIBIRU}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PORT_NIBIRU}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PORT_NIBIRU}091\"%" $HOME/.nibid/config/app.toml
```
Настраиваем прунинг
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.nibid/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.nibid/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.nibid/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"
```
Сбрасываем данные
```
nibid tendermint unsafe-reset-all --home $HOME/.nibid
```
Создаем сервисный файл
```
printf "[Unit]
Description=Nibiru Service
After=network.target

[Service]
Type=simple
User=$USER
ExecStart=$(which nibid) start --home $HOME/.nibid
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/nibid.service
```
Запускаем сервис и проверяем логи
```
sudo systemctl daemon-reload && \
sudo systemctl enable nibid && \
sudo systemctl restart nibid && \
sudo journalctl -u nibid -f -o cat
```
Ждем окончания синхронизации, проверить синхронизации можно командой
```
nibid status 2>&1 | jq .SyncInfo

#Если вывод показывает false, синхронизация завершена.
```
State sync
```
sudo systemctl stop nibid
nibid tendermint unsafe-reset-all --home $HOME/.nibid --keep-addr-book
```
```
sed -i 's|enable =.*|enable = true|g' $HOME/.nibid/config/config.toml 
sed -i 's|rpc_servers =.*|rpc_servers = "'$(curl -s https://networks.testnet.nibiru.fi/$CHAIN_ID_NIBIRU/rpc_servers)'"|g' $HOME/.nibid/config/config.toml
sed -i 's|trust_height =.*|trust_height = "'$(curl -s https://networks.testnet.nibiru.fi/$CHAIN_ID_NIBIRU/trust_height)'"|g' $HOME/.nibid/config/config.toml
sed -i 's|trust_hash =.*|trust_hash = "'$(curl -s https://networks.testnet.nibiru.fi/$CHAIN_ID_NIBIRU/trust_hash)'"|g' $HOME/.nibid/config/config.toml
```
```
sudo systemctl restart nibid
```
Создаем кошелек
```
nibid keys add $MONIKER_NIBIRU
```
 восстанавливаем кошелек командой и вводим мнемоник фразу
 ```
 nibid keys add $MONIKER_NIBIRU --recover
 ```
 Создаем переменную с адресом кошелька и валидатора
 ```
WALLET_NIBIRU=$(nibid keys show $MONIKER_NIBIRU -a)
VALOPER_NIBIRU=$(nibid keys show $MONIKER_NIBIRU --bech val -a)
```
```
echo "export WALLET_NIBIRU="${WALLET_NIBIRU}"" >> $HOME/.bash_profile
echo "export VALOPER_NIBIRU="${VALOPER_NIBIRU}"" >> $HOME/.bash_profile
source $HOME/.bash_profile
echo -e "\nwallet_NIBIRU > ${WALLET_NIBIRU}.\n"
echo -e "\nvaloper_NIBIRU > ${VALOPER_NIBIRU}.\n"
```
Проверяем свой баланс
```
nibid q bank balances $WALLET_NIBIRU
```
После завершения синхронизации и пополнении кошелька, создаем валидатора
```
nibid tx staking create-validator \
--amount 9000000unibi \
--from $WALLET_NIBIRU \
--commission-max-change-rate "0.01" \
--commission-max-rate "0.2" \
--commission-rate "0.07" \
--min-self-delegation "1" \
--pubkey  $(nibid tendermint show-validator) \
--moniker $MONIKER_NIBIRU \
--chain-id $CHAIN_ID_NIBIRU \
--min-self-delegation "1000000" \
--identity="" \
--details="" \
--website="" \
--fees=5000unibi \
-y
```
Для удаления ноды используйте следующие команды
```
sudo systemctl stop nibid
sudo systemctl disable nibid
sudo rm -rf $HOME/.nibid 
sudo rm -rf $HOME/nibiru
sudo rm -rf /etc/systemd/system/nibid.service
sudo rm -rf /usr/local/bin/nibid
sudo systemctl daemon-reload
```
##### Полезные команды


Рестарт ноды
```
sudo systemctl restart nibid
```
Проверка логов
```
sudo journalctl -u nibid -f -o cat
```
Узнать адрес валидатора
```
nibid keys show $MONIKER_NIBIRU --bech val -a
`````
Внести изменения в валидатора
````
nibid tx staking edit-validator --identity="" --details="" --website="" \
--from $WALLET_NIBIRU --chain-id $CHAIN_ID_NIBIRU -y


#identity - PGP ключ c keybase.io (устанавливает аватар валидатора)
#details - текстовое описание валидатора




