---
titleTemplate: '%s'
theme: seriph
background: https://source.unsplash.com/1920x1080/?developer
highlighter: shiki
lineNumbers: true
info: false
css: unocss 
---
# FOUNDRY : FACTORY SOUS STEROÎDES

## CREER DES FIXTURES

Pour Sylius, Symfony et n'importe quel projet PHP en fait

---
layout: center
---

# De quoi on parle ?

- Qui suis-je ?
- Qu'est-ce que Foundry ?
- Exemples
  - Exemple de la collection de livre n°1
  - Exemple de la collection de livre n°2
- On y verra :
  - Debug
  - Fixtures
  - Reusable Model Factory "States"
  - Beaucoup de fonctions utiles
- Au sein de frameworks

---
layout: center
class: text-center
src: ./pages/presentations.md
---

---
layout: section
---
# zenstruck/foundry

### Kevin Bond : Open Source, PHP, Symfony Developer, Symfony Core Member.

Package officiellement recommandé par Symfony

---
layout: center
---

Foundry permet de créer des fixtures :

<v-clicks>

- expressive
- auto-complétable
- chargée à la demande 
- au sein de Symfony et/ou Doctrine

</v-clicks>

<!--
Expliquer pour foundry "expressive"
-->

---
layout: center
---

<img src="/composer_require.png" />

---
layout: center
---

<img src="/make_factory.png" />

<!-- Une factory est une classe permettant d'instancier un ou des objets -->

---
layout: default
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
---

# Exemples

---
layout: center
---

# Exemple : collection de livre

## Ajouter un avis

---
layout: center
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
---

# Quelques traits

<v-clicks>

- ResetDatabase : (re)création de la base via les schémas en place
- Factories : Initialise Foundry au sein du Kernel

</v-clicks>

---
layout: center
---

```php
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
title: Les stories

---

## Les stories

---
layout: center
hideOnToc: true
---

<img src="/make_story.png">

---
layout: center
---

```php
final class DefaultBookStory extends Story
{
    public function build(): void
    {
        BookFactory::createOne();
    }
}
```

```php
class ReviewTest extends AbstractTest
{
    public function testUserLeaveAReviewStory(): void
    {
        DefaultBookStory::load();
        $user = UserFactory::createOne();
        $token = $this->getToken(['email' => $user->getEmail(), 'password' => $user->getPassword()]);
        static::createClientWithCredentials($token)->request('POST', '/books/1/reviews', [
            'json' => [
                'content' => 'Lorem ipsum dolor sit amet',
            ],
        ]);
        $this->assertResponseIsSuccessful();
        $this->assertJsonContains(['@id' => '/reviews/1', 'book' => '/books/1']);
    }
```

---
layout: center
---

```php{1-7,13}
final class DefaultBookStory extends Story
{
    public function build(): void
    {
        BookFactory::createOne();
    }
}

class ReviewTest extends AbstractTest
{
    public function testUserLeaveAReviewStory(): void
    {
        DefaultBookStory::load();
        $user = UserFactory::createOne();
        $token = $this->getToken(['email' => $user->getEmail(), 'password' => $user->getPassword()]);
        static::createClientWithCredentials($token)->request('POST', '/books/1/reviews', [
            'json' => [
                'content' => 'Lorem ipsum dolor sit amet',
            ],
        ]);
        $this->assertResponseIsSuccessful();
        $this->assertJsonContains(['@id' => '/reviews/1', 'book' => '/books/1']);
    }
```

---
layout: center
---

# Exemple : collection de livre

## Récupérer les avis

---
layout: center
---

```php
class ReviewTest extends AbstractTest
{
    use ResetDatabase, Factories;

    public function testGetReviews(): void
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
---

# Il faut ajouter des reviews à mon livre, non ? 

---
layout: center
---

```php
class ReviewTest extends AbstractTest
{
    use ResetDatabase, Factories;

    public function testGetReviews(): void
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
---

```php{8-16}
class ReviewTest extends AbstractTest
{
    use ResetDatabase, Factories;

    public function testGetReviews(): void
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
---

## Plusieurs soucis

<v-clicks>

- Performance
- Je teste des choses qui ne sont pas liées à mon test

</v-clicks>

---
layout: center
---

```php
class ReviewTest extends AbstractTest
{
    use ResetDatabase, Factories;

    public function testGetReviews(): void
    {
        ReviewFactory::createMany(50, function() {
            return [
                'book' => BookFactory::findOrCreate(['isbn' => 143987391287]),
                'reviewer' => UserFactory::randomOrCreate(),
            ];
        });
        static::createClient()->request('GET', '/book/1/reviews');
        $this->assertResponseIsSuccessful();
        $this->assertJsonContains(['@id' => '/books/1/reviews']);
    }
}
```

<!--
ISBN, pas insb 

Switch rapidement our highlight les défauts 
-->

---
layout: center
---

```php{7-14}
class ReviewTest extends AbstractTest
{
    use ResetDatabase, Factories;

    public function testGetReviews(): void
    {
        ReviewFactory::createMany(50, function() {
            return [
                'book' => BookFactory::findOrCreate(['isbn' => 143987391287]),
                'reviewer' => UserFactory::randomOrCreate(),
            ];
        });
        static::createClient()->request('GET', '/book/1/reviews');
        $this->assertResponseIsSuccessful();
        $this->assertJsonContains(['@id' => '/books/1/reviews']);
    }
}
```

---
layout: center
title: Beaucoup de fonctionnalités

---

# On peut faire autrement ?

new(), findOrCreate, randomOrCreate et les autres fonctions ?

---
layout: center
---

# Les states ?

<v-clicks>

### (pas les USA)

</v-clicks>

---
layout: center
---

```php
final class BookFactory extends ModelFactory
{
    protected function getDefaults(): array
    {
        return [
            'isbn' => self::faker()->isbn10(),
            'name' => self::faker()->name(),
            'author' => AuthorFactory::randomOrCreate(),
        ];
    }
    
    public function withIsbn($isbn): self
    {
        return $this->addState(['isbn' => $isbn]);
    }
```

---
layout: center
---

```php{12-15}
final class BookFactory extends ModelFactory
{
    protected function getDefaults(): array
    {
        return [
            'isbn' => self::faker()->isbn10(),
            'name' => self::faker()->name(),
            'author' => AuthorFactory::randomOrCreate(),
        ];
    }
    
    public function withIsbn($isbn): self
    {
        return $this->addState(['isbn' => $isbn]);
    }
```

<!--
Le "find" arrive après
-->

---
layout: default
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

```php
public function testAuthorGetReviewFind(): void
    {
        BookFactory::new()->withIsbn(193847625)->create();
        ReviewFactory::createMany(50, [
            'book' => BookFactory::find(['isbn' => 193847625]),
            'reviewer' => UserFactory::randomOrCreate(),
        ]);
        static::createClient()->request('GET',
            sprintf('/books/%s/reviews', BookFactory::find(['isbn' => 193847625])->getId())
        );
        $this->assertResponseIsSuccessful();
        $this->assertJsonContains(['@id' =>
            sprintf('/books/%s/reviews', BookFactory::find(['isbn' => 193847625])->getId())]
        );
    }
```

<!--
On utilise Find, ensuite je parle inconvénient de répétition
-->

---
layout: center
---

# C'est lourd et on se répète

---
layout: center
---

```php{3,5,8,10}
public function testAuthorGetReviewFind(): void
    {
        $book = BookFactory::new()->withIsbn(193847625)->create();
        ReviewFactory::createMany(50, [
            'book' => $book,
            'reviewer' => UserFactory::randomOrCreate(),
        ]);
        static::createClient()->request('GET',sprintf('/books/%s/reviews', $book->getId()));
        $this->assertResponseIsSuccessful();
        $this->assertJsonContains(['@id' => sprintf('/books/%s/reviews', $book->getId())]);
    }
```

---
layout: center
---

# L'extraire dans une story 

---
layout: center
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
---

# C'est pas top

<v-clicks>

- ça fait pas exactement ce que ça semble dire faire
- on fait beaucoup de code pour pas grand chose
- ok mais faut des preuves

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
---

```php
class ReviewTest extends AbstractTest
{
    public function testGetReviewsStory(): void
    {
        DefaultReviewsStory::load();
        die();
        static::createClient()->request('GET', sprintf('/books/%s/reviews', BookFactory::random()->getId()));
        $this->assertResponseIsSuccessful();
        $this->assertJsonContains(['@id' => sprintf('/books/%s/reviews', BookFactory::random()->getId())]);
    }
}
```

---
layout: full
---

<img src="/dump_db_1.png" class="w-full h-full" />

---
layout: center
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

<!--
modif de factory puis story
-->

---
layout: center
---

```php
final class ReviewFactory extends ModelFactory
{
    protected function getDefaults(): array
    {
        return [
            'reviewer' => UserFactory::new(),
            'content' => self::faker()->text(),
            'book' => UserFactory::new()
        ];
    }
```

---
layout: center
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

    public function testGetReviewsStory(): void
    {
        $book = BookFactory::random();
        static::createClient()->request('GET', 
            sprintf('/books/%s/reviews', $book->getId())
        );
        $this->assertResponseIsSuccessful();
        $this->assertJsonContains(['@id' => sprintf('/books/%s/reviews', $book->getId())]);
    }
}
```
---
layout: default
---

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
---

```php
final class DefaultBookStory extends Story
{
    public function build(): void
    {
        $this->addState(
            'thefasttrack',
            BookFactory::createOne(['name' => 'The Fast Track'])
        );
    }
}

final class DefaultReviewsStory extends Story
{
    public function build(): void
    {
        ReviewFactory::createMany(10, [
            'book' => DefaultBookStory::get('thefasttrack'),
            //'book' => DefaultBookStory::$thefasttrack,
        ]);
    }
}
```

---
layout: center
---

```php
final class DefaultBookStory extends Story
{
    public const THE_FAST_TRACK = 'thefasttrack'
    
    public function build(): void
    {
        $this->addState(
            self::THE_FAST_TRACK,
            BookFactory::createOne(['name' => 'The Fast Track'])
        );
    }
}
```

---
layout: center
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

    public function testGetReviewsStory(): void
    {
        static::createClient()->request('GET',
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
            ),
            'hydra:totalItems' => 10
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

## API-platform

---
layout: full
hideInToc: true
---

```php
// src/Factory/BookFactory.php
protected function getDefaults(): array
{
    return [
        'author' => self::faker()->name(),
        'description' => self::faker()->text(),
        'isbn' => self::faker()->isbn13(),
        'publication_date' => \DateTimeImmutable::createFromMutable(self::faker()->dateTime()),
        'title' => self::faker()->sentence(4),
    ];
}
// src/Factory/ReviewFactory.php
use function Zenstruck\Foundry\lazy;

protected function getDefaults(): array
{
    return [
        'author' => self::faker()->name(),
        'body' => self::faker()->text(),
        'book' => lazy(fn() => BookFactory::randomOrCreate()),
        'publicationDate' => \DateTimeImmutable::createFromMutable(self::faker()->dateTime()),
        'rating' => self::faker()->numberBetween(0, 5),
    ];
}
```

---
layout: center
hideInToc: true
---

# Lazy permet de s'assurer que la valeur ne sera calculée que si elle est utilisée.

- Default est appelé à chaque fois que la factory est instanciée
- Permet de limiter les effets de bord

```php
use Zenstruck\Foundry\Attributes\LazyValue;
use function Zenstruck\Foundry\lazy;

protected function getDefaults(): array
{
    return [
        'category' => new LazyValue(fn() => CategoryFactory::random()),
        'file' => lazy(fn() => create_temp_file()), // or use the lazy() helper function
    ];
}
```

---
layout: center
hideInToc: true
---

```php
// src/Story/DefaultBooksStory.php

final class DefaultBooksStory extends Story
{
    public function build(): void
    {
        BookFactory::createMany(100);
    }
}
}
// src/Story/DefaultReviewsStory.php

final class DefaultReviewsStory extends Story
{
    public function build(): void
    {
        ReviewFactory::createMany(200);
    }
}

//src/DataFixtures/AppFixtures.php

class AppFixtures extends Fixture
{
    public function load(ObjectManager $manager): void
    {
        DefaultBooksStory::load();
        DefaultReviewsStory::load();
    }
}
```

---
layout: center
hideInToc: true
---

<img src="/doc_apip_alice.png">

---
layout: center
hideInToc: true
---

# Et dans un vrai projet ?

## Sylius

---
layout: default
---

```yaml 
sylius_fixtures:
  suites:
    default:
      fixtures:
        locale:
          options:
            locales:
              - 'en_US'
              - 'de_DE'
              - 'fr_FR'
              - 'pl_PL'
              - 'es_ES'
              - 'es_MX'
              - 'pt_PT'
              - 'zh_CN'
        currency:
          options:
            currencies:
              - 'EUR'
              - 'USD'
              - 'PLN'
              - 'CAD'
              - 'CNY'
              - 'NZD'
              - 'GBP'
              - 'AUD'
              - 'MXN'

        geographical:
          options:
            countries:
              - 'US'
              - 'FR'
              - 'DE'
              - 'AU'
              - 'CA'
              - 'MX'
              - 'NZ'
              - 'PT'
              - 'ES'
              - 'CN'
              - 'GB'
              - 'PL'
```

---
layout: center
---

# akawakaweb/sylius-fixtures-plugin

---
layout: center
---

### Cas pratique : Modifier la monnaie par défaut

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
---

## Je veux ajouter un second numéro de téléphone

```php
// src/Entity/Customer/Customer.php

#[ORM\Entity]
#[ORM\Table(name: 'sylius_customer')]
class Customer extends BaseCustomer
{
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

<!-- https://symfony.com/bundles/ZenstruckFoundryBundle/current/index.html#using-your-factory -->

---
layout: end
---

Disponible depuis aujourd'hui en 0.1 !

<!-- Remercier Akawaka, les tilleuls, le staff notamment Thibault et Grégoire Hebert -->
