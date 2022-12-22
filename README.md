README.md
#!/bin/bash

function forta_passphrase {
  echo -e "Придумайте парольную фразу для вашего кошелька" && sleep 1
  read forta_passphrase
}

function tools_install {
  sudo apt update
  sudo apt install mc build-essential wget htop curl jq -y
}

function docker_install {
  curl -s https://raw.githubusercontent.com/razumv/helpers/main/tools/install_docker.sh | bash
}

function forta_install {
  # sudo curl https://dist.forta.network/artifacts/forta -o /usr/local/bin/forta
  # sudo chmod 755 /usr/local/bin/forta
  sudo curl https://dist.forta.network/pgp.public -o /usr/share/keyrings/forta-keyring.asc -s
  echo 'deb [signed-by=/usr/share/keyrings/forta-keyring.asc] https://dist.forta.network/repositories/apt stable main' | sudo tee -a /etc/apt/sources.list.d/forta.list
  sudo apt update
  sudo apt install forta -y
}

function forta_init {
  /usr/bin/forta init --passphrase $forta_passphrase > $HOME/init_forta.txt
}

function forta_env {
  sudo mkdir -p /lib/systemd/system/forta.service.d/
  sudo tee <<EOF >/dev/null /lib/systemd/system/forta.service.d/env.conf
  [Service]
  Environment="FORTA_DIR=$HOME/.forta"
  Environment="FORTA_PASSPHRASE=$forta_passphrase"
EOF
}

function forta_address {
  forta_address=`cat init_forta.txt | grep "Scanner address" | awk '{print $3}'`
  echo 'export forta_address='${forta_address} >> $HOME/.profile
}

function forta_config {
  sudo tee <<EOF >/dev/null $HOME/.forta/config.yml
  chainId: 10

  scan:
    jsonRpc:
      url: https://mainnet.optimism.io

  trace:
    enabled: false

EOF
}

function logo {
  curl -s https://raw.githubusercontent.com/razumv/helpers/main/doubletop.sh | bash
}

function line {
  echo "-----------------------------------------------------------------------------"
}

function colors {
  GREEN="\e[1m\e[32m"
  RED="\e[1m\e[39m"
  NORMAL="\e[0m"
}



colors
line
logo
line
echo -e "${GREEN}1. Введите парольную фразу для кошелька... ${NORMAL}" && sleep 1
forta_passphrase
line
echo -e "${GREEN}2. Ставим зависимости... ${NORMAL}" && sleep 1
line
tools_install
docker_install
forta_install
sudo systemctl stop forta
line
echo -e "${GREEN}3. Инициализируем Forta, добавляем переменные и добавляем конфиг... ${NORMAL}" && sleep 1
line
forta_init
forta_env
forta_config
forta_address
line
echo -e "${GREEN}Инициализация завершена! ${NORMAL}" && sleep 1
echo -e "${GREEN}Пополняем в сети MATIC на пару копеек для транзакции адрес: $forta_address ${NORMAL}" && sleep 1
echo -e "${GREEN}И переходим к следующему пункту гайда... ${NORMAL}" && sleep 1
