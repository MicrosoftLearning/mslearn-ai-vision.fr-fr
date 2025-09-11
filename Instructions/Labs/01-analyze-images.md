---
lab:
  title: Analyser les images
  description: "Utilisez Azure\_AI\_Vision\_Image\_Analysis pour analyser des images, suggérer des légendes et des balises, et détecter des objets et des personnes."
---

# Analyser les images

Azure AI Vision est une fonctionnalité d’intelligence artificielle qui permet aux systèmes logiciels d’interpréter les entrées visuelles en analysant les images. Dans Microsoft Azure, le service **Azur AI Vision** fournit des modèles prédéfinis pour les tâches courantes de vision par ordinateur, notamment l’analyse des images pour suggérer des légendes et des balises, la détection d’objets courants et de personnes. Vous pouvez également utiliser le service Azure AI Vision pour supprimer l’arrière-plan ou créer un détourage d’avant-plan des images.

> **Remarque** : Cet exercice est basé sur une version préliminaire du logiciel SDK, qui est susceptible d'être modifiée. Le cas échéant, nous avons utilisé des versions spécifiques de certains packages, qui ne correspondent pas forcément aux versions les plus récentes disponibles. Il se peut que vous rencontriez des comportements, inattendus, des avertissements ou des erreurs.

Cet exercice repose sur le kit de développement logiciel (SDK) Python pour Azure Vision, mais il est possible de développer des applications de vision à l’aide de plusieurs kits de développement logiciel (SDK) spécifiques à différentes langues, notamment :

* [Analyse Azure AI Vision pour JavaScript](https://www.npmjs.com/package/@azure-rest/ai-vision-image-analysis)
* [Analyse Azure AI Vision pour Microsoft .NET](https://www.nuget.org/packages/Azure.AI.Vision.ImageAnalysis)
* [Analyse Azure AI Vision pour Java](https://mvnrepository.com/artifact/com.azure/azure-ai-vision-imageanalysis)

Cet exercice prend environ **30** minutes.

## Approvisionnez une ressource Azure AI Vision

Vous aurez besoin d’approvisionner une ressource Azure AI Vision si vous n’en avez pas déjà une dans votre abonnement.

> **Remarque** : dans cet exercice, vous allez utiliser une ressource **Vision par ordinateur** autonome. Vous pouvez également utiliser les services Azure AI Vision au sein d’une ressource multi-service *Azure AI Services*, soit directement, soit dans un projet *Azure AI Foundry*.

1. Ouvrez le [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com`, puis connectez-vous avec vos informations d’identification Azure. Fermez tous les messages de bienvenue ou conseils qui s’affichent.
1. Sélectionnez **Créer une ressource**.
1. Dans la barre de recherche, recherchez `Computer Vision`, sélectionnez **Vision par ordinateur**, et créez la ressource avec les paramètres suivants :
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *créez ou sélectionnez un groupe de ressources*
    - **Région** : *choisissez parmi **USA Est**, **USA Ouest**, **France Centre**, **Corée Centre**, **Europe Nord**, **Asie Sud-Est**, **Europe Ouest**, ou **Asie Est**\**
    - **Nom** : *nom valide de votre ressource Vision par ordinateur*
    - **Niveau tarifaire** : F0 gratuit

    \*Les fonctionnalités complètes d’Azure AI Vision 4.0 sont actuellement disponibles uniquement dans ces régions.

1. Cochez les cases nécessaires et créez la ressource.
1. Attendez la fin du déploiement, puis visualisez les détails du déploiement.
1. Après le déploiement de la ressource, accédez-y. Dans le volet de navigation, sous le nœud **Gestion des ressources**, affichez la page **Clés et point de terminaison**. Vous aurez besoin du point de terminaison et de l’une des clés de cette page dans la procédure suivante.

## Développez une application d’analyse d’image avec le kit de développement logiciel (SDK) Azure AI Vision.

Dans cet exercice, vous allez effectuer une application cliente partiellement implémentée qui utilise le kit de développement logiciel (SDK) d’Azure AI Vision pour analyser des images.

### Préparer la configuration de l’application

1. Dans le portail Azure, utilisez le bouton **[\>_]**, situé à droite de la barre de recherche en haut de la page, pour créer un nouveau Cloud Shell dans le portail Azure, en sélectionnant un environnement ***PowerShell*** sans stockage dans votre abonnement.

    Cloud Shell fournit une interface de ligne de commande via un volet situé en bas du portail Azure.

    > **Remarque** : si vous avez déjà créé un Cloud Shell qui utilise un environnement *Bash*, basculez-le vers ***PowerShell***.

    > **Remarque** : si le portail sollicite la sélection d’un stockage pour la persistance des fichiers, sélectionnez **Aucun compte de stockage requis**, choisissez l’abonnement concerné et validez avec **Appliquer**.

1. Dans la barre d’outils Cloud Shell, dans le menu **Paramètres**, sélectionnez **Accéder à la version classique** (cela est nécessaire pour utiliser l’éditeur de code).

    **<font color="red">Assurez-vous d’avoir basculé vers la version classique du Cloud Shell avant de continuer.</font>**

1. Redimensionnez le volet du cloud shell de manière à toujours visualiser la page **Clés et point de terminaison** de votre ressource Vision par ordinateur.

    > **Conseil** : vous pouvez redimensionner le volet en faisant glisser la bordure supérieure. Vous pouvez également utiliser les boutons de réduction et d’agrandissement pour alterner entre le Cloud Shell et l’interface principale du portail.

1. Dans le volet Cloud Shell, entrez les commandes suivantes pour cloner le référentiel GitHub contenant les fichiers de code de cet exercice (saisissez la commande, ou copiez-la vers le presse-papiers et cliquez avec le bouton droit dans la ligne de commande pour la coller sous forme de texte brut) :

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **Conseil** : lorsque vous collez des commandes dans Cloud Shell, la sortie peut prendre une grande quantité de mémoire tampon d’écran. Vous pouvez effacer le contenu de l’écran en saisissant la commande `cls` pour faciliter le focus sur chaque tâche.

1. Après le clonage du référentiel, utilisez la commande suivante pour naviguer jusqu’au dossier contenant les fichiers de code de l’application et en afficher le contenu :   

    ```
   cd mslearn-ai-vision/Labfiles/analyze-images/python/image-analysis
   ls -a -l
    ```

    Le dossier contient les fichiers de code et de configuration de votre application. Il contient également un sous-dossier **/images** qui renferme quelques fichiers image destinés à l’analyse par votre application.
    
1. Installez le package du kit de développement logiciel (SDK) Azure AI Vision ainsi que les autres packages requis en exécutant les commandes suivantes :

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-vision-imageanalysis==1.0.0
    ```

1. Saisissez la commande suivante pour modifier le fichier de configuration de votre application :

    ```
   code .env
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Dans le fichier de code, mettez à jour les valeurs de configuration qu’il contient pour y intégrer le **point de terminaison** et la **clé** d’authentification de votre ressource Vision par ordinateur (copiés depuis sa page **Clés et point de terminaison** dans le portail Azure).
1. Une fois que vous avez remplacé les espaces réservés, utilisez la commande **Ctrl+S** pour enregistrer vos modifications, puis utilisez la commande **Ctrl+Q** pour fermer l’éditeur de code tout en gardant la ligne de commande Cloud Shell ouverte.

### Ajouter du code pour suggérer une légende

1. Dans la ligne de commande cloud shell, saisissez la commande suivante pour ouvrir le fichier de code de l’application cliente :

    ```
   code image-analysis.py
    ```

    > **Conseil** : vous pouvez agrandir le volet cloud shell et déplacer la barre de fractionnement entre la console de ligne de commande et l’éditeur de code afin de voir plus facilement le code.

1. Dans le fichier de code, recherchez le commentaire **Importer les espaces de noms** et ajoutez le code suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit de développement logiciel (SDK) Azure AI Vision :

    ```python
   # import namespaces
   from azure.ai.vision.imageanalysis import ImageAnalysisClient
   from azure.ai.vision.imageanalysis.models import VisualFeatures
   from azure.core.credentials import AzureKeyCredential
    ```

1. Dans la fonction **Main**, notez que le code permettant de charger les paramètres de configuration et de déterminer le fichier image à analyser est déjà fourni. Ensuite, trouvez le commentaire **Authentifier le client Azure AI Vision** et ajoutez le code suivant pour créer et authentifier un objet client Azure AI Vision (veillez à respecter les niveaux de retrait appropriés).

    ```python
   # Authenticate Azure AI Vision client
   cv_client = ImageAnalysisClient(
        endpoint=ai_endpoint,
        credential=AzureKeyCredential(ai_key))
    ```

1. Dans la fonction **Main**, sous le code que vous venez d’ajouter, recherchez le commentaire **Analyser l’image** et saisissez le code suivant :

    ```python
   # Analyze image
   with open(image_file, "rb") as f:
        image_data = f.read()
   print(f'\nAnalyzing {image_file}\n')

   result = cv_client.analyze(
        image_data=image_data,
        visual_features=[
            VisualFeatures.CAPTION,
            VisualFeatures.DENSE_CAPTIONS,
            VisualFeatures.TAGS,
            VisualFeatures.OBJECTS,
            VisualFeatures.PEOPLE],
   )
    ```

1. Recherchez le commentaire **Voir les légendes de l’image**, ajoutez le code suivant pour afficher les légendes standard et détaillées de l’image :

    ```python
   # Get image captions
   if result.caption is not None:
        print("\nCaption:")
        print(" Caption: '{}' (confidence: {:.2f}%)".format(result.caption.text, result.caption.confidence * 100))
    
   if result.dense_captions is not None:
        print("\nDense Captions:")
        for caption in result.dense_captions.list:
            print(" Caption: '{}' (confidence: {:.2f}%)".format(caption.text, caption.confidence * 100))
    ```

1. Enregistrez vos modifications (*CTRL+S*) et redimensionnez les volets afin de voir clairement la console de ligne de commande tout en gardant l’éditeur de code ouvert. Puis, saisissez la commande suivante pour exécuter le programme avec l’argument **images/street.jpg** :

    ```
   python image-analysis.py images/street.jpg
    ```

1. Observez la sortie, qui doit inclure une légende suggérée pour l’image **street.jpg**, similaire à celle-ci :

    ![Image d’une rue animée.](../media/street.jpg)

1. Réexécutez le programme, cette fois-ci avec l’argument **images/building.jpg**, afin de voir la légende générée pour l’image **building.jpg**, qui se présente comme suit :

    ![Image d’un bâtiment.](../media/building.jpg)

1. Répétez l’étape précédente pour générer une légende pour le fichier **images/person.jpg**, qui s’affiche comme suit :

    ![Image d’une personne.](../media/person.jpg)

### Ajouter du code pour générer des balises suggérées

Il peut parfois être utile d’identifier des *balises* pertinentes qui fournissent des indices sur le contenu d’une image.

1. Dans l’éditeur de code, à l’intérieur de la fonction **AnalyzeImage**, trouvez le commentaire **Voir les balises d’image** et ajoutez le code suivant :

    ```python
   # Get image tags
   if result.tags is not None:
        print("\nTags:")
        for tag in result.tags.list:
            print(" Tag: '{}' (confidence: {:.2f}%)".format(tag.name, tag.confidence * 100))
    ```

1. Enregistrez vos modifications (*CTRL+S*) et exécutez le programme avec l’argument **images/street.jpg**. Vous allez constater qu’en plus de la légende d’image, une liste de balises suggérées est affichée.
1. Réexécutez le programme pour les fichiers **images/building.jpg** et **images/person.jpg**.

### Ajouter du code pour détecter et localiser des objets

1. Dans l’éditeur de code, dans la fonction **AnalyzeImage**, recherchez le commentaire **Voir les objets dans l’image** et ajoutez le code suivant pour répertorier les objets détectés dans l’image, puis appelez la fonction fournie pour annoter l’image avec ces objets détectés :

    ```python
   # Get objects in the image
   if result.objects is not None:
        print("\nObjects in image:")
        for detected_object in result.objects.list:
            # Print object tag and confidence
            print(" {} (confidence: {:.2f}%)".format(detected_object.tags[0].name, detected_object.tags[0].confidence * 100))
        # Annotate objects in the image
        show_objects(image_file, result.objects.list)
    ```

1. Enregistrez vos modifications (*CTRL+S*) et exécutez le programme avec l’argument **images/street.jpg**. Vous allez constater qu’en plus de la légende et des balises, un fichier nommé **objects.jpg** est généré.
1. Utilisez la commande spécifique à Azure Cloud Shell **télécharger** pour télécharger le fichier **objects.jpg** :

    ```
   download objects.jpg
    ```

    La commande de téléchargement crée un lien contextuel en bas à droite de votre navigateur, que vous pouvez sélectionner pour télécharger et ouvrir le fichier. L’image devrait ressembler à ceci :

    ![une image avec des cadres englobants d’objets.](../media/objects.jpg)

1. Réexécutez le programme pour les fichiers **images/building.jpg** et **images/person.jpg**, en téléchargeant le fichier objects.jpg généré après chaque exécution.

### Ajouter du code pour détecter et localiser des personnes

1. Depuis l’éditeur de code, dans la fonction **AnalyzeImage**, recherchez le commentaire **Voir les personnes dans l’image** et ajoutez le code suivant pour répertorier les personnes détectées avec un niveau de confiance d’au moins 20 %, et appelez une fonction fournie pour annoter ces personnes dans une image :

    ```Python
   # Get people in the image
   if result.people is not None:
        print("\nPeople in image:")

        for detected_person in result.people.list:
            if detected_person.confidence > 0.2:
                # Print location and confidence of each person detected
                print(" {} (confidence: {:.2f}%)".format(detected_person.bounding_box, detected_person.confidence * 100))
        # Annotate people in the image
        show_people(image_file, result.people.list)
    ```

1. Enregistrez vos modifications (*CTRL+S*) et exécutez le programme avec l’argument **images/street.jpg**. Vous allez constater qu’en plus de la légende, des balises suggérées et du fichier objects.jpg, une liste des emplacements de personnes et un fichier nommé **people.jpg** sont générés.

1. Utilisez la commande spécifique à Azure Cloud Shell **télécharger** pour télécharger le fichier **objects.jpg** :

    ```
   download people.jpg
    ```

    La commande de téléchargement crée un lien contextuel en bas à droite de votre navigateur, que vous pouvez sélectionner pour télécharger et ouvrir le fichier. L’image devrait ressembler à ceci :

    ![une image avec des cadres englobants pour les personnes détectées.](../media/people.jpg)

1. Réexécutez le programme pour les fichiers **images/building.jpg** et **images/person.jpg**, en téléchargeant le fichier people.jpg généré après chaque exécution.

   > **Conseil :** si les cadres englobants retournés par le modèle vous semblent incohérents, vérifiez le score de confiance JSON et essayez d’augmenter le filtrage par score de confiance dans votre application.

## Nettoyer les ressources

Si vous avez terminé d’explorer Azure AI Vision, vous devez supprimer les ressources que vous avez créées dans cet exercice pour éviter d’entraîner des coûts Azure inutiles :

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.

1. Dans la barre de recherche en haut, recherchez *Vision par ordinateur*, et sélectionnez la ressource Vision par ordinateur que vous avez créée lors de cette activité.

1. Dans la page de la ressource, sélectionnez **Supprimer** et suivez les instructions pour supprimer la ressource.

