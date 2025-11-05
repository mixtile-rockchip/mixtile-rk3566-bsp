# mixtile-rk3566-bsp

## Build

### 1. Install Docker

```bash
# Ubuntu / Debian
sudo apt-get update
sudo apt-get install docker.io
```
### 2. Build Docker image

```bash
git clone https://github.com/mixtile-rockchip/mixtile-rk3566-bsp.git
cd mixtile-rk3566-bsp
sudo docker build -t rk3566-bsp-build-env ./
```
```bash
# View the built docker image
mixtile~/build$ sudo docker images
REPOSITORY          TAG       IMAGE ID       CREATED         SIZE
rk3566-bsp-build-env   latest    ad587023e07a   3 minutes ago   1.7GB
```
### 3. Install Repo

```bash
# repo depends on python3, make sure you have a python3 environment before using it
# sudo apt-get install -y python3 && sudo ln -snf /usr/bin/python3 /usr/bin/python
mkdir -p ~/.bin
PATH="${HOME}/.bin:${PATH}"
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/.bin/repo
chmod a+rx ~/.bin/repo
```
### 4. Download code

```bash
# Before downloading git, install git tool and configure Git information
# sudo apt-get install -y git git-lfs
# git config --global user.email "you@example.com"
# git config --global user.name "Your Name"
# You can create a new workspace directory and do the following
mkdir workspace && cd workspace
repo init -u https://github.com/mixtile-rockchip/manifests.git -b rk3566-rk3568-rkr6 -m rk3566_rk3568_linux6.1_release.xml
repo sync -c
repo forall -c '
  if [ -d .git ]; then
    echo "===> $REPO_PATH"
    git lfs install --local
    git lfs pull || true
  fi
'
```
### 5. Build using Docker

```bash
# You need to execute the following command in the source directory, the blade3 directory
sudo docker run --privileged -it --rm -v $(pwd):/workspace rk3566-bsp-build-env
cd /workspace
./build.sh chip    # Follow the prompts to select rk3566 and rockchip_xxxx_defconfig
./build.sh all
```
### 6. Build output

```bash
Directory: output/update/Image
.
├── boot.img -> ../../../kernel/boot.img
├── MiniLoaderAll.bin -> ../../../u-boot/rk35xx_spl_loader_v1.13.112.bin
├── misc.img -> ../../firmware/misc.img
├── oem.img -> ../../firmware/oem.img
├── package-file
├── parameter.txt -> ../../../device/rockchip/.chips/rk35xx/parameter.txt
├── recovery.img -> ../../../buildroot/output/rockchip_rk35xx_recovery/images/recovery.img
├── rootfs.img -> ../../../debian/linaro-rootfs.img
├── uboot.img -> ../../../u-boot/uboot.img
├── update.img
├── update.raw.img
└── userdata.img -> ../../firmware/userdata.img
```
### 7. Upgrade Firmware
