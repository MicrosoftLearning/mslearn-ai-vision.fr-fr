---
lab:
  title: Classer des images avec Azure AI Custom Vision
---

# Classer des images avec Azure AI Custom Vision

Le service **Azure AI Custom Vision** vous permet de créer des modèles de vision par ordinateur entraînés sur vos propres images. Vous pouvez l’utiliser pour effectuer l’apprentissage de modèles de *classification d’images* et de *détection d’objet*, que vous pouvez ensuite publier et consommer à partir de vos applications.

Dans cet exercice, vous allez utiliser le service Custom Vision pour effectuer l’apprentissage d’un modèle de classification d’images permettant d’identifier trois classes de fruits (pomme, banane et orange).

## Cloner le référentiel pour ce cours

Si vous n’avez pas déjà cloné le référentiel de code **mslearn-ai-vision** dans l’environnement où vous travaillez sur ce labo, procédez comme suit. Sinon, ouvrez le dossier cloné dans Visual Studio Code.

1. Démarrez Visual Studio Code.
2. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git : Cloner** pour cloner le référentiel `https://github.com/MicrosoftLearning/mslearn-ai-vision` vers un dossier local (peu importe quel dossier).
3. Lorsque le référentiel a été cloné, ouvrez le dossier dans Visual Studio Code.
4. Attendez que des fichiers supplémentaires soient installés pour prendre en charge les projets de code C# dans le référentiel.

    > **Remarque** : si vous êtes invité à ajouter des ressources requises pour générer et déboguer, sélectionnez **Not Now** (Pas maintenant). Si un message vous signale qu’*un projet Azure Functions a été détecté dans le dossier*, vous pouvez fermer ce message en toute sécurité.

## Créer des ressources Custom Vision

Avant de pouvoir effectuer l’apprentissage d’un modèle, vous avez besoin de ressources Azure de *formation* et de *prédiction*. Vous pouvez créer des ressources **Custom Vision** pour chacune de ces tâches, ou bien créer une seule ressource **Azure AI Services** et l’utiliser pour l’une ou l’autre (ou les deux).

Dans cet exercice, vous allez créer des ressources **Custom Vision** pour la formation et la prédiction afin de pouvoir gérer séparément l’accès et les coûts de ces charges de travail.

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

Pour effectuer l’apprentissage d’un modèle de classification d’images, vous devez créer un projet Custom Vision basé sur votre ressource de formation. Pour ce faire, vous allez utiliser le portail Custom Vision.

1. Dans Visual Studio Code, affichez les images de la formation dans le dossier **07-image-classification/training-images** où vous avez cloné le référentiel. Ce dossier contient des sous-dossiers d’images de pommes, de bananes et d’oranges.
2. Dans un autre onglet de navigateur, ouvrez le portail Custom Vision à l’adresse `https://customvision.ai`. Si vous y êtes invité, connectez-vous en utilisant le compte Microsoft associé à votre abonnement Azure et acceptez les conditions de service.
3. Dans le portail Custom Vision, créez un nouveau projet avec les paramètres suivants :
    - **Nom** : Classer les fruits
    - **Description** : Classification d’images pour les fruits
    - **Ressource**: *La ressource Custom Vision que vous avez créée précédemment*
    - **Types de projets** : Classification
    - **Types de classification** : Multiclasse (une balise par image)
    - **Domaines** : Nourriture
4. Dans le nouveau projet, cliquez sur **\[+\] Ajouter des images**, puis sélectionnez tous les fichiers du dossier **training-images/apple** que vous avez consulté précédemment. Téléchargez ensuite les fichiers image en spécifiant la balise *apple*, comme suit :

![Charger apple avec la balise apple](../media/upload_apples.jpg)
   
5. Répétez l’étape précédente pour charger les images dans le dossier **banana** avec la balise *banana* et les images du dossier **orange** avec la balise *orange*.
6. Explorez les images que vous avez chargées dans le projet Custom Vision. Il doit y avoir 15 images de chaque classe, comme suit :

![Images avec balises de fruits : 15 apples, 15 bananas et 15 oranges](../media/fruit.jpg)
    
7. Dans le projet Custom Vision, au-dessus des images, cliquez sur **Former** pour former un modèle de classification à l’aide des images avec balises. Sélectionnez l’option **Entraînement rapide**, puis attendez la fin de l’itération de formation (cette opération peut prendre une minute).
8. Une fois la formation de l’itération de modèle terminée, examinez les métriques de performance *Précision*, *Rappel* et *AP* : elles mesurent la précision de la prédiction du modèle de classification et doivent toutes être élevées.

> **Remarque** : Les métriques de performances sont basées sur un seuil de probabilité de 50 % pour chaque prédiction (en d’autres termes, si le modèle calcule une probabilité de 50 % ou supérieure qu’une image est d’une classe particulière, alors cette classe est prédite). Vous pouvez ajuster ce seuil en haut à gauche de la page.

## Tester le modèle

Maintenant que vous avez effectué l’apprentissage du modèle, vous pouvez le tester.

1. Au-dessus des métriques de performances, cliquez sur **Test rapide**.
2. Dans la zone **URL de l’image**, entrez `https://aka.ms/apple-image`, puis cliquez sur &#10132;
3. Vérifiez les prédictions retournées par votre modèle. Le score de probabilité pour *apple* devrait être le plus élevé, comme ceci :

![Une image avec une prédiction de classe apple](../media/test-apple.jpg)

4. Fermez la fenêtre **Test rapide**.

## Afficher les paramètres du projet

Le projet que vous avez créé a été affecté à un identificateur unique, que vous devez spécifier dans n’importe quel code qui interagit avec celui-ci.

1. Cliquez sur l’icône *Paramètres* (&9881;) en haut à droite de la page **Performances** pour afficher les paramètres du projet.
2. Sous **Général** (à gauche), notez l’**ID de projet** qui identifie ce projet de façon unique.
3. La clé et le point de terminaison s’affichent à droite, sous la section **Ressources**. Vous y trouverez les détails de la ressource de *formation* (vous pouvez également obtenir ces informations en consultant la ressource dans le Portail Azure).

## Utiliser l’API de *formation*

Le portail Custom Vision fournit une interface utilisateur pratique que vous pouvez utiliser non seulement pour charger et marquer des images, mais également pour effectuer l’apprentissage de modèles. Toutefois, dans certains scénarios, vous aurez peut-être intérêt à automatiser la formation de modèles à l’aide de l’API de formation Custom Vision.

> **Remarque** : Dans cet exercice, vous pouvez choisir d’utiliser l’API à partir du SDK **C#** ou **Python** . Dans les étapes qui suivent, effectuez les actions appropriées pour votre langage préféré.

1. Dans Visual Studio Code, dans le volet **Explorer**, accédez au dossier **07-custom-vision-image-classification** et développez le dossier **C-Sharp** ou **Python** en fonction de votre préférence de langage.
2. Cliquez avec le bouton droit de la souris sur le dossier **train-classifier** et ouvrez un terminal intégré. Installez ensuite le package de formation Custom Vision en exécutant la commande appropriée pour votre préférence de langage :

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training --version 2.0.0
```

**Python**

```
pip install azure-cognitiveservices-vision-customvision==3.1.0
```

3. Affichez le contenu du dossier **train-classifier** ; notez qu’il contient un fichier pour les paramètres de configuration :
    - **C#** : appsettings.json
    - **Python** : .env

    Ouvrez le fichier de configuration et mettez à jour les valeurs de configuration qu’il contient pour refléter le point de terminaison et la clé de votre ressource de *formation* Custom Vision, ainsi que l’ID du projet de classification que vous avez créé précédemment. Enregistrez vos modifications.
4. Notez que le dossier **train-classifier** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : train-classifier.py

    Ouvrez le fichier de code et passez en revue le code qu’il contient, en notant les détails suivants :
    - Les espaces de noms du package que vous avez installé sont importés
    - La fonction **Main** récupère les paramètres de configuration et utilise la clé et le point de terminaison pour créer un **CustomVisionTrainingClient** authentifié, qui est ensuite utilisé avec l’ID de projet pour créer une référence **Project** sur votre projet.
    - La fonction **Upload_Images** récupère les balises définies dans le projet Custom Vision, puis charge les fichiers image à partir de dossiers ayant un nom correspondant dans le projet, en affectant l’ID de balise approprié.
    - La fonction **Train_Model** crée une nouvelle itération de formation pour le projet et attend la fin de la formation.
5. Revenez au terminal intégré pour accéder au dossier **train-classifier**, puis entrez la commande suivante pour exécuter le programme :

**C#**

```
dotnet run
```

**Python**

```
python train-classifier.py
```
    
6. Attendez la fin de l’exécution du programme. Revenez ensuite à votre navigateur et affichez la page **Images de formation** de votre projet dans le portail Custom Vision (actualisez le navigateur si nécessaire).
7. Vérifiez que certaines nouvelles images étiquetées ont été ajoutées au projet. Affichez ensuite la page **Performances** et vérifiez qu’une nouvelle itération a été créée.

## Publier le modèle de classification d’images

Vous êtes maintenant prêt à publier votre modèle formé et à l’utiliser à partir d’une application cliente.

1. Sur le portail Custom Vision, sur la page **Performances**, cliquez sur **&#128504; Publier** pour publier le modèle formé avec les paramètres suivants :
    - **Nom du modèle** : fruit-classifier
    - **Ressource de prédiction** : *ressource de **prédiction** que vous avez créée précédemment et qui se termine par « -Prediction » (<u>autre que</u> la ressource de formation)* .
2. En haut à gauche de la page **Paramètres du projet**, cliquez sur l’icône *Galerie de projets* (&#128065;) pour revenir à la page d’accueil du portail Custom Vision, où votre projet est maintenant répertorié.
3. Sur la page d’accueil du portail Custom Vision, en haut à droite, cliquez sur l’icône *Paramètres* (&#9881;) pour afficher les paramètres de votre service Custom Vision. Ensuite, sous **Ressources**, recherchez votre ressource de *prédiction* qui se termine par « -Prediction » (<u>et non</u> la ressource de formation) pour indiquer ses valeurs de **Clé** et de **Point de terminaison** (vous pouvez également obtenir ces informations en affichant la ressource dans le Portail Azure).

## Utiliser le classifieur d’images à partir d’une application cliente

Maintenant que vous avez publié le modèle de classification d’images, vous pouvez l’utiliser à partir d’une application cliente. Là encore, vous avez le choix entre les langages **C#** et **Python**.

1. Dans Visual Studio Code, dans le dossier **07-custom-vision-image-classification**, au niveau du sous-dossier correspondant au langage que vous avez choisi (**C-Sharp** ou **Python**), cliquez avec le bouton droit de la souris sur le dossier **test-classifier** et ouvrez un terminal intégré. Entrez ensuite la commande suivante, selon votre kit de développement logiciel (SDK), pour installer le package de prédiction Custom Vision :

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.CustomVision.Prediction --version 2.0.0
```

**Python**

```
pip install azure-cognitiveservices-vision-customvision==3.1.0
```

> **Remarque** : Le package SDK Python inclut à la fois des packages de formation et de prédiction, et peut déjà être installé.

2. Développez le dossier **test-classifier** pour afficher les fichiers qu’il contient, qui sont utilisés pour implémenter une application cliente de test pour votre modèle de classification d’images.
3. Ouvrez le fichier de configuration de votre application cliente (*appsettings.json* pour C# ou *.env* pour Python) et mettez à jour les valeurs de configuration qu’il contient pour refléter le point de terminaison et la clé de votre ressource de *prédiction* Custom Vision, l’ID de projet de votre projet de classification et le nom de votre modèle publié (qui doit être *fruit-classifier*). Enregistrez vos modifications.
4. Ouvrez le fichier de code de votre application cliente (*Program.cs* pour C#, *test-classification.py* pour Python) et examinez le code qu’il contient, en notant les détails suivants :
    - Les espaces de noms du package que vous avez installé sont importés
    - La fonction **Main** récupère les paramètres de configuration et utilise la clé et le point de terminaison pour créer un **CustomVisionPredictionClient** authentifié.
    - L’objet client de prédiction est utilisé pour prédire une classe pour chaque image dans le dossier **test-images**, en spécifiant l’ID de projet et le nom du modèle pour chaque requête. Chaque prédiction inclut une probabilité pour chaque classe possible, et seules les balises prédites ayant une probabilité supérieure à 50 % sont affichées.
5. Revenez au terminal intégré pour accéder au dossier **test-classifier**, puis entrez la commande suivante (en fonction du SDK) pour exécuter le programme :

**C#**

```
dotnet run
```

**Python**

```
python test-classifier.py
```

6. Affichez l’étiquette (balise) et les scores de probabilité pour chaque prédiction. Vous pouvez afficher les images dans le dossier **test-images** pour vérifier que le modèle les a classées correctement.

## Plus d’informations

Pour plus d’informations sur la classification d’images avec le service Custom Vision, consultez la [documentation Custom Vision](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/).
