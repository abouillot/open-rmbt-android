# rtr 2

🧭 Prérequis

WSL2 Ubuntu

Docker installé et fonctionnel

Projet cloné sur WSL, par ex. :

Install wsl
Install docker
Install wsl debian

activate docker in wsl debian
Launch wsl debian

```
wsl -d debian
```

sudo apt-get update
sudo apt-get install git wget unzip

🧩 1. Installer les SDK Tools Android (hors Docker)

On installe les Command Line Tools une seule fois dans WSL :

```
mkdir -p $HOME/android-sdk/cmdline-tools
cd $HOME/android-sdk/cmdline-tools
wget https://dl.google.com/android/repository/commandlinetools-linux-11076708_latest.zip -O cmdtools.zip
unzip cmdtools.zip -d tmp
mv tmp/cmdline-tools latest
rm -rf tmp cmdtools.zip
```
Vérification :
```
ls $HOME/android-sdk/cmdline-tools/latest/bin
# Doit afficher sdkmanager, avdmanager, etc.
```

📁 2. Préparer la configuration du projet

```
cd ~
git clone https://github.com/rtr-nettest/open-rmbt-android rmbt_clean
# fix config.json if needed Le projet nécessite un fichier de config :
# cd ~/akostest-open-rmbt-android/app/src/openRmbt
# cp config-example.json config.json
cd ~/rmbt_clean/
```
récupération du sous projet `netmonster_core`
```
git submodule update netmonster_core
```


🐳 3. Lancer le conteneur Docker avec le bon montage
```
sudo docker run --rm -it \
  -v "$PWD":/src \
  -v "$HOME/android-sdk":/opt/android-sdk-linux \
  -w /src \
  mobiledevops/android-sdk-image \
  bash
```
Dans le conteneur, vérifier que le SDK est bien monté :
```
ls /opt/android-sdk-linux/cmdline-tools/latest/bin
# sdkmanager doit être présent
```

🔑 4. Accepter les licences Android SDK (dans le conteneur)

```
export ANDROID_SDK_ROOT=/opt/android-sdk-linux

yes | /opt/android-sdk-linux/cmdline-tools/latest/bin/sdkmanager \
  --sdk_root=/opt/android-sdk-linux --licenses
```

📦 5. Installer les SDK nécessaires
```
/opt/android-sdk-linux/cmdline-tools/latest/bin/sdkmanager \
  --sdk_root=/opt/android-sdk-linux \
  "platforms;android-34" \
  "platforms;android-35" \
  "build-tools;34.0.0"
```
🛠️ 6. Corriger l’erreur Git “dubious ownership”

(Uniquement si elle apparaît)
```
git config --global --add safe.directory /src
```


🏗️ 7. Construire l’APK

Toujours dans le conteneur :
```
./gradlew :app:assembleOpenRmbtDebug
```
📱 8. Récupérer l'APK

Il se trouve dans :
```
app/build/outputs/apk/openRmbt/debug/
```
Ex. :
```
ls app/build/outputs/apk/openRmbt/debug/
```
Accessible directement dans ton projet WSL, car /src était monté via $PWD.
