---
lab:
  title: Détecter et analyser les visages
  module: 'Module 10 - Detecting, Analyzing, and Recognizing Faces'
---

# Détecter et analyser les visages

La possibilité de détecter et d’analyser les visages humains est une fonctionnalité fondamentale de l’IA. Dans cet exercice, vous allez découvrir deux services Azure AI Services que vous pouvez utiliser pour travailler avec les visages dans des images : le service **Azure AI Vision** et le service **Visage**.

> **Remarque** : depuis le 21 juin 2022, les fonctionnalités d’Azure AI services qui retournent des informations d’identification personnelle sont limitées aux clients qui ont reçu [un accès limité](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access). En outre, les fonctionnalités qui déduisent l’état émotionnel ne sont plus disponibles. Ces restrictions peuvent affecter cet exercice réalisé en labo. Nous nous efforçons de résoudre ce problème, mais en attendant, vous risquez de rencontrer certaines erreurs lors des étapes ci-dessous, et nous vous prions de bien vouloir nous en excuser. Pour plus d’informations sur les modifications apportées par Microsoft, et leur motif, consultez [Responsible AI investments and safeguards for facial recognition](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/) (Investissements responsables en matière d'IA et mesures de protection pour la reconnaissance faciale).

## Cloner le référentiel pour ce cours

Si vous ne l’avez pas déjà fait, vous devez cloner le référentiel de code pour ce cours :

1. Démarrez Visual Studio Code.
2. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git : Cloner** pour cloner le référentiel `https://github.com/MicrosoftLearning/mslearn-ai-vision` vers un dossier local (peu importe quel dossier).
3. Lorsque le référentiel a été cloné, ouvrez le dossier dans Visual Studio Code.
4. Attendez que des fichiers supplémentaires soient installés pour prendre en charge les projets de code C# dans le référentiel.

    > **Remarque** : Si vous êtes invité à ajouter des ressources requises pour générer et déboguer, sélectionnez **Not Now** (Pas maintenant).

## Provisionner une ressource Azure AI Services

Si vous n’en avez pas encore dans votre abonnement, vous devez configurer une ressource **Azure AI Services**.

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
2. Dans la barre de recherche supérieure, recherchez *Azure AI services*, sélectionnez **Azure AI Services** et créez une ressource de compte multiservices Azure AI services avec les paramètres suivants :
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *Choisissez ou créez un groupe de ressources. (Si vous utilisez un abonnement restreint, vous n’avez peut-être pas l’autorisation de créer un groupe de ressources. Dans ce cas, utilisez le groupe fourni.)*
    - **Région** : *choisissez n’importe quelle région disponible*
    - **Nom** : *Entrez un nom unique.*
    - **Niveau tarifaire** : Standard S0
3. Cochez les cases nécessaires et créez la ressource.
4. Attendez la fin du déploiement, puis visualisez les détails du déploiement.
5. Une fois la ressource déployée, accédez-y et affichez sa page **Clés et point de terminaison**. Vous aurez besoin du point de terminaison et de l’une des clés de cette page dans la procédure suivante.

## Se préparer à l’utilisation du kit de développement logiciel (SDK) d’Azure AI Vision

Dans cet exercice, vous allez effectuer une application cliente partiellement implémentée qui utilise le kit de développement logiciel (SDK) d’Azure AI Vision pour analyser les visages dans une image.

> **Remarque** : Vous pouvez choisir d’utiliser le kit de développement logiciel (SDK) pour **C#** ou **Python**. Dans les étapes qui suivent, effectuez les actions appropriées pour votre langage préféré.

1. Dans Visual Studio Code, dans le volet **Explorateur**, accédez au dossier **04-face** et développez le dossier **C-Sharp** ou **Python** en fonction de votre préférence de langage.
2. Cliquez avec le bouton droit sur le dossier **Vision par ordinateur** et ouvrez un terminal intégré. Installez ensuite le package du kit de développement logiciel (SDK) d’Azure AI Vision en exécutant la commande appropriée pour votre préférence de langage :

    **C#**

    ```
    dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 6.0.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-vision-computervision==0.7.0
    ```
    
3. Affichez le contenu du dossier **computer-vision**, et notez qu’il contient un fichier pour les paramètres de configuration :
    - **C#** : appsettings.json
    - **Python** : .env

4. Ouvrez le fichier de configuration et mettez à jour les valeurs de configuration qu’il contient pour refléter le **point de terminaison** et une **clé** d’authentification pour votre ressource Azure AI services. Enregistrez vos modifications.

5. Notez que le dossier **computer-vision** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : detect-faces.py

6. Ouvrez le fichier de code et, en haut, sous les références d’espace de noms existantes, recherchez le commentaire **Importer des espaces de noms**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit de développement logiciel (SDK) d’Azure AI Vision :

    **C#**

    ```C#
    // import namespaces
    using Microsoft.Azure.CognitiveServices.Vision.ComputerVision;
    using Microsoft.Azure.CognitiveServices.Vision.ComputerVision.Models;
    ```

    **Python**

    ```Python
    # import namespaces
    from azure.cognitiveservices.vision.computervision import ComputerVisionClient
    from azure.cognitiveservices.vision.computervision.models import VisualFeatureTypes
    from msrest.authentication import CognitiveServicesCredentials
    ```

## Afficher l’image que vous allez analyser

Dans cet exercice, vous allez utiliser le service Azure AI Vision pour analyser une image avec des personnes.

1. Dans Visual Studio Code, développez le dossier **computer-vision** et le dossier **images** qu’il contient.
2. Sélectionnez l’image **people.jpg** pour l’afficher.

## Détecter des visages dans une image

Vous êtes maintenant prêt à utiliser le kit de développement logiciel (SDK) pour appeler le service Vision et détecter les visages dans une image.

1. Dans le fichier de code de votre application cliente (**Program.cs** ou **detect-faces.py**), dans la fonction **Main**, notez que le code pour charger les paramètres de configuration a été fourni. Recherchez ensuite le commentaire **Authentifier le client Azure AI Vision**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour créer et authentifier un objet client Azure AI Vision :

    **C#**

    ```C#
    // Authenticate Azure AI Vision client
    ApiKeyServiceClientCredentials credentials = new ApiKeyServiceClientCredentials(cogSvcKey);
    cvClient = new ComputerVisionClient(credentials)
    {
        Endpoint = cogSvcEndpoint
    };
    ```

    **Python**

    ```Python
    # Authenticate Azure AI Vision client
    credential = CognitiveServicesCredentials(cog_key) 
    cv_client = ComputerVisionClient(cog_endpoint, credential)
    ```

2. Dans la fonction **Main**, sous le code que vous venez d’ajouter, notez que le code spécifie le chemin d’accès à un fichier image, puis transmet le chemin d’accès de l’image à une fonction nommée **AnalyzeFaces**. Cette fonction n’est pas encore entièrement implémentée.

3. Dans la fonction **AnalyzeFaces**, sous le commentaire **Spécifier les fonctionnalités à récupérer (visages)** , ajoutez le code suivant :

    **C#**

    ```C#
    // Specify features to be retrieved (faces)
    List<VisualFeatureTypes?> features = new List<VisualFeatureTypes?>()
    {
        VisualFeatureTypes.Faces
    };
    ```

    **Python**

    ```Python
    # Specify features to be retrieved (faces)
    features = [VisualFeatureTypes.faces]
    ```

4. Dans la fonction **AnalyzeFaces**, sous le commentaire **Obtenir l’analyse de l’image**, ajoutez le code suivant :

**C#**

```C
// Get image analysis
using (var imageData = File.OpenRead(imageFile))
{    
    var analysis = await cvClient.AnalyzeImageInStreamAsync(imageData, features);

    // Get faces
    if (analysis.Faces.Count > 0)
    {
        Console.WriteLine($"{analysis.Faces.Count} faces detected.");

        // Prepare image for drawing
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.LightGreen, 3);
        Font font = new Font("Arial", 3);
        SolidBrush brush = new SolidBrush(Color.LightGreen);

        // Draw and annotate each face
        foreach (var face in analysis.Faces)
        {
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
            string annotation = $"Person at approximately {r.Left}, {r.Top}";
            graphics.DrawString(annotation,font,brush,r.Left, r.Top);
        }

        // Save annotated image
        String output_file = "detected_faces.jpg";
        image.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file);   
    }
}        
```

**Python**

```Python
# Get image analysis
with open(image_file, mode="rb") as image_data:
    analysis = cv_client.analyze_image_in_stream(image_data , features)

    # Get faces
    if analysis.faces:
        print(len(analysis.faces), 'faces detected.')

        # Prepare image for drawing
        fig = plt.figure(figsize=(8, 6))
        plt.axis('off')
        image = Image.open(image_file)
        draw = ImageDraw.Draw(image)
        color = 'lightgreen'

        # Draw and annotate each face
        for face in analysis.faces:
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline=color, width=5)
            annotation = 'Person at approximately {}, {}'.format(r.left, r.top)
            plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

        # Save annotated image
        plt.imshow(image)
        outputfile = 'detected_faces.jpg'
        fig.savefig(outputfile)

        print('Results saved in', outputfile)
```

5. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **computer-vision** et entrez la commande suivante pour exécuter le programme :

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python detect-faces.py
    ```

6. Observez la sortie, qui doit indiquer le nombre de visages détectés.
7. Affichez le fichier **detected_faces.jpg** généré dans le même dossier que votre fichier de code pour voir les visages annotés. Dans ce cas, votre code a utilisé les attributs du visage pour définir la position de l’angle supérieur gauche du cadre ainsi que les coordonnées du cadre englobant pour dessiner un rectangle autour de chaque visage.

## Préparer l’utilisation du Kit de développement logiciel (SDK) Visage

Bien que le service **AZure AI Vision** offre une détection des visages de base (ainsi que de nombreuses autres fonctionnalités d’analyse d’image), le service **Visage** apporte des fonctionnalités plus complètes pour l’analyse et la reconnaissance faciales.

1. Dans Visual Studio Code, dans le volet **Explorateur**, accédez au dossier **19-face** et développez le dossier **C-Sharp** ou **Python** en fonction de votre préférence de langage.
2. Cliquez avec le bouton droit sur le dossier **face-api** et ouvrez un terminal intégré. Installez ensuite le package SDK Visage en exécutant la commande appropriée pour votre préférence de langage :

    **C#**

    ```
    dotnet add package Microsoft.Azure.CognitiveServices.Vision.Face --version 2.6.0-preview.1
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-vision-face==0.4.1
    ```
    
3. Affichez le contenu du dossier **face-api**, et notez qu’il contient un fichier pour les paramètres de configuration :
    - **C#** : appsettings.json
    - **Python** : .env

4. Ouvrez le fichier de configuration et mettez à jour les valeurs de configuration qu’il contient pour refléter le **point de terminaison** et une **clé** d’authentification pour votre ressource Azure AI services. Enregistrez vos modifications.

5. Notez que le dossier **face-api** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : analyze-faces.py

6. Ouvrez le fichier de code et, en haut, sous les références d’espace de noms existantes, recherchez le commentaire **Importer des espaces de noms**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit de développement logiciel (SDK) de Vision :

    **C#**

    ```C#
    // Import namespaces
    using Microsoft.Azure.CognitiveServices.Vision.Face;
    using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
    ```

    **Python**

    ```Python
    # Import namespaces
    from azure.cognitiveservices.vision.face import FaceClient
    from azure.cognitiveservices.vision.face.models import FaceAttributeType
    from msrest.authentication import CognitiveServicesCredentials
    ```

7. Dans la fonction **Main**, notez que le code pour charger les paramètres de configuration a été fourni. Recherchez ensuite le commentaire **Authentifier le client Visage**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour créer et authentifier un objet **FaceClient** :

    **C#**

    ```C#
    // Authenticate Face client
    ApiKeyServiceClientCredentials credentials = new ApiKeyServiceClientCredentials(cogSvcKey);
    faceClient = new FaceClient(credentials)
    {
        Endpoint = cogSvcEndpoint
    };
    ```

    **Python**

    ```Python
    # Authenticate Face client
    credentials = CognitiveServicesCredentials(cog_key)
    face_client = FaceClient(cog_endpoint, credentials)
    ```

8. Dans la fonction **Main**, sous le code que vous venez d’ajouter, notez que le code affiche un menu qui vous permet d’appeler des fonctions dans votre code pour explorer les fonctionnalités du service Visage. Vous allez implémenter ces fonctions dans le reste de cet exercice.

## Détecter et analyser les visages

L’une des fonctionnalités les plus fondamentales du service Visage consiste à détecter les visages dans une image et à déterminer leurs attributs, tels que la posture de tête, le niveau de flou, la présence de lunettes, etc.

1. Dans le fichier de code de votre application, dans la fonction **Main**, examinez le code qui s’exécute si l’utilisateur sélectionne l’option de menu **1**. Ce code appelle la fonction **DetectFaces**, en passant le chemin vers un fichier image.
2. Recherchez la fonction **DetectFaces** dans le fichier de code et, sous le commentaire **Spécifier les fonctionnalités faciales à récupérer**, ajoutez le code suivant :

    **C#**

    ```C#
    // Specify facial features to be retrieved
    List<FaceAttributeType?> features = new List<FaceAttributeType?>
    {
        FaceAttributeType.Occlusion,
        FaceAttributeType.Blur,
        FaceAttributeType.Glasses
    };
    ```

    **Python**

    ```Python
    # Specify facial features to be retrieved
    features = [FaceAttributeType.occlusion,
                FaceAttributeType.blur,
                FaceAttributeType.glasses]
    ```

3. Dans la fonction **DetectFaces**, sous le code que vous venez d’ajouter, recherchez le commentaire **Obtenir des visages** et ajoutez le code suivant :

**C#**

```C
// Get faces
using (var imageData = File.OpenRead(imageFile))
{    
    var detected_faces = await faceClient.Face.DetectWithStreamAsync(imageData, returnFaceAttributes: features, returnFaceId: false);

    if (detected_faces.Count > 0)
    {
        Console.WriteLine($"{detected_faces.Count} faces detected.");

        // Prepare image for drawing
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.LightGreen, 3);
        Font font = new Font("Arial", 4);
        SolidBrush brush = new SolidBrush(Color.Black);
        int faceCount=0;

        // Draw and annotate each face
        foreach (var face in detected_faces)
        {
            faceCount++;
            Console.WriteLine($"\nFace number {faceCount}");
            
            // Get face properties
            Console.WriteLine($" - Mouth Occluded: {face.FaceAttributes.Occlusion.MouthOccluded}");
            Console.WriteLine($" - Eye Occluded: {face.FaceAttributes.Occlusion.EyeOccluded}");
            Console.WriteLine($" - Blur: {face.FaceAttributes.Blur.BlurLevel}");
            Console.WriteLine($" - Glasses: {face.FaceAttributes.Glasses}");

            // Draw and annotate face
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
            string annotation = $"Face ID: {face.FaceId}";
            graphics.DrawString(annotation,font,brush,r.Left, r.Top);
        }

        // Save annotated image
        String output_file = "detected_faces.jpg";
        image.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file);   
    }
}
```

**Python**

```Python
# Get faces
with open(image_file, mode="rb") as image_data:
    detected_faces = face_client.face.detect_with_stream(image=image_data,
                                                            return_face_attributes=features,                     return_face_id=False)

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

            detected_attributes = face.face_attributes.as_dict()
            if 'blur' in detected_attributes:
                print(' - Blur:')
                for blur_name in detected_attributes['blur']:
                    print('   - {}: {}'.format(blur_name, detected_attributes['blur'][blur_name]))
                    
            if 'occlusion' in detected_attributes:
                print(' - Occlusion:')
                for occlusion_name in detected_attributes['occlusion']:
                    print('   - {}: {}'.format(occlusion_name, detected_attributes['occlusion'][occlusion_name]))

            if 'glasses' in detected_attributes:
                print(' - Glasses:{}'.format(detected_attributes['glasses']))

            # Draw and annotate face
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline=color, width=5)
            annotation = 'Face ID: {}'.format(face.face_id)
            plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

        # Save annotated image
        plt.imshow(image)
        outputfile = 'detected_faces.jpg'
        fig.savefig(outputfile)

        print('\nResults saved in', outputfile)
```

4. Examinez le code que vous avez ajouté à la fonction **DetectFaces**. Il analyse un fichier image et détecte les visages qu’il contient, y compris les attributs relatifs à la couverture du visage, le niveau de flou et la présence de lunettes. Les détails de chaque visage sont affichés, notamment un identificateur de visage unique affecté à chaque visage. L'emplacement des visages est indiqué sur l’image à l’aide de rectangles englobants.
5. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **face-api** et entrez la commande suivante pour exécuter le programme :

    **C#**

    ```
    dotnet run
    ```

    *La sortie C# peut afficher des avertissements sur les fonctions asynchrones utilisant maintenant l'opérateur **await**. Vous pouvez les ignorer.*

    **Python**

    ```
    python analyze-faces.py
    ```

6. Lorsque vous y êtes invité, entrez **1** et observez la sortie, qui doit inclure l’ID et les attributs de chaque visage détecté.
7. Affichez le fichier **detected_faces.jpg** généré dans le même dossier que votre fichier de code pour voir les visages annotés.

## Plus d’informations

Il existe plusieurs fonctionnalités supplémentaires disponibles dans le service **Visage**. Toutefois, conformément à la [Norme de l’IA responsable](https://aka.ms/aah91ff), celles-ci sont limitées par une stratégie d’accès limité. Ces fonctionnalités incluent l’identification, la vérification et la création de modèles de reconnaissance faciale. Pour en savoir plus et demander l’accès, consultez la section[Accès limité pour Azure AI Services](https://docs.microsoft.com/en-us/azure/cognitive-services/cognitive-services-limited-access).

Pour plus d’informations sur l’utilisation du service **Azure AI Vision** pour la détection des visages, consultez la [documentation d’Azure AI Vision](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-detecting-faces).

Pour en savoir plus sur le service **Visage**, consultez la [documentation Visage](https://docs.microsoft.com/azure/cognitive-services/face/).
