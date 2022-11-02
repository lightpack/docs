# Moment

<p class="tip">Moment is Lightpack's date and time utility class.</p>

Designing a highly flexible date and time library can be daunting. At the same time it can become so big that it may demand its own separate project.

**Moment** tries to solve some of the most frequently accessed date and time functions keeping its core dead simple and small. You can access an instance of Moment class by calling `moment()` function.

## Formatting

By default, **Moment** returns **datetime** in `Y-m-d H:i:s` format. To change this, you can use `format()` method.

For example:

```php
moment()->format('d-M, Y H:ia');
```

## Available methods:

Below we document all the utilities provided by **Moment**.

### now()

To get current **datetime**:

```php
moment()->now();
```

You can optionally pass it the **datetime** format as string.

```php
moment()->now('d-M, Y');
```

### travel()

To travel ahead of or behind of current date and time:

```php
moment()->travel('+5 minutes');
moment()->travel('-5 minutes');
moment()->travel('+5 hours');
moment()->travel('-5 hours');
moment()->travel('+5 days');
moment()->travel('-5 days');
moment()->travel('+5 weeks');
moment()->travel('-5 weeks');
moment()->travel('2 AM');
moment()->travel('2 PM');
moment()->travel('noon');
moment()->travel('midnight');
moment()->travel('last day of this month');
moment()->travel('last day of next month');
moment()->travel('last day of previous month');
moment()->travel('first day of april');
moment()->travel('last day of april');
moment()->travel('first day of april last year');
moment()->travel('first day of april next year');
```

You can optionally pass the **datetime** format as second argument.

```php
moment()->travel('+5 days', 'd-M, Y');
```

### today()

To get **today's** date:

```php
moment()->today();
```

You can optionally pass it the **datetime** format as string.

```php
moment()->today('d-M, Y');
```

### tomorrow()

To get **tomorrow's** date:

```php
moment()->tomorrow();
```

You can optionally pass it the **datetime** format as string.

```php
moment()->tomorrow('d-M, Y');
```

### yesterday()

To get **yesterday's** date:

```php
moment()->yesterday();
```

You can optionally pass it the **datetime** format as string.

```php
moment()->yesterday('d-M, Y');
```

### next()

To get the date of **upcoming** next day by name:

```php
moment()->next('monday');
moment()->next('tuesday');
moment()->next('wednesday');
moment()->next('thursday');
moment()->next('friday');
moment()->next('saturday');
moment()->next('sunday');
```

You can optionally pass it the **datetime** format as string.

```php
moment()->next('sunday', 'd-M, Y');
```

### last()

To get the date of **last** day by name:

```php
moment()->last('monday');
moment()->last('tuesday');
moment()->last('wednesday');
moment()->last('thursday');
moment()->last('friday');
moment()->last('saturday');
moment()->last('sunday');
```

You can optionally pass it the **datetime** format as string.

```php
moment()->last('sunday', 'd-M, Y');
```

### thisMonthEnd()

To get the date of last day of **current** month:

```php
moment()->thisMonthEnd();
```

You can optionally pass it the **datetime** format as string.

```php
moment()->thisMonthEnd('d-M, Y');
```

### nextMonthEnd()

To get the date of last day of **next** month:

```php
moment()->nextMonthEnd();
```

You can optionally pass it the **datetime** format as string.

```php
moment()->nextMonthEnd('d-M, Y');
```

### lastMonthEnd()

To get the date of **last day** of last month:

```php
moment()->lastMonthEnd();
```

You can optionally pass it the **datetime** format as string.

```php
moment()->lastMonthEnd('d-M, Y');
```

### daysBetween()

To get the **number** of days between two dates:

```php
moment()->daysBetween('2021-07-23', '2021-05-18');
```

### diff()

To get the difference between two **datetimes** as an interval:

```php
$diff = moment()->diff('2021-07-23 14:25:45', '2019-03-14 08:23:12');
```

Now you can access a number of properties on `$diff` object:

```php
$diff->y; // Number of years.
$diff->m; // Number of months.
$diff->d; // Number of days.
$diff->h; // Number of hours.
$diff->i; // Number of minutes.
$diff->s; // Number of seconds.
```

For example:

```php
// 2 years, 4 months, 9 days
"{$diff->y} years, {$diff->m} months, {$diff->d} days";
```

### fromNow()

Use this function to get date and time as a friendly text like following:

```text
just now
a minute ago
5 minutes ago
an hour ago
10 hours ago
yesterday
5 days ago
a week ago
2 weeks ago
a month ago
5 months ago
a year ago
3 years ago
```

Pass it the **datetime** string you want to compare:

```php
moment()->fromNow('2021-07-20 11:30:45');
```

### create()

Use this function to create a `DateTime` object. Optionally pass it the **datetime** string as argument which defaults to `now`.

Create a DateTime object with datetime `now`:

```php
moment()->create();
```

Or, create a DateTime object with datetime `2021-07-23`:

```php
moment()->create('2021-07-23');
```

Because it returns `DateTime` object, hence you can apply all the methods exposed via `DateTime` class in **PHP**.

For example:

```php
// Add 2 hours in current datetime and
// return datetime in Y-m-d H:i:s format

moment()->create()->modify('+2 hours')->format('Y-m-d H:i:s');
```

Another example:

```php
// Add 5 days in 2021-07-23 and 
// return datetime in Y-m-d format

moment()->create('2021-07-23')->modify('+5 days')->format('Y-m-d'),
```