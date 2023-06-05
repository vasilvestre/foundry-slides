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

<img src="/composer_require.png" />

---
layout: center
hideInToc: true
---

<img src="/make_factory.png" />

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

```php{3,7,8,9}
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
hideInToc: true
---

```php
class ReviewTest extends AbstractTest
{
    use ResetDatabase, Factories;

    public function testAuthorGetReviews(): void
    {
        $book = BookFactory::createOne();
        static::createClient()->request('GET', sprintf('/book/%s/reviews', $book->getId()));
        $this->assertResponseIsSuccessful();
        $this->assertJsonContains(['@id' => '/books/1/reviews']);
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

- Performance
- Je teste des choses qui ne sont pas liées à mon test

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
        static::createClientWithCredentials($token)->request('GET', sprintf('/books/1/reviews'));
        $this->assertResponseIsSuccessful();
        $this->assertJsonContains(['@id' => '/books/1/reviews']);
    }
}
```

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
        static::createClientWithCredentials($token)->request('GET', sprintf('/books/1/reviews'));
        $this->assertResponseIsSuccessful();
        $this->assertJsonContains(['@id' => '/books/1/reviews']);
    }
}
```

---
layout: center
level: 2
title: Les Storys
---

## Les storys

<img src="make_story.png">

---
layout: end
hideInToc: true
---

<!-- Remercier Akawaka, le staff notamment Thibault et Grégoire Hebert -->