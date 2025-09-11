---
lab:
  title: Développer une application de conversation activée par la vision
  description: "Utilisez Azure\_AI Foundry pour créer une application d’IA générative qui prend en charge les données d’image."
---

# Développer une application de conversation activée par la vision

Dans cet exercice, vous utilisez le modèle d’IA générative *Phi-4-multimodal-instruct* pour générer des réponses aux invites qui incluent des images. Vous allez développer une application qui aide l’IA à fournir des produits frais dans une épicerie grâce á Azure AI Foundry et au service d’inférence de modèle Azure AI.

> **Remarque** : Cet exercice est basé sur une version préliminaire du logiciel SDK, qui est susceptible d'être modifiée. Le cas échéant, nous avons utilisé des versions spécifiques de certains packages, qui ne correspondent pas forcément aux versions les plus récentes disponibles. Il se peut que vous rencontriez des comportements, inattendus, des avertissements ou des erreurs.

Bien que cet exercice soit basé sur le SDK Python Azure AI Foundry, vous pouvez développer des applications de chat IA à l'aide de plusieurs SDK spécifiques à chaque langage, notamment :

- [Projets Azure AI pour Python](https://pypi.org/project/azure-ai-projects)
- [Projets Azure AI pour Microsoft .NET](https://www.nuget.org/packages/Azure.AI.Projects)
- [Projets Azure AI pour JavaScript](https://www.npmjs.com/package/@azure/ai-projects)

Cet exercice prend environ **30** minutes.

## Ouvrir le portail Azure AI Foundry

Commençons par nous connecter au portail Azure AI Foundry.

1. Dans un navigateur web, ouvrez le [portail Azure AI Foundry](https://ai.azure.com) à l’adresse `https://ai.azure.com` et connectez-vous en utilisant vos informations d’identification Azure. Fermez les conseils ou les volets de démarrage rapide ouverts la première fois que vous vous connectez et, si nécessaire, utilisez le logo **Azure AI Foundry** en haut à gauche pour accéder à la page d’accueil, qui ressemble à l’image suivante (fermez le volet **Aide** s’il est ouvert) :

    ![Capture d’écran du portail Azure AI Foundry.](./media/ai-foundry-home.png)

1. Examinez les informations sur la page d’accueil.

## Choisir un modèle pour démarrer un projet

Un *projet* Azure AI fournit un espace de travail collaboratif pour le développement de solutions d’IA. Commençons par choisir un modèle avec lequel nous voulons travailler, puis créons un projet pour l’utiliser.

> **Remarque** : les projets AI Foundry peuvent reposer sur une ressource *Azure AI Foundry*, qui donne accès à des modèles d’IA (y compris Azure OpenAI), aux services Azure AI et à d’autres ressources pour le développement d’assistants d’IA et de solutions de discussion. Les projets peuvent aussi être basés sur des ressources *Hub IA*, qui incluent des connexions aux ressources Azure pour le stockage sécurisé, la capacité de calcul et des outils spécialisés. Les projets fondés sur Azure AI Foundry conviennent parfaitement aux développeurs souhaitant gérer eux-mêmes les ressources pour le développement d’assistants d’IA ou d’applications de discussion. Les projets basés sur Hub IA sont plus adaptés aux équipes de développement en entreprise qui travaillent sur des solutions d’IA complexes.

1. Dans la page d’accueil, dans la section **Explorer les modèles et les fonctionnalités**, recherchez le modèle `Phi-4-multimodal-instruct` ; que nous utiliserons dans notre projet.

1. Dans les résultats de recherche, sélectionnez le modèle **Phi-4-multimodal-instruct** pour consulter ses détails, puis, en haut de la page du modèle, sélectionnez **Utiliser ce modèle**.

1. Lorsque vous êtes invité à créer un projet, entrez un nom valide pour votre projet et développez les **options avancées**.

1. Sélectionnez **Personnaliser** et spécifiez les paramètres suivants pour votre hub :
    - **Ressource Azure AI Foundry** : *un nom valide pour votre ressource Azure AI Foundry.*
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *créez ou sélectionnez un groupe de ressources*
    - **Région** : *Sélectionnez n’importe quelle **recommandation d’AI Foundry***\*

    > \* Certaines ressources Azure AI sont limitées par des quotas de modèles régionaux. Si une limite de quota est atteinte plus tard dans l’exercice, vous devrez peut-être créer une autre ressource dans une autre région.

1. Sélectionnez **Créer** et attendez que votre projet, incluant le déploiement du modèle Phi-4-multimodal-instruct que vous avez sélectionné, soit créé.

    > Note : selon la sélection de votre modèle, vous pouvez recevoir des invites supplémentaires pendant le processus de création du projet. Acceptez les termes et finalisez le déploiement.

1. Une fois votre projet créé, votre modèle s’affiche dans la page **Modèles + points de terminaison** :

    ![Capture d’écran de la page du modèle de déploiement.](./media/ai-foundry-model-deployment.png)

## Tester le modèle dans le terrain de jeu

Vous avez désormais la capacité de tester votre modèle de déploiement multimodal avec une requête fondée sur une image dans le terrain de jeu de conversation.

1. Sélectionnez **Ouvrir dans le terrain de jeu** dans la page du modèle de déploiement.

1. Dans un nouvel onglet de navigateur, téléchargez [mango.jpeg](https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/gen-ai-vision/mango.jpeg) à partir de `https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/gen-ai-vision/mango.jpeg` et enregistrez-le dans un dossier sur votre système de fichiers local.

1. Dans la page du terrain de jeu de conversation, dans le volet **Configuration**, vérifiez si votre modèle de déploiement **Phi-4-multimodal-instruct** est sélectionné.

1. Dans le panneau principal de session de conversation, sous la zone d’entrée de conversation, utilisez le bouton Attacher (**&#128206 ;**) pour charger le fichier image *mango.jpeg*, puis ajoutez le texte `What desserts could I make with this fruit?` et envoyez l’invite.

    ![Capture d’écran de la page du terrain de jeu de conversation.](../media/chat-playground-image.png)

1. Passez en revue la réponse, qui devrait fournir des conseils pertinents sur les desserts que vous pouvez faire à l’aide d’une mangue.

## Créer une application cliente

Maintenant que vous avez déployé le modèle, vous pouvez utiliser le déploiement dans une application cliente.

### Préparer la configuration de l’application

1. Dans le portail Azure AI Foundry, affichez la page **Vue d’ensemble** de votre projet.

1. Dans la zone **Points de terminaison et clés**, assurez-vous que la bibliothèque **Azure AI Foundry** est sélectionnée, et notez le **point de terminaison de projet Azure AI Foundry**. Vous utiliserez cette chaîne de connexion pour vous connecter à votre projet dans une application cliente.

1. Ouvrez un nouvel onglet de navigateur (en gardant le portail Azure AI Foundry ouvert dans l’onglet existant). Dans un nouvel onglet du navigateur, ouvrez le [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com` et connectez-vous en utilisant vos informations d’identification Azure.

    Fermez les notifications de bienvenue pour afficher la page d’accueil du portail Azure.

1. Utilisez le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un environnement Cloud Shell dans le portail Azure, puis sélectionnez un environnement ***PowerShell*** avec aucun stockage dans votre abonnement.

    Cloud Shell fournit une interface de ligne de commande via un volet situé en bas du portail Azure. Vous pouvez redimensionner ou agrandir ce volet pour faciliter le travail.

    > **Remarque** : si vous avez déjà créé un Cloud Shell qui utilise un environnement *Bash*, basculez-le vers ***PowerShell***.

    > **Remarque** : si le portail sollicite la sélection d’un stockage pour la persistance des fichiers, sélectionnez **Aucun compte de stockage requis**, choisissez l’abonnement concerné et validez avec **Appliquer**.

1. Dans la barre d’outils Cloud Shell, dans le menu **Paramètres**, sélectionnez **Accéder à la version classique** (cela est nécessaire pour utiliser l’éditeur de code).

    **<font color="red">Assurez-vous d’avoir basculé vers la version classique du Cloud Shell avant de continuer.</font>**

1. Dans le volet Cloud Shell, saisissez les commandes suivantes pour cloner le référentiel GitHub contenant les fichiers de code pour cet exercice (saisissez la commande, ou copiez-la dans le presse-papiers puis effectuez un clic droit dans la ligne de commande pour la coller en texte brut) :

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **Conseil** : lorsque vous collez des commandes dans cloudshell, la sortie peut prendre une grande quantité de mémoire tampon d’écran. Vous pouvez effacer le contenu de l’écran en saisissant la commande `cls` pour faciliter le focus sur chaque tâche.

1. Une fois le référentiel cloné, accédez au dossier contenant les fichiers de code de l’application :  

    ```
   cd mslearn-ai-vision/Labfiles/gen-ai-vision/python
    ```

1. Dans le volet de ligne de commande Cloud Shell, saisissez la commande suivante pour installer les bibliothèques que vous utiliserez, c’est-à-dire :

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-identity azure-ai-projects openai
    ```

1. Saisissez la commande suivante pour modifier le fichier de configuration fourni :

    ```
   code .env
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Dans le fichier de code, remplacez l’espace réservé **your_project_endpoint** par le point de terminaison de projet Foundry (copié depuis la page **Vue d’ensemble** du projet dans le portail Azure AI Foundry), et l’espace réservé **your_model_deployment** par le nom que vous avez attribué à votre modèle de déploiement Phi-4-multimodal-instruct.

1. Une fois que vous avez remplacé les espaces réservés, dans l’éditeur de code, utilisez la commande **Ctrl+S** ou **Clic droit > Enregistrer** dans l’éditeur de code pour enregistrer vos modifications, puis utilisez la commande **Ctrl+Q** ou **Clic droit > Quitter** pour fermer l’éditeur tout en gardant la ligne de commande Cloud Shell ouverte.

### Écrire du code pour vous connecter à votre projet et obtenir un client de conversation instantanée pour votre modèle

> **Conseil** : lorsque vous ajoutez du code, veillez à conserver la mise en retrait correcte.

1. Saisissez la commande suivante pour modifier le fichier de code fourni :

    ```
   code chat-app.py
    ```

1. Dans le fichier de code, notez les instructions existantes qui ont été ajoutées en haut du fichier pour importer les espaces de noms SDK nécessaires. Ensuite, repérez le commentaire **Ajouter des références**, puis ajoutez le code suivant pour référencer les espaces de noms des bibliothèques que vous avez installées précédemment :

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from openai import AzureOpenAI
    ```

1. Dans la fonction **main**, sous le commentaire **Obtenir les paramètres de configuration**, notez que le code charge les valeurs de chaîne de connexion du projet et de nom de déploiement du modèle que vous avez définies dans le fichier de configuration.
1. Dans la fonction **main**, sous le commentaire **Obtenir les paramètres de configuration**, notez que le code charge les valeurs de chaîne de connexion du projet et de nom de déploiement du modèle que vous avez définies dans le fichier de configuration.
1. Recherchez le commentaire **Initialiser le client du projet** et ajoutez le code suivant pour vous connecter à votre projet Azure AI Foundry :

    > **Conseil** : veillez à respecter le niveau d’indentation correct dans votre code.

    ```python
   # Initialize the project client
   project_client = AIProjectClient(            
            credential=DefaultAzureCredential(
                exclude_environment_credential=True,
                exclude_managed_identity_credential=True
            ),
            endpoint=project_endpoint,
        )
    ```

1. Recherchez le commentaire **Obtenir un client de conversation instantanée**, puis ajoutez le code suivant pour créer un objet client permettant d’interagir avec un modèle :

    ```python
   # Get a chat client
   openai_client = project_client.get_openai_client(api_version="2024-10-21")
    ```

### Écrire du code pour envoyer une invite basée sur une URL

1. Notez que le code inclut une boucle pour permettre à un utilisateur d’entrer une invite jusqu’à ce qu’il entre « quitter ». Ensuite, dans la section de boucle, recherchez le commentaire **Obtenir une réponse à l‘entrée d’une image**, puis ajoutez le code suivant pour envoyer une invite incluant l’image suivante :

    ![Photo d’une mangue.](../media/orange.jpeg)

    ```python
   # Get a response to image input
   image_url = "https://github.com/MicrosoftLearning/mslearn-ai-vision/raw/refs/heads/main/Labfiles/gen-ai-vision/orange.jpeg"
   image_format = "jpeg"
   request = Request(image_url, headers={"User-Agent": "Mozilla/5.0"})
   image_data = base64.b64encode(urlopen(request).read()).decode("utf-8")
   data_url = f"data:image/{image_format};base64,{image_data}"

   response = openai_client.chat.completions.create(
        model=model_deployment,
        messages=[
            {"role": "system", "content": system_message},
            { "role": "user", "content": [  
                { "type": "text", "text": prompt},
                { "type": "image_url", "image_url": {"url": data_url}}
            ] } 
        ]
   )
   print(response.choices[0].message.content)
    ```

1. Utilisez la commande **Ctrl+S** pour enregistrer les modifications que vous avez apportées au fichier de code. Laissez-le encore.

## Se connecter à Azure et exécuter l’application

1. Dans le volet de la ligne de commande Cloud Shell, entrez la commande suivante pour vous connecter à Azure.

    ```
   az login
    ```

    **<font color="red">Vous devez vous connecter à Azure, même si la session Cloud Shell est déjà authentifiée.</font>**

    > **Remarque** :dans la plupart des scénarios, l’utilisation d’*az login* suffit. Toutefois, si vous avez des abonnements dans plusieurs locataires, vous devrez peut-être spécifier le locataire à l’aide du paramètre *--tenant*. Pour plus d’informations, consultez [Se connecter à Azure de manière interactive à l’aide d’Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively).
    
1. Lorsque l’invite apparaît, suivez les instructions pour ouvrir la page de connexion dans un nouvel onglet et entrez le code d’authentification fourni ainsi que vos informations d’identification Azure. Effectuez ensuite le processus de connexion dans la ligne de commande, en sélectionnant l’abonnement contenant votre hub Azure AI Foundry si nécessaire.

1. Une fois la connexion effectuée, entrez la commande suivante pour exécuter l’application :

    ```
   python chat-app.py
    ```

1. Quand vous y êtes invité, entrez l’invite suivante :

    ```
   Suggest some recipes that include this fruit
    ```

1. Passez en revue la réponse. Ensuite, entrez `quit` pour quitter le programme.

### Modifier le code pour charger un fichier image local

1. Dans l’éditeur de code de votre code d’application, dans la section boucle, recherchez le code que vous avez ajouté précédemment sous le commentaire **Obtenir une réponse à l’entrée d’image**. Modifiez ensuite le code comme suit pour charger ce fichier image local :

    ![Photo d’un fruit du dragon.](../media/mystery-fruit.jpeg)

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
   response = openai_client.chat.completions.create(
            model=model_deployment,
            messages=[
                {"role": "system", "content": system_message},
                { "role": "user", "content": [  
                    { "type": "text", "text": prompt},
                    { "type": "image_url", "image_url": {"url": data_url}}
                ] } 
            ]
   )
   print(response.choices[0].message.content)
    ```

1. Utilisez la commande **Ctrl+S** pour enregistrer les modifications que vous avez apportées au fichier de code. Vous pouvez également fermer l’éditeur de code (**Ctrl+Q**) si vous le souhaitez.

1. Dans le volet de ligne de commande Cloud Shell sous l’Éditeur de code, entrez la commande suivante pour exécuter l’application :

    ```
   python chat-app.py
    ```

1. Quand vous y êtes invité, entrez l’invite suivante :

    ```
   What is this fruit? What recipes could I use it in?
    ```

15. Passez en revue la réponse. Ensuite, entrez `quit` pour quitter le programme.

    > **Remarque** : dans cette application simple, nous n‘avons pas implémenté de logique pour conserver l‘historique des conversations. Par conséquent, le modèle traite chaque invite comme une nouvelle requête sans contexte de l‘invite précédente.

## Nettoyage

Si vous avez terminé d’explorer Azure AI Foundry, vous devez supprimer les ressources que vous avez créées dans cet exercice pour éviter d’entraîner des coûts Azure inutiles.

1. Ouvrez le [portail Azure](https://portal.azure.com) et affichez le contenu du groupe de ressources où vous avez déployé les ressources utilisées dans cet exercice.
1. Dans la barre d’outils, sélectionnez **Supprimer le groupe de ressources**.
1. Entrez le nom du groupe de ressources et confirmez que vous souhaitez le supprimer.
