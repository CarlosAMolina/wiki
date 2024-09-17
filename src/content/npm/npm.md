# npm

## Global installation without sudo

This section explains how to execute commands like the following one without sudo:

```bash
npm install --global web-ext
```

First, check if you can run the previous command without sudo privileges. I think that if npm was installed using [nvm](https://github.com/nvm-sh/nvm), sudo privileges are not necessary. This is an important point with nvm because the solution below of [adding prefix is incompatible with nvm](https://github.com/nvm-sh/nvm/issues/2340#issuecomment-1799261678).

If you need sudo privileges, run:

```bash
npm config set prefix '~/.local/'
# Required bash configuration: `export PATH=~/.local/bin/:$PATH`
```

Resources:

- <https://stackoverflow.com/a/59227497>
