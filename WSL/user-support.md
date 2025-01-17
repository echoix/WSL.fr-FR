---
title: Autorisations et compte d’utilisateur Linux
description: Informations de référence sur les comptes d’utilisateur et la gestion des autorisations avec le sous-système Windows pour Linux.
keywords: BashOnWindows, bash, wsl, windows, sous-système windows pour linux, sous-système windows, comptes d’utilisateur
ms.date: 09/11/2017
ms.topic: article
ms.assetid: f70e685f-24c6-4908-9546-bf4f0291d8fd
ms.custom: seodec18
ms.localizationpriority: high
ms.openlocfilehash: d8434283e459ae25637fac0c0b1877ca07d9a255
ms.sourcegitcommit: 0b5a9f8982dfff07fc8df32d74d97293654f8e12
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2019
ms.locfileid: "71269710"
---
# <a name="user-accounts-and-permissions-for-windows-subsystem-for-linux"></a>Comptes d’utilisateur et autorisations pour le sous-système Windows pour Linux

La création de votre utilisateur Linux constitue la première étape de la configuration d’une nouvelle distribution Linux sur WSL.  Le premier compte d’utilisateur que vous créez est automatiquement configuré avec quelques attributs spéciaux :

1. Il s’agit de votre utilisateur par défaut ; il se connecte automatiquement au lancement.
1. Il s’agit de l’administrateur Linux (membre du groupe sudo) par défaut.

Chaque distribution Linux exécutée sur le sous-système Windows pour Linux a ses propres comptes d’utilisateur et mots de passe Linux.  Vous devez configurer un compte d’utilisateur Linux chaque fois que vous ajoutez, réinstallez ou réinitialisez une distribution.  Les comptes d’utilisateur Linux ne sont pas uniquement indépendants par distribution. Ils sont également indépendants de votre compte d’utilisateur Windows.

## <a name="resetting-your-linux-password"></a>Réinitialisation de votre mot de passe Linux

Si vous avez accès à votre compte d’utilisateur Linux et que vous connaissez votre mot de passe actuel, changez-le à l’aide de l’outil de réinitialisation de mot de passe Linux de cette distribution, très probablement `passwd`.

Si ce n’est pas le cas, en fonction de la distribution, vous pouvez peut-être réinitialiser votre mot de passe en réinitialisant l’utilisateur par défaut.

WSL offre une étiquette utilisateur par défaut pour identifier le compte d’utilisateur qui se connecte automatiquement quand vous démarrez un sous-système WSL.  Étant donné que de nombreuses distributions incluent des commandes permettant de définir l’utilisateur par défaut sur root et un utilisateur racine sans mot de passe défini, la définition de l’utilisateur par défaut sur root s’avère pratique pour effectuer des opérations telles que la réinitialisation de mot de passe.

### <a name="for-creators-update-and-earlier"></a>Pour Creators Update et versions antérieures
Si vous exécutez Windows 10 Creators Update ou version antérieure, vous pouvez changer l’utilisateur Bash par défaut en exécutant les commandes suivantes :

1. Définissez l’utilisateur par défaut sur `root` :

    ```console
    C:\> lxrun /setdefaultuser root
    ```

1. Exécutez `bash.exe` pour vous connecter maintenant en tant que `root` :

    ```console
    C:\> bash.exe
    ```

1. Réinitialisez votre mot de passe à l’aide de la commande de mot de passe de la distribution, puis fermez la console Linux :

    ```BASH
    $ passwd username
    $ exit
    ```

1. À partir de Windows CMD, réinitialisez l’utilisateur par défaut sur votre compte d’utilisateur Linux normal :

    ```console
    C:\> lxrun.exe /setdefaultuser username
    ```

### <a name="for-fall-creators-update-and-later"></a>Pour Fall Creators Update et versions ultérieures
Pour connaître les commandes disponibles pour une distribution en particulier, exécutez `[distro.exe] /?`.
    
Par exemple, avec Ubuntu installé :

```console
C:\> ubuntu.exe /?

Launches or configures a linux distribution.

Usage:
    <no args>
      - Launches the distro's default behavior. By default, this launches your default shell.

    run <command line>
      - Run the given command line in that distro, using the default configuration.
      - Everything after `run ` is passed to the linux LaunchProcess cal

    config [setting [value]]
      - Configure certain settings for this distro.
      - Settings are any of the following (by default)
        - `--default-user <username>`: Set the default user for this distro to <username>

    clean
      - Uninstalls the distro. The appx remains on your machine. This can be
        useful for "factory resetting" your instance. This removes the linux
        filesystem from the disk, but not the app from your PC, so you don't
        need to redownload the entire tar.gz again.

    help
      - Print this usage message.
```

Instructions pas à pas sur Ubuntu :

1. Ouvrez CMD.
1. Définissez l’utilisateur Linux par défaut sur `root` :

    ```console
    C:\> ubuntu config --default-user root
    ```    

1. Lancez votre distribution Linux (`ubuntu`).  Vous vous connectez automatiquement en tant que `root` :

1. Réinitialisez votre mot de passe à l’aide de la commande `passwd` :

    ```BASH
    $ passwd username
    ```

1. À partir de Windows CMD, réinitialisez l’utilisateur par défaut sur votre compte d’utilisateur Linux normal.

    ```console
    C:\> ubuntu config --default-user username
    ```

## <a name="permissions"></a>Permissions

Il existe deux concepts importants à garder à l’esprit concernant les autorisations dans WSL :

1. Le modèle d’autorisation Windows régit les droits d’un processus sur les ressources Windows
2. Le modèle d’autorisation Linux contrôle les droits d’un processus sur les ressources Linux

Lors de l’exécution de Linux sur WSL, Linux a les mêmes autorisations Windows que le processus qui le lance. Linux peut être lancé dans l’un des deux niveaux d’autorisation suivants :

* Autorisation normale (non élevée) : Linux s’exécute avec les autorisations de l’utilisateur connecté
* Autorisation élevée/administrateur : Linux s’exécute avec des autorisations Windows élevées/administrateur

> Étant donné que les processus élevés peuvent accéder aux paramètres système et aux données système/protégées et les modifier (et donc les endommager), **évitez** de lancer des processus élevés, sauf si vous avez absolument besoin de le faire, qu’il s’agisse d’applications, d’outils ou de shells Windows ou Linux !

Les autorisations Windows ci-dessus sont indépendantes des autorisations au sein d’une instance Linux : Les « privilèges racine » Linux impactent uniquement les droits de l’utilisateur au sein de l’environnement et du système de fichiers Linux ; ils n’ont aucun impact sur les privilèges Windows accordés. Ainsi, l’exécution d’un processus Linux en tant que racine (par exemple, via `sudo`) accorde uniquement à ce processus des droits d’administrateur au sein de l’environnement Linux.

**Exemple :**     
Une session Bash avec des privilèges d’administrateur Windows peut accéder à `cd /mnt/c/Users/Administrator` alors qu’une session Bash sans privilèges d’administrateur obtient une erreur « autorisation refusée ».

Dans Linux, le fait de taper `sudo cd /mnt/c/Users/Administrator` n’accorde pas l’accès au répertoire de l’administrateur, car les autorisations dans Windows sont gérées par Windows.

Le modèle d’autorisation Linux est important à l’intérieur de l’environnement Linux où l’utilisateur dispose d’autorisations basées sur l’utilisateur Linux actuel.

**Exemple :**  
Un utilisateur du groupe sudo peut exécuter `sudo apt update`.
