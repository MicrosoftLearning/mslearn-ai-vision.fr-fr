---
lab:
  title: Lire le texte contenu dans des images
  module: Module 11 - Reading Text in Images and Documents
---

# Lire le texte contenu dans des images

La reconnaissance optique de caractères (OCR) est un sous-ensemble de la vision par ordinateur qui traite de la lecture de texte dans des images et des documents. Le service **Azure AI Vision** fournit deux API pour la lecture de texte, que vous allez découvrir dans cet exercice.

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
   cd mslearn-ai-vision/Labfiles/05-ocr
    ```

## Se préparer à l’utilisation du kit de développement logiciel (SDK) d’Azure AI Vision

Dans cet exercice, vous allez effectuer une application cliente partiellement implémentée qui utilise le kit de développement logiciel (SDK) d’Azure AI Vision pour lire du texte.

> **Remarque** : Vous pouvez choisir d’utiliser le kit de développement logiciel (SDK) pour **C#** ou **Python**. Dans les étapes qui suivent, effectuez les actions appropriées pour votre langage préféré.

1. Accédez au dossier qui contient les fichiers de code de l’application pour votre langue d’interface par défaut de l’utilisateur :  

    **C#**

    ```
   cd C-Sharp/read-text
    ```
    
    **Python**

    ```
   cd Python/read-text
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
1. Une fois que vous avez remplacé les espaces réservés, utilisez la commande **CTRL+S** ou **Clic droit > Enregistrer** dans l’éditeur de code pour enregistrer vos modifications, puis utilisez la commande **CTRL+Q** ou **Clic droit > Quitter** pour fermer l’éditeur tout en gardant la ligne de commande du Cloud Shell ouverte.

## Utiliser le kit de développement logiciel (SDK) d’Azure AI Vision pour lire du texte dans une image

L’une des fonctionnalités du **kit de développement logiciel (SDK) d’Azure AI Vision** consiste à lire du texte dans une image. Dans cet exercice, vous allez effectuer une application cliente partiellement implémentée qui utilise le kit de développement logiciel (SDK) d’Azure AI Vision pour lire du texte dans une image.

1. Le dossier **read-text** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : read-text.py

    Ouvrez le fichier de code et, en haut, sous les références d’espace de noms existantes, recherchez le commentaire **Importer des espaces de noms**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit de développement logiciel (SDK) d’Azure AI Vision :

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

1. Dans le fichier de code de votre application cliente, dans la fonction **Main**, le code pour charger les paramètres de configuration a été fourni. Recherchez ensuite le commentaire **Authentifier le client Azure AI Vision**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour créer et authentifier un objet client Azure AI Vision :

    **C#**
    
    ```C#
    // Authenticate Azure AI Vision client
    ImageAnalysisClient client = new ImageAnalysisClient(
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

1. Dans la fonction **Main**, sous le code que vous venez d’ajouter, notez que le code spécifie le chemin d’accès à un fichier image, puis transmet le chemin d’accès de l’image à la fonction **GetTextRead**. Cette fonction n’est pas encore entièrement implémentée.

1. Ajoutons du code au corps de la fonction **GetTextRead**. Recherchez le commentaire **Utiliser la fonction Analyser l’image pour lire du texte dans l’image**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant, notant que les fonctionnalités visuelles sont spécifiées lors de l’appel à la fonction `Analyze` :

    **C#**

    ```C#
    // Use Analyze image function to read text in image
    ImageAnalysisResult result = client.Analyze(
        BinaryData.FromStream(stream),
        // Specify the features to be retrieved
        VisualFeatures.Read);
    
    stream.Close();
    
    // Display analysis results
    if (result.Read != null)
    {
        Console.WriteLine($"Text:");
    
        // Load the image using SkiaSharp
        using SKBitmap bitmap = SKBitmap.Decode(imageFile);
        // Create canvas to draw on the bitmap
        using SKCanvas canvas = new SKCanvas(bitmap);

        // Create paint for drawing polygons (bounding boxes)
        SKPaint paint = new SKPaint
        {
            Color = SKColors.Cyan,
            StrokeWidth = 3,
            Style = SKPaintStyle.Stroke,
            IsAntialias = true
        };

        foreach (var line in result.Read.Blocks.SelectMany(block => block.Lines))
        {
            // Return the text detected in the image
    
    
        }
            
        // Save the annotated image using SkiaSharp
        using (SKFileWStream output = new SKFileWStream("text.jpg"))
        {
            // Encode the bitmap into JPEG format with full quality (100)
            bitmap.Encode(output, SKEncodedImageFormat.Jpeg, 100);
        }

        Console.WriteLine("\nResults saved in text.jpg\n");
    }
    ```
    
    **Python**
    
    ```Python
    # Use Analyze image function to read text in image
    result = cv_client.analyze(
        image_data=image_data,
        visual_features=[VisualFeatures.READ]
    )

    # Display the image and overlay it with the extracted text
    if result.read is not None:
        print("\nText:")

        # Prepare image for drawing
        image = Image.open(image_file)
        fig = plt.figure(figsize=(image.width/100, image.height/100))
        plt.axis('off')
        draw = ImageDraw.Draw(image)
        color = 'cyan'

        for line in result.read.blocks[0].lines:
            # Return the text detected in the image

            
        # Save image
        plt.imshow(image)
        plt.tight_layout(pad=0)
        outputfile = 'text.jpg'
        fig.savefig(outputfile)
        print('\n  Results saved in', outputfile)
    ```

1. Dans le code que vous venez d’ajouter dans la fonction **GetTextRead**, et sous le commentaire **Retourner le texte détecté dans l’image**, ajoutez le code suivant (ce code affiche le texte de l’image sur la console et génère l’image **text.jpg** qui met en évidence le texte de l’image) :

    **C#**
    
    ```C#
    // Return the text detected in the image
    Console.WriteLine($"   '{line.Text}'");
    
    // Draw bounding box around line
    bool drawLinePolygon = true;
    
    // Return the position bounding box around each line
    
    
    
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    
    
    
    // Draw line bounding polygon
    if (drawLinePolygon)
    {
        var r = line.BoundingPolygon;
        SKPoint[] polygonPoints = new SKPoint[]
        {
            new SKPoint(r[0].X, r[0].Y),
            new SKPoint(r[1].X, r[1].Y),
            new SKPoint(r[2].X, r[2].Y),
            new SKPoint(r[3].X, r[3].Y)
        };

    DrawPolygon(canvas, polygonPoints, paint);
    }
    ```
    
    **Python**
    
    ```Python
    # Return the text detected in the image
    print(f"  {line.text}")    
    
    drawLinePolygon = True
    
    r = line.bounding_polygon
    bounding_polygon = ((r[0].x, r[0].y),(r[1].x, r[1].y),(r[2].x, r[2].y),(r[3].x, r[3].y))
    
    # Return the position bounding box around each line
    
    
    # Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    
    
    # Draw line bounding polygon
    if drawLinePolygon:
        draw.polygon(bounding_polygon, outline=color, width=3)
    ```

1. Pour le fichier d’application C# uniquement, une fonction d’assistance est toujours nécessaire pour dessiner un polygone. Sous le commentaire **Méthode d’assistance pour dessiner un polygone à partir d’un tableau de SKPoints**, ajoutez le code suivant :

    **C#**
   
    ```C#
    // Helper method to draw a polygon given an array of SKPoints
    static void DrawPolygon(SKCanvas canvas, SKPoint[] points, SKPaint paint)
    {
        if (points == null || points.Length == 0)
            return;

        using (var path = new SKPath())
        {
            path.MoveTo(points[0]);
            for (int i = 1; i < points.Length; i++)
            {
                path.LineTo(points[i]);
            }
            path.Close();
            canvas.DrawPath(path, paint);
        }
    }
    ```

1. Dans la fonction **Main**, examinez le code qui s’exécute si l’utilisateur choisit l’option de menu **1**. Ce code appelle la fonction **GetTextRead** en transmettant le chemin d’accès vers le fichier image *Lincoln.jpg*.

1. Enregistrez vos modifications et fermez l’éditeur de code.

1. Dans la barre d’outils de Cloud Shell, sélectionnez **Charger/Télécharger des fichiers** puis **Télécharger**. Dans la nouvelle boîte de dialogue, saisissez le chemin d’accès au fichier suivant, puis sélectionnez **Télécharger** :

    **C#**
   
    ```
    mslearn-ai-vision/Labfiles/05-ocr/C-Sharp/read-text/images/Lincoln.jpg
    ```

    **Python**

    ```
    mslearn-ai-vision/Labfiles/05-ocr/Python/read-text/images/Lincoln.jpg
    ```
       
1. Ouvrez l’image **Lincoln.jpg** pour l’afficher.

1. Après avoir affiché l’image que votre code traite, saisissez la commande suivante pour exécuter le programme :

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. Lorsque vous y êtes invité, entrez **1** et observez la sortie, qui est le texte extrait de l’image.

1. Dans le dossier **read-text**, une image **text.jpg** a été créée. Vous pouvez la télécharger en utilisant le chemin d'accès de fichier `mslearn-ai-vision/Labfiles/05-ocr/***C-Sharp or Python***/read-text/text.jpg` et constater la présence d’un polygone autour de chaque *ligne* de texte.

1. Ouvrez à nouveau le fichier de code et recherchez le commentaire **Retourner le cadre de position englobant autour de chaque ligne**. Ensuite, sous ce commentaire, ajoutez le code suivant :

    **C#**
    
    ```C#
    // Return the position bounding box around each line
    Console.WriteLine($"   Bounding Polygon: [{string.Join(" ", line.BoundingPolygon)}]");
    ```
    
    **Python**
    
    ```Python
    # Return the position bounding box around each line
    print("   Bounding Polygon: {}".format(bounding_polygon))
    ```

1. Enregistrez vos modifications et fermez l’éditeur de code, puis saisissez la commande suivante pour exécuter le programme :

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. Lorsque vous y êtes invité, entrez **1** et observez la sortie, qui doit être chaque ligne de texte de l’image avec sa position respective dans l’image.

1. Ouvrez à nouveau le fichier de code et trouvez le commentaire **Retourner chaque mot détecté dans l’image et le cadre englobant autour de chaque mot avec le niveau de confiance associé**. Ensuite, sous ce commentaire, ajoutez le code suivant :

    **C#**
    
    ```C#
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    foreach (DetectedTextWord word in line.Words)
    {
        Console.WriteLine($"     Word: '{word.Text}', Confidence {word.Confidence:F4}, Bounding Polygon: [{string.Join(" ", word.BoundingPolygon)}]");
        
        // Draw word bounding polygon
        drawLinePolygon = false;
        var r = word.BoundingPolygon;
    
        // Convert the bounding polygon into an array of SKPoints    
        SKPoint[] polygonPoints = new SKPoint[]
        {
            new SKPoint(r[0].X, r[0].Y),
            new SKPoint(r[1].X, r[1].Y),
            new SKPoint(r[2].X, r[2].Y),
            new SKPoint(r[3].X, r[3].Y)
        };

        // Draw the word polygon on the canvas
        DrawPolygon(canvas, polygonPoints, paint);
    }
    ```
    
    **Python**
    
    ```Python
    # Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    for word in line.words:
        r = word.bounding_polygon
        bounding_polygon = ((r[0].x, r[0].y),(r[1].x, r[1].y),(r[2].x, r[2].y),(r[3].x, r[3].y))
        print(f"    Word: '{word.text}', Bounding Polygon: {bounding_polygon}, Confidence: {word.confidence:.4f}")
    
        # Draw word bounding polygon
        drawLinePolygon = False
        draw.polygon(bounding_polygon, outline=color, width=3)
    ```

1. Enregistrez vos modifications et fermez l’éditeur de code, puis saisissez la commande suivante pour exécuter le programme :

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. Lorsque vous y êtes invité, entrez **1** et observez la sortie, qui doit être chaque mot du texte de l’image avec sa position respective dans l’image. Notez que le niveau de confiance de chaque mot est également renvoyé.

1. Téléchargez à nouveau l’image **text.jpg** et notez qu’un polygone entoure chaque *mot*.

## Utiliser le kit de développement logiciel (SDK) d’Azure AI Vision pour lire du texte manuscrit dans une image

Dans l’exercice précédent, vous lisez du texte bien défini issu d’une image, mais vous pouvez également lire du texte sur des notes manuscrites ou des documents. La bonne nouvelle est que le **kit de développement logiciel (SDK) d’Azure AI Vision** peut également lire du texte manuscrit avec exactement le même code que celui que vous avez utilisé pour lire du texte bien défini. Nous allons utiliser le même code que pour l’exercice précédent, mais cette fois nous allons utiliser une autre image.

1. Téléchargez **Note.jpg** en utilisant le chemin d’accès `mslearn-ai-vision/Labfiles/05-ocr/***C-Sharp or Python***/read-text/images/Note.jpg` pour afficher l’image suivante traitée par votre code.

1. Dans le fichier de code de votre application, dans la fonction **Main**, examinez le code qui s’exécute si l’utilisateur sélectionne l’option de menu **2**. Ce code appelle la fonction **GetTextRead** en transmettant le chemin d’accès vers le fichier image *Note.jpg*.

1. Dans le terminal, saisissez la commande ci-dessous pour exécuter le programme :

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. Lorsque vous y êtes invité, entrez **2** et observez la sortie, qui est le texte extrait de l’image de la note.

1. Téléchargez à nouveau l’image **text.jpg** et notez qu’un polygone entoure chaque *mot* de la note.

## Nettoyer les ressources

Si vous n’utilisez pas les ressources Azure créées dans ce labo pour d’autres modules de formation, vous pouvez les supprimer pour éviter d’autres frais. Voici comment procéder :

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.

1. Dans la barre de recherche en haut, recherchez *Vision par ordinateur*, et sélectionnez la ressource Vision par ordinateur que vous avez créée lors de cette activité.

1. Dans la page de la ressource, sélectionnez **Supprimer** et suivez les instructions pour supprimer la ressource.

## Plus d’informations

Pour plus d’informations sur l’utilisation du service **Azure AI Vision** pour la lecture de texte, consultez la [documentation d’Azure AI Vision](https://learn.microsoft.com/azure/ai-services/computer-vision/concept-ocr).
