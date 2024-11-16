---
lab:
  title: "Détecter des objets dans des images avec Azure\_AI\_Custom\_Vision"
---

# Détecter des objets dans des images avec Azure AI Custom Vision

Dans cet exercice, vous allez utiliser le service Custom Vision pour effectuer l’apprentissage d’un modèle de *détection d’objet* permettant de détecter et localiser trois classes de fruits (pomme, banane et orange) dans une image.

## Cloner le référentiel pour ce cours

Si vous avez déjà cloné le référentiel de code **mslearn-ai-vision** dans l’environnement dans lequel vous travaillez sur ce labo, ouvrez-le dans Visual Studio Code ; sinon, procédez comme suit pour le cloner maintenant.

1. Démarrez Visual Studio Code.
2. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git : Cloner** pour cloner le référentiel `https://github.com/MicrosoftLearning/mslearn-ai-vision` vers un dossier local (peu importe quel dossier).
3. Lorsque le référentiel a été cloné, ouvrez le dossier dans Visual Studio Code.
4. Attendez que des fichiers supplémentaires soient installés pour prendre en charge les projets de code C# dans le référentiel.

    > **Remarque** : Si vous êtes invité à ajouter des ressources requises pour générer et déboguer, sélectionnez **Not Now** (Pas maintenant).

## Créer des ressources Custom Vision

Si vous disposez déjà de ressources de formation et de prédiction **Custom Vision** dans votre abonnement Azure, vous pouvez les utiliser ou utiliser un compte multiservices existant dans cet exercice. Sinon, utilisez les instructions suivantes pour les créer.

> **Remarque** : si vous utilisez un compte multiservices, la clé et le point de terminaison pour la formation et pour la prédiction sont identiques.

1. Dans un nouvel onglet de navigateur, ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
2. Sélectionnez le bouton **&#65291;Créer une ressource**, recherchez *Custom Vision*, puis créez une ressource **Custom Vision** avec les paramètres suivants :
    - **Options de création** : Les deux
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *Choisissez ou créez un groupe de ressources. (Si vous utilisez un abonnement restreint, vous n’avez peut-être pas l’autorisation de créer un groupe de ressources. Dans ce cas, utilisez le groupe fourni.)*
    - **Région** : *choisissez n’importe quelle région disponible*
    - **Nom** : *Entrez un nom unique.*
    - **Niveau tarifaire de formation** : F0
    - **Niveau tarifaire de prédiction** : F0

    > **Remarque** : Si vous avez déjà un service de vision personnalisée F0 dans votre abonnement, sélectionnez **S0** pour celui-ci.

3. Attendez que les ressources soient créées, puis affichez les détails du déploiement. Vous noterez que deux ressources Custom Vision sont approvisionnées : une pour la formation et une autre pour la prédiction (que vous pouvez facilement repérer par le suffixe **-Prediction**). Vous pouvez les visualiser en naviguant vers le groupe de ressources dans lequel vous les avez créées.

> **Important** : Chaque ressource a son propre *point de terminaison* et *ses propres clés*, qui sont utilisées pour gérer l’accès à partir de votre code. Pour effectuer l’apprentissage d’un modèle de classification d’images, votre code doit utiliser la ressource de *formation* (avec son point de terminaison et sa clé) ; si vous souhaitez utiliser le modèle formé pour prédire les classes d’images, votre code doit utiliser la ressource de *prédiction* (avec son point de terminaison et sa clé).

## Créer un projet Custom Vision

Pour former un modèle de détection d’objets, vous devez créer un projet Custom Vision basé sur votre ressource de formation. Pour ce faire, vous allez utiliser le portail Custom Vision.

1. Dans un nouvel onglet de navigateur, ouvrez le portail Custom Vision à l’adresse `https://customvision.ai` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
2. Créez un nouveau projet avec les paramètres suivants :
    - **Nom** : Détecter les fruits
    - **Description** : Détection d’objet pour les fruits.
    - **Ressource**: *La ressource Custom Vision que vous avez créée précédemment*
    - **Types de projets** : Détection d’objets
    - **Domaines** : Général
3. Attendez que le projet soit créé et ouvert dans le navigateur.

## Ajouter et baliser des images

Pour former un modèle de détection d’objets, vous devez télécharger des images qui contiennent les classes que vous souhaitez que le modèle identifie, et les marquer pour qu’elles indiquent des zones englobantes pour chaque instance d’objet.

1. Dans Visual Studio Code, affichez les images de formation dans le dossier **03-object-detection/training-images** où vous avez cloné le référentiel. Ce dossier contient des images de fruits.
2. Dans le portail Vision personnalisée, dans votre projet de détection d'objets, sélectionnez **Ajouter des images** et téléchargez toutes les images du dossier extrait.
3. Une fois les images téléchargées, sélectionnez la première pour l’ouvrir.
4. Maintenez la souris sur un objet dans l’image jusqu’à ce qu’une zone détectée automatiquement soit affichée comme l’image ci-dessous. Sélectionnez ensuite l’objet et, si nécessaire, redimensionnez la région pour l’entourer.

    ![Région par défaut d’un objet](../media/object-region.jpg)

    Vous pouvez aussi simplement faire glisser le curseur autour de l’objet pour créer une région.

5. Lorsque la région entoure l’objet, ajoutez une nouvelle balise avec le type d’objet approprié (*apple*, *banana* ou *orange*) comme indiqué ici :

    ![Objet balisé dans une image](../media/object-tag.jpg)

6. Sélectionnez et marquez chaque autre objet dans l’image, en redimensionnant les régions et en ajoutant de nouvelles balises selon les besoins.

    ![Deux objets avec des balises dans une image](../media/object-tags.jpg)

7. Utilisez le lien **>** situé à droite pour accéder à l’image suivante et baliser ses objets. Il vous suffit ensuite de continuer à travailler sur la totalité de la collection d’images, en marquant chaque pomme, banane et orange.

8. Une fois que vous avez terminé de marquer la dernière image, fermez l’éditeur **Détails de l’image**. Dans la page **Images de formation**, sous **Balises**, sélectionnez **Avec balise** pour afficher toutes vos images étiquetées :

![Images avec balise dans un projet](../media/tagged-images.jpg)

## Utiliser l’API de formation pour charger des images

Vous pouvez utiliser l’interface utilisateur du portail Custom Vision pour étiqueter vos images, mais de nombreuses équipes de développement IA utilisent d’autres outils qui génèrent des fichiers contenant des informations sur les balises et les régions d’objets dans les images. Dans des scénarios comme celui-ci, vous pouvez utiliser l’API de formation Custom Vision pour charger des images étiquetées dans le projet.

> **Remarque** : Dans cet exercice, vous pouvez choisir d’utiliser l’API à partir du SDK **C#** ou **Python**. Dans les étapes qui suivent, effectuez les actions appropriées pour votre langage préféré.

1. Cliquez sur l’icône *paramètres* (&#9881;) en haut à droite de la page **Images de formation** dans le portail Custom Vision pour afficher les paramètres du projet.
2. Sous **Général** (à gauche), notez l’**ID de projet** qui identifie ce projet de façon unique.
3. La clé et le point de terminaison s’affichent à droite, sous la section **Ressources**. Vous y trouverez les détails de la ressource de *formation* (vous pouvez également obtenir ces informations en consultant la ressource dans le Portail Azure).
4. Dans Visual Studio Code, dans le dossier **03-object-detection**, développez le dossier **C-Sharp** ou **Python** en fonction de votre préférence de langage.
5. Cliquez avec le bouton droit de la souris sur le dossier **train-detector** et ouvrez un terminal intégré. Installez ensuite le package de formation Custom Vision en exécutant la commande appropriée pour votre préférence de langage :

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training --version 2.0.0
```

**Python**

```
pip install azure-cognitiveservices-vision-customvision==3.1.1
```

6. Affichez le contenu du dossier **train-detector**, et notez qu’il contient un fichier pour les paramètres de configuration :
    - **C#** : appsettings.json
    - **Python** : .env

    Ouvrez le fichier de configuration et mettez à jour les valeurs de configuration qu’il contient pour refléter le point de terminaison et la clé de votre ressource de *formation* Custom Vision, ainsi que l’ID du projet de détection d’objet que vous avez créé précédemment. Enregistrez vos modifications.

7. Dans le dossier **train-detector**, ouvrez **tagged-images.json** et examinez le fichier JSON qu’il contient. Le fichier JSON définit une liste d’images, chacune contenant une ou plusieurs régions étiquetées. Chaque région balisée comprend un nom de balise, les coordonnées supérieure et gauche, ainsi que les dimensions de largeur et de hauteur du cadre englobant contenant l’objet étiqueté.

    > **Remarque** : Les coordonnées et les dimensions de ce fichier indiquent des points relatifs sur l’image. Par exemple, une valeur de *hauteur* de 0,7 indique une zone représentant 70 % de la hauteur de l’image. Certains outils d’étiquetage génèrent d’autres formats de fichier dans lesquels les valeurs de coordonnées et de dimension représentent des pixels, des pouces ou d’autres unités de mesure.

8. Notez que le dossier **train-detector** contient un sous-dossier dans lequel sont stockés les fichiers image référencés dans le fichier JSON.

9. Notez également que le dossier **train-detector** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : train-detector.py

    Ouvrez le fichier de code et passez en revue le code qu’il contient, en notant les détails suivants :
    - Les espaces de noms du package que vous avez installé sont importés
    - La fonction **Main** récupère les paramètres de configuration et utilise la clé et le point de terminaison pour créer un **CustomVisionTrainingClient** authentifié, qui est ensuite utilisé avec l’ID de projet pour créer une référence **Project** sur votre projet.
    - La fonction **Upload_Images** extrait les informations de région étiquetées du fichier JSON et les utilise pour créer un lot d’images avec des régions, qu’elle charge ensuite dans le projet.

10. Revenez au terminal intégré pour accéder au dossier **train-detector**, puis entrez la commande suivante pour exécuter le programme :
    
    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python train-detector.py
    ```
    
11. Attendez la fin de l’exécution du programme. Revenez ensuite à votre navigateur et affichez la page **Images de formation** de votre projet dans le portail Custom Vision (actualisez le navigateur si nécessaire).
12. Vérifiez que certaines nouvelles images étiquetées ont été ajoutées au projet.

## Formation et test d’un modèle

Maintenant que vous avez balisé les images de votre projet, vous pouvez former un modèle. pour

1. Dans le projet Custom Vision, cliquez sur **Former** pour former un modèle de détection d’objets à l’aide des images balisées. Sélectionnez l’option **Entraînement rapide**.
2. Attendez la fin de la formation (cela peut prendre une dizaine de minutes), puis vérifiez les mesures de performance *Précision*, *Rappel*, et *mAP* ; elles mesurent la précision de prédiction du modèle de classification et doivent toutes être élevées.
3. En haut à droite de la page, cliquez sur **Test rapide**, puis dans la zone **URL de l’image**, entrez `https://aka.ms/apple-orange` et affichez la prédiction générée. Fermez ensuite la fenêtre **Test rapide**.

## Publier le modèle de détection d’objet

Vous êtes maintenant prêt à publier votre modèle formé et à l’utiliser à partir d’une application cliente.

1. Sur le portail Custom Vision, sur la page **Performances**, cliquez sur **&#128504; Publier** pour publier le modèle formé avec les paramètres suivants :
    - **Nom du modèle** : fruit-detector
    - **Ressource de prédiction** : *ressource de **prédiction** que vous avez créée précédemment et qui se termine par « -Prediction » (<u>autre que</u> la ressource de formation)* .
2. En haut à gauche de la page **Paramètres du projet**, cliquez sur l’icône *Galerie de projets* (&#128065;) pour revenir à la page d’accueil du portail Custom Vision, où votre projet est maintenant répertorié.
3. Sur la page d’accueil du portail Custom Vision, en haut à droite, cliquez sur l’icône *Paramètres* (&#9881;) pour afficher les paramètres de votre service Custom Vision. Ensuite, sous **Ressources**, recherchez votre ressource de *prédiction* qui se termine par « -Prediction » (<u>et non</u> la ressource de formation) pour indiquer ses valeurs de **Clé** et de **Point de terminaison** (vous pouvez également obtenir ces informations en affichant la ressource dans le Portail Azure).

## Utiliser le classifieur d’images à partir d’une application cliente

Maintenant que vous avez publié le modèle de classification d’images, vous pouvez l’utiliser à partir d’une application cliente. Là encore, vous avez le choix entre les langages **C#** et **Python**.

1. Dans Visual Studio Code, accédez au dossier **03-object-detection** et, dans le dossier correspond au langage que vous avez choisi (**C-Sharp** ou **Python**), développez le dossier **test-detector**.
2. Cliquez avec le bouton droit de la souris sur le dossier **test-detector** et ouvrez un terminal intégré. Entrez ensuite la commande suivante, selon votre kit de développement logiciel (SDK), pour installer le package de prédiction Custom Vision :

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.CustomVision.Prediction --version 2.0.0
```

**Python**

```
pip install azure-cognitiveservices-vision-customvision==3.1.1
```

> **Remarque** : Le package SDK Python inclut à la fois des packages de formation et de prédiction, et peut déjà être installé.

3. Ouvrez le fichier de configuration de votre application cliente (*appsettings.json* pour C# ou *.env* pour Python) et mettez à jour les valeurs de configuration qu’il contient pour refléter le point de terminaison et la clé de votre ressource de *prédiction* Custom Vision, l’ID de projet de votre projet de détection d’objet et le nom de votre modèle publié (qui doit être *fruit-detector*). Enregistrez vos modifications.
4. Ouvrez le fichier de code de votre application cliente (*Program.cs* pour C#, *test-detector.py* pour Python) et examinez le code qu’il contient, en notant les détails suivants :
    - Les espaces de noms du package que vous avez installé sont importés
    - La fonction **Main** récupère les paramètres de configuration et utilise la clé et le point de terminaison pour créer un **CustomVisionPredictionClient** authentifié.
    - L’objet client de prédiction est utilisé pour obtenir des prédictions de détection d’objet pour l’image **produce.jpg**, en spécifiant l’ID de projet et le nom du modèle dans la requête. Les régions marquées comme prédites sont ensuite dessinées sur l’image, et le résultat est enregistré dans un fichier **output.jpg**.
5. Revenez au terminal intégré pour accéder au dossier **test-detector**, puis entrez la commande suivante pour exécuter le programme :

**C#**

```
dotnet run
```

**Python**

```
python test-detector.py
```

6. Une fois le programme terminé, affichez le fichier **output.jpg** obtenu pour voir les objets détectés dans l’image.

## Plus d’informations

Pour plus d’informations sur la détection d’objet avec le service Custom Vision, consultez la [documentation Custom Vision](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/).
