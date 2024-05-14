---
lab:
  title: Analyser des vidéos avec Video Indexer
  module: Module 8 - Getting Started with Azure AI Vision
---

# Analyser des vidéos avec Video Indexer

Une grande proportion des données créées et consommées aujourd’hui est au format vidéo. **Azure AI Video Indexer** est un service basé sur l’IA que vous pouvez utiliser pour indexer des vidéos et en extraire des insights.

> **Remarque** : depuis le 21 juin 2022, les fonctionnalités d’Azure AI services qui retournent des informations d’identification personnelle sont limitées aux clients qui ont reçu [un accès limité](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access). Sans approbation d’accès limité, la reconnaissance des personnes et des célébrités avec Video Indexer pour ce labo n’est pas disponible. Pour plus d’informations sur les modifications apportées par Microsoft, et leur motif, consultez [Responsible AI investments and safeguards for facial recognition](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/) (Investissements responsables en matière d'IA et mesures de protection pour la reconnaissance faciale).

## Cloner le référentiel pour ce cours

Si vous avez récemment cloné le référentiel de code **mslearn-ai-vision** dans l’environnement dans lequel vous travaillez sur ce labo, ouvrez-le dans Visual Studio Code ; sinon, procédez comme suit pour le cloner maintenant.

1. Démarrez Visual Studio Code.
2. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git : Cloner** pour cloner le référentiel `https://github.com/MicrosoftLearning/mslearn-ai-vision` vers un dossier local (peu importe quel dossier).
3. Lorsque le référentiel a été cloné, ouvrez le dossier dans Visual Studio Code.
4. Attendez que des fichiers supplémentaires soient installés pour prendre en charge les projets de code C# dans le référentiel.

    > **Remarque** : si vous êtes invité à ajouter des ressources requises pour générer et déboguer, sélectionnez **Not Now** (Pas maintenant).

## Charger une vidéo sur Video Indexer

Tout d’abord, vous devrez vous connecter au portail Video Indexer et charger une vidéo.

> **Conseil** : Si la page Video Indexer est lente à charger dans l’environnement de labo hébergé, utilisez votre navigateur installé localement. Vous pourrez revenir à la machine virtuelle hébergée pour les tâches ultérieures.

1. Dans votre navigateur, ouvrez le portail Video Indexer à l’adresse `https://www.videoindexer.ai`.
2. Si vous disposez d’un compte Video Indexer existant, connectez-vous. Sinon, créez un compte gratuit et connectez-vous à l’aide de votre compte Microsoft (ou tout autre type de compte valide). Si vous ne parvenez pas à vous connecter, essayez d’ouvrir une session de navigation privée.
3. Dans un nouvel onglet, téléchargez la vidéo IA responsable en visitant `https://aka.ms/responsible-ai-video`. Enregistrez le fichier.
4. Dans Video Indexer, sélectionnez l’option **Charger**. Sélectionnez ensuite l’option permettant de **Parcourir les fichiers**, sélectionnez la vidéo téléchargée, puis cliquez sur **Ajouter**. Remplacez le nom par défaut par **IA responsable**, passez en revue les paramètres par défaut, cochez la case pour vérifier la conformité avec les politiques de Microsoft concernant la reconnaissance faciale, puis chargez le fichier.
5. Une fois le fichier chargé, patientez quelques minutes pendant que Video Indexer l’indexe automatiquement.

> **Remarque** : Dans cet exercice, nous utilisons cette vidéo pour explorer les fonctionnalités de Video Indexer, mais vous devriez prendre le temps de la regarder entièrement lorsque vous aurez terminé l’exercice, car elle contient des informations et des conseils utiles relatifs au développement responsable d’applications compatibles avec l’IA. 

## Passer en revue les insights vidéo

Le processus d’indexation extrait des insights de la vidéo, que vous pouvez afficher dans le portail.

1. Dans le portail Video Indexer, une fois la vidéo indexée, sélectionnez-la pour l’afficher. Vous verrez le lecteur vidéo ainsi qu’un volet qui affiche les insights extraits de la vidéo.

    > **Remarque** : en raison de la stratégie d’accès limité qui protège les identités des personnes, vous ne voyez peut-être pas de noms lorsque vous indexez la vidéo.

![Video Indexer avec un lecteur vidéo et un volet Insights](../media/video-indexer-insights.png)

2. Durant la lecture de la vidéo, sélectionnez l’onglet **Chronologie** pour afficher une transcription audio de la vidéo.

![Video Indexer avec un lecteur vidéo et un volet Chronologie affichant la transcription de la vidéo.](../media/video-indexer-transcript.png)

3. En haut à droite du portail, sélectionnez le symbole **Affichage** (qui ressemble à &#128455;), puis, dans la liste des insights, en plus de **Transcription**, sélectionnez **OCR** et **Haut-parleurs**.

![Menu d’affichage de Video Indexer avec Transcription, OCR et Haut-parleurs sélectionnés](../media/video-indexer-view-menu.png)

4. Notez que le volet **Chronologie** comprend désormais :
    - Une transcription de la narration audio.
    - Le texte visible dans la vidéo.
    - Des indications des intervenants qui apparaissent dans la vidéo. Certaines personnes connues sont automatiquement reconnues par leur nom, d’autres sont indiquées par un numéro (par exemple *Speaker #1*).
5. Revenez au volet **Insights** et affichez les insights. Notamment :
    - Les personnes individuelles qui apparaissent dans la vidéo.
    - Les sujets abordés dans la vidéo.
    - Les étiquettes des objets qui apparaissent dans la vidéo.
    - Les entités nommées, telles que les personnes et les marques qui apparaissent dans la vidéo.
    - Les scènes clés.
6. Avec le volet **Insights** visible, sélectionnez à nouveau le symbole **Affichage**, puis, dans la liste des insights, ajoutez **Mots clés** et **Sentiments** au volet.

    Les insights trouvés peuvent vous aider à déterminer les principaux thèmes de la vidéo. Par exemple, les **sujets** de cette vidéo montrent clairement qu’elle porte sur la technologie, la responsabilité sociale et l’éthique.

## Rechercher des insights

Vous pouvez utiliser Video Indexer pour rechercher des insights dans la vidéo.

1. Dans le volet **Insights**, dans la zone **Recherche**, saisissez *Abeille*. Vous devrez peut-être défiler vers le bas dans le volet Insights pour afficher les résultats de tous les types d’insights.
2. Notez qu’une *étiquette* correspondante est trouvée, avec son emplacement dans la vidéo indiquée en dessous.
3. Sélectionnez le début de la section où la présence d’une abeille est indiquée et affichez la vidéo à ce moment-là (vous devrez peut-être suspendre la vidéo et sélectionner soigneusement ; l’abeille apparaît seulement brièvement !)
4. Désactivez la zone **Recherche** pour afficher tous les insights de la vidéo.

![Résultats de la recherche Video Indexer pour Bee](../media/video-indexer-search.png)

## Utiliser les widgets de Video Indexer

Le portail Video Indexer constitue une interface utile pour gérer les projets d’indexation vidéo. Toutefois, il peut arriver que vous souhaitiez rendre la vidéo et ses insights accessibles aux personnes qui n’ont pas accès à votre compte Video Indexer. Video Indexer fournit des widgets que vous pouvez incorporer dans une page web à cet effet.

1. Dans Visual Studio Code, dans le dossier **06-video-indexer**, ouvrez **analyze-video.html**. Il s’agit d’une page HTML de base à laquelle vous allez ajouter le **Lecteur** Video Indexer et des widgets **Insights**. Notez la référence au script **vb.widgets.mediator.js** dans l’en-tête. Ce script permet à plusieurs widgets Video Indexer sur la page d’interagir les uns avec les autres.
2. Dans le portail Video Indexer, revenez à la page **Fichiers multimédias** et ouvrez votre vidéo **IA responsable**.
3. Sous le lecteur vidéo, sélectionnez **&lt;/&gt; Incorporer** pour afficher le code iframe HTML afin d’incorporer les widgets.
4. Dans la boîte de dialogue **Partager et incorporer**, sélectionnez le widget **Lecteur**, définissez la taille de la vidéo sur 560 x 315, puis copiez le code incorporé dans le Presse-papiers.
5. Dans Visual Studio Code, dans le fichier **analyze-video.html**, collez le code copié sous le commentaire **&lt;-- Emplacement du widget Lecteur -- &gt;** .
6. De retour dans la boîte de dialogue **Partager et incorporer**, sélectionnez le widget **Insights**, puis copiez le code incorporé dans le Presse-papiers. Fermez ensuite la boîte de dialogue **Partager et Incorporer**, revenez à Visual Studio Code, puis collez le code copié sous le commentaire **&lt;-- Emplacement du widget Insights -- &gt;** .
7. Enregistrez le fichier . Ensuite, dans le volet **Explorateur**, cliquez avec le bouton droit sur **analyze-video.html**, puis sélectionnez **Révéler dans l’Explorateur de fichiers**.
8. Dans l’Explorateur de fichiers, ouvrez **analyze-video.html** dans votre navigateur pour afficher la page web.
9. Testez les widgets, en utilisant le widget **Insights** pour rechercher des insights et y accéder dans la vidéo.

![Widgets Video Indexer dans une page web](../media/video-indexer-widgets.png)

## Utiliser l’API REST Video Indexer

Video Indexer fournit une API REST que vous pouvez utiliser pour charger et gérer des vidéos dans votre compte.

### Obtenir les détails de l’API

Pour utiliser l’API Video Indexer, vous avez besoin de quelques informations afin d’authentifier les requêtes :

1. Dans le portail Video Indexer, développez le volet gauche et sélectionnez la page **Paramètres du compte**.
2. Notez l’**ID de compte** sur cette page. Vous en aurez besoin ultérieurement.
3. Ouvrez un nouvel onglet de navigateur et accédez au portail des développeurs Video Indexer à l’adresse `https://api-portal.videoindexer.ai`. Connectez-vous à l’aide des informations d’identification de votre compte Video Indexer.
4. Dans la page **Profil**, affichez les **Abonnements** associés à votre profil.
5. Sur la page présentant vos abonnements, notez que vous avez reçu deux clés (primaire et secondaire) pour chaque abonnement. Sélectionnez ensuite **Afficher** sur l’une des clés pour la voir. Vous aurez besoin de cette clé sous peu.

### Utiliser l’API REST

Maintenant que vous disposez de l’ID de compte et d’une clé API, vous pouvez utiliser l’API REST pour travailler avec des vidéos dans votre compte. Dans cette procédure, vous allez utiliser un script PowerShell pour effectuer des appels REST ; mais les mêmes principes s’appliquent avec des utilitaires HTTP tels que cURL ou Postman, ou tout langage de programmation capable d’envoyer et de recevoir du code JSON sur HTTP.

Toutes les interactions avec l’API REST Video Indexer suivent le même modèle :

- Une demande initiale à la méthode **AccessToken** avec la clé API dans l’en-tête est utilisée pour obtenir un jeton d’accès.
- Les demandes suivantes utilisent le jeton d’accès pour s’authentifier lors de l’appel de méthodes REST pour utiliser des vidéos.

1. Dans Visual Studio Code, dans le dossier **06-video-indexer**, ouvrez **get-videos.ps1**.
2. Dans le script PowerShell, remplacez les espaces réservés **YOUR_ACCOUNT_ID** et **YOUR_API_KEY** par les valeurs de l’ID de compte et de la clé API que vous avez identifiées précédemment.
3. Notez que l’*emplacement* d’un compte gratuit est « essai ». Si vous avez créé un compte Video Indexer illimité (avec une ressource Azure associée), vous pouvez le remplacer par l’emplacement où votre ressource Azure est approvisionnée (par exemple, « eastus »).
4. Passez en revue le code dans le script, en notant que deux méthodes REST sont appelées : une pour obtenir un jeton d’accès et une autre pour répertorier les vidéos dans votre compte.
5. Enregistrez vos modifications, puis, en haut à droite du volet de script, utilisez le bouton **&#9655;** pour exécuter le script.
6. Affichez la réponse JSON du service REST, qui doit contenir les détails de la vidéo **IA responsable** que vous avez indexée précédemment.

## Plus d’informations

La reconnaissance des personnes et des célébrités reste disponible. Toutefois, conformément à la [Norme de l’IA responsable](https://aka.ms/aah91ff), celle-ci est limitée par une stratégie d’accès limité. Ces caractéristiques incluent l’identification faciale et la reconnaissance des célébrités. Pour en savoir plus et demander l’accès, consultez la section[Accès limité pour Azure AI Services](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access).

Pour plus d’informations sur **Video Indexer**, consultez la [documentation Video Indexer](https://learn.microsoft.com/azure/azure-video-indexer/).
