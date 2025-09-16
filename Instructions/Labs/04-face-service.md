---
lab:
  title: Détecter et analyser les visages
  module: Module 4 - Detecting and Analyze Faces
---

# Détecter et analyser les visages

La possibilité de détecter et d’analyser les visages humains est une fonctionnalité fondamentale de l’IA. Dans cet exercice, vous allez découvrir deux services Azure AI Services que vous pouvez utiliser pour travailler avec les visages dans des images : le service **Azure AI Vision** et le service **Visage**.

> **Important** : Ce labo peut être terminé sans demander d’accès supplémentaire aux fonctionnalités restreintes.

> **Remarque** : depuis le 21 juin 2022, les fonctionnalités d’Azure AI services qui retournent des informations d’identification personnelle sont limitées aux clients qui ont reçu [un accès limité](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access). En outre, les fonctionnalités qui déduisent l’état émotionnel ne sont plus disponibles. Pour plus d’informations sur les modifications apportées par Microsoft, et leur motif, consultez [Responsible AI investments and safeguards for facial recognition](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/) (Investissements responsables en matière d'IA et mesures de protection pour la reconnaissance faciale).

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
1. Attendez la fin du déploiement, puis visualisez les détails du déploiement.
1. Une fois la ressource déployée, accédez-y et affichez sa page **Clés et point de terminaison**. Vous aurez besoin du point de terminaison et de l’une des clés de cette page dans la procédure suivante.

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
   cd mslearn-ai-vision/Labfiles/04-face
    ```

## Se préparer à l’utilisation du kit de développement logiciel (SDK) d’Azure AI Vision

Dans cet exercice, vous allez effectuer une application cliente partiellement implémentée qui utilise le kit de développement logiciel (SDK) d’Azure AI Vision pour analyser des images.

> **Remarque** : Vous pouvez choisir d’utiliser le kit de développement logiciel (SDK) pour **C#** ou **Python**. Dans les étapes qui suivent, effectuez les actions appropriées pour votre langage préféré.

1. Accédez au dossier qui contient les fichiers de code de l’application pour votre langue d’interface par défaut de l’utilisateur :  

    **C#**

    ```
   cd C-Sharp/computer-vision
    ```
    
    **Python**

    ```
   cd Python/computer-vision
    ```

1. Installez le package du kit de développement logiciel (SDK) Azure AI Vision et les dépendances requises en exécutant les commandes appropriées pour votre langue choisie :

    **C#**
    
    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0
    dotnet add package SkiaSharp --version 3.116.1
    dotnet add package SkiaSharp.NativeAssets.Linux --version 3.116.1
    ``` 

    **Python**
    
    ```
    pip install azure-ai-vision-imageanalysis==1.0.0
    pip install dotenv
    pip install matplotlib
    ```

1. À l’aide de la commande `ls`, vous pouvez afficher le contenu du dossier **computer-vision**. Notez qu’il contient un fichier pour les paramètres de configuration :

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

1. Mettez à jour les valeurs de configuration dans le fichier de code en renseignant le **point de terminaison** et une **clé** d’authentification de votre ressource de Vision par ordinateur.
1. Une fois que vous avez remplacé les espaces réservés, utilisez la commande **Ctrl+S** pour enregistrer vos modifications, puis utilisez la commande **Ctrl+Q** pour fermer l’éditeur de code tout en gardant la ligne de commande Cloud Shell ouverte.

## Afficher l’image que vous allez analyser

Dans cet exercice, vous allez utiliser le service Azure AI Vision pour analyser une image avec des personnes.

1. Dans la barre d’outils de Cloud Shell, sélectionnez **Charger/Télécharger des fichiers** puis **Télécharger**. Dans la nouvelle boîte de dialogue, saisissez le chemin d’accès au fichier suivant, puis sélectionnez **Télécharger** :

    ```
    mslearn-ai-vision/Labfiles/04-face/C-Sharp/computer-vision/images/people.jpg
    ```

1. Afin de l’afficher, ouvrez l’image **people.jpg**.

## Détecter des visages dans une image

Vous êtes maintenant prêt à utiliser le kit de développement logiciel (SDK) pour appeler le service Vision et détecter les visages dans une image.

1. Notez que le dossier **computer-vision** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : detect-people.py

1. Entrez la commande suivante pour ouvrir le fichier de code de l’application cliente :

    **C#**

    ```
   code Program.cs
    ```

    **Python**

    ```
   code image-analysis.py
    ```

1. Sous le commentaire **Importer les espaces de noms**, ajoutez le code suivant, spécifique à une langue, afin d’importer les espaces de noms dont vous aurez besoin pour utiliser le kit de développement logiciel (SDK) Azure AI Vision :

    **C#**
    
    ```C#
    // Import namespaces
    using Azure.AI.Vision.ImageAnalysis;
    using SkiaSharp;
    ```
    
    **Python**
    
    ```Python
    # import namespaces
    from azure.ai.vision.imageanalysis import ImageAnalysisClient
    from azure.ai.vision.imageanalysis.models import VisualFeatures
    from azure.core.credentials import AzureKeyCredential
    ```
    
1. Dans la fonction **Main**, notez que le code pour charger les paramètres de configuration a été fourni. Recherchez ensuite le commentaire **Authentifier le client Azure AI Vision**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour créer et authentifier un objet client Azure AI Vision :

    **C#**

    ```C#
    // Authenticate Azure AI Vision client
    ImageAnalysisClient cvClient = new ImageAnalysisClient(
        new Uri(aiSvcEndpoint),
        new AzureKeyCredential(aiSvcKey));
    ```

    **Python**

    ```Python
    # Authenticate Azure AI Vision client
    cv_client = ImageAnalysisClient(
        endpoint=ai_endpoint,
        credential=AzureKeyCredential(ai_key)
    )
    ```

1. Dans la fonction **Main**, sous le code que vous venez d’ajouter, notez que le code spécifie le chemin d’accès à un fichier image, puis le transmet à une fonction nommée **AnalyzeImage**. Cette fonction n’est pas encore entièrement implémentée.

1. Dans la fonction **AnalyzeImage**, sous le commentaire **Obtenir un résultat en spécifiant les fonctionnalités à récupérer (PEOPLE)**, ajoutez le code suivant :

    **C#**

    ```C#
    // Get result with specified features to be retrieved (PEOPLE)
    ImageAnalysisResult result = client.Analyze(
        BinaryData.FromStream(stream),
        VisualFeatures.People);
    ```

    **Python**

    ```Python
    # Get result with specified features to be retrieved (PEOPLE)
    result = cv_client.analyze(
        image_data=image_data,
        visual_features=[
            VisualFeatures.PEOPLE],
    )
    ```

1. Dans la fonction **AnalyzeImage**, sous le commentaire **Dessiner un cadre englobant autour des personnes détectées**, ajoutez le code suivant :

    **C#**

    ```C
    // Draw bounding box around detected people
    foreach (DetectedPerson person in result.People.Values)
    {
        if (person.Confidence > 0.5) 
        {
            // Draw object bounding box
            var r = person.BoundingBox;
            SKRect rect = new SKRect(r.X, r.Y, r.X + r.Width, r.Y + r.Height);
            canvas.DrawRect(rect, paint);
        }

        // Return the confidence of the person detected
        //Console.WriteLine($"   Bounding box {person.BoundingBox.ToString()}, Confidence: {person.Confidence:F2}");
    }
    ```

    **Python**
    
    ```Python
    # Draw bounding box around detected people
    for detected_people in result.people.list:
        if(detected_people.confidence > 0.5):
            # Draw object bounding box
            r = detected_people.bounding_box
            bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height))
            draw.rectangle(bounding_box, outline=color, width=3)

        # Return the confidence of the person detected
        #print(" {} (confidence: {:.2f}%)".format(detected_people.bounding_box, detected_people.confidence * 100))
    ```

1. Enregistrez vos modifications, fermez l’éditeur de code, puis saisissez la commande suivante pour exécuter le programme :

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python detect-people.py
    ```

6. Observez la sortie, qui doit indiquer le nombre de visages détectés.
7. Téléchargez le fichier **people.jpg** généré dans le même dossier que votre fichier de code pour visualiser les visages annotés. Dans ce cas, votre code a utilisé les attributs du visage pour définir la position de l’angle supérieur gauche du cadre ainsi que les coordonnées du cadre englobant pour dessiner un rectangle autour de chaque visage.

Si vous souhaitez voir le score de confiance de toutes les personnes détectées par le service, vous pouvez supprimer les marques de commentaire de la ligne de code sous le commentaire `Return the confidence of the person detected` et réexécuter le code.

## Préparer l’utilisation du Kit de développement logiciel (SDK) Visage

Bien que le service **AZure AI Vision** offre une détection des visages de base (ainsi que de nombreuses autres fonctionnalités d’analyse d’image), le service **Visage** apporte des fonctionnalités plus complètes pour l’analyse et la reconnaissance faciales.

1. Accédez au dossier **face-api** en exécutant la commande `cd ../face-api`. Installez ensuite le package SDK Visage en exécutant la commande appropriée pour votre préférence de langage :

    **C#**

    ```
    dotnet add package Azure.AI.Vision.Face -v 1.0.0-beta.2
    ```

    **Python**

    ```
    pip install azure-ai-vision-face==1.0.0b2
    ```
    
1. Affichez le contenu du dossier **face-api**, et notez qu’il contient un fichier pour les paramètres de configuration :
    - **C#** : appsettings.json
    - **Python** : .env

1. Ouvrez le fichier de configuration et mettez à jour les valeurs de configuration qu’il contient pour refléter le **point de terminaison** et une **clé** d’authentification pour votre ressource Azure AI services. Enregistrez vos modifications.

1. Notez que le dossier **face-api** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : analyze-faces.py

1. Ouvrez le fichier de code et, en haut, sous les références d’espace de noms existantes, recherchez le commentaire **Importer des espaces de noms**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit de développement logiciel (SDK) de Vision :

    **C#**

    ```C#
    // Import namespaces
    using Azure;
    using Azure.AI.Vision.Face;
    using SkiaSharp;
    ```

    **Python**

    ```Python
    # Import namespaces
    from azure.ai.vision.face import FaceClient
    from azure.ai.vision.face.models import FaceDetectionModel, FaceRecognitionModel, FaceAttributeTypeDetection03
    from azure.core.credentials import AzureKeyCredential
    ```

1. Dans la fonction **Main**, notez que le code pour charger les paramètres de configuration a été fourni. Recherchez ensuite le commentaire **Authentifier le client Visage**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour créer et authentifier un objet **FaceClient** :

    **C#**

    ```C#
    // Authenticate Face client
    faceClient = new FaceClient(
        new Uri(cogSvcEndpoint),
        new AzureKeyCredential(cogSvcKey));
    ```

    **Python**

    ```Python
    # Authenticate Face client
    face_client = FaceClient(
        endpoint=cog_endpoint,
        credential=AzureKeyCredential(cog_key)
    )
    ```

1. Dans la fonction **Main**, sous le code que vous venez d’ajouter, notez que le code affiche un menu qui vous permet d’appeler des fonctions dans votre code pour explorer les fonctionnalités du service Visage. Vous allez implémenter ces fonctions dans le reste de cet exercice.

## Détecter et analyser les visages

L’une des fonctionnalités les plus fondamentales du service Visage consiste à détecter les visages dans une image et à déterminer leurs attributs, tels que la posture de la tête, le niveau de flou, la présence d’un masque, etc.

1. Dans le fichier de code de votre application, dans la fonction **Main**, examinez le code qui s’exécute si l’utilisateur sélectionne l’option de menu **1**. Ce code appelle la fonction **DetectFaces**, en passant le chemin vers un fichier image.
1. Recherchez la fonction **DetectFaces** dans le fichier de code et, sous le commentaire **Spécifier les fonctionnalités faciales à récupérer**, ajoutez le code suivant :

    **C#**

    ```C#
    // Specify facial features to be retrieved
    FaceAttributeType[] features = new FaceAttributeType[]
    {
        FaceAttributeType.Detection03.HeadPose,
        FaceAttributeType.Detection03.Blur,
        FaceAttributeType.Detection03.Mask
    };
    ```

    **Python**

    ```Python
    # Specify facial features to be retrieved
    features = [FaceAttributeTypeDetection03.HEAD_POSE,
                FaceAttributeTypeDetection03.BLUR,
                FaceAttributeTypeDetection03.MASK]
    ```

1. Dans la fonction **DetectFaces**, sous le code que vous venez d’ajouter, recherchez le commentaire **Obtenir des visages** et ajoutez le code suivant :

    **C#**

    ```C#
    // Get faces
    using (var imageData = File.OpenRead(imageFile))
    {    
        var response = await faceClient.DetectAsync(
            BinaryData.FromStream(imageData),
            FaceDetectionModel.Detection03,
            FaceRecognitionModel.Recognition04,
            returnFaceId: false,
            returnFaceAttributes: features);
        IReadOnlyList<FaceDetectionResult> detected_faces = response.Value;

        if (detected_faces.Count() > 0)
        {
            Console.WriteLine($"{detected_faces.Count()} faces detected.");

            // Load the image using SkiaSharp
            using SKBitmap bitmap = SKBitmap.Decode(imageFile);
            using SKCanvas canvas = new SKCanvas(bitmap);

            // Set up paint styles for drawing
            SKPaint rectPaint = new SKPaint
            {
                Color = SKColors.LightGreen,
                StrokeWidth = 3,
                Style = SKPaintStyle.Stroke,
                IsAntialias = true
            };

            SKPaint textPaint = new SKPaint
            {
                Color = SKColors.White,
                TextSize = 16,
                IsAntialias = true
            };

            int faceCount=0;

            // Draw and annotate each face
            foreach (var face in detected_faces)
            {
                faceCount++;
                Console.WriteLine($"\nFace number {faceCount}");
            
                // Get face properties
                Console.WriteLine($" - Head Pose (Yaw): {face.FaceAttributes.HeadPose.Yaw}");
                Console.WriteLine($" - Head Pose (Pitch): {face.FaceAttributes.HeadPose.Pitch}");
                Console.WriteLine($" - Head Pose (Roll): {face.FaceAttributes.HeadPose.Roll}");
                Console.WriteLine($" - Blur: {face.FaceAttributes.Blur.BlurLevel}");
                Console.WriteLine($" - Mask: {face.FaceAttributes.Mask.Type}");

                // Draw and annotate face
                var r = face.FaceRectangle;

                // Create an SKRect from the face rectangle data
                SKRect rect = new SKRect(r.Left, r.Top, r.Left + r.Width, r.Top + r.Height);
                canvas.DrawRect(rect, rectPaint);

                string annotation = $"Face number {faceCount}";
                canvas.DrawText(annotation, r.Left, r.Top, textPaint);
            }

            // Save annotated image
            using (SKFileWStream output = new SKFileWStream("detected_faces.jpg"))
            {
                bitmap.Encode(output, SKEncodedImageFormat.Jpeg, 100);
            }

            Console.WriteLine(" Results saved in detected_faces.jpg");   
        }
    }
    ```

    **Python**

    ```Python
    # Get faces
    with open(image_file, mode="rb") as image_data:
        detected_faces = face_client.detect(
            image_content=image_data.read(),
            detection_model=FaceDetectionModel.DETECTION03,
            recognition_model=FaceRecognitionModel.RECOGNITION04,
            return_face_id=False,
            return_face_attributes=features,
        )

        if len(detected_faces) > 0:
            print(len(detected_faces), 'faces detected.')

            # Prepare image for drawing
            fig = plt.figure(figsize=(8, 6))
            plt.axis('off')
            image = Image.open(image_file)
            draw = ImageDraw.Draw(image)
            color = 'lightgreen'
            face_count = 0

            # Draw and annotate each face
            for face in detected_faces:
    
                # Get face properties
                face_count += 1
                print('\nFace number {}'.format(face_count))

                print(' - Head Pose (Yaw): {}'.format(face.face_attributes.head_pose.yaw))
                print(' - Head Pose (Pitch): {}'.format(face.face_attributes.head_pose.pitch))
                print(' - Head Pose (Roll): {}'.format(face.face_attributes.head_pose.roll))
                print(' - Blur: {}'.format(face.face_attributes.blur.blur_level))
                print(' - Mask: {}'.format(face.face_attributes.mask.type))

                # Draw and annotate face
                r = face.face_rectangle
                bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
                draw = ImageDraw.Draw(image)
                draw.rectangle(bounding_box, outline=color, width=5)
                annotation = 'Face number {}'.format(face_count)
                plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

            # Save annotated image
            plt.imshow(image)
            outputfile = 'detected_faces.jpg'
            fig.savefig(outputfile)

            print('\nResults saved in', outputfile)
    ```

1. Examinez le code que vous avez ajouté à la fonction **DetectFaces**. Il analyse un fichier image et détecte les visages qu’il contient, y compris les attributs relatifs à la posture de la tête, le niveau de flou et la présence d’un masque. Les détails de chaque visage sont affichés, notamment un identificateur de visage unique affecté à chaque visage. L'emplacement des visages est indiqué sur l’image à l’aide de rectangles englobants.
1. Enregistrez vos modifications et fermez l’éditeur de code, puis saisissez la commande suivante pour exécuter le programme :

    **C#**

    ```
    dotnet run
    ```

    *La sortie C# peut afficher des avertissements sur les fonctions asynchrones n’utilisant pas l’opérateur **await**. Vous pouvez les ignorer.*

    **Python**

    ```
    python analyze-faces.py
    ```

1. Lorsque vous y êtes invité, entrez **1** et observez la sortie, qui doit inclure l’ID et les attributs de chaque visage détecté.
1. Téléchargez le fichier **detected_faces.jpg** généré dans le même dossier que votre fichier de code pour visualiser les visages annotés.

## Nettoyer les ressources

Si vous n’utilisez pas les ressources Azure créées dans ce labo pour d’autres modules de formation, vous pouvez les supprimer pour éviter d’autres frais.

1. Ouvrez le Portail Azure à l'adresse `https://portal.azure.com` et, dans la barre de recherche supérieure, recherchez les ressources que vous avez créées dans ce labo.

1. Dans la page de ressources, sélectionnez **Supprimer** et suivez les instructions pour supprimer la ressource. Vous pouvez également supprimer l’intégralité du groupe de ressources pour nettoyer toutes les ressources en même temps.

## Plus d’informations

Il existe plusieurs fonctionnalités supplémentaires disponibles dans le service **Visage**. Toutefois, conformément à la [Norme de l’IA responsable](https://aka.ms/aah91ff), celles-ci sont limitées par une stratégie d’accès limité. Ces fonctionnalités incluent l’identification, la vérification et la création de modèles de reconnaissance faciale. Pour en savoir plus et demander l’accès, consultez la section[Accès limité pour Azure AI Services](https://docs.microsoft.com/en-us/azure/cognitive-services/cognitive-services-limited-access).

Pour plus d’informations sur l’utilisation du service **Azure AI Vision** pour la détection des visages, consultez la [documentation d’Azure AI Vision](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-detecting-faces).

Pour en savoir plus sur le service **Visage**, consultez la [documentation Visage](https://learn.microsoft.com/azure/ai-services/computer-vision/overview-identity).
