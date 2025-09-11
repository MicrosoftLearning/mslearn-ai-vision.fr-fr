---
lab:
  title: Détecter et analyser les visages
  description: "Utilisez le service Visage Azure\_AI\_Vision pour mettre en œuvre des solutions de détection et d’analyse des visages."
---

# Détecter et analyser les visages

La possibilité de détecter et d’analyser les visages humains est une fonctionnalité fondamentale de l’IA. Dans cet exercice, vous allez explorer le service **Visage** pour travailler avec des visages.

> **Remarque** : Cet exercice est basé sur une version préliminaire du logiciel SDK, qui est susceptible d'être modifiée. Le cas échéant, nous avons utilisé des versions spécifiques de certains packages, qui ne correspondent pas forcément aux versions les plus récentes disponibles. Il se peut que vous rencontriez des comportements, inattendus, des avertissements ou des erreurs.

Bien que cet exercice repose sur le kit de développement logiciel (SDK) Azure Vision Face Python, vous pouvez développer des applications de vision à l’aide de plusieurs kits SDK spécifiques à un langage, notamment :

* [Visage Azure AI Vision pour JavaScript](https://www.npmjs.com/package/@azure-rest/ai-vision-face)
* [Azure AI Vision Face for Microsoft .NET](https://www.nuget.org/packages/Azure.AI.Vision.Face)
* [Azure AI Vision Face for Java](https://central.sonatype.com/artifact/com.azure/azure-ai-vision-face)

Cet exercice prend environ **30** minutes.

> **Remarque** : les fonctionnalités des Azure AI services qui renvoient des informations d’identification personnelle sont réservées aux clients bénéficiant d’un [accès limité](https://learn.microsoft.com/legal/cognitive-services/computer-vision/limited-access-identity). Cet exercice n’inclut pas les tâches de reconnaissance faciale et peut être effectué sans demander d’accès supplémentaire aux fonctionnalités restreintes.

## Approvisionner une ressource d’API Azure AI Face

Si vous n’en avez pas déjà une dans votre abonnement, vous devrez approvisionner une ressource d’API Azure AI Face.

> **Remarque** : dans cet exercice, vous allez utiliser une ressource autonome **Visage**. Vous pouvez également utiliser les services Azure AI Face dans une ressource multiservice *Azure AI Services*, soit directement, soit dans un projet *Azure AI Foundry*.

1. Ouvrez le [portail Azure](https://portal.azure.com) sur `https://portal.azure.com` et connectez-vous à l’aide de vos informations d’identification Azure. Fermez tous les messages de bienvenue ou conseils qui s’affichent.
1. Sélectionnez **Créer une ressource**.
1. Dans la barre de recherche, recherchez `Face`, sélectionnez **Visage**, et créez la ressource avec les paramètres suivants :
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *créez ou sélectionnez un groupe de ressources*
    - **Région** : *choisissez n’importe quelle région disponible*
    - **Nom** : *nom valide de votre ressource Visage*
    - **Niveau tarifaire** : F0 gratuit

1. Créez la ressource et attendez que le déploiement soit terminé, puis affichez les détails du déploiement.
1. Une fois la ressource déployée, accédez-y et sous le nœud **Gestion des ressources** dans le volet de navigation, affichez sa page **Clés et point de terminaison**. Vous aurez besoin du point de terminaison et de l’une des clés de cette page dans la procédure suivante.

## Développer une application d’analyse faciale avec le kit de développement logiciel (SDK) de Visage

Dans cet exercice, vous allez développer une application cliente partiellement mise en œuvre qui utilise le kit de développement logiciel (SDK) Azure Face pour détecter et analyser les visages humains dans les images.

### Préparer la configuration de l’application

1. Dans le portail Azure, utilisez le bouton **[\>_]** situé à droite de la barre de recherche en haut de la page pour créer un nouveau Cloud Shell dans le portail Azure, en sélectionnant un environnement ***PowerShell*** sans stockage dans votre abonnement.

    Cloud Shell fournit une interface de ligne de commande via un volet situé en bas du portail Azure.

    > **Remarque** : si vous avez déjà créé un Cloud Shell qui utilise un environnement *Bash*, basculez-le vers ***PowerShell***.

    > **Remarque** : si le portail sollicite la sélection d’un stockage pour la persistance des fichiers, choisissez **Aucun compte de stockage requis**, sélectionnez l’abonnement concerné et appuyez sur **Appliquer**.

1. Dans la barre d’outils Cloud Shell, dans le menu **Paramètres**, sélectionnez **Accéder à la version classique** (cela est nécessaire pour utiliser l’éditeur de code).

    **<font color="red">Assurez-vous d’avoir basculé vers la version classique du Cloud Shell avant de continuer.</font>**

1. Redimensionnez le volet Cloud Shell afin de pouvoir continuer à voir la page **Clés et point de terminaison** de votre ressource Visage.

    > **Conseil** : vous pouvez redimensionner le volet en faisant glisser la bordure supérieure. Vous pouvez également utiliser les boutons de réduction et d’agrandissement pour alterner entre le Cloud Shell et l’interface principale du portail.

1. Dans le volet Cloud Shell, entrez les commandes suivantes pour cloner le référentiel GitHub contenant les fichiers de code de cet exercice (saisissez la commande, ou copiez-la vers le presse-papiers et cliquez avec le bouton droit dans la ligne de commande pour la coller sous forme de texte brut) :

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **Conseil** : lorsque vous collez des commandes dans Cloud Shell, la sortie peut prendre une grande quantité de mémoire tampon d’écran. Vous pouvez effacer le contenu de l’écran en saisissant la commande `cls` pour faciliter le focus sur chaque tâche.

1. Une fois le référentiel cloné, utilisez la commande suivante pour accéder aux fichiers de code de l’application :

    ```
   cd mslearn-ai-vision/Labfiles/face/python/face-api
   ls -a -l
    ```

    Le dossier contient les fichiers de code et de configuration de votre application. Il contient également un sous-dossier **/images**, qui contient des fichiers image que votre application doit analyser.

1. Installez le package du kit de développement logiciel (SDK) Azure AI Vision et d’autres packages requis en exécutant les commandes suivantes :

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-vision-face==1.0.0b2
    ```

1. Entrez la commande suivante pour modifier le fichier de configuration de votre application :

    ```
   code .env
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Dans le fichier de code, mettez à jour les valeurs de configuration de sorte qu’il correspondent au **point de terminaison** et une **clé** d’authentification pour votre ressource Visage (copiée à partir de sa page **Clés et point de terminaison** dans le portail Azure).
1. Une fois que vous avez remplacé les espaces réservés, utilisez la commande **Ctrl+S** pour enregistrer vos modifications, puis utilisez la commande **Ctrl+Q** pour fermer l’éditeur de code tout en gardant la ligne de commande Cloud Shell ouverte.

### Ajouter du code pour créer un client d’API Visage

1. Dans la ligne de commande Cloud Shell, entrez la commande suivante pour ouvrir le fichier de code de l’application cliente :

    ```
   code analyze-faces.py
    ```

    > **Conseil** : vous pouvez agrandir le volet Cloud Shell et déplacer la barre de fractionnement entre la console de ligne de commande et l’éditeur de code afin de voir plus facilement le code.

1. Dans le fichier de code, recherchez le commentaire **Importer les espaces de noms** et ajoutez le code suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit de développement logiciel (SDK) Azure AI Vision :

    ```python
   # Import namespaces
   from azure.ai.vision.face import FaceClient
   from azure.ai.vision.face.models import FaceDetectionModel, FaceRecognitionModel, FaceAttributeTypeDetection01
   from azure.core.credentials import AzureKeyCredential
    ```

1. Dans la fonction **Main**, notez que le code permettant de charger les paramètres de configuration et de déterminer l’image à analyser a été fourni. Recherchez ensuite le commentaire **Authentifier le client Visage** et ajoutez le code suivant pour créer et authentifier un objet **FaceClient** :

    ```python
   # Authenticate Face client
   face_client = FaceClient(
        endpoint=cog_endpoint,
        credential=AzureKeyCredential(cog_key))
    ```

### Ajouter du code pour détecter et analyser des visages

1. Dans le fichier de code de votre application, dans la fonction **Main**, recherchez le commentaire **Spécifier les caractéristiques faciales à récupérer** et ajoutez le code suivant :

    ```python
   # Specify facial features to be retrieved
   features = [FaceAttributeTypeDetection01.HEAD_POSE,
                FaceAttributeTypeDetection01.OCCLUSION,
                FaceAttributeTypeDetection01.ACCESSORIES]
    ```

1. Dans la fonction **Main**, sous le code que vous venez d’ajouter, recherchez le commentaire **Obtenir les visages** et ajoutez le code suivant pour imprimer les informations sur les caractéristiques faciales et appeler une fonction qui annote l’image avec le cadre englobant pour chaque visage détecté (en fonction de la propriété **face_rectangle** de chaque visage) :

    ```Python
   # Get faces
   with open(image_file, mode="rb") as image_data:
        detected_faces = face_client.detect(
            image_content=image_data.read(),
            detection_model=FaceDetectionModel.DETECTION01,
            recognition_model=FaceRecognitionModel.RECOGNITION01,
            return_face_id=False,
            return_face_attributes=features,
        )

   face_count = 0
   if len(detected_faces) > 0:
        print(len(detected_faces), 'faces detected.')
        for face in detected_faces:
    
            # Get face properties
            face_count += 1
            print('\nFace number {}'.format(face_count))
            print(' - Head Pose (Yaw): {}'.format(face.face_attributes.head_pose.yaw))
            print(' - Head Pose (Pitch): {}'.format(face.face_attributes.head_pose.pitch))
            print(' - Head Pose (Roll): {}'.format(face.face_attributes.head_pose.roll))
            print(' - Forehead occluded?: {}'.format(face.face_attributes.occlusion["foreheadOccluded"]))
            print(' - Eye occluded?: {}'.format(face.face_attributes.occlusion["eyeOccluded"]))
            print(' - Mouth occluded?: {}'.format(face.face_attributes.occlusion["mouthOccluded"]))
            print(' - Accessories:')
            for accessory in face.face_attributes.accessories:
                print('   - {}'.format(accessory.type))
            # Annotate faces in the image
            annotate_faces(image_file, detected_faces)
    ```

1. Examinez le code que vous avez ajouté à la fonction **Main**. Il analyse un fichier image et détecte tous les visages qu’il contient, notamment les attributs relatifs à la position de la tête, à l’occlusion et à la présence d’accessoires tels que des lunettes. De plus, une fonction est appelée pour annoter l’image d’origine avec un cadre englobant pour chaque visage détecté.
1. Enregistrez vos modifications (*CTRL+S*), mais laissez l’éditeur de code ouvert au cas où vous devez corriger les fautes de frappe.

1. Redimensionnez les volets pour afficher davantage de console, puis entrez la commande suivante pour exécuter le programme avec l’argument *images/face1.jpg* :

    ```
   python analyze-faces.py images/face1.jpg
    ```

    L’application exécute et analyse l’image suivante :

    ![Photographie d’une statue représentant une personne.](../media/face1.jpg)

1. Observez le résultat, qui devrait inclure l’ID et les attributs de chaque visage détecté. 
1. Notez qu’un fichier image nommé **detected_faces.jpg** est également généré. Utilisez la commande **téléchargement** (spécifique à Azure Cloud Shell) pour le télécharger :

    ```
   download detected_faces.jpg
    ```

    La commande de téléchargement crée un lien contextuel en bas à droite de votre navigateur, que vous pouvez sélectionner pour télécharger et ouvrir le fichier. L’image doit ressembler à ce qui suit :

    ![Image avec le visage mis en évidence.](../media/detected_faces1.jpg)

1. Exécutez à nouveau le programme, en spécifiant cette fois le paramètre *images/face2.jpg* pour extraire le texte de l’image suivante :

    ![Image d’une autre personne.](../media/face2.jpg)

    ```
   python analyze-faces.py images/face2.jpg
    ```

1. Téléchargez et affichez le fichier **detected_faces.jpg** obtenu :

    ```
   download detected_faces.jpg
    ```

    L’image obtenue devrait ressembler à ceci :

    ![Une autre image avec un visage mis en évidence.](../media/detected_faces2.jpg)

1. Exécutez à nouveau le programme, en spécifiant cette fois le paramètre *images/faces.jpg* pour extraire le texte de cette image :

    ![Photo des deux personnes.](../media/faces.jpg)

    ```
   python analyze-faces.py images/faces.jpg
    ```

1. Téléchargez et affichez le fichier **detected_faces.jpg** obtenu :

    ```
   download detected_faces.jpg
    ```

    L’image obtenue devrait ressembler à ceci :

    ![Une autre image avec un visage mis en évidence.](../media/detected_faces3.jpg)

## Nettoyer les ressources

Si vous avez terminé d’explorer Azure AI Vision, vous devez supprimer les ressources que vous avez créées dans cet exercice pour éviter des coûts Azure inutiles :

1. Ouvrez le Portail Azure à l'adresse `https://portal.azure.com` et, dans la barre de recherche supérieure, recherchez les ressources que vous avez créées dans ce labo.

1. Dans la page de ressources, sélectionnez **Supprimer** et suivez les instructions pour supprimer la ressource. Vous pouvez également supprimer l’intégralité du groupe de ressources pour nettoyer toutes les ressources en même temps.
