# Mémo libGDX

par *flashjaysan*

## Créer un projet libGDX

Les développeurs de libGDX fournissent un fichier *jar* exécutable permettant de paramétrer un projet utilisant libGDX.

- Télécharger le générateur de projet de libGDX (fichier `gdx-setup.jar`) sur le [site officiel de libGDX](https://libgdx.badlogicgames.com/download.html).

**Attention !** Java 13 n'est pas compatible avec ce générateur de projet. J'ai pu le faire fonctionner avec Java 11 mais on m'a conseillé d'installer et d'utiliser de préférence Java 8.

- Exécutez le fichier `gdx-setup.jar`.
- Dans le libGDX Project Setup, saisissez les informations nécessaires :
  - Le champ `Name` correspond au nom de votre jeu.
  - Le champ `Package` correspond au package du projet.
  - Le champ `Game Class` correspond au nom de la classe racine de votre jeu.
  - Le champ `Destination` correspond à l'emplacement sur disque de votre projet.
  - Le champ `Android SDK` correspond à l'emplacement sur disque où se trouve le SDK Android.
- La section `Sub Projects` vous permet de définir sur quelles plateformes cibles vous souhaitez exporter votre jeu. Vous pouvez tout cocher ou ne cocher qu'une seule plateforme.
- La section `Extensions` vous propose d'ajouter au projet des extensions fréquemment utilisées. Vous pouvez tout décocher pour commencer.
- Cliquez sur le bouton `Generate`.

**Remarque :** Pour déterminer l'emplacement du SDK Android, vous pouvez utiliser Android Studio :

- Ouvrez Android Studio.
- Cliquez sur le menu deroulant `Configure -> SDK Manager`.
- La section `Appearance & Behavior -> System Settings -> Android SDK` indique l'emplacement du SDK Android.

## Ouvrir un projet libGDX dans Android Studio

- Sur l'écran d'accueil d'Android Studio, cliquez sur l'option `Import project (gradle, eclipse adt, etc.)`.
- Sélectionnez le dossier contenant le projet libGDX.

## Exécuter un projet

### Windows

**Attention !** Vous devez avoir paramétré le projet en cochant `Desktop` dans la section `Sub Projects`.

#### Créer une configuration pour Windows

- Cliquez sur le menu `Run-> Edit Configurations`.
- Cliquez sur le bouton `+`.
- Sélectionnez `Application`.
- Dans le champ `Name`, saisissez `Desktop`.
- Dans le champ `Use classpath of module`, sélectionnez le dossier `desktop` (ou `desktop_main`).
- Cliquez sur le bouton `...` du champ `Main class`.
- Sélectionnez la classe `DesktopLauncher`.
- Dans le champ `Working directory`, sélectionnez le dossier `android/assets/` ou `core/assets/` du projet.
- Cliquez sur le bouton `Apply` puis sur le bouton `OK`.

#### Exécuter le projet

- Cliquez sur le bouton `Open edit Run/Debug configurations dialog` et sélectionnez la configuration `Desktop`.
- Cliquez sur le bouton `Run (Desktop)`.

### Android

**Attention !** Vous devez avoir paramétré le projet en cochant `Android` dans la section `Sub Projects`.

### IOS

**Attention !** Vous devez avoir paramétré le projet en cochant `Ios` dans la section `Sub Projects`.

Before you can run on iOS you need to install the RoboVM IntelliJ IDEA plugin or Intel Multi-OS Engine. Run -> Edit Configurations..., click the plus (+) button and select RoboVM iOS. Set the Name to iOS. The Module should be set to ios if not already done. To run on the simulator select Simulator as radio button and select a Device. Click Apply and Run If you run on a device, you need to provision it to be able to deploy to it!

### HTML

**Attention !** Vous devez avoir paramétré le projet en cochant `Html` dans la section `Sub Projects`.

View -> Tool Window -> Terminal, in the terminal, make sure you are in the root folder of your project. Then execute gradlew.bat html:superDev (Windows) or ./gradlew html:superDev (Linux, Mac OS X). This will take a while, as your Java code is compiled to Javascript. Once you see the message The code server is ready, fire up your browser and go to http://localhost:8080/index.html. This is your app running in the browser! When you change any of your Java code or assets, just click the SuperDev refresh button while you are on the site and the server will recompile your code and reload the page! To kill the process, simply press CTRL + C in the terminal window. 

IMPORTANT The port 9876 is used when doing normal GWT development. Therefore, trying to access the game in any other way than using the port 8080 will not launch your project.

Once this bug in the Gradle tooling API is fixed, we can simplify running the HTML5 by using the Gradle integration. At the moment, the Gradle process will run forever even if canceled.

## Debugger votre projet

Follow the steps for running the project, but instead of launching via the run (Play) button, launch your configuration via the debug (bug) button. Debugging the HTML5 project is a bit more involved.

Run the superDev Gradle task as before. Go to http://localhost:8080/html, click on the SuperDev Refresh button and hit Compile. In Chrome, press F12 to bring up the developer tools, go to the sources tab and find the Java file you want to debug. Set breakpoints, step and inspect variables using the power of source maps!

## Exporter votre projet

[A COMPLETER]
It's easiest to package your application from the command line, or using the Gradle task within IntelliJ IDEA. To see the relevant Gradle tasks, check the Gradle command line documentation.

## Structure de libGDX

### Modules

libGDX est composé de six interfaces qui fournissent les fonctionnalités pour interagir avec le système de la plateforme cible. Chaque plateforme cible possède une implémentation spécifique de ces interfaces. 

- **Application :** exécute l'application et transmet à l'API cliente les évènements de niveau application. Fournit des fonctionnalités d'identification et de requètes.
- **Files :** expose le système de fichiers sous jacent de la plateforme cible. Fournit une abstraction sur différent types d'emplacements de fichiers par dessus un système de gestion de fichier personalisé. N'est pas compatible avec la classe `File` Java.
- **Input:** informe l'API cliente des entrées utilisateur. Les évènements et le sondage sont pris en charge.
- **Net:** fournit les moyens d'accéder à des ressources via HTTP/HTTPS d'une manière cross-plateforme ainsi que la création de serveur TCP et de sockets client.
- **Audio:** fournit les moyens de lire des effets audio et de la music en streaming ainsi que l'accès direct au matériel audio pour les entrées sorties PCM.
- **Graphics:** expose OpenGL ES 2.0 (quand il est disponible) et permet la récupération et le paramétrage des modes videos et d'autres choses similaires.

### Classes de démarrage

Le seul code spécifique à une plateforme cible que vous devez écrire est contenu dans les classes de démarrage. Pour chaque plateforme cible, un bout de code va instancier une implémentation concrète de l'interface `Application`, fournie par les classes de bas niveau pour la plateforme.

Pour Windows et Mac OS cela peut ressembler au code suivant (qui utilise la couche *Lwjgl*) :

```java
public class DesktopStarter {
   public static void main(String[] argv) {
      LwjglApplicationConfiguration config = new LwjglApplicationConfiguration();
      new LwjglApplication(new MyGame(), config);
   }
}
```

Pour Android, la classe de démarrage ressemblera plutôt au code suivant :

```java
public class AndroidStarter extends AndroidApplication {
   public void onCreate(Bundle bundle) {
      super.onCreate(bundle);
      AndroidApplicationConfiguration config = new AndroidApplicationConfiguration();
      initialize(new MyGame(), config);
   }
}
```

Ces deux classes vivent généralement dans des projets séparés (par exemple un projet `Desktop` et un projet `Android`).

Le code réel de l'application est situé dans une classe qui implémente l'interface `ApplicationListener` (`MyGame` dans les exemples précédents). Une instance de cette classe est passée aux méthodes d'initialisation respectives à chaque implémentation bas-niveau de l'`Application`. L'application va ensuite appeler les méthodes de l'interface `ApplicationListener` au moment approprié.

### Accès aux modules

Les modules décrits précédemment peuvent être accéder via des champs statiques de la classe `Gdx`. C'est essentiellement un jeu de variables globales qui permettent un accès facile à n'importe quel module de libGDX. Généralement considéré comme une mauvaise pratique, nous avons décidé d'utiliser ce mécanisme pour alléger la difficulté associée avec le passage de références aux choses utilisées souvent dans tous les types d'endroits dans le code.

Par exemple, pour accéder au module `audio`, utilisez simplement le code suivant :

```java
// creates a new AudioDevice to which 16-bit PCM samples can be written
AudioDevice audioDevice = Gdx.audio.newAudioDevice(44100, false);
```

`Gdx.audio` est une référence à l'implémentation bas-niveau qui a été instanciée au démarrage de l'application par l'instance de l'interface `Application`. Les autres modules sont accessibles de la même manière (par exemple, `Gdx.app` pour ccéder à l'implémentation du module `Application`, `Gdx.files` pour accéder à l'implémentation du module `Files`, etc...).

## Cycle de vie d'un jeu avec libGDX

Une application libGDX a un cycle de vie bien défini qui gouverne les états d'une application tels que la création, la pause, la reprise, le rendu et la destruction d'une application.

### ApplicationListener

Un développeur d'application exploite ces évènements du cycle de vie en implémentant l'interface `ApplicationListener` et en passant une instance de cette implémentation à l'implémentation concrète de l'interface `Application` d'une plateforme spécifique. De là, l'interface `Application` appellera l'`ApplicationListener` à chaque fois qu'un évènement de niveau application surviendra. Une implémentation basique d'`ApplicationListener` pourrait ressembler au code suivant :

```java
public class MyGame implements ApplicationListener {
   public void create () {
   }

   public void render () {        
   }

   public void resize (int width, int height) { 
   }

   public void pause () { 
   }

   public void resume () {
   }

   public void dispose () { 
   }
}
```

**Remarque :** Vous pouvez également créer une classe dérivée de la classe `ApplicationAdapter` si toutes les méthodes de l'interface ne sont pas utiles.

Une fois passée à l'`Application`, les méthodes de l'`ApplicationListener` seront appelées comme suit :

- `create()` : Méthode appelée une seule fois lorsque l'application est créée.
- `resize(int width, int height)` : Méthode appelée chaque fois que l'écran de jeu est redimensionné et que le jeu n'est pas dans un état de pause. Elle est également appelée une fois juste après la méthode `create()`. Les paramètres correspondent à la nouvelle largeur et hauteur de l'écran de jeu redimensionné (en pixels).
- `render()` : Méthode appelée par la boucle de jeu depuis l'application chaque fois qu'un rendu doit être exécuté. Les mises à jour de la logique de jeu sont généralement exécutées dans cette méthode.
- `pause()` : Sur Android, cette méthode est appelée lorsque le bouton `Home` est appuyé ou si un appel est reçu. Sur ordinateur, elle est appelée juste avant la méthode `dispose()` lorsque le jeu se termine. Une bon endroit pour sauvegarder l'état du jeu.
- `resume()` : Cette méthode est seulement appelée sur Android, lorsque l'application reprend après un état de pause.
- `dispose()` : Méthode appelée lorsque l'application est détruite. Elle est précédée d'un appel à la méthode `pause()`.

Le diagramme suivant illustre visuellement le cycle de vie :

![diagramme du cycle de vie](libgdx_diagramme_cycle_de_vie.png)

### Où est la boucle principale ?

LibGDX est basé sur les évènements par nature. Cela est principalement dû à la façon dont fonctionnent *Android* et *JavaScript*. Une boucle principale explicite n'existe pas. Cependant la méthode `ApplicationListener.render()` peut être considérée comme le corps d'une telle boucle.
