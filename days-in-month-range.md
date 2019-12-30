# Days in Month Range

```
MIN(
  MAX(DAYS(EOMONTH("current column date", 0), "range start date"), 0),
  DAY(EOMONTH("current column date", 0))
)
-
MIN(
  MAX(DAYS(EOMONTH("current column date", 0), "range end date"), 0),
  DAY(EOMONTH("current column date", 0))
)
```

## Overview

This formula calculates the number of days in a given month that is included in a given date range. For example:

**Date range:** January 7, 2020 to April 12, 2020

| Dec 2019 | Jan 2020 | Feb 2020 | Mar 2020 | Apr 2020 | May 2020 |
|----------|----------|----------|----------|----------|----------|
|    0     |    24    |    28    |    31    |    12    |    0     |


## Goals

1. Months before the range start date should be 0.
2. The month of the range start date should be less than the number of days in the month.
3. The month of the range end date should be less than the number of days in the month.
4. Months after the range end date should be 0.
5. If a Month contains both the range start date and range end date, it should have the difference between the 2 dates.

## Functions

+ [`DAYS`][DAYS]: The DAYS function returns the number of days between two dates.
+ [`DAY`][DAY]: Returns the day of the month that a specific date falls on, in numeric format.
+ [`EOMONTH`][EOMONTH]: Returns a date representing the last day of a month which falls a specified number of months before or after another date.
+ [`MAX`][MAX]: Returns the maximum value in a numeric dataset.
+ [`MIN`][MIN]: Returns the minimum value in a numeric dataset.

## Steps

**1. Calculate the number of days between the end of month of current column and the range start.**

```
DAYS(
  EOMONTH("current column date", 0),
  "range start date"
)
```

Using [`EOMONTH`][EOMONTH] we find the last day of the current column date. This could be the 28th, 29th, 30th, or 31st depending on the month. Next, we use the [`DAYS`][DAYS] function to calculate the number of days between the end of the month and the range start date. This number will be negative if the range start date is in a future month from the current column date. If the range start is in a previous month or current month it will be >= 0.

**2. Floor the result at `0` to ensure no negative days.**

```
MAX(
  DAYS(EOMONTH("current column date", 0), "range start date"),
  0
)
```

By wrapping the result in a [`MAX`][MAX] with `0`, we ensure that range start dates in future months will result in a `0` result.

**3. Find the number of days in the current month.**

```
DAY(EOMONTH("current column date", 0))
```

Using the [`DAY`][DAY] function with [`EOMONTH`][EOMONTH], we can get the number of days in the current month.

**4. Combine Step 2 and 3 to find the days remaining in the current month.**

```
MIN(
  MAX(DAYS(EOMONTH("current column date", 0), "range start date"), 0),
  DAY(EOMONTH("current column date", 0))
)
```

Now we combine together Step 2 and Step 3 by using the [`MIN`][MIN] function. The result from Step 2 will produce the number of days between the end of the current month and the range start. If the range start is in a previous month, Step 2 will have a number greater than the number of days in the current month. If the range start is in the current month, Step 2 will have a number less than or equal to the number of days in the current month. By calculating the min with Step 3 (the number of days in the current month), we will get one of 3 possible results:

+ `0`: if the range start date is in a future month from the current column date
+ `0 to "number of days in month"`: if the range start is in the same month as the current column date
+ `"number of days in month"`: if the range start is in a previous month to the current column date

**5. Pause here and review what we have built.**

Using our formula so far, here is what we have created:

**Date range:** January 7, 2020 to April 12, 2020

| Dec 2019 | Jan 2020 | Feb 2020 | Mar 2020 | Apr 2020 | May 2020 |
|----------|----------|----------|----------|----------|----------|
|    0     |    24    |    28    |    31    |    30    |    31    |

+ What we have so far:
  - We have `0` for months that were before the range start date.
  - We have accounted for the partial month when the range starts.
  - We have full months for months after the month of the range start.
+ What we are missing:
  - We have not accounted for the partial month when the range ends.
  - We do not have `0` for months after the range end date.

In order to satisfy what we are missing we can redo our current steps but use the range end date instead of the range start date.

**6. Calculate days remaining in the current month from the range end date.**

```
MIN(
  MAX(DAYS(EOMONTH("current column date", 0), "range end date"), 0),
  DAY(EOMONTH("current column date", 0))
)
```

Here, we have simply copied what we had in Step 4, but replaced the "range start date" with the "range end date". Here is what that gives us:

**Date range:** January 7, 2020 to April 12, 2020

| Dec 2019 | Jan 2020 | Feb 2020 | Mar 2020 | Apr 2020 | May 2020 |
|----------|----------|----------|----------|----------|----------|
|    0     |    0     |    0     |    0     |    18    |    31    |

+ We have `0` for months before the range end date.
+ We have a partial month for the month of the range end date.
+ We have full months for months after the range end date.

There is one key item to note. Our row shows `18`, which is the number of days remaining in the month starting from the range end date. However, we want this row to show a `12` as that is the number of days which should be included in the range. To accomplish this, we can simply subtract our formula from Step 4 with our formula here. Months after the range start and end date will subtract to `0`.

**7. Subtract Step 4 and Step 6 to create final result.**

```
MIN(
  MAX(DAYS(EOMONTH("current column date", 0), "range start date"), 0),
  DAY(EOMONTH("current column date", 0))
)
-
MIN(
  MAX(DAYS(EOMONTH("current column date", 0), "range end date"), 0),
  DAY(EOMONTH("current column date", 0))
)
```

By subtracting our 2 formulas we are able to achieve our result as illustrated below:

**Date range:** January 7, 2020 to April 12, 2020

|                          | Dec 2019 | Jan 2020 | Feb 2020 | Mar 2020 | Apr 2020 | May 2020 |
|--------------------------|----------|----------|----------|----------|----------|----------|
| Formula with range start |    0     |    24    |    28    |    31    |    30    |    31    |
| Formula with range end   |    0     |    0     |    0     |    0     |    18    |    31    |
| Difference of formulas   |    0     |    24    |    28    |    31    |    12    |    0     |

## Minified Formula

```
MIN(MAX(DAYS(EOMONTH("current column date", 0), "range start date"), 0), DAY(EOMONTH("current column date", 0))) - MIN(MAX(DAYS(EOMONTH("current column date", 0), "range end date"), 0), DAY(EOMONTH("current column date", 0)))
```

[DAYS]: https://support.google.com/docs/answer/9061296?hl=en
[DAY]: https://support.google.com/docs/answer/3093040?hl=en
[EOMONTH]: https://support.google.com/docs/answer/3093044?hl=en
[MAX]: https://support.google.com/docs/answer/3094013?hl=en
[MIN]: https://support.google.com/docs/answer/3094017?hl=en
