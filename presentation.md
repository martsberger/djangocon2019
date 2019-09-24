# Level up the ORM

* Brad Martsberger
* Senior Software Engineer, Lumere, Inc.
* github.com/martsberger
{^ .title-info}

---

## Django 1.8 adds Case/When

Case/When low level Conditional Expressions

```python
Book.objects.annotate(
    five_star_count=Count(Case(When(Q(userrating__rating=5), then=1), default=None)),
    four_star_count=Count(Case(When(Q(userrating__rating=4), then=1), default=None)),
    three_star_count=Count(Case(When(Q(userrating__rating=3), then=1), default=None)),
    two_star_count=Count(Case(When(Q(userrating__rating=2), then=1), default=None)),
    one_star_count=Count(Case(When(Q(userrating__rating=1), then=1), default=None))
)
```
{^ .fragment}

---

## Django 2.0 adds filter to Aggregates

```python
Book.objects.annotate(
    five_star_count=Count('userrating', filter=Q(userrating__rating=5)),
    four_star_count=Count('userrating', filter=Q(userrating__rating=4)),
    three_star_count=Count('userrating', filter=Q(userrating__rating=3)),
    two_star_count=Count('userrating', filter=Q(userrating__rating=2)),
    one_star_count=Count('userrating', filter=Q(userrating__rating=1)))
```

---

## Django 2.2 adds bulk_update

```python
for instance in big_queryset:
    instance.attribute = complex_function(instance)
    instance.save()
```

A single SQL statement is generated from:
{: .fragment}

```python
instances = []
for instance in big_queryset:
    instance.attribute = complex_function(instance)
    instances.append(instance)
TheModel.objects.bulk_update(instances)
```
{^ .fragment}

```python
whens = [When(Q(pk=instance.pk), then=complex_function(instance))
         for instance in big_queryset]
pks = list(big_queryset.values_list('pk', flat=True))
TheModel.objects.filter(pk__in=pks).update(attribute=Case(*whens))
```
{^ .fragment}

---

## Third party app django-pivot

How many books in each genre has each author written?

```python
%> pip install django-pivot
```
{^ .fragment}

```python
authors = Author.objects.annotate(book_count=Count('book')).order_by('-book_count')
authors = pivot(authors, ['first_name', 'last_name'],
                         'book__genres__name',
                         'pk', Count, default=0)
```
{^ .fragment}

---

## Django 1.11 Added Subquery/OuterRef

[Subqueries have a remarkable performance benefit](https://medium.com/@hansonkd/the-dramatic-benefits-of-django-subqueries-and-annotations-4195e0dafb16)

[Making it possible doesn't make it easy](https://code.djangoproject.com/ticket/28296)

```python
authors = Author.objects.annotate(book_count=Count('book'))
```
{^ .fragment}

```python
authors = Author.objects.annotate(
    book_count=Coalesce(
               Subquery(Book.authors.through.objects
                            .filter(author_id=OuterRef('pk'))
                            .values('author_id')
                            .annotate(count=Count('book_id'))
                            .values('count')), 0)
)
```
{^ .fragment}

```python
%> pip install django_sql_utils
```
{^ .fragment}

```python
authors = Author.objects.annotate(book_count=SubqueryCount('book'))
```
{^ .fragment}
