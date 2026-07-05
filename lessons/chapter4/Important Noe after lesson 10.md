# systemctl daemon-reload tells systemd:

“Reload your service unit files because I edited one.”

When you edit:

/etc/systemd/system/frontend.service

systemd does not automatically re-read the changed file immediately.

So if you run:

sudo systemctl start frontend.service

without daemon reload, systemd may still use the old cached service definition:

ExecStart=/usr/bin/npm start

After:

sudo systemctl daemon-reload

systemd reads the updated file and knows the new command:

ExecStart=/usr/bin/npx serve -s build -l 3000

# If a process is managed by systemd, manage it with systemctl, not kill, unless you have a very specific emergency reason.
