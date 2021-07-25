# Moment

<p class="tip">Moment is Lightpack's date and time utility class.</p>

Designing a highly flexible date and time library can be daunting. At the same time it can become so big that it may demand its own separate project.

**Moment** tries to solve some of the most frequently accessed date and time functions keeping its core dead simple and small. 

## Formatting

By default, **Moment** returns **datetime** in `Y-m-d H:i:s` format. To change this, you can use `format()` method.

For example:

```php
Moment::format('d-M, Y H:ia');
```

## Available methods:

Below we document all the utilities provided by **Moment**.

### now()

To get current **datetime**:

```php
Moment::now();
```

You can optionally pass it the **datetime** format as string.

```php
Moment::now('d-M, Y');
```

### travel()

To travel ahead of or behind of current date and time:

```php
Moment::travel('+5 minutes');
Moment::travel('-5 minutes');
Moment::travel('+5 hours');
Moment::travel('-5 hours');
Moment::travel('+5 days');
Moment::travel('-5 days');
Moment::travel('+5 weeks');
Moment::travel('-5 weeks');
Moment::travel('2 AM');
Moment::travel('2 PM');
Moment::travel('noon');
Moment::travel('midnight');
Moment::travel('last day of this month');
Moment::travel('last day of next month');
Moment::travel('last day of previous month');
Moment::travel('first day of april');
Moment::travel('last day of april');
Moment::travel('first day of april last year');
Moment::travel('first day of april next year');
```

You can optionally pass the **datetime** format as second argument.

```php
Moment::travel('+5 days', 'd-M, Y');
```

### today()

To get **today's** date:

```php
Moment::today();
```

You can optionally pass it the **datetime** format as string.

```php
Moment::today('d-M, Y');
```

### tomorrow()

To get **tomorrow's** date:

```php
Moment::tomorrow();
```

You can optionally pass it the **datetime** format as string.

```php
Moment::tomorrow('d-M, Y');
```

### yesterday()

To get **yesterday's** date:

```php
Moment::yesterday();
```

You can optionally pass it the **datetime** format as string.

```php
Moment::yesterday('d-M, Y');
```

### next()

To get the date of **upcoming** next day by name:

```php
Moment::next('monday');
Moment::next('tuesday');
Moment::next('wednesday');
Moment::next('thursday');
Moment::next('friday');
Moment::next('saturday');
Moment::next('sunday');
```

You can optionally pass it the **datetime** format as string.

```php
Moment::next('sunday', 'd-M, Y');
```

### last()

To get the date of **last** day by name:

```php
Moment::last('monday');
Moment::last('tuesday');
Moment::last('wednesday');
Moment::last('thursday');
Moment::last('friday');
Moment::last('saturday');
Moment::last('sunday');
```

You can optionally pass it the **datetime** format as string.

```php
Moment::last('sunday', 'd-M, Y');
```

### thisMonthEnd()

To get the date of last day of **current** month:

```php
Moment::thisMonthEnd();
```

You can optionally pass it the **datetime** format as string.

```php
Moment::thisMonthEnd('d-M, Y');
```

### nextMonthEnd()

To get the date of last day of **next** month:

```php
Moment::nextMonthEnd();
```

You can optionally pass it the **datetime** format as string.

```php
Moment::nextMonthEnd('d-M, Y');
```

### lastMonthEnd()

To get the date of **last day** of last month:

```php
Moment::lastMonthEnd();
```

You can optionally pass it the **datetime** format as string.

```php
Moment::lastMonthEnd('d-M, Y');
```

### daysBetween()

To get the **number** of days between two dates:

```php
Moment::daysBetween('2021-07-23', '2021-05-18');
```

### diff()

To get the difference between two **datetimes** as an interval:

```php
$diff = Moment::diff('2021-07-23 14:25:45', '2019-03-14 08:23:12');
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
Moment::fromNow('2021-07-20 11:30:45');
```

### create()

Use this function to create a `DateTime` object. Optionally pass it the **datetime** string as argument which defaults to `now`.

Create a DateTime object with datetime `now`:

```php
Moment::create();
```

Or, create a DateTime object with datetime `2021-07-23`:

```php
Moment::create('2021-07-23');
```

Because it returns `DateTime` object, hence you can apply all the methods exposed via `DateTime` class in **PHP**.

For example:

```php
// Add 2 hours in current datetime and
// return datetime in Y-m-d H:i:s format

Moment::create()->modify('+2 hours')->format('Y-m-d H:i:s');
```

Another example:

```php
// Add 5 days in 2021-07-23 and 
// return datetime in Y-m-d format

Moment::create('2021-07-23')->modify('+5 days')->format('Y-m-d'),
```