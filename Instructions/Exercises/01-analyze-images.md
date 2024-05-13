---
lab:
  title: Analyser des images avec Azure AI Vision
  module: Module 2 - Develop computer vision solutions with Azure AI Vision
---

# Analyser des images avec Azure AI Vision

Azure AI Vision est une fonctionnalité d’intelligence artificielle qui permet aux systèmes logiciels d’interpréter les entrées visuelles en analysant les images. Dans Microsoft Azure, le service **Azur AI Vision** fournit des modèles prédéfinis pour les tâches courantes de vision par ordinateur, notamment l’analyse des images pour suggérer des légendes et des balises, la détection d’objets courants et de personnes. Vous pouvez également utiliser le service Azure AI Vision pour supprimer l’arrière-plan ou créer un détourage d’avant-plan des images.

## Cloner le référentiel pour cette formation

Si vous n’avez pas déjà cloné le référentiel de code **Azure AI Vision** dans l’environnement où vous travaillez sur ce labo, procédez comme suit. Sinon, ouvrez le dossier cloné dans Visual Studio Code.

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

Dans cet exercice, vous allez effectuer une application cliente partiellement implémentée qui utilise le kit de développement logiciel (SDK) d’Azure AI Vision pour analyser des images.

> **Remarque** : Vous pouvez choisir d’utiliser le kit de développement logiciel (SDK) pour **C#** ou **Python**. Dans les étapes qui suivent, effectuez les actions appropriées pour votre langage préféré.

1. Dans Visual Studio Code, dans le volet **Explorer**, accédez au dossier **Labfiles/01-analyze-images** et développez le dossier **C-Sharp** ou **Python** en fonction de votre préférence de langage.
2. Cliquez avec le bouton droit de la souris sur le dossier **image-analysis** et ouvrez un terminal intégré. Installez ensuite le package du kit de développement logiciel (SDK) d’Azure AI Vision en exécutant la commande appropriée pour votre préférence de langage :

    **C#**
    
    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0-beta.1
    ```

    > **Remarque** : si vous êtes invité à installer des extensions du kit de développement, vous pouvez fermer le message en toute sécurité.

    **Python**
    
    ```
    pip install azure-ai-vision-imageanalysis==1.0.0b1
    ```
    
3. Affichez le contenu du dossier **image-analysis**, et notez qu’il contient un fichier pour les paramètres de configuration :
    - **C#** : appsettings.json
    - **Python** : .env

    Ouvrez le fichier de configuration et mettez à jour les valeurs de configuration qu’il contient pour refléter le **point de terminaison** et une **clé** d’authentification pour votre ressource Azure AI services. Enregistrez vos modifications.
4. Notez que le dossier **image-analysis** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : image-analysis.py

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
    
## Afficher les images que vous allez analyser

Dans cet exercice, vous allez utiliser le service Azure AI Vision pour analyser plusieurs images.

1. Dans Visual Studio Code, développez le dossier **image-analysis** et le dossier **images** qu’il contient.
2. Sélectionnez tour à tour chacun des fichiers image pour les afficher dans Visual Studio Code.

## Analyser une image pour suggérer une légende

Vous êtes maintenant prêt à utiliser le kit de développement logiciel (SDK) pour appeler le service Vision et analyser une image.

1. Dans le fichier de code de votre application cliente (**Program.cs** ou **image-analysis.py**), dans la fonction **Main**, notez que le code pour charger les paramètres de configuration a été fourni. Recherchez ensuite le commentaire **Authentifier le client Azure AI Vision**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour créer et authentifier un objet client Azure AI Vision :

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

2. Dans la fonction **Main**, sous le code que vous venez d’ajouter, notez que le code spécifie le chemin d’accès à un fichier image, puis transmet le chemin d’accès de l’image à deux autres fonctions (**AnalyzeImage** et **BackgroundForeground**). Ces fonctions ne sont pas encore entièrement implémentées.

3. Dans la fonction **AnalyzeImage**, sous le commentaire **Obtenir un résultat en spécifiant les fonctionnalités à récupérer**, ajoutez le code suivant :

**C#**

```C#
// Get result with specified features to be retrieved
ImageAnalysisResult result = client.Analyze(
    BinaryData.FromStream(stream),
    VisualFeatures.Caption | 
    VisualFeatures.DenseCaptions |
    VisualFeatures.Objects |
    VisualFeatures.Tags |
    VisualFeatures.People);
```

**Python**

```Python
# Get result with specified features to be retrieved
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
    
4. Dans la fonction **AnalyzeImage**, sous le commentaire **Afficher l’analyse des résultats**, ajoutez le code suivant (y compris les commentaires indiquant où vous ajouterez du code ultérieurement.) :

**C#**

```C#
// Display analysis results
// Get image captions
if (result.Caption.Text != null)
{
    Console.WriteLine(" Caption:");
    Console.WriteLine($"   \"{result.Caption.Text}\", Confidence {result.Caption.Confidence:0.00}\n");
}

// Get image dense captions
Console.WriteLine(" Dense Captions:");
foreach (DenseCaption denseCaption in result.DenseCaptions.Values)
{
    Console.WriteLine($"   Caption: '{denseCaption.Text}', Confidence: {denseCaption.Confidence:0.00}");
}

// Get image tags


// Get objects in the image


// Get people in the image
```

**Python**

```Python
# Display analysis results
# Get image captions
if result.caption is not None:
    print("\nCaption:")
    print(" Caption: '{}' (confidence: {:.2f}%)".format(result.caption.text, result.caption.confidence * 100))

# Get image dense captions
if result.dense_captions is not None:
    print("\nDense Captions:")
    for caption in result.dense_captions.list:
        print(" Caption: '{}' (confidence: {:.2f}%)".format(caption.text, caption.confidence * 100))

# Get image tags


# Get objects in the image


# Get people in the image

```
    
5. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **image-analysis** et entrez la commande suivante pour exécuter le programme avec l’argument **images/street.jpg** :

**C#**

```
dotnet run images/street.jpg
```

**Python**

```
python image-analysis.py images/street.jpg
```
    
6. Observez la sortie, qui doit inclure une légende suggérée pour l’image **street.jpg**.
7. Réexécutez le programme, cette fois avec l’argument **images/building.jpg** pour voir la légende générée pour l’image **building.jpg**.
8. Répétez l’étape précédente pour générer une légende pour le fichier **images/person.jpg**.

## Obtenir des balises suggérées pour une image

Il peut parfois être utile d’identifier des *balises* pertinentes qui fournissent des indices sur le contenu d’une image.

1. Dans la fonction **AnalyzeImage**, sous le commentaire **Obtenir des balises d’image**, ajoutez le code suivant :

**C#**

```C#
// Get image tags
if (result.Tags.Values.Count > 0)
{
    Console.WriteLine($"\n Tags:");
    foreach (DetectedTag tag in result.Tags.Values)
    {
        Console.WriteLine($"   '{tag.Name}', Confidence: {tag.Confidence:F2}");
    }
}
```

**Python**

```Python
# Get image tags
if result.tags is not None:
    print("\nTags:")
    for tag in result.tags.list:
        print(" Tag: '{}' (confidence: {:.2f}%)".format(tag.name, tag.confidence * 100))
```

2. Enregistrez vos modifications et exécutez le programme une fois pour chacun des fichiers d’images dans le dossier **images**. Observez que, en plus de la légende de l’image, une liste de balises suggérées s’affiche.

## Détecter et localiser des objets dans une image

*La détection d’objets* est une forme spécifique de vision par ordinateur dans laquelle des objets individuels au sein d’une image sont identifiés et leur emplacement indiqué par un cadre englobant.

1. Dans la fonction **AnalyzeImage**, sous le commentaire **Obtenir des objets dans l’image**, ajoutez le code suivant :

**C#**

```C#
// Get objects in the image
if (result.Objects.Values.Count > 0)
{
    Console.WriteLine(" Objects:");

    // Prepare image for drawing
    stream.Close();
    System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.WhiteSmoke);

    foreach (DetectedObject detectedObject in result.Objects.Values)
    {
        Console.WriteLine($"   \"{detectedObject.Tags[0].Name}\"");

        // Draw object bounding box
        var r = detectedObject.BoundingBox;
        Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        graphics.DrawString(detectedObject.Tags[0].Name,font,brush,(float)r.X, (float)r.Y);
    }

    // Save annotated image
    String output_file = "objects.jpg";
    image.Save(output_file);
    Console.WriteLine("  Results saved in " + output_file + "\n");
}
```

**Python**

```Python
# Get objects in the image
if result.objects is not None:
    print("\nObjects in image:")

    # Prepare image for drawing
    image = Image.open(image_filename)
    fig = plt.figure(figsize=(image.width/100, image.height/100))
    plt.axis('off')
    draw = ImageDraw.Draw(image)
    color = 'cyan'

    for detected_object in result.objects.list:
        # Print object name
        print(" {} (confidence: {:.2f}%)".format(detected_object.tags[0].name, detected_object.tags[0].confidence * 100))
        
        # Draw object bounding box
        r = detected_object.bounding_box
        bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height)) 
        draw.rectangle(bounding_box, outline=color, width=3)
        plt.annotate(detected_object.tags[0].name,(r.x, r.y), backgroundcolor=color)

    # Save annotated image
    plt.imshow(image)
    plt.tight_layout(pad=0)
    outputfile = 'objects.jpg'
    fig.savefig(outputfile)
    print('  Results saved in', outputfile)
```

2. Enregistrez vos modifications et exécutez le programme une fois pour chacun des fichiers image dans le dossier **images**, en observant tous les objets détectés. Après chaque exécution, affichez le fichier **objects.jpg** généré dans le même dossier que votre fichier de code pour voir les objets annotés.

## Détecter et localiser des personnes sur une image

*La détection de personnes* est une forme spécifique de vision par ordinateur dans laquelle chaque personne au sein d’une image est identifiée et son emplacement indiqué par un cadre englobant.

1. Dans la fonction **AnalyzeImage**, sous le commentaire **Obtenir des personnes dans l’image**, ajoutez le code suivant :

**C#**

```C#
// Get people in the image
if (result.People.Values.Count > 0)
{
    Console.WriteLine($" People:");

    // Prepare image for drawing
    System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.WhiteSmoke);

    foreach (DetectedPerson person in result.People.Values)
    {
        // Draw object bounding box
        var r = person.BoundingBox;
        Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        
        // Return the confidence of the person detected
        //Console.WriteLine($"   Bounding box {person.BoundingBox.ToString()}, Confidence: {person.Confidence:F2}");
    }

    // Save annotated image
    String output_file = "persons.jpg";
    image.Save(output_file);
    Console.WriteLine("  Results saved in " + output_file + "\n");
}
```

**Python**

```Python
# Get people in the image
if result.people is not None:
    print("\nPeople in image:")

    # Prepare image for drawing
    image = Image.open(image_filename)
    fig = plt.figure(figsize=(image.width/100, image.height/100))
    plt.axis('off')
    draw = ImageDraw.Draw(image)
    color = 'cyan'

    for detected_people in result.people.list:
        # Draw object bounding box
        r = detected_people.bounding_box
        bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height))
        draw.rectangle(bounding_box, outline=color, width=3)

        # Return the confidence of the person detected
        #print(" {} (confidence: {:.2f}%)".format(detected_people.bounding_box, detected_people.confidence * 100))
        
    # Save annotated image
    plt.imshow(image)
    plt.tight_layout(pad=0)
    outputfile = 'people.jpg'
    fig.savefig(outputfile)
    print('  Results saved in', outputfile)
```

2. (Facultatif) Supprimez les commentaire de la commande **Console.Writeline** sous la section **Renvoyer le niveau de confiance de la personne détectée** pour passer en revue le niveau de confiance retourné par une personne détectée à une position particulière de l’image.

3. Enregistrez vos modifications et exécutez le programme une fois pour chacun des fichiers image dans le dossier **images**, en observant tous les objets détectés. Après chaque exécution, affichez le fichier **objects.jpg** généré dans le même dossier que votre fichier de code pour voir les objets annotés.

> **Remarque** : Dans les tâches précédentes, vous avez utilisé une méthode unique pour analyser l’image, puis ajouté de façon incrémentielle du code pour analyser et afficher les résultats. Le Kit de développement logiciel (SDK) fournit également des méthodes individuelles pour suggérer des légendes, identifier des balises, détecter des objets, etc., ce qui signifie que vous pouvez utiliser la méthode la plus appropriée pour retourner uniquement les informations dont vous avez besoin, réduisant ainsi la taille de la charge utile de données qui doit être retournée. Consultez la [documentation du SDK .NET](https://learn.microsoft.com/dotnet/api/overview/azure/cognitiveservices/computervision?view=azure-dotnet) ou la [documentation du SDK Python](https://learn.microsoft.com/python/api/azure-cognitiveservices-vision-computervision/azure.cognitiveservices.vision.computervision) pour plus d’informations.

## Supprimer l’arrière-plan ou générer un détourage d’avant-plan d’une image

Dans certains cas, vous devrez peut-être supprimer l’arrière-plan d’une image ou vous souhaiterez créer un détourage d’avant-plan de cette image. Commençons par la suppression de l’arrière-plan.

1. Dans votre fichier de code, recherchez la fonction **BackgroundForeground** ; ensuite, sous le commentaire **Supprimer l’arrière-plan de l’image ou générer un détourage d’avant-plan**, ajoutez le code suivant :

**C#**

```C#
// Remove the background from the image or generate a foreground matte
Console.WriteLine($" Background removal:");
// Define the API version and mode
string apiVersion = "2023-02-01-preview";
string mode = "backgroundRemoval"; // Can be "foregroundMatting" or "backgroundRemoval"

string url = $"computervision/imageanalysis:segment?api-version={apiVersion}&mode={mode}";

// Make the REST call
using (var client = new HttpClient())
{
    var contentType = new MediaTypeWithQualityHeaderValue("application/json");
    client.BaseAddress = new Uri(endpoint);
    client.DefaultRequestHeaders.Accept.Add(contentType);
    client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", key);

    var data = new
    {
        url = $"https://github.com/MicrosoftLearning/mslearn-ai-vision/blob/main/Labfiles/01-analyze-images/Python/image-analysis/{imageFile}?raw=true"
    };

    var jsonData = JsonSerializer.Serialize(data);
    var contentData = new StringContent(jsonData, Encoding.UTF8, contentType);
    var response = await client.PostAsync(url, contentData);

    if (response.IsSuccessStatusCode) {
        File.WriteAllBytes("background.png", response.Content.ReadAsByteArrayAsync().Result);
        Console.WriteLine("  Results saved in background.png\n");
    }
    else
    {
        Console.WriteLine($"API error: {response.ReasonPhrase} - Check your body url, key, and endpoint.");
    }
}
```

**Python**

```Python
# Remove the background from the image or generate a foreground matte
print('\nRemoving background from image...')
    
url = "{}computervision/imageanalysis:segment?api-version={}&mode={}".format(endpoint, api_version, mode)

headers= {
    "Ocp-Apim-Subscription-Key": key, 
    "Content-Type": "application/json" 
}

image_url="https://github.com/MicrosoftLearning/mslearn-ai-vision/blob/main/Labfiles/01-analyze-images/Python/image-analysis/{}?raw=true".format(image_file)  

body = {
    "url": image_url,
}
    
response = requests.post(url, headers=headers, json=body)

image=response.content
with open("backgroundForeground.png", "wb") as file:
    file.write(image)
print('  Results saved in backgroundForeground.png \n')
```
    
2. Enregistrez vos modifications et exécutez le programme une fois pour chacun des fichiers image dans le dossier **images**. Ouvrez le fichier **background.png** généré dans le même dossier que votre fichier de code pour chaque image.  Observez le résultat de l’arrière-plan qui a été supprimé de chacune des images.

Nous allons maintenant générer un détourage d’avant-plan pour nos images.

3. Dans votre fichier de code, recherchez la fonction **BackgroundForeground** ; puis, sous le commentaire **Définir la version et le mode de l’API**, modifiez la variable de mode comme étant `foregroundMatting`.

4. Enregistrez vos modifications et exécutez le programme une fois pour chacun des fichiers image dans le dossier **images**. Ouvrez le fichier **background.png** généré dans le même dossier que votre fichier de code pour chaque image.  Observez le résultat du détourage d’avant-plan qui a été généré pour vos images.

## Nettoyer les ressources

Si vous n’utilisez pas les ressources Azure créées dans ce labo pour d’autres modules de formation, vous pouvez les supprimer pour éviter d’autres frais. Voici comment procéder :

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.

2. Dans la barre de recherche supérieure, recherchez *Compte multiservices Azure AI services*, sélectionnez la ressource de compte multiservices Azure AI services que vous avez créée dans ce labo.

3. Dans la page des ressources, sélectionnez **Supprimer** et suivez les instructions pour supprimer la ressource.

## Plus d’informations

Dans cet exercice, vous avez exploré certaines des fonctionnalités d’analyse et de manipulation d’images du service Azure AI Vision. Le service inclut également des fonctionnalités de détection d’objets et de personnes, ainsi que d’autres tâches de vision par ordinateur.

Pour en savoir plus sur l’utilisation du service **Azure AI Vision**, consultez la [documentation d’Azure AI Vision](https://learn.microsoft.com/azure/ai-services/computer-vision/).
