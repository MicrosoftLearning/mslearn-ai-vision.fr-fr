---
lab:
  title: Détecter les objets dans les images
  description: "Utilisez le service Azure\_AI\_Custom\_Vision pour entraîner un modèle de détection d’objets."
---

# Détecter les objets dans les images

Le service **Azure AI Custom Vision** vous permet de créer des modèles de vision par ordinateur entraînés sur vos propres images. Vous pouvez l’utiliser pour effectuer l’apprentissage de modèles de *classification d’images* et de *détection d’objet*, que vous pouvez ensuite publier et consommer à partir de vos applications.

Dans cet exercice, vous allez utiliser le service Custom Vision pour effectuer l’apprentissage d’un modèle de *détection d’objet* permettant de détecter et localiser trois classes de fruits (pomme, banane et orange) dans une image.

Bien que cet exercice repose sur le kit de développement logiciel (SDK) Python d’Azure Custom Vision, vous pouvez développer des applications de vision avec différents kits SDK spécifiques à une langue, notamment :

* [Azure Custom Vision pour JavaScript (entraînement)](https://www.npmjs.com/package/@azure/cognitiveservices-customvision-training)
* [Azure Custom Vision pour JavaScript (prédiction)](https://www.npmjs.com/package/@azure/cognitiveservices-customvision-prediction)
* [Azure Custom Vision pour Microsoft .NET (entraînement)](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training/)
* [Azure Custom Vision pour Microsoft .NET (prédiction)](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.CustomVision.Prediction/)
* [Azure Custom Vision pour Java (entraînement)](https://search.maven.org/artifact/com.azure/azure-cognitiveservices-customvision-training/1.1.0-preview.2/jar)
* [Azure Custom Vision pour Java (prédiction)](https://search.maven.org/artifact/com.azure/azure-cognitiveservices-customvision-prediction/1.1.0-preview.2/jar)

Cet exercice prend environ **45** minutes.

## Créer des ressources Custom Vision

Avant de pouvoir effectuer l’apprentissage d’un modèle, vous avez besoin de ressources Azure de *formation* et de *prédiction*. Vous pouvez créer des ressources **Custom Vision** pour chacune de ces tâches, ou bien créer une seule ressource et l’utiliser pour les deux. Dans cet exercice, vous allez créer des ressources **Custom Vision** pour l’entraînement et la prédiction.

1. Ouvrez le [portail Azure](https://portal.azure.com) sur `https://portal.azure.com` et connectez-vous à l’aide de vos informations d’identification Azure. Fermez tous les messages de bienvenue ou conseils qui s’affichent.
1. Sélectionnez **Créer une ressource**.
1. Dans la barre de recherche, recherchez `Custom Vision`, sélectionnez **Custom Vision**, et créez la ressource avec les paramètres suivants :
    - **Options de création** : Les deux
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *créez ou sélectionnez un groupe de ressources*
    - **Région** : *choisissez n’importe quelle région disponible*
    - **Nom** : *nom valide de votre ressource Custom Vision*
    - **Niveau tarifaire de formation** : F0
    - **Niveau tarifaire de prédiction** : F0

1. Créez la ressource et attendez que le déploiement soit terminé, puis affichez les détails du déploiement. Notez que deux ressources Custom Vision sont approvisionnées ; une pour l’entraînement, et une autre pour la prédiction.

    > **Remarque** : Chaque ressource a son propre *point de terminaison* et *ses propres clés*, qui sont utilisées pour gérer l’accès à partir de votre code. Pour effectuer l’apprentissage d’un modèle de classification d’images, votre code doit utiliser la ressource de *formation* (avec son point de terminaison et sa clé) ; si vous souhaitez utiliser le modèle formé pour prédire les classes d’images, votre code doit utiliser la ressource de *prédiction* (avec son point de terminaison et sa clé).

1. Une fois les ressources déployées, accédez au groupe de ressources pour les afficher. Vous devriez voir deux ressources Custom Vision, dont l’une avec le suffixe ***-Prediction***.

## Créer un projet Custom Vision dans le portail Custom Vision

Pour former un modèle de détection d’objets, vous devez créer un projet Custom Vision basé sur votre ressource de formation. Pour ce faire, vous allez utiliser le portail Custom Vision.

1. Ouvrez un nouvel onglet du navigateur (en gardant l’onglet du portail Azure ouvert. Vous y reviendrez plus tard).
1. Dans le nouvel onglet du navigateur, ouvrez le [portail Custom Vision](https://customvision.ai) à l’adresse `https://customvision.ai`. Si vous y êtes invité, connectez-vous à l’aide de vos informations d’identification Azure et acceptez les conditions d’utilisation du service.
1. Créez un nouveau projet avec les paramètres suivants :
    - **Nom :** `Detect Fruit`
    - **Description** : `Object detection for fruit.`
    - **Ressource** : *Votre ressource Custom Vision*
    - **Types de projets** : Détection d’objets
    - **Domaines** : Général
1. Attendez que le projet soit créé et ouvert dans le navigateur.

## Charger et étiqueter des images

Maintenant que vous disposez d’un projet de détection d’objets, vous pouvez charger et étiqueter des images pour entraîner un modèle.

### Charger et étiqueter des images dans le portail Custom Vision

Le portail Custom Vision inclut des outils visuels que vous pouvez utiliser pour charger des images et marquer des régions qui contiennent plusieurs types d’objets.

1. Dans un nouvel onglet du navigateur, téléchargez les [images d’entraînement](https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/main/Labfiles/object-detection/training-images.zip) depuis `https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/main/Labfiles/object-detection/training-images.zip` et extrayez le dossier compressé pour en afficher le contenu. Ce dossier contient des images de fruits.
1. Dans le portail Vision personnalisée, dans votre projet de détection d'objets, sélectionnez **Ajouter des images** et téléchargez toutes les images du dossier extrait.
1. Une fois les images téléchargées, sélectionnez la première pour l’ouvrir.
1. Maintenez la souris sur un objet dans l’image jusqu’à ce qu’une zone détectée automatiquement soit affichée comme l’image ci-dessous. Sélectionnez ensuite l’objet et, si nécessaire, redimensionnez la région pour l’entourer.

    ![Capture d’écran de la région par défaut d’un objet.](../media/object-region.jpg)

    Vous pouvez aussi simplement faire glisser le curseur autour de l’objet pour créer une région.

1. Lorsque la région entoure l’objet, ajoutez une nouvelle balise avec le type d’objet approprié (*apple*, *banana* ou *orange*) comme indiqué ici :

    ![Capture d’écran d’un objet étiqueté dans une image.](../media/object-tag.jpg)

1. Sélectionnez et marquez chaque autre objet dans l’image, en redimensionnant les régions et en ajoutant de nouvelles balises selon les besoins.

    ![Capture d’écran de deux objets étiquetés sur une image.](../media/object-tags.jpg)

1. Utilisez le lien **>** situé à droite pour accéder à l’image suivante et baliser ses objets. Il vous suffit ensuite de continuer à travailler sur la totalité de la collection d’images, en marquant chaque pomme, banane et orange.

1. Une fois que vous avez terminé de marquer la dernière image, fermez l’éditeur **Détails de l’image**. Dans la page **Images de formation**, sous **Balises**, sélectionnez **Avec balise** pour afficher toutes vos images étiquetées :

![Capture d’écran d’images étiquetées dans un projet.](../media/tagged-images.jpg)

### Utiliser le SDK Custom Vision pour charger des images

Vous pouvez utiliser l’interface utilisateur du portail Custom Vision pour étiqueter vos images, mais de nombreuses équipes de développement IA utilisent d’autres outils qui génèrent des fichiers contenant des informations sur les balises et les régions d’objets dans les images. Dans des scénarios comme celui-ci, vous pouvez utiliser l’API de formation Custom Vision pour charger des images étiquetées dans le projet.

1. Cliquez sur l’icône *paramètres* (&#9881;) en haut à droite de la page **Images de formation** dans le portail Custom Vision pour afficher les paramètres du projet.
1. Sous **Général** (à gauche), notez l’**ID de projet** qui identifie ce projet de façon unique.
1. La **Clé** et le **Point de terminaison** s’affichent à droite, sous la section **Ressources**. Vous y trouverez les détails de la ressource de *formation* (vous pouvez également obtenir ces informations en consultant la ressource dans le Portail Azure).
1. Revenez à l’onglet du navigateur contenant le portail Azure (en gardant l’onglet du portail Custom Vision ouvert. Vous y reviendrez plus tard).
1. Dans le portail Azure, utilisez le bouton **[\>_]** situé à droite de la barre de recherche en haut de la page pour créer un nouveau Cloud Shell dans le portail Azure, en sélectionnant un environnement ***PowerShell*** sans stockage dans votre abonnement.

    Cloud Shell fournit une interface de ligne de commande via un volet situé en bas du portail Azure.

    > **Remarque** : si vous avez déjà créé un Cloud Shell qui utilise un environnement *Bash*, basculez-le vers ***PowerShell***.

    > **Remarque** : si le portail sollicite la sélection d’un stockage pour la persistance des fichiers, choisissez **Aucun compte de stockage requis**, sélectionnez l’abonnement concerné et appuyez sur **Appliquer**.

1. Dans la barre d’outils Cloud Shell, dans le menu **Paramètres**, sélectionnez **Accéder à la version classique** (cela est nécessaire pour utiliser l’éditeur de code).

    **<font color="red">Assurez-vous d’avoir basculé vers la version classique du Cloud Shell avant de continuer.</font>**

1. Redimensionnez le volet Cloud Shell afin que vous puissiez en voir davantage.

    > **Conseil** : vous pouvez redimensionner le volet en faisant glisser la bordure supérieure. Vous pouvez également utiliser les boutons de réduction et d’agrandissement pour alterner entre le Cloud Shell et l’interface principale du portail.

1. Dans le volet Cloud Shell, saisissez les commandes suivantes pour cloner le référentiel GitHub contenant les fichiers de code pour cet exercice (saisissez la commande, ou copiez-la dans le presse-papiers puis effectuez un clic droit dans la ligne de commande pour la coller en texte brut) :

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **Conseil** : lorsque vous collez des commandes dans cloudshell, la sortie peut prendre une grande quantité de mémoire tampon d’écran. Vous pouvez effacer le contenu de l’écran en saisissant la commande `cls` pour faciliter le focus sur chaque tâche.

1. Une fois le référentiel cloné, utilisez la commande suivante pour accéder aux fichiers de code de l’application :

    ```
   cd mslearn-ai-vision/Labfiles/object-detection/python/train-detector
   ls -a -l
    ```

    Le dossier contient les fichiers de code et de configuration de votre application. Il contient également un fichier **tagged-images.json** qui contient à son tour des coordonnées de cadre englobant pour les objets dans plusieurs images et un sous-dossier **/images**, qui contient les images.

1. Installez le kit de développement logiciel (SDK) Azure AI Custom Vision pour l’entraînement et les éventuels autres packages requis à l’aide des commandes ci-dessous :

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-cognitiveservices-vision-customvision
    ```

1. Entrez la commande suivante pour modifier le fichier de configuration de votre application :

    ```
   code .env
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Dans le fichier de code, mettez à jour les valeurs de configuration qu’il contient afin de tenir compte du **point de terminaison** et une **clé** d’authentification de votre ressource *d’entraînement* Custom Vision, ainsi que l’**ID de projet** du projet Custom Vision que vous avez créé précédemment.
1. Une fois les espaces réservés remplacés, utilisez la commande **CTRL+S** dans l’éditeur de code pour enregistrer vos modifications, puis la commande **CTRL+Q** pour fermer l’éditeur tout en gardant la ligne de commande Cloud Shell ouverte.
1. Dans la ligne de commande Cloud Shell, entrez la commande suivante pour ouvrir le fichier **tagged-images.json** pour afficher les informations d’étiquetage des fichiers image dans le sous-dossier **/images** :

    ```
   code tagged-images.json
    ```
    
     JSON définit une liste d’images, chacune contenant une ou plusieurs régions étiquetées. Chaque région balisée comprend un nom de balise, les coordonnées supérieure et gauche, ainsi que les dimensions de largeur et de hauteur du cadre englobant contenant l’objet étiqueté.

    > **Remarque** : Les coordonnées et les dimensions de ce fichier indiquent des points relatifs sur l’image. Par exemple, une valeur de *hauteur* de 0,7 indique une zone représentant 70 % de la hauteur de l’image. Certains outils d’étiquetage génèrent d’autres formats de fichier dans lesquels les valeurs de coordonnées et de dimension représentent des pixels, des pouces ou d’autres unités de mesure.

1. Fermez le fichier JSON sans enregistrer de modifications (*CTRL_Q*).

1. Dans la ligne de commande Cloud Shell, entrez la commande suivante pour ouvrir le fichier de code de l’application cliente :

    ```
   code add-tagged-images.py
    ```

1. Notez les détails suivants dans le fichier de code :
    - Les espaces de noms du kit SDK Azure AI Custom Vision sont importés.
    - La fonction **Main** récupère les paramètres de configuration et utilise la clé et le point de terminaison pour créer un **CustomVisionTrainingClient** authentifié, qui est ensuite utilisé avec l’ID de projet pour créer une référence **Project** sur votre projet.
    - La fonction **Upload_Images** extrait les informations de région étiquetées du fichier JSON et les utilise pour créer un lot d’images avec des régions, qu’elle charge ensuite dans le projet.

1. Fermez l’éditeur de code (*CTRL+Q*) et entrez la commande suivante pour exécuter le programme :

    ```
   python add-tagged-images.py
    ```

1. Attendez la fin de l’exécution du programme.
1. Revenez à l’onglet du navigateur contenant le portail Custom Vision (en gardant l’onglet Cloud Shell du portail Azure ouvert) et affichez la page **Images d’entraînement** de votre projet (actualisez le navigateur si nécessaire).
1. Vérifiez que certaines nouvelles images étiquetées ont été ajoutées au projet.

## Formation et test d’un modèle

Maintenant que vous avez balisé les images de votre projet, vous pouvez former un modèle.

1. Dans le projet Custom Vision, cliquez sur **Entraîner** (&#9881;<sub>&#9881;</sub>) pour entraîner un modèle de détection d’objets à l’aide des images balisées. Sélectionnez l’option **Entraînement rapide**.
1. Attendez que l’entraînement se termine (cette opération peut prendre dix minutes).

    > **Conseil** : Azure Cloud Shell a un délai d’inactivité de 20 minutes, après quoi la session est abandonnée. Pendant que vous attendez la fin de l’entraînement, revenez de temps en temps au shell cloud et entrez une commande comme `ls` pour maintenir la session active.

1. Dans le portail Custom Vision, une fois l’entraînement terminé, passez en revue les métriques de performances *précision*, *rappel* et *mAP* : elles mesurent le degré de précision des prédictions du modèle de détection d’objet et doivent toutes être élevées.
1. En haut à droite de la page, cliquez sur **Test rapide**, puis, dans la zone **URL de l’image**, tapez `https://aka.ms/test-fruit` et cliquez sur le bouton *Image de test rapide* (&#10132;).
1. Affichez la prédiction générée.

    ![Capture d’écran de la détection d’objets de test](../media/test-object-detection.png)

1. Fermez la fenêtre **Test rapide**.

## Utiliser le détecteur d’objets dans une application cliente

Vous pouvez désormais publier votre modèle entraîné et l’utiliser dans une application cliente.

### Publier le modèle de détection d’objet

1. Sur le portail Custom Vision, sur la page **Performances**, cliquez sur **&#128504; Publier** pour publier le modèle formé avec les paramètres suivants :
    - **Nom du modèle** : `fruit-detector`
    - **Ressource de prédiction** : *ressource de **prédiction** que vous avez créée précédemment et qui se termine par « -Prediction » (<u>autre que</u> la ressource de formation)* .
1. En haut à gauche de la page **Paramètres du projet**, cliquez sur l’icône *Galerie de projets* (&#128065;) pour revenir à la page d’accueil du portail Custom Vision, où votre projet est maintenant répertorié.
1. Sur la page d’accueil du portail Custom Vision, en haut à droite, cliquez sur l’icône *Paramètres* (&#9881;) pour afficher les paramètres de votre service Custom Vision. Ensuite, sous **Ressources**, recherchez votre ressource de *prédiction* qui se termine par « -Prediction » (<u>et non</u> la ressource de formation) pour indiquer ses valeurs de **Clé** et de **Point de terminaison** (vous pouvez également obtenir ces informations en affichant la ressource dans le Portail Azure).

## Utiliser le classifieur d’images à partir d’une application cliente

Maintenant que vous avez publié le modèle de classification d’images, vous pouvez l’utiliser à partir d’une application cliente. Là encore, vous avez le choix entre les langages **C#** et **Python**.

1. Revenez à l'onglet du navigateur contenant le portail Azure et le volet Cloud Shell.
1. Dans Cloud Shell, exécutez les commandes suivantes pour basculer vers le dossier de votre application cliente et afficher les fichiers qu’il contient :

    ```
   cd ../test-detector
   ls -a -l
    ```

    Le dossier contient les fichiers de code et de configuration de votre application. Il contient également le fichier image **produce.jpg ** suivant, que vous utiliserez pour tester votre modèle.

    ![Image d’un fruit.](../media/produce.jpg)

1. Installez le kit de développement logiciel (SDK) Azure AI Custom Vision pour la prédiction et les éventuels autres packages requis en exécutant les commandes suivantes :

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-cognitiveservices-vision-customvision
    ```

1. Entrez la commande suivante pour modifier le fichier de configuration de votre application :

    ```
   code .env
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Mettez à jour les valeurs de configuration afin de tenir compte du **point de terminaison** et la **clé** de votre ressource de *<u>prédiction</u>* Custom Vision, l’**ID de projet** du projet de détection d’objets, ainsi que le nom de votre modèle publié (qui doit être *fruit-detector*) Enregistrez vos modifications (*Ctrl+S*) et fermez l’éditeur de code (*Ctrl+Q*).

1. Dans la ligne de commande Cloud Shell, entrez la commande suivante pour ouvrir le fichier de code de l’application cliente :

    ```
   code test-detector.py
    ```

1. Passez en revue le code, en notant les détails suivants :
    - Les espaces de noms du kit SDK Azure AI Custom Vision sont importés.
    - La fonction **Main** récupère les paramètres de configuration et utilise la clé et le point de terminaison pour créer un **CustomVisionPredictionClient** authentifié.
    - L’objet client de prédiction est utilisé pour obtenir des prédictions de détection d’objet pour l’image **produce.jpg**, en spécifiant l’ID de projet et le nom du modèle dans la requête. Les régions marquées comme prédites sont ensuite dessinées sur l’image, et le résultat est enregistré dans un fichier **output.jpg**.
1. Fermez l’éditeur de code et entrez la commande suivante pour exécuter le programme :

    ```
   python test-detector.py
    ```

1. Passez en revue les résultats du programme, qui dressent la liste de chaque objet détecté dans l’image.
1. Notez qu’un fichier image nommé **output.jpg** est généré. Utilisez la commande **téléchargement** (spécifique à Azure Cloud Shell) pour le télécharger :

    ```
   download output.jpg
    ```

    La commande de téléchargement crée un lien contextuel en bas à droite de votre navigateur, que vous pouvez sélectionner pour télécharger et ouvrir le fichier. L’image doit ressembler à ce qui suit :

    ![Image avec les objets détectés mis en évidence.](../media/object-detection-output.jpg )

## Nettoyer les ressources

Si vous n’utilisez pas les ressources Azure créées dans ce labo pour d’autres modules de formation, vous pouvez les supprimer pour éviter d’autres frais.

1. Ouvrez le Portail Azure à l'adresse `https://portal.azure.com` et, dans la barre de recherche supérieure, recherchez les ressources que vous avez créées dans ce labo.

1. Dans la page de ressources, sélectionnez **Supprimer** et suivez les instructions pour supprimer la ressource. Vous pouvez également supprimer l’intégralité du groupe de ressources pour nettoyer toutes les ressources en même temps.
   
## Plus d’informations

Pour plus d’informations sur la détection d’objet avec le service Custom Vision, consultez la [documentation Custom Vision](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/).
