---
lab:
  title: Lire le texte contenu dans des images
  module: Module 11 - Reading Text in Images and Documents
---

# Lire le texte contenu dans des images

La reconnaissance optique de caractères (OCR) est un sous-ensemble de la vision par ordinateur qui traite de la lecture de texte dans des images et des documents. Le service **Azure AI Vision** fournit deux API pour la lecture de texte, que vous allez découvrir dans cet exercice.

## Cloner le référentiel pour cette formation

Si vous ne l’avez pas encore fait, vous devez cloner le référentiel de code pour ce cours :

1. Démarrez Visual Studio Code.
2. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git : Cloner** pour cloner le référentiel `https://github.com/MicrosoftLearning/mslearn-ai-vision` vers un dossier local (peu importe quel dossier).
3. Lorsque le référentiel a été cloné, ouvrez le dossier dans Visual Studio Code.
4. Attendez que des fichiers supplémentaires soient installés pour prendre en charge les projets de code C# dans le référentiel.

    > **Remarque** : si vous êtes invité à ajouter des ressources requises pour générer et déboguer, sélectionnez **Not Now** (Pas maintenant). Si un message vous signale qu’*un projet Azure Functions a été détecté dans le dossier*, vous pouvez fermer ce message en toute sécurité.

## Provisionner une ressource Azure AI Services

Si vous n’en avez pas encore dans votre abonnement, vous devez configurer une ressource **Azure AI Services**.

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
2. Dans la barre de recherche supérieure, recherchez *Azure AI services*, sélectionnez **Azure AI Services** et créez une ressource de compte multiservices Azure AI services avec les paramètres suivants :
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *Choisissez ou créez un groupe de ressources. (Si vous utilisez un abonnement restreint, vous n’avez peut-être pas l’autorisation de créer un groupe de ressources. Dans ce cas, utilisez le groupe fourni.)*
    - **Région** : *vous avez le choix entre USA Est, France Centre, Corée Centre, Europe Nord, Asie Sud-Est, Europe Ouest, USA Ouest ou Asie Est\**
    - **Nom** : *Entrez un nom unique.*
    - **Niveau tarifaire** : Standard S0

    \*Les fonctionnalités d’Azure AI Vision 4.0 sont actuellement disponibles uniquement dans ces régions.

3. Cochez les cases nécessaires et créez la ressource.
4. Attendez la fin du déploiement, puis visualisez les détails du déploiement.
5. Une fois la ressource déployée, accédez-y et affichez sa page **Clés et point de terminaison**. Vous aurez besoin du point de terminaison et de l’une des clés de cette page dans la procédure suivante.

## Se préparer à l’utilisation du kit de développement logiciel (SDK) d’Azure AI Vision

Dans cet exercice, vous allez effectuer une application cliente partiellement implémentée qui utilise le kit de développement logiciel (SDK) d’Azure AI Vision pour lire du texte.

> **Remarque** : Vous pouvez choisir d’utiliser le kit de développement logiciel (SDK) pour **C#** ou **Python**. Dans les étapes qui suivent, effectuez les actions appropriées pour votre langage préféré.

1. Dans Visual Studio Code, dans le volet **Explorer**, accédez au dossier **Labfiles\05-ocr** et développez le dossier **C-Sharp** ou **Python** en fonction de votre préférence de langage.
2. Cliquez avec le bouton droit sur le dossier **create-search**, puis sélectionnez Ouvrir dans le terminal intégré. Installez ensuite le package du kit de développement logiciel (SDK) d’Azure AI Vision en exécutant la commande appropriée pour votre préférence de langage :

    **C#**
    
    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0-beta.3
    ```

    > **Remarque** : si vous êtes invité à installer des extensions du kit de développement, vous pouvez fermer le message en toute sécurité.

    **Python**
    
    ```
    pip install azure-ai-vision-imageanalysis==1.0.0b3
    ```

3. Affichez le contenu du dossier **read-text**, et notez qu’il contient un fichier pour les paramètres de configuration :

    - **C#**  : appsettings.json
    - **Python** : .env

    Ouvrez le fichier de configuration et mettez à jour les valeurs de configuration qu’il contient pour refléter le **point de terminaison** et une **clé** d’authentification pour votre ressource Azure AI services. Enregistrez vos modifications.


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
    ```
    
    **Python**
    
    ```Python
    # import namespaces
    from azure.ai.vision.imageanalysis import ImageAnalysisClient
    from azure.ai.vision.imageanalysis.models import VisualFeatures
    from azure.core.credentials import AzureKeyCredential
    ```

2. Dans le fichier de code de votre application cliente, dans la fonction **Main**, le code pour charger les paramètres de configuration a été fourni. Recherchez ensuite le commentaire **Authentifier le client Azure AI Vision**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour créer et authentifier un objet client Azure AI Vision :

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

3. Dans la fonction **Main**, sous le code que vous venez d’ajouter, notez que le code spécifie le chemin d’accès à un fichier image, puis transmet le chemin d’accès de l’image à la fonction **GetTextRead**. Cette fonction n’est pas encore entièrement implémentée.

4. Ajoutons du code au corps de la fonction **GetTextRead**. Recherchez le commentaire **Utiliser la fonction Analyser l’image pour lire du texte dans l’image**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant, notant que les fonctionnalités visuelles sont spécifiées lors de l’appel à la fonction `Analyze` :

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
    
        // Prepare image for drawing
        System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.Cyan, 3);
        
        foreach (var line in result.Read.Blocks.SelectMany(block => block.Lines))
        {
            // Return the text detected in the image
    
    
        }
            
        // Save image
        String output_file = "text.jpg";
        image.Save(output_file);
        Console.WriteLine("\nResults saved in " + output_file + "\n");   
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

5. Dans le code que vous venez d’ajouter dans la fonction **GetTextRead**, et sous le commentaire **Retourner le texte détecté dans l’image**, ajoutez le code suivant (ce code affiche le texte de l’image sur la console et génère l’image **text.jpg** qui met en évidence le texte de l’image) :

    **C#**
    
    ```C#
    // Return the text detected in the image
    Console.WriteLine($"   '{line.Text}'");
    
    // Draw bounding box around line
    var drawLinePolygon = true;
    
    // Return the position bounding box around each line
    
    
    
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    
    
    
    // Draw line bounding polygon
    if (drawLinePolygon)
    {
        var r = line.BoundingPolygon;
    
        Point[] polygonPoints = {
            new Point(r[0].X, r[0].Y),
            new Point(r[1].X, r[1].Y),
            new Point(r[2].X, r[2].Y),
            new Point(r[3].X, r[3].Y)
        };
    
        graphics.DrawPolygon(pen, polygonPoints);
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

6. Dans le dossier **read-text/images**, sélectionnez **Lincoln.jpg** pour afficher le fichier traité par votre code.

7. Dans le fichier de code de votre application, dans la fonction **Main**, examinez le code qui s’exécute si l’utilisateur sélectionne l’option de menu **1**. Ce code appelle la fonction **GetTextRead** en transmettant le chemin d’accès vers le fichier image *Lincoln.jpg*.

8. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **clock-client** et entrez la commande suivante pour exécuter le programme :

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

9. Lorsque vous y êtes invité, entrez **1** et observez la sortie, qui est le texte extrait de l’image.

10. Dans le dossier **read-text**, sélectionnez l’image **text.jpg** et observez la présence d’un polygone autour de chaque *ligne* de texte.

11. Revenez au fichier de code dans Visual Studio Code et recherchez le commentaire **Renvoyer le cadre englobant de position autour de chaque ligne**. Ensuite, sous ce commentaire, ajoutez le code suivant :

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

12. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **clock-client** et entrez la commande suivante pour exécuter le programme :

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

13. Lorsque vous y êtes invité, entrez **1** et observez la sortie, qui doit être chaque ligne de texte de l’image avec sa position respective dans l’image.


14. Revenez au fichier de code dans Visual Studio Code et recherchez le commentaire **Renvoyer chaque mot détecté dans l’image et le cadre englobant de position autour de chaque mot avec le niveau de confiance de chaque mot**. Ensuite, sous ce commentaire, ajoutez le code suivant :

    **C#**
    
    ```C#
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    foreach (DetectedTextWord word in line.Words)
    {
        Console.WriteLine($"     Word: '{word.Text}', Confidence {word.Confidence:F4}, Bounding Polygon: [{string.Join(" ", word.BoundingPolygon)}]");
        
        // Draw word bounding polygon
        drawLinePolygon = false;
        var r = word.BoundingPolygon;
    
        Point[] polygonPoints = {
            new Point(r[0].X, r[0].Y),
            new Point(r[1].X, r[1].Y),
            new Point(r[2].X, r[2].Y),
            new Point(r[3].X, r[3].Y)
        };
    
        graphics.DrawPolygon(pen, polygonPoints);
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

15. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **clock-client** et entrez la commande suivante pour exécuter le programme :

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

16. Lorsque vous y êtes invité, entrez **1** et observez la sortie, qui doit être chaque mot du texte de l’image avec sa position respective dans l’image. Notez que le niveau de confiance de chaque mot est également renvoyé.

17. Dans le dossier **read-text**, sélectionnez l’image **text.jpg** et observez la présence d’un polygone autour de chaque *mot*.

## Utiliser le kit de développement logiciel (SDK) d’Azure AI Vision pour lire du texte manuscrit dans une image

Dans l’exercice précédent, vous lisez du texte bien défini issu d’une image, mais vous pouvez également lire du texte sur des notes manuscrites ou des documents. La bonne nouvelle est que le **kit de développement logiciel (SDK) d’Azure AI Vision** peut également lire du texte manuscrit avec exactement le même code que celui que vous avez utilisé pour lire du texte bien défini. Nous allons utiliser le même code que pour l’exercice précédent, mais cette fois nous allons utiliser une autre image.

1. Dans le dossier **read-text/images**, sélectionnez **Note.jpg** pour afficher le fichier que votre code va traiter.

2. Dans le fichier de code de votre application, dans la fonction **Main**, examinez le code qui s’exécute si l’utilisateur sélectionne l’option de menu **2**. Ce code appelle la fonction **GetTextRead** en transmettant le chemin d’accès vers le fichier image *Note.jpg*.

3. Depuis le terminal intégré pour le dossier **read-text**, entrez la commande suivante pour exécuter le programme :

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

4. Lorsque vous y êtes invité, entrez **2** et observez la sortie, qui est le texte extrait de l’image de la note.

5. Dans le dossier **read-text**, sélectionnez l’image **text.jpg** et observez la présence d’un polygone autour de chaque *mot* de la note.

## Nettoyer les ressources

Si vous n’utilisez pas les ressources Azure créées dans ce labo pour d’autres modules de formation, vous pouvez les supprimer pour éviter d’autres frais. Voici comment procéder :

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.

2. Dans la barre de recherche supérieure, recherchez *Compte multiservices Azure AI services*, sélectionnez la ressource de compte multiservices Azure AI services que vous avez créée dans ce labo.

3. Dans la page des ressources, sélectionnez **Supprimer** et suivez les instructions pour supprimer la ressource.

## Plus d’informations

Pour plus d’informations sur l’utilisation du service **Azure AI Vision** pour la lecture de texte, consultez la [documentation d’Azure AI Vision](https://learn.microsoft.com/azure/ai-services/computer-vision/concept-ocr).
