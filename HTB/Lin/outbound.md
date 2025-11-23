# Unfortunately, as I haven't pre-prepped this box, I haven't written all the commands used.
There were 2 CVES mainly used:
- https://github.com/hakaioffsec/CVE-2025-49113-exploit
- https://gist.github.com/strikoder/49a945eeff34362d58ae0eea2caa2fa5
we found creds in cat /var/www/html/roundcube/configs/config.inc.php
Then more creds in the db.
we decrypted that by decrypt.sh in the /var/www/html/roundcube dir.
