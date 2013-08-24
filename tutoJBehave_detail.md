#Tutoriel JBehave : Prise en main détaillée - JDEV2013 


**JBehave** a été le premier framework Behavior Driven Development développé par Dan North, initiateur du concept BDD. 
JBehave permet de transformer les scénarios d'une story (descriptions de fonctionnalités écrites en «langage naturel») en tests automatisés. Dans le principe, cette transformation est possible à l'aide du framework de tests [JUnit](http://junit.org/) : chaque phrase de la description textuelle d'un scénario (une étape du scénario)  sera implémentée comme une méthode dans un fichier de tests. Le lien entre la description textuelle et le code Java est réalisé via des annotations.
JBehave est un framework destiné à Java et aux langages de la JVM tels que Groovy, Scala,... 
Il est disponible sur : [http://Jbehave.org](http://jbehave.org)

Ce tutoriel va vous permettre de mettre en place pas à pas un projet BDD sous Jbehave. Une prise en main rapide de JBehave est proposée [ici]()


## Installation préalable du plug-in JBehave pour Eclipse

JBehave fournit un plugin Eclipse avec un mode éditeur JBehave personnalisé. La documentation est disponible sur : [http://jbehave.org/eclipse-integration.html](http://jbehave.org/eclipse-integration.html). Ce plug-in propose entre autres une coloration syntaxique, une détection de lien hypertexte étape et un lien vers méthode Java correspondante, une autocomplétion, une validation des étapes.

Dans le cadre de ce tutoriel, nous utiliserons donc la version d' **Eclipse IDE for Java Developers** qui peut être téléchargée depuis [http://www.eclipse.org/downloads/] (http://www.eclipse.org/downloads/)

*Remarque* : La documentation sur le plugin JBehave pour Eclipse indique que ce dernier a pour l'instant été testé sur Eclipse Indigo et Juno, avec une JDK 1.6 ou supérieur

Pour mettre en place ce plug-in, nous utiliserons l'installeur de plug-in d'Eclipse disponible à partir du menu **Help → Install New Software...**
Il suffit  alors d'ajouter le le site  [http://jbehave.org/reference/eclipse/updates/](http://jbehave.org/reference/eclipse/updates/). Cochez ensuite la case **Jbehave Eclipse** et continuez de suivre les instructions d'installation classique de plug-in. 
A la fin de l'installation, Eclipse doit redémarrer pour prendre en compte le nouveau plug-in.


##  Mise en place d'un premier projet BDD :

La mise en place d'un projet BDD sous JBehave se fera en plusieurs étapes :

1. Création d'un projet Maven
2. Ajout de dépendances Maven (`pom.xml`)
3. Description textuelle d'une story et de ses scénarios exécutables dans un fichier `nom_story.story`
4. Configuration de l'environnement de tests des scénarios (fichier java)
(Implémentation d'un lanceur héritant de `JunitStory` ou `JunitStories` permettant de  lier les scénarios et le code)
5. Implémentation Java de chaque étape des scénarios de la story dans un fichier `NomStorySteps.java`


### Création d'un projet Maven

Créez un nouveau projet Maven sous Eclipse : **File → New → Project... → Maven → Maven Project**

Vérifiez que **maven-archetype-quickstart** soit bien sélectionné.

Saisissez un **groupId** (identifiant de groupe): `fr.cnrs.devlog.jdev` par exemple.
Saisissez un **artifactId** (identifiant de composant) : `demobdd` par exemple. L'artifactId correspond au nom unique du projet au sein du groupe qui le développe.


###  Mise à jour des dépendances Maven

Mettre à jour votre `pom.xml` :

- en rajoutant les propriétés `maven.compiler.source` et `maven.compiler.target` avec la version Java 1.7 
- en vérifiant la version utilisée pour [JUnit](http://junit.org/)
- en rajoutant une dépendance sur JBehave. Pour connaître la dernière version de JBehave, rendez-vous sur : [http://JBehave.org/download.html](http://JBehave.org/download.html) puis cliquez sur [Core Distribution](https://nexus.codehaus.org/content/repositories/releases/org/JBehave/JBehave-distribution/)

```XML
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>fr.cnrs.devlog.jdev</groupId>
	<artifactId>demobdd</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>demobdd</name>
	<url>http://maven.apache.org</url>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<maven.compiler.source>1.7</maven.compiler.source>
		<maven.compiler.target>1.7</maven.compiler.target>
		<JBehave-core.version>3.8</JBehave-core.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.11</version>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.jbehave</groupId>
			<artifactId>jbehave-core</artifactId>
			<version>${JBehave-core.version}</version>
		</dependency>
	</dependencies>
</project>
```

*Remarque* : Sous Eclipse, une modification du `pom.xml` peut nécessiter une mise à jour du projet. Placez-vous sur le projet dans la vue Package Explorer, clic droit puis **Maven → Update Project** et cochez **Force Update et Snapshots/Releases**.

### Description textuelle d'une Story et de ses scénarios exécutables 

#### De la story « orale » à la story «textuelle» sous JBehave
Ron Jeffries définit les 3 composantes d’une story comme le modèle des 3C :  Carte, Conversation, Confirmation :
- chaque User story doit être écrite de manière synthétique sur une **Carte**  et respecter le formalisme suivant :
`As a <type of user>`   `I want <goal>`   `so that <reason>` 
- chaque User Story doit faire l'objet de discussions au sein de l'équipe : une **Conversation** doit s'instaurer.
- ces discussions ont pour but d'identifier les différents scénarios de la story.  Des critères d'acceptance, en rapport avec les différents scénarios identifiés, sont alors définis afin de permettre la **Confirmation** de la story.

Le BDD se place dans le cadre de la **spécification par l’exemple**. Chaque scénario doit être illustré par un exemple concret afin que les règles métiers soient moins abstraites et mieux comprises par tous. La description de chaque exemple se fait sur un modèle commun qui consiste à décomposer le scénario en plusieurs étapes successives :
- Tout d'abord définir et construire le contexte initial  dans lequel le scénario va se dérouler ( *Etant donné que, Soit* ),
- ensuite provoquer des événements ou des actions sollicitant le système ( *Quand* )
- et afin vérifier que le comportement attendu a bien eu lieu ( *Alors* )

Voici un exemple de story qui décrit le concept du BDD en utilisant la syntaxe JBehave
(extrait de [http://JBehave.org/reference/stable/story-syntax.html](http://JBehave.org/reference/stable/story-syntax.html)) :

```GHERKIN
A story is a collection of scenarios

Narrative:
In order to communicate effectively to the business some functionality
As a development team
I want to use Behaviour-Driven Development

Scenario: A scenario is a collection of executable steps of different type
Given step represents a precondition to an event
When step represents the occurrence of the event
Then step represents the outcome of the event

Scenario: Another scenario exploring different combination of events
Given a [precondition]
When a negative event occurs
Then a the outcome should [be-captured]

Examples: 
|precondition|be-captured|
|abc|be captured |
|xyz|not be captured| 
```

Dans **JBehave**, une **story** est composée de **scénarios** et chaque scénario est composé d' **étapes** (step) :
- la description d'une story est introduite par le mot clé `Narrative` suivi de 3 éléments de narration  `In order to` , `As a` et `I want to`
- chaque scénario est identifié par le mot clé `Scenario`. Ce mot clé peut être suivi ou non d'un titre qui décrit explicitement le critère d'acceptance de la story associé à ce scénario. Le mot clé `Scenario` est optionnel si la story n'est composé que d'un seul scénario.
- les différentes étapes (steps) du scénario sont décrites à partir des 3 mots clés : `Given`, `When` et `Then`.

JBehave propose quelques mots clés supplémentaires comme `And`,`Exemples`,`GivenStories`,`Meta` et `Step`. 
La description complète de la grammaire de JBehave est consultable sur [http://JBehave.org/reference/stable/grammar.html](http://JBehave.org/reference/stable/grammar.html)

La syntaxe de JBehave est supportée par la classe [`RegexStoryParser`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/parsers/RegexStoryParser.html) du paquetage `org.JBehave.core.parsers`

La documentation sur les différentes classes disponibles dans le framework JBehave est disponible sur : [http://JBehave.org/reference/stable/javadoc.html](http://JBehave.org/reference/stable/javadoc.html)


**JBehave** a été écrit par Dan North  et a été le premier framework BDD.
Par abus de langage, le formalisme `Given` `When` `Then` est  appelé langage **Gherkin**. Gherkin est le langage utilisé par le framework **Cucumber** destiné à l'origine à Ruby, mais qui est également aujourd'hui très utilisé en Java via [Cucumber-JVM](https://github.com/cucumber/cucumber-jvm).

Le site officiel du framework Cucumber est : [http://cukes.info/](http://cukes.info/)
Une description de la syntaxe du langage Gherkhin est disponible sur [https://github.com/cucumber/cucumber/wiki/Gherkin](https://github.com/cucumber/cucumber/wiki/Gherkin).

Le framework JBehave supporte également la syntaxe Gherkin grâce à la classe [`GherkinStoryParser`](http://jbehave.org/reference/stable/javadoc/gherkin/org/jbehave/core/parsers/gherkin/GherkinStoryParser.html) du paquetage `org.jbehave.core.parsers.gherkin` 
La syntaxe JBehave et la synatxe du langage Gherkin sont très proches (comme le montre la partie [Story Syntax](http://JBehave.org/reference/stable/story-syntax.html) du tutoriel de JBehave).


#### Description de la user story à partir de la syntaxe JBehave : `nom_de_la_story.story` 

JBehave permet de transformer les scénarios d'une story en tests automatisés. La première étape consiste à écrire la story et ses scénarios dans un fichier `nom_de_la_story.story`

Pour vous familiariser avec JBehave, vous allez commencer par un exemple simple : **faire une addition à l'aide d'une calculatrice**.
Tout d'abord, vous allez devoir créer un fichier `calculatrice_addition.story` destiné à contenir la description textuelle de la story. Pour simplifier la prise en main de JBehave, ce fichier sera enregistré dans le dossier `src/test/java` (dans le paquetage  `fr.cnrs.devlog.jdev.demobdd`) 
Pour cela, deux solutions s'offrent alors à vous : soit créer ce fichier manuellement, soit utiliser le plug-in Jbehave installé précédemment.

Pour utiliser le plug-in JBehave, il vous suffit de vous placer dans la vue **Package Explorer** **Package Explorer** à l'emplacement où vous souhaitez enregistrer la story (package `fr.cnrs.devlog.jdev.demobdd` du dossier `src/test/java`pour cet exemple). Après un clic droit, choisissez : **New → Other... → Jbehave → New story** puis entrez le nom de la story **calculatrice_addition**. Un fichier `calculatrice_addition.story` contenant un template est alors créé et ouvert dans l'éditeur de scénarios, il vous suffit de le modifier afin qu'il contienne la description textuelle suivante : 

```Gherkin
Narrative:
In order to pouvoir faire des calculs le plus rapidement possible
As a utilisateur
I want to utiliser une calculatrice pour additionner deux nombres

Scenario: Addition de deux nombres valides
Given une calculatrice
When j'additionne 1 et 2 
Then le resultat doit être égal à 3
```

Une fois la story écrite, vérifiez que les mots clés de la syntaxe JBehave apparaissent bien en gras. A chaque étape du scénario que vous venez d'écrire, Eclipse indique des warnings de type *«No Step is matching»*, ce qui est normal puisque nous n'avons pas encore implémenté les méthodes Java correspondantes à chacune des étapes du scénario. Aucun lien hypertexte du fichier texte (`.story`) vers le fichier code (`.java`) n'est donc possible pour le moment.

### Configuration de l'environnement de tests des scénarios
Une fois, les scénarios décrits avec la syntaxe JBehave, il est donc nécessaire de s'interroger sur comment lancer ces scénarios de manière automatique et surtout comment visualiser les résultats obtenus, ce qui revient à configurer l'environnement de tests de nos scénarios.
Dans la partie [Running Stories](http://jbehave.org/reference/stable/running-stories.html), le tutoriel de JBehave indique plusieurs possibilités pour configurer l’environnement d'exécution de tests des scénarios. Dans tous les cas, la contrainte à respecter est que la classe qui lance les scénarios doit être un [`Embedder`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/embedder/Embedder.html). 

Il serait bien sûr possible d'écrire son propre [`Embedder`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/embedder/Embedder.html). , de l'exécuter dans un `main`(voir annexe 1) et de lire les résultats sur la console. Afin de simplifier le code, nous préférons utiliser directement un lanceur du [framework JUnit](http://junit.org/) ([`JUnitStory`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/junit/JUnitStory.html) ou [`JUnitStories`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/junit/JUnitStories.html)).
En effet JBehave propose une intégration de JUnit grâce au paquetage de base `org.jbehave.core.junit` et aux deux classes [`Embeddable`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/Embeddable.html):
- [`JUnitStory`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/junit/JUnitStory.html) qui permet de lancer une seule story
- [`JUnitStories`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/junit/JUnitStories.html) qui permet de lancer plusieurs stories,

*Remarque:* D'autres framework de tests tels que TestNG ou  Spring Test peuvent également être utilisé avec JBehave, mais leur intégration n'est pas de base comme pour JUnit.


#### Implémentation d'un simple lanceur héritant de [`JUnitStory`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/junit/JUnitStory.html)

Pour l'instant vous n'avez créé qu'une seule story, le lanceur qui hérite de [`JUnitStory`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/junit/JUnitStory.html) est donc suffisant pour configurer l'environnement de tests.
Attention par défaut le lanceur (fichier `.java`) supposera qu'il existe dans le même répertoire un fichier contenant les scénarios textuels (fichier `.story`) ayant le même nom mais avec une casse différente : les majuscules de la casse CamelCase du nom du fichier `.java` devenant des minuscules précédées d'un underscore (hormis la première) dans le nom du fichier `.story`.
Ainsi pour un lanceur nommé `NomStory.java`, JBehave cherchera, par défaut dans le même paquetage, à lancer le fichier de scénarios `nom_story.story`. S'il ne le trouve pas, il générera une erreur. Pour le lanceur, il est donc important de respecter la casse et l'emplacement lorsque [`JUnitStory`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/junit/JUnitStory.html) est utilisé.
Dans le package de `src/test/java` qui contient déjà le fichier de scénarios `calculatrice_addtition.story`, créez et implémentez le lanceur `CalculatriceAddition.java` de la manière suivante :

```JAVA
import org.jbehave.core.junit.JUnitStory;
public class CalculatriceAddition  extends JUnitStory {
	 public CalculatriceAddition() {
		super();
		this.configuredEmbedder().candidateSteps().add(new AdditionSteps());
	}
 }
```

Le lanceur permet de faire le mapping entre les étapes des scénarios écrites de manière textuelle (`calculatrice_addition.story`) et les méthodes Java implémentant ces étapes (`AdditionSteps.java`) 
Pour pouvoir exécuter le lanceur, vous devez maintenant créer la classe `AdditionSteps.java` de la manière suivante. N'oubliez pas l'héritage sur [`Steps`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/steps/Steps.html) (qui est une implémentation par défaut de l'interface [`CandidateSteps`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/steps/CandidateSteps.html)) !

```JAVA
import org.jbehave.core.steps.Steps;
public class AdditionSteps extends Steps {
}
```


Lancez maintenant la classe `CalculatriceAddition` comme un test unitaire ( **Run As →JUnit Test** )

Le test est au vert et la console nous indique :
```
Reports view generated with 1 stories (of which 1 pending) containing 1 scenarios (of which 1 pending)
```

Chaque test JBehave est lancé de manière indépendante. A la fin de chaque test, JBehave consolide les résultats dans un unique rapport. Les rapports sont un élément essentiel qui permettent de vérifier si les stories se sont correctement exécutées. Il est donc indispensable de pouvoir visualiser le contenu de ces rapports...


#### Visualisation du contenu des rapports Jbehave

La classe [StoryReporter](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/reporters/StoryReporter.html) est au coeur des rapports JBehave.
La classe [StoryReporterBuilder](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/reporters/StoryReporterBuilder.html) permet de configurer différents rapport avec des formats préconfigurés tels que : CONSOLE, TXT, HTML, HTML_TEMPLATE and XML.
La documentation sur les rapports est disponible dans la partie [Reporting Stories](http://jbehave.org/reference/stable/reporting-stories.html du site http://jbehave.org/)

##### Visualisation du contenu du rapport JBehave dans la console

Afin de visualiser le contenu du rapport dans la console, vous devez rajouter dans la classe `CalculatriceAddition.java` la méthode suivante :

```JAVA
@Override
public Configuration configuration() {
	  
	Configuration configuration = new MostUsefulConfiguration();
	// CONSOLE Reporting
	StoryReporterBuilder storyReporterBuilder;
	storyReporterBuilder = new StoryReporterBuilder();
	storyReporterBuilder.withDefaultFormats();
	storyReporterBuilder.withFormats(org.jbehave.core.reporters.Format.CONSOLE); 
	configuration.useStoryReporterBuilder(storyReporterBuilder);

	return configuration;
	 }
```

La classe [MostUsefulConfiguration](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/configuration/MostUsefulConfiguration.html) contient les configurations par défaut. 

Ajoutez à cette configuration de base un [StoryReporterBuilder](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/reporters/StoryReporterBuilder.html) personnalisé associé à la console. 

Exécutez la classe  `CalculatriceAddition.java` afin de vous assurer que vous visualisez bien désormais le contenu du rapport dans la console. Nous reviendrons sur le contenu dans la suite du tutoriel.

##### Visualisation du contenu du rapport JBehave dans la vue JUnit
La vue JUnit ne propose pour l'instant qu'une barre verte. Pour identifier à quelle étape un scénario échoue, il serait plus confortable de pouvoir visualiser chaque étape de chaque scénario comme un test spécifique directement dans la vue JUnit. Le projet [jbehave-junit-runner](https://github.com/codecentric/jbehave-junit-runner) permet d'obtenir ce résultat. Le site de ce projet est : [https://github.com/codecentric/jbehave-junit-runner] (https://github.com/codecentric/jbehave-junit-runner) 

Pour pouvoir utiliser [jbehave-junit-runner](https://github.com/codecentric/jbehave-junit-runner) dans votre projet, vous devez juste rajouter une dépendance maven dans votre fichier `pom.xml`
```XML
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>jbehave-junit-runner</artifactId>
    <version>1.0.1</version>
</dependency>
```

Pour indiquer à votre environnement de tests que vous souhaitez disposer des rapports dans la vue JUnit, il vous suffit juste d'annoter par `@RunWith(JUnitReportingRunner.class)` les classes concernées, c'est-à-dire les classes héritant de `JUnitStory` ou de `JunitStories`.
 
Dans la classe `CalculatriceAddition` de votre projet, ajoutez l'annotation `@RunWith` ainsi que les imports nécessaires pour pouvoir utiliser [jbehave-junit-runner](https://github.com/codecentric/jbehave-junit-runner).

```JAVA
import org.junit.runner.RunWith;
import de.codecentric.jbehave.junit.monitoring.JUnitReportingRunner;

@RunWith(JUnitReportingRunner.class)
public class CalculatriceAddition extends JUnitStory {
// code déjà existant
 }
```
Exécutez et consultez la vue JUnit qui propose maintenant des informations plus détaillées et affiche l'intégralité des étapes qui ont été jouées : les étapes `BeforeStories`, toutes les étapes du scénario et les étapes `AfterStories`.

##### Visualisation du contenu du rapport de JBehave au format HTML

Un rapport au format HTML nommé `reports.html` est disponible après l'exécution des scénarios dans le répertoire : `target/jbehave/view`. 
Il est possibe d'améliorer la présentation de ce rapport, nous ne le ferons pas dans le cadre de ce tutoriel....


#### Utilisation d'une StepsFactory 
La configuration actuelle de l'environnment de tests est une configuration minimale qui vous permettrait de commencer un développement BDD.

En pratique, la configuration usuelle pour créer des classes identifiables comme des [`Steps`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/steps/Steps.html) par JBehave consiste plutôt à utiliser à utiliser une fabrique de Steps via la méthode [`stepsFactory`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/ConfigurableEmbedder.html#stepsFactory()] qui renvoie une [`InjectableStepsFactory`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/steps/InjectableStepsFactory.html) que JBehave va utiliser pour créer les [`CandidateSteps`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/steps/CandidateSteps.html) qui sont les classes contenant l'implémentation en java des différentes étapes du scénario.

Tout l'intérêt de cette approche est liée à la simple déclaration de la classe considérée comme [`CandidateSteps`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/steps/CandidateSteps.html) qui va ainsi apparaître comme une simple comme une "Plain Old Java Class".

Pour transformer votre projet en ce sens, vous devez modifier la classe `CalculatriceAddition` en redéfinissant la méthode `stepsFactory`, ce qui vous amène à supprimer dans le constructeur l'appel à `candidateSteps` pour vous amener vers un code semblable au suivant :

```JAVA
@RunWith(JUnitReportingRunner.class)
public class CalculatriceAddition extends JUnitStory {

	public CalculatriceAddition() {
		super();
	}

	@Override
	public Configuration configuration() {

		Configuration configuration = new MostUsefulConfiguration();

		StoryReporterBuilder storyReporterBuilder = 
				new StoryReporterBuilder()
				.withDefaultFormats()
				.withFormats(org.jbehave.core.reporters.Format.CONSOLE);
		configuration.useStoryReporterBuilder(storyReporterBuilder);
		return configuration;
	}
	
	  @Override
	    public InjectableStepsFactory stepsFactory() {
        return new InstanceStepsFactory(configuration(), new AdditionSteps());
	    }
}
```

Vous pouvez maintenant modifier la déclaration de la classe `AdditionSteps` en supprimant l'héritage.
La classe `AdditionSteps` sera bien considérée comme [`CandidateSteps`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/steps/CandidateSteps.html) parJBehave grâce à l'injection, mais « vue de l'exétrieur » elle apparaît désormais simplement comme une "Plain Old Java Class".

```JAVA
public class AdditionSteps {
 //code existant
}
```

#### Configuration plus complète de l'environnement de tests

La configuration actuelle de notre [`Embedder`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/embedder/Embedder.html), bien que minimaliste pour l'instant, est suffisante pour la suite du tutoriel.

Si vous souhaitez plus tard mettre en place une configuration plus complète de votre environnement de test, n'hésitez pas à consulter les parties [Reporting Stories](http://jbehave.org/reference/stable/reporting-stories.html), [Running Stories](http://jbehave.org/reference/stable/running-stories.html) et [Configuration](http://jbehave.org/reference/stable/configuration.html) du tutoriel proposé sur le site de [Jbehave](http://jbehave.org)

On pourra noter entre autres que le constructeur permet de personnaliser la configuration de l'environnement de tests (c-à-d la configuration de l'[`Embedder`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/embedder/Embedder.html)) avec des méthodes qui sont susceptibles de contrôler l'exécution des tests comme `useThreads` ou `useStoryTimeoutInSecs` et même d'interagir sur la perception des tests en décidant si un test en échec doit arrêter ou non l'execution comme `doIgnoreFailureInStories`...


### Implémentation Java de chaque étape des scénarios de la story

Vous allez maintenant réaliser le mapping entre les étapes décrites de manière textuelle dans le fichier `.story` et les méthodes à implémenter dans le fichier `.java` que jBehave considére comme des steps.

La configuration du générateur de rapport dans la console va vous faciliter la tâche car il indique directement le code à utiliser.
Commencez par consulter dans la console le contenu du rapport fourni par JBehave après l'exécution de la classe `CalculatriceAddition`. 
```
Narrative:
In order to pouvoir faire des calculs le plus rapidement possible
As a utilisateur
I want to utiliser une calculatrice pour additionner deux nombres
Scenario: Addition de deux nombres valides
Given une calculatrice (PENDING)
When j'additionne 1 et 2 (PENDING)
Then le resultat doit être égal 3 (PENDING)
```
Le début du rapport reprend la partie `Narrative` de la story et indique pour chaque étape du scénario ce qui s'est passé pendant le temps d'exécution.

#### Etapes `PENDING` 
La partie [Pending Steps](http://jbehave.org/reference/stable/pending-steps.html) du tutoriel de Jbehave indique qu'une étape `PENDING` est une étape présente dans un fichier textuel `.story` qui n'a pas de méthodes correspondantes dans la(les) classe(s) `.java` considérée(s) comme des `Steps` par Jbehave. Cela revient à dire qu'une étape est `PENDING` lorsque le mapping entre le texte et le code n'est pas encore réalisé.

Pour le moment toutes nos étapes sont marquées `PENDING`, ce qui est normal puisque la classe `AdditionSteps` est actuellement vide.


#### Configuration de la [`PendingStepStrategy`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/failures/PendingStepStrategy.html) dans l'environnement de tests 

Actuellement, lorsque vous exécutez la classe `CalculatriceAddition` les tests passent : ils sont au vert.

Dans les projets BDD, la méthode de travail la plus courante consiste à faire échouer les tests lorsqu'une étape est `PENDING`.
Pour mettre en place cette stratégie de tests, il est nécessaire de modifier la [`Configuration`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/configuration/Configuration.html).

La [`PendingStepStrategy`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/failures/PendingStepStrategy.html) va permettre d'indiquer à JBehave la stratégie à tenir lorsque les steps ne sont pas trouvés (état `PENDING`). Par défaut, la stratégie est  [`PassingUponPendingStep`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/failures/PassingUponPendingStep.html) c-à-d que qu'une étape à l'état `PENDING` ne fait pas échouer le test. Pour indiquer que le test doit échouer sur une étape non trouvée il faut utiliser la stratégie [`FailingUponPendingStep`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/failures/FailingUponPendingStep.html) de la manière suivante :

```JAVA
@Override
public Configuration configuration() {

	// code existant
	
	configuration.usePendingStepStrategy(new FailingUponPendingStep());

	return configuration;
	}
```

Modifiez le code de votre classe `CalculatriceAddition`. Exécutez-le afin de vérifier que désormais, l'exécution échoue et que les tests sont bien au rouge.

#### Mapping texte/code : implémentation des étapes java (steps) 
Continuez à examiner le rapport affiché dans la console. Après les parties `Narrative` et `Scenario`, le rapport propose pour chaque étape `PENDING`, le code qui pourrait être implémenté dans la classe `AdditionSteps` afin de réaliser le mapping entre le texte du fichier `.story` et le code du fichier `.java`:

```JAVA
@Given("une calculatrice")
@Pending
public void givenUneCalculatrice() {
  // PENDING
}

@When("j'additionne 1 et 2")
@Pending
public void whenJadditionne1Et2() {
  // PENDING
}

@Then("le resultat doit \u00EAtre \u00E9gal 3")
@Pending
public void thenLeResultatDoitÊtreÉgal3() {
  // PENDING
}
```

Les étapes Java, que nous appellerons les **steps**, sont définies grâce à des annotations spécifiques: `@Given` , `@When` , `@Then`. 
La valeur de chaque annotation correspond à la phrase décrivant l'étape du scénario dans le fichier `.story`

Copiez-Collez dans la classe `AdditionSteps` le code précédent, extrait de l'affichage du rapport sur la console. Ne pas oubliez d'importer les packages nécessaires à la prise en compte des annotations de Jbehave.

Exécutez la classe `CalculatriceAddition` et constatez que les tests sont toujours au rouge.

Supprimez l'annotation `@Pending` et exécutez à nouveau. Vos tests seront désormais au vert. En effet, le but de l'annotation  `@Pending` est d'indiquer à Jbehave que le step n'a pas encore été implémenté. Il est donc conseillé de laisser cette annotation tant que l'implémentation n'a pas encore été réalisée

Vous pouvez donc remettre une annotation `@Pending` sur les étapes `@When` et `@Then` puisque vous allez maintenant implémenter l'étape  `@Given` de votre scénario.

#### Implémentation du code de test et du code métier
Grâce au plug-in Jbehave, il est possible depuis une étape dans l'éditeur de scénario d'accéder directement à la méthode correspondante dans le fichier java soit par **Ctrl+Clic** sur l'étape concernée, soit par un **Ctrl+G** (Go !). 

Dans l'éditeur de scénarios de `classe calculatrice_addition.story`, placez-vous sur la ligne du scénario correspondant à l'étape : `Given une calculatrice`. Un **Ctrl+Clic** vous amène directement sur la ligne de code `public void givenUneCalculatrice()` du fichier `AdditionSteps.java` dans `src/main/test`.
Il ne vous reste plus qu'à implémenter cette méthode en instanciant une nouvelle `Calculatrice` et en créant dans le package adéquat du dossier `src/main/java`, la classe `Calculatrice.java` afin de pouvoir relancer les tests:

```JAVA
private Calculatrice calculatrice;

@Given("une calculatrice")
public void givenUneCalculatrice() {
  calculatrice = new Calculatrice();
  }
```


Placez-vous maintenant dans l'éditeur de scénarios de `calculatrice_addition.story` sur la ligne du scénario correspondant à l'étape : `When j'additionne 1 et 2`.
Un **Ctrl+Clic** ne vous amène pas sur `@When("j'additionne 1 et 2")` mais indique `No step Matching`... 
En effet 1 et 2 sont deux valeurs dans l'étape `When` du scénario textuel, il faut donc que l'annotation` @When` les considèrent comme des variables pour que le mapping soit possible. Par défaut dans une annotation, les mots commençant par `$` désignent les variables.
Transfomez l'annotation `@When` du fichier `AdditionSteps.java` de la manière suivante : 
`@When("j'additionne $nombre1 et $nombre2")`. Désormais un **Ctrl+Clic** sur l'étape `When j'additionne 1 et 2` amène bien sur la méthode correspondante du fichier `AdditionSteps.java`.
Il ne vous reste plus qu'à implémenter cette méthode :
```JAVA
@When("j'additionne $nombre1 et $nombre2")
public void whenAdditionner(int nombre1, int nombre2) {
  calculatrice.additionner(nombre1, nombre2);
  } 
```

Pour une meilleure lisibilité, le nom de la méthode a été renommé.

Par défaut, les variables de l'annotation sont injectées en paramètres de la méthode java dans le même ordre. La variable `$nombre1` sera injectée comme premier paramètre de la méthode et la variable `$nombre2` comme second. Pour plus de lisibilité, les paramètres d'entrée ont été nommés à l'identique des variables et dans le même ordre...

N'oubliez pas de supprimez l'annotation `@Pending` et d'implémentez le plus simplement possible  la classe `Calculatrice.java`. 
```JAVA
public class Calculatrice {

	int resultat;
	
	public void additionner(int operande1, int operande2) {
		resultat = operande1 + operande2;
		
	}
	
	public int getResultat() {
		return resultat;
	}
 ```
**Remarque** : Dans le cas d'un projet plus complexe, l'implémentation de cette classe métier pourrait se faire par une approche de type **TDD**.

Exécutez la classe `CalculatriceAddition.java` et vérifiez que tous les tests passent bien au vert.


Vous devez maintenant implémenter la méthode correspondant à l'annotation `@Then` dans laquelle le résultat doit également apparaître comme une variable. Voici une implémentation possible pour cette méthode qui a été également renommée :
```JAVA
@Then("le resultat doit \u00EAtre \u00E9gal \u00E0 $resultat")
public void thenLeResultatEst(int resultat) {
	Assert.assertEquals(resultat, calculatrice.getResultat());
  }
```

Exécutez la classe `CalculatriceAddition.java` et vérifiez que tous les tests passent bien au vert.
Afin de vérifier le bon comportement de votre système, modifiez momentanément le scénario en falsifiant le résultat afin que les tests soient au rouge comme par exemple avec `Then le resultat doit être égal à 4`

Et voilà, vous venez de mettre en place votre premier scénario en mode BDD !
Dans un projet plus complexe, voilà à quoi pourrait ressembler votre cycle de développement alliant BDD et TDD.

![Cycle de développement BDD et TDD](http://arnauld.github.io/incubation/jbehave-get-started/bdd-cycle-around-tdd-cycles.png "Image extraite de : http://arnauld.github.io/incubation/Getting-Started-with-JBehave.html")


### Nouveaux scénarios pour `calculatrice_addition.story`

Vous allez écrire deux nouveaux scénarios dans votre story `calculatrice_addition`.

Pour ajouter le premier cénario, vous pouvez commencer par un simple copier-coller du scénario précédent et le modifier avec les valeurs de ce nouveau scénario :
```GHERKIN
Scenario: une autre addition de deux nombres valides
Given une calculatrice
When j'additionne 99 et 1 
Then le resultat doit être égal à 100
```
Exécutez `CalculatriceAddition.java` pour vérifier que les tests passent toujours au vert.


Pour ajouter le second scénario, vous allez utiliser la complétion proposée par l'éditeur de scnéarios. Placez-vous en fin du fichier  `calculatrice_addition.story`. La complétion **Ctrl+Espace** permet d'obtenir la liste de tous les mots clés.
Choisissez `Scénario` et complétez de la manière suivante : `Scenario: une dernière addition de deux nombres valides`

La complétion **Ctrl+J** permet de rechercher toutes les étapes déjà écrites. Grâce à des **Ctrl+J** successifs, écrivez le scénario suivant :
```GHERKIN
Scenario: une dernière addition de deux nombres valides
Given une calculatrice
When j'additionne 0 et 1000
Then le resultat doit être égal à 1000
```
Exécutez `CalculatriceAddition.java` pour vérifier que les tests passent toujours au vert.


#### Utilisation de l'annotation @Alias

Reformulez l'étape `Then` du dernier scénario de la manière suivante : `Then le resultat est 1000` et relancez les tests. 
Que se passe-t-il ? 

Les tests sont au rouge car il n'y a pas de mapping entre l'étape du scénario et le code java. En effet, rappelons que pour que le mapping existe, la **valeur de l'annotation** correspondante du fichier `java` doit contenir la **phrase décrivant l'étape du scénario** dans le fichier `story`.

Or pour l'instant, aucune annotation `@Then` ne contient `le resultat est 1000` ou `le resultat est $resultat`. Pourtant l'implémentation de cette étape existe déjà dans notre fichier `AdditionSteps.java`. Pour éviter la duplication de code, vous allez utiliser l'annotation `@Alias` qui permet d'exprimer de manière différente une même étape du scénario.
```JAVA
@Then("le resultat doit \u00EAtre \u00E9gal \u00E0 $resultat")
@Alias("le resultat est $resultat")
public void thenLeResultatEst(int resultat) {
   Assert.assertEquals(resultat, calculatrice.getResultat());
 }
```
Exécutez `CalculatriceAddition.java` pour vérifier que les tests passent toujours au vert. 

**Remarque** : La partie [Aliases](http://jbehave.org/reference/stable/aliases.html) du tutoriel Jbehave est consacré aux alias. Dans le cas d'alias multiples, il est nécessaire d'utiliser la notation `@Aliases`:
```JAVA
@Then("le resultat doit \u00EAtre \u00E9gal \u00E0 $resultat")
@Aliases(values={"le resultat est $resultat", 
		  "le resultat devient $resultat"})
public void thenLeResultatEst(int resultat) {
	Assert.assertEquals(resultat, calculatrice.getResultat());
  }
```

#### Expressions Régulières : pattern variants
Les alias permettent d'exprimer autrement la description textuelle d'une même étape.
Si la différence textuelle est petite (autour d'un mot par exemple), il est préférable d'utiliser un **pattern** plutôt qu'un `@Alias`. 

Reformulez l'étape `Then` du deuxième scénario : `Then la somme est 100`

Grâce aux **directives de patterns `{|}`**, vous allez pouvoir regrouper dans la valeur de l'annotation `@Alias`, à la fois la description de l'étape `Then` du deuxième scénario et celle du troisième scénario.
```JAVA
@Then("le resultat doit \u00EAtre \u00E9gal \u00E0 $resultat")
	@Alias("{le resultat| la somme} est $resultat")
	public void thenLeResultatEst(int resultat) {
		Assert.assertEquals(resultat, calculatrice.getResultat());
	}
```

Exécutez `CalculatriceAddition.java` pour vérifier que les tests passent toujours au vert. 

Le tutoriel JBehave consacre une partie aux [Patterns variants](http://jbehave.org/reference/stable/pattern-variants.html). La construction des patterns Jbehave s'appuye sur le [`PatternVariantBuilder`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/steps/PatternVariantBuilder.html).

#### Utilisation de l'annotation `@Named`

Pour illustrer l'annotation `@Named`, vous allez écrire un nouveau scénario :
```GHERKIN
Scenario: la soustraction : une addition particulière
Given une calculatrice
When je soustrais 5 moins 2
Then le resultat est 3
```

Les étapes `Given` et `Then` sont déjà implémentées. Exécutez `CalculatriceAddition.java` afin de récupérer dans la console le squelette de code proposé par le rapport pour l'étape: `When je soustrais 5 moins 2`
et implémentez cette étape :
```JAVA
@When("je soustrais $nombre1 moins $nombre2")
   public void whenSoustraire(int nb1, int nb2) {
     calculatrice.additionner(nb1, -nb2);
     }
```

Exécutez `CalculatriceAddition.java` pour vérifier que les tests passent toujours au vert. 
Rappelons que, par défaut, les variables de l'annotation sont injectées en paramètre dans la méthode java dans leur ordre d'appartition. Ainsi, la variable `$nombre1` sera injectée comme premier paramètre `nb1` et la variable `$nombre2` comme second parametre `nb2`. 

Modifiez le code de la méthode `whenSoustraire` de la manière suivante :
```JAVA
@When("je soustrais $nombre1 moins $nombre2")
   public void whenSoustraire(@Named("nombre1") int nb1, @Named("nombre2")int nb2) {
     calculatrice.additionner(nb1, -nb2);
     }
```

Exécutez `CalculatriceAddition.java` pour vérifier que les tests passent toujours au vert. 
L'annotation n'a rien changé pour le moment.

Inverser les annotations `@Named`:
```JAVA
@When("je soustrais $nombre1 moins $nombre2")
   public void whenSoustraire(@Named("nombre2") int nb1, @Named("nombre1")int nb2) {
     calculatrice.additionner(nb1, -nb2);
     }
```

Exécutez `CalculatriceAddition.java` et constatez que les tests sont désormais rouges !

L'annotation `@Named` permet de renommer les paramètres.
Par défaut, les variables de l'annotation sont injectées en tant que paramètres dans leur ordre d'apparition. Si cet ordre ne convient pas, il est possible d'annoter chaque paramètre par `@Named`, pour indiquer explicitement la variable qu'il référence. L'annotation `@Named` permet de renommer les paramètres, elle permet ainsi de jouer sur l'ordre des paramètres. 

Remettre les annotations `@Named` correctement afin que les tests passent au vert pour le suite du tutoriel.

### Scénarios paramétrés

Reprenez l'ensemble des scénarios du fichier `calculatrice_addition.story` :

```GHERKIN
Scenario: Addition de deux nombres valides
Given une calculatrice
When j'additionne 1 et 2 
Then le resultat doit être égal à 3

Scenario: une autre addition de deux nombres valides
Given une calculatrice
When j'additionne 99 et 1 
Then la somme est 100

Scenario: une dernière addition de deux nombres valides
Given une calculatrice
When j'additionne 0 et 1000
Then le resultat est 1000

Scenario: la soustraction: une addition particulière 
Given une calculatrice
When je soustrais 5 moins 2
Then le resultat est 3
```

La lecture des trois premiers scénarios montrent une duplication dans le descriptif des étapes. 
Il est possible de factoriser ces descriptions et de les regrouper au sein d'un même scénario dit **paramétré** grâce au mot clé  **`Examples`**

Remplacez lez trois premiers scénarios par un **scénario paramétré**, par exemple:
```GHERKIN
Scenario: Addition de deux nombres valides
Given une calculatrice
When j'additionne <nombre1> et <nombre2> 
Then la somme est <somme>

Examples:
|nombre1|nombre2|somme|
|1|2|3|
|99|1|100|
|0|1000|1000|
```


Exécutez `CalculatriceAddition.java` …
Le rapport de la console indique que l'étape `When` est `FAILED`. En effet, la description textuelle a changé, puisque qu'elle comporte maintenant des paramètres alors que précédemment elle comportait directement des valeurs. Il est donc nécessaire d'ajouter un **alias** sur `@When` :
```JAVA
@When("j'additionne $nombre1 et $nombre2")
@Alias("j'additionne <nombre1> et <nombre2>")
```

Exécutez `CalculatriceAddition.java`.

Mais, rajoutez l'alias ne suffit pas... Avec des tables de paramètres, il est obligatoire de nommer les paramètres avec `@Named` comme le préconise la partie [Parametrised Scenarios](http://jbehave.org/reference/stable/parametrised-scenarios.html) du tutoriel de Jbehave :
``` JAVA
@When("j'additionne $nombre1 et $nombre2")
@Alias("j'additionne <nombre1> et <nombre2>")
public void whenAdditionner(@Named("nombre1")int nombre1, @Named("nombre2")int nombre2) 
   {
   calculatrice.additionner(nombre1, nombre2);
   }
```


Exécutez `CalculatriceAddition.java`... Le rapport de la console indique que l'étape `Then` est `FAILED`.
Modifiez cette étape avec un `@Alias` (ou `@Aliases`) et une annotation `@Named`.
```JAVA 
@Then("le resultat doit \u00EAtre \u00E9gal \u00E0 $resultat")
@Aliases(values={"{le|la} {resultat|somme} est $resultat",
		 "la somme est <somme>"})
public void thenLeResultatEst(@Named("somme")int resultat) {
   Assert.assertEquals(resultat, calculatrice.getResultat());
   }
```
Exécutez `CalculatriceAddition.java` et vérifier que les tests passent désormais au vert !


### A propos du mot clé `And`

Lors de l'écriture des scénarios, les trois mots clés `Given`, `When`, et `Then` sont obligatoires pour définir les étapes du scénario. 
Mais, un seul mot clé `Given`, un seul mot clé `When`, et un seul mot clé `Then` peuvent être utilisés. 

Le mot clé `And` va permettre d'enrichir la description des scénarios. Par exemple, si deux étapes `When` doivent être décrites, le mot clé `When` décrira la première et le mot clé `And` la seconde.
Pour la calculatrice, il aurait été possible par exemple d'écrire le scénario suivant :
```GHERKIN
Scenario: une autre description de l'addition 
Given une calculatrice
And l'utilisateur saisit 1 sur la calculatrice
And l'utilisateur appuie sur la touche +
And l'utilisateur saisit 2 sur la calculatrice
When l'utilisateur appuie sur la touche =
Then le resultat est 3
```

Le mot clé `And` permet de rajouter une étape supplémentaire au scénario. Il  ne peut se trouver qu'après un des trois mots clés de base `Given`, `When`, et `Then` ou après un autre `And`. Il peut y avoir autant de `And` que le nécessite la description.


### Des stories en français ...
Par défaut la syntaxe Jbehave est en anglais. Or, la réussite d'un projet BDD est fortement liée à l'écriture des stories de manière collaborative dans un langage naturel et compris de tous.
Il est donc important de pouvoir configurer les mots clés de Jbehave dans une autre langue.

La partie Configurable Elements du chapitre [Configuration](http://jbehave.org/reference/stable/configuration.html) du tutoriel Jbehave indique qu'il est possible de configurer les mots clés ([Keywords](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/configuration/Keywords.html)).

Voici une méthode simple pour personnaliser vos mots clés et faciliter l'écriture et la compréhension de vos scénarios.
Il suffit de modifier la configuration via la méthode `public Configuration useKeywords(Keywords keywords)` qui prendra en paramètres vos propores mots clés.

Les modifications à effectuer pour mettre en place une internationalisation des mots clés concerne donc le lanceur de story c-à-d la classe `CalculatriceAddition.java`

```JAVA
public class CalculatriceAddition  extends JUnitStory {
	
	@Override
	public Configuration configuration( ) {
		//code déjà écrit...	
		  
		//Internationalisation
		 configuration.useKeywords(setKeyWords());

		  return configuration;
		 }

	//code déjà écrit...

	//Ne pas oublier de rajouter la méthode privée setKeyWords
	// pour mettre en place la francisation des mots clés ...
	private Keywords setKeyWords() {
	    	Map<String, String> keywords = Keywords.defaultKeywords();
	        keywords.put(Keywords.GIVEN, "Soit");
	        keywords.put(Keywords.WHEN, "Quand");
	        keywords.put(Keywords.THEN, "Alors");
	        keywords.put(Keywords.AND, "Et");
			return new Keywords(keywords);
	 }
		
 }
```

Exécutez `CalculatriceAddition.java`.Si vous n'avez pas changé les mots clés dans le fichier `calculatrice_addition.story`, vous aurez des erreurs...

Remplacez donc tous les `Given` par des `Soit`, tous les `When` par des `Quand` et tous les `Then` par des `Alors` …
Exécutez `CalculatriceAddition.java` et vérifier que les tests passent désormais au vert !
 

