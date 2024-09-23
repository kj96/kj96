#### **Generate `.htpasswd` for Basic Authentication (Optional)**

You can enable basic HTTP authentication for an extra layer of security. You'll need to generate an `.htpasswd` file using `htpasswd` (from `apache2-utils`).

First, install the `htpasswd` utility (if you don't have it):

bash

Copy code

`sudo apt-get install apache2-utils`

Then, generate the `.htpasswd` file:

bash

Copy code

`mkdir -p auth htpasswd -Bc auth/htpasswd username`

- Replace `username` with the desired username.
- You will be prompted to enter a password.

This will generate a file called `htpasswd` in the `auth` directory, which contains the username and encrypted password.