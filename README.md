# TLG_JoinCaptchaBot

<p align="center">
    <img width="100%" height="100%" src="https://gist.githubusercontent.com/J-Rios/05d7a8fc04166fa19f31a9608033d10b/raw/32dee32a530c0a0994736fe2d02a1747478bd0e3/captchas.png">
</p>

Telegram Bot to verify if a new member joining a group is a human.
Upon a new user join a group, the Bot send an image-based [captcha challenge](https://en.wikipedia.org/wiki/CAPTCHA) that must be solved to allow the user stay in the group. If the new user fails to solve the captcha within a set time limit, they are removed from the group. Additionally, any message from a new user that includes a URL prior to the completion of the captcha will be considered Spam and will be deleted.


* . Set Telegram Bot account Token (get it from @BotFather) in "src/settings.py" file:

    ```python
    'TOKEN' : 'XXXXXXXXX:XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX'
    ```

## Configuration

All Bot configurations can be done easily by modifying them in the **"src/settings.json"** file.

For more experienced users, you can use environment variables to setup all that properties (this is really useful for advance deployment when using [Virtual Environments](https://docs.python.org/3/tutorial/venv.html) and/or [Docker](https://docs.docker.com/get-started/) to isolate the Bot process execution).


## Usage

To ease it usage in Linux, a **Makefile** is provided.

- Check usage help information:

    ```bash
    make
    ```

- Launch the Bot:

    ```bash
    make run
    ```

- Check if the Bot is running:

    ```bash
    make status
    ```

- Stop the Bot:

    ```bash
    make kill
    ```

## Systemd service

For systemd based systems, you can setup the Bot as daemon service.

To do that, you need to create a new service description file for the Bot as follow:

```bash
[vim or nano] /etc/systemd/system/tlg_joincaptcha_bot.service
```

File content:

```bash
[Unit]
Description=Telegram Join Captcha Bot Daemon
Wants=network-online.target
After=network-online.target

[Service]
Type=forking
WorkingDirectory=/path/to/src/
ExecStart=/path/to/tools/run
ExecReload=/path/to/tools/kill

[Install]
WantedBy=multi-user.target
```

Then, to add the new service into systemd, you should enable it:

```bash
systemctl enable --now tlg_joincaptcha_bot.service
```

Now, you can start the service (Bot) by:

```bash
systemctl start tlg_joincaptcha_bot.service
```

You can check if the service (Bot) is running by:

```bash
systemctl status tlg_joincaptcha_bot.service
```

Remember that, if you wan't to disable it, you should execute:

```bash
systemctl disable tlg_joincaptcha_bot.service
```

## Docker

You can also run the bot on [Docker](https://docs.docker.com/get-started/). This allows easy server migration and automates the installation and setup. Look at the [docker specific documentation](docker/README.md) for more details about how to create a Docker Container for Captcha Bot.

## Make Bot Private

By default, the Bot is **Public**, so any Telegram user can add and use the Bot in any group, but you can set it to be **Private** so the Bot just can be used in allowed groups (Bot owner allows them with **/allow_group** command).

You can set Bot to be Private in "settings.py" file:

```python
"BOT_PRIVATE" : True,
```

**Note:** If you have a Public Bot and set it to Private, it will leave any group where is not allowed to be used when a new user joins.

**Note:** Telegram Private Groups could changes their chat ID when it become a public super-group, so the Bot will leave the group and the owner has to set the new group chat ID with /allow_group.

## Scalability (Polling or Webhook)

By default, Bot checks and receives updates from Telegram Servers by **Polling** (it periodically requests and gets from Telegram Server if there is any new updates in the Bot account corresponding to that Bot Token), this is really simple and can be used for low to median scale Bots. However, you can configure the Bot to use **Webhook** instead if you expect to handle a large number of users/groups (with webhook, the Telegram Server is the one that will connect to you machine and send updates to the Bot when there is any new update).

To use Webhook instead Polling, you need a signed certificate file in the system, you can create the key file and self-sign the cert through openssl tool:

```bash
openssl req -newkey rsa:2048 -sha256 -nodes -keyout private.key -x509 -days 3650 -out cert.pem
```

Once you have the key and cert files, setup the next lines in "settings.py" file to point to expected host system address, port, path and certificate files:

```python
"WEBHOOK_IP": "0.0.0.0",
"WEBHOOK_PORT": 8443,
"WEBHOOK_PATH": "/"
"WEBHOOK_CERT" : SCRIPT_PATH + "/cert.pem",
"WEBHOOK_CERT_PRIV_KEY" : SCRIPT_PATH + "/private.key",
```

(Optional) In case you want to use a reverse proxy between Telegram Server and the system that runs the Bot, you need to setup the Proxy Webhook URL setting:

```python
"WEBHOOK_URL": "https://example.com:8443/"
```

Then, you need to change Bot connection mode from polling to webhook by setting to True the next configuration:

```python
"CAPTCHABOT_USE_WEBHOOK": True,
```

To go back and use Polling instead Webhook, just set the config back to False:

```python
"CAPTCHABOT_USE_WEBHOOK": False,
```
