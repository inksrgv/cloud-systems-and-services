# Nginx Extra

<details>
<summary> Техническое задание </summary>

Попробовать взломать nginx другой команды. Проверить минимум три уязвимости - например path traversal, перебор страниц через ffuf и/или любые другие на ваш выбор.
Взлом считается успешным, если вы попали туда, куда не планировалось попадать пользователю, даже если там ничего нет. Успешность взлома не влияет на оценку лаб обеих команд. 
В отчет приложить скрины попыток взлома, описание уязвимостей, на которые проверяли и итог - успешен взлом или нет.
Тк открывать такие нжинксы в интернет не лучшая идея, для решения лабы предлагаю либо встретиться с другой командой в одном помещении и поднять локальную сеть, либо запустить у себя нжинкс с их конфигом. Но при этом настройки нжинкс со стороны админа и изменения в конфиге считаться попыткой взлома не будут.
Просьба договориться с другими командами кто кого ломает. Взлом нжинкса одной команды не должен фигурировать в отчетах больше двух раз.

</details>

## Настройка доступа сервиса для команды коллег "Тучки"

Прокидываем 443 порт на белый ip адрес при помощи сервиса `ngrok`:
```bash
ngrok http 443
```

Cо стороны коллег была проделана аналогичная команда и впоследствии работа шла с полученным адресом. 

Для начала было решено проверить заголовки: 
![image](https://github.com/user-attachments/assets/49ce2a6c-4e1a-48fe-816d-e1fe7f0537bd)


Потом проверим сервис с помощью `nikto`:
```bash
root@irina-honor:~# nikto -h https://5a06-3-127-202-173.ngrok-free.app/
- Nikto v2.1.5
---------------------------------------------------------------------------
+ Target IP:          3.125.209.94
+ Target Hostname:    5a06-3-127-202-173.ngrok-free.app
+ Target Port:        443
---------------------------------------------------------------------------
+ SSL Info:        Subject: /CN=*.eu.ngrok.io
                   Ciphers: TLS_AES_128_GCM_SHA256
                   Issuer:  /C=US/O=Let's Encrypt/CN=R10
+ Start Time:         2024-11-26 22:18:13 (GMT3)
---------------------------------------------------------------------------
+ Server: No banner retrieved
+ The anti-clickjacking X-Frame-Options header is not present.
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Server is using a wildcard certificate: '*.eu.ngrok.io'
+ Hostname '5a06-3-127-202-173.ngrok-free.app' does not match certificate's CN '*.eu.ngrok.io'
+ 6544 items checked: 0 error(s) and 1 item(s) reported on remote host
+ End Time:           2024-11-26 22:21:47 (GMT3) (8 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
root@irina-honor:~#
```


Посмотрим ещё `nmap`:
```bash
root@irina-honor:~# nmap --script ssl-enum-ciphers -p 443 5a06-3-127-202-173.ngrok-free.app
Starting Nmap 7.80 ( https://nmap.org ) at 2024-11-26 22:08 MSK
Nmap scan report for 5a06-3-127-202-173.ngrok-free.app (3.125.223.134)
Host is up (0.0026s latency).
Other addresses for 5a06-3-127-202-173.ngrok-free.app (not scanned): 18.158.249.75 3.124.142.205 3.125.209.94 3.125.102.39 18.192.31.165 2a05:d014:21b:8e02::6e:2 2a05:d014:21b:8e02::6e:5 2a05:d014:21b:8e01::6e:4 2a05:d014:21b:8e01::6e:1 2a05:d014:21b:8e00::6e:3 2a05:d014:21b:8e00::6e:0
rDNS record for 3.125.223.134: ec2-3-125-223-134.eu-central-1.compute.amazonaws.com

PORT    STATE SERVICE
443/tcp open  https
| ssl-enum-ciphers:
|   TLSv1.0:
|     ciphers:
|       TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA (secp256r1) - A
|       TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA (secp256r1) - A
|       TLS_RSA_WITH_AES_128_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_AES_256_CBC_SHA (rsa 2048) - A
|     compressors:
|       NULL
|     cipher preference: server
|   TLSv1.1:
|     ciphers:
|       TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA (secp256r1) - A
|       TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA (secp256r1) - A
|       TLS_RSA_WITH_AES_128_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_AES_256_CBC_SHA (rsa 2048) - A
|     compressors:
|       NULL
|     cipher preference: server
|   TLSv1.2:
|     ciphers:
|       TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA (secp256r1) - A
|       TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256 (secp256r1) - A
|       TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA (secp256r1) - A
|       TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384 (secp256r1) - A
|       TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256 (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256 (secp256r1) - A
|       TLS_RSA_WITH_AES_128_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_AES_128_GCM_SHA256 (rsa 2048) - A
|       TLS_RSA_WITH_AES_256_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_AES_256_GCM_SHA384 (rsa 2048) - A
|     compressors:
|       NULL
|     cipher preference: client
|_  least strength: A

Nmap done: 1 IP address (1 host up) scanned in 4.95 seconds
```

А ещё `openssl`:
```bash
root@irina-honor:~# openssl s_client -connect 5a06-3-127-202-173.ngrok-free.app:443 -tls1_2
CONNECTED(00000003)
depth=2 C = US, O = Internet Security Research Group, CN = ISRG Root X1
verify return:1
depth=1 C = US, O = Let's Encrypt, CN = E5
verify return:1
depth=0 CN = *.ngrok-free.app
verify return:1
---
Certificate chain
 0 s:CN = *.ngrok-free.app
   i:C = US, O = Let's Encrypt, CN = E5
   a:PKEY: id-ecPublicKey, 256 (bit); sigalg: ecdsa-with-SHA384
   v:NotBefore: Oct  5 17:09:30 2024 GMT; NotAfter: Jan  3 17:09:29 2025 GMT
 1 s:C = US, O = Let's Encrypt, CN = E5
   i:C = US, O = Internet Security Research Group, CN = ISRG Root X1
   a:PKEY: id-ecPublicKey, 384 (bit); sigalg: RSA-SHA256
   v:NotBefore: Mar 13 00:00:00 2024 GMT; NotAfter: Mar 12 23:59:59 2027 GMT
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIDkjCCAxigAwIBAgISBGoHbuaaLjdNEAcgeQCpmrLvMAoGCCqGSM49BAMDMDIx
m7viAjEAniZyZ7sPcxjL5Nw6F5WUYM7zeT6KOXJzAmgrS7uxwrfpqAnml/1qqe/S
VtDRfK3X
-----END CERTIFICATE-----
subject=CN = *.ngrok-free.app
issuer=C = US, O = Let's Encrypt, CN = E5
---
No client certificate CA names sent
Peer signing digest: SHA256
Peer signature type: ECDSA
Server Temp Key: X25519, 253 bits
---
SSL handshake has read 2436 bytes and written 323 bytes
Verification: OK
---
New, TLSv1.2, Cipher is ECDHE-ECDSA-AES128-GCM-SHA256
Server public key is 256 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-ECDSA-AES128-GCM-SHA256
    Session-ID: 5F1CE1D598CD3C65D98D4E3D915BD57456D3032B1279F4FBCCB954A77ED9B647
    Session-ID-ctx:
    Master-Key: D8EE319E146FC33A0C84AA7D8797D348F9049F0A57660DF84D434851C6EF93DF4E1B8AD005A969CAB5B9DB211675C2E4
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket:
    0000 - 52 80 3d 4b b8 6a f9 0e-4b 1b ec f8 d7 28 ce 7f   R.=K.j..K....(..
    0010 - e0 ea 8a a5 23 2e ff ad-d9 77 5b 6b 5f a0 5f 90   ....#....w[k_._.
    0020 - 46 32 c8 fc ca 06 5d 68-b0 36 be f4 2f 4d 4c 83   F2....]h.6../ML.
    0030 - 42 50 2c f1 cd 95 4e fc-d7 fd 10 6d e8 0e e3 65   BP,...N....m...e
    0040 - a7 cd f7 c3 ef 1b 79 16-9f 40 77 47 18 3f 4c 35   ......y..@wG.?L5
    0050 - f2 c2 6a d8 05 4e 21 cb-9a 8c ed 69 d6 60 55 8f   ..j..N!....i.`U.
    0060 - 16 bd af 6f 0f ba 3c 2b-05 89 68 25 3c da 19 b0   ...o..<+..h%<...
    0070 - fa 72 21 20 21 f8 d9 79-ec                        .r! !..y.

    Start Time: 1732646777
    Timeout   : 7200 (sec)
    Verify return code: 0 (ok)
    Extended master secret: yes
---
closed
root@irina-honor:~#
```


Теперь `ffuf`:
```bash
root@irina-honor:~# ffuf -w ~/SecLists-master/Discovery/Web-Content/big.txt -u https://5a06-3-127-202-173.ngrok-free.app/FUZZ -t 50 -mc all

        /'___\  /'___\           /'___\
       /\ \__/ /\ \__/  __  __  /\ \__/
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
         \ \_\   \ \_\  \ \____/  \ \_\
          \/_/    \/_/   \/___/    \/_/

       v1.1.0
________________________________________________

 :: Method           : GET
 :: URL              : https://5a06-3-127-202-173.ngrok-free.app/FUZZ
 :: Wordlist         : FUZZ: /root/SecLists-master/Discovery/Web-Content/big.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 50
 :: Matcher          : Response status: all
________________________________________________

.listing                [Status: 404, Size: 70, Words: 4, Lines: 4]
!_archives              [Status: 404, Size: 70, Words: 4, Lines: 4]
0                       [Status: 404, Size: 70, Words: 4, Lines: 4]
<...>
:: Progress: [20478/20478] :: Job [1/1] :: 46 req/sec :: Duration: [0:07:19] :: Errors: 546 ::
root@irina-honor:~#
```

Мы бы хотели и дальше повзламывать nginx, но все реквесты и лимиты на месяц в ngrok уже закончились
![image](https://github.com/user-attachments/assets/d1d2b4af-4e7d-461e-a16d-0473365ed590)

