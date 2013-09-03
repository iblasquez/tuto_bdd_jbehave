#Tutoriel JBehave : Prise en main rapide - JDEV2013 


**JBehave** a été le premier framework Behavior Driven Development développé par Dan North, initiateur du concept BDD. 
JBehave permet de transformer les scénarios d'une story (descriptions de fonctionnalités écrites en «langage naturel») en tests automatisés. Dans le principe, cette transformation est possible à l'aide du framework de tests [JUnit](http://junit.org/) : chaque phrase de la description textuelle d'un scénario (une étape du scénario)  sera implémentée comme une méthode dans un fichier de tests. Le lien entre la description textuelle et le code Java est réalisé via des annotations.
JBehave est un framework destiné à Java et aux langages de la JVM tels que Groovy, Scala,... 
Il est disponible sur : [http://Jbehave.org](http://jbehave.org)


## Installation préalable du plug-in JBehave pour Eclipse

JBehave fournit un plugin Eclipse avec un mode éditeur JBehave personnalisé. La documentation est disponible sur : [http://jbehave.org/eclipse-integration.html](http://jbehave.org/eclipse-integration.html). Ce plug-in propose entre autres une coloration syntaxique, une détection de lien hypertexte d'une étape textuelle vers la méthode Java correspondante, une autocomplétion, une validation des étapes.

Dans le cadre de ce tutoriel, nous utiliserons une version d' **Eclipse IDE for Java Developers** téléchargable depuis [http://www.eclipse.org/downloads/] (http://www.eclipse.org/downloads/).

Pour mettre en place ce plug-in, nous utiliserons l'installeur de plug-in d'Eclipse : **Help → Install New Software...** en indiquant le site  [http://jbehave.org/reference/eclipse/updates/](http://jbehave.org/reference/eclipse/updates/). 


##  Mise en place d'un premier projet BDD : Résumé en quelques clics...

La mise en place d'un projet BDD sous JBehave se fera en plusieurs étapes :

1. Création d'un projet Maven
2. Mise à jour du `pom.xml`
3. Description textuelle d'une story et de ses scénarios exécutables dans un fichier `nom_story.story`
4. Configuration de l'environnement de tests des scénarios (Implémentation d'un lanceur héritant de [`JUnitStory`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/junit/JUnitStory.html) ou [`JUnitStories`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/junit/JUnitStories.html) permettant de  lier les scénarios au code : fichier java dans `src/test/java`)
5. Implémentation du code de test pour chaque étape des scénarios de la story dans un fichier `NomStorySteps.java` (dans `src/test/java`)
6. Implémentation en TDD du code métier de l'application (dans `src/main/java`)



### 1. Création d'un projet Maven

Créez un nouveau projet Maven sous Eclipse : **File → New → Project... → Maven → Maven Project** avec par exemple `fr.cnrs.devlog.jdev` comme **groupId** et `demobdd` comme **artifactId**.


### 2. Mise à jour du `pom.xml`

Mettez à jour votre `pom.xml` :
- en rajoutant les propriétés `maven.compiler.source` et `maven.compiler.target` avec la version Java 1.7 
- en vérifiant la version utilisée pour [JUnit](http://junit.org/)
- en rajoutant une dépendance sur JBehave (dernière version de JBehave disponible sur [http://JBehave.org/download.html](http://JBehave.org/download.html) dans [Core Distribution](https://nexus.codehaus.org/content/repositories/releases/org/JBehave/JBehave-distribution/))
- en rajoutant une dépendance sur le projet [jbehave-junit-runner](https://github.com/codecentric/jbehave-junit-runner) qui permet visualiser chaque étape de chaque scénario comme un test spécifique directement dans la vue JUnit.

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
		<dependency>
			<groupId>de.codecentric</groupId>
			<artifactId>jbehave-junit-runner</artifactId>
			<version>1.0.1</version>
		</dependency>
	</dependencies>
</project>

```

### 3. Description textuelle d'une story et de ses scénarios exécutables dans un fichier `nom_story.story` 
Voici une story simple qui consiste à **faire une addition à l'aide d'une calculatrice**. La story et ses scénarios seront écrits dans `calculatrice_addition.story`.

```
Narrative:
In order to pouvoir faire des calculs le plus rapidement possible
As a utilisateur
I want to utiliser une calculatrice pour additionner deux nombres

Scenario: Addition de deux nombres valides
Given une calculatrice
When j'additionne <nombre1> et <nombre2> 
Then la somme est <somme>

Examples:
|nombre1|nombre2|somme|
|1|2|3|
|99|1|100|
|0|1000|1000|

Scenario: la soustraction qui est une addition particulière
Given une calculatrice
When je soustrais 5 moins 2
Then le resultat est 3

Scenario: une soustraction avec un nombre négatif
Given une calculatrice
When je soustrais 5 moins -2
Then la solution est 7
```


**Emplacement de la story :**

- si une seule story dans le projet : directement dans le dossier **`src/test/java`**
- si plusieurs stories dans le projet : dans le dossier **`target`** 


*Remarque :* L'écriture de la story peut être facilitée par le plug-in JBehave. Pour cela, placez-vous dans la vue **Package Explorer** à l'emplacement souhaité.
Un clic droit puis **New → Other... → Jbehave → New story** permettent d'entrer le nom de la story **calculatrice_addition**. Un fichier `calculatrice_addition.story` contenant un template est alors créé et ouvert dans l'éditeur de scénarios. 


### 4. Configuration de l'environnement de tests des scénarios (fichier java dans `src/test/java`)

Le lanceur permet de faire le mapping entre les étapes des scénarios écrites de manière textuelle (`calculatrice_addition.story`) et les méthodes Java implémentant ces étapes (`AdditionSteps.java`). Il permet de configurer l'environnement de tests des scénarios c-à-d de s'interroger sur comment lancer les scénarios et surtout comment visualiser les résultats obtenus. 

*Remarque :* La configuration proposée ci-dessous est minimale. Pour mettre en place une configuration plus complète de votre environnement de tests, consultez les parties [Reporting Stories](http://jbehave.org/reference/stable/reporting-stories.html), [Running Stories](http://jbehave.org/reference/stable/running-stories.html), [Configuration](http://jbehave.org/reference/stable/configuration.html), [Pending Steps](http://jbehave.org/reference/stable/pending-steps.html) du tutoriel proposé sur le site de [Jbehave](http://jbehave.org).

#### 4.1 Lanceur héritant de [`JUnitStory`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/junit/JUnitStory.html) pour une seule story

**Attention il est important de respecter la casse et l'emplacement lorsque [`JUnitStory`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/junit/JUnitStory.html) est utilisé !** En effet, pour un lanceur nommé `NomStory.java`, JBehave cherchera, par défaut dans le même paquetage, à lancer le fichier de scénarios `nom_story.story`. S'il ne le trouve pas, il générera une erreur. 

Le lanceur `CalculatriceAddition.java` devra être implémenté dans le package du dossier `src/test/java` contenant `calculatrice_addtition.story`:

```JAVA
package fr.cnrs.devlog.jdev.demobdd;

import org.jbehave.core.configuration.Configuration;
import org.jbehave.core.configuration.MostUsefulConfiguration;
import org.jbehave.core.failures.FailingUponPendingStep;
import org.jbehave.core.junit.JUnitStory;
import org.jbehave.core.reporters.StoryReporterBuilder;
import org.jbehave.core.steps.InjectableStepsFactory;
import org.jbehave.core.steps.InstanceStepsFactory;
import org.junit.runner.RunWith;

import de.codecentric.jbehave.junit.monitoring.JUnitReportingRunner;

@RunWith(JUnitReportingRunner.class)
public class CalculatriceAddition extends JUnitStory {

	public Configuration configuration() {

		Configuration configuration = new MostUsefulConfiguration();
		// CONSOLE Reporting
		StoryReporterBuilder storyReporterBuilder;
		storyReporterBuilder = new StoryReporterBuilder();
		storyReporterBuilder.withDefaultFormats();
		storyReporterBuilder.withFormats(org.jbehave.core.reporters.Format.CONSOLE);
		configuration.useStoryReporterBuilder(storyReporterBuilder);

		// Stratégie de tests
		configuration.usePendingStepStrategy(new FailingUponPendingStep());

		return configuration;
	}

	public CalculatriceAddition() {
		super();
	}

	@Override
	public InjectableStepsFactory stepsFactory() {
		return new InstanceStepsFactory(configuration(), new AdditionSteps());
	}
}

}
```

**La classe [MostUsefulConfiguration](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/configuration/MostUsefulConfiguration.html)** contient les configurations par défaut.  A cette configuration de base :

- est **ajouté un [StoryReporterBuilder](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/reporters/StoryReporterBuilder.html)** personnalisé, associé à la console. La classe [StoryReporterBuilder](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/reporters/StoryReporterBuilder.html) permet de configurer différents rapport avec des formats préconfigurés tels que : CONSOLE, TXT, HTML, HTML_TEMPLATE and XML.

- est **modifiée la [`PendingStepStrategy`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/failures/PendingStepStrategy.html)** afin que les tests lorsqu'une étape est `PENDING`. Par défaut, la stratégie est  [`PassingUponPendingStep`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/failures/PassingUponPendingStep.html). Pour indiquer que le test doit échouer sur une étape non trouvée il faut utiliser la stratégie [`FailingUponPendingStep`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/failures/FailingUponPendingStep.html)



**Une fabrique de [`Steps`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/steps/Steps.html)** est également mise en place dans le lanceur **via la méthode [`stepsFactory`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/ConfigurableEmbedder.html#stepsFactory()**. Rappelons que le lanceur permet aussi de faire le mapping entre les étapes des scénarios écrites de manière textuelle (`calculatrice_addition.story`) et les méthodes Java implémentant ces étapes (`AdditionSteps.java`) qui doivent être écrites dans une classe héritant de [`Steps`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/steps/Steps.html) (qui est une implémentation par défaut de l'interface [`CandidateSteps`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/steps/CandidateSteps.html)) 
L'intérêt d'utiliser une fabrique de [`Steps`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/steps/Steps.html) est liée à la simple déclaration de la classe considérée comme [`CandidateSteps`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/steps/CandidateSteps.html) (`AdditionSteps.java`) qui va ainsi apparaître comme une simple comme une "Plain Old Java Class".





### 5. Implémentation du code de test pour chaque étape des scénarios de la story dans un fichier `NomStorySteps.java` (dans `src/test/java`)
Pour JBehave, l'implémentation java des différentes étapes du scénario doit se trouver dans une classe qui implémente l'interface [`CandidateSteps`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/steps/CandidateSteps.html).

Grâce à la fabrique de [`Steps`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/steps/Steps.html) définie dans le lanceur, la classe `AdditionSteps` peut être écrite comme une simple **"Plain Old Java Class"**. 

Il n'est pas nécessaire de faire apparaître l'héritage sur [`Steps`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/steps/Steps.html) puisque l'injection permet à Jbheave de la considérer directement comme [`CandidateSteps`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/steps/CandidateSteps.html).

*Remarque :* Pour écrire le squelette de code de cette classe aidez-vous du rapport de tests dans la console qui propose pour chaque étape `PENDING` le code qui pourrait être implémenté afin de réaliser le mapping entre les étapes textuelles du scénarios et celles du code java. Une étape est `PENDING` lorsque le mapping entre le texte et le code n'est pas encore réalisé.

```JAVA
package fr.cnrs.devlog.jdev.demobdd;

import org.jbehave.core.annotations.Alias;
import org.jbehave.core.annotations.Aliases;
import org.jbehave.core.annotations.Given;
import org.jbehave.core.annotations.Named;
import org.jbehave.core.annotations.Then;
import org.jbehave.core.annotations.When;
import org.jbehave.core.steps.Steps;factory
import org.junit.Assert;

public class AdditionSteps {
	private Calculatrice calculatrice;

	@Given("une calculatrice")
	public void givenUneCalculatrice() {
		calculatrice = new Calculatrice();
	}

	@When("j'additionne $nombre1 et $nombre2")
	@Alias("j'additionne <nombre1> et <nombre2>")
	public void whenAdditionner(@Named("nombre1") int nombre1,
			@Named("nombre2") int nombre2) {
		calculatrice.additionner(nombre1, nombre2);
	}

	@When("je soustrais $nombre1 moins $nombre2")
	public void whenSoustraire(@Named("nombre1") int nb1,
			@Named("nombre2") int nb2) {
		calculatrice.additionner(nb1, -nb2);
	}

	@Then("la somme est <somme>")
	@Alias("{le resultat|la solution} est $resultat")
	public void thenLeResultatEst(@Named("somme") int resultat) {
		Assert.assertEquals(resultat, calculatrice.getResultat());
	}

}
```


Les étapes Java, appelées des **steps**, sont définies grâce à des annotations spécifiques: **`@Given`** , **`@When`** ,**`@Then`**. 

Pour que le mapping existe, la **valeur de l'annotation** de la step du fichier `java` doit contenir la **phrase décrivant l'étape du scénario** dans le fichier `story`.

Dans une annotation :
- les mots commençant par **`$`** désignent les **variables**. Pour que le mapping soit possible, les valeurs mentionnées dans les étapes textuelles doivent être transformées variables dans une annotation.
- les mots entre **`<`** et **`>`** désignent les **paramètres** utilisées dans les scénarios paramatrés.
- des **directives de patterns `{|}`** peuvent être utilisées pour exprimer autrement la description textuelle d'une même étape si la différence textuelle est petite. Si la différence textuele est importante, il sera préférable d'utiliser une annotation `@Alias`. 

**L'annotation `@Alias`** permet d'exprimer de manière différente une même étape du scénario pour éviter la duplication de code. Dans le cas d'alias multiples, il est nécessaire d'utiliser l'annotation **`@Aliases`**

**L'annotation `@Named`** permet de renommer les paramètres et de jouer sur l'ordre des paramètres. Elle est obligatoire dans le cas des scénarios paramétrés. Par défaut, les variables de l'annotation sont injectées en tant que paramètres de la méthode java dans leur ordre d'apparition.  La variable `$nombre1` sera injectée comme premier paramètre de la méthode et la variable `$nombre2` comme second. Si cet ordre ne convient pas, il est possible d'annoter chaque paramètre par `@Named`, pour indiquer explicitement la variable qu'il référence.

**L'annotation `@Pending`** permet d'indiquer à Jbehave que le step n'a pas encore été implémenté : il est conseillé de laisser cette annotation tant que l'implémentation n'a pas encore été réalisée.

*Remarque:* Pour plus de renseignements, consultez les parties [Aliases](http://jbehave.org/reference/stable/aliases.html), [Pending Steps](http://jbehave.org/reference/stable/pending-steps.html), [Patterns variants](http://jbehave.org/reference/stable/pattern-variants.html) et [Parametrised Scenarios](http://jbehave.org/reference/stable/parametrised-scenarios.html) du tutoriel proposé sur le site de [Jbehave](http://jbehave.org).

### 6.Implémentation en TDD du code métier de l'application (dans `src/main/java`)
Implémentez au fur et à mesure en TDD la classe `Calculatrice` dans le package adéquat du dossier `src/main/java`:

```JAVA
package fr.cnrs.devlog.jdev.demobdd;

public class Calculatrice {

	int resultat;

	public void additionner(int operande1, int operande2) {
		resultat = operande1 + operande2;

	}

	public int getResultat() {
		return resultat;
	}

}
```

### Astuces plug-in JBehave sous Eclipse
Grâce au plug-in Jbehave, il est possible depuis une étape dans l'éditeur de scénario d'accéder directement à la méthode correspondante dans le fichier java soit par **Ctrl+Clic** sur l'étape concernée, soit par un **Ctrl+G** (Go !).


Le plug-in JBehave propose la complétion dans l'éditeur de scnéarios. La complétion **Ctrl+Espace** permet d'obtenir la liste de tous les mots clés.
La complétion **Ctrl+J** permet de rechercher toutes les étapes déjà écrites. Grâce à des **Ctrl+J** successifs, il est possible d'écrire un nouveau scénario à partir d'étapes déjà existantes.

### Des stories en français ...
Par défaut la syntaxe Jbehave est en anglais. Or, la réussite d'un projet BDD est fortement liée à l'écriture des stories de manière collaborative dans un langage naturel et compris de tous. Il est donc important de pouvoir configurer les mots clés de Jbehave ([Keywords](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/configuration/Keywords.html)) dans une autre langue.

Une méthode simple pour personnaliser et mettre en place une internationalisation sur les mots clés consiste à modifier la configuration du lanceur de story(ies) via la méthode `public Configuration useKeywords(Keywords keywords)` qui prendra en paramètres le nouveaux mots clés.

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

Une fois ce code implémenté, les scénarios devront être écrits avec des `Soit`, `Quand` et `Alors` (au lieu des `Given`, `When` et `Then`)


#### Lanceur héritant de [`JUnitStories`](http://jbehave.org/reference/stable/javadoc/core/org/jbehave/core/junit/JUnitStories.html)  pour de multiples stories


A partir de deux stories (dans un premier temps toujours dans le dossier `src/test/java'
 - `calculatrice_addition.story` (écrit précdédemment)
 - `calculatrice_mulitplication.story` composé de :
``` GHERKIN
Narrative:
In order to pouvoir faire des calculs le plus rapidement possible
As a utilisateur
I want to utiliser une calculatrice pour multiplier deux nombres

Scenario: Multiplier de deux nombres valides
Given une calculatrice
When je multiplie 5 par 2
Then la solution est 10
```


A partir du fichier `OperationSteps.java` contenant les steps suivantes (il s'agit en fait du fichier `AdditionSteps.java` complété et renommé)
Remarque : Le code des Steps peut aussi être refactoré durant un développement BDD...
```JAVA
public class OperationSteps {
    private Calculatrice calculatrice;
  
    @Given("une calculatrice")
    public void givenUneCalculatrice() {
        calculatrice = new Calculatrice();
    }
  
    @When("j'additionne $nombre1 et $nombre2")
    @Alias("j'additionne <nombre1> et <nombre2>")
    public void whenAdditionner(@Named("nombre1") int nombre1,
            @Named("nombre2") int nombre2) {
        calculatrice.additionner(nombre1, nombre2);
    }
  
    @When("je soustrais $nombre1 moins $nombre2")
    public void whenSoustraire(@Named("nombre1") int nb1,
            @Named("nombre2") int nb2) {
        calculatrice.additionner(nb1, -nb2);
    }
      
    @When("je multiplie $nombre1 par $nombre2")
    public void whenMultiplier(int nombre1, int nombre2) {
        calculatrice.multiplier(nombre1, nombre2);
    }
  
    @Then("la somme est <somme>")
    @Alias("{le resultat|la solution} est $resultat")
    public void thenLeResultatEst(@Named("somme") int resultat) {
        Assert.assertEquals(resultat, calculatrice.getResultat());
    }
 }

```


A partir du fichier `Calculatrice.java` suivant :
```JAVA
public class Calculatrice {
  
    int resultat;
  
    public void additionner(int operande1, int operande2) {
        resultat = operande1 + operande2;
  
    }
      
    public void multiplier (int operande1, int operande2) {
        resultat = operande1 * operande2;
  
    }
  
  
    public int getResultat() {
        return resultat;
    }
  
} 
```

Les paramétrages précédents utilisés pour un lanceur de story unique restent valables pour notre lanceur multiple c-à-c les méthodes `public Configuration configuration( )` et   `public InjectableStepsFactory stepsFactory()`
Par contre, la classe `JunitStories` possède une méthode abstraite à rededéfinir `List<String> storyPaths()` qui doit indiquer le chemin d'accès aux stories, puisqu'il n'est plus possible cette fois-ci de se baser sur le nom du lanceur.

Ecrire le fichier `CalculatriceStories.java` suivant :
```JAVA
@RunWith(JUnitReportingRunner.class)
public class CalculatriceStories extends JUnitStories {
  
  
    public Configuration configuration() {
  
       Configuration configuration = new MostUsefulConfiguration();
       	StoryReporterBuilder storyReporterBuilder;
       	storyReporterBuilder = new StoryReporterBuilder();
       	storyReporterBuilder.withDefaultFormats();
       	storyReporterBuilder.withFormats(org.jbehave.core.reporters.Format.CONSOLE);
       	configuration.useStoryReporterBuilder(storyReporterBuilder);
        
	configuration.usePendingStepStrategy(new FailingUponPendingStep());
	
	return configuration;
    }
  
  
    @Override
    public InjectableStepsFactory stepsFactory() {
       	return new InstanceStepsFactory(configuration(), new OperationSteps());
    }
  
  
    @Override
    protected List<String> storyPaths() {
       	String codeLocation = CodeLocations.codeLocationFromClass(this.getClass()).getFile();
       	List<String> storyPaths =  new StoryFinder().findPaths(codeLocation,Arrays.asList("**/*.story"), Arrays.asList(""));
    return storyPaths;
     }
  
} 
```



Lancez `CalculatriceStories.java` . Les tests s'exécutent bien avec le code précédent si les deux fichiers stories `calculatrice_addition.story` et `calculatrice_mulitplication.story` se trouvent dans le dossier `src/test/java`.


##### Lanceur pour de multiples stories dans le dossier `target`

Lorsqu'on travaille avec un projet maven, il est d'usage de placer les fichiers `.story` dans le dossier `src/test/resources`.
Remarque : Pourquoi le dossier `src/test/resources` est_il propice aux fichiers `.story` ?
Les ressources maven du dossier `src/test/resources`  sont copiées dans le répertoire `target/test-classes` . 
Si le lanceur est dans un dossier `src/test/java` ou `src/main resources` son byte code (fichier `.class`) se trouvera également dans le répertoire `target/test-classes`.Donc le lanceur et les stories se retouverons dans le même répertoire maven  `target/test-classes` …







