<p style="font-size:14px" align="right">
Join our telegram <a href="https://t.me/kjnotes" target="_blank"><img src="https://user-images.githubusercontent.com/50621007/168689534-796f181e-3e4c-43a5-8183-9888fc92cfa7.png" width="30"/></a>
Visit our website <a href="https://kjnodes.com/" target="_blank"><img src="https://user-images.githubusercontent.com/50621007/168689709-7e537ca6-b6b8-4adc-9bd0-186ea4ea4aed.png" width="30"/></a>
</p>

<p align="center">
  <img height="100" height="auto" src="https://user-images.githubusercontent.com/50621007/172356220-b8326ceb-9950-4226-b66e-da69099aaf6e.png">
</p>

# Kujira Testnet Icin Node Kurulumu — harpoon-4

Orijinal Dokumantasyon:
>- [Validator setup instructions](https://docs.kujira.app/run-a-node)

Gezgin:
>-  https://kujira.explorers.guru/


## Hardware Requirements
Like any Cosmos-SDK chain, the hardware requirements are pretty modest.

### Minimum Hardware Requirements
 - 3x CPUs
 - 4GB RAM
 - 80GB Disk
 
### Optimal Hardware Requirements 
 - 4x CPUs
 - 8GB RAM
 - 200GB of storage (SSD or NVME)
 
## Kujira Fullnode'unuzu Kurun
### Hizli script kurulum
Aşağıdaki otomatik komut dosyasını kullanarak kujira fullnode'unuzu birkaç dakika içinde kurabilirsiniz. Doğrulayıcı düğüm adınızı girmenizi isteyecektir!
```
wget https://raw.githubusercontent.com/berkcaNode/KujiraTr-Script-Kurulum/main/kujiraberk.sh && bash kujiraberk.sh
```

## Yukleme Sonrası

Kurulum bittiğinde lütfen değişkenleri sisteme yükleyin
```
source $HOME/.bash_profile
```

Ardından, doğrulayıcınızın blokları senkronize ettiğinden emin olmalısınız. Senkronizasyon durumunu kontrol etmek için aşağıdaki komutu kullanabilirsiniz.
```
kujirad status 2>&1 | jq .SyncInfo
```

### Cuzdan Olustur
Yeni cüzdan oluşturmak için aşağıdaki komutu kullanabilirsiniz. Hatırlatıcıyı (mnemonic) kaydetmeyi unutmayın
```
kujirad keys add $WALLET
```

### Cüzdan bilgilerini kaydet

Cüzdan adresi ekle
```
WALLET_ADDRESS=$(kujirad keys show $WALLET -a)
```

valoper adresi ekle
```
VALOPER_ADDRESS=$(kujirad keys show $WALLET --bech val -a)
```

Değişkenleri sisteme yükle
```
echo 'export WALLET_ADDRESS='${WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export VALOPER_ADDRESS='${VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Cuzdanınıza jeton alın
Dogrulayıcı olusturmak icin once cuzdanınıza testnet jetonlari almaniz gerekir.
Cuzdanınıza jeton almak için [Kujira discord server]([(https://discord.gg/TdQEj2Jy)]katılın ve **#faucet-requests** kanalina gidin

Musluktan jeton talep etmek için:
```
!faucet <YOUR_WALLET_ADDRESS>
```

### Doğrulayıcı oluştur (dugumunuz senkronize olmadiysa bu adima gecmeyiniz)

Doğrulayıcı oluşturmadan önce lütfen en az 1 kujiranız olduğundan (1 kujira 1000000 ukujiye eşittir) ve düğümünüzün senkronize edildiğinden emin olun.

Cüzdan bakiyenizi kontrol etmek için:

```
kujirad query bank balances $WALLET_ADDRESS
```
> Cüzdanınızda bakiye gozukmuyorsa muhtemelen, dugumunuz hala esitleniyordur. Lütfen senkronizasyonun bitmesini bekleyin ve ardından devam edin

Aşağıdaki doğrulayıcı çalıştırma komutunuzu oluşturmak için

```
kujirad tx staking create-validator \
  --amount 1999000ukuji \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(kujirad tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $CHAIN_ID \
  --fees 250ukuji
```


## Faydalı komutlar

### Servis Yonetimi

Günlükleri kontrol et

```
journalctl -fu kujirad -o cat
```

Servisi baslat

```
systemctl start kujirad
```

Servisi durdur
```
systemctl stop kujirad
```

Servisi yeniden baslat
```
systemctl restart kujirad
```

### Düğüm bilgisi

Senkronizasyon bilgisi
```
kujirad status 2>&1 | jq .SyncInfo
```

Doğrulayıcı bilgisi
```
kujirad status 2>&1 | jq .ValidatorInfo
```

Dugum bilgisi
```
kujirad status 2>&1 | jq .NodeInfo
```

Dugum kimligini göster
```
kujirad tendermint show-node-id
```

Jeton transferi
```
kujirad tx bank send $WALLET_ADDRESS <TO_WALLET_ADDRESS> 10000000ukuji --fees 250ukuji
```

### Oylama
```
kujirad tx gov vote 1 yes --from $WALLET --chain-id=$CHAIN_ID --fees 250ukuji
```



### Dogrulayıcı yonetimi
Dogrulayıcıyı duzenle
```
kujirad tx staking edit-validator \
--moniker=$NODENAME \
--identity=1C5ACD2EEF363C3A \
--website="http://kjnodes.com" \
--details="Providing professional staking services with high performance and availability. Find me at Discord: kjnodes#8455 and Telegram: @kjnodes" \
--chain-id=$CHAIN_ID \
--from=$WALLET \
--fees 250ukuji
```

Unjail validator
```
kujirad tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$CHAIN_ID \
  --gas=auto \
  --fees 250ukuji
```

### Dugumu sil
Bu komutlar düğümü sunucudan tamamen kaldıracaktır. Kendi sorumluluğunuzda kullanın!

```
systemctl stop kujirad
systemctl disable kujirad
rm /etc/systemd/system/kujira* -rf
rm $(which kujirad) -rf
rm $HOME/.kujira* -rf
rm $HOME/kujira-core -rf
```
