# Python Issue Fixes and Suggestions

## Foreword
All our python projects use Pipenv. Make sure you've installed Pyenv and Pipenv.

## Delete and recreate your project pipenv
After trying any of the below fixes delete and recreate your pipenv to ensure it is fresh and up to date

Within the python project run
```shell-script
pipenv --rm
pipenv install --dev
```

## Update tools with brew
I recommend installing everything you can with `brew` and keeping them up to date.
Update and check brew installations with
```shell-script
brew update
brew upgrade
brew cleanup
brew doctor
```

A handy shortcut is adding
```shell-script
alias brewup='brew update; brew upgrade; brew cleanup; brew doctor'
```
to your `~/.zshrc` file (shell start up script) and running this regularly.

## Pyenv
Make sure pyenv is up to date and configured correctly. I recommend installing it with brew and keeping it up to date with the above commands. Check their docs for how to update it by other installation methods if required.
`pyenv rehash`

Check what `python` currently points to
`which python`

This should look like `"/Users/<USER>/.pyenv/shims/python"`. If it does not, make sure you have followed the pyenv installation instructions completely including adding the required lines to your `.zshrc` (or other shell startup script).


## Make sure xcode developer tools are installed
Sometimes mac OS updates lose your command line developer tools. Re/install them with.
```shell-script
sudo rm -rf /Library/Developer/CommandLineTools
xcode-select --install
```

## Update pip tools
Some of these depends on exactly how your tools are installed.

Update pip and package tools with `pip install -U pip setuptools wheel`

I recommend installing Pipenv with Brew but if you are using pipenv installed through pip then update it with `pip install -U pipenv`

## Clear the PyCharm cache
If you're having python issues in PyCharm it could be cache issues. 
Try invalidating and restarting by clicking `File -> Invalidate Caches / Restart...`.
