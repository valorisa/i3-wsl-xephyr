# i3-wsl-xephyr

Un guide complet pour installer et configurer i3 window manager sur Windows Subsystem for Linux (WSL) avec Xephyr comme serveur X imbriqué.

![i3 sur WSL avec Xephyr](https://github.com/valorisa/i3-wsl-xephyr/blob/main/Screenshot4.png)

- [i3-wsl-xephyr](#i3-wsl-xephyr)
  - [Introduction](#introduction)
  - [Pourquoi utiliser i3 avec WSL et Xephyr?](#pourquoi-utiliser-i3-avec-wsl-et-xephyr)
  - [Prérequis](#prérequis)
  - [Installation](#installation)
    - [Étape 1: Configurer WSL](#étape-1-configurer-wsl)
    - [Étape 2: Installer les dépendances](#étape-2-installer-les-dépendances)
    - [Étape 3: Installer i3](#étape-3-installer-i3)
      - [Option A: Installation depuis les dépôts (plus simple)](#option-a-installation-depuis-les-dépôts-plus-simple)
      - [Option B: Compilation depuis les sources (recommandé pour WSL)](#option-b-compilation-depuis-les-sources-recommandé-pour-wsl)
    - [Étape 4: Installer Xephyr](#étape-4-installer-xephyr)
    - [Étape 5: Configurer le serveur X pour WSL](#étape-5-configurer-le-serveur-x-pour-wsl)
  - [Configuration](#configuration)
    - [Configuration de base i3](#configuration-de-base-i3)
    - [Script de lancement pour i3 dans Xephyr (`start-i3-xephyr.sh`)](#script-de-lancement-pour-i3-dans-xephyr-start-i3-xephyrsh)
    - [Script de lancement principal (`launch-i3-wsl.sh`)](#script-de-lancement-principal-launch-i3-wslsh)
  - [Utilisation](#utilisation)
    - [Lancer i3 dans Xephyr](#lancer-i3-dans-xephyr)
    - [Raccourcis clavier essentiels](#raccourcis-clavier-essentiels)
    - [Personnalisation avancée](#personnalisation-avancée)
      - [Changement de la touche `$mod`](#changement-de-la-touche-mod)
      - [Ajout des espaces entre les fenêtres (i3-gaps)](#ajout-des-espaces-entre-les-fenêtres-i3-gaps)
      - [Personnalisation de la barre de statut](#personnalisation-de-la-barre-de-statut)
  - [Problèmes connus et solutions](#problèmes-connus-et-solutions)
    - [Problème: "Cannot open display"](#problème-cannot-open-display)
    - [Problème: "Another window manager seems to be running"](#problème-another-window-manager-seems-to-be-running)
    - [Problème: Interface graphique lente ou saccadée](#problème-interface-graphique-lente-ou-saccadée)
    - [Problème: Raccourcis clavier qui ne fonctionnent pas](#problème-raccourcis-clavier-qui-ne-fonctionnent-pas)
  - [Amélioration des performances](#amélioration-des-performances)
  - [Alternatives](#alternatives)
  - [Ressources supplémentaires](#ressources-supplémentaires)
  - [Contribution](#contribution)
  - [Licence](#licence)

## Introduction

Ce dépôt fournit un guide étape par étape et les outils nécessaires pour installer et configurer i3, un gestionnaire de fenêtres en mosaïque populaire sous Linux, dans un environnement WSL (Windows Subsystem for Linux) en utilisant Xephyr comme serveur X imbriqué.
i3 est apprécié pour sa légèreté, sa flexibilité, et son efficacité pour la gestion des fenêtres au clavier.
Xephyr est un serveur X11 qui s'exécute dans une fenêtre, permettant d'exécuter des applications graphiques Linux dans un environnement X indépendant et isolé.

## Pourquoi utiliser i3 avec WSL et Xephyr?

- **Productivité améliorée**: i3 offre une gestion de fenêtres efficace basée sur le clavier.
- **Environnement de développement Linux**: Profitez des outils Linux sans quitter Windows.
- **Isolation propre**: Xephyr évite les problèmes de compatibilité potentiels avec certains serveurs X Windows et permet d'avoir un environnement i3 dédié.
- **Intégration transparente**: Fonctionne comme une application Windows tout en gérant un environnement Linux complet.
- **Stabilité accrue**: Évite certains problèmes courants entre WSL et les serveurs X directs pour une session i3 complète.

## Prérequis

- Windows 10 ou 11 avec WSL2 installé et configuré (WSL1 peut fonctionner mais WSL2 est recommandé).
- Distribution Linux (Ubuntu recommandé) installée via WSL.
- Accès administrateur sur Windows (pour l'installation de WSL si ce n'est pas déjà fait).
- Un serveur X fonctionnel sur Windows (ex: VcXsrv, MobaXterm (X server intégré), X410, GWSL). Celui-ci est nécessaire pour que Xephyr puisse s'afficher.
- Connaissances de base sur les commandes Linux et la navigation dans les fichiers.

## Installation

### Étape 1: Configurer WSL

Si WSL n'est pas déjà configuré, ouvrez PowerShell en tant qu'administrateur et exécutez:

```powershell
# Installer WSL et la distribution Linux par défaut (Ubuntu)
wsl --install

# Redémarrer votre ordinateur si demandé
```

Si vous avez déjà WSL mais que vous devez passer à WSL2 (recommandé):

```powershell
# Lister les distributions installées et leur version
wsl -l -v

# Mettre à jour une distribution vers WSL2 (remplacer <DistroName>)
# wsl --set-version <DistroName> 2

# Définir WSL2 comme version par défaut pour les nouvelles installations
wsl --set-default-version 2
```

### Étape 2: Installer les dépendances

Ouvrez votre terminal WSL (Ubuntu recommandé) et installez les dépendances requises pour i3 et Xephyr:

```bash
# Mettre à jour le système
sudo apt update && sudo apt upgrade -y

# Installer les dépendances de compilation pour i3 et les outils X
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
    ninja-build \
    xserver-xephyr \
    x11-apps \
    dmenu \
    i3status \
    i3lock
```

### Étape 3: Installer i3

Il existe deux approches pour installer i3: via les dépôts (plus simple) ou en compilant depuis les sources (plus à jour, parfois plus stable sous WSL).

#### Option A: Installation depuis les dépôts (plus simple)

Cette option installe i3 et les outils associés directement depuis les dépôts de votre distribution.

```bash
sudo apt install -y i3
```

(Note: `i3status`, `i3lock`, `dmenu` devraient déjà être installés avec la commande de l'Étape 2).

#### Option B: Compilation depuis les sources (recommandé pour WSL)

La compilation depuis les sources est souvent plus fiable pour WSL car certaines versions packagées peuvent avoir des problèmes ou être moins récentes.

```bash
# Cloner le dépôt i3
git clone [https://github.com/i3/i3.git](https://github.com/i3/i3.git) ~/i3-src
cd ~/i3-src

# Créer et entrer dans le dossier de compilation
mkdir -p build && cd build

# Configurer la compilation (utilise le répertoire parent ../i3-src pour les sources)
meson ..

# Compiler
ninja

# Installer
sudo ninja install
```

### Étape 4: Installer Xephyr

Xephyr est essentiel pour notre approche car il crée un serveur X imbriqué. S'il n'a pas été installé à l'étape 2 :

```bash
sudo apt install -y xserver-xephyr x11-apps
```

### Étape 5: Configurer le serveur X pour WSL

Pour que les applications graphiques WSL (y compris Xephyr lui-même) puissent s'afficher sur votre serveur X Windows, la variable `DISPLAY` doit être correctement configurée.

```bash
# Ajoutez la configuration suivante à votre ~/.bashrc (ou ~/.zshrc si vous utilisez zsh)
# pour définir automatiquement la variable DISPLAY au démarrage de votre shell WSL.

CONFIG_BLOCK='
# Configuration de la variable DISPLAY pour les applications graphiques WSL
# Permet aux applications WSL de se connecter au serveur X de l''hôte Windows.
export DISPLAY_WSL_HOST="$(awk "/nameserver / {print \$2; exit}" /etc/resolv.conf):0.0"
# Pour que Xephyr (et d''autres applications graphiques) puisse se lancer,
# dé-commentez la ligne suivante ou assurez-vous que DISPLAY est défini sur cette valeur.
# export DISPLAY="$DISPLAY_WSL_HOST"
'

# Vérifie si le bloc de configuration existe déjà pour éviter les doublons
if ! grep -Fxq '# Configuration de la variable DISPLAY pour les applications graphiques WSL' ~/.bashrc; then
    echo "$CONFIG_BLOCK" >> ~/.bashrc
    echo "Configuration DISPLAY pour le serveur X de l'hôte ajoutée à ~/.bashrc."
    echo "Veuillez recharger votre shell (source ~/.bashrc) ou ouvrir un nouveau terminal."
else
    echo "La configuration DISPLAY pour le serveur X de l'hôte semble déjà exister dans ~/.bashrc."
fi

# Pour la session courante, vous pouvez exporter la variable manuellement si vous ne rechargez pas le shell :
# export DISPLAY="$(awk "/nameserver / {print \$2; exit}" /etc/resolv.conf):0.0"
```

**Note importante :** Le script `launch-i3-wsl.sh` (voir ci-dessous) s'occupera de définir `DISPLAY` correctement pour lancer Xephyr. La configuration ci-dessus dans `.bashrc` est utile si vous souhaitez lancer d'autres applications graphiques directement depuis WSL vers votre serveur X Windows.

## Configuration

### Configuration de base i3

Créez un fichier de configuration i3 de base. Vous pourrez le personnaliser ultérieurement.

```bash
mkdir -p ~/.config/i3

# Créez un fichier de configuration i3 de base.
cat > ~/.config/i3/config << EOL
# Fichier de configuration i3 de base
# Pour une configuration plus complète, vous pouvez copier le fichier par défaut
# (généralement dans /etc/i3/config ou /usr/local/etc/i3/config) et le modifier.

# Définir la touche Mod (Mod1 = Alt, Mod4 = Touche Windows)
set \$mod Mod1

# Police pour les titres de fenêtres, la barre d'état, etc.
font pango:monospace 8

# Utiliser Mouse+\$mod pour glisser les fenêtres flottantes
floating_modifier \$mod

# --- Raccourcis clavier essentiels ---
# Ouvrir un nouveau terminal
bindsym \$mod+Return exec i3-sensible-terminal

# Lancer dmenu (lanceur d'applications)
bindsym \$mod+d exec dmenu_run

# Tuer la fenêtre active
bindsym \$mod+Shift+q kill

# Changer le focus entre les fenêtres
bindsym \$mod+j focus left
bindsym \$mod+k focus down
bindsym \$mod+l focus up
bindsym \$mod+odiaeresis focus right # Remplacer 'odiaeresis' (ö) par la touche à droite de 'l' sur votre clavier (ex: semicolon pour QWERTY)

# Alternative avec les flèches
bindsym \$mod+Left focus left
bindsym \$mod+Down focus down
bindsym \$mod+Up focus up
bindsym \$mod+Right focus right

# Déplacer la fenêtre active
bindsym \$mod+Shift+j move left
bindsym \$mod+Shift+k move down
bindsym \$mod+Shift+l move up
bindsym \$mod+Shift+odiaeresis move right # Idem, adapter à votre clavier

# Alternative avec les flèches
bindsym \$mod+Shift+Left move left
bindsym \$mod+Shift+Down move down
bindsym \$mod+Shift+Up move up
bindsym \$mod+Shift+Right move right

# Diviser en orientation horizontale
bindsym \$mod+h split h

# Diviser en orientation verticale
bindsym \$mod+v split v

# Mettre la fenêtre active en mode plein écran
bindsym \$mod+f fullscreen toggle

# Changer le layout du conteneur
bindsym \$mod+s layout stacking
bindsym \$mod+w layout tabbed
bindsym \$mod+e layout toggle split

# Mettre une fenêtre en mode flottant et vice-versa
bindsym \$mod+Shift+space floating toggle

# Changer le focus entre les fenêtres flottantes et en tuiles
bindsym \$mod+space focus mode_toggle

# Espaces de travail (Workspaces)
set \$ws1 "1"
set \$ws2 "2"
set \$ws3 "3"
set \$ws4 "4"
set \$ws5 "5"
set \$ws6 "6"
set \$ws7 "7"
set \$ws8 "8"
set \$ws9 "9"
set \$ws10 "10"

# Aller à un espace de travail
bindsym \$mod+1 workspace \$ws1
bindsym \$mod+2 workspace \$ws2
bindsym \$mod+3 workspace \$ws3
bindsym \$mod+4 workspace \$ws4
bindsym \$mod+5 workspace \$ws5
bindsym \$mod+6 workspace \$ws6
bindsym \$mod+7 workspace \$ws7
bindsym \$mod+8 workspace \$ws8
bindsym \$mod+9 workspace \$ws9
bindsym \$mod+0 workspace \$ws10

# Déplacer la fenêtre active vers un espace de travail
bindsym \$mod+Shift+1 move container to workspace \$ws1
bindsym \$mod+Shift+2 move container to workspace \$ws2
bindsym \$mod+Shift+3 move container to workspace \$ws3
bindsym \$mod+Shift+4 move container to workspace \$ws4
bindsym \$mod+Shift+5 move container to workspace \$ws5
bindsym \$mod+Shift+6 move container to workspace \$ws6
bindsym \$mod+Shift+7 move container to workspace \$ws7
bindsym \$mod+Shift+8 move container to workspace \$ws8
bindsym \$mod+Shift+9 move container to workspace \$ws9
bindsym \$mod+Shift+0 move container to workspace \$ws10

# Redimensionner une fenêtre (entrer en mode redimensionnement)
bindsym \$mod+r mode "resize"

# Mode redimensionnement
mode "resize" {
        bindsym j resize shrink width 10 px or 10 ppt
        bindsym k resize grow height 10 px or 10 ppt
        bindsym l resize shrink height 10 px or 10 ppt
        bindsym odiaeresis resize grow width 10 px or 10 ppt # Idem, adapter à votre clavier

        bindsym Left resize shrink width 10 px or 10 ppt
        bindsym Down resize grow height 10 px or 10 ppt
        bindsym Up resize shrink height 10 px or 10 ppt
        bindsym Right resize grow width 10 px or 10 ppt

        # Retour au mode normal
        bindsym Return mode "default"
        bindsym Escape mode "default"
        bindsym \$mod+r mode "default"
}

# Barre d'état i3bar
bar {
        status_command i3status
        position top
        font pango:monospace 10
}

# Redémarrer i3 en place (pour prendre en compte les modifications de la configuration)
bindsym \$mod+Shift+r restart

# Quitter i3 (affiche une barre de confirmation)
bindsym \$mod+Shift+e exec "i3-nagbar -t warning -m 'Voulez-vous vraiment quitter i3 ? Ceci mettra fin à votre session Xephyr.' -B 'Oui, quitter i3' 'i3-msg exit'"
EOL
```

**Note sur la configuration i3 :** Les raccourcis `odiaeresis` (ö) sont utilisés comme exemple pour la touche à droite de `L` sur un clavier QWERTZ. Adaptez-les à votre disposition (souvent `semicolon` (;) sur QWERTY).

### Script de lancement pour i3 dans Xephyr (`start-i3-xephyr.sh`)

Ce script s'occupe de démarrer Xephyr et i3 à l'intérieur.

```bash
cat > ~/start-i3-xephyr.sh << EOL
#!/bin/bash
set -e # Quitter immédiatement si une commande échoue

# Résolution pour Xephyr (peut être personnalisée)
# Utilise le premier argument passé au script, sinon 1920x1080 par défaut.
RESOLUTION="\${1:-1920x1080}"
# Choisir un numéro d'affichage aléatoire et disponible pour Xephyr (entre :1 et :100)
# Ceci est une méthode simple ; une version plus robuste vérifierait si l'affichage est réellement libre.
XEPHYR_DISPLAY_NUM=":\$((\$RANDOM % 100 + 1))"

echo "Lancement de Xephyr sur l'affichage \$XEPHYR_DISPLAY_NUM avec la résolution \$RESOLUTION..."
# Options pour Xephyr:
# -ac : désactive le contrôle d'accès (nécessaire pour que i3 puisse se connecter facilement depuis WSL)
# -br : crée une fenêtre racine noire (sinon, elle peut être grise)
# -noreset : empêche la réinitialisation du serveur X à la fermeture du dernier client
# -screen RESOLUTION : définit la taille de l'écran de Xephyr
# \$XEPHYR_DISPLAY_NUM : le numéro d'affichage pour ce serveur Xephyr
Xephyr -ac -br -noreset -screen "\$RESOLUTION" "\$XEPHYR_DISPLAY_NUM" &
XEPHYR_PID=\$!

# Attendre que Xephyr soit prêt.
# Une meilleure méthode serait de sonder jusqu'à ce que l'affichage soit disponible (ex: avec xdpyinfo).
# Pour la simplicité, nous utilisons une pause. Augmentez si Xephyr prend plus de temps à démarrer.
sleep 2

# Exporter la variable DISPLAY pour pointer vers notre nouveau serveur Xephyr
# Toutes les applications graphiques lancées après cette ligne utiliseront ce serveur Xephyr.
export DISPLAY="\$XEPHYR_DISPLAY_NUM"
echo "Xephyr démarré (PID: \$XEPHYR_PID). Variable DISPLAY interne définie sur \$DISPLAY."

echo "Lancement d'i3..."
# Lancer i3 window manager. i3status et dmenu devraient être dans le PATH.
i3 &
I3_PID=\$!
echo "i3 démarré (PID: \$I3_PID)."

# Gérer la terminaison propre lorsque le script reçoit SIGINT (Ctrl+C) ou SIGTERM
cleanup() {
    echo "Arrêt de i3 et Xephyr..."
    # Tuer i3 d'abord, de manière la plus propre possible
    if kill -0 \$I3_PID 2>/dev/null; then
        i3-msg exit # Demande à i3 de quitter proprement s'il est toujours en cours
        # Attendre un peu qu'i3 se termine
        for _ in 1 2 3 4 5; do kill -0 \$I3_PID 2>/dev/null || break; sleep 0.5; done
        # Forcer si toujours en cours
        kill \$I3_PID 2>/dev/null
    fi
    
    # Tuer Xephyr
    if kill -0 \$XEPHYR_PID 2>/dev/null; then
        kill \$XEPHYR_PID 2>/dev/null
    fi
    
    wait \$I3_PID 2>/dev/null
    wait \$XEPHYR_PID 2>/dev/null
    echo "Terminé."
    exit 0
}

trap cleanup SIGINT SIGTERM

echo "i3 est en cours d'exécution dans Xephyr sur l'affichage \$DISPLAY."
echo "Utilisez \$mod+Shift+e (ou la combinaison définie dans votre config i3) pour quitter i3."
echo "Fermer la fenêtre Xephyr ou faire Ctrl+C dans ce terminal arrêtera également la session."

# Attendre qu'i3 se termine. Ceci est crucial pour que le script ne se termine pas prématurément
# et pour que le 'trap' puisse fonctionner correctement.
# Si i3 se termine (par ex. via \$mod+Shift+e), 'wait' se débloquera.
wait \$I3_PID

# Appeler cleanup au cas où i3 se termine par lui-même ou si wait est interrompu
cleanup
EOL

# Rendre le script exécutable
chmod +x ~/start-i3-xephyr.sh
```

### Script de lancement principal (`launch-i3-wsl.sh`)

Pour simplifier encore plus le lancement, ce script vérifie les dépendances, configure `DISPLAY` pour Xephyr et lance `start-i3-xephyr.sh`.

```bash
cat > ~/launch-i3-wsl.sh << EOL
#!/bin/bash
set -e # Quitter immédiatement si une commande échoue

echo "Vérification des dépendances..."

# S'assurer que Xephyr est installé
if ! command -v Xephyr &> /dev/null; then
    echo "Xephyr n'est pas installé. Tentative d'installation..."
    sudo apt update && sudo apt install -y xserver-xephyr x11-apps
fi

# S'assurer qu'i3 est installé
if ! command -v i3 &> /dev/null; then
    echo "i3 n'est pas installé. Tentative d'installation..."
    sudo apt update && sudo apt install -y i3 i3status i3lock dmenu
fi

# Configurer la variable DISPLAY pour que WSL puisse communiquer avec le serveur X de Windows.
# Ceci est nécessaire pour que Xephyr (qui est un client X) puisse se lancer et s'afficher sur Windows.
# Il utilise /etc/resolv.conf pour trouver l'IP de l'hôte Windows (pour WSL2).
HOST_IP=""
if [ -f /etc/resolv.conf ] && grep -q nameserver /etc/resolv.conf; then
     HOST_IP=\$(awk '/nameserver / {print \$2; exit}' /etc/resolv.conf)
else
    echo "AVERTISSEMENT: Impossible de déterminer automatiquement l'adresse IP de l'hôte Windows depuis /etc/resolv.conf."
    echo "Si Xephyr ne se lance pas, assurez-vous que votre serveur X Windows est accessible et que DISPLAY est bien configuré."
    echo "Vous pouvez essayer de définir DISPLAY manuellement, ex: export DISPLAY=\"YOUR_WINDOWS_IP:0.0\""
fi

if [ -n "\$HOST_IP" ]; then
    export DISPLAY="\${HOST_IP}:0.0"
    echo "Variable DISPLAY pour le serveur X de l'hôte (pour Xephyr) définie sur: \$DISPLAY"
else
    echo "Variable DISPLAY non définie automatiquement. En espérant qu'elle soit déjà correcte."
fi


# Vérifier si la connexion au serveur X Windows est possible (ex: VcXsrv, MobaXterm X server, X410)
# 'xset -q' est une commande simple pour tester la connexion au serveur X.
if ! xset -q &>/dev/null; then
    echo "ERREUR: Impossible de se connecter au serveur X sur Windows (actuellement \$DISPLAY)."
    echo "Veuillez vous assurer que :"
    echo "  1. Un serveur X (comme VcXsrv, MobaXterm, X410, GWSL) est en cours d'exécution sur Windows."
    echo "  2. Ce serveur X est configuré pour accepter les clients (souvent une option comme 'Disable Access Control' ou en écoutant sur toutes les interfaces)."
    echo "  3. Le pare-feu Windows autorise les connexions pour votre serveur X ou sur le port TCP 6000 (par défaut)."
    echo "  4. La variable DISPLAY (\$DISPLAY) est correctement définie pour atteindre ce serveur X."
    exit 1
else
    echo "Connexion au serveur X de Windows (\$DISPLAY) réussie."
fi

echo "Lancement d'i3 dans Xephyr via le script start-i3-xephyr.sh..."
# Passe tous les arguments de launch-i3-wsl.sh à start-i3-xephyr.sh
# Cela permet par exemple de passer une résolution : ~/launch-i3-wsl.sh 1600x900
~/start-i3-xephyr.sh "\$@"

EOL

# Rendre le script exécutable
chmod +x ~/launch-i3-wsl.sh
```

## Utilisation

### Lancer i3 dans Xephyr

Pour démarrer i3 dans Xephyr, suivez ces étapes :

1. Assurez-vous qu'un serveur X est en cours d'exécution sur Windows (ex: VcXsrv, MobaXterm, X410, GWSL) et qu'il est configuré pour accepter les connexions (souvent une option "Disable access control").
2. Ouvrez votre terminal WSL.
3. Exécutez le script de lancement :

    ```bash
    # Pour démarrer i3 dans Xephyr avec la résolution par défaut (1920x1080) :
    ~/launch-i3-wsl.sh

    # Pour démarrer avec une résolution spécifique (par exemple 1280x720) :
    ~/launch-i3-wsl.sh 1280x720
    ```

    Une fenêtre Xephyr devrait s'ouvrir avec i3 fonctionnant à l'intérieur.

### Raccourcis clavier essentiels

Voici les raccourcis clavier de base configurés dans le fichier `~/.config/i3/config` fourni (par défaut, `$mod` est la touche `Alt`):

| Raccourci                      | Action                                             |
| ------------------------------ | -------------------------------------------------- |
| `$mod+Entrée`                  | Ouvrir un terminal                                 |
| `$mod+d`                       | Ouvrir le lanceur d'applications (dmenu)           |
| `$mod+Shift+q`                 | Fermer la fenêtre active (kill)                    |
| `$mod+1` à `$mod+0`            | Changer d'espace de travail                        |
| `$mod+Shift+1` à `$mod+Shift+0`| Déplacer la fenêtre active vers un espace de travail |
| `$mod+h`                       | Diviser horizontalement                            |
| `$mod+v`                       | Diviser verticalement                              |
| `$mod+f`                       | Activer/Désactiver le mode plein écran             |
| `$mod+s`                       | Mode empilé (stacking)                             |
| `$mod+w`                       | Mode à onglets (tabbed)                            |
| `$mod+e`                       | Changer l'organisation (toggle split)              |
| `$mod+Shift+space`             | Activer/Désactiver le mode flottant pour une fenêtre |
| `$mod+r`                       | Entrer en mode redimensionnement                   |
| _En mode redimensionnement:_    |                                                    |
| `j, k, l, ö` ou flèches        | Redimensionner la fenêtre                          |
| `Entrée` ou `Echap` ou `$mod+r`| Quitter le mode redimensionnement                  |
| `$mod+Shift+r`                 | Redémarrer i3 (recharge la configuration)          |
| `$mod+Shift+e`                 | Quitter i3 (avec confirmation)                     |

Adaptez `ö` (odiaeresis) ou `semicolon` aux touches correspondantes sur votre clavier pour `focus right`, `move right`, et `resize grow width`.

### Personnalisation avancée

Pour une personnalisation plus avancée, modifiez le fichier de configuration i3 :

```bash
nano ~/.config/i3/config
```

Quelques personnalisations populaires :

#### Changement de la touche `$mod`

Pour utiliser la touche Windows (`Super_L` ou `Mod4`) au lieu de `Alt` (`Mod1`):

```text
# Décommenter ou changer la ligne suivante dans ~/.config/i3/config
# set $mod Mod4  # Pour la touche Windows
set $mod Mod1  # Pour la touche Alt (par défaut dans notre config)
```

N'oubliez pas de redémarrer i3 (`$mod+Shift+r`) pour appliquer les changements.

#### Ajout des espaces entre les fenêtres (i3-gaps)

Si vous voulez ajouter des "gaps" (espaces) entre les fenêtres, vous pouvez installer `i3-gaps` (une version modifiée de i3) à la place de i3 standard.

1. Désinstallez i3 standard si vous l'aviez compilé ou installé :

    ```bash
    # Si compilé:
    # cd ~/i3-src/build && sudo ninja uninstall
    # Si installé via apt:
    # sudo apt remove i3
    ```

2. Installez les dépendances supplémentaires (la plupart devraient déjà être là) :

    ```bash
    sudo apt install -y libxcb1-dev libxcb-keysyms1-dev libpango1.0-dev \
      libxcb-util0-dev libxcb-icccm4-dev libyajl-dev \
      libstartup-notification0-dev libxcb-randr0-dev \
      libev-dev libxcb-cursor-dev libxcb-xinerama0-dev \
      libxcb-xkb-dev libxkbcommon-dev libxkbcommon-x11-dev \
      autoconf libxcb-xrm0 libxcb-xrm-dev automake libxcb-shape0-dev \
      meson ninja-build # Assurez-vous que meson et ninja sont bien là
    ```

3. Clonez et compilez i3-gaps :

    ```bash
    git clone [https://github.com/Airblader/i3](https://github.com/Airblader/i3) ~/i3-gaps
    cd ~/i3-gaps
    
    # Optionnel: checkout une version spécifique si besoin (ex: pour stabilité)
    # git checkout <tag_ou_commit>

    mkdir -p build && cd build
    meson ..
    ninja
    sudo ninja install
    ```

4. Puis ajoutez à votre configuration i3 (`~/.config/i3/config`):

    ```text
    # Espacement entre les fenêtres (gaps)
    gaps inner 10
    gaps outer 5

    # Vous pouvez aussi avoir des gaps intelligents qui se réduisent s'il n'y a qu'une fenêtre
    # smart_gaps on
    ```

    Redémarrez i3 (`$mod+Shift+r`).

#### Personnalisation de la barre de statut

Pour une barre de statut plus informative, vous pouvez installer et configurer `i3blocks` ou `polybar` à la place de `i3status`.

Exemple avec `i3blocks`:

```bash
sudo apt install -y i3blocks
```

Puis modifiez la configuration de la barre dans `~/.config/i3/config`:

```text
bar {
    # status_command i3status
    status_command i3blocks
    position top
    # ... autres configurations de la barre ...
}
```

Vous devrez ensuite configurer `i3blocks` (généralement via `~/.config/i3blocks/config`).

## Problèmes connus et solutions

### Problème: "Cannot open display"

**Symptômes**: Les applications graphiques (y compris Xephyr ou i3) affichent "Cannot open display", "Authorization required", ou "Xephyr cannot open host display".

**Solutions**:

1. **Serveur X Windows non lancé ou mal configuré**:
    - Assurez-vous qu'un serveur X (VcXsrv, MobaXterm, X410, etc.) est en cours d'exécution sur Windows.
    - Vérifiez que ce serveur X est configuré pour accepter les clients. Pour VcXsrv, cela signifie souvent décocher "Enable access control" ou ajouter l'IP de WSL aux hôtes autorisés. Pour MobaXterm, le serveur X intégré est généralement bien configuré.
2. **Variable `DISPLAY` incorrecte dans WSL pour lancer Xephyr**:
    - Le script `launch-i3-wsl.sh` tente de configurer `DISPLAY` pour pointer vers votre serveur X Windows (ex: `export DISPLAY=$(awk '/nameserver / {print $2; exit}' /etc/resolv.conf):0.0`). Vérifiez que cette variable est correcte avant de lancer `~/launch-i3-wsl.sh`. Vous pouvez tester avec `echo $DISPLAY` puis `xeyes` (si `x11-apps` est installé).
3. **Pare-feu Windows**:
    - Assurez-vous que le pare-feu Windows autorise les connexions entrantes pour votre serveur X, ou spécifiquement pour le port TCP 6000 (ou le port utilisé par votre serveur X). Souvent, lors de la première exécution de VcXsrv, Windows demande une autorisation de pare-feu.
4. **WSL ne peut pas atteindre l'hôte Windows**:
    - Rare, mais vérifiez la connectivité réseau entre WSL2 et l'hôte Windows si `nameserver` dans `/etc/resolv.conf` ne pointe pas vers une IP joignable de l'hôte.

### Problème: "Another window manager seems to be running"

**Symptômes**: i3 affiche "ERROR: Another window manager seems to be running (X error 10)" lorsqu'il essaie de démarrer dans Xephyr.

**Solutions**:

1. **Xephyr n'a pas démarré correctement ou `DISPLAY` pointe mal**:
    - Ceci arrive si i3 est lancé en essayant de gérer l'affichage du serveur X Windows principal au lieu de l'affichage Xephyr.
    - Le script `start-i3-xephyr.sh` est conçu pour lancer Xephyr sur un nouvel affichage (ex: `:1`, `:2`, etc.) et ensuite définir `export DISPLAY` sur ce nouvel affichage _avant_ de lancer `i3`. Vérifiez les messages de ce script.
2. **Configuration i3 incorrecte**: Très rare que cela cause ce message spécifique si Xephyr est utilisé.

### Problème: Interface graphique lente ou saccadée

**Symptômes**: Performances lentes, animations saccadées dans la fenêtre Xephyr.

**Solutions**:

1. **Réduisez la résolution de Xephyr**:
    Lancez avec une résolution plus faible : `~/launch-i3-wsl.sh 1280x720`
2. **Désactivez les effets graphiques** si vous en avez ajouté (ex: compositeur comme `picom`). i3 par défaut est très léger.
3. **Vérifiez la charge CPU/mémoire** sur Windows et dans WSL (`top` ou `htop` dans WSL).
4. **WSL2 Performance**: Assurez-vous que les ressources allouées à WSL2 sont suffisantes. Vous pouvez configurer cela dans le fichier `.wslconfig` sur Windows (voir la section "Amélioration des performances").
5. **Serveur X Windows**: Certains serveurs X sur Windows peuvent être plus performants que d'autres. VcXsrv est souvent recommandé pour sa légèreté.

### Problème: Raccourcis clavier qui ne fonctionnent pas

**Symptômes**: Certains raccourcis i3 (`$mod+touche`) ne répondent pas.

**Solutions**:

1. **Conflits de raccourcis**:
    - Windows lui-même ou d'autres applications Windows pourraient intercepter les raccourcis avant qu'ils n'atteignent Xephyr/i3 (surtout si `$mod` est `Alt`).
    - Le serveur X sur Windows (VcXsrv, etc.) pourrait avoir ses propres raccourcis.
2. **Touche `$mod`**:
    - Essayez de changer la touche `$mod` dans `~/.config/i3/config` pour `Mod4` (touche Windows) si `Mod1` (Alt) pose problème : `set $mod Mod4`. Redémarrez i3 (`$mod+Shift+r`).
3. **Focus de la fenêtre**: Assurez-vous que la fenêtre Xephyr a bien le focus.
4. **Disposition du clavier**: Si les touches comme `;` ou `ö` ne fonctionnent pas comme attendu pour les mouvements/redimensionnements, vérifiez leurs `keysyms` avec l'outil `xev` (lancez `xev` dans un terminal à l'intérieur de la session i3/Xephyr, puis appuyez sur les touches). Adaptez votre configuration i3 en conséquence.

## Amélioration des performances

Pour améliorer les performances de i3 sous WSL avec Xephyr:

1. **Désactivez les effets graphiques inutiles**:
    - Évitez les compositeurs (comme `compton`/`picom`) si la performance est critique, ou configurez-les a minima.
    - Utilisez des fonds d'écran unis ou désactivez-les si vous utilisez un gestionnaire de fond d'écran.
2. **Optimisez WSL2 via `.wslconfig`**:
    Créez ou modifiez le fichier `.wslconfig` dans votre dossier utilisateur Windows (`C:\Users\<VotreUtilisateur>\.wslconfig`).
    Exemple de configuration :

    ```text
    [wsl2]
    memory=4GB        # Limite la mémoire à 4GB (adaptez selon vos besoins)
    processors=2      # Limite le nombre de processeurs virtuels à 2 (adaptez)
    # swap=0            # Optionnel: désactiver le swap WSL si vous avez assez de RAM
    # localhostforwarding=true
    ```

    Après avoir modifié `.wslconfig`, vous devez redémarrer WSL : ouvrez PowerShell et exécutez `wsl --shutdown`, puis relancez votre distribution WSL.
3. **Réduisez le polling (si applicable)**:
    Certaines configurations de barre d'état ou widgets peuvent rafraîchir très fréquemment. Si vous utilisez `i3status` ou `i3blocks`, assurez-vous que les intervalles de rafraîchissement ne sont pas trop courts pour les modules qui consomment des ressources.

## Alternatives

Si cette approche ne fonctionne pas bien pour vous, voici quelques alternatives:

1. **VcXsrv/autre serveur X direct avec i3**: Utilisez un serveur X Windows en mode plein écran ou fenêtré, et lancez i3 directement pour qu'il gère cet affichage.

    ```bash
    # Assurez-vous que DISPLAY est configuré pour pointer vers VcXsrv/autre
    export DISPLAY=$(awk '/nameserver / {print $2; exit}' /etc/resolv.conf):0.0
    # Lancez i3 (assurez-vous qu'aucun autre gestionnaire de fenêtres n'est actif sur cet X server)
    i3
    ```

    Cela peut être plus simple mais moins isolé que Xephyr.
2. **WSLg (pour Windows 11)**: Windows 11 inclut WSLg, qui permet d'exécuter des applications graphiques Linux nativement sans serveur X tiers. Vous pourriez potentiellement lancer i3 via WSLg, bien que ce ne soit pas son cas d'usage principal et pourrait nécessiter des configurations avancées.
3. **Autres dépôts GitHub pour i3 sur WSL**: Explorez des projets comme `i3-on-wsl` qui peuvent offrir des scripts ou configurations différents.
4. **Machine virtuelle Linux complète**: Pour une expérience Linux de bureau plus traditionnelle et potentiellement plus stable pour les interfaces graphiques complexes, une VM dédiée (VirtualBox, VMware) reste une option robuste.

## Ressources supplémentaires

- [Documentation officielle i3](https://i3wm.org/docs/)
- [Guide utilisateur i3](https://i3wm.org/docs/userguide.html)
- [Xephyr - ArchWiki](https://wiki.archlinux.org/title/Xephyr) (contient des informations utiles même si c'est pour Arch)
- [WSL - Documentation Microsoft](https://learn.microsoft.com/fr-fr/windows/wsl/)

## Contribution

Les contributions à ce projet sont les bienvenues ! N'hésitez pas à :

1. Forker ce dépôt.
2. Créer une branche pour vos modifications (`git checkout -b feature/amelioration`).
3. Committer vos changements (`git commit -am 'Ajout de la fonctionnalité X'`).
4. Pousser vers la branche (`git push origin feature/amelioration`).
5. Créer une Pull Request.
    Si vous rencontrez des problèmes ou avez des suggestions, n'hésitez pas à ouvrir une "Issue" sur GitHub.

## Licence

Ce projet est sous licence MIT. Voir le fichier `LICENSE` pour plus de détails.

---

Créé par valorisa pour la communauté i3 et WSL. Faites bon usage de votre nouvel environnement de travail !
