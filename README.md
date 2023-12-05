# FastAPI example

Install fastapi with:
    
```
pip install fastapi
pip install "uvicorn[standard]"

pip install python-multipart    # for file upload
```

Run the server with:

```
uvicorn main:app --reload
```


## To configure HTTPS:

Following the guide: https://dev.to/rajshirolkar/fastapi-over-https-for-development-on-windows-2p7d
0. Install Brew: (https://linux.how2shout.com/how-to-install-brew-ubuntu-20-04-lts-linux/)

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

(echo; echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"') >> /home/$USER/.bashrc

eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"

brew doctor
```

1. Install https://github.com/FiloSottile/mkcert:
```
sudo apt install libnss3-tools
brew install mkcert
```

2. Generate the certificate and add to your CA with the mkcert utility
```
mkcert -install
mkcert localhost
```
