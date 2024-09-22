# Tutorial de configuração de um gateway lora no TTS

Nesse tutorial apresentarei a instalação passo a passo de um gateway LoraWan SX1302 do fabricante Waveshare com um RaspberryPi 3.

 [LoraWan SX1302 Wiki](https://www.waveshare.com/wiki/SX1302_LoRaWAN_Gateway_HAT)

## 1. Instalação do SO do Raspberry

Utilize o Raspberry Pi Imager para instalar o Raspberry pi OS  (Legacy, 64-bit) Lite no seu cartão SD.

[Raspberry Pi Image Download](https://www.raspberrypi.com/software/)

Configure o login e senha e habilite conexão SSH e com o wi-fi se desejar.

Abra o terminal do Raspberry Pi e digite a sequinte linha de comando:

```
sudo raspi-config
```

Vá para opção `3. Interface Options`, habilite a SPI e I2C, para habilitar a Porta serial, na primeira opção perguntará se deseja ativar para o console, escolha No e Yes para a segunda opção. Reboot YES.

## 2. Instalação do sistema para o Gateway

Na linha de comando do Raspberry Pi, digitaremos os seguintes comandos em ordem de acordo com o manual do fabricante para instalação da biblioteca:

```
sudo apt update
sudo apt install git
mkdir Documents
cd Documents
git clone https://github.com/Lora-net/sx1302_hal.git
cd sx1302_hal
make clean all
make all
cp tools/reset_lgw.sh util_chip_id/
cp tools/reset_lgw.sh packet_forwarder/
```

#Verificar no site do fabricante se os pinos do shield são os mesmo que estão na configuração do arquivo reset_lgw.sh. 
```
cd ~/Documents/sx1302_hal/packet_forwarder/
nano reset_lgw.sh
```
```
# GPIO mapping has to be adapted with HW
#
SX1302_RESET_PIN=23     # SX1302 reset
SX1302_POWER_EN_PIN=18  # SX1302 power enable
SX1261_RESET_PIN=22     # SX1261 reset (LBT / Spectral Scan)
AD5338R_RESET_PIN=13    # AD5338R reset (full-duplex CN490 reference design)
```
Caso seja diferente os pinos modificar o arquivo reset_lgw.sh e salvar.

### Obter o EUI do Gateway
Rode os seguintes comandos: 
```
cd ~/Documents/sx1302_hal/util_chip_id/
./chip_id 
```

O resultado do comando `./chip_id` deve ser algo similar a:

```
CoreCell reset through GPIO23...
SX1261 reset through GPIO23...
CoreCell power enable through GPIO18...
CoreCell ADC reset through GPIO13...
Opening SPI communication interface
Note: chip version is 0x12 (v1.2)
INFO: using legacy timestamp
ARB: dual demodulation disabled for all SF
INFO: found temperature sensor on port 0x39

INFO: concentrator EUI: 0xXXXXXXXXXXXXXXXX

Closing SPI communication interface
CoreCell reset through GPIO23...
SX1261 reset through GPIO23...
CoreCell power enable through GPIO18...
CoreCell ADC reset through GPIO13...
```

Onde `0xXXXXXXXXXXXXXXXX` é o código EUI é o gateway_id que será utilizado na configuração do Passo 4.

## 3. Configurando o TTS

Com o código EUI em mãos podemos criar a conta e configurar no gateway no TTS.

Crie uma conta no site:

[The Things Network](https://id.thethingsnetwork.org/)

Após realizar o login, iremos iniciar a configuração, para isso acesse o console:

[Console TTN](https://console.cloud.thethings.network/)


Siga o passo a passo para configurar o gateway, escolha o país onde será instalado e a região.

### Registrando o Gateway

No menu superior clique em `Gateways` e depois no botão `+ Register gateway`. Digite o EUI do gateway e complete com as informações do seu dispositivo e região.

### Criando a Aplication

Clique no menu `Applications` e clique no botão `+ Create application`. 

Preencha os dados de Application Id, name e description.

Após criar a application, iremos cadastrar os End-Devices para permitir a comunicação dos mesmos com o TTS. Para isso, basta acessar a página da aplicação e clicar no botão `+ Register end device`, para cadastrar o device, tenha em mãos os dados de configuração do mesmo, como: Marca do fornecedor, modelo, classe de operação, região, plano de frequência, e id do chip lora do end device.

## 4. Iniciar o gateway
Para rodar o gateway precisamos do arquivo de configuração, no caminho `cd ~/Documents/sx1302_hal/packet_forwarder/` encontram-se vários modelos de acordo com a a banda, frequência e canal da sua região, verifique qual seria o compatível com as normas regulatórias vigentes.

Neste tutorial utilizamos um modelo para AU915, que seria o correto para a região, atualize os campos para satisfazer as suas necessidades. Utilize o código EUI no gateway_id obtido no passo 2 e o server_address de cadastro no TTS. 

Acessar e criar arquivo no caminho:

```
cd ~/Documents/sx1302_hal/packet_forwarder/
```
```
nano global_conf.json.sx1250.AU915
```
Modificar JSON abaixo com os dados de EUI (encontrado Passo 2) e "server_address"(encontrado Passo 3 ), e colar no novo arquivo.
```
{
    "SX130x_conf": {
        "com_type": "SPI",
        "com_path": "/dev/spidev0.0",
        "lorawan_public": true,
        "clksrc": 0,
        "antenna_gain": 0, /* antenna gain, in dBi */
        "full_duplex": false,
        "fine_timestamp": {
            "enable": false,
            "mode": "all_sf" /* high_capacity or all_sf */
        },
        "sx1261_conf": {
            "spi_path": "/dev/spidev0.1",
            "rssi_offset": 0, /* dB */
            "spectral_scan": {
                "enable": false,
                "freq_start": 915000000,
                "nb_chan": 8,
                "nb_scan": 2000,
                "pace_s": 10
            },
            "lbt": {
                "enable": false /* LBT for 500 Khz channels is not supported */
            }
        },
        "radio_0": {
            "enable": true,
            "type": "SX1250",
            "freq": 917200000,
            "rssi_offset": -215.4,
            "rssi_tcomp": {
                "coeff_a": 0,
                "coeff_b": 0,
                "coeff_c": 20.41,
                "coeff_d": 2162.56,
                "coeff_e": 0
            },
            "tx_enable": true,
            "tx_freq_min": 915000000,
            "tx_freq_max": 928000000,
            "tx_gain_lut": [
                {
                    "rf_power": 12,
                    "pa_gain": 0,
                    "pwr_idx": 15
                },
                {
                    "rf_power": 13,
                    "pa_gain": 0,
                    "pwr_idx": 16
                },
                {
                    "rf_power": 14,
                    "pa_gain": 0,
                    "pwr_idx": 17
                },
                {
                    "rf_power": 15,
                    "pa_gain": 0,
                    "pwr_idx": 19
                },
                {
                    "rf_power": 16,
                    "pa_gain": 0,
                    "pwr_idx": 20
                },
                {
                    "rf_power": 17,
                    "pa_gain": 0,
                    "pwr_idx": 22
                },
                {
                    "rf_power": 18,
                    "pa_gain": 1,
                    "pwr_idx": 1
                },
                {
                    "rf_power": 19,
                    "pa_gain": 1,
                    "pwr_idx": 2
                },
                {
                    "rf_power": 20,
                    "pa_gain": 1,
                    "pwr_idx": 3
                },
                {
                    "rf_power": 21,
                    "pa_gain": 1,
                    "pwr_idx": 4
                },
                {
                    "rf_power": 22,
                    "pa_gain": 1,
                    "pwr_idx": 5
                },
                {
                    "rf_power": 23,
                    "pa_gain": 1,
                    "pwr_idx": 6
                },
                {
                    "rf_power": 24,
                    "pa_gain": 1,
                    "pwr_idx": 7
                },
                {
                    "rf_power": 25,
                    "pa_gain": 1,
                    "pwr_idx": 9
                },
                {
                    "rf_power": 26,
                    "pa_gain": 1,
                    "pwr_idx": 11
                },
                {
                    "rf_power": 27,
                    "pa_gain": 1,
                    "pwr_idx": 14
                }
            ]
        },
        "radio_1": {
            "enable": true,
            "type": "SX1250",
            "freq": 917900000,
            "rssi_offset": -215.4,
            "rssi_tcomp": {
                "coeff_a": 0,
                "coeff_b": 0,
                "coeff_c": 20.41,
                "coeff_d": 2162.56,
                "coeff_e": 0
            },
            "tx_enable": false
        },
        "chan_multiSF_All": {
            "spreading_factor_enable": [
                5,
                6,
                7,
                8,
                9,
                10,
                11,
                12
            ]
        },
        "chan_multiSF_0": {
            "enable": true,
            "radio": 0,
            "if": -400000
        }, /* Freq : 916.8 MHz*/
        "chan_multiSF_1": {
            "enable": true,
            "radio": 0,
            "if": -200000
        }, /* Freq : 917.0 MHz*/
        "chan_multiSF_2": {
            "enable": true,
            "radio": 0,
            "if": 0
        }, /* Freq : 917.2 MHz*/
        "chan_multiSF_3": {
            "enable": true,
            "radio": 0,
            "if": 200000
        }, /* Freq : 917.4 MHz*/
        "chan_multiSF_4": {
            "enable": true,
            "radio": 1,
            "if": -300000
        }, /* Freq : 917.6 MHz*/
        "chan_multiSF_5": {
            "enable": true,
            "radio": 1,
            "if": -100000
        }, /* Freq : 917.8 MHz*/
        "chan_multiSF_6": {
            "enable": true,
            "radio": 1,
            "if": 100000
        }, /* Freq : 918.0 MHz*/
        "chan_multiSF_7": {
            "enable": true,
            "radio": 1,
            "if": 300000
        }, /* Freq : 918.2 MHz*/
        "chan_Lora_std": {
            "enable": true,
            "radio": 0,
            "if": 300000,
            "bandwidth": 500000,
            "spread_factor": 8, /* Freq : 904.6 MHz*/
            "implicit_hdr": false,
            "implicit_payload_length": 17,
            "implicit_crc_en": false,
            "implicit_coderate": 1
        },
        "chan_FSK": {
            "enable": false,
            "radio": 1,
            "if": 300000,
            "bandwidth": 125000,
            "datarate": 50000
        } /* Freq : 868.8 MHz*/
    },
    "gateway_conf": {
        "gateway_ID": "XXXXXXXXXXXXXXXX",
        /* change with default server address/ports */
        "server_address": "nam1.cloud.thethings.network",
        "serv_port_up": 1700,
        "serv_port_down": 1700,
        /* adjust the following parameters for your network */
        "keepalive_interval": 10,
        "stat_interval": 30,
        "push_timeout_ms": 100,
        /* forward only valid packets */
        "forward_crc_valid": true,
        "forward_crc_error": false,
        "forward_crc_disabled": false,
        /* GPS configuration */
        "gps_tty_path": "/dev/ttyS0",
        /*"fake_gps":"true",*/
        /* GPS reference coordinates */
        "ref_latitude": 0,
        "ref_longitude": 0,
        "ref_altitude": 0,
        /* Beaconing parameters */
        "beacon_period": 0, /* disable class B beacon */
        "beacon_freq_hz": 923300000,
        "beacon_datarate": 12,
        "beacon_bw_hz": 500000,
        "beacon_power": 14,
        "beacon_infodesc": 0,
        "servers": [
            {
                "gateway_ID": "XXXXXXXXXXXXXXXX",
                "server_address": "nam1.cloud.thethings.network",
                "serv_port_up": 1700,
                "serv_port_down": 1700,
                "serv_enabled": true
            }
        ]
    },
    "debug_conf": {
        "ref_payload": [
            {
                "id": "0xCAFE1234"
            },
            {
                "id": "0xCAFE2345"
            }
        ],
        "log_file": "loragw_hal.log"
    }
}
```
Rode o servidor com o comando a seguir, onde global_conf.json.sx1250.AU915 é o nome do arquivo criado anteriormente
```
#cd ~/Documents/sx1302_hal/packet_forwarder/
./lora_pkt_fwd -c global_conf.json.sx1250.AU915
```
## 5 Iniciar como serviço

No seu diretório de construção, há um arquivo de script:
```
cd ~/Documents/sx1302_hal/tools/systemd
```
Editar o arquivo abaixo: 
```
nano lora_pkt_fwd.service
```
Verificar o caminho para os arquivos no WorkingDirectory e ExecStart, como abaixo:
```
[Unit]
Description=LoRa Packet Forwarder
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
WorkingDirectory=/home/pi/Documents/sx1302_hal/packet_forwarder
ExecStart=/home/pi/Documents/sx1302_hal/packet_forwarder/lora_pkt_fwd -c /home/pi/Documents/sx1302_hal/packet_forwarder/global_conf.json.sx1250.AU915
Restart=always
RestartSec=30
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=lora_pkt_fwd

[Install]
WantedBy=multi-user.target
```
Em seguida, copie o arquivo de serviço, registre-o e habilite:
```
sudo cp lora_pkt_fwd.service /etc/systemd/system
```
Habilite o serviço para inicialização automática com os comandos:
```
sudo systemctl daemon-reload
sudo systemctl enable lora_pkt_fwd.service
sudo reboot
```

