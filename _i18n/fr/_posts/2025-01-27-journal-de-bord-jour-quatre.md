---
layout: article
title: "Journal de bord de l’apprentissage du Domain-Driven Design : Jour 4"
date:   2025-01-27 08:00:00
categories: ddd-logbook
lang: fr
resume: "Cet article montre comment intégrer l’authentification utilisateur avec Firebase dans un projet DDD reposant sur l’Architecture Hexagonale et le CQRS. Il détaille le fonctionnement du CommandBus/QueryBus et souligne l’importance de ne pas commencer par les préoccupations techniques avant de modéliser le domaine métier."
permalink: /ddd-logbook/2025-01-27
---

Après trois articles axés sur la théorie - indispensables pour poser des bases solides - je vous propose aujourd’hui un cas concret : l’authentification d’un utilisateur. Cet article s’éloigne légèrement du **Domain Driven Design (DDD)** pour se concentrer sur la mise en place et l’utilisation de l’**Architecture Hexagonale** et le **CQRS**. Je mentionnerai tout de même l’**Entity User**, mais l’essentiel portera sur l’aspect pratique. Si vous avez besoin d’un rappel sur ces concepts, vous pouvez vous référer à l’article du jour 2 qui les détaille (mettre un lien vers le jour 2).

## Firebase : pour simplifier l’authentification utilisateur

***Firebase*** est une plateforme développée par Google qui fournit un ensemble d'outils et de services destinés à faciliter le développement, la gestion et la scalabilité des applications web et mobiles. Dans notre cas, elle sert de provider d’identité, simplifiant l’implémentation d’un système d’authentification. Cela permet notamment d’intégrer des options comme l’identification via Google ou Apple, sans se soucier du stockage des identifiants ou des mots de passe.

### Voici le processus général :

1. ***Firebase*** se charge de l’authentification et renvoie un ***JWT*** (JSON Web Token).  
2. Ce ***JWT***, un standard pour échanger des informations sécurisées entre un client et un serveur, contient des données comme le nom et l’email de l’utilisateur.  
3. À chaque requête, on utilise ce ***JWT*** pour vérifier l’identité de l’utilisateur sans solliciter Firebase à nouveau. Pour cela, on délègue la partie login à Firebase, qui se charge de l’authentification de l’utilisateur et retourne un ***JWT***.

![Use case to be authenticated](/assets/images/2025-01-27/use-case.png)

### Vision du Futur

Commencer un projet **DDD** en se focalisant sur l’authentification est une erreur. Cela détourne l’attention du cœur du domaine métier. L’objectif du **DDD** et de l’**Architecture Hexagonale** est de prioriser le métier en mettant de côté les aspects techniques au départ.

Cependant, une fois ***Firebase*** intégré, il est impératif de valider les ***JWT*** pour extraire des informations comme le nom et l’email.  
Pour ce faire, on utilise l’authentification de ***Symfony***, en créant un ***Authenticator*** Custom pour s’adapter à ***Firebase***. Notons qu’il existe plusieurs ***Authenticators*** déjà fournis dans ***Symfony***, mais qu’aucun ne correspond à notre besoin. Le code de cette authentication n’est pas tellement intéressant au regard de la thématique de cette série d’articles, je ne m’y attarderai donc pas.

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

## Le CommandBus : pour orchestrer les commandes

L’entrée dans le système repose sur un **Primary Adapter** qui dispatche une commande CQRS AuthenticiteUserCommand via le **CommandBus**. Une fois la commande exécutée pour authentifier l’utilisateur, une **Query** GetUserQuery permet de récupérer ses données. Notons que l’***Authenticator*** fait partie de la couche **Infrastructure**.

Le **CommandBus** est un système centralisé :  
- Il reçoit une commande en paramètre  
- Il identifie le **CommandHandler** approprié  
- Il exécute sa méthode ***Handle*** 
- Il retourne l’identité du résultat  

Pour respecter l’**inversion de dépendance** et les couches de l’**Architecture Hexagonale**, on crée une interface ***CommandBusInterface*** dans la couche **Application**. L’implémentation du CommandBus en utilisant ***Messenger*** (un composant de Symfony pour avoir une gestion de Bus) se trouve dans la couche **Infrastructure**.

**Interface CommandBusInterface:**  

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

**Implémentation du CommandBus utilisant Messenger:** 

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

### Vision du Futur

Il faudrait gérer les transactions.

## Command et CommandHandler

Une **Command** est un objet simple, immuable, possédant les propriétés nécessaires à la réalisation de la modification sur le système. Pour respecter le contrat d’interface défini dans le CommandBus, il faut d’abord créer une interface pour les **Command**.

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

Cette interface permet d’avoir un retour du **CommandHandler** en utilisant les Generics de ***PHPStan***.

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

Cette **Command** prend en paramètre un JWT envoyé par Firebase et un identifiant de provider qui permet de savoir qui est le provider d’identité (Google, Facebook…). L’id de provider est validée en fonction des valeurs supportées par notre système (Google, Apple et Firebase). Il est important de valider la cohérence des données qui rentrent dans le Domain, cela évite les comportements inattendus.

Ensuite vient le **CommandHandler**.
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

J’ai utilisé une fois encore l’inversion de dépendance en injectant une interface d’un service `AuthenticateUserInterface` - qui se trouve dans la couche **Application** - dans le **CommandHandler** - qui se trouve aussi dans la couche **Application**. L’implémentation se trouve, quant à elle, dans la couche **Infrastructure** car elle dépend du service externe.

### Vision du Futur

- Il faudrait valider le JWT dans la Command comme dans le provider pour ne pas avoir cette gestion d’erreur.  
- Il faudrait aussi réfléchir à l’utilisation du pattern _Stratégie_ pour éviter le match.

Dans ce **CommandeHandler**, on authentifie l’utilisateur en utilisant la méthode qui correspond au provider identifié, puis on retourne l’utilisateur authentifié.

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

## QueryBus et QueryHandler

Une fois l’utilisateur authentifié, le **QueryBus** entre en jeu pour récupérer ses données métier. Les **Query** servent à lire des informations sans modifier le système. Ici aussi, une interface ***QueryBusInterface*** est définie en **Application**, et son implémentation avec ***Messenger*** se situe en **Infrastructure**.

Pour utiliser l’inversion de dépendance et respecter les couches de l’**Architecture Hexagonale**, j’ai créé une interface ***QueryBusInterface*** dans la couche Application. L’implémentation du QueryBus en utilisant ***Messenger*** se trouve dans la couche **Infrastructure**.

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

**Implémentation du MessengerQueryBus**

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

### Implémentation de la Query et du QueryHandler

Après l’interface et l’implémentation du QueryBus, il faut implémenter la **Query** et le **QueryHandler**. Afin de respecter le contrat d’interface défini dans le QueryBu`, on crée une interface pour les **Query**.

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

Cette interface permet d’avoir un retour du **QueryHandler** en utilisant les Generics de ***PHPStan***.

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

Cette **Query** utilise l’identifiant du User et retourne une **Entity User**.  

Ensuite vient le **QueryHandler** :

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

Une fois encore, j’ai utilisé l’inversion de dépendance en injectant dans le **QueryHandler** de la couche **Application** une interface d’un Repository ***UserRepositoryInterface*** de la couche **Domain**. L’implémentation se trouve toujours dans **Infrastructure** car elle dépend de la base de données que nous utilisons. L’inversion de dépendance laisse la possibilité de sauvegarder les informations où l’on veut et de changer de base de données.

### Vision du Futur

Pour suivre le **CQRS**, il faudrait lire dans le **Read Model**. Je le mettrai en place plus tard.

Ce **QueryHandler** retourne une **Entity User**.

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

Dans cette Entity, on voit plusieurs choses dont on a parlé dans les articles précédents : les **Value Object**, les **Event**, les **Factory**...

### Vision du Futur

Pour l’instant, il est possible de laisser les ***Attributs Doctrine*** dans l’**Entity**, mais je ne le recommande pas. Cela oblige en effet à faire des compromis entre le **DDD** et l’***ORM*** et cela est déconseillé, surtout dans le **Domain**.

Après avoir fait cet ***Authenticator***, à chaque requête, on passe dans cette classe pour authentifier le JWT envoyé. Il faut le faire à chaque fois car les requêtes à une ***API REST*** sont _stateless_ (c’est à dire qu’on ne conserve pas d’état entre les requêtes).

Dans cet article j’ai détaillé l’implémentation d’une fonctionnalité du début à la fin, en partant de l’implémentation de la couche **Infrastructure** jusqu’au **Domain**. Je dirai pour conclure que cette fonctionnalité a été totalement pensée à l’envers. Dans le **DDD**, il faut commencer par la modélisation des **Entity** puis descendre jusqu’à l’implémentation des **Interfaces** définies dans le **Domain**. Cela peut également permettre de faire du **TDD** (Test-Driven Developpement) sur nos **Entity**.

J’évoquerai plus en détail la structure de cette **Entity** et de ces méthodes dans le prochain article qui parlera de la création d’un User.