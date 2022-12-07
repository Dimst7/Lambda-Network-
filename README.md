# Lambda -Network-

## Заполняем форму:

https://docs.google.com/forms/d/e/1FAIpQLSf5hmriYnRgAa7zD3IuFcptuAMl-3xZM9L-z94yrqdZwXH8Cg/viewform

В итоге должны будут выдать роль в дискорде
Там же можно будет запросить эйрдроп для ранних птичек в ветке early-bird-airdrop если ещё доступно, для этого в ветке вводим 
/claim АДРЕСЭФИРКОШЕЛЬКА

## Минимальные официальные системные требования:

4 CPU
500 ГБ  SSD
32 ГБ RAM


## 1. Обновляем пакеты и систему
```
sudo apt update && sudo apt upgrade -y
```
## затем
```
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## 2. Установка GO
```
ver="1.18.2"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
source ~/.bash_profile
```

## 3. Установка бинарного файла ноды
```
git clone https://github.com/LambdaIM/lambdavm.git
cd lambdavm && git checkout v1.0.0
make install
```

## 4. Проверяем версию(должно быть 1.0.0)
```
lambdavm version
```

## 5. Сохраняем идентификатор цепочки
```
lambdavm config chain-id lambdatest_92001-2
```

## 6. Инициализируем узел(меняем имя на свое)
```
lambdavm init ИМЯНОДЫ --chain-id lambdatest_92001-2
```

## 7. Скопируйте Genesis файл
```
wget https://raw.githubusercontent.com/LambdaIM/testnets/main/lambdatest_92001-2/genesis.json
mv genesis.json ~/.lambdavm/config/
lambdavm validate-genesis
```

## 8. Добавляем SEED
```
SEEDS=`curl -sL https://raw.githubusercontent.com/LambdaIM/testnets/main/lambdatest_92001-2/seeds.txt | awk '{print $1}' | paste -s -d, -`
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" ~/.lambdavm/config/config.toml
```
## 9. Добавляем PEERS
```
PEERS=`curl -sL https://raw.githubusercontent.com/LambdaIM/testnets/main/lambdatest_92001-2/peers.txt | sort -R | head -n 10 | awk '{print $1}' | paste -s -d, -`
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" ~/.lambdavm/config/config.toml
```
## 9.5 Прунинг
```
pruning="custom" 
pruning_keep_recent="100" 
pruning_keep_every="0" 
pruning_interval="50"
```
```
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.lambdavm/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.lambdavm/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.lambdavm/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.lambdavm/config/app.toml
```
## 10. Создаем кошелек (придумываем пароль и СОХРАНЯЕМ! мнемоническую фразу)
```
lambdavm keys add wallet
```
## 11. Запрашиваем токены тестовой сети

В дискорде находим канал faucet-testnet и вводим команду /faucet после чего, в отдельном окне вставляем наш кошелек.
Через некоторое время токены поступят на баланс нашего кошелька.


## 12.Создаем сервисный файл
```
printf "[Unit]
Description=lambda node
After=network-online.target

[Service]
User=root
ExecStart=`which lambdavm` start --x-crisis-skip-assert-invariants
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/lambdavm.service
```
## 13. Запускаем сервис
```
sudo systemctl daemon-reload 
sudo systemctl enable lambdavm 
sudo systemctl restart lambdavm
```

### Проверяем логи
```
sudo journalctl -u lambdavm -f
```

### После синхронизации, можно будет узнать баланс введя команду:
```
lambdavm q bank balances АДРЕСКОШЕЛЬКА 
```

### Создаем валидатора после завершения синхронизации! Проверить можно командой(статус должен быть false):
```
lambdavm status 2>&1 | jq .SyncInfo
```

## 14. Создание валидатора(выделенные имя ноды и имя кошелька меняем на свои, выделенные строки  внизу заполняем если есть что заполнить, если нет то строку удаляем)
```
lambdavm tx staking create-validator \
  --amount=1000000000000000000ulamb \
  --pubkey=$(lambdavm tendermint show-validator) \
  --moniker=ИМЯНОДЫ \
  --chain-id=lambdatest_92001-2 \
  --commission-rate="0.05" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1000000" \
  --gas="300000" \
  --gas-prices="0.025ulamb" \
  --from=ИМЯКОШЕЛЬКА
  --website="" \
  --identity="" \
  --details="" \
  --broadcast-mode block
```


### Проверяем логи
```
sudo journalctl -u lambdavm -f
```
### Эксплорер блоков
```
https://explorer.lambda.top/
```
### Дашборд\Валидаторы
```
https://app.lambda.top/staking
```
### Снять Награды
```
lambdavm tx distribution withdraw-rewards $(lambdavm keys show wallet --bech val -a) --commission --from "ИМЯ_КОШЕЛЬКА" --fees=10000ulamb --chain-id lambdatest_92001-2
```
### Делегировать своему валидатору
```
lambdavm tx staking delegate Адрес_Вашего_валидатора 1000000000000000000ulamb --gas 300000 --chain-id lambdatest_92001-2 --from ИМЯКОШЕЛЬКА --fees=10000ulamb -y
```
### Выйти из тюрьмы
```
lambdavm tx slashing unjail \
  --broadcast-mode=block \
  --from="ИМЯ_КОШЕЛЬКА" \
  --chain-id=lambdatest_92001-2 \
  --fees=10000ulamb \
  --gas=300000
```
### Ещё можно тут НФТшку заклеймить:
```
https://galxe.com/LambdaNetwork/campaign/GC21gUwyrZ
```


