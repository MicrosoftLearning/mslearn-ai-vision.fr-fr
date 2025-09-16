---
lab:
  title: Classifier des images avec un modèle personnalisé Azure AI Vision
  module: Module 2 - Develop computer vision solutions with Azure AI Vision
---

# Classifier des images avec un modèle personnalisé Azure AI Vision

Azure AI Vision vous permet d’effectuer l’apprentissage des modèles personnalisés pour classifier et détecter des objets assortis des étiquettes que vous spécifiez. Dans ce labo, nous allons créer un modèle de classification d’images personnalisé pour classifier des images de fruits.

## Provisionner une ressource Vision par ordinateur

Vous devrez approvisionner une ressource **Vision par ordinateur** si vous n’en avez pas déjà une dans votre abonnement.

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
1. Sélectionnez **Créer une ressource**.
1. Dans la barre de recherche, recherchez *Vision par ordinateur*, sélectionnez **Vision par ordinateur**, et créez la ressource avec les paramètres suivants :
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *Choisissez ou créez un groupe de ressources. (Si vous utilisez un abonnement restreint, vous n’avez peut-être pas l’autorisation de créer un groupe de ressources. Dans ce cas, utilisez le groupe fourni.)*
    - **Région** : *choisissez parmi USA Est, USA Ouest, France Centre, Corée Centre, Europe Nord, Asie Sud-Est, Europe Ouest et Asie Est\**
    - **Nom** : *Entrez un nom unique.*
    - **Niveau tarifaire** : F0 gratuit

    \*Les fonctionnalités complètes d’Azure AI Vision 4.0 sont actuellement disponibles uniquement dans ces régions.

1. Cochez les cases nécessaires et créez la ressource.
<!--4. When the resource has been deployed, go to it and view its **Keys and Endpoint** page. You will need the endpoint and one of the keys from this page in a future step. Save them off or leave this browser tab open.-->

Nous avons également besoin d’un compte de stockage pour stocker les images d’entraînement.

1. Dans le Portail Azure, recherchez et sélectionnez **Comptes de stockage**, puis créez un compte de stockage avec les paramètres suivants :
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *choisissez le même groupe de ressources que celui dans lequel vous avez créé votre ressource Custom Vision*
    - **Nom du compte de stockage** : customclassifySUFFIX 
        - *Remarque : remplacez le jeton `SUFFIX` par vos initiales ou une autre valeur afin de garantir que le nom de la ressource soit globalement unique.*
    - **Région** : *choisissez la même région que celle utilisée pour votre ressource Azure AI Services*
    - **Service principal** : stockage Blob Azure ou Azure Data Lake Storage Gen2
    - **Charge de travail principale** : autre
    - **Performances** : standard
    - **Redondance** : stockage localement redondant (LRS)

1. Une fois la ressource déployée, sélectionnez **Aller à la ressource**.
1. Activez l’accès public sur le compte de stockage. Dans le volet gauche, accédez à **Configuration** dans le groupe **Paramètres** et activez *Autoriser l’accès anonyme aux objets blob*. Cliquez sur **Enregistrer**
1. Dans le volet gauche, dans **Stockage de données**, sélectionnez **Conteneurs**, créez un conteneur nommé `fruit`, puis définissez **Niveau d’accès anonyme** sur *Conteneur (accès en lecture anonyme pour les conteneurs et les objets blob)*.

    > **Remarque** : si le **niveau d’accès anonyme** est désactivé, actualisez la page du navigateur.
   
## Cloner le référentiel pour ce cours

Les fichiers image pour l'entraînement de votre modèle ont été fournis dans un référentiel GitHub. Vous allez cloner le référentiel et télécharger les images vers votre compte de stockage en utilisant Cloud Shell depuis le Portail Azure. 

> **Conseil** : si vous avez récemment cloné le référentiel **mslearn-ai-vision**, vous pouvez ignorer la tâche de clonage. Autrement, procédez comme indiqué pour cloner le référentiel vers votre environnement de développement.

1. Dans le portail Azure, cliquez sur le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un environnement Cloud Shell dans le portail Azure, puis sélectionnez un environnement ***PowerShell***. Cloud Shell fournit une interface de ligne de commande dans un volet situé en bas du portail Azure.

    > **Remarque** : si vous avez déjà créé un Cloud Shell qui utilise un environnement *Bash*, basculez-le vers ***PowerShell***.

1. Dans la barre d’outils Cloud Shell, dans le menu **Paramètres**, sélectionnez **Accéder à la version classique** (cela est nécessaire pour utiliser l’éditeur de code).

    > **Conseil** : lorsque vous collez des commandes dans cloudshell, la sortie peut prendre une grande quantité de mémoire tampon d’écran. Vous pouvez effacer le contenu de l’écran en saisissant la commande `cls` pour faciliter le focus sur chaque tâche.

1. Dans le volet PowerShell, entrez les commandes suivantes pour cloner le référentiel GitHub pour cet exercice :

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/microsoftlearning/mslearn-ai-vision mslearn-ai-vision
    ```

1. Une fois le référentiel cloné, naviguez jusqu’au dossier contenant les fichiers de l’exercice :  

    ```
   cd mslearn-ai-vision/Labfiles/02-image-classification
    ```

1. Exécutez la commande `code replace.ps1` et examinez le code. Vous verrez qu’il remplace le nom de votre compte de stockage pour l’espace réservé dans un fichier JSON (le fichier COCO) que nous utiliserons dans une étape ultérieure.
1. Remplacez l’espace réservé dans *la première ligne du fichier uniquement* par le nom de votre compte de stockage.
1. Après avoir remplacé l’espace réservé, dans l’éditeur de code, utilisez la commande **CTRL+S** ou **Faire un clic droit sur > Enregistrer** pour sauvegarder vos modifications, puis utilisez la commande **CTRL+Q** ou **Faire un clic droit sur > Quitter** pour fermer l’éditeur de code tout en laissant la ligne de commande cloud shell ouverte.
1. Exécutez le script avec la commande suivante :

    ```powershell
    ./replace.ps1
    ```

1. Vous pouvez examnier le fichier COCO pour vous assurer qu'il contient bien le nom de votre compte de stockage. Exécutez `code training-images/training_labels.json` et affichez les premières entrées. Dans le champ *absolute_url*, vous devriez voir quelque chose ressemblant à *"https://myStorage.blob.core.windows.net/fruit/.. .*. Si vous ne voyez pas la modification attendue, vérifiez que vous avez mis à jour uniquement le premier espace réservé dans le script PowerShell.
1. Fermez l’éditeur de code.
1. Exécutez la commande suivante, en remplaçant `<your-storage-account>` par le nom de votre compte de stockage, pour charger le contenu du dossier **training-images** vers le conteneur `fruit` que vous avez créé précédemment.

    ```powershell
    az storage blob upload-batch --account-name <your-storage-account> -d fruit -s ./training-images/
    ```

1. Ouvrez le conteneur `fruit` et vérifiez que les fichiers ont bien été chargés.

## Créer un projet d'apprentissage d'un modèle personnalisé

Ensuite, vous allez créer un projet d’apprentissage pour la classification personnalisée d’images dans Vision Studio.

1. Dans le navigateur web, accédez à `https://portal.vision.cognitive.azure.com/` et connectez-vous au compte Microsoft où vous avez créé votre ressource Azure AI.
1. Sélectionnez la vignette **Personnaliser les modèles avec des images** (qui se trouve dans l’onglet **Analyse d’image** si elle ne s’affiche pas dans votre vue par défaut).
1. Sélectionnez le compte Azure AI Services que vous avez créé.
1. Dans votre projet, sélectionnez **Ajouter un nouveau jeu de données** en haut. Configurez  en utilisant les paramètres suivants :
    - **Nom du jeu de données** : training_images
    - **Type de modèle** : classification d’images
    - **Sélectionner un conteneur de stockage Blob Azure** : sélectionnez **Sélectionner un conteneur**
        - **Abonnement** : *votre abonnement Azure*
        - **Compte de stockage** : * le compte de stockage que vous avez créé*
        - **Conteneur d’objets blob** : fruit
    - Cochez la case pour « autoriser Vision Studio à lire et écrire sur votre stockage Blob »
1. Sélectionnez le jeu de données **training_images**.

À ce stade de la création du projet, vous devez généralement sélectionner **Créer un projet d’étiquetage de données Azure ML** et étiqueter vos images, ce qui génère un fichier COCO. Vous êtes invité à essayer cela si vous avez du temps, mais pour les besoins de ce labo, nous avons déjà étiqueté les images et généré le fichier COCO correspondant.

1. Sélectionnez **Ajouter un fichier COCO**
1. Dans la liste déroulante, sélectionnez **Importer un fichier COCO à partir d’un conteneur d’objets blob**
1. Étant donné que vous avez déjà connecté votre conteneur nommé `fruit`, Vision Studio recherche cela pour un fichier COCO. Dans la liste déroulante, sélectionnez **training_labels.json**, puis ajoutez le fichier COCO.
1. Accédez aux **Modèles personnalisés** à gauche, puis sélectionnez **Entraîner un nouveau modèle**. Utilisez les paramètres suivants :
    - **Nom du modèle** : classifyfruit
    - **Type de modèle** : classification d’images
    - **Choisir un jeu de données d’entraînement** : training_images
    - Conservez les autres valeurs par défaut, puis sélectionnez **Entraîner le modèle**

L’apprentissage peut prendre un certain temps. Le budget par défaut prévoit jusqu’à une heure, mais pour ce petit jeu de données, cette opération est généralement beaucoup plus rapide. Cliquez sur le bouton **Actualiser** toutes les deux minutes jusqu’à ce que l’état du travail passe à *Réussi*. Sélectionnez le modèle.

Ici, vous pouvez afficher le niveau de performance du travail d'apprentissage. Examinez la précision et l'exactitude du modèle entraîné.

## Testez votre modèle personnalisé

Votre modèle a été entraîné et est prêt à être testé.

1. En haut de la page de votre modèle personnalisé, sélectionnez **Essayer**.
1. Dans la liste déroulante spécifiant le modèle que vous souhaitez utiliser, sélectionnez le modèle **classifyfruit**, puis accédez au dossier **02-image-classification\test-images**.
1. Sélectionnez chaque image et affichez les résultats. Dans la zone de résultats, sélectionnez l’onglet **JSON** pour examiner la réponse JSON complète.

<!-- Option coding example to run-->
## Nettoyer les ressources

Si vous n’utilisez pas les ressources Azure créées dans ce labo pour d’autres modules de formation, vous pouvez les supprimer pour éviter d’autres frais.

1. Ouvrez le Portail Azure à l'adresse `https://portal.azure.com` et, dans la barre de recherche supérieure, recherchez les ressources que vous avez créées dans ce labo.

2. Dans la page de ressources, sélectionnez **Supprimer** et suivez les instructions pour supprimer la ressource. Vous pouvez également supprimer l’intégralité du groupe de ressources pour nettoyer toutes les ressources en même temps.
