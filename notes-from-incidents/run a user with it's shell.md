Because >> is shell redirection, not part of echo.

This command:

sudo -u grace bash -c 'echo "hello" >> /srv/techcorp/project-atlas/docs/release-notes.txt'

means:

Run a new bash shell as user grace, and inside that shell run:

echo "hello" >> file

Why important?

If you wrote:

sudo -u grace echo "hello" >> /srv/techcorp/project-atlas/docs/release-notes.txt

only echo "hello" runs as grace.

But the >> file part is handled by your current shell, before/around sudo, so the file write may happen as your user, not grace.

So we use:

bash -c '... >> file'

to make sure both the command and the redirection run as grace.

Real meaning:

sudo -u grace bash -c 'echo "hello" >> file'

= “Grace tries to append hello to this file.”


# we can also add && to run another command in same bash"
