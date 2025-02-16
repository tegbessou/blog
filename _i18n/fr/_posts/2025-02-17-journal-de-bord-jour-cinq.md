---
layout: article
title: "Journal de bord de l’apprentissage du Domain-Driven Design : Jour 5"
date:   2025-02-17 08:00:00
categories: ddd-logbook
lang: fr
resume: "Ce cinquième article explore la mise en place d’un Supporting Domain dédié à la gestion des pays, tout en illustrant comment appliquer le Test-Driven Development pour créer et valider les Entities et Value Objects. Il aborde également la gestion d’une liste de pays via Api Platform, soulignant l’importance de séparer les préoccupations et de veiller à la cohérence du Domain."
permalink: /ddd-logbook/2025-02-17
---

Dans cet article, je vais aborder la notion de « Country » dans le cadre d’un **Supporting Domain** en prenant comme exemple un **Domain Country**. Je vais également détailler l’exposition d’une API permettant de récupérer les informations d’un pays et de vérifier son existence. Je commencerai par rappeler la définition d’un **Supporting Subdomain** ; ensuite je vous expliquerai comment le mettre en place pas à pas.

## Supporting Subdomain : qu’est-ce que c’est ?

Comme son nom l’indique, un **Supporting Subdomain** est avant tout un Subdomain : il représente une partie du **Domain** qui peut être séparé en **module**. Dans un projet d’entreprise, une équipe distincte pourrait en être responsable. Un **Subdomain** peut parfaitement disposer de son propre **Ubiquitous Language** lorsqu’il appartient à un **Bounded Context** différent du **Core Domain**.

Bien qu’important pour le métier, le **Supporting Subdomain** l’est pourtant moins que le **Core Domain**. Il n’est pertinent de créer un **Supporting Subdomain** que s’il apporte une valeur spécifique ou répond à un besoin particulier.

Dans le cas de notre API de gestion de bouteilles de vins, j’ai créé un **Domain Country** pour séparer cette notion du **Core Domain** qui gère le **Bottle Inventory**. Cette séparation permet d’organiser le code dans deux **Bounded Context** distincts. De plus, ce **Domain** est essentiel pour pouvoir proposer une liste de pays lors de la création d’une bouteille, tout en s’assurant que le pays existe réellement. Toutefois, comme il reste moins crucial que le **Core Domain**, il correspond parfaitement à la définition d’un **Supporting Subdomain**.

## Comment mettre en place le Domain ?

Dans cet article, je vais illustrer la mise en place du **Domain** à travers deux use cases. J’ai choisi de ne pas séparer ces cas en plusieurs parties, car ils sont assez répétitifs et ne représentent pas un véritable défi en terme de **Domain-Driven Design (DDD)**. Voici les deux scénarios que je vais aborder :

1. La création d’un **Country**  
2. La récupération de la liste de tous les **Country**

Je vous propose d’aborder ce sujet selon une approche différente de celle abordée pour le use case d’authentification. Je commencerai donc par la définition de l’**Entity**, accompagnée de tests unitaires afin d’intégrer une démarche orientée **Test-Driven Development (TDD)**.

## Présentation rapide du TDD

Le **Test-Driven Development (TDD)** est une méthodologie de développement très intéressante qui mériterait un article complet à elle-seule. Si vous cherchez des informations pour approfondir le sujet, je vous recommande vivement le livre *Test-Driven Development : By Example* de Kent Beck.

L’idée fondamentale derrière cette méthodologie est d’écrire les tests avant d’écrire la fonctionnalité correspondante. Cependant, l’essence même de cette approche réside dans une démarche progressive et rigoureuse, étape par étape (faire du pas-à-pas).

Dans le but d’illustrer cette méthode, je vous propose d’examiner les tests liés à la création d’une **Entity Country**.

### Première étape : test de la création d’une Entity Country

Nous commencerons par un test simple permettant de valider la création correcte d’une **Entity Country**.

{% highlight php linenos %}
public function testCreateSuccess(): void
{
    $country = Country::create(
        CountryId::fromString('af785dbb-4ac1-4786-a5aa-1fed08f6ec26'),
        CountryName::fromString('France'),
    );

    $this->assertInstanceOf(
        Country::class,
        $country,
    );
    $this->assertEquals(
        'af785dbb-4ac1-4786-a5aa-1fed08f6ec26',
        $country->id()->value(),
    );
    $this->assertEquals(
        'France',
        $country->name()->value(),
    );
}
{% endhighlight %}

Ce test permet de décrire précisément ce qui est attendu en sortie de la méthode **Factory** create. Dans une démarche **TDD**, le but est d’obtenir un test vert (c’est-à-dire réussi) le plus rapidement possible (c’est-à-dire en succès).

Lors de la première exécution du test, une erreur remonte immédiatement : la classe n’existe pas.

![First error Unit Test](/assets/images/2025-02-17/first-step-test.png)

Cette situation, tout à fait normale, fait partie intégrante de la démarche **TDD**. Chaque étape de correction vise à résoudre l’erreur actuelle pour progressivement construire une implémentation validée par les tests.

Pour obtenir un test vert rapidement, j’ai donc commencé par créer l’**Entity Country**, ce qui a permis de modifier le message d’erreur.

![Second error Unit Test](/assets/images/2025-02-17/first-bis-step-test.png)

J’ai ensuite créé une method ***create*** dans l’**Entity Country**. Dans un premier temps, cela ne fait rien, mais elle permet de faire progresser le passage au vert du test.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Country\Domain\Entity;

final readonly class Country
{
    public static function create(): self
    {

    }
}
{% endhighlight %}

Le message d’erreur dit maintenant que le **Value Object CountryId** n’existe pas.

![Third error Unit Test](/assets/images/2025-02-17/second-step-test.png)

Afin de régler le problème, je le crée, mais le plus simplement possible.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Country\Domain\ValueObject;

final readonly class CountryId
{
    
}
{% endhighlight %}

Le message suivant précise qu’il ne trouve pas la méthode fromString, que je rajoute également.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Country\Domain\ValueObject;

final readonly class CountryId
{
    public static function fromString(string $value): self
    {
        return new self();
    }
}
{% endhighlight %}

On a désormais un message qui dit que le Value Object CountryName n’existe pas.

![Fourth error Unit Test](/assets/images/2025-02-17/third-step-test.png)

Je crée donc cette classe, et comme lors de la création du Value Object CountryId, j’anticipe en ajoutant une method fromString.

### Vision du futur

L’idée est de n’anticiper que de petites portions de code, étape par étape, afin de rester concentré sur l’objectif principal : faire passer le test au vert le plus vite possible.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Country\Domain\ValueObject;

final readonly class CountryName
{
    public static function fromString(string $value): self
    {
        return new self();
    }
}
{% endhighlight %}

Le message d’erreur des tests indique maintenant que la méthode Factory de l’Entity Country ne retourne pas d’Entity. Pour résoudre ce problème, j’adopte l’approche la plus simple possible.

![Fifth error Unit Test](/assets/images/2025-02-17/fifth-step-test.png)

Pour le corriger je fais en sorte que la méthode ***create*** retourne une **Entity Country**.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Country\Domain\Entity;

final readonly class Country
{
    public static function create(): self
    {
        return new self();
    }
}
{% endhighlight %}

Maintenant, la method retourne bien une **Entity**, mais les tests signalent une nouvelle erreur : l’absence de Method pour exposer mon **id**. Ce genre de problème se reproduira également pour l’attribut **name**.

![Sixth error Unit Test](/assets/images/2025-02-17/sixth-step-test.png)

Je vais maintenant devoir créer ces Method. Avant de lex implémenter, il est crucial de savoir ce qu’elles doivent retourner. Pour aller au plus simple, elles vont renvoyer exactement ce que le test attend qu’elles retournent.

Ce qui nous donne donc :

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Country\Domain\Entity;

use App\Country\Domain\ValueObject\CountryId;
use App\Country\Domain\ValueObject\CountryName;

final readonly class Country
{
    public static function create(): self
    {
        return new self();
    }

    public function id(): CountryId
    {
        return CountryId::fromString('af785dbb-4ac1-4786-a5aa-1fed08f6ec26');
    }

    public function name(): CountryName
    {
        return CountryName::fromString(
            'France'
        );
    }
}
{% endhighlight %}

Le test devrait maintenant être réussi. Cependant, dans les **Value Object**, il manque la method ***value*** pour récupérer leur valeur. Je vais donc l’ajouter.

Une autre problématique réside dans l’assignation de la valeur passée dans la méthode ***fromString***. Je vais régler ces deux problèmes en une seule opération.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Country\Domain\ValueObject;

final readonly class CountryId
{
    public function __construct(
        private string $value,
    ) {}

    public static function fromString(string $value): self
    {
        return new self($value);
    }

    public function value(): string
    {
        return $this->value;
    }
}
{% endhighlight %}

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Country\Domain\ValueObject;

final readonly class CountryName
{
    public function __construct(
        private string $value,
    ) {}

    public static function fromString(string $value): self
    {
        return new self($value);
    }

    public function value(): string
    {
        return $this->value;
    }
}
{% endhighlight %}

Voici donc à quoi ressemble les **Value Object** à ce stade. L’objectif de faire passer les tests au vert est atteint, ce qui est parfait.

![Seventh error Unit Test](/assets/images/2025-02-17/seventh-step-test.png)

Tout cela est bien beau, mais nous avons un peu triché en mettant les valeurs en dur dans l’**Entity**. Nous avons atteint l’objectif du test rapidement, de la manière la plus naïve possible. Il faut désormais aborder la deuxième étape : refactorer le code pour l’améliorer et le rendre vraiment fonctionnel.

### Deuxième étape : rendre le code fonctionnel

Pour cela, il faut que l’**Entity Country** dispose de deux propriétés distinctes pour l’**id** et le **name**. Il faut ensuite que les method ***id*** et ***name*** retournent les valeurs des propriétés. Le refactoring doit être réalisé par petites étapes, en veillant à ne travailler que sur ce qui est couvert par le test. Puisque le test de cet exemple se concentre sur la method **create**, je vais me limiter à refactorer du code qui concerne cette method, sans toucher à d’autres **Entities**.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Country\Domain\Entity;

use App\Country\Domain\ValueObject\CountryId;
use App\Country\Domain\ValueObject\CountryName;

final readonly class Country
{
    public function __construct(
        private CountryId $id,
        private CountryName $name,
    ) {}

    public static function create(): self
    {
        return new self();
    }

    public function id(): CountryId
    {
        return $this->id;
    }

    public function name(): CountryName
    {
        return $this->name;
    }
}
{% endhighlight %}

Voilà quoi ressemble l’**Entity**. Relançons maintenant les tests pour voir si cela marche toujours.

![Eighth error Unit Test](/assets/images/2025-02-17/eighth-step-test.png)

Et là, c’est le drame ! Le test est cassé. Retour à la case départ : le faire passer vert au plus vite.
D’après le message d’erreur, le constructeur de l’**Entity Country** doit prendre deux paramètres. Or, en regardant de plus près le code de l’**Entity**, on remarque qu’aucun argument n’est passé au constructeur dans la method **create**. Il faut cependant que cette method prenne ces deux paramètres pour pouvoir les transmettre au ***constructeur*** de l’**Entity**.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Country\Domain\Entity;

use App\Country\Domain\ValueObject\CountryId;
use App\Country\Domain\ValueObject\CountryName;

final readonly class Country
{
    public function __construct(
        private CountryId $id,
        private CountryName $name,
    ) {}

    public static function create(
        CountryId $id,
        CountryName $name,
    ): self
    {
        return new self(
            $id,
            $name,
        );
    }

    public function id(): CountryId
    {
        return $this->id;
    }

    public function name(): CountryName
    {
        return $this->name;
    }
}
{% endhighlight %}

En relançant le test, celui-ci est désormais dans le vert.

![Nineth error Unit Test](/assets/images/2025-02-17/nineth-step-test.png)

On a donc créé la method **create** de l’**Entity Country** en suivant les bonnes pratiques du TDD.

### Ce qu’il faut retenir

En appliquant le **TDD**, on peut être sûr que le test reflète parfaitement le besoin du métier et que ce dernier est correctement validé. Je vous recommande de l’adopter autant que possible, car cela facilite la conception de vos use case métier et garantit que votre code métier à un code coverage le plus proche possible des 100%.

## Ajouter de la valeur aux Value Object

Pour renforcer la valeur des **Value Objects**, il est essentiel d’ajouter des vérifications sur leur contenu. On peut par exemple valider que la **Value Object** ***CountryId*** est bien un ***UUID***: pour cela, on peut utiliser une librairie existante ou écrire son propre code. Il faut, bien entendu, commencer par écrire un test, puis implémenter la validation dans l’**Entity** correspondante.

Voici à quoi ressemble le test :

{% highlight php linenos %}
public function testCreateBadIdNotUuid(): void
{
    $this->expectException(\InvalidArgumentException::class);

    Country::create(
        CountryId::fromString('12'),
        CountryName::fromString('France'),
    );
}
{% endhighlight %}

Sans surprise, il ne passe pas.

![Ten error Unit Test](/assets/images/2025-02-17/ten-step-test.png)

Pour faire passer le test au vert facilement, on doit ajouter une vérification dans le *Value Object* pour s’assurer que la valeur passée est bien un ***UUID***.

Pour cela, j’utilise la librairie PHP : https://github.com/webmozarts/assert. Premièrement, je l’installe en suivant la documentation officielle. Ensuite, il suffit d’ajouter la vérification appropriée pour émettre l’exception voulue.
Voici le code de la **Value Object** mis à jour :

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Country\Domain\ValueObject;

use Webmozart\Assert\Assert;

final readonly class CountryId
{
    private string $value;

    public function __construct(
        string $value,
    ) {
        Assert::uuid($value);

        $this->value = $value;
    }

    public static function fromString(string $value): self
    {
        return new self($value);
    }

    public function value(): string
    {
        return $this->value;
    }
}
{% endhighlight %}

Le test devrait passer :

![Eleven error Unit Test](/assets/images/2025-02-17/eleven-step-test.png)

Parfait !
Ce **Domain**, très simple et avec peu de règles, est terminé. Dans le futur nous verrons des **Domain** plus complexes.

## Mise en place des use case

Nous allons maintenant développer les use case dans le Domain que nous venons de créer.

### Création du Country

Nous commençons par importer des produits depuis un fichier récupéré sur Internet.

![Import countries use case](/assets/images/2025-02-17/import-country-usecase.png)

La première étape est d’écrire le test du **Primary Adapter** qui est la ***Command Symfony ImportCountryCommand***.

{% highlight php linenos %}
public function testExecute(): void
{
    self::bootkernel();
    $application = new Application(self::$kernel);
    $command = $application->find('country:import');
    $commandTester = new CommandTester(Scommand);
    $commandTester->execute([]);
    $commandTester->assertCommandIsSuccessful();
    $output = $commandTester->getDisplay();
    $this->assertStringContainsString('[OK] Countries created: 241', $output);
}
{% endhighlight %}

Le code en question est lié à ***Symfony***, donc je ne m’y attarderai pas trop longtemps. L’essentiel ici est de comprendre l’assert qui valide le test : à la fin de la commande, le message ‘[OK] Countries created: 241’ doit s’afficher, avec le nombre de pays créés.

Une fois cette vérification faite, nous pouvons passer au code. Celui de la ***Command Symfony*** n’est pas très intéressant car il s’agit principalement de la lecture de fichier, ce qui relève de la couche **Infrastructure**, et non du **Domain**.

{% highlight php linenos %}
private function handleFile(): int
{
    $countryCreated = 0;
    $handle = fopen($this->getFilePath(), 'r');

    if ($handle === false) {
        throw new \RuntimeException('Unable to open file');
    }

    while (($data = fgetcsv($handle, 1000)) !== false) {
        if ($data[4] === null) {
            continue;
        }

        $this->commandBus->dispatch(new CreateCountryCommand($data[4]));
        ++$countryCreated;
    }
    fclose($handle);

    return $countryCreated;
}
{% endhighlight %}

### Vision du futur

Ici on pourrait aussi mettre en forme un tableau avec tous les noms des pays et dispatch une **Command** pour importer tous les pays d'un seul coup. Cette façon de faire respecterais plus le **CQRS** ou le changement sur le système doit être englobé dans une seule **Command**. Je pense que ce changement devrait être fait.

Voici un extrait de la ***Command Symfony***, qui montre la partie responsable de la lecture du fichier et du dispatch de la **Command**. Je passe rapidement cette section, car elle n’est pas cruciale ici. Ce qu’il faut retenir, c’est que le fichier est lu et une **Command** est dispatchée avec le **nom du pays** qu’il contient. Pour ce **Domain**, je n’ai besoin d’aucune autre information, donc je ne prends que le **nom**. Cependant, on pourrait très bien imaginer traiter d’autres informations, selon les besoins du **Domain Country** ou du **Core Domain**.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Country\Application\Command;

use TegCorp\SharedKernelBundle\Application\Command\CommandInterface;

/**
 * @implements CommandInterface<void>
 */
final readonly class CreateCountryCommand implements CommandInterface
{
    public function __construct(
        public string $name,
    ) {
    }
}
{% endhighlight %}

La Command vraiment très simple ne prend qu’un **nom**.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Country\Application\Command;

#[AsCommandHandler]
final readonly class CreateCountryCommandHandler
{
    public function __construct(
        private CountryRepositoryInterface $countryRepository,
        private DomainEventDispatcherInterface $dispatcher,
        private IdFactory $idFactory,
    ) {
    }

    /**
     * @throws CountryAlreadyExistsException
     */
    public function __invoke(CreateCountryCommand $command): void
    {
        if ($this->countryRepository->ofName(CountryName::fromString($command->name)) !== null) {
            throw new CountryAlreadyExistsException();
        }

        $country = Country::create(
            CountryId::fromString($this->idFactory->create()),
            CountryName::fromString($command->name),
        );

        $this->countryRepository->add($country);

        $this->dispatcher->dispatch($country);
    }
}
{% endhighlight %}

Le **CommandHandler** fait partie de la couche **Application**, il permet de relier la couche **Infrastructure** au **Domain**. Le **CommandHandler** vérifie si le **Country** n’existe pas déjà, en utilisant le **Repository**. Si le pays existe déjà, on ***throw une exception***. On peut se permettre de rechercher par **nom** pour garantir l’unicité, car chaque **nom de pays** est unique. La méthode **Factory** de notre **Entity Country** permet de créer le Pays. Une fois créé, on l’enregistre dans le système à l’aide du **Repository**. Pour finir, on dispatche les **Domain Events** liés à la création du **Country**.

Le **Repository** est une notion du **Domain**, nous créons donc une ***Interface*** pour utiliser l’**Inversion de dépendance** : on déclare une ***Interface*** dans le **Domain** pour un **Repository** (avec la method ***add*** par exemple), et on implémente dans la couche **Infrastructure** la method ***add*** du **Repository** qui enregistre la nouvelle **Entity** dans la base de données (dans notre cas).

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Country\Domain\Repository;

/**
 * @extends RepositoryInterface<Country>
 */
interface CountryRepositoryInterface extends RepositoryInterface
{
    public function ofName(CountryName $name): ?Country;

    public function add(Country $country): void;
}
{% endhighlight %}

Dans le **Repository**, on définit les deux method nécessaires pour interagir avec le Domain :
– ***ofName***: récupère un pays par son **nom** ou renvoie ***null*** si le **Country** n’existe pas
– ***add***: se charge d’enregistrer le **Country** dans le système

L’implémentation du Repository n’est pas vraiment intéressante, car liée à Symfony, mais je la montre quand même pour compléter l’implémentation.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Country\Infrastructure\Doctrine\Repository;

final class CountryDoctrineRepository implements CountryRepositoryInterface
{
    private const string ENTITY_CLASS = CountryDoctrine::class;
    private const string ALIAS = 'country';

    public function __construct(EntityManagerInterface $entityManager)
    {
        parent::__construct($entityManager, self::ENTITY_CLASS, self::ALIAS);
    }

    #[\Override]
    public function ofName(
        CountryName $name,
    ): ?Country {
        $country = $this->entityManager
            ->getRepository(self::ENTITY_CLASS)
            ->findOneBy(['name' => $name->value()])
        ;

        if ($country === null) {
            return null;
        }

        return CountryMapper::toDomain($country);
    }

    #[\Override]
    public function add(Country $country): void
    {
        $countryDoctrine = CountryMapper::toInfrastructurePersist($country);

        $this->entityManager->persist($countryDoctrine);
        $this->entityManager->flush();
    }
}
{% endhighlight %}

Je ne rentrerais pas dans le détail de l’implémentation, mais nous retrouvons bien les deux method déclarées dans l’***Interface*** du **Repository**.

Cela clôt l’implémentation du use case pour la création du **Country**. Je vais maintenant ajouter une petite fonctionnalité dans le **Domain**: l’enregistrement d’un **Domain Event** lorsqu’une **Entity Country** est créée.

## Enregistrement d’un Domain Event lié à la création d’une Entity Country

Pour cela, j’ai créé une ***Interface*** que les **Entity** doivent implémenter pour enregistrer leurs événements métiers. Ensuite, j’utilise un ***Event Dispatcher*** pour dispatcher tous les événements d’une **Entity**.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Shared\Domain\Entity;

use App\Shared\Domain\Event\DomainEventInterface;

interface EntityWithDomainEventInterface
{
    /**
     * @return DomainEventInterface[]
     */
    public static function getRecordedEvent(): array;

    public static function recordEvent(DomainEventInterface $event): void;

    public static function eraseRecordedEvents(): void;
}
{% endhighlight %}

Cette ***Interface*** définit trois method :
1. ***getRecordedEvent*** : permet de récupérer tous les **Domain Event** enregistrés sur une **Entity**
2. ***recordEvent*** : permet d’enregistrer un **Domain Event** suite à une action sur le **Domain**
3. ***eraseRecordedEvents*** : permet de supprimer les **Domain Event** quand on les a dispatchés afin de ne pas les traiter deux fois.

Pour créer les method liées à cette ***Interface***, il est nécessaire que les **Domain Event** eux-mêmes implémentent une ***Interface***. Celle-ci sera ensuite utilisée pour typer les **Domain Event**.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Shared\Domain\Event;

interface DomainEventInterface
{
}
{% endhighlight %}

Il faut ensuite faire en sorte que l’**Entity Country** implémente ces method. Pour ce faire, j’ai choisi de créer un ***Trait PHP***, ce qui permet d’ajouter ces method à l’**Entity** sans avoir à utiliser l’héritage.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Shared\Domain\Entity;

use App\Shared\Domain\Event\DomainEventInterface;

trait EntityDomainEventTrait
{
    private static array $recordedEvents = [];

    #[\Override]
    public static function getRecordedEvent(): array
    {
        return self::$recordedEvents;
    }

    #[\Override]
    public static function recordEvent(DomainEventInterface $event): void
    {
        self::$recordedEvents[] = $event;
    }

    #[\Override]
    public static function eraseRecordedEvents(): void
    {
        self::$recordedEvents = [];
    }
}
{% endhighlight %}

Pour que l’**Entity Country** dispose de ces method, elle doit implémenter l’***Interface*** ***EntityWithDomainEventInterface*** et utiliser le ***trait*** ***EntityDomainEventTrait***.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Country\Domain\Entity;

final class Country implements EntityWithDomainEventInterface
{
    use EntityDomainEventTrait;

    public function __construct(
        private CountryId $id,
        private CountryName $name,
    ) {
    }

    public static function create(
        CountryId $id,
        CountryName $name,
    ): self {
        return new self(
            $id,
            $name,
        );
    }

    public function id(): CountryId
    {
        return $this->id;
    }

    public function name(): CountryName
    {
        return $this->name;
    }
}
{% endhighlight %}

À présent, l’**Entity Country** est prête à enregistrer des **Domain Event**. L’***Event*** sera en charge d’indiquer qu’un **Country** a été créé, avec pour valeurs l’**id** et le **nom**. Pour cela, on crée un ***Event*** **CountryCreated**. Un **Domain Event** doit être identifiable pour vérifier que le même ***Event*** n’est pas consommé plusieurs fois. Il doit également avoir une date de publication afin d’être traité dans l’ordre d’enregistrement,  si cet ordre à de l’importance.

Pour répondre à ces besoins, j’ai donc créé une ***Abstract Class*** qui permet d’ajouter ces informations à chaque ***Event*** que l’on créera en faisant de l’héritage.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Shared\Domain\Event;

use Ramsey\Uuid\Uuid;
use Symfony\Contracts\EventDispatcher\Event;

abstract class DomainEvent extends Event
{
    public readonly string $id;
    public readonly int $occurredOn;

    public function __construct(
    ) {
        $this->id = Uuid::uuid4()->toString();
        $this->occurredOn = time();
    }
}
{% endhighlight %}

On crée l’***Event*** qui va étendre cette classe et implémenter l’***Interface*** précédemment créée.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Country\Domain\Event;

use App\Shared\Domain\Event\DomainEvent;
use App\Shared\Domain\Event\DomainEventInterface;

final class CountryCreated extends DomainEvent implements DomainEventInterface
{
    public function __construct(
        public string $countryId,
        public string $name,
    ) {
        parent::__construct();
    }
}
{% endhighlight %}

Il ne reste plus qu’à enregistrer cet ***Event*** au moment de la création du **Country**.
Enregistrer un **Domain Event** à chaque action dans le système n’est pas obligatoire. C’est cependant très utile pour respecter la **séparation des préoccupations** (**Separation of concerns).** Le **CommandHandler** a pour but de créer l’**Entity Country**. Si d’autres actions doivent être effectuées, elles ne doivent pas faire partie de ce **CommandHandler**. Il est donc important de segmenter le code et les ***Event*** : c’est un bon moyen d’y parvenir. On pourrait imaginer que cet ***Event*** soit écouté pour effectuer des actions telles que l’écriture du **Country** dans le modèle de lecture, ou pour notifier un autre **Domain** qu’un nouveau **Country** a été créé.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Country\Domain\Entity;

final class Country implements EntityWithDomainEventInterface
{
    use EntityDomainEventTrait;

    public function __construct(
        private CountryId $id,
        private CountryName $name,
    ) {
    }

    public static function create(
        CountryId $id,
        CountryName $name,
    ): self {
        $country = new self(
            $id,
            $name,
        );

        $country::recordEvent(
            new CountryCreated(
                $country->id->value(),
                $country->name->value(),
            )
        );

        return $country;
    }

    public function id(): CountryId
    {
        return $this->id;
    }

    public function name(): CountryName
    {
        return $this->name;
    }
}
{% endhighlight %}

L'***Event*** est enregistré et sera dispatché dans le **CommandHandler** grâce à cette ligne :

{% highlight php linenos %}
$this->dispatcher->dispatch($country);
{% endhighlight %}

### Vision du futur
C'est très important de ***dispatch*** le **Domain Event** après que l'action soit effectué.

La partie création de l’Entity Country est désormais terminée.

## Exposer une liste de pays

On souhaite maintenant pouvoir exposer une liste de pays via une API. Pour cela nous allons utiliser ***Api Platform*** (https://api-platform.com/), un framework pour faciliter la création d’API REST en respectant les standards du secteur.

### Explication rapide d’API Platform

Ce framework très complet se base sur des classes ***Resources*** dans lesquelles nous déclarons des opérations. Ces opérations sont liées aux verbes HTTP (GET, PUT, POST, DELETE) et chaque opération se voit attribuer des ***Processor*** ( pour PUT, DELETE et POST) et des ***Provider*** (pour GET).

***API Platform*** offre une multitude d’autres fonctionnalités. J’en effleure seulement la surface pour que vous puissiez comprendre ce qui se passe en lisant le code.

Voilà mon use case :

![Read list product](/assets/images/2025-02-17/reas-list-product.png)

Comme expliqué plus haut, en utilisant ***Api Platform***, le **Primary Adapter** est le ***Country Provider***.

J’écris d’abord un test rapide pour récupérer la liste de mes pays.

{% highlight php linenos %}
public function testGetCollection(): void
{
    $this->get('/api/countries');

    $this->assertResponseIsSuccessful();
    $this->assertJsonContains([
        '@context' => '/api/contexts/Country',
        '@id' => '/api/countries',
        '@type' => 'Collection',
        'member' => [
            [
                '@type' => 'Country',
                'name' => 'Afghanistan',
            ],
            [
                '@type' => 'Country',
                'name' => 'Afrique du Sud',
            ],
        ],
        'totalItems' => 30,
    ]);
}
{% endhighlight %}

Dans ce test, je fais une requête sur l’uri ‘/api/countries’, qui renvoie une liste de **Country** avec leur **nom**. Je dois récupérer trente pays, car la pagination est activée pour les récupérer par lot de trente (ce qui géré est par défaut par ***Api Platform***).

On commence donc pas créer la ***Resource Country***, qui est différente de l’**Entity Country**. La responsabilité de cette ***Resource*** est de porter l’opération d’***Api Platform*** et de renvoyer les données via notre API.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Country\Infrastructure\ApiPlatform\Resource;

#[ApiResource(
    shortName: 'Country',
)]
#[GetCollection(
    '/countries',
    filters: [CountryFilter::class],
    provider: GetCountryCollectionProvider::class,
)]
final class GetCollectionCountryResource
{
    public function __construct(
        #[ApiProperty(readable: false, writable: false, identifier: true)]
        public ?AbstractUid $id = null,
        #[ApiProperty]
        public ?string $name = null,
    ) {
    }

    public static function fromModel(Country $country): self
    {
        return new self(
            new Uuid($country->id),
            $country->name,
        );
    }
}
{% endhighlight %}

On ajoute une opération de type ***GetCollection***, car on veut récupérer une liste de ***Resource Country***. On spécifie un ***Provider*** qui sera chargé de dispatcher la **Query** pour lire les données dans le système. On définit ensuite les champs à retourner, on ajoute l’**attribut** ***ApiProperty*** pour indiquer que ces propriétés doivent être exposées via l’API. Pour finir, on définit une method ***fromModel*** qui permet de transformer les **Entity Country** en **Resource Country**.

J’ai également précisé un argument ***filters*** dans l’***attribut*** ***GetCollection*** Cela permet de définir un ou plusieurs filtres, utiles pour le côté interne d’***Api Platform*** et pour apparaître dans les metadata du retour de notre API. Je ne rentre pas plus dans le détail car ce n’est pas essentiel pour l’apprentissage du **DDD**, mais voilà à quoi ressemble cette classe ***CountryFilter***.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Country\Infrastructure\ApiPlatform\OpenApi;

use ApiPlatform\Metadata\FilterInterface;
use Symfony\Component\PropertyInfo\Type;

final readonly class CountryFilter implements FilterInterface
{
    #[\Override]
    public function getDescription(string $resourceClass): array
    {
        return [
            'name' => [
                'property' => 'name',
                'type' => Type::BUILTIN_TYPE_STRING,
                'required' => false,
            ],
        ];
    }
}
{% endhighlight %}

Ainsi que le Provider :

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Country\Infrastructure\ApiPlatform\State\Provider;

/**
 * @implements ProviderInterface<GetCollectionCountryResource>
 */
final readonly class GetCountryCollectionProvider implements ProviderInterface
{
    public function __construct(
        private QueryBusInterface $queryBus,
        private Pagination $pagination,
    ) {
    }

    /**
     * @return Paginator<GetCollectionCountryResource>|list<GetCollectionCountryResource>
     */
    #[\Override]
    public function provide(Operation $operation, array $uriVariables = [], array $context = []): array|Paginator
    {
        $name = $context['filters']['name'] ?? null;
        $page = $limit = 0;

        if ($this->pagination->isEnabled($operation)) {
            $page = $this->pagination->getPage($context);
            $limit = $this->pagination->getLimit($operation, $context);
        }

        $models = $this->queryBus->ask(new GetCountriesQuery($name, $page, $limit));

        $resources = [];

        foreach ($models as $model) {
            $resources[] = GetCollectionCountryResource::fromModel($model);
        }

        if (null !== $paginator = $models->paginator()) {
            $resources = new Paginator(
                new \ArrayIterator($resources),
                (float) $paginator->getCurrentPage(),
                (float) $paginator->getItemsPerPage(),
                (float) $paginator->getLastPage(),
                (float) $paginator->getTotalItems(),
            );
        }

        return $resources;
    }
}
{% endhighlight %}

Ce ***Provider*** contient beaucoup de code spécifique à ***Api Platform***, mais je vais quand même prendre un peu de temps pour vous l’expliquer, car cela touche à certaines réflexions concernant le **DDD**.

Premièrement, je récupère le filtre par nom s’il est spécifié et j’initialise la pagination à 0. Puis je récupère la pagination dans l’url si c’est activé. Ensuite, je dispatche la **Query**, responsable de lire les données dans le système. Après cela, je parcours toutes les **Entity** retournées par la **Query** et je les transforme en ***Resource***. Pour finir, je mets en place la pagination (si elle est activée pour cette ***Resource***) et je retourne les ***Resources***, qu’elles soient paginées ou non.

### Vision du futur
Gérer la pagination ne relève pas de la responsabilité du **QueryHandler**, c’est une problématique de l’**Infrastructure** ou de **l’Application**. À nous de nous adapter. Pour faire cela, il faudrait mettre en place un modèle de lecture (via des projections), mais je me suis rendu compte de cela plus tard. Pour l’instant, la pagination est donc portée par le **Repository**, ce qui n’est pas une bonne pratique. Si vous voulez un exemple plus détaillé, j’ai écrit un article à ce sujet que vous pouvez retrouver [*ici*](https://huguesgobet.com/fr/other/2024-12-16). 

Maintenant que tout cela est plus clair, voici la Query qu’on dispatch.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Country\Application\Query;

use App\Country\Application\Adapter\CountryRepositoryInterface;
use App\Shared\Application\Query\QueryInterface;

/**
 * @implements QueryInterface<CountryRepositoryInterface>
 */
final readonly class GetCountriesQuery implements QueryInterface
{
    public function __construct(
        public ?string $name = null,
        public ?int $page = null,
        public ?int $limit = null,
    ) {
    }
}
{% endhighlight %}

Ensuite le QueryHandler :

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Country\Application\Query;

use App\Country\Application\Adapter\CountryRepositoryInterface;
use App\Shared\Application\Query\AsQueryHandler;

#[AsQueryHandler]
final readonly class GetCountriesQueryHandler
{
    public function __construct(
        private CountryRepositoryInterface $countryRepository,
    ) {
    }

    public function __invoke(GetCountriesQuery $query): CountryRepositoryInterface
    {
        $countryRepository = $this->countryRepository;

        $countryRepository = $countryRepository->sortName();

        if ($query->name !== null) {
            $countryRepository = $countryRepository->withName($query->name);
        }

        if ($query->page !== null && $query->limit !== null) {
            $countryRepository = $countryRepository->withPagination($query->page, $query->limit);
        }

        return $countryRepository;
    }
}
{% endhighlight %}

Dans ce **QueryHandler**, je récupère le **Repository** et je lui indique que je veux trier les résultats par nom. Si un filtre par **nom** est spécifié, je demande au **Repository** de ne récupérer que les **Country** correspondant à ce filtre. Si la pagination est activée, je demande au **Repository** de l’appliquer (donc de limiter à trente éléments et de récupérer les **Country** à partir de la page 2, par exemple).

Ce qui peut surprendre, c’est le retour du **Repository** : cela permet de coller au fonctionnement d’***Api Platform*** et à sa manière de gérer la pagination. Le fait de retourner le **Repository** n’enfreint aucune règle du **CQRS** ou du **DDD**, c’est pourquoi j’ai accepté de suivre cette approche.

### Vision du futur

En utilisant un modèle de lecture, je pourrais m’abstraire de cette partie un peu étrange et le **Repository** ne porterait plus la responsabilité de la lecture, ce qui éliminerait les bizarrerie au sein du **Domain**.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Country\Domain\Repository;

use App\Country\Domain\Entity\Country;
use App\Shared\Domain\Repository\RepositoryInterface;

/**
* @extends RepositoryInterface<Country>
*/
interface CountryRepositoryInterface extends RepositoryInterface
{
    public function ofName(string Sname): ?Country;

    public function add (Country Scountry): void;

    public function withName(string $name): self;

    public function sortName(): self;
}
{% endhighlight %}

Le **Repository** ressemble à ça. Je vais vous montrer l’implémentation avec ***Doctrine***. Je ne l’expliquerai pas car cela n’a aucun intérêt pour l’article. Je trouve cependant important de vous montrer tout le code pour cette partie.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Country\Infrastructure\Doctrine\Repository;

/**
 * @extends DoctrineRepository<Country>
 */
final class CountryDoctrineAdapter extends DoctrineRepository implements CountryAdapterInterface
{
    private const string ENTITY_CLASS = Country::class;
    private const string ALIAS = 'country';

    public function __construct(
        DocumentManager $documentManager,
    ) {
        parent::__construct($documentManager, self::ENTITY_CLASS);
    }

    #[\Override]
    public function ofName(string $name): ?Country
    {
        return $this->documentManager->getRepository(self::ENTITY_CLASS)->findOneBy(['name' => $name]);
    }

    #[\Override]
    public function add(Country $country): void
    {
        $this->documentManager->persist($country);
        $this->documentManager->flush();
    }

    #[\Override]
    public function withName(
        string $name,
    ): self {
        return $this->filter(static function (Builder $qb) use ($name): void {
            $qb->field('name')->text($name);
        });
    }

    #[\Override]
    public function sortName(): self
    {
        return $this->filter(static function (Builder $qb): void {
            $qb->sort('name', 'ASC');
        });
    }
}
{% endhighlight %}

C’était un article très copieux ! J’y ai abordé le **TDD** ainsi que la création d’un **Supporting Domain**. Je vous ai également présenté deux nouveaux use case de **Domain**. Pourtant, je n’ai pas encore eu l’occasion d’aborder le **Core Domain**. Ce sera l’objet d’un prochain article, où je discuterai de la communication entre les **Domain** et les nombreuses erreurs que j’ai commises à ce sujet.