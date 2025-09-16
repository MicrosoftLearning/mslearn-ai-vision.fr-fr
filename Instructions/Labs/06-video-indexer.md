---
lab:
  title: Analyser une vidéo
  description: "Utilisez Azure\_AI\_Video\_Indexer pour analyser une vidéo."
---

# Analyser une vidéo

Une grande proportion des données créées et consommées aujourd’hui est au format vidéo. **Azure AI Video Indexer** est un service basé sur l’IA que vous pouvez utiliser pour indexer des vidéos et en extraire des insights.

> **Remarque** : depuis le 21 juin 2022, les fonctionnalités d’Azure AI services qui retournent des informations d’identification personnelle sont limitées aux clients qui ont reçu [un accès limité](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access). Sans approbation d’accès limité, la reconnaissance des personnes et des célébrités avec Video Indexer pour ce labo n’est pas disponible. Pour plus d’informations sur les modifications apportées par Microsoft, et leur motif, consultez [Responsible AI investments and safeguards for facial recognition](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/) (Investissements responsables en matière d'IA et mesures de protection pour la reconnaissance faciale).

## Charger une vidéo sur Video Indexer

Tout d’abord, vous devrez vous connecter au portail Video Indexer et charger une vidéo.

1. Dans votre navigateur, ouvrez le [portail Video Indexer](https://www.videoindexer.ai) à l’adresse `https://www.videoindexer.ai`.
1. Si vous disposez d’un compte Video Indexer existant, connectez-vous. Sinon, créez un compte gratuit et connectez-vous à l’aide de votre compte Microsoft (ou tout autre type de compte valide). Si vous ne parvenez pas à vous connecter, essayez d’ouvrir une session de navigation privée.

    > Note : si c’est la première fois que vous vous connectez, un formulaire contextuel peut s’afficher pour vous demander de confirmer l’utilisation que vous comptez faire du service. 

1. Dans un nouvel onglet, téléchargez la vidéo IA responsable en visitant `https://aka.ms/responsible-ai-video`. Enregistrez le fichier.
1. Dans Video Indexer, sélectionnez l’option **Charger**. Sélectionnez ensuite l’option permettant de **Parcourir les fichiers**, sélectionnez la vidéo téléchargée, puis cliquez sur **Ajouter**. Modifiez le texte dans le champ **Nom de fichier** pour **IA responsable**. Sélectionnez **Vérifier + charger**, passer en revue la présentation du résumé, cochez la case pour vérifier la conformité avec les stratégies de Microsoft pour la reconnaissance faciale, puis chargez le fichier.
1. Une fois le fichier chargé, patientez quelques minutes pendant que Video Indexer l’indexe automatiquement.

> **Remarque** : Dans cet exercice, nous utilisons cette vidéo pour explorer les fonctionnalités de Video Indexer, mais vous devriez prendre le temps de la regarder entièrement lorsque vous aurez terminé l’exercice, car elle contient des informations et des conseils utiles relatifs au développement responsable d’applications compatibles avec l’IA. 

## Passer en revue les insights vidéo

Le processus d’indexation extrait des insights de la vidéo, que vous pouvez afficher dans le portail.

1. Dans le portail Video Indexer, une fois la vidéo indexée, sélectionnez-la pour l’afficher. Vous verrez le lecteur vidéo ainsi qu’un volet qui affiche les insights extraits de la vidéo.

    > **Remarque** : en raison de la stratégie d’accès limité qui protège les identités des personnes, vous ne voyez peut-être pas de noms lorsque vous indexez la vidéo.

    ![Video Indexer avec un lecteur vidéo et un volet Insights](../media/video-indexer-insights.png)

1. Durant la lecture de la vidéo, sélectionnez l’onglet **Chronologie** pour afficher une transcription audio de la vidéo.

    ![Video Indexer avec un lecteur vidéo et un volet Chronologie affichant la transcription de la vidéo.](../media/video-indexer-transcript.png)

1. En haut à droite du portail, sélectionnez le symbole **Affichage** (qui ressemble à &#128455;), puis, dans la liste des insights, en plus de **Transcription**, sélectionnez **OCR** et **Haut-parleurs**.

    ![Menu d’affichage de Video Indexer avec Transcription, OCR et Haut-parleurs sélectionnés](../media/video-indexer-view-menu.png)

1. Notez que le volet **Chronologie** comprend désormais :
    - Une transcription de la narration audio.
    - Le texte visible dans la vidéo.
    - Des indications des intervenants qui apparaissent dans la vidéo. Certaines personnes connues sont automatiquement reconnues par leur nom, d’autres sont indiquées par un numéro (par exemple *Speaker #1*).
1. Revenez au volet **Insights** et affichez les insights. Notamment :
    - Les personnes individuelles qui apparaissent dans la vidéo.
    - Les sujets abordés dans la vidéo.
    - Les étiquettes des objets qui apparaissent dans la vidéo.
    - Les entités nommées, telles que les personnes et les marques qui apparaissent dans la vidéo.
    - Les scènes clés.
1. Avec le volet **Insights** visible, sélectionnez à nouveau le symbole **Affichage**, puis, dans la liste des insights, ajoutez **Mots clés** et **Sentiments** au volet.

    Les insights trouvés peuvent vous aider à déterminer les principaux thèmes de la vidéo. Par exemple, les **sujets** de cette vidéo montrent clairement qu’elle porte sur la technologie, la responsabilité sociale et l’éthique.

## Rechercher des insights

Vous pouvez utiliser Video Indexer pour rechercher des insights dans la vidéo.

1. Dans le volet **Insights**, dans la zone **Recherche**, saisissez *Abeille*. Vous devrez peut-être défiler vers le bas dans le volet Insights pour afficher les résultats de tous les types d’insights.
1. Notez qu’une *étiquette* correspondante est trouvée, avec son emplacement dans la vidéo indiquée en dessous.
1. Sélectionnez le début de la section où la présence d’une abeille est indiquée et affichez la vidéo à ce moment-là (vous devrez peut-être suspendre la vidéo et sélectionner soigneusement ; l’abeille apparaît seulement brièvement !)

    ![Résultats de la recherche Video Indexer pour Bee](../media/video-indexer-search.png)

1. Désactivez la zone **Recherche** pour afficher tous les insights de la vidéo.

## Utiliser l’API REST Video Indexer

Video Indexer fournit une API REST que vous pouvez utiliser pour charger et gérer des vidéos dans votre compte.

1. Dans un nouvel onglet du navigateur, ouvrez le [portail Azure](https://portal.azure.com) sur `https://portal.azure.com` et connectez-vous à l’aide de vos informations d’identification Azure. Conservez l’onglet existant avec le portail Video Indexer ouvert.
1. Dans le portail Azure, utilisez le bouton **[\>_]** situé à droite de la barre de recherche en haut de la page pour créer un nouveau Cloud Shell dans le portail Azure, en sélectionnant un environnement ***PowerShell*** sans stockage dans votre abonnement.

    Cloud Shell fournit une interface de ligne de commande via un volet situé en bas du portail Azure.

    > **Remarque** : si vous avez déjà créé un Cloud Shell qui utilise un environnement *Bash*, basculez-le vers ***PowerShell***.

    > **Remarque** : si le portail sollicite la sélection d’un stockage pour la persistance des fichiers, choisissez **Aucun compte de stockage requis**, sélectionnez l’abonnement concerné et appuyez sur **Appliquer**.

1. Dans la barre d’outils Cloud Shell, dans le menu **Paramètres**, sélectionnez **Accéder à la version classique** (cela est nécessaire pour utiliser l’éditeur de code).

    **<font color="red">Assurez-vous d’avoir basculé vers la version classique du Cloud Shell avant de continuer.</font>**

1. Redimensionnez le volet Cloud Shell afin que vous puissiez en voir davantage.

    > **Conseil** : vous pouvez redimensionner le volet en faisant glisser la bordure supérieure. Vous pouvez également utiliser les boutons de réduction et d’agrandissement pour alterner entre le Cloud Shell et l’interface principale du portail.

1. Dans le volet Cloud Shell, saisissez les commandes suivantes pour cloner le référentiel GitHub contenant les fichiers de code pour cet exercice (saisissez la commande, ou copiez-la dans le presse-papiers puis effectuez un clic droit dans la ligne de commande pour la coller en texte brut) :

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **Conseil** : lorsque vous collez des commandes dans cloudshell, la sortie peut prendre une grande quantité de mémoire tampon d’écran. Vous pouvez effacer le contenu de l’écran en saisissant la commande `cls` pour faciliter le focus sur chaque tâche.

1. Une fois le référentiel cloné, accédez au dossier qui contient le fichier de code de l’application pour cet exercice :  

    ```
   cd mslearn-ai-vision/Labfiles/video-indexer
    ```

### Obtenir les détails de l’API

Pour utiliser l’API Video Indexer, vous avez besoin de quelques informations afin d’authentifier les requêtes :

1. Dans le portail Video Indexer, développez le volet gauche et sélectionnez la page **Paramètres du compte**.
1. Notez l’**ID de compte** sur cette page. Vous en aurez besoin ultérieurement.
1. Ouvrez un nouvel onglet de navigateur et accédez au [portail des développeurs Video Indexer](https://api-portal.videoindexer.ai) sur https://api-portal.videoindexer.ai en vous connectant avec vos informations d’identification Azure.
1. Dans la page **Profil**, affichez les **Abonnements** associés à votre profil.
1. Sur la page présentant vos abonnements, notez que vous avez reçu deux clés (primaire et secondaire) pour chaque abonnement. Sélectionnez ensuite **Afficher** sur l’une des clés pour la voir. Vous aurez besoin de cette clé sous peu.

### Utiliser l’API REST

Maintenant que vous disposez de l’ID de compte et d’une clé API, vous pouvez utiliser l’API REST pour travailler avec des vidéos dans votre compte. Dans cette procédure, vous allez utiliser un script PowerShell pour effectuer des appels REST ; mais les mêmes principes s’appliquent avec des utilitaires HTTP tels que cURL ou Postman, ou tout langage de programmation capable d’envoyer et de recevoir du code JSON sur HTTP.

Toutes les interactions avec l’API REST Video Indexer suivent le même modèle :

- Une demande initiale à la méthode **AccessToken** avec la clé API dans l’en-tête est utilisée pour obtenir un jeton d’accès.
- Les demandes suivantes utilisent le jeton d’accès pour s’authentifier lors de l’appel de méthodes REST pour utiliser des vidéos.

1. Dans Cloud Shell, utilisez la commande suivante pour ouvrir le script PowerShell :

    ```
   code get-videos.ps1
    ```
    
1. Dans le script PowerShell, remplacez les espaces réservés **YOUR_ACCOUNT_ID** et **YOUR_API_KEY** par les valeurs de l’ID de compte et de la clé API que vous avez identifiées précédemment.
1. Notez que l’*emplacement* d’un compte gratuit est « essai ». Si vous avez créé un compte Video Indexer illimité (avec une ressource Azure associée), vous pouvez le remplacer par l’emplacement où votre ressource Azure est approvisionnée (par exemple, « eastus »).
1. Passez en revue le code dans le script, en notant que deux méthodes REST sont appelées : une pour obtenir un jeton d’accès et une autre pour répertorier les vidéos dans votre compte.
1. Enregistrez vos modifications (appuyez sur *CTRL+S*), fermez l’éditeur de code (appuyez sur *CTRL+Q*), puis exécutez la commande suivante pour exécuter le script :

    ```
   ./get-videos.ps1
    ```
    
1. Affichez la réponse JSON du service REST, qui doit contenir les détails de la vidéo **IA responsable** que vous avez indexée précédemment.

## Utiliser les widgets de Video Indexer

Le portail Video Indexer constitue une interface utile pour gérer les projets d’indexation vidéo. Toutefois, il peut arriver que vous souhaitiez rendre la vidéo et ses insights accessibles aux personnes qui n’ont pas accès à votre compte Video Indexer. Video Indexer fournit des widgets que vous pouvez incorporer dans une page web à cet effet.

1. Utilisez la commande `ls` pour afficher le contenu du dossier **video-indexer**. Notez qu’il contient un fichier **analyze-video.html**. Il s’agit d’une page HTML de base à laquelle vous allez ajouter le **Lecteur** Video Indexer et des widgets **Insights**.
1. Entrez la commande suivante pour modifier le fichier :

    ```
   code analyze-video.html
    ```

    Le fichier s’ouvre dans un éditeur de code.
   
1. Notez la référence au script **vb.widgets.mediator.js** dans l’en-tête. Ce script permet à plusieurs widgets Video Indexer sur la page d’interagir les uns avec les autres.
1. Dans le portail Video Indexer, revenez à la page **Fichiers multimédias** et ouvrez votre vidéo **IA responsable**.
1. Sous le lecteur vidéo, sélectionnez **&lt;/&gt; Incorporer** pour afficher le code iframe HTML afin d’incorporer les widgets.
1. Dans la boîte de dialogue **Partager et incorporer**, sélectionnez le widget **Lecteur**, définissez la taille de la vidéo sur 560 x 315, puis copiez le code incorporé dans le Presse-papiers.
1. Dans Cloud Shell du portail Azure, dans l’éditeur de code du fichier **analyze-video.html**, collez le code copié sous le commentaire **&lt;-- Emplacement du widget du lecteur ici -- &gt;**.
1. De retour dans le portail Video Indexer, dans la boîte de dialogue **Partager et incorporer**, sélectionnez le widget **Informations**, puis copiez le code incorporé dans le presse-papiers. Fermez ensuite la boîte de dialogue **Partager et Incorporer**, revenez au portail Azure, puis collez le code copié sous le commentaire **&lt;-- Emplacement du widget Informations ici -- &gt;**.
1. Après avoir modifié le fichier, dans l’éditeur de code, enregistrez vos modifications (*CTRL+S*), puis fermez l’éditeur de code (*CTRL+Q*) tout en gardant la ligne de commande Cloud Shell ouverte.
1. Dans la barre d’outils cloud shell, entrez la commande suivante (spécifique à Cloud Shell) pour télécharger le fichier HTML que vous avez modifié :

    ```
    download analyze-video.html
    ```

    La commande de téléchargement crée un lien contextuel en bas à droite de votre navigateur, que vous pouvez sélectionner pour télécharger et ouvrir le fichier qui devrait ressembler à ceci :

    ![Widgets Video Indexer dans une page web](../media/video-indexer-widgets.png)

1. Testez les widgets, en utilisant le widget **Insights** pour rechercher des insights et y accéder dans la vidéo.

