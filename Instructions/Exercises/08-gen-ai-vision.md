---
lab:
  title: Développer une application de conversation activée par la vision
  description: "Découvrez comment utiliser Azure\_AI Foundry pour créer une application d’IA générative qui prend en charge les données d’image."
---

# Développer une application de conversation activée par la vision

Dans cet exercice, vous utilisez le modèle d’IA générative *Phi-4-multimodal-instruct* pour générer des réponses aux invites qui incluent des images. Vous allez développer une application qui aide l’IA à fournir des produits frais dans une épicerie grâce á Azure AI Foundry et au service d’inférence de modèle Azure AI.

Cet exercice prend environ **30** minutes.

## Créer un projet Azure AI Foundry

Commençons par créer un projet Azure AI Foundry.

1. Dans un navigateur web, ouvrez le [portail Azure AI Foundry](https://ai.azure.com) à l’adresse `https://ai.azure.com` et connectez-vous en utilisant vos informations d’identification Azure. Fermez les conseils ou les volets de démarrage rapide ouverts la première fois que vous vous connectez et, si nécessaire, utilisez le logo **Azure AI Foundry** en haut à gauche pour accéder à la page d’accueil, qui ressemble à l’image suivante :

    ![Capture d’écran du portail Azure AI Foundry.](../media/ai-foundry-home.png)

2. Sur la page d’accueil, sélectionnez **+Créer un projet**.
3. Dans l’assistant **Créer un projet**, saisissez un nom valide et, si un hub existant est suggéré, choisissez l’option permettant d’en créer un. Passez ensuite en revue les ressources Azure qui seront créées automatiquement pour prendre en charge votre hub et votre projet.
4. Sélectionnez **Personnaliser** et spécifiez les paramètres suivants pour votre hub :
    - **Nom du hub** : *un nom valide pour votre hub*
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *créez ou sélectionnez un groupe de ressources*
    - **Région** : Sélectionnez l’une des régions suivantes\* :
        - USA Est
        - USA Est 2
        - Centre-Nord des États-Unis
        - États-Unis - partie centrale méridionale
        - Suède Centre
        - USA Ouest
        - USA Ouest 3
    - **Connecter Azure AI Services ou Azure OpenAI** : *créer une nouvelle ressource AI Services*
    - **Connecter la Recherche Azure AI** : ignorer la connexion

    > \* Au moment de l’écriture, le modèle Microsoft *Phi-4-multimodal-instruct* que nous allons utiliser dans cet exercice est disponible dans ces régions. Vous pouvez consulter les dernières disponibilités régionales de certains modèles dans la [documentation Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/deploy-models-serverless-availability#region-availability). Si une limite de quota régionale est atteinte plus tard dans l’exercice, vous devrez peut-être créer une autre ressource dans une autre région.

5. Sélectionnez **Suivant** et passez en revue votre configuration. Sélectionnez **Créer** et patientez jusqu’à ce que l’opération se termine.
6. Une fois votre projet créé, fermez les conseils affichés et passez en revue la page du projet dans le portail Azure AI Foundry, qui doit ressembler à l’image suivante :

    ![Capture d’écran des détails d’un projet Azure AI dans le portail Azure AI Foundry.](../media/ai-foundry-project.png)

## Déployer un modèle multimodal

Vous êtes maintenant prêt à déployer un modèle multimodal qui peut prendre en charge l’entrée basée sur l’image. Vous pouvez choisir parmi plusieurs modèles, notamment le modèle OpenAI *gpt-4o*. Dans cet exercice, nous allons utiliser un modèle *Phi-4-modal-instruct* qui prend en charge les invites qui incluent des images.

1. Dans la barre d’outils située en haut à droite de la page de votre projet Azure AI Foundry, utilisez l’icône **Fonctionnalités en préversion** (**&#9215;**) pour vérifier que la fonctionnalité **Déployer des modèles vers le service d’inférence de modèles Azure AI** est activée. Cette fonctionnalité garantit que votre déploiement de modèle est disponible pour le service Inférence Azure AI, que vous utiliserez dans votre code d’application.
2. Dans le volet de gauche de votre projet, dans la section **Mes ressources**, sélectionnez la page **Modèles + points de terminaison**.
3. Sur la page **Modèles + points de terminaison**, dans l’onglet **Déploiements de modèles**, dans le menu **+ Déployer un modèle**, sélectionnez **Déployer le modèle de base**.
4. Recherchez le modèle **Phi-4-multimodal-instruct** dans la liste, puis sélectionnez-le et confirmez-le.
5. Acceptez le contrat de licence si vous y êtes invité, puis déployez le modèle avec les paramètres suivants en sélectionnant **Personnaliser** dans les détails du déploiement :
    - **Nom du déploiement** : *nom valide pour votre modèle de déploiement*
    - **Type de déploiement** : standard global
    - **Détails du déploiement** : *utilisez les paramètres par défaut*
6. Attendez que l’état d’approvisionnement du déploiement soit **Terminé**.

## Tester le modèle dans le terrain de jeu

Maintenant que vous disposez d’un déploiement de modèle multimodal, vous pouvez le tester avec une invite basée sur l’image dans le terrain de jeu de conversation.

1. Dans le volet de navigation de gauche, sélectionnez la page **Terrains de jeu**, puis ouvrez le terrain de jeu **Conversation**.
1. 1. Dans un nouvel onglet de navigateur, téléchargez [mango.jpeg](https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/08-gen-ai-vision/mango.jpeg) à partir de `https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/08-gen-ai-vision/mango.jpeg` et enregistrez-le dans un dossier sur votre système de fichiers local.
1. Dans la page du terrain de jeu de conversation, dans le volet **Configuration**, vérifiez si votre modèle de déploiement **Phi-4-multimodal-instruct** est sélectionné.
1. Dans le panneau principal de session de conversation, sous la zone d’entrée de conversation, utilisez le bouton Attacher (**&#128206 ;**) pour charger le fichier image *mango.jpeg*, puis ajoutez le texte `What desserts could I make with this fruit?` et envoyez l’invite.

    ![Capture d’écran du terrain de jeu de conversation avec une invite basée sur une image.](../media/chat-playground-image.png)

1. Passez en revue la réponse, qui devrait fournir des conseils pertinents sur les desserts que vous pouvez faire à l’aide d’une mangue.

## Créer une application cliente

Maintenant que vous avez déployé le modèle, vous pouvez utiliser le déploiement dans une application cliente.

> **Conseil** : vous pouvez choisir de développer votre solution à l’aide de Python ou de Microsoft C# *(bientôt disponible)*. Suivez les instructions de la section appropriée pour votre langue choisie.

### Préparer la configuration de l’application

1. Dans le portail Azure AI Foundry, affichez la page **Vue d’ensemble** de votre projet.
2. Dans la zone **Détails du projet**, notez la **chaîne de connexion du projet**. Vous utiliserez cette chaîne de connexion pour vous connecter à votre projet dans une application cliente.
3. Ouvrez un nouvel onglet de navigateur (en gardant le portail Azure AI Foundry ouvert dans l’onglet existant). Dans un nouvel onglet du navigateur, ouvrez le [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com` et connectez-vous en utilisant vos informations d’identification Azure.

    Fermez les notifications de bienvenue pour afficher la page d’accueil du portail Azure.

1. Utilisez le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un environnement Cloud Shell dans le portail Azure, puis sélectionnez un environnement ***PowerShell*** avec aucun stockage dans votre abonnement.

    Cloud Shell fournit une interface de ligne de commande via un volet situé en bas du portail Azure. Vous pouvez redimensionner ou agrandir ce volet pour faciliter le travail.

    > **Remarque** : si vous avez déjà créé un Cloud Shell qui utilise un environnement *Bash*, basculez-le vers ***PowerShell***.

5. Dans la barre d’outils Cloud Shell, dans le menu **Paramètres**, sélectionnez **Accéder à la version classique** (cela est nécessaire pour utiliser l’éditeur de code).

    **<font color="red">Assurez-vous d’avoir basculé vers la version classique du Cloud Shell avant de continuer.</font>**

1. Dans le volet Cloud Shell, saisissez les commandes suivantes pour cloner le référentiel GitHub contenant les fichiers de code pour cet exercice (saisissez la commande, ou copiez-la dans le presse-papiers puis effectuez un clic droit dans la ligne de commande pour la coller en texte brut) :


    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **Conseil** : lorsque vous collez des commandes dans cloudshell, la sortie peut prendre une grande quantité de mémoire tampon d’écran. Vous pouvez effacer le contenu de l’écran en saisissant la commande `cls` pour faciliter le focus sur chaque tâche.

7. Une fois le référentiel cloné, accédez au dossier contenant les fichiers de code de l’application :  

    **Python**

    ```
   cd mslearn-ai-vision/Labfiles/08-gen-ai-vision/python
    ```

    **C#**

    ```
   cd mslearn-ai-vision/Labfiles/08-gen-ai-vision/c-sharp
    ```

8. Dans le volet de ligne de commande Cloud Shell, saisissez la commande suivante pour installer les bibliothèques que vous utiliserez, c’est-à-dire :

    **Python**

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects azure-ai-inference
    ```

    **C#**

    ```
   dotnet add package Azure.Identity
   dotnet add package Azure.AI.Inference --version 1.0.0-beta.3
   dotnet add package Azure.AI.Projects --version 1.0.0-beta.3
    ```

9. Saisissez la commande suivante pour modifier le fichier de configuration fourni :

    **Python**

    ```
   code .env
    ```

    **C#**

    ```
   code appsettings.json
    ```

    Le fichier s’ouvre dans un éditeur de code.

10. Dans le fichier de code, remplacez l’espace réservé **your_project_connection_string** par la chaîne de connexion de votre projet (copiée depuis la page **Vue d'ensemble** du projet dans le portail Azure AI Foundry), et remplacez **your_model_deployment** par le nom que vous avez attribué à votre déploiement du modèle Phi-4-multimodal-instruct.
11. Une fois que vous avez remplacé les espaces réservés, dans l’éditeur de code, utilisez la commande **Ctrl+S** ou **Clic droit > Enregistrer** dans l’éditeur de code pour enregistrer vos modifications, puis utilisez la commande **Ctrl+Q** ou **Clic droit > Quitter** pour fermer l’éditeur tout en gardant la ligne de commande Cloud Shell ouverte.

### Écrire du code pour vous connecter à votre projet et obtenir un client de conversation instantanée pour votre modèle

> **Conseil** : lorsque vous ajoutez du code, veillez à conserver la mise en retrait correcte.

1. Saisissez la commande suivante pour modifier le fichier de code fourni :

    **Python**

    ```
   code chat-app.py
    ```

    **C#**

    ```
   code Program.cs
    ```

2. Dans le fichier de code, notez les instructions existantes qui ont été ajoutées en haut du fichier pour importer les espaces de noms SDK nécessaires. Ensuite, repérez le commentaire **Ajouter des références**, puis ajoutez le code suivant pour référencer les espaces de noms des bibliothèques que vous avez installées précédemment :

    **Python**

    ```python
   # Add references
   from dotenv import load_dotenv
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from azure.ai.inference.models import (
        SystemMessage,
        UserMessage,
        TextContentItem,
        ImageContentItem,
        ImageUrl,
   )
    ```

    **C#**

    ```csharp
   // Add references
   using Azure.Identity;
   using Azure.AI.Projects;
   using Azure.AI.Inference;
    ```

3. Dans la fonction **main**, sous le commentaire **Obtenir les paramètres de configuration**, notez que le code charge les valeurs de chaîne de connexion du projet et de nom de déploiement du modèle que vous avez définies dans le fichier de configuration.
4. Recherchez le commentaire **Initialiser le client du projet**, puis ajoutez le code suivant pour vous connecter à votre projet Azure AI Foundry à l’aide des informations d’identification Azure avec lesquelles vous êtes actuellement connecté.

    **Python**

    ```python
   # Initialize the project client
   project_client = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential())
    ```

    **C#**

    ```csharp
   // Initialize the project client
   var projectClient = new AIProjectClient(project_connection,
                        new DefaultAzureCredential());
    ```

5. Recherchez le commentaire **Obtenir un client de conversation instantanée** et ajoutez le code suivant pour créer un objet client afin de converser avec votre modèle :

    **Python**

    ```python
   # Get a chat client
   chat_client = project_client.inference.get_chat_completions_client(model=model_deployment)
    ```

    **C#**

    ```csharp
   // Get a chat client
   ChatCompletionsClient chat = projectClient.GetChatCompletionsClient();
    ```

### Écrire du code pour envoyer une invite basée sur une URL

1. Notez que le code inclut une boucle pour permettre à un utilisateur d’entrer une invite jusqu’à ce qu’il entre « quitter ». Ensuite, dans la section de boucle, recherchez le commentaire **Obtenir une réponse à l‘entrée d’une image**, puis ajoutez le code suivant pour envoyer une invite incluant l’image suivante :

    ![Photo d’une mangue.](../media/orange.jpeg)

    **Python**

    ```python
   # Get a response to image input
   image_url = "https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/08-gen-ai-vision/orange.jpeg"
   image_format = "jpeg"
   request = Request(image_url, headers={"User-Agent": "Mozilla/5.0"})
   image_data = base64.b64encode(urlopen(request).read()).decode("utf-8")
   data_url = f"data:image/{image_format};base64,{image_data}"

   response = chat_client.complete(
        messages=[
            SystemMessage(system_message),
            UserMessage(content=[
                TextContentItem(text=prompt),
                ImageContentItem(image_url=ImageUrl(url=data_url))
            ]),
        ]
   )
   print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
   // Get a response to image input
   string imageUrl = "https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/08-gen-ai-vision/orange.jpeg";
   ChatCompletionsOptions requestOptions = new ChatCompletionsOptions()
   {
        Messages = {
           new ChatRequestSystemMessage(system_message),
           new ChatRequestUserMessage([
                new ChatMessageTextContentItem(prompt),
                new ChatMessageImageContentItem(new Uri(imageUrl))
            ]),
        },
        Model = model_deployment
   };
   var response = chat.Complete(requestOptions);
   Console.WriteLine(response.Value.Content);
    ```

2. Utilisez la commande **Ctrl+S** pour enregistrer les modifications que vous avez apportées au fichier de code. Laissez-le encore.

3. Dans le volet de ligne de commande Cloud Shell sous l’Éditeur de code, entrez la commande suivante pour exécuter l’application :

    **Python**

    ```
   python chat-app.py
    ```

    **C#**

    ```
   dotnet run
    ```

4. Quand vous y êtes invité, entrez l’invite suivante :

    ```
   Suggest some recipes that include this fruit
    ```

5. Vérifiez la réponse. Ensuite, entrez `quit` pour quitter le programme.

### Modifier le code pour charger un fichier image local

1. Dans l’éditeur de code de votre code d’application, dans la section boucle, recherchez le code que vous avez ajouté précédemment sous le commentaire **Obtenir une réponse à l’entrée d’image**. Modifiez ensuite le code comme suit pour charger ce fichier image local :

    ![Photo d’un fruit du dragon.](../media/mystery-fruit.jpeg)

    **Python**

    ```python
   # Get a response to image input
   script_dir = Path(__file__).parent  # Get the directory of the script
   image_path = script_dir / 'mystery-fruit.jpeg'
   mime_type = "image/jpeg"

    # Read and encode the image file
    with open(image_path, "rb") as image_file:
        base64_encoded_data = base64.b64encode(image_file.read()).decode('utf-8')

    # Include the image file data in the prompt
    data_url = f"data:{mime_type};base64,{base64_encoded_data}"
    response = chat_client.complete(
        messages=[
            SystemMessage(system_message),
            UserMessage(content=[
                TextContentItem(text=prompt),
                ImageContentItem(image_url=ImageUrl(url=data_url))
            ]),
        ]
    )
    print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
   // Get a response to image input
   string imagePath = "mystery-fruit.jpeg";
   string mimeType = "image/jpeg";
    
   // Read and encode the image file
   byte[] imageBytes = File.ReadAllBytes(imagePath);
   var binaryImage = new BinaryData(imageBytes);
    
   // Include the image file data in the prompt
   ChatCompletionsOptions requestOptions = new ChatCompletionsOptions()
   {
        Messages = {
            new ChatRequestSystemMessage(system_message),
            new ChatRequestUserMessage([
                new ChatMessageTextContentItem(prompt),
                new ChatMessageImageContentItem(bytes: binaryImage, mimeType: mimeType) 
            ]),
        },
        Model = model_deployment
   };
   var response = chat.Complete(requestOptions);
   Console.WriteLine(response.Value.Content);
    ```

2. Utilisez la commande **Ctrl+S** pour enregistrer les modifications que vous avez apportées au fichier de code. Vous pouvez également fermer l’éditeur de code (**Ctrl+Q**) si vous le souhaitez.

3. Dans le volet de ligne de commande Cloud Shell sous l’Éditeur de code, entrez la commande suivante pour exécuter l’application :

    **Python**

    ```
   python chat-app.py
    ```

    **C#**

    ```
   dotnet run
    ```

4. Quand vous y êtes invité, entrez l’invite suivante :

    ```
   What is this fruit? What recipes could I use it in?
    ```

5. Vérifiez la réponse. Ensuite, entrez `quit` pour quitter le programme.

    > **Remarque** : dans cette application simple, nous n‘avons pas implémenté de logique pour conserver l‘historique des conversations. Par conséquent, le modèle traite chaque invite comme une nouvelle requête sans contexte de l‘invite précédente.

## Pour en apprendre davantage (si le temps le permet) :

Vous avez appris à utiliser le SDK Inférence Azure AI et un modèle multimodal pour implémenter une application d’IA générative capable de répondre aux invites basées sur des images. Si vous avez du temps, voici quelques idées d’exploration plus poussée.

### Utiliser un autre modèle multimodal

Vous avez utilisé un modèle *Phi-4-modal-instruct* pour générer une réponse à une invite basée sur une image. Essayons maintenant un modèle OpenAI *gpt-4o*.

1. Dans Azure AI Foundry, déployez un modèle **gpt-4o** sur un point de terminaison d’inférence de modèle Azure AI (vous devrez peut-être créer une ressource dans une autre région).
1. Mettez à jour le fichier de configuration du code de votre application (*.env* pour Python, *appsettings.json* pour C#) afin de spécifier le nom de votre modèle gpt-4o.
1. Exécutez l’application comme avant, en utilisant les mêmes invites (vous pouvez revenir au code qui utilise une image basée sur une URL si vous le souhaitez).

### Utiliser des API OpenAI

Le code que vous avez utilisé dans cet exercice est basé sur le SDK Azure AI Inference, qui fonctionne avec n’importe quel modèle déployé sur un point de terminaison d’inférence de modèle Azure AI. Lorsque vous utilisez un modèle OpenAI, vous pouvez également utiliser le SDK OpenAI.

Les instructions suivantes supposent que vous ayez effectué cet exercice et la tâche supplémentaire ci-dessus pour déployer et tester un modèle **gpt-4o**.

1. Installez (ou mettez à jour) les packages nécessaires pour votre application :

    **Python**

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects openai
    ```
    
    **C#**

    ```
   dotnet add package Azure.Identity
   dotnet add package Azure.AI.Projects --prerelease
   dotnet add package Azure.AI.OpenAI --prerelease
    ```

1. Mettez à jour les espaces de noms dans votre fichier de code (en supprimant les références *azure.ai-inference*) :

    **Python**

    ```python
   # Add references
   from dotenv import load_dotenv
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   import openai
    ```

    **C#**

    ```csharp
   // Add references
   using Azure.Identity;
   using Azure.AI.Projects;
   using OpenAI.Chat;
   using Azure.AI.OpenAI;
    ```

1. Modifiez le code pour obtenir un client de conversation :

    **Python**

    ```python
   # Get a chat client
   chat_client = project_client.inference.get_azure_openai_client(api_version="2024-10-21")
    ```

    **C#**

    ```csharp
   // Get a chat client
   ChatClient chatClient = projectClient.GetAzureOpenAIChatClient(model_deployment);
    ```

1. Modifier le code pour obtenir un résultat basé sur un fichier image local

    **Python**

    ```python
   # Get a response to image input
   script_dir = Path(__file__).parent  # Get the directory of the script
   image_path = script_dir / 'mystery-fruit.jpeg'
   mime_type = "image/jpeg"

   # Read and encode the image file
   with open(image_path, "rb") as image_file:
        base64_encoded_data = base64.b64encode(image_file.read()).decode('utf-8')

   # Include the image file data in the prompt
   data_url = f"data:{mime_type};base64,{base64_encoded_data}"
   response = chat_client.chat.completions.create(
        model=model_deployment,
        messages=[
            { "role": "system", "content": system_message },
            { "role": "user", "content": [  
                { 
                    "type": "text", 
                    "text": prompt 
                },
                { 
                    "type": "image_url",
                    "image_url": {
                        "url": data_url
                    }
                }
            ] } 
        ]
   )
   completion = response.choices[0].message.content
   print(completion)
    ```

    **C#**

    ```csharp
   // Get a response to image input
   string imagePath = "mystery-fruit.jpeg";
   string mimeType = "image/jpeg";
    
   // Read and encode the image file
   byte[] imageBytes = File.ReadAllBytes(imagePath);
   var binaryImage = new BinaryData(imageBytes);

   // Include the image file data in the prompt
   List<ChatMessage> messages =
   [
        new SystemChatMessage(system_message),
        new UserChatMessage(
            ChatMessageContentPart.CreateTextPart(prompt),
            ChatMessageContentPart.CreateImagePart(binaryImage, mimeType)),
   ];

   ChatCompletion completion = chatClient.CompleteChat(messages);
   Console.WriteLine(completion.Content[0].Text);
    ```

1. Enregistrez vos modifications et exécutez l’application pour la tester avec les mêmes invites que celles que vous avez utilisées précédemment.

## Nettoyage

.Si vous avez terminé d‘explorer Azure AI Foundry, vous devez supprimer les ressources que vous avez créées dans cet exercice pour éviter d‘entraîner des coûts Azure inutiles.

1. Revenez à l’onglet du navigateur contenant le portail Azure (ou ouvrez à nouveau le [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com` dans un nouvel onglet de navigateur) et affichez le contenu du groupe de ressources dans lequel vous avez déployé les ressources utilisées dans cet exercice.
1. Dans la barre d’outils, sélectionnez **Supprimer le groupe de ressources**.
1. Entrez le nom du groupe de ressources et confirmez que vous souhaitez le supprimer.
