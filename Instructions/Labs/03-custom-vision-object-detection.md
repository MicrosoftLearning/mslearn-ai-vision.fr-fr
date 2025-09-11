---
lab:
  title: "Détecter des objets dans des images avec Azure\_AI\_Custom\_Vision"
---

# Détecter des objets dans des images avec Azure AI Custom Vision

Dans cet exercice, vous allez utiliser le service Custom Vision pour effectuer l’apprentissage d’un modèle de *détection d’objet* permettant de détecter et localiser trois classes de fruits (pomme, banane et orange) dans une image.

## Créer des ressources Custom Vision

Si vous disposez déjà de ressources de formation et de prédiction **Custom Vision** dans votre abonnement Azure, vous pouvez les utiliser ou utiliser un compte multiservices existant dans cet exercice. Sinon, utilisez les instructions suivantes pour les créer.

> **Remarque** : si vous utilisez un compte multiservices, la clé et le point de terminaison pour la formation et pour la prédiction sont identiques.

1. Dans un nouvel onglet de navigateur, ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
1. Sélectionnez le bouton **&#65291;Créer une ressource**, recherchez *Custom Vision*, puis créez une ressource **Custom Vision** avec les paramètres suivants :
    - **Options de création** : Les deux
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *Choisissez ou créez un groupe de ressources. (Si vous utilisez un abonnement restreint, vous n’avez peut-être pas l’autorisation de créer un groupe de ressources. Dans ce cas, utilisez le groupe fourni.)*
    - **Région** : *choisissez n’importe quelle région disponible*
    - **Nom** : *Entrez un nom unique.*
    - **Niveau tarifaire de formation** : F0
    - **Niveau tarifaire de prédiction** : F0

    > **Remarque** : Si vous avez déjà un service de vision personnalisée F0 dans votre abonnement, sélectionnez **S0** pour celui-ci.

1. Attendez que les ressources soient créées, puis affichez les détails du déploiement. Vous noterez que deux ressources Custom Vision sont approvisionnées : une pour la formation et une autre pour la prédiction (que vous pouvez facilement repérer par le suffixe **-Prediction**). Vous pouvez les visualiser en naviguant vers le groupe de ressources dans lequel vous les avez créées.

> **Important** : Chaque ressource a son propre *point de terminaison* et *ses propres clés*, qui sont utilisées pour gérer l’accès à partir de votre code. Pour effectuer l’apprentissage d’un modèle de classification d’images, votre code doit utiliser la ressource de *formation* (avec son point de terminaison et sa clé) ; si vous souhaitez utiliser le modèle formé pour prédire les classes d’images, votre code doit utiliser la ressource de *prédiction* (avec son point de terminaison et sa clé).

## Cloner le référentiel pour ce cours

Vous allez développer votre code à l’aide de Cloud Shell à partir du portail Azure. Les fichiers de code de votre application ont été fournis dans un référentiel GitHub.

> **Conseil** : si vous avez récemment cloné le référentiel **mslearn-ai-vision**, vous pouvez ignorer cette tâche. Dans le cas contraire, procédez comme suit pour le cloner dans votre environnement de développement.

1. Dans le portail Azure, cliquez sur le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un environnement Cloud Shell dans le portail Azure, puis sélectionnez un environnement ***PowerShell***. Cloud Shell fournit une interface de ligne de commande dans un volet situé en bas du portail Azure.

    > **Remarque** : si vous avez déjà créé un Cloud Shell qui utilise un environnement *Bash*, basculez-le vers ***PowerShell***.

1. Dans la barre d’outils Cloud Shell, dans le menu **Paramètres**, sélectionnez **Accéder à la version classique** (cela est nécessaire pour utiliser l’éditeur de code).

    > **Conseil** : lorsque vous collez des commandes dans cloudshell, la sortie peut prendre une grande quantité de mémoire tampon d’écran. Vous pouvez effacer le contenu de l’écran en saisissant la commande `cls` pour faciliter le focus sur chaque tâche.

1. Dans le volet PowerShell, entrez les commandes suivantes pour cloner le référentiel GitHub pour cet exercice :

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/microsoftlearning/mslearn-ai-vision mslearn-ai-vision
    ```

1. Une fois le référentiel cloné, accédez au dossier contenant les fichiers de code de l’application :  

    ```
   cd mslearn-ai-vision/Labfiles/03-object-detection
    ```

## Créer un projet Custom Vision

Pour former un modèle de détection d’objets, vous devez créer un projet Custom Vision basé sur votre ressource de formation. Pour ce faire, vous allez utiliser le portail Custom Vision.

1. Dans un nouvel onglet de navigateur, ouvrez le portail Custom Vision à l’adresse `https://customvision.ai` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
1. Créez un nouveau projet avec les paramètres suivants :
    - **Nom** : Détecter les fruits
    - **Description** : Détection d’objet pour les fruits.
    - **Ressource**: *La ressource Custom Vision que vous avez créée précédemment*
    - **Types de projets** : Détection d’objets
    - **Domaines** : Général
1. Attendez que le projet soit créé et ouvert dans le navigateur.

## Ajouter et baliser des images

Pour former un modèle de détection d’objets, vous devez télécharger des images qui contiennent les classes que vous souhaitez que le modèle identifie, et les marquer pour qu’elles indiquent des zones englobantes pour chaque instance d’objet.

1. Téléchargez les images d’entraînement depuis `https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/main/Labfiles/03-object-detection/training-images.zip` et extrayez le dossier compressé pour afficher son contenu. Ce dossier contient des images de fruits.
1. Dans le portail Vision personnalisée, dans votre projet de détection d'objets, sélectionnez **Ajouter des images** et téléchargez toutes les images du dossier extrait.
1. Une fois les images téléchargées, sélectionnez la première pour l’ouvrir.
1. Maintenez la souris sur un objet dans l’image jusqu’à ce qu’une zone détectée automatiquement soit affichée comme l’image ci-dessous. Sélectionnez ensuite l’objet et, si nécessaire, redimensionnez la région pour l’entourer.

    ![Région par défaut d’un objet](../media/object-region.jpg)

    Vous pouvez aussi simplement faire glisser le curseur autour de l’objet pour créer une région.

1. Lorsque la région entoure l’objet, ajoutez une nouvelle balise avec le type d’objet approprié (*apple*, *banana* ou *orange*) comme indiqué ici :

    ![Objet balisé dans une image](../media/object-tag.jpg)

1. Sélectionnez et marquez chaque autre objet dans l’image, en redimensionnant les régions et en ajoutant de nouvelles balises selon les besoins.

    ![Deux objets avec des balises dans une image](../media/object-tags.jpg)

1. Utilisez le lien **>** situé à droite pour accéder à l’image suivante et baliser ses objets. Il vous suffit ensuite de continuer à travailler sur la totalité de la collection d’images, en marquant chaque pomme, banane et orange.

1. Une fois que vous avez terminé de marquer la dernière image, fermez l’éditeur **Détails de l’image**. Dans la page **Images de formation**, sous **Balises**, sélectionnez **Avec balise** pour afficher toutes vos images étiquetées :

![Images avec balise dans un projet](../media/tagged-images.jpg)

## Utiliser l’API de formation pour charger des images

Vous pouvez utiliser l’interface utilisateur du portail Custom Vision pour étiqueter vos images, mais de nombreuses équipes de développement IA utilisent d’autres outils qui génèrent des fichiers contenant des informations sur les balises et les régions d’objets dans les images. Dans des scénarios comme celui-ci, vous pouvez utiliser l’API de formation Custom Vision pour charger des images étiquetées dans le projet.

> **Remarque** : Dans cet exercice, vous pouvez choisir d’utiliser l’API à partir du SDK **C#** ou **Python**. Dans les étapes qui suivent, effectuez les actions appropriées pour votre langage préféré.

1. Cliquez sur l’icône *paramètres* (&#9881;) en haut à droite de la page **Images de formation** dans le portail Custom Vision pour afficher les paramètres du projet.
1. Sous **Général** (à gauche), notez l’**ID de projet** qui identifie ce projet de façon unique.
1. La clé et le point de terminaison s’affichent à droite, sous la section **Ressources**. Vous y trouverez les détails de la ressource de *formation* (vous pouvez également obtenir ces informations en consultant la ressource dans le Portail Azure).
1. De retour dans le Portail Azure, exécutez la commande `cd C-Sharp/train-detector` ou `cd Python/train-detector` selon votre langue choisie.
1. Installez le package de formation Custom Vision en exécutant la commande appropriée pour votre préférence de langage :

    **C#**

    ```
   dotnet add package Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training --version 2.0.0
    ```

    **Python**

    ```
   pip install azure-cognitiveservices-vision-customvision==3.1.1
    ```

1. Grâce à la commande `ls`, vous pouvez consulter le contenu du dossier **train-detector**. Notez qu’il contient un fichier pour les paramètres de configuration :

    - **C#** : appsettings.json
    - **Python** : .env

1. Saisissez la commande suivante pour modifier le fichier de configuration fourni :

    **C#**

    ```
   code appsettings.json
    ```

    **Python**

    ```
   code .env
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Dans le fichier de code, mettez à jour les valeurs de configuration qu’il contient pour qu’elles reflètent le **point de terminaison** et une **clé** d’authentification pour votre ressource d’*entraînement* Custom Vision, ainsi que l’ID de projet pour le projet de détection d’objet que vous avez créé précédemment.
1. Une fois que vous avez remplacé les espaces réservés, utilisez la commande **CTRL+S** ou **Clic droit > Enregistrer** dans l’éditeur de code pour enregistrer vos modifications, puis utilisez la commande **CTRL+Q** ou **Clic droit > Quitter** pour fermer l’éditeur tout en gardant la ligne de commande du Cloud Shell ouverte.
1. Exécutez `code tagged-images.json` pour ouvrir le fichier et examiner le JSON qu’il contient. Le fichier JSON définit une liste d’images, chacune contenant une ou plusieurs régions étiquetées. Chaque région balisée comprend un nom de balise, les coordonnées supérieure et gauche, ainsi que les dimensions de largeur et de hauteur du cadre englobant contenant l’objet étiqueté.

    > **Remarque** : Les coordonnées et les dimensions de ce fichier indiquent des points relatifs sur l’image. Par exemple, une valeur de *hauteur* de 0,7 indique une zone représentant 70 % de la hauteur de l’image. Certains outils d’étiquetage génèrent d’autres formats de fichier dans lesquels les valeurs de coordonnées et de dimension représentent des pixels, des pouces ou d’autres unités de mesure.

1. Notez que le dossier **train-detector** contient un sous-dossier dans lequel sont stockés les fichiers image référencés dans le fichier JSON.

1. Notez également que le dossier **train-detector** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : train-detector.py

    Ouvrez le fichier de code et passez en revue le code qu’il contient, en notant les détails suivants :
    - Les espaces de noms du package que vous avez installé sont importés
    - La fonction **Main** récupère les paramètres de configuration et utilise la clé et le point de terminaison pour créer un **CustomVisionTrainingClient** authentifié, qui est ensuite utilisé avec l’ID de projet pour créer une référence **Project** sur votre projet.
    - La fonction **Upload_Images** extrait les informations de région étiquetées du fichier JSON et les utilise pour créer un lot d’images avec des régions, qu’elle charge ensuite dans le projet.

1. Exécutez la commande suivante pour lancer le programme :
    
    **C#**
    
    ```
   dotnet run
    ```
    
    **Python**
    
    ```
   python train-detector.py
    ```
    
1. Attendez la fin de l’exécution du programme. Revenez ensuite à votre navigateur et affichez la page **Images de formation** de votre projet dans le portail Custom Vision (actualisez le navigateur si nécessaire).
1. Vérifiez que certaines nouvelles images étiquetées ont été ajoutées au projet.

## Formation et test d’un modèle

Maintenant que vous avez balisé les images de votre projet, vous pouvez former un modèle.

1. Dans le projet Custom Vision, cliquez sur **Former** pour former un modèle de détection d’objets à l’aide des images balisées. Sélectionnez l’option **Entraînement rapide**.
1. Attendez la fin de la formation (cela peut prendre une dizaine de minutes), puis vérifiez les mesures de performance *Précision*, *Rappel*, et *mAP* ; elles mesurent la précision de prédiction du modèle de classification et doivent toutes être élevées.
1. En haut à droite de la page, cliquez sur **Test rapide**, puis dans la zone **URL de l’image**, entrez `https://aka.ms/apple-orange` et affichez la prédiction générée. Fermez ensuite la fenêtre **Test rapide**.

## Publier le modèle de détection d’objet

Vous êtes maintenant prêt à publier votre modèle formé et à l’utiliser à partir d’une application cliente.

1. Sur le portail Custom Vision, sur la page **Performances**, cliquez sur **&#128504; Publier** pour publier le modèle formé avec les paramètres suivants :
    - **Nom du modèle** : fruit-detector
    - **Ressource de prédiction** : *ressource de **prédiction** que vous avez créée précédemment et qui se termine par « -Prediction » (<u>autre que</u> la ressource de formation)* .
1. En haut à gauche de la page **Paramètres du projet**, cliquez sur l’icône *Galerie de projets* (&#128065;) pour revenir à la page d’accueil du portail Custom Vision, où votre projet est maintenant répertorié.
1. Sur la page d’accueil du portail Custom Vision, en haut à droite, cliquez sur l’icône *Paramètres* (&#9881;) pour afficher les paramètres de votre service Custom Vision. Ensuite, sous **Ressources**, recherchez votre ressource de *prédiction* qui se termine par « -Prediction » (<u>et non</u> la ressource de formation) pour indiquer ses valeurs de **Clé** et de **Point de terminaison** (vous pouvez également obtenir ces informations en affichant la ressource dans le Portail Azure).

## Utiliser le classifieur d’images à partir d’une application cliente

Maintenant que vous avez publié le modèle de classification d’images, vous pouvez l’utiliser à partir d’une application cliente. Là encore, vous avez le choix entre les langages **C#** et **Python**.

1. Naviguez jusqu’au dossier **test-detector** dans le Portail Azure grâce à la commande `cd ../test-detector`.
1. Saisissez la commande spécifique au kit de développement logiciel (SDK) ci-dessous pour installer le package prédiction Custom Vision :

    **C#**

    ```
   dotnet add package Microsoft.Azure.CognitiveServices.Vision.CustomVision.Prediction --version 2.0.0
    ```

    **Python**

    ```
   pip install azure-cognitiveservices-vision-customvision==3.1.1
    ```

> **Remarque** : Le package SDK Python inclut à la fois des packages de formation et de prédiction, et peut déjà être installé.

1. Ouvrez le fichier de configuration de votre application cliente (*appsettings.json* pour C# ou *.env* pour Python) et mettez à jour les valeurs de configuration qu’il contient pour refléter le point de terminaison et la clé de votre ressource de *prédiction* Custom Vision, l’ID de projet de votre projet de détection d’objet et le nom de votre modèle publié (qui doit être *fruit-detector*). Enregistrer vos modifications et fermez le fichier.
1. Ouvrez le fichier de code de votre application cliente (*Program.cs* pour C#, *test-detector.py* pour Python) et examinez le code qu’il contient, en notant les détails suivants :
    - Les espaces de noms du package que vous avez installé sont importés
    - La fonction **Main** récupère les paramètres de configuration et utilise la clé et le point de terminaison pour créer un **CustomVisionPredictionClient** authentifié.
    - L’objet client de prédiction est utilisé pour obtenir des prédictions de détection d’objet pour l’image **produce.jpg**, en spécifiant l’ID de projet et le nom du modèle dans la requête. Les régions marquées comme prédites sont ensuite dessinées sur l’image, et le résultat est enregistré dans un fichier **output.jpg**.
1. Fermez l’éditeur de code et saisissez la commande suivante pour exécuter le programme :

    **C#**

    ```
   dotnet run
    ```

    **Python**

    ```
   python test-detector.py
    ```

1. Une fois le programme terminés, dans la barre d’outils du cloud shell, sélectionnez **Charger/Télécharger des fichiers**, puis **Télécharger**. Dans la nouvelle boîte de dialogue, saisissez le chemin d’accès au fichier suivant, puis sélectionnez **Télécharger** :

    **C#**
   
    ```
   mslearn-ai-vision/Labfiles/03-object-detection/C-Sharp/test-detector/output.jpg
    ```

    **Python**
   
    ```
   mslearn-ai-vision/Labfiles/03-object-detection/Python/test-detector/output.jpg
    ```

1. Affichez le fichier **output.jpg** résultant pour afficher les objets détectés dans l’image.

## Nettoyer les ressources

Si vous n’utilisez pas les ressources Azure créées dans ce labo pour d’autres modules de formation, vous pouvez les supprimer pour éviter d’autres frais.

1. Ouvrez le Portail Azure à l'adresse `https://portal.azure.com` et, dans la barre de recherche supérieure, recherchez les ressources que vous avez créées dans ce labo.

1. Dans la page de ressources, sélectionnez **Supprimer** et suivez les instructions pour supprimer la ressource. Vous pouvez également supprimer l’intégralité du groupe de ressources pour nettoyer toutes les ressources en même temps.
   
## Plus d’informations

Pour plus d’informations sur la détection d’objet avec le service Custom Vision, consultez la [documentation Custom Vision](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/).
