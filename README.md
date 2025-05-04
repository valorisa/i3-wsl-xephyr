# i3-wsl-xephyr

Un guide complet pour installer et configurer i3 window manager sur Windows Subsystem for Linux (WSL) avec Xephyr comme serveur X imbriqué.

![i3 sur WSL avec Xephyr](https://raw.githubusercontent.com/username/i3-wsl-xephyr/main/screenshots/demo des matières

- [Introduction](#introduction)
- [Pourquoi utiliser i3 avec WSL et Xephyr?](#pourquoi-utiliser-i3-avec-wsl-et-xephyr)
- [Prérequis](#prérequis)
- [Installation](#installation)
  - [Étape 1: Configurer WSL](#étape-1-configurer-wsl)
  - [Étape 2: Installer les dépendances](#étape-2-installer-les-dépendances)
  - [Étape 3: Installer i3](#étape-3-installer-i3)
  - [Étape 4: Installer Xephyr](#étape-4-installer-xephyr)
  - [Étape 5: Configurer le serveur X](#étape-5-configurer-le-serveur-x)
- [Configuration](#configuration)
  - [Configuration de base i3](#configuration-de-base-i3)
  - [Configuration de Xephyr](#configuration-de-xephyr)
  - [Script de lancement automatique](#script-de-lancement-automatique)
- [Utilisation](#utilisation)
  - [Lancer i3 dans Xephyr](#lancer-i3-dans-xephyr)
  - [Raccourcis clavier essentiels](#raccourcis-clavier-essentiels)
  - [Personnalisation avancée](#personnalisation-avancée)
- [Problèmes connus et solutions](#problèmes-connus-et-solutions)
- [Amélioration des performances](#amélioration-des-performances)
- [Alternatives](#alternatives)
- [Ressources supplémentaires](#ressources-supplémentaires)
- [Contribution](#contribution)
- [Licence](#licence)

## Introduction

Ce dépôt fournit un guide étape par étape et les outils nécessaires pour installer et configurer i3, un gestionnaire de fenêtres en mosaïque populaire sous Linux, dans un environnement WSL (Windows Subsystem for Linux) en utilisant Xephyr comme serveur X imbriqué.

i3 est apprécié pour sa légèreté, sa flexibilité, et son efficacité pour la gestion des fenêtres au clavier. Xephyr est un serveur X11 qui s'exécute dans une fenêtre, permettant d'exécuter des applications graphiques Linux dans un environnement X indépendant.

## Pourquoi utiliser i3 avec WSL et Xephyr?

- **Productivité améliorée**: i3 offre une gestion de fenêtres efficace basée sur le clavier.
- **Environnement de développement Linux**: Profitez des outils Linux sans quitter Windows.
- **Isolation propre**: Xephyr évite les problèmes de compatibilité avec certains serveurs X Windows.
- **Intégration transparente**: Fonctionne comme une application Windows tout en gérant un environnement Linux complet.
- **Stabilité accrue**: Évite certains problèmes courants entre WSL et les serveurs X directs.

## Prérequis

- Windows 10 ou 11 avec WSL2 installé et configuré
- Distribution Linux (Ubuntu recommandé) installée via WSL
- Accès administrateur sur Windows (pour certaines installations)
- Connaissances de base sur les commandes Linux et la navigation dans les fichiers

## Installation

### Étape 1: Configurer WSL

Si WSL n'est pas déjà configuré, ouvrez PowerShell en tant qu'administrateur et exécutez:

```powershell
# Installer WSL
wsl --install

# Redémarrer votre ordinateur si demandé
```

Si vous avez déjà WSL mais que vous devez passer à WSL2:

```powershell
# Définir WSL2 comme version par défaut
wsl --set-default-version 2

# Vérifier l'installation
wsl -l -v
```

### Étape 2: Installer les dépendances

Ouvrez votre terminal WSL (Ubuntu recommandé) et installez les dépendances requises:

```bash
# Mettre à jour le système
sudo apt update && sudo apt upgrade -y

# Installer les dépendances de base
sudo apt install -y \
    build-essential \
    git \
    libxcb1-dev \
    libxcb-keysyms1-dev \
    libpango1.0-dev \
    libxcb-util0-dev \
    libxcb-icccm4-dev \
    libyajl-dev \
    libstartup-notification0-dev \
    libxcb-randr0-dev \
    libev-dev \
    libxcb-cursor-dev \
    libxcb-xinerama0-dev \
    libxcb-xkb-dev \
    libxkbcommon-dev \
    libxkbcommon-x11-dev \
    autoconf \
    xutils-dev \
    libtool \
    automake \
    libxcb-shape0-dev \
    libxcb-xrm-dev \
    meson \
    ninja-build
```

### Étape 3: Installer i3

Il existe deux approches pour installer i3: via les dépôts ou en compilant depuis les sources.

#### Option A: Installation depuis les dépôts (plus simple)

```bash
sudo apt install -y i3 i3status i3lock dmenu
```

#### Option B: Compilation depuis les sources (recommandé pour WSL)

La compilation depuis les sources est souvent plus fiable pour WSL car certaines versions packagées peuvent avoir des problèmes:

```bash
# Cloner le dépôt i3
git clone https://github.com/i3/i3.git ~/i3-src
cd ~/i3-src

# Créer et entrer dans le dossier de compilation
mkdir -p build && cd build

# Configurer la compilation
meson setup . ..

# Compiler
ninja

# Installer
sudo ninja install
```

### Étape 4: Installer Xephyr

Xephyr est essentiel pour notre approche car il crée un serveur X imbriqué:

```bash
sudo apt install -y xserver-xephyr x11-apps
```

### Étape 5: Configurer le serveur X

Configurez la variable DISPLAY pour pouvoir communiquer avec le serveur X:

```bash
# Ajouter ces lignes à votre ~/.bashrc ou ~/.zshrc
echo '# Configuration pour i3 et Xephyr
export DISPLAY="$(awk "/nameserver / {print \$2; exit}" /etc/resolv.conf):0.0"' >> ~/.bashrc

# Recharger le fichier de configuration
source ~/.bashrc
```

## Configuration

### Configuration de base i3

Créez les dossiers nécessaires pour la configuration d'i3:

```bash
mkdir -p ~/.config/i3
```

Créez un fichier de configuration i3 de base:

```bash
cat > ~/.config/i3/config  ~/start-i3-xephyr.sh /dev/null

# Lancer Xephyr
Xephyr -ac -br -noreset -screen "$RESOLUTION" "$DISPLAY_ID" &
XEPHYR_PID=$!

# Attendre que Xephyr soit prêt
sleep 1

# Définir DISPLAY pour pointer vers Xephyr
export DISPLAY="$DISPLAY_ID"

# Lancer i3
i3 &
I3_PID=$!

# Gérer la terminaison propre
function cleanup {
    echo "Arrêt de i3 et Xephyr..."
    kill $I3_PID 2>/dev/null
    kill $XEPHYR_PID 2>/dev/null
    exit 0
}

# Capturer Ctrl+C pour une sortie propre
trap cleanup SIGINT SIGTERM

# Garder le script en cours d'exécution jusqu'à ce que l'utilisateur appuie sur Ctrl+C
echo "i3 est en cours d'exécution dans Xephyr. Appuyez sur Ctrl+C pour quitter."
wait $I3_PID
cleanup
EOL

# Rendre le script exécutable
chmod +x ~/start-i3-xephyr.sh
```

### Script de lancement automatique

Pour simplifier encore plus le lancement, créez un script qui vérifie et configure tout ce qui est nécessaire:

```bash
cat > ~/launch-i3-wsl.sh  /dev/null; then
    echo "Xephyr n'est pas installé. Installation en cours..."
    sudo apt update && sudo apt install -y xserver-xephyr x11-apps
fi

# Vérifier que i3 est installé
if ! command -v i3 &> /dev/null; then
    echo "i3 n'est pas installé. Installation en cours..."
    sudo apt install -y i3 i3status i3lock dmenu
fi

# Configurer la variable DISPLAY pour WSL
HOST_IP=$(awk '/nameserver / {print $2; exit}' /etc/resolv.conf)
export DISPLAY="${HOST_IP}:0.0"

# Vérifier si la connexion au serveur X Windows est possible
if ! xset q &>/dev/null; then
    echo "Impossible de se connecter au serveur X sur Windows."
    echo "Assurez-vous qu'un serveur X (comme MobaXterm, VcXsrv, etc.) est en cours d'exécution sur Windows."
    echo "Vérifiez aussi que le pare-feu Windows autorise les connexions au port 6000."
    exit 1
fi

# Lancer i3 dans Xephyr
~/start-i3-xephyr.sh
EOL

# Rendre le script exécutable
chmod +x ~/launch-i3-wsl.sh
```

## Utilisation

### Lancer i3 dans Xephyr

Pour démarrer i3 dans Xephyr, suivez ces étapes:

1. Assurez-vous qu'un serveur X est en cours d'exécution sur Windows (MobaXterm, VcXsrv, X410, etc.)
2. Ouvrez votre terminal WSL
3. Exécutez le script de lancement:

```bash
~/launch-i3-wsl.sh
```

Vous devriez voir une fenêtre Xephyr s'ouvrir avec i3 fonctionnant à l'intérieur.

### Raccourcis clavier essentiels

Voici les raccourcis clavier de base pour i3 (par défaut, $mod est Alt):

| Raccourci | Action |
|-----------|--------|
| $mod+Entrée | Ouvrir un terminal |
| $mod+d | Ouvrir le lanceur d'applications (dmenu) |
| $mod+1 à $mod+0 | Changer d'espace de travail |
| $mod+Shift+1 à $mod+Shift+0 | Déplacer la fenêtre vers un espace de travail |
| $mod+h | Diviser horizontalement |
| $mod+v | Diviser verticalement |
| $mod+f | Mode plein écran |
| $mod+s | Mode empilé |
| $mod+w | Mode à onglets |
| $mod+e | Mode fractionné |
| $mod+Shift+q | Fermer la fenêtre |
| $mod+Shift+r | Redémarrer i3 |
| $mod+Shift+e | Quitter i3 |
| $mod+r | Mode redimensionnement (utiliser les flèches ensuite) |

### Personnalisation avancée

Pour une personnalisation plus avancée, vous pouvez modifier le fichier de configuration i3:

```bash
nano ~/.config/i3/config
```

Quelques personnalisations populaires:

#### Changement de la touche $mod

Pour utiliser la touche Windows au lieu de Alt:

```
set $mod Mod4  # Au lieu de Mod1
```

#### Ajout des espaces entre les fenêtres

Si vous voulez ajouter des "gaps" entre les fenêtres, vous pouvez installer i3-gaps (une version modifiée de i3) à la place de i3 standard. Voici comment:

```bash
# Installer les dépendances
sudo apt install -y libxcb1-dev libxcb-keysyms1-dev libpango1.0-dev \
  libxcb-util0-dev libxcb-icccm4-dev libyajl-dev \
  libstartup-notification0-dev libxcb-randr0-dev \
  libev-dev libxcb-cursor-dev libxcb-xinerama0-dev \
  libxcb-xkb-dev libxkbcommon-dev libxkbcommon-x11-dev \
  autoconf libxcb-xrm0 libxcb-xrm-dev automake libxcb-shape0-dev

# Cloner i3-gaps
git clone https://github.com/Airblader/i3 ~/i3-gaps
cd ~/i3-gaps

# Pour WSL, il est souvent recommandé d'utiliser un commit spécifique stable
git checkout fdf5d3a  # Ce commit est connu pour être stable sous WSL

# Compiler et installer
autoreconf --force --install
rm -rf build
mkdir -p build && cd build
../configure --prefix=/usr --sysconfdir=/etc --disable-sanitizers
make
sudo make install
```

Puis ajoutez à votre configuration i3:

```
# Espacement entre les fenêtres
gaps inner 10
gaps outer 5
```

#### Personnalisation de la barre de statut

Pour une barre de statut plus informative, vous pouvez installer et configurer i3blocks ou polybar à la place de i3status:

```bash
sudo apt install -y i3blocks
```

Puis modifiez la configuration de la barre dans ~/.config/i3/config:

```
bar {
    status_command i3blocks
    position top
    # ... autres configurations ...
}
```

## Problèmes connus et solutions

### Problème: "Cannot open display"

**Symptômes**: Les applications graphiques affichent "Cannot open display" ou "Xephyr cannot open host display".

**Solutions**:
1. Vérifiez que le serveur X est en cours d'exécution sur Windows
2. Confirmez que la variable DISPLAY est bien configurée:
   ```bash
   export DISPLAY=$(awk '/nameserver / {print $2; exit}' /etc/resolv.conf):0.0
   ```
3. Vérifiez les paramètres du pare-feu Windows: autorisez les connexions entrantes sur le port 6000

### Problème: "Another window manager seems to be running"

**Symptômes**: i3 affiche "ERROR: Another window manager seems to be running"

**Solutions**:
1. Utilisez Xephyr comme serveur X imbriqué:
   ```bash
   Xephyr -ac -br -noreset -screen 1280x720 :1 &
   export DISPLAY=:1
   i3
   ```
2. Si vous n'utilisez pas Xephyr, vérifiez qu'aucun autre gestionnaire de fenêtres n'est en cours d'exécution

### Problème: Interface graphique lente ou saccadée

**Symptômes**: Performances lentes, animations saccadées

**Solutions**:
1. Réduisez la résolution de Xephyr:
   ```bash
   Xephyr -ac -br -noreset -screen 1024x768 :1
   ```
2. Désactivez les effets graphiques dans la configuration i3
3. Vérifiez la charge CPU/mémoire avec `top` ou `htop`

### Problème: Raccourcis clavier qui ne fonctionnent pas

**Symptômes**: Certains raccourcis i3 ne répondent pas

**Solutions**:
1. Vérifiez que les raccourcis ne sont pas capturés par Windows
2. Modifiez la touche $mod dans la configuration i3:
   ```
   set $mod Mod1  # Alt
   # ou
   set $mod Mod4  # Touche Windows
   ```

## Amélioration des performances

Pour améliorer les performances de i3 sous WSL avec Xephyr:

1. **Désactivez les effets inutiles**:
   - Évitez les compositeurs comme compton/picom
   - Utilisez des fonds d'écran unis ou désactivez-les

2. **Optimisez WSL**:
   Créez/modifiez le fichier `.wslconfig` dans votre dossier utilisateur Windows (`C:\Users\`):
   ```
   [wsl2]
   memory=4GB
   processors=2
   swap=2GB
   ```

3. **Réduisez le polling**:
   Dans votre configuration i3, augmentez les délais de rafraîchissement:
   ```
   force_display_urgency_hint 500 ms
   ```

## Alternatives

Si cette approche ne fonctionne pas bien pour vous, voici quelques alternatives:

1. **VcXsrv avec i3**: Utilisez VcXsrv en mode plein écran avec i3 directement, sans Xephyr
   ```bash
   export DISPLAY=$(awk '/nameserver / {print $2; exit}' /etc/resolv.conf):0.0
   i3
   ```

2. **i3-on-wsl**: Un dépôt GitHub avec une configuration i3 préoptimisée pour WSL
   ```bash
   git clone https://github.com/Lamarcke/i3-on-wsl.git
   ```

3. **Machine virtuelle Linux**: Pour une expérience plus fiable, vous pouvez utiliser une machine virtuelle Linux complète

## Ressources supplémentaires

- [Documentation officielle i3](https://i3wm.org/docs/)
- [Guide utilisateur i3](https://i3wm.org/docs/userguide.html)
- [Xephyr - XWiki](https://www.x.org/wiki/Xephyr/)
- [WSL - Documentation Microsoft](https://learn.microsoft.com/fr-fr/windows/wsl/)
- [i3-on-wsl (GitHub)](https://github.com/Lamarcke/i3-on-wsl)

## Contribution

Les contributions à ce projet sont les bienvenues! N'hésitez pas à:

1. Fork ce dépôt
2. Créer une branche pour vos modifications (`git checkout -b feature/amelioration`)
3. Committer vos changements (`git commit -am 'Ajout de fonctionnalité X'`)
4. Pousser vers la branche (`git push origin feature/amelioration`)
5. Créer une Pull Request

## Licence

Ce projet est sous licence MIT. Voir le fichier [LICENSE](LICENSE) pour plus de détails.

---

Créé avec ❤️ pour la communauté i3 et WSL. Faites bon usage de votre nouvel environnement de travail!

Citations:
[1] https://pplx-res.cloudinary.com/image/private/user_uploads/40251661/ofPYrOhhVlJsIHS/Screenshot4.jpg
[2] https://github.com/Lamarcke/i3-on-wsl
[3] https://github.com/pzmarzly/xephyr-now/blob/master/README.md
[4] https://ha.zardo.us/blog/i3wsl
[5] https://www.reddit.com/r/bashonubuntuonwindows/comments/zffsva/my_little_frankenstein_fully_working_i3_on_wsl2/?tl=fr
[6] https://learn.microsoft.com/fr-fr/windows/wsl/troubleshooting
[7] https://github.com/microsoft/wslg/issues/154
[8] https://gist.github.com/taoy/8b44beef41f833280e0edc96c8e3306a
[9] https://misterderpie.com/posts/ubuntu-wsl2-i3/
[10] https://github.com/mikeroyal/WSL-Guide
[11] https://labex.io/fr/tutorials/git-installing-a-git-server-299593
[12] https://gist.github.com/dcasati/fe4e8b366ea0e006566968f6a0e860c2
[13] https://www.youtube.com/watch?v=1YVx_C2MZOg
[14] https://github.com/ethanhs/WSL-PROGRAMS
[15] https://github.com/budRich/xwmplay/blob/master/README.md
[16] https://www.reddit.com/r/bashonubuntuonwindows/comments/zffsva/my_little_frankenstein_fully_working_i3_on_wsl2/
[17] https://gist.github.com/tdcosta100/385636cbae39fc8cd0937139e87b1c74?permalink_comment_id=3937026
[18] https://github.com/Xyene/wsl-dotfiles
[19] https://doc.fedora-fr.org/wiki/I3wm
[20] https://learn.microsoft.com/fr-fr/windows/wsl/tutorials/wsl-git
[21] https://github.com/sirredbeard/awesome-wsl

---
