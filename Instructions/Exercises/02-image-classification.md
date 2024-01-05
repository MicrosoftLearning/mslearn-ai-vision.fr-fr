---
lab:
  title: "Classer des images avec un modèle personnalisé Azure\_AI\_Vision"
  module: Module 2 - Develop computer vision solutions with Azure AI Vision
---

# Classer des images avec un modèle personnalisé Azure AI Vision

Azure AI Vision vous permet d’effectuer l’apprentissage des modèles personnalisés pour classifier et détecter des objets assortis des étiquettes que vous spécifiez. Dans ce labo, nous allons créer un modèle de classification d’images personnalisé pour classifier des images de fruits.

## Cloner le référentiel pour cette formation

Si vous n’avez pas déjà cloné le référentiel de code **Azure AI Vision** dans l’environnement où vous travaillez sur ce labo, procédez comme suit. Sinon, ouvrez le dossier cloné dans Visual Studio Code.

1. Démarrez Visual Studio Code.
2. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git : Cloner** pour cloner le référentiel `https://github.com/MicrosoftLearning/mslearn-ai-vision` vers un dossier local (peu importe quel dossier).
3. Lorsque le référentiel a été cloné, ouvrez le dossier dans Visual Studio Code.
4. Attendez que des fichiers supplémentaires soient installés pour prendre en charge les projets de code C# dans le référentiel.

    > **Remarque** : Si vous êtes invité à ajouter des ressources requises pour générer et déboguer, sélectionnez **Not Now** (Pas maintenant). Si le message *Projet de fonction Azure détecté dans le dossier* s’affiche, vous pouvez le fermer en toute sécurité.

## Approvisionner des ressources Azure

Si vous n’en avez pas encore dans votre abonnement, vous devez provisionner une ressource **Azure AI Services**.

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
2. Dans la barre de recherche supérieure, recherchez *Azure AI Services*, sélectionnez **Azure AI Services** et créez une ressource de compte multiservices Azure AI Services avec les paramètres suivants :
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *Choisissez ou créez un groupe de ressources. (Si vous utilisez un abonnement restreint, vous n’avez peut-être pas l’autorisation de créer un groupe de ressources. Dans ce cas, utilisez le groupe fourni.)*
    - **Région** : *choisissez parmi USA Est, Europe Ouest, USA Ouest 2\**
    - **Nom** : *Entrez un nom unique.*
    - **Niveau tarifaire** : Standard S0

    \*Les étiquettes de modèle personnalisé Azure AI Vision 4.0 sont actuellement disponibles uniquement dans ces régions.

3. Cochez les cases nécessaires et créez la ressource.
<!--4. When the resource has been deployed, go to it and view its **Keys and Endpoint** page. You will need the endpoint and one of the keys from this page in a future step. Save them off or leave this browser tab open.-->

Nous avons également besoin d’un compte de stockage pour stocker les images d’entraînement.

1. Dans le Portail Azure, recherchez et sélectionnez **Comptes de stockage**, puis créez un compte de stockage avec les paramètres suivants :
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *choisissez le même groupe de ressources que celui dans lequel vous avez créé votre ressource Azure AI Services*
    - **Nom du compte de stockage** : customclassifySUFFIX 
        - *remarque : remplacez le jeton `SUFFIX` par vos initiales ou une autre valeur pour vous assurer que le nom de la ressource est globalement unique.*
    - **Région** : *choisissez la même région que celle utilisée pour votre ressource Azure AI Services*
    - **Performances** : standard
    - **Redondance** : stockage localement redondant (LRS)
1. Pendant la création de votre compte de stockage, accédez à Visual Studio Code et développez le dossier **Labfiles/02-image-classification**.
1. Dans ce dossier, sélectionnez **replace.ps1** et passez en revue le code. Vous verrez qu’il remplace le nom de votre compte de stockage pour l’espace réservé dans un fichier JSON (le fichier COCO) que nous utiliserons dans une étape ultérieure. Remplacez l’espace réservé dans *la première ligne du fichier uniquement* par le nom de votre compte de stockage. Enregistrez le fichier.
1. Cliquez avec le bouton droit sur le dossier **02-image-classification** et ouvrez un terminal intégré. Exécutez la commande suivante :

    ```powershell
    ./replace.ps1
    ```

1. Vous pouvez examnier le fichier COCO pour vous assurer qu'il contient bien le nom de votre compte de stockage. Sélectionnez **training-images/training_labels.json** et affichez les premières entrées. Dans le champ *absolute_url*, vous devriez voir quelque chose ressemblant à *"https://myStorage.blob.core.windows.net/fruit/.. .*. Si vous ne voyez pas la modification attendue, vérifiez que vous avez mis à jour uniquement le premier espace réservé dans le script PowerShell.
1. Fermez les fichiers JSON et PowerShell, puis revenez à la fenêtre du navigateur.
1. Votre compte de stockage devrait être terminé. Accédez à votre compte de stockage.
1. Activez l’accès public sur le compte de stockage. Dans le volet gauche, accédez à **Configuration** dans le groupe **Paramètres** et activez *Autoriser l’accès anonyme aux objets blob*. Cliquez sur **Enregistrer**
1. Dans le volet gauche, sélectionnez **Conteneurs**, créez un conteneur nommé `fruit`, puis définissez **Niveau d’accès anonyme ** sur *Conteneur (accès en lecture anonyme pour les conteneurs et les objets blob)*.

    > **Remarque** : si le **niveau d’accès anonyme** est désactivé, actualisez la page du navigateur.

1. Accédez à `fruit`, puis chargez les images (et le fichier JSON) dans **Labfiles/02-image-classification/training-images** dans ce conteneur.

## Créer un projet d'apprentissage d'un modèle personnalisé

Ensuite, vous allez créer un projet d’apprentissage pour la classification personnalisée d’images dans Vision Studio.

1. Dans le navigateur web, accédez à `https://portal.vision.cognitive.azure.com/` et connectez-vous au compte Microsoft où vous avez créé votre ressource Azure AI.
1. Sélectionnez la vignette **Personnaliser les modèles avec des images** (qui se trouve dans l’onglet **Analyse d’image** si elle ne s’affiche pas dans votre vue par défaut) et, si vous y êtes invité, sélectionnez la ressource Azure AI que vous avez créée.
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