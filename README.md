

# Przewodnik Uruchomienia Operatora EigenLayer na Testnecie Goerli

Przewodnik ten krok po kroku przeprowadzi Cię przez proces stania się operatorem na testnecie Goerli dla EigenLayer.

## Oficjalna Dokumentacja

Możesz zapoznać się z oficjalną dokumentacją dostępną na [docs.eigenlayer.xyz/operator-guides/operator-installation/](https://docs.eigenlayer.xyz/operator-guides/operator-installation/).

## To jakieś czary?

Te instrukcje wymagają pewien poziom umiejętności obsługi komputera za pomocą terminala/wiersza poleceń. Jeśli wydaje się to dla Ciebie zbyt skomplikowane, mogę zająć się tym procesem za Ciebie. Oczywiście w zamian za wirtualne szekle - 95 usdt/c, a w cenie:

- postawienie serwera - potrzeba udostępnienia wyrtualnej karty płatniczej (mogę też przprowadzić przez proces na Contabo)
- konfiguracja serwera - potrzebne dane do logowania ssh (w przypadku pominięcia pierszego punktu)
- instalacja i rejestracja operatora - potrzebne dane do uzupełniania pliku `metadata.json`

Jeśłi jesteś zainteresowany zapraszam do kontaktu - **jan.misiurek@gmail.com**

# INSTRUKCJE

## Wymagania Wstępne

### Konfiguracja Serwera

1. Potrzebujemy serwera do uruchomienia noda. VPS od [Contabo](https://contabo.com/en/vps/) będzie odpowiedni.
   - Zalecany wybór to **CLOUD VPS 2** z **400 GB/SDD** dla Storage Type oraz **Ubuntu 20.04** jako Image.

### Przygotowanie Serwera

1. Oczekuj na e-mail z danymi do logowania do VPS.

2. Po zalogowaniu wykonaj aktualizację VPS:

   ```bash
   sudo apt-get update && sudo apt-get upgrade -y
   ```

3. Zainstaluj Docker:

   ```bash
   sudo apt install docker.io
   ```

   - Sprawdź wersję, aby upewnić się, że instalacja przebiegła pomyślnie:

     ```bash
     docker --version
     ```
   ![Alt text](images/1.png)

    
4. Zainstaluj Docker-Compose:

   ```bash
   sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   sudo chmod +x /usr/local/bin/docker-compose

   docker-compose --version
   ```
   ![Alt text](images/2.png)


5. Zainstaluj GO:

   - Pobierz:

     ```bash
     wget https://golang.org/dl/go1.21.4.linux-amd64.tar.gz
     ```

   - Rozpakuj:

     ```bash
     tar -C /usr/local -xzf go1.21.4.linux-amd64.tar.gz
     ```

   - Dodaj do środowiska:

     ```bash
     export PATH=$PATH:/usr/local/go/bin
     ```

   - Sprawdź wersję:

     ```bash
     go version
     ```
   ![Alt text](images/3.png)

## Instalacja EigenLayer CLI

1. Klonuj repozytorium i zbuduj projekt:

   ```bash
   git clone https://github.com/Layr-Labs/eigenlayer-cli.git
   cd eigenlayer-cli
   mkdir -p build
   go build -o build/eigenlayer cmd/eigenlayer/main.go
   ```
   ![Alt text](images/4.png)


2. Skopiuj plik do systemu:

   ```bash
   cp ./build/eigenlayer /usr/local/bin/
   ```

   - Sprawdź, czy komenda `eigenlayer` działa:

     ```bash
     eigenlayer
     ```
   ![Alt text](images/5.png)
   
## Generowanie Kluczy

1. Generuj klucze prywatne ECDSA i BLS za pomocą CLI. Zapisz je w bezpiecznym miejscu. `<YOUR_NAME>` zastąp wybraną nazwą.

   ```bash
   eigenlayer operator keys create --key-type ecdsa <YOUR_NAME>
   eigenlayer operator keys create --key-type bls <YOUR_NAME>
   ```
   ![Alt text](images/6.png)

   - Aby sprawdzić klucze publiczne:

     ```bash
     eigenlayer operator keys list
     ```

## Konfiguracja Plików

1. Wygeneruj i edytuj pliki konfiguracyjne:

   ```bash
   eigenlayer operator config create
   ```

   - Jeśli wybierzesz opcję `yes`, Eigen zapyta o kilka danych do wstępnej konfiguracji.

2. Opublikuj plik `metadata.json`, aby zapewnić jego dostęp dla EigenLayer. Możesz wykorzystać GitHubi - utwórz nowe repozytorium (koniecznie publiczne) i stwórz w nim plik `metadata.json`. Są też inne opcje dohostingu jak np. Pastebin. Poniższy schemat uzupełniij swoimi danymi:

   ```json
   {
     "name": "<OPERATOR_NAME>",
     "website": "<WEBSITE>",
     "description": "<DESCRIPTION>",
     "logo": "<LOGO_URL>",
     "twitter": "<TWITTER_HANDLE>"
   }
   ```

3. Edytuj plik `operator.yaml`, wklejając RAW URL do Twojego pliku `metadata.json` i uzupełniając pozostałe dane:

   ```bash
    nano operator.yaml

   ```


   ```yaml
   operator:
     address: <YOUR_ADDRESS>
     earnings_receiver_address: <YOUR_ADDRESS>
     delegation_approver_address: "0x0000000000000000000000000000000000000000"
     staker_opt_out_window_blocks: 0
     metadata_url: <YOUR_METADATA_URL>
     el_delegation_manager_address: 0x1b7b8F6b258f95Cf9596EabB9aa18B62940Eb0a8
     eth_rpc_url: https://rpc.ankr.com/eth_goerli
     private_key_store_path: /root/.eigenlayer/operator_keys/<WALLET_NAME>.ecdsa.key.json
     signer_type: local_keystore
     chain_id: 5
   ```

po edycji naciskamy CRTL+X, następnie Y, a na koniec ENTER

## Rejestracja Operatora

1. Przed rejestracją, upewnij się, że masz tokeny Goerli ETH do opłacenia gas. Można je zdobyć na [faucet Alchemy](https://www.alchemy.com/faucets/ethereum-goerli).

2. Zarejestruj operatora:

   ```bash
   eigenlayer operator register operator.yaml
   ```

3. Sprawdź status:

   ```bash
   eigenlayer operator status operator.yaml
   ```

   ![Alt text](images/7.png)

   Status można również sprawdzić na stronie [Goerli EigenLayer Operator](https://goerli.eigenlayer.xyz/operator).



