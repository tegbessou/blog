---
layout: article
title: "Pagination: Business or Technical Need?"
date:   2024-12-14 15:00:00
categories: other
lang: en
resume: "This article explores pagination within a software architecture context, emphasizing DDD and CQRS. It illustrates how to separate technical and business concerns to maintain the domain’s integrity."
permalink: /other/2024-12-16
---

During the development of an application for an e-commerce site, I recently came across a thorny problem: how to manage <u>product</u> pagination while respecting the principles of **Domain-Driven Design (DDD)**, **Hexagonal Architecture**, and **Command Query Responsibility Segregation (CQRS)**? For a long time, I wondered whether pagination should be part of the **Domain**, and if not, how to link the **Infrastructure** layer or the **Application** layer to the **Repository** located in the **Domain** layer. How can we comply with the rules of the architectures we’ve chosen to use? In this article, I share my approach and analyses.

## Reminder of the Different Development Methodologies Used

### **Domain-Driven Design**

**DDD** is a software development approach that places the **business domain** at the heart of development. To achieve this, a number of software patterns are used, such as **Entity**, **Aggregate**, **Value Object**, **Repository**, etc. However, using only technical patterns is a misuse of **DDD**. The essential part lies in using the **Ubiquitous Language** and modeling the **Domain**. **DDD** encourages close collaboration between developers and business experts to create a common model. In addition, the concept of **Bounded Context** is essential for delimiting sub-domains and managing system complexity.

### **Hexagonal Architecture**

**Hexagonal Architecture** (also known as Port/Adapter architecture) advocates separating an application into several layers and aims to make the **Domain** independent of external factors (database, framework, etc.). Two layers are mandatory: the **Domain**, with the business code (**Entity**, **Aggregate**, **Value Object**, **Repository**…), and the **Infrastructure**, which manages communications with the outside world through input Adapters (the ***Controllers***) and output Adapters (the **Repository**).

The two layers communicate using dependency inversion. An interface is defined in the **Domain** (for example, a **Repository** with the ***Add*** method), and the **Infrastructure** provides the implementation of the ***Add*** method of the **Repository**, registering the new **Entity** in the database.

![Architecture héxagonale](/assets/images/2024-12-16/hexagonal-architecture.png)

### **Command Query Responsibility Segregation (CQRS)**

**CQRS** is a pattern that stipulates that any action on the system is either a **Query**, to read data from the system, or a **Command**, to perform an action and modify the system. This action returns nothing or the identifier of the created resource. **CQRS** also requires separating the write model from the read model.

![CQRS](/assets/images/2024-12-16/cqrs.png)

## Practical Case: Paginated <u>Product</u> List for an E-commerce Site

For this article, I’ll use a classic example for e-commerce sites: a paginated list of <u>products</u> that can be searched by <u>product name</u>. I’ll then explain why the solution may seem simple but actually raises a question.

![Use case to read a product](/assets/images/2024-12-16/en-use-case-product.png)

## Initial Implementation

To start, I created an **Entity** <u>Product</u> in the **Domain**:

{% highlight php linenos %}
namespace App\Domain\Entity;

final class Product
{
    public function __construct(
        private ProductId $id,
        private ProductName $name,
        private ProductPrice $price,
    ) {}

    public function id(): ProductId
    {
        return $this->id;
    }

    public function name(): ProductName
    {
        return $this->name;
    }

    public function price(): ProductPrice
    {
        return $this->price;
    }
}
{% endhighlight %}

The <u>id</u>, <u>name</u>, and <u>price</u> are all **Value Object**s.

The **Repository** interface is also part of the **Domain**:

{% highlight php linenos %}
namespace App\Domain\Repository;

use App\Domain\Entity\Product;
use App\Domain\ValueObject\ProductName;

interface ProductRepository
{
    /**
     * @return Product[]
     */
    public function getProductsWithName(ProductName $name, int $itemPerPage, int $page): array;
}
{% endhighlight %}

I’m not going to discuss the implementation of this interface in this article, as it’s not relevant to the topic at hand.

The **Query** and the **QueryHandler**, which are responsible for formatting and linking the **Domain** to the outside world, can be found either in the **Infrastructure** layer or in the **Application** layer. To keep this example simple, I’ve placed them in the **Infrastructure** layer, although I personally prefer to put them in the **Application** layer.

Here is the **Query**:

{% highlight php linenos %}

namespace App\Infrastructure\Query;

final readonly class ListProductsWithNameQuery
{
    public function __construct(
        public string $name,
        public int $itemPerPage,
        public int $page,
    ) {
    }
}
{% endhighlight %}

And the **QueryHandler**:

{% highlight php linenos %}
namespace App\Infrastructure\Query;

final readonly class ListProductsWithNameQueryHandler
{
    public function __construct(
        private ProductRepository $productRepository
    ) {}

    /**
     * @return Product[]
     */
    public function __invoke(ListProductsWithNameQuery $query): array
    {
        return $this->productRepository->getProductsWithName(
            ProductName::fromString($query->name),
            $query->itemPerPage,
            $query->page
        );
    }
}
{% endhighlight %}

I’ve placed the ***Controller*** in the **Infrastructure** layer. Its purpose is to transform what comes from the HTTP request into a **Query** and then dispatch it.

{% highlight php linenos %}
namespace App\Infrastructure\Controller;

final class ProductController extends AbstractController
{
    public function __construct(
        private readonly QueryBus $queryBus,
    ) {}

    public function __invoke(Request $request): Response
    {
        $products = $this->queryBus->ask(
            new ListProductsWithNameQuery(
                $request->query->get('name'),
                (int) $request->query->get('itemPerPage'),
                (int) $request->query->get('page'),
            )
        );

        $paginatedProduct = new PaginatedProduct(
            products: $products,
            total: count($products),
            page: (int) $request->query->get('page'),
        );

        return $this->json($paginatedProduct);
    }
}
{% endhighlight %}

## Analysis of the Implementation

This example of implementing a paginated <u>product</u> search respects the layer separation of **Hexagonal Architecture**: the business code is well separated from the **Infrastructure** layer. It also partially respects **CQRS**, since I’m using a **Query** to interrogate the system.

On the other hand, the read model is not separated from the write model, which violates **CQRS**. Initially, I didn’t think this was a big deal until it caused a problem: my **Domain** is polluted by concepts from the **Infrastructure** (or presentation) layer. Since pagination is not part of the **Domain**, it adds no value and may vary depending on the primary Adapter (HTTP, Console…). The name filter is a business Use Case, but it doesn’t add much to the **Domain** and is also part of the presentation layer.

So, how do we manage pagination without polluting the **Domain**?

This is certainly not the only way, but I chose to fully comply with **CQRS** and separate the write model from the read model. I moved the read model into the **Infrastructure** (or **Application**) layer; it can even be extracted from the project managing the <u>Product</u> **Domain**. To do this, whenever I act on the model (create, modify, or delete an **Aggregate** or **Entity**), I reflect this in the read model.

## Benefits of This Approach

- The read model is adapted for presentation needs. You’re not forced to save **Aggregate** or **Entity** data if you don’t need it.
- You don’t have to adhere to the structure of your write model. For example, if you manage a <u>party</u> with <u>invitations</u> and need to filter these <u>invitations</u> to show only those for a specific user, you create a dedicated model for <u>invitations</u> in your read model. You include all the necessary information, whether it comes from the <u>party</u> or the <u>invitation</u>. Data duplication doesn’t matter here.
- No storage constraints for the model. You can use the same server to reduce costs; I simply recommend separating the databases for minimal physical separation. You can also use storage optimized for search, like Elasticsearch or MongoDB. Since this is not a software architecture issue, I won’t go into more detail (as usual!).

## Implementing the Solution

I’ll detail the steps I took to separate the write model from the read model.

### Transforming the Write Model into a Read Model

I use **Domain events** and **Projections** for this.

- What is a **Domain event**? It’s an event dispatched when an action occurs in the **Domain**. For example, when you create a <u>product</u>, you dispatch a <u>product created</u> event.
- What is a **Projection**? It’s the act of projecting the write model onto the read model. It’s part of the **Infrastructure** or **Application** layer. A **Projection** is called in response to an event emitted by the **Domain**.

### Creating a Read Model

Now I’ll present the creation of the read model and then show how the reading part is transformed to use the Read Model.

![Use case to create read model for product](/assets/images/2024-12-16/en-use-case-create-read-model-product.png)

Here is the **Entity** <u>Product</u> with the event recorded when the <u>Product</u> is created:

{% highlight php linenos %}
namespace App\Domain\Entity;

final class Product extends Aggregate
{
    public function __construct(
        private ProductId $id,
        private ProductName $name,
        private ProductPrice $price,
    ) {
    }

    public function create(
        ProductId $id,
        ProductName $name,
        ProductPrice $price,
    ): self {
        $product =  new self(
            $id,
            $name,
            $price,
        );

        $this->record(new ProductCreated(
            $id->value(),
            $name->value(),
            $price->value(),
        ));

        return $product;
    }

    public function id(): ProductId
    {
        return $this->id;
    }

    public function name(): ProductName
    {
        return $this->name;
    }

    public function price(): ProductPrice
    {
        return $this->price;
    }
}
{% endhighlight %}

Here is the **Projection**, which can be called or already be present in an ***EventListener*** or a ***MessageHandler***. For the sake of a short example, I’ve placed the code directly in the Adapter. I recommend extracting it into a class that can be reused if you decide to change the Adapter later.
{% highlight php linenos %}
namespace App\Infrastructure\Projection;

final readonly class CreateProductProjection
{
    public function __construct(
        private ProductAdapterInterface $productAdapter,
    ) {
    }

    public function __invoke(ProductCreated $event): void
    {
        $product = new Product(
            $event->productId,
            $event->name,
            $event->price,
        );

        $this->productAdapter->add($product);
    }
}
{% endhighlight %}

This is the Adapter responsible for storing the data in the read model:

{% highlight php linenos %}
namespace App\Infrastructure\Adapter;

interface ProductAdapterInterface
{
    public function add(Product $product): void;
}
{% endhighlight %}

The implementation depends on the solution you choose, so I won’t go into detail.

Here’s what the <u>Product</u> read model looks like:

{% highlight php linenos %}
namespace App\Infrastructure\ReadModel;

final readonly class Product
{
    public function __construct(
        public string $id,
        public string $name,
        public float $price,
    ) {}
}
{% endhighlight %}

Now that the read model is ready, it needs to be queried.

### Querying the Read Model

To do this, I took the Use Case diagram and modified it slightly:

![Use case to read a product](/assets/images/2024-12-16/en-use-case-list-product.png)

The read Adapter is now no longer in the **Domain**, but in the **Infrastructure** layer.

{% highlight php linenos %}
namespace App\Infrastructure\Adapter;

interface ProductAdapterInterface
{
    public function add(Product $product): void;

    /**
     * @return Product[]
     */
    public function getProductsWithName(string $name, int $itemPerPage, int $page): array;
}
{% endhighlight %}

The Adapter now handles reading from the read model. It contains the method that used to be in the **Repository** in the previous example. I changed the <u>name</u> parameter to a simple string. Since we’re outside the **Domain**, it doesn’t need to be a **Value Object**.

The **Query** does not change:

{% highlight php linenos %}
namespace App\Infrastructure\Query;

final readonly class ListProductsWithNameQuery
{
    public function __construct(
        public string $name,
        public int $itemPerPage,
        public int $page,
    ) {
    }
}
{% endhighlight %}

Now, the **QueryHandler** will no longer call the **Domain** **Repository**, but the Adapter that reads from the Read Model:

{% highlight php linenos %}
namespace App\Infrastructure\Query;

final readonly class ListProductsWithNameQueryHandler
{
    public function __construct(
        private ProductAdapterInterface $productAdapter
    ) {}

    /**
     * @return Product[]
     */
    public function __invoke(ListProductsWithNameQuery $query): array
    {
        return $this->productAdapter->getProductsWithName(
            $query->name,
            $query->itemPerPage,
            $query->page
        );
    }
}
{% endhighlight %}

The ***Controller*** also remains unchanged:

{% highlight php linenos %}
namespace App\Infrastructure\Controller;

final class ProductController extends AbstractController
{
    public function __construct(
        private readonly QueryBus $queryBus,
    ) {}

    public function __invoke(Request $request): Response
    {
        $products = $this->queryBus->ask(
            new ListProductsWithNameQuery(
                $request->query->get('name'),
                (int) $request->query->get('itemPerPage'),
                (int) $request->query->get('page'),
            )
        );

        $paginatedProduct = new PaginatedProduct(
            products: $products,
            total: count($products),
            page: (int) $request->query->get('page'),
        );

        return $this->json($paginatedProduct);
    }
}
{% endhighlight %}

I’ve tried to answer the question: is pagination a business need or a technical need? From what we’ve seen, pagination is part of the business use case but does not fall under the **Domain**’s responsibility. It’s therefore necessary to integrate it by dissociating the read model from the **Domain** model. This approach maintains the integrity of the **Domain** by keeping it away from issues outside its scope.

If you’re interested, you can find all the code for this article on [*GitHub*](https://github.com/tegbessou/ddd-pagination). You’ll find the code without the Read Model as well as the latest version.

See you in a fortnight for the next article in the **DDD** Logbook.