# Moment

<p class="tip">Moment is Lightpack's date and time utility class.</p>

Designing a highly flexible date and time library can be daunting. At the same time it can become so big that it may demand its own separate project.

**Moment** tries to solve some of the most frequently accessed date and time functions keeping its core dead simple and small. 

Below we document all the utilities provided by **Moment**.

## today()

To get today's date:

```php
Moment::today();
```

## tomorrow()

To get tomorrow's date:

```php
Moment::tomorrow();
```

## yesterday()

To get yesterday's date:

```php
Moment::yesterday();
```

## next()

To get the date of upcoming next day by name:

```php
Moment::next('monday');
Moment::next('tuesday');
Moment::next('wednesday');
Moment::next('thursday');
Moment::next('friday');
Moment::next('saturday');
Moment::next('sunday');
```

## last()

To get the date of last day by name:

```php
Moment::last('monday');
Moment::last('tuesday');
Moment::last('wednesday');
Moment::last('thursday');
Moment::last('friday');
Moment::last('saturday');
Moment::last('sunday');
```

## thisMonthEnd()

To get the date of last day of current month:

```php
Moment::thisMonthEnd();
```

## nextMonthEnd()

To get the date of last day of next month:

```php
Moment::nextMonthEnd();
```