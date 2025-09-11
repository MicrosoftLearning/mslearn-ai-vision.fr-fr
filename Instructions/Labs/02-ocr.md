---
lab:
  title: Lisez le texte contenu dans des images
  description: "Utilisez la reconnaissance optique de caractères (OCR) dans le service d’analyse d’images Azure\_AI\_Vision pour rechercher et extraire du texte dans des images."
---

# Lisez le texte contenu dans des images

La reconnaissance optique de caractères (OCR) est un sous-ensemble de la vision par ordinateur qui traite de la lecture de texte dans des images et des documents. Le service d’analyse d’images **Azure AI Vision** fournit une API permettant de lire du texte, que vous allez explorer dans cet exercice.

> **Remarque** : Cet exercice est basé sur une version préliminaire du logiciel SDK, qui est susceptible d'être modifiée. Le cas échéant, nous avons utilisé des versions spécifiques de certains packages, qui ne correspondent pas forcément aux versions les plus récentes disponibles. Il se peut que vous rencontriez des comportements, inattendus, des avertissements ou des erreurs.

Cet exercice repose sur le kit de développement logiciel (SDK) Python pour l’analyse Azure Vision, mais il est possible de développer des applications de vision à l’aide de plusieurs kits SDK spécifiques à différentes langues, notamment :

* [Analyse Azure AI Vision pour JavaScript](https://www.npmjs.com/package/@azure-rest/ai-vision-image-analysis)
* [Analyse Azure AI Vision pour Microsoft .NET](https://www.nuget.org/packages/Azure.AI.Vision.ImageAnalysis)
* [Analyse Azure AI Vision pour Java](https://mvnrepository.com/artifact/com.azure/azure-ai-vision-imageanalysis)

Cet exercice prend environ **30** minutes.

## Approvisionnez une ressource Azure AI Vision

Vous aurez besoin d’approvisionner une ressource Azure AI Vision si vous n’en avez pas déjà une dans votre abonnement.

> **Remarque** : Dans cet exercice, vous allez utiliser une ressource **Vision par ordinateur** autonome. Vous pouvez également utiliser les services Azure AI Vision au sein d’une ressource multi-service *Azure AI Services*, soit directement, soit dans un projet *Azure AI Foundry*.

1. Ouvrez le [portail Azure](https://portal.azure.com) sur `https://portal.azure.com` et connectez-vous à l’aide de vos informations d’identification Azure. Fermez tous les messages de bienvenue ou conseils qui s’affichent.
1. Sélectionnez **Créer une ressource**.
1. Dans la barre de recherche, recherchez `Computer Vision`, sélectionnez **Vision par ordinateur**, et créez la ressource avec les paramètres suivants :
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *créez ou sélectionnez un groupe de ressources*
    - **Région** : *Choisissez parmi **USA Est**, **USA Ouest**, **France Centre**, **Corée Centre**, **Europe Nord**, **Asie Sud-Est**, **Europe Ouest**, ou **Asie Est**\**
    - **Nom** : *Nom valide de votre ressource Vision par ordinateur*
    - **Niveau tarifaire** : F0 gratuit

    \*Les fonctionnalités complètes d’Azure AI Vision 4.0 sont actuellement disponibles uniquement dans ces régions.

1. Cochez les cases nécessaires et créez la ressource.
1. Attendez la fin du déploiement, puis visualisez les détails du déploiement.
1. Une fois la ressource déployée, accédez-y et sous le nœud **Gestion des ressources** dans le volet de navigation, affichez sa page **Clés et point de terminaison**. Vous aurez besoin du point de terminaison et de l’une des clés de cette page dans la procédure suivante.

## Développer une application d’extraction de texte avec le kit de développement logiciel (SDK) Azure AI Vision

Dans cet exercice, vous allez développer une application cliente partiellement mise en œuvre qui utilise le kit de développement logiciel (SDK) Azure AI Vision pour extraire des textes à partir des images.

### Préparer la configuration de l’application

1. Dans le portail Azure, utilisez le bouton **[\>_]** situé à droite de la barre de recherche en haut de la page pour créer un nouveau Cloud Shell dans le portail Azure, en sélectionnant un environnement ***PowerShell*** sans stockage dans votre abonnement.

    Cloud Shell fournit une interface de ligne de commande via un volet situé en bas du portail Azure.

    > **Remarque** : si vous avez déjà créé un Cloud Shell qui utilise un environnement *Bash*, basculez-le vers ***PowerShell***.

    > **Remarque** : si le portail sollicite la sélection d’un stockage pour la persistance des fichiers, choisissez **Aucun compte de stockage requis**, sélectionnez l’abonnement concerné et appuyez sur **Appliquer**.

1. Dans la barre d’outils Cloud Shell, dans le menu **Paramètres**, sélectionnez **Accéder à la version classique** (cela est nécessaire pour utiliser l’éditeur de code).

    **<font color="red">Assurez-vous d’avoir basculé vers la version classique du Cloud Shell avant de continuer.</font>**

1. Redimensionnez le volet Cloud Shell pour que vous puissiez toujours voir la page **Clés et point de terminaison** de votre ressource Vision par ordinateur.

    > **Conseil** : vous pouvez redimensionner le volet en faisant glisser la bordure supérieure. Vous pouvez également utiliser les boutons de réduction et d’agrandissement pour alterner entre le Cloud Shell et l’interface principale du portail.

1. Dans le volet Cloud Shell, entrez les commandes suivantes pour cloner le référentiel GitHub contenant les fichiers de code de cet exercice (saisissez la commande, ou copiez-la vers le presse-papiers et cliquez avec le bouton droit dans la ligne de commande pour la coller sous forme de texte brut) :

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **Conseil** : lorsque vous collez des commandes dans Cloud Shell, la sortie peut prendre une grande quantité de mémoire tampon d’écran. Vous pouvez effacer le contenu de l’écran en saisissant la commande `cls` pour faciliter le focus sur chaque tâche.

1. Une fois le référentiel cloné, utilisez la commande suivante pour accéder aux fichiers de code de l’application :

    ```
   cd mslearn-ai-vision/Labfiles/ocr/python/read-text
   ls -a -l
    ```

    Le dossier contient les fichiers de code et de configuration de votre application. Il contient également un sous-dossier **/images**, qui contient des fichiers image que votre application doit analyser.

1. Installez le package du kit de développement logiciel (SDK) Azure AI Vision et d’autres packages requis en exécutant les commandes suivantes :

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-vision-imageanalysis==1.0.0
    ```

1. Entrez la commande suivante pour modifier le fichier de configuration de votre application :

    ```
   code .env
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Dans le fichier de code, mettez à jour les valeurs de configuration qu’il contient pour y intégrer le **point de terminaison** et une **clé** d’authentification de votre ressource Vision par ordinateur (copiés depuis sa page **Clés et point de terminaison** dans le portail Azure).
1. Une fois que vous avez remplacé les espaces réservés, utilisez la commande **Ctrl+S** pour enregistrer vos modifications, puis utilisez la commande **Ctrl+Q** pour fermer l’éditeur de code tout en gardant la ligne de commande Cloud Shell ouverte.

### Ajouter du code pour lire le texte d’une image

1. Dans la ligne de commande Cloud Shell, entrez la commande suivante pour ouvrir le fichier de code de l’application cliente :

    ```
   code read-text.py
    ```

    > **Conseil** : vous pouvez agrandir le volet Cloud Shell et déplacer la barre de fractionnement entre la console de ligne de commande et l’éditeur de code afin de voir plus facilement le code.

1. Dans le fichier de code, recherchez le commentaire **Importer les espaces de noms** et ajoutez le code suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit de développement logiciel (SDK) Azure AI Vision :

    ```python
   # import namespaces
   from azure.ai.vision.imageanalysis import ImageAnalysisClient
   from azure.ai.vision.imageanalysis.models import VisualFeatures
   from azure.core.credentials import AzureKeyCredential
    ```

1. Dans la fonction **Main**, le code permettant de charger les paramètres de configuration et de déterminer le fichier à analyser a été fourni. Recherchez ensuite le commentaire **Authentifier le client Azure AI Vision** et ajoutez le code spécifique à une langue suivant pour créer et authentifier un objet client d’analyse d’images Azure AI Vision :

    ```python
   # Authenticate Azure AI Vision client
   cv_client = ImageAnalysisClient(
        endpoint=ai_endpoint,
        credential=AzureKeyCredential(ai_key))
    ```

1. Dans la fonction **Main**, sous le code que vous venez d’ajouter, recherchez le commentaire **Lire le texte dans l’image** et ajoutez le code suivant pour utiliser le client d’analyse d’image afin de lire le texte dans l’image :

    ```python
   # Read text in image
   with open(image_file, "rb") as f:
        image_data = f.read()
   print (f"\nReading text in {image_file}")

   result = cv_client.analyze(
        image_data=image_data,
        visual_features=[VisualFeatures.READ])
    ```

1. Recherchez le commentaire **Imprimer le texte** et ajoutez le code suivant (y compris le commentaire final) pour imprimer les lignes de texte trouvées et appeler une fonction pour les annoter dans l’image (à l’aide de la **bounding_polygon** retournée pour chaque ligne de texte) :

    ```python
   # Print the text
   if result.read is not None:
        print("\nText:")
    
        for line in result.read.blocks[0].lines:
            print(f" {line.text}")        
        # Annotate the text in the image
        annotate_lines(image_file, result.read)

        # Find individual words in each line
        
    ```

1. Enregistrez vos modifications (*CTRL+S*), mais laissez l’éditeur de code ouvert au cas où vous devez corriger les fautes de frappe.

1. Redimensionnez les volets pour afficher davantage de console, puis entrez la commande suivante pour exécuter le programme :

    ```
   python read-text.py images/Lincoln.jpg
    ```

1. Le programme lit le texte dans le fichier image spécifié (*images/Lincoln.jpg*), qui ressemble à ceci :

    ![Photographie d’une statue d’Abraham Lincoln.](../media/Lincoln.jpg)

1. Dans le dossier **read-text**, une image **lines.jpg** a été créée. Utilisez la commande **téléchargement** (spécifique à Azure Cloud Shell) pour le télécharger :

    ```
   download lines.jpg
    ```

    La commande de téléchargement crée un lien contextuel en bas à droite de votre navigateur, que vous pouvez sélectionner pour télécharger et ouvrir le fichier. L’image doit ressembler à ce qui suit :

    ![Image avec le texte mis en évidence.](../media/text.jpg)

1. Exécutez à nouveau le programme, en spécifiant cette fois le paramètre *images/Business-card.jpg* pour extraire le texte de l’image suivante :

    ![Image d’une carte de visite scannée.](../media/Business-card.jpg)

    ```
   python read-text.py images/Business-card.jpg
    ```

1. Téléchargez et affichez le fichier de **lines.jpg ** obtenu :

    ```
   download lines.jpg
    ```

1. Exécutez à nouveau le programme, en spécifiant cette fois le paramètre *images/Note.jpg* pour extraire le texte de cette image :

    ![Photographie d’une liste d’achats manuscrites.](../media/Note.jpg)

    ```
   python read-text.py images/Note.jpg
    ```

1. Téléchargez et affichez le fichier de **lines.jpg ** obtenu :

    ```
   download lines.jpg
    ```

### Ajouter le code pour retourner la position des mots individuels

1. Redimensionnez les volets pour voir plus de fichiers de code. Ensuite, recherchez le commentaire **Rechercher des mots individuels dans chaque ligne** et ajoutez le code suivant (assurez-vous que le niveau de retrait reste adéquat) :

    ```python
   # Find individual words in each line
   print ("\nIndividual words:")
   for line in result.read.blocks[0].lines:
        for word in line.words:
            print(f"  {word.text} (Confidence: {word.confidence:.2f}%)")
   # Annotate the words in the image
   annotate_words(image_file, result.read)
    ```

1. Enregistrez vos modifications (*CTRL+S*). Ensuite, dans le volet de ligne de commande, exécutez à nouveau le programme pour extraire le texte de l’image *images/Lincoln.jpg*.
1. Observez le résultat, qui devrait inclure chaque mot individuel de l’image, ainsi que le niveau de confiance associé à leur prédiction.
1. Dans le dossier **read-text**, une image **words.jpg** a été créée. Utilisez la commande **téléchargement** (spécifique à Azure Cloud Shell) pour le télécharger et l’afficher :

    ```
   download words.jpg
    ```

1. Réexécutez le programme pour *images/Business-card.jpg* et *images/Note.jpg* ; affichez le fichier **words.jpg** généré pour chaque image.

## Nettoyer les ressources

Si vous avez terminé d’explorer Azure AI Vision, vous devez supprimer les ressources que vous avez créées dans cet exercice pour éviter des coûts Azure inutiles :

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.

1. Dans la barre de recherche située en haut de la page, recherchez *Vision par ordinateur* et sélectionnez la ressource Vision par ordinateur que vous avez créée dans cette activité.

1. Dans la page de la ressource, sélectionnez **Supprimer** et suivez les instructions pour supprimer la ressource.
