---
layout: article
title: "Logbook of Learning Domain-Driven Design: Day 5"
date:   2025-02-17 08:00:00
categories: ddd-logbook
lang: en
resume: "This fifth article explains how to establish a Supporting Domain for managing countries while demonstrating the Test-Driven Development process for creating and validating Entities and Value Objects. It also addresses exposing a country list via Api Platform, emphasizing concern separation and Domain consistency."
permalink: /ddd-logbook/2025-02-17
---

In this article, I will discuss the concept of **Country** within the framework of a **Supporting Domain**, using a **Domain Country** as an example. I will also detail the exposure of an API that allows retrieving information about a **Country** and verifying its existence. I will begin by revisiting the definition of a **Supporting Subdomain**, then explain how to implement it step by step.

## Supporting Subdomain: What Is It?

As the name suggests, a **Supporting Subdomain** is, first and foremost, a **Subdomain**: it represents a part of the **Domain** that can be separated into a **module**. In a business project, a dedicated team could be responsible for it. A **Subdomain** can also have its own **Ubiquitous Language** when it belongs to a **Bounded Context** different from the **Core Domain**.

Although important to the business, the **Supporting Subdomain** is less critical than the **Core Domain**. Creating a **Supporting Domain** is only relevant if it provides specific value or addresses a particular need.

In the case of our wine bottle management API, I created a **Domain Country** to separate this concept from the **Core Domain**, which manages the **Bottle Inventory**. This separation helps organize the code into two distinct **Bounded Contexts**. Additionally, this **Domain** is essential for providing a list of countries when creating a bottle while ensuring that the country actually exists. However, since it remains less crucial than the **Core Domain**, it fits perfectly within the definition of a **Supporting Subdomain**.

## How to Implement the Domain?

In this article, I will illustrate the implementation of the **Domain** through two use cases. I have chosen not to separate these cases into multiple sections because they are quite repetitive and do not pose a significant challenge in terms of **Domain-Driven Design (DDD)**. Here are the two scenarios I will cover:
- Creating a **Country**
- Retrieving the list of all **Countries**

I propose tackling this topic using a different approach than the one taken for the authentication use case. Therefore, I will start by defining the **Entity**, along with unit tests, to integrate a **Test-Driven Development (TDD)** approach.

## Quick Introduction to TDD

**Test-Driven Development (TDD)** is a very interesting development methodology that would deserve a dedicated article (maybe an idea for a future special edition). If you’re looking for more in-depth information, I highly recommend the book *Test-Driven Development: By Example* by Kent Beck.

The fundamental idea behind this methodology is to write tests before writing the corresponding functionality. However, the essence of this approach lies in a step-by-step, progressive, and rigorous process.

To illustrate this method, I will walk you through the tests related to the creation of a **Country Entity**.

### Step 1: Testing the Creation of a Country Entity

We will begin with a simple test to validate the correct creation of a **Country Entity**.

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

This test precisely defines the expected output of the **Factory** ***create*** method. In a **TDD** approach, the goal is to obtain a green test (i.e., a successful test) as quickly as possible.

Upon the first execution of the test, an error occurs immediately: the class does not exist.

![First error Unit Test](/assets/images/2025-02-17/first-step-test.png)

This situation is completely normal and is an integral part of the **TDD** process. Each correction step aims to fix the current error in order to gradually build an implementation validated by tests.

To quickly obtain a green test, I started by creating the Country Entity, which changed the error message.

![First bis error Unit Test](/assets/images/2025-02-17/first-bis-step-test.png)

I then created a ***create*** method in the **Country Entity**. At first, this method does nothing, but it helps progress toward a successful test.

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

The new error message indicates that the **Value Object CountryId** does not exist.

![Second error Unit Test](/assets/images/2025-02-17/second-step-test.png)

To resolve this issue, I created the **CountryId Value Object**, keeping it as simple as possible.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Country\Domain\ValueObject;

final readonly class CountryId
{
    
}
{% endhighlight %}

The next message states that it cannot find the ***fromString*** method. So, I added it.

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

At this point, we get another message indicating that the **Value Object CountryName** does not exist.

![Third error Unit Test](/assets/images/2025-02-17/third-step-test.png)

I then created the class, and just like with **CountryId**, I anticipated the need by adding a ***fromString*** method.

### Future Considerations

The idea is to only anticipate small portions of code, step by step, to stay focused on the main goal: making the test pass (turn green) as quickly as possible.

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

The test error message has now evolved and indicates that the **Factory** method of the **Country Entity** does not return an **Entity**. To fix this issue, we adopt the simplest possible approach.

![Fifth error Unit Test](/assets/images/2025-02-17/fifth-step-test.png)

To correct this, I ensure that the ***create*** method returns a **Country Entity**.

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

Now, the method does return an **Entity**, but the tests report a new error: the absence of a method to expose the **identifier**. The same issue will also arise for the **name** attribute.

![Sixth error Unit Test](/assets/images/2025-02-17/sixth-step-test.png)

I now need to create these methods. Before implementing them, it is crucial to determine what they should return. To keep it simple, they will return exactly what the test expects them to return.

At this point: 

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

The test should now pass. However, in the **Value Objects**, the ***value*** method is missing, which is needed to retrieve their value. I will now add it.

Another issue lies in assigning the value passed into the ***fromString*** method. I will resolve both problems in a single operation.

Current State of the **Value Objects**.

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

At this stage, here’s what the **Value Objects** look like. The goal of making the tests pass (turn green) has been achieved, which is perfect.

![Seventh error Unit Test](/assets/images/2025-02-17/seventh-step-test.png)

All of this is great, but we took a shortcut by hardcoding the values inside the **Entity**. We achieved the test's goal quickly, in the most naive way possible. Now, we need to move on to the second step: refactoring the code to improve it and make it fully functional.

### Step Two: Making the Code Functional

To achieve this, the **Country Entity** must have two distinct properties: one for the **ID** and one for the **name**. Additionally, the ***id*** and ***name*** methods should return the values of these properties. The refactoring must be done in small steps, ensuring that we only work on what is covered by the test. Since the test in this example focuses on the ***create*** method, I will limit my refactoring to code related to this method, without modifying other **Entities**.

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

This is what the **Entity** looks like now. Let’s rerun the tests to see if they still pass. 

![Eighth error Unit Test](/assets/images/2025-02-17/eighth-step-test.png)

And then... disaster strikes! The test fails. We’re back to square one: getting the test to pass again as quickly as possible.
According to the error message, the **Country Entity** constructor must take two parameters. However, looking at the **Entity**'s code, we notice that no arguments are passed to the constructor inside the ***create*** method. Therefore, this method must accept these two parameters so they can be passed to the **Entity**'s constructor.

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

After rerunning the test, it finally passes (turns green).

![Nineth error Unit Test](/assets/images/2025-02-17/nineth-step-test.png)

We have now created the **Country Entity**'s method following **TDD** best practices.

### Key Takeaways

By following **TDD**, we ensure that the test perfectly reflects the business requirement and that it is correctly validated. I highly recommend adopting this approach as much as possible, as it simplifies the design of business use cases and ensures that your business logic has near 100% code coverage.

## Adding Value to Value Object

To increase the value of **Value Objects**, it is essential to add validation rules for their content. For example, we can verify that the **CountryId Value Object** is a valid ***UUID***. To do this, we can either use an existing library or write our own validation logic. Of course, we start by writing a test, then implement the validation inside the corresponding **Entity**.

Here is what the test looks like:

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

As expected, it fails.

![Ten error Unit Test](/assets/images/2025-02-17/ten-step-test.png)

To make the test pass quickly, we need to add a verification in the **Value Object** to ensure that the provided value is a valid ***UUID***.
For this, I use the PHP library: https://github.com/webmozarts/assert. First, I install it following the official documentation. Then, I simply add the appropriate validation to throw the expected exception.
Here is the updated **Value Object** code:

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

Now, let’s run the test:

![Eleven error Unit Test](/assets/images/2025-02-17/eleven-step-test.png)

Perfect! 
This **Domain**, being very simple and with few rules, is now complete. In the future, we will explore more complex **Domains**.

## Implementing Use Cases

Now, we will proceed to develop the use cases within the Domain we just created.

### Creating a Country

We start by importing products from a file retrieved online.

![Import countries use case](/assets/images/2025-02-17/import-country-usecase.png)

The first step is to write the test for the **Primary Adapter**, which is the ***Symfony Command ImportCountryCommand***.

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

The code in question is related to ***Symfony***, so I won’t dwell on it too much. The key point here is understanding the assertion that validates the test: at the end of the command, the message "[OK] Countries created: 241" must be displayed, showing the number of countries created.

Once this verification is complete, we can move on to the code. The ***Symfony Command*** itself is not particularly interesting, as it mainly handles file reading, which falls under the **Infrastructure** layer , not the **Domain**.

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


### Future Considerations

Here, we could also format a table containing all the country names and dispatch a **Command** to import them all at once. This approach would better align with **CQRS**, where any change to the system should be encapsulated in a single **Command**. I believe this modification should be made.

Here is an excerpt from the ***Symfony Command***, showing the section responsible for reading the file and dispatching the **Command**. I will go over this part quickly, as it is not crucial here. What matters is that the file is read, and a **Command** is dispatched with the country name contained in it. For this **Domain**, I only need the name—no other information. However, depending on the needs of the **Domain Country** or the **Core Domain**, additional details could be processed.

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

The Command is very simple and only takes a **name** as input.

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

The **CommandHandler** is part of the **Application** layer. It serves as a bridge between the **Infrastructure** layer and the **Domain**. The **CommandHandler** first checks if the **Country** already exists using the **Repository**. If the **Country** is already present, an exception is thrown. Since each **Country** **name** is unique, we can safely search by **name** to ensure uniqueness. The **Factory** method of our **Country Entity** allows us to create the **Country**. Once created, it is stored in the system using the **Repository**. Finally, we dispatch the **Domain Events** related to the creation of the **Country**.

The **Repository** is a **Domain**-level concept, so we create an ***Interface*** to apply **Dependency Inversion**: We declare an ***Interface*** for a **Repository** (e.g., with an add method) inside the **Domain**. We then implement the add method in the **Infrastructure**  layer, which saves the new **Entity** in the database.

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

In the **Repository**, we define two essential methods to interact with the **Domain**:
– ***ofName***: Retrieves a **Country** by its name or returns null if the **Country** does not exist.
– ***add***: Saves the **Country** in the system.

The implementation of the **Repository** is not particularly complex since it is ***Symfony-related***, but I am including it here to complete the implementation.

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

I won’t go into the details of the implementation, but we can clearly see the two methods declared in the **Repository** ***Interface***. 

This concludes the implementation of the use case for creating a **Country**. Now, I will add a small feature to the **Domain**: registering a **Domain Event** when a **Country Entity** is created.

## Registering a Domain Event for the Creation of a Country Entity
To achieve this, I created an ***Interface*** that **Entities** must implement to record their business events. Then, I use an ***Event Dispatcher*** to dispatch all the events of an **Entity**.

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

This ***Interface*** defines three methods:
1. ***getRecordedEvent***: Retrieves all **Domain Events** recorded for an **Entity**.
2. ***recordEvent***: Records a **Domain Event** after an action in the **Domain**.
3. ***eraseRecordedEvents***: Deletes recorded **Domain Events** after they are dispatched, ensuring they are not processed twice.

To create the methods linked to this ***Interface***, **Domain Events** themselves must also implement an ***Interface***. This ***Interface*** will be used to type the **Domain Events**.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Shared\Domain\Event;

interface DomainEventInterface
{
}
{% endhighlight %}

Next, we must ensure that the **Country Entity** implements these methods. To do so, I chose to create a ***PHP Trait***, allowing us to add these methods to the **Entity** without using inheritance.

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

For the **Country Entity** to use these methods, it must: Implement the ***interface*** ***EntityWithDomainEventInterface***, and Use the Trait. ***EntityDomainEventTrait***.

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

At this point, the **Country Entity** is ready to record **Domain Events**. The **Domain Event** will indicate that a **Country** has been created, containing the **ID** and **name**. To implement this, we create a **Domain Event**: **CountryCreated**. A **Domain Event** must be identifiable to ensure the same **Domain Event** is not processed multiple times. It must also have a publication date, so it can be processed in the correct order if event sequencing is important.

To meet these requirements, I created an ***Abstract Class***, which adds these details to every **Domain Event** we create through inheritance.

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

Next, we create the ***Event*** that extends this class and implements the previously created ***Interface***.

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

The final step is to register this **Domain Event** when the **Country** is created.

Registering a **Domain Event** for every system action is not mandatory. However, it is very useful to maintain a clear separation of concerns. The **CommandHandler** is responsible for creating the **Country Entity**. If additional actions need to be performed, they should not be part of the **CommandHandler** itself. Segmenting code and **Domain Events** is a great way to achieve this separation. For instance, this **Domain Event** could be listened to in order to: Write the **Country** into the read model, and Notify another **Domain** that a new **Country** has been created.

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

The **Domain Event** is registered and will be dispatched in the **CommandHandler** using this line:

{% highlight php linenos %}
$this->dispatcher->dispatch($country);
{% endhighlight %}

### Future Considerations
It's very important to dispatch the **Domain Event** after the action has been performed.

The creation process for the **Country Entity** is now complete.

## Exposing a List of Countries

Now, we want to expose a list of countries via an API. To achieve this, we will use ***API Platform*** (https://api-platform.com/), a framework that simplifies the creation of REST APIs while adhering to industry standards.

### Quick Overview of API Platform

This comprehensive framework is based on ***Resource*** classes, where we declare operations. These operations are linked to HTTP verbs (GET, PUT, POST, DELETE), and each operation is assigned: A ***Processor*** (for PUT, DELETE, and POST). A ***Provider*** (for GET).
API Platform offers a vast array of features, but I will only scratch the surface to ensure you understand the key elements when reading the code.

Here is my use case:

![Read list product](/assets/images/2025-02-17/reas-list-product.png)

As explained earlier, when using ***API Platform***, the **Primary Adapter** is the **Country** ***Provider***.

I start by writing a quick test to retrieve my list of **countries**.

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

In this test, I send a request to the URI /api/countries, which returns a list of **Country** objects with their names. I expect to retrieve thirty **countries**, as pagination is enabled by default in ***API Platform***, grouping results into batches of thirty.

The next step is to create the ***Resource Country***, which is different from the **Country Entity**. The responsibility of this ***Resource*** is to: Manage the ***API Platform*** operation and return the requested data via the API.

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

To achieve this, we: Add a ***GetCollection*** operation, since we want to retrieve a list of ***Country Resources***. Specify a ***Provider***, which will be responsible for dispatching the **Query** to fetch the data. Define the fields to be returned. Add the ***ApiProperty*** attribute to indicate that these properties must be exposed via the API. Create a ***fromModel*** method to transform **Country Entities** into ***Country Resources***.

Additionally, I have specified a filters argument in the ***GetCollection*** attribute. This allows us to define one or multiple filters, which are useful: Internally for ***API Platform***. To appear in the API response metadata. I won’t go into further detail here, as it’s not essential for understanding **DDD**, but here is what the ***CountryFilter*** class looks like:

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

The ***Country Provider***:

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

This ***Provider*** contains a lot of ***API Platform***-specific code, but I will take a moment to explain it, as it touches on some important **DDD** considerations.

First, I retrieve the name filter (if specified) and initialize pagination at 0. Then, I extract the pagination parameters from the URL (if pagination is enabled). Next, I dispatch the **Query**, responsible for fetching data from the system. After that, I iterate over the **Entities** returned by the **Query** and convert them into ***Resources***. Finally, I apply pagination (if enabled) and return the ***Resources***, whether paginated or not.

### Future Considerations
Managing pagination should not be the responsibility of the **QueryHandler**—this is a concern of the **Infrastructure** or **Application** layer. We should adapt accordingly. To handle this correctly, we should implement a read model (via projections), but I realized this too late. For now, pagination is handled by the **Repository**, which is not a best practice. If you want a more detailed example, I have written an article on this topic [*ici*](https://huguesgobet.com/other/2024-12-16). 

Now that everything is clear, here is the **Query** we dispatch.

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

Within the **QueryHandler**:

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

I retrieve the **Repository** and specify that I want to sort results by name. If a **name** filter is provided, I instruct the **Repository** to only fetch matching **Countries**. If pagination is enabled, I tell the **Repository** to apply it (e.g., limiting results to thirty elements and fetching countries from page 2 onward).

What might seem unusual is that the **Repository** itself is returned—this is to align with ***API Platform***'s structure and its pagination handling. Returning the **Repository** does not violate **CQRS** or **DDD** principles, which is why I accepted this approach.

### Future Considerations

Using a read model would allow me to avoid this slightly unusual approach and remove pagination responsibility from the **Repository**, eliminating these oddities within the **Domain**.

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

This is what the **Repository** looks like. I will now show the ***Doctrine*** implementation. I won’t explain it, as it is not relevant to this article, but I believe it is important to show all the code for this section.

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

This was a very in-depth article! I covered **TDD** as well as the creation of a **Supporting Domain**. I also introduced two new **Domain** use cases. However, I haven’t yet had the opportunity to discuss the **Core Domain**. That will be the focus of a future article, where I will talk about communication between **Domains** and the many mistakes I made on this topic.