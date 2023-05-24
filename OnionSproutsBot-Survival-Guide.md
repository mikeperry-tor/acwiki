OnionSproutsBot is installed in telegram-bot-01 machine, with SSH fingerprints:
* `3072 SHA256:YZDOLtAVAVdsXWL6NuwY0zQmkjhFdGtWY7EWgHoIjEE root@telegram-bot-01.torproject.org (RSA)`
* `256 SHA256:ly7O04pZvAugOjEb1aJ+cn5kI6KNYjC89MIvMgdDhmk root@telegram-bot-01.torproject.org (ED25519)`

(Re)starting OnionSproutsBot
---------------------

1. Log into telegram-bot-01 machine.
2. Change to the telegrambot user by running `sudo -u telegrambot -s`
3. Re)start the onionsproutsbot process via its systemd script: `systemctl --user [start|stop|status] onionsproutsbot`.

Updating to a new version
-------------------------

1. Log into telegram-bot-01 machine.
2. Change to the telegrambot user by running `sudo -u telegrambot -s`
3. Update the repo `cd ~/onionsproutsbot && git pull`
4. Update the translations, copying the ones are completed (or above 90%) in [weblate](https://hosted.weblate.org/projects/tor/onionsprouts-bot/) from the translations repo: `cd ~/translation && git pull && cp <...>.po ~/onionsproutsbot/OnionSproutsBot/locales`
5. Activate the virtualenv `. ~/venv/bin/activate`
6. Install the latest dependencies of the bot `pip install -r ~/onionsproutsbot/requirements.txt`
7. Restart onionsprouts bot `systemctl --user restart onionsproutsbot`

If the new version is producing errors try to delete `~/database/OSB.db` and restart again. OnionSproutsBot runs from the git repo in `~/onionsproutsbot`, the version present there will be the one running.