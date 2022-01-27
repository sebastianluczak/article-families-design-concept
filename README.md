# Business Feature: "Article Families"
> TODO: Add Domain Dictionary

## Description of feature

We want to have articles in child-parent relations and manipulate them from Admin Panel.

```behat
As an Author,
Using Admin Panel,
Being logged in as Author,
Being fully authenticated,
While I'm editing or creating Article
I want to choose many articles as related
Using simple choice dropdown
```
<details>
  <summary>
    Please note that there's no opposite functionality, adding parent article from child perspective*** !
  </summary>
  <p>
    Reasons are:
    1. opposite funtionality would require us to rewrite the way Articles are working in our CMS bundle, or
    2. with "custom implementation" we would lose basically everything our CMS provides and have to do it ourselves
  </p>
</details>

## Concept

***Each*** `ArticlePage` (feature name: "Article") in the system can have ***many*** `relatedArticles` (feature name: "Article Families").

## Design

Entity `App\Entity\Pages\ArticlePage` contains self-reference relation such as below:
```php
    /**
     * @var ArrayCollection
     *
     * @ORM\ManyToMany(targetEntity="App\Entity\Pages\ArticlePage")
     * @ORM\JoinTable(name="app_article_related_articles",
     *     joinColumns={@ORM\JoinColumn(name="article_page_id", referencedColumnName="id")},
     *     inverseJoinColumns={@ORM\JoinColumn(name="article_related_page_id", referencedColumnName="id")}
     * )
     */
    protected $relatedArticles;
```

Custom `FormType` has been created to handle AdminList to correctly handle incoming Requests from KumaCMS
It's localed in `App\Form\Pages\ArticlePageAdminType` and cointains basic logic of building Form fields in Symfony.

Field is handled by `EntityClass` which allows us to not use faulty `CollectionType`, but requires to add custom `query_builder`.

```php
$builder->add('relatedArticles',  EntityType::class, [
            'class' => ArticlePage::class,
            'choice_label' => 'title',
            'query_builder' => function (EntityRepository $er) {
                return $er->createQueryBuilder('t')
                    ->orderBy('t.title', 'ASC');
            },
            'multiple' => true,
            'expanded' => false,
            'attr' => [
                'class' => 'js-advanced-select',
                'data-placeholder' => 'Choose the related articles'
            ],
            'required' => true
        ])
```

Frontend perspective, we're using Object pulled as `PersistentCollection` from DB that has `\IterableAcess` interface.
Usage is as simple as iterating over array and using `ArticlePage` as object reference.
No external or additional Twig

```twig
    {% set relatedArticles = page.getRelatedArticles() %}
    {% if relatedArticles is not empty  %}
    <section class="articles articles--article-related padding-block">
        <div class="articles--article-related__title">
            <h2 class="articles--article-related__title__text">
                Daugiau istorijų
            </h2>
            <div class="articles--article-related__title__tags">
                {% for tag in page.tags %}
                    <a href="" class="articles--article-related__title__tags__item">#{{ tag.name }}</a>
                {% endfor %}
            </div>
        </div>
        <div class="glider-contain">
            <div class="glider articles-row">

                {% for article in relatedArticles %}
                    {% include 'Components/_arcticle_card.html.twig' with {'article': article} %}
                {% endfor %}
            </div>
            <button aria-label="Previous" class="glider-prev">
                <img src="/frontend/img/dummy/icons/arrow_left.svg" alt="arrow left">
            </button>
            <button aria-label="Next" class="glider-next">
                <img src="/frontend/img/dummy/icons/arrow_right.svg" alt="arrow right">
            </button>
        </div>
    </section>
    {% endif %}
```

## Usage

From AdminPanel side related articles are handled in Article Creation Wizard Service™®.
http://localhost/en/admin/my-articles/

By clicking "Add new" or editing existing Article we can add ***child*** ***articles*** as related articles in below section.

![image](https://user-images.githubusercontent.com/1628839/151363885-4fd5307a-24a1-4144-ac8c-e043b37aa83d.png)


Autocomplete feature gives us access to filtering by Article Title.

![image](https://user-images.githubusercontent.com/1628839/151364059-4ecbe74c-5043-43fe-9d93-42bc61f3488b.png)



Articles selected became ***child*** articles and article you were creating became a ***parent*** for them.

![image](https://user-images.githubusercontent.com/1628839/151364446-d32be368-cbe4-4994-86df-900458b67adb.png)



If any ***Parent*** Article has child articles set up in AdminPanel they'll show up on ArticlePage dynamically at the end.
![image](https://user-images.githubusercontent.com/1628839/151366027-7df16fb5-81ba-4d76-a236-137b3b725f8d.png)



## Current problems

* Article Families can accept any article without checking roles, permissions etc.
* No ability to add some Article as Parent on child page, yet
* Was not tested how it behaves in different situations, i.e. 1 article, 2 articles, 75226 articles, "bsdbdfbdfb", a lizard
