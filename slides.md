---
titleTemplate: '%s'
theme: seriph
background: https://source.unsplash.com/1920x1080/?developer
highlighter: shiki
lineNumbers: true
info: false
css: unocss
hideInToc: true 
---
# CREER DES FIXTURES AVEC FOUNDRY

Pour Sylius, Symfony et n'importe quel projet PHP en fait

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---
layout: center
hideInToc: true
---

# De quoi on parle ?

<Toc />

---
layout: center
class: text-center
src: ./pages/presentations.md
---

---
layout: section
title: Foundry
level: 1
---
# zenstruck/foundry

---
layout: center
hideInToc: true
---

Foundry permet de créer des fixtures :

<v-clicks>

- expressive
- auto-complétable
- chargée à la demande 
- au sein de Symfony et/ou Doctrine

</v-clicks>

---
layout: center
hideInToc: true
---

<img src="/composer_require.png" />

---
layout: center
hideInToc: true
---

<img src="/make_factory.png" />

<!-- Une factory est une classe permettant d'instancier un ou des objets -->

---
layout: default
hideInToc: true
---

```php
final class UserFactory extends ModelFactory
{
    /** @todo inject services if required */
    public function __construct()
    {
        parent::__construct();
    }

    protected function getDefaults(): array
    {
        return [
            'email' => self::faker()->text(180),
            'password' => self::faker()->text(),
            'roles' => [],
        ];
    }

    protected function initialize(): self
    {
        return $this;
    }

    protected static function getClass(): string
    {
        return User::class;
    }
}
```

---
layout: default
hideInToc: true
---

```php {monaco-diff}
final class UserFactory extends ModelFactory
{
    /** @todo inject services if required */
    public function __construct()
    {
        parent::__construct();
    }

    protected function getDefaults(): array
    {
        return [
            'email' => self::faker()->text(180),
            'password' => self::faker()->text(),
            'roles' => [],
        ];
    }

    protected function initialize(): self
    {
        return $this;
    }

    protected static function getClass(): string
    {
        return User::class;
    }
}
~~~
final class UserFactory extends ModelFactory
{
    /** @todo inject services if required */
    public function __construct()
    {
        parent::__construct();
    }

    protected function getDefaults(): array
    {
        return [
            'email' => self::faker()->email(),
            'password' => self::faker()->password(),
            'roles' => [],
        ];
    }

    protected function initialize(): self
    {
        return $this;
    }

    protected static function getClass(): string
    {
        return User::class;
    }
}
```

---
layout: center
title: Exemple de la collection de livre n°1
---

# Exemple : collection de livre

## Ajouter un avis

---
layout: center
hideInToc: true
---

```php
class ReviewTest extends ApiTestCase
{
    public function testUserLeaveAReview(): void
    {
        static::createClient()->request('POST', sprintf("/books/%s/reviews", $book->getId()), [
            'json' => [
                'content' => 'Lorem ipsum dolor sit amet',
            ],
        ]);
        $this->assertResponseIsSuccessful();
        $this->assertJsonContains(['@id' => '/reviews/1']);
    }
}
```

---
layout: center
hideInToc: true
---

# Quelques traits

<v-clicks>

- ResetDatabase : (re)création de la base via les schémas en place
- Factories : Initialise Foundry au sein du Kernel

</v-clicks>

---
layout: center
hideInToc: true
---

```php{3,7,8,9,15-16}
class ReviewTest extends AbstractTest
{
    use ResetDatabase, Factories;

    public function testUserLeaveAReview(): void
    {
        $book = BookFactory::createOne();
        $user = UserFactory::createOne();
        $token = $this->getToken(['email' => $user->getEmail(), 'password' => $user->getPassword()]);
        static::createClientWithCredentials($token)->request('POST', sprintf("/books/%s/reviews", $book->getId()), [
            'json' => [
                'content' => 'Lorem ipsum dolor sit amet',
            ],
        ]);
        $this->assertResponseIsSuccessful();
        $this->assertJsonContains(['@id' => '/reviews/1']);
    }
}
```

---
layout: center
title: Exemple de la collection de livre n°2
---

# Exemple : collection de livre

## Récupérer les avis

---
layout: center
hideInToc: true
---

```php
class ReviewTest extends AbstractTest
{
    use ResetDatabase, Factories;

    public function testAuthorGetReviews(): void
    {
        $book = BookFactory::createOne();
        static::createClient()->request('GET', '/book/1/reviews');
        $this->assertResponseIsSuccessful();
        $this->assertJsonContains(['@id' => '/books/1/reviews']);
    }
}
```

---
layout: center
hideInToc: true
---

# Il faut ajouter des reviws à mon livre, non ? 

---
layout: center
hideInToc: true
---

```php
class ReviewTest extends AbstractTest
{
    use ResetDatabase, Factories;

    public function testAuthorGetReviews(): void
    {
        $book = BookFactory::createOne();
        while(!$jenAiMarre) {
            $user = UserFactory::createOne();
            $token = $this->getToken(['email' => $user->getEmail(), 'password' => $user->getPassword()]);
            static::createClientWithCredentials($token)->request('POST', sprintf("/books/%s/reviews", $book->getId()), [
                'json' => [
                    'content' => 'Lorem ipsum dolor sit amet',
                ],
            ]);
        }
        static::createClient()->request('GET', '/book/1/reviews');
        $this->assertResponseIsSuccessful();
        $this->assertJsonContains(['@id' => '/reviews/1']);
    }
}
```

---
layout: center
hideInToc: true
---

```php{8-16}
class ReviewTest extends AbstractTest
{
    use ResetDatabase, Factories;

    public function testAuthorGetReviews(): void
    {
        $book = BookFactory::createOne();
        while(!$jenAiMarre) {
            $user = UserFactory::createOne();
            $token = $this->getToken(['email' => $user->getEmail(), 'password' => $user->getPassword()]);
            static::createClientWithCredentials($token)->request('POST', sprintf("/books/%s/reviews", $book->getId()), [
                'json' => [
                    'content' => 'Lorem ipsum dolor sit amet',
                ],
            ]);
        }
        static::createClient()->request('GET', sprintf('/book/%s/reviews', $book->getId()));
        $this->assertResponseIsSuccessful();
        $this->assertJsonContains(['@id' => '/reviews/1']);
    }
}
```

---
layout: center
hideInToc: true
---

## Plusieurs soucis

<v-clicks>

- Performance
- Je teste des choses qui ne sont pas liées à mon test

</v-clicks>

---
layout: center
hideInToc: true
---

```php
class ReviewTest extends AbstractTest
{
    use ResetDatabase, Factories;

    public function testAuthorGetReviews(): void
    {
        $user = UserFactory::new()->asAuthor()->createOne();
        ReviewFactory::createMany(50, function($user) {
            return [
                'book' => BookFactory::findOrCreate(['isbn' => 143987391287, 'author' => $user->getAuthor()]),
                'reviewer' => UserFactory::randomOrCreate(),
            ];
        });
        $token = $this->getToken(['email' => $user->getEmail(), 'password' => $user->getPassword()]);
        static::createClientWithCredentials($token)->request('GET', '/books/1/reviews');
        $this->assertResponseIsSuccessful();
        $this->assertJsonContains(['@id' => '/books/1/reviews']);
    }
}
```

<!-- Switch rapidement our highlight les défauts -->

---
layout: center
hideInToc: true
---

```php{7-14}
class ReviewTest extends AbstractTest
{
    use ResetDatabase, Factories;

    public function testAuthorGetReviews(): void
    {
        $user = UserFactory::new()->asAuthor()->createOne();
        ReviewFactory::createMany(50, function($user) {
            return [
                'book' => BookFactory::findOrCreate(['isbn' => 143987391287, 'author' => $user->getAuthor()]),
                'reviewer' => UserFactory::randomOrCreate(),
            ];
        });
        $token = $this->getToken(['email' => $user->getEmail(), 'password' => $user->getPassword()]);
        static::createClientWithCredentials($token)->request('GET', '/books/1/reviews');
        $this->assertResponseIsSuccessful();
        $this->assertJsonContains(['@id' => '/books/1/reviews']);
    }
}
```

---
layout: center
title: Reusable Model Factory "States"
level: 2
---

# T'as ajouté un AsAuthor ?

<v-clicks>

## Les "states" (pas les USA)

</v-clicks>

---
layout: center
hideInToc: true
---

```php
final class PostFactory extends ModelFactory
{
    // ...

    public function published(): self
    {
        // call setPublishedAt() and pass a random DateTime
        return $this->addState(['published_at' => self::faker()->dateTime()]);
    }

    public function unpublished(): self
    {
        return $this->addState(['published_at' => null]);
    }

    public function withViewCount(int $count = null): self
    {
        return $this->addState(function () use ($count) {
            return ['view_count' => $count ?? self::faker()->numberBetween(0, 10000)];
        });
    }
}
```
---
layout: center
hideInToc: true
---

```php
final class UserFactory extends ModelFactory
{
    /**
     * @todo inject services if required
     */
    public function __construct()
    {
        parent::__construct();
    }

    protected function getDefaults(): array
    {
        return [
            'email' => self::faker()->email(),
            'password' => 'password',
            'roles' => [],
        ];
    }

    public function asAuthor(): self
    {
        return $this->addState(['author' => AuthorFactory::new()->create()]);
    }
```

---
layout: center
title: Beaucoup de fonctionnalités
level: 2
---

# On peut faire autrement ?

new(), State, findOrCreate, randomOrCreate et les autres fonctions ?


<!--
Je reprend mon exemple précédent
-->

---
layout: center
hideInToc: true
---

```php
class ReviewTest extends AbstractTest
{
    public function testAuthorGetReviews(): void
    {
        $user = UserFactory::new()->asAuthor()->createOne();
        ReviewFactory::createMany(50, function($user) {
            return [
                'book' => BookFactory::findOrCreate(['isbn' => 143987391287, 'author' => $user->getAuthor()]),
                'reviewer' => UserFactory::randomOrCreate(),
            ];
        });
        $token = $this->getToken(['email' => $user->getEmail(), 'password' => $user->getPassword()]);
        static::createClientWithCredentials($token)->request('GET', '/books/1/reviews');
        $this->assertResponseIsSuccessful();
        $this->assertJsonContains(['@id' => '/books/1/reviews']);
    }
}
```

<!-- Switch rapidement our highlight les défauts -->

---
layout: center
hideInToc: true
---

```php{13,15}
class ReviewTest extends AbstractTest
{
    public function testAuthorGetReviews(): void
    {
        $user = UserFactory::new()->asAuthor()->createOne();
        ReviewFactory::createMany(50, function($user) {
            return [
                'book' => BookFactory::findOrCreate(['isbn' => 143987391287, 'author' => $user->getAuthor()]),
                'reviewer' => UserFactory::randomOrCreate(),
            ];
        });
        $token = $this->getToken(['email' => $user->getEmail(), 'password' => $user->getPassword()]);
        static::createClientWithCredentials($token)->request('GET', '/books/1/reviews');
        $this->assertResponseIsSuccessful();
        $this->assertJsonContains(['@id' => '/books/1/reviews']);
    }
}
```

<!--
Le "find" arrive après
-->

---
layout: center
hideInToc: true
---

```php
// src/Factory/AuthorFactory

protected function getDefaults(): array
{
    return [
        'account' => UserFactory::new(),
    ];
}
```
```php{1-7,10-16,18-21,23-33}
public function testAuthorGetReviewFind(): void
    {
        BookFactory::new(['isbn' => 193847625, 'authors' => [
            AuthorFactory::new(['lastname' => 'hooper', 'firstname' => 'grace'])],
        ])->create();
        ReviewFactory::createMany(50, [
            'book' => BookFactory::find(['isbn' => 193847625]),
            'reviewer' => UserFactory::randomOrCreate(),
        ]);
        $token = $this->getToken([
            'email' => AuthorFactory::find(['lastname' => 'hooper'])->getAccount()->getEmail(), 
            'password' => 'password'
        ]);
        static::createClientWithCredentials($token)->request('GET',
            sprintf('/books/%s/reviews', BookFactory::find(['isbn' => 193847625])->getId())
        );
        $this->assertResponseIsSuccessful();
        $this->assertJsonContains(['@id' =>
            sprintf('/books/%s/reviews', BookFactory::find(['isbn' => 193847625])->getId())]
        );
    }
```

<!--
Ici on élimine les variables et utilise Find, ensuite je parle inconvénient
-->

---
layout: center
hideInToc: true
---

# C'est lourd et on se répète

---
layout: center
hideInToc: true
---

```php
class ReviewTest extends AbstractTest
{
    public function testAuthorGetReviewFirst(): void
    {
        BookFactory::new(['authors' => [AuthorFactory::new()]])->create();
        ReviewFactory::createMany(50, [
            'book' => BookFactory::first(),
            'reviewer' => UserFactory::randomOrCreate(),
        ]);
        $token = $this->getToken(['email' => UserFactory::first()->getEmail(), 'password' => 'password']);
        static::createClientWithCredentials($token)->request('GET', 
            sprintf('/books/%s/reviews', BookFactory::first()->getId())
        );
        $this->assertResponseIsSuccessful();
        $this->assertJsonContains(['@id' => sprintf('/books/%s/reviews', BookFactory::first()->getId())]);
    }
}
```

<!-- Switch rapidement our highlight les défauts -->

---
layout: center
hideInToc: true
---

```php{5-13,15}
class ReviewTest extends AbstractTest
{
    public function testAuthorGetReviewFirst(): void
    {
        BookFactory::new(['authors' => [AuthorFactory::new()]])->create();
        ReviewFactory::createMany(50, [
            'book' => BookFactory::first(),
            'reviewer' => UserFactory::randomOrCreate(),
        ]);
        $token = $this->getToken(['email' => UserFactory::first()->getEmail(), 'password' => 'password']);
        static::createClientWithCredentials($token)->request('GET', 
            sprintf('/books/%s/reviews', BookFactory::first()->getId())
        );
        $this->assertResponseIsSuccessful();
        $this->assertJsonContains(['@id' => sprintf('/books/%s/reviews', BookFactory::first()->getId())]);
    }
}
```

---
layout: center
level: 2
title: Les Storys
---

## Les storys

---
layout: center
hideOnToc: true
---

<img src="/make_story.png">

<!--
Je parle du random après
-->

---
layout: center
hideInToc: true
---

```php
final class DefaultReviewsStory extends Story
{
    public function build(): void
    {
        AuthorFactory::new()->withBook()->createMany(10);
        ReviewFactory::createMany(50, [
            'book' => BookFactory::random(),
            'reviewer' => UserFactory::randomOrCreate(),
        ]);
    }
}
```

---
layout: center
hideOnToc: true
---

# C'est pas top

<v-clicks>

- ça fait pas exactement ce que ça semble dire faire
- on fait beaucoup de code pour pas grand chose
- ok faut des preuves

</v-clicks>

---
layout: center
title: On peut visualiser sa donnée facilement !
level: 3
---

# On peut visualiser sa donnée facilement !

<img src="/pepe_pray.png">

---
layout: center
hideInToc: true
---

```php
class ReviewTest extends AbstractTest
{
    public function testAuthorGetReviewsStory(): void
    {
        DefaultReviewsStory::load();
        die();
        $token = $this->getToken(['email' => UserFactory::random()->getEmail(), 'password' => 'password']);
        static::createClientWithCredentials($token)->request('GET', sprintf('/books/%s/reviews', BookFactory::random()->getId()));
        $this->assertResponseIsSuccessful();
        $this->assertJsonContains(['@id' => sprintf('/books/%s/reviews', BookFactory::random()->getId())]);
    }
}
```

---
layout: full
hideInToc: true
---

<img src="/dump_db_1.png" class="w-full h-full" />

---
layout: center
hideInToc: true
---

```php
final class ReviewFactory extends ModelFactory
{
    protected function getDefaults(): array
    {
        return [
            'reviewer' => UserFactory::new(),
            'content' => self::faker()->text(),
            'book' => BookFactory::randomOrCreate()
        ];
    }
```

---
layout: center
hideInToc: true
---

```php
final class DefaultReviewsStory extends Story
{
    public function build(): void
    {
        ReviewFactory::createMany(50);
    }
}
```
```php
class ReviewTest extends AbstractTest
{
    public function setUp(): void
    {
        DefaultReviewsStory::load();
    }

    public function testAuthorGetReviewsStory(): void
    {
        $token = $this->getToken(['email' => UserFactory::random()->getEmail(), 'password' => 'password']);
        static::createClientWithCredentials($token)->request('GET', 
            sprintf('/books/%s/reviews', BookFactory::random()->getId())
        );
        $this->assertResponseIsSuccessful();
        $this->assertJsonContains(['@id' => sprintf('/books/%s/reviews', BookFactory::random()->getId())]);
    }
}
```
<!-- Oui ça marche... mais parceque j'ai qu'un livre :/ -->

---
layout: center
hideInToc: true
---

```php
final class ReviewFactory extends ModelFactory
{
    protected function getDefaults(): array
    {
        return [
            'reviewer' => UserFactory::new(),
            'content' => self::faker()->text(),
            'book' => BookFactory::createOne()
        ];
    }
```

```diff
--- Expected
+++ Actual
@@ @@
 array (
   '@context' => '/contexts/Review',
-  '@id' => '/books/49/reviews',
+  '@id' => '/books/42/reviews',
   '@type' => 'hydra:Collection',
   'hydra:totalItems' => 1,
   'hydra:member' =>
```

---
layout: center
hideInToc: true
---

```php
final class DefaultBookStory extends Story
{
    public const THE_FAST_TRACK = 'thefasttrack';

    public function build(): void
    {
        $this->addState(
            self::THE_FAST_TRACK,
            BookFactory::createOne(['name' => 'The Fast Track'])
        );
    }
}

final class DefaultReviewsStory extends Story
{
    public function build(): void
    {
        ReviewFactory::createMany(50);
        ReviewFactory::createMany(10, [
            'book' => DefaultBookStory::get(DefaultBookStory::THE_FAST_TRACK),
        ]);
    }
}
```

---
layout: center
hideInToc: true
---

```php
class ReviewTest extends AbstractTest
{
    use ResetDatabase;
    use Factories;

    public function setUp(): void
    {
        DefaultReviewsStory::load();
    }

    public function testAuthorGetReviewsStory(): void
    {
        $token = $this->getToken(['email' => UserFactory::random()->getEmail(), 'password' => 'password']);
        static::createClientWithCredentials($token)->request('GET',
            sprintf(
                '/books/%s/reviews',
                DefaultBookStory::get(DefaultBookStory::THE_FAST_TRACK)->getId()
            )
        );
        $this->assertResponseIsSuccessful();
        $this->assertJsonContains([
            '@id' => sprintf(
                '/books/%s/reviews',
                DefaultBookStory::get(DefaultBookStory::THE_FAST_TRACK)->getId()
            )
        ]);
    }
}
```

---
layout: center
title: On peut s'en servir de fixtures (dev/prod)
---

# En tant que fixtures

<img src="/composer_require_fixtures.png">

---
layout: center
hideInToc: true
---

```php
<?php

namespace App\DataFixtures;

use App\Factory\AuthorFactory;
use App\Factory\UserFactory;
use Doctrine\Bundle\FixturesBundle\Fixture;
use Doctrine\Persistence\ObjectManager;

class AppFixtures extends Fixture
{
    public function load(ObjectManager $manager): void
    {
        UserFactory::createMany(10);
        AuthorFactory::createMany(2);
    }
}
```

---
layout: center
title: Et dans un vrai projet ?
---

# Et dans un vrai projet ?

## Sylius

---
layout: default
hideInToc: true
---

```yaml 
sylius_fixtures:
    suites:
        default: # this key is always called whenever the sylius:fixtures:load command is called, below we are extending it with new fixtures
            fixtures:
                currency:
                    options:
                        currencies: ['PLN','HUF']
                channel:
                    options:
                        custom:
                            pl_web_store: # creating new channel
                                name: "PL Web Store"
                                code: "PL_WEB"
                                locales:
                                    - "%locale%"
                                currencies:
                                    - "PLN"
                                enabled: true
                                hostname: "localhost"
                            hun_web_store:
                                name: "Hun Web Store"
                                code: "HUN_WEB"
                                locales:
                                    - "%locale%"
                                currencies:
                                    - "HUF"
                                enabled: true
                                hostname: "localhost"
```

---
layout: default
hideInToc: true
---


```yaml

                shipping_method:
                    options:
                        custom:
                            ups_eu: # creating a new shipping_method and adding channel to it
                                code: "ups_eu"
                                name: "UPS_eu"
                                enabled: true
                                channels:
                                    - "PL_WEB"
                                    - "HUN_WEB"
                payment_method:
                    options:
                        custom:
                            cash_on_delivery_pl:
                                code: "cash_on_delivery_eu"
                                name: "Cash on delivery_eu"
                                channels:
                                    - "PL_WEB"
                            bank_transfer:
                                code: "bank_transfer_eu"
                                name: "Bank transfer_eu"
                                channels:
                                    - "PL_WEB"
                                    - "HUN_WEB"
                                enabled: true
```

---
layout: center
title: Faker à la rescousse !
level: 2
---

# Faker à la rescousse !

---
layout: center
hideInToc: true
---

```php
// src/Foundry/Story/DefaultCurrenciesStory.php

declare(strict_types=1);

namespace App\Foundry\Story;

use Akawakaweb\ShopFixturesPlugin\Foundry\Factory\CurrencyFactory;
use Akawakaweb\ShopFixturesPlugin\Foundry\Story\DefaultCurrenciesStoryInterface;
use Zenstruck\Foundry\Factory;
use Zenstruck\Foundry\Story;

final class DefaultCurrenciesStory extends Story implements DefaultCurrenciesStoryInterface
{
    public function build(): void
    {
        CurrencyFactory::new()->withCode('EUR')->create();
    }
}
```

```yaml
# config/services.yaml

when@dev: &fixtures_dev
    services:
        sylius.fixtures_plugin.story.default_currencies: '@App\Foundry\Story\DefaultCurrenciesStory'
```

<!-- Customizing stories -->

---
layout: default
hideInToc: true
---

## Je veux ajouter un second numéro de téléphone

### Cas pratique : Modifier la monnaie par défaut

```php
// src/Entity/Customer/Customer.php

#[ORM\Entity]
#[ORM\Table(name: 'sylius_customer')]
class Customer extends BaseCustomer
{
    /**
     * @ORM\Column
     */
    #[ORM\Column]
    private ?string $secondPhoneNumber = null;

    public function getSecondPhoneNumber(): ?string
    {
        return $this->secondPhoneNumber;
    }

    public function setSecondPhoneNumber(?string $secondPhoneNumber): void
    {
        $this->secondPhoneNumber = $secondPhoneNumber;
    }
}
```

---
layout: default
hideInToc: true
---

```php
// src/Foundry/DefaultValues/CustomerDefaultValues.php

final class CustomerDefaultValues implements DefaultValuesInterface
{
    public function __construct(
        private DefaultValuesInterface $decorated,
    ) {
    }

    public function __invoke(Generator $faker): array
    {
        return array_merge(($this->decorated)($faker), [
            'secondPhoneNumber' => $faker->phoneNumber(),
        ]);
    }
}
```

```yaml
when@dev: &fixtures_dev
    services:
        App\Foundry\DefaultValues\CustomerDefaultValues:
            decorates: 'sylius.fixtures_plugin.default_values.customer'
            arguments:
                $decorated: '@.inner'
```

---
layout: end
hideInToc: true
---

Disponible prochainement.. rdv le 22 pour la conférence de Loïc Frémont

<!-- Remercier Akawaka, le staff notamment Thibault et Grégoire Hebert -->