---
layout: article
title: "Logbook of Learning Domain-Driven Design: Day 4"
date:   2025-01-27 08:00:00
categories: ddd-logbook
lang: en
resume: "This article explores how to integrate user authentication with Firebase in a DDD-based project using Hexagonal Architecture and CQRS. It demonstrates the CommandBus/QueryBus mechanism and emphasizes not focusing on technical concerns before modeling the core domain."
permalink: /ddd-logbook/2025-01-27
---

After three theory-focused articles – essential for laying a solid foundation –  today, I’m offering a concrete example: user authentication. This article slightly  diverges from **Domain-Driven Design (DDD)** to emphasize the implementation  and use of **Hexagonal Architecture** and **CQRS**. While I will briefly mention the **User Entity** the focus will primarily be on practical aspects. If you need a refresher on these concepts, you can refer to the [Day 2](https://huguesgobet.com/ddd-logbook/2024-12-30) article that provides a detailed explanation.

## Firebase: Simplifying User Authentication

***Firebase*** is a platform developed by Google that offers a suite of tools and services to streamline the development, management, and scalability of web and mobile applications. In our case, it acts as an identity provider, simplifying the implementation of an authentication system. This allows for features like Google or Apple login without requiring the storage of credentials or passwords.

### The general process is as follows:

1. Firebase manages authentication and returns a ***JWT*** (JSON Web Token).  
2. This ***JWT***, a standard for securely exchanging information between a client and server, contains user data such as their name and email.  
3. For each request, the ***JWT*** is used to verify the user’s identity without needing to contact Firebase again. Essentially, Firebase handles the login process and issues the JWT for subsequent verification.

![Use case to be authenticated](/assets/images/2025-01-27/use-case.png)

### Future Outlook

Starting a **DDD** project by focusing on authentication is a mistake, as it shifts focus away from the **core business domain**. The goal of **DDD** and **Hexagonal Architecture** is to prioritize the business domain, leaving technical considerations aside during the initial stages.

However, once ***Firebase*** is integrated, validating ***JWT***s becomes essential to extract information such as the user’s name and email.

This is achieved using ***Symfony*** authentication by creating a ***Custom Authenticator*** tailored to Firebase. While Symfony provides several built-in Authenticators, none fully meet our specific requirements. Since the code for this authentication process is not directly relevant to the main theme of this series, I will not go into further detail.

{% highlight php linenos %}
try {
    SuserAuthenticated = $this->commandBus->dispatch (
        new AuthenticateUserCommand(
            Stoken,
            SproviderId,
        ),
    );
} catch (InvalidTokenException $exception) {
    $this->logger->error(
        'Log in user: Invalid token',
        [
            'exception' => $exception,
            'provider' => $providerId,
        ],
    );

    throw new AuthenticationException($exception->getMessage());
} catch (ExpiredTokenException $exception) {
    $this->logger->error(
        'Log in user: Token expired',
        [
            'exception' => $exception,
            'provider' => $providerId,
        ],
    );

    throw new AuthenticationException($exception->getMessage());
} catch (InvalidPayloadException $exception) {
    $this->logger->error(
        'Log in user: Invalid payload',
        [
            'exception' => $exception,
            'provider' => $providerId,
        ],
    );

    throw new AuthenticationException($exception->getMessage());
}

Suser = $this->queryBus->ask(
    new GetUserQuery(
        SuserAuthenticated→email(),
    )
);

Semail = Suser→email();

return new SelfValidatingPassport(
    new UserBadge(
        $email,
        fn () => Sthis->loadUser (Semail),
    ),
);
{% endhighlight %}

## The CommandBus: Orchestrating Commands

System entry relies on a **Primary Adapter** that dispatches a **CQRS** `AuthenticiteUserCommand` via the **CommandBus**. Once the command is executed to authenticate the user, a **Query** `GetUserQuery` is used to retrieve their data. It is worth noting that the Authenticator is part of the **Infrastructure** layer.

The **CommandBus** operates as a centralized system:

- It receives a command as a parameter.  
- It identifies the appropriate **CommandHandler**.  
- It executes the Handle method.  
- It returns the result of the operation.

To adhere to **dependency inversion** principles and the layering of **Hexagonal Architecture**, we create a `CommandBusInterface` in the **Application** layer. The implementation of the **CommandBus**, utilizing ***Messenger*** (a Symfony component for managing a Bus), resides in the **Infrastructure** layer.

**CommandBusInterface**

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace TegCorp\SharedKernelBundle\Application\Command;

interface CommandBusInterface
{
    /**
     * @template T
     *
     * @param CommandInterface<T> $command
     *
     * @return T
     */
    public function dispatch(CommandInterface $command): mixed;
}
{% endhighlight %}

**Implementation of CommandBus Using Messenger**

{% highlight php linenos %}
final class MessengerCommandBus implements CommandBusInterface
{
    use HandleTrait;

    public function __construct(
        MessageBusInterface $commandBus,
    ) {
        $this->messageBus = $commandBus;
    }

    /**
     * @template T
     *
     * @param CommandInterface<T> $command
     *
     * @return T
     */
    #[\Override]
    public function dispatch(CommandInterface $command): mixed
    {
        try {
            /* @var T */
            $result = $this->handle($command);

            return $result;
        } catch (HandlerFailedException $e) {
            if ($exception = current($e->getWrappedExceptions())) {
                throw $exception;
            }

            throw $e;
        }
    }
}
{% endhighlight %}

### Future Outlook

Transaction management should be incorporated.

## Command and CommandHandler

A **Command** is a simple, immutable object that contains the necessary properties to execute a change in the system. To comply with the interface contract defined in the CommandBus, an **Interface** for Commands must first be created.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace TegCorp\SharedKernelBundle\Application\Command;

/**
 * @template T
 */
interface CommandInterface
{
}
{% endhighlight %}

This Interface enables the **CommandHandler** to provide feedback using ***PHPStan Generics***.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Security\Application\Command;

use App\Security\Domain\Service\AuthenticateUserFromProviderInterface;
use App\Security\Domain\ValueObject\UserAuthenticated;
use TegCorp\SharedKernelBundle\Application\Command\CommandInterface;
use TegCorp\SharedKernelBundle\Infrastructure\Webmozart\Assert;

/**
 * @implements CommandInterface<UserAuthenticated>
 */
final readonly class AuthenticateUserCommand implements CommandInterface
{
    public string $providerId;

    public function __construct(
        public string $token,
        string $providerId,
    ) {
        Assert::inArray(
            $providerId,
            [
                AuthenticateUserFromProviderInterface::IDENTITY_PROVIDER_APPLE,
                AuthenticateUserFromProviderInterface::IDENTITY_PROVIDER_GOOGLE,
                AuthenticateUserFromProviderInterface::IDENTITY_PROVIDER_FIREBASE,
            ],
        );

        $this->providerId = $providerId;
    }
}
{% endhighlight %}

This **Command** takes as parameters a ***JWT*** provided by ***Firebase*** and a provider identifier, which indicates the identity provider (e.g., Google, Facebook). The provider ID is validated against the supported values in our system (Google, Apple, and Firebase). Ensuring the consistency of data entering the Domain is crucial to prevent unexpected behavior.

Next comes the **CommandHandler**.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Security\Application\Command;

use App\Security\Domain\Entity\User;
use App\Security\Domain\Exception\InvalidTokenException;
use App\Security\Domain\Repository\UserRepositoryInterface;
use App\Security\Domain\Service\AuthenticateUser;
use App\Security\Domain\ValueObject\UserAuthenticated;
use App\Security\Domain\ValueObject\UserEmail;
use TegCorp\SharedKernelBundle\Application\Command\AsCommandHandler;
use TegCorp\SharedKernelBundle\Domain\Service\DomainEventDispatcherInterface;

#[AsCommandHandler]
final readonly class AuthenticateUserCommandHandler
{
    public function __construct(
        private AuthenticateUser $authenticateUser,
        private DomainEventDispatcherInterface $dispatcher,
        private UserRepositoryInterface $userRepository,
    ) {
    }

    /**
     * @throws InvalidTokenException
     */
    public function __invoke(AuthenticateUserCommand $authenticateUserCommand): UserAuthenticated
    {
        if ($authenticateUserCommand->token === '') {
            throw new InvalidTokenException();
        }

        match ($providerId) {
            AuthenticateUserFromProviderInterface::IDENTITY_PROVIDER_APPLE => $userAuthenticated = $this->authenticateUserFromProvider->authenticateUserWithApple($token),
            AuthenticateUserFromProviderInterface::IDENTITY_PROVIDER_GOOGLE => $userAuthenticated = $this->authenticateUserFromProvider->authenticateUserWithGoogle($token),
            AuthenticateUserFromProviderInterface::IDENTITY_PROVIDER_FIREBASE => $userAuthenticated = $this->authenticateUserFromProvider->authenticateUserWithFirebase($token),
            default => throw new IdentityProviderDoesntExistException('Invalid provider id'),
        };

        return $userAuthenticated;
    }
}
{% endhighlight %}

By leveraging dependency inversion, an interface for the `AuthenticateUserInterface` service – located in the **Application** layer – is injected into the **CommandHandler**, which is also in the **Application** layer. The implementation, however, is located in the **Infrastructure** layer since it depends on an external service.

### Future Outlook

***JWT*** validation should be handled both within the **Command** and at the provider level to ensure robust error management. Consider using the **Strategy** pattern to avoid match-based logic.

In this **CommandHandler**, the user is authenticated by invoking the appropriate method for the identified provider, and the authenticated user is then returned.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Security\Domain\ValueObject;

final readonly class UserAuthenticated
{
    private UserEmail $email;

    public function __construct(
        string $email,
    ) {
        $this->email = UserEmail::fromString($email);
    }

    public function email(): UserEmail
    {
        return $this->email;
    }
}
{% endhighlight %}

## QueryBus and QueryHandler

Once the user is authenticated, the **QueryBus** is utilized to retrieve their business data. **Queries** are read-only operations used to retrieve information without modifying the system. Here too, a `QueryBusInterface` is defined in the **Application** layer, while its implementation using ***Messenger*** is situated in the **Infrastructure** layer.

To adhere to dependency inversion and the layering principles of **Hexagonal Architecture**, an interface `QueryBusInterface` is created in the **Application** layer. The implementation of `QueryBus` using ***Messenger*** resides in the **Infrastructure** layer.

**Interface QueryBusInterface**

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace TegCorp\SharedKernelBundle\Application\Query;

interface QueryBusInterface
{
    /**
     * @template T
     *
     * @param QueryInterface<T> $query
     *
     * @return T
     */
    public function ask(QueryInterface $query): mixed;
}
{% endhighlight %}

**Implementation of MessengerQueryBus**

{% highlight php linenos %}
final class MessengerQueryBus implements QueryBusInterface
{
    use HandleTrait;

    public function __construct(MessageBusInterface $queryBus)
    {
        $this->messageBus = $queryBus;
    }

    /**
     * @template T
     *
     * @param QueryInterface<T> $query
     *
     * @return T
     */
    #[\Override]
    public function ask(QueryInterface $query): mixed
    {
        try {
            /* @var T */
            return $this->handle($query);
        } catch (HandlerFailedException $e) {
            if ($exception = current($e->getWrappedExceptions())) {
                throw $exception;
            }

            throw $e;
        }
    }
}
{% endhighlight %}

### Implementation of the Query and QueryHandler

After defining the QueryBus interface and implementing its functionality, the next step is to implement the **Query** and **QueryHandler**. To meet the QueryBus interface contract, an interface for **Queries** is created.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace TegCorp\SharedKernelBundle\Application\Query;

/**
 * @template T
 */
interface QueryInterface
{
}
{% endhighlight %}

This Interface allows the **QueryHandler** to provide feedback using **PHPStan Generics**.

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Security\Application\Query;

use App\Security\Application\ReadModel\User;
use TegCorp\SharedKernelBundle\Application\Query\QueryInterface;

/**
 * @implements QueryInterface<User>
 */
final readonly class GetUserQuery implements QueryInterface
{
    public function __construct(
        public string $email,
    ) {
    }
}
{% endhighlight %}

The **Query** takes the user’s identifier as input and returns a **User Entity**. Next comes the **QueryHandler**:

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Security\Application\Query;

use App\Security\Application\Adapter\UserAdapterInterface;
use App\Security\Application\ReadModel\User;
use App\Security\Domain\Exception\UserNotFoundException;
use TegCorp\SharedKernelBundle\Application\Query\AsQueryHandler;

#[AsQueryHandler]
final readonly class GetUserQueryHandler
{
    public function __construct(
        private UserAdapterInterface $userAdapter,
    ) {
    }

    /**
     * @throws UserNotFoundException
     */
    public function __invoke(GetUserQuery $getUserQuery): User
    {
        $user = $this->userAdapter->ofId($getUserQuery->email);

        if ($user === null) {
            throw new UserNotFoundException();
        }

        return $user;
    }
}
{% endhighlight %}

Dependency inversion is once again applied by injecting into the **QueryHandler** (from the **Application** layer) an interface for a repository, ***UserRepositoryInterface***, located in the **Domain** layer. The implementation, as always, resides in the **Infrastructure** layer since it depends on the database in use.

Dependency inversion allows flexibility in choosing where to store information and enables switching databases as needed.

### Future Outlook

To fully implement **CQRS** principles, the data should be retrieved from the **Read Model**. This will be addressed later.

This **QueryHandler** returns a **User Entity**:

{% highlight php linenos %}
<?php

declare(strict_types=1);

namespace App\Security\Domain\Entity;

use App\Security\Domain\Event\UserCreated;
use App\Security\Domain\ValueObject\UserEmail;
use App\Security\Domain\ValueObject\UserId;
use Doctrine\ORM\Mapping as ORM;
use TegCorp\SharedKernelBundle\Domain\Entity\EntityDomainEventTrait;
use TegCorp\SharedKernelBundle\Domain\Entity\EntityWithDomainEventInterface;

#[ORM\Entity]
final class User implements EntityWithDomainEventInterface
{
    use EntityDomainEventTrait;

    public function __construct(
        #[ORM\Embedded(columnPrefix: false)]
        private UserId $id,
        #[ORM\Embedded(columnPrefix: false)]
        private UserEmail $email,
    ) {
    }

    public static function create(
        UserId $id,
        UserEmail $email,
    ): self {
        $user = new self(
            $id,
            $email,
        );

        $user::recordEvent(
            new UserCreated(
                $user->id->value(),
                $user->email->value(),
            ),
        );

        return $user;
    }

    public function id(): UserId
    {
        return $this->id;
    }

    public function email(): UserEmail
    {
        return $this->email;
    }
}
{% endhighlight %}

Within this **Entity**, several previously discussed concepts are evident: **Value Objects**, **Events**, **Factories**, etc.

### Future Outlook

Currently, **Doctrine Attributes** can remain within the **Entity**, but this is not recommended. This approach forces compromises between **DDD** and **ORM**, which is discouraged, particularly in the **Domain** layer.

After creating this ***Authenticator***, the ***JWT*** sent with each request is authenticated through this class. This must be done for every request, as ***REST API*** calls are stateless (no state is preserved between requests).

In this article, I detailed the implementation of a feature from start to finish, starting from the **Infrastructure** layer and working up to the **Domain** layer. To conclude, this feature was designed entirely in reverse. In **DDD**, the process should start with modeling **Entities** and then proceed to implementing **Interfaces** defined in the **Domain** layer. This method also facilitates **Test-Driven Development (TDD)** for Entities.

In the next article, I will delve deeper into the structure of the **User Entity** and its methods, focusing on its creation.
