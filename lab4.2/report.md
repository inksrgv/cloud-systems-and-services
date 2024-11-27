
```bash
sgsoul@irina-honor:~$ vault server -dev
==> Vault server configuration:

Administrative Namespace:
             Api Address: http://127.0.0.1:8200
                     Cgo: disabled
         Cluster Address: https://127.0.0.1:8201
   Environment Variables: DBUS_SESSION_BUS_ADDRESS, DISPLAY, GOTRACEBACK, HOME, HOSTTYPE, LANG, LESSCLOSE, LESSOPEN, LOGNAME, LS_COLORS, MOTD_SHOWN, NAME, OLDPWD, PATH, PULSE_SERVER, PWD, SHELL, SHLVL, TERM, USER, VAULT_ADDR, VAULT_TOKEN, WAYLAND_DISPLAY, WSL2_GUI_APPS_ENABLED, WSLENV, WSL_DISTRO_NAME, WSL_INTEROP, WT_PROFILE_ID, WT_SESSION, XDG_DATA_DIRS, XDG_RUNTIME_DIR, _
              Go Version: go1.22.8
              Listener 1: tcp (addr: "127.0.0.1:8200", cluster address: "127.0.0.1:8201", disable_request_limiter: "false", max_request_duration: "1m30s", max_request_size: "33554432", tls: "disabled")
               Log Level:
                   Mlock: supported: true, enabled: false
           Recovery Mode: false
                 Storage: inmem
                 Version: Vault v1.18.2, built 2024-11-20T11:24:56Z
             Version Sha: e36bac59ddb8e10e8912c0ddb44416c850939855

==> Vault server started! Log data will stream in below:

2024-11-24T17:03:42.949+0300 [INFO]  proxy environment: http_proxy="" https_proxy="" no_proxy=""
2024-11-24T17:03:43.019+0300 [INFO]  core: successful mount: namespace="" path=secret/ type=kv version="v0.20.0+builtin"
WARNING! dev mode is enabled! In this mode, Vault runs entirely in-memory
and starts unsealed with a single unseal key. The root token is already
authenticated to the CLI, so you can immediately begin using Vault.

You may need to set the following environment variables:

    $ export VAULT_ADDR='http://127.0.0.1:8200'

The unseal key and root token are displayed below in case you want to
seal/unseal the Vault or re-authenticate.

Unseal Key: JmUBxIIgAR/9exc=
Root Token: hvs.05R3263NjM

Development mode should NOT be used in production installations!

2024-11-24T18:05:32.227+0300 [INFO]  core: enabled credential backend: path=userpass/ type=userpass version="v1.18.2+builtin.vault"
```

![Снимок экрана 2024-11-24 170558](https://github.com/user-attachments/assets/473c802d-aed9-44c2-b3c7-a08a01815758)

![Снимок экрана 2024-11-24 180437](https://github.com/user-attachments/assets/c5173430-359a-4b63-824d-2f660fc76625)

```bash
ngrok http 8200
```

![Снимок экрана 2024-11-25 204340](https://github.com/user-attachments/assets/1e02da9c-c876-4cea-918a-c329eab875e9)

```bash
sgsoul@irina-honor:/mnt/c/Users/Irina/Desktop/корзина2/clouds/galaxy$ ll
total 0
drwxrwxrwx 1 sgsoul sgsoul 4096 Nov 25 20:02 ./
drwxrwxrwx 1 sgsoul sgsoul 4096 Oct 11 23:29 ../
-rwxrwxrwx 1 sgsoul sgsoul   35 Oct 12 00:05 ansible.cfg*
-rwxrwxrwx 1 sgsoul sgsoul   53 Oct 12 00:08 hosts.ini*
-rwxrwxrwx 1 sgsoul sgsoul  167 Oct 12 00:00 playbook.yml*
drwxrwxrwx 1 sgsoul sgsoul 4096 Oct 11 23:50 roles/
-rwxrwxrwx 1 sgsoul sgsoul   49 Nov 25 20:03 vault_credentials.yml*
sgsoul@irina-honor:/mnt/c/Users/Irina/Desktop/корзина2/clouds/galaxy$ ansible-vault encrypt vault_credentials.yml
[WARNING]: Ansible is being run in a world writable directory (/mnt/c/Users/Irina/Desktop/корзина2/clouds/galaxy),
ignoring it as an ansible.cfg source. For more information see
https://docs.ansible.com/ansible/devel/reference_appendices/config.html#cfg-in-world-writable-dir
New Vault password:
Confirm New Vault password:
Encryption successful
sgsoul@irina-honor:/mnt/c/Users/Irina/Desktop/корзина2/clouds/galaxy$
```

![image](https://github.com/user-attachments/assets/2bc1f7c3-dd15-48df-b4ae-99ecb8a0fbc0)

![Снимок экрана 2024-11-25 201023](https://github.com/user-attachments/assets/3aa577df-3aa4-462d-a011-ef6f8303085a)

![image](https://github.com/user-attachments/assets/7994606d-145e-4200-9505-5d30d3375566)


![Снимок экрана 2024-11-25 205122](https://github.com/user-attachments/assets/df70c15c-4ee2-4e6a-ad7e-813cdf95fea2)
