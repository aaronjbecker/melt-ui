---
title: Date Range Field
description:
  An enhanced alternative to using multiple native date inputs for selecting a date range.
---

<script>
	import { APIReference, Preview, Callout } from '$docs/components'
	import { Code } from '$docs/components/markdown'
	export let snippets
	export let previews
	export let schemas
</script>

## Overview

The **Date Range Field** is an enhanced alternative to the rather limited date range selection setup
using two native inputs. It provides a more user-friendly interface for selecting dates and times,
is fully accessible, and works in all modern browsers & devices.

<Callout type="warning">
Before jumping into the docs for this builder, it's recommended that you understand how we work with dates &
times in Melt, which you can read about <a href="/docs/dates" target="_blank" class="underline">here</a>.
</Callout>

## Anatomy

- **Field**: The element which contains the date segments
- **Start Segment**: An individual segment of the start date (day, month, year, etc.)
- **End Segment**: An individual segment of the end date (day, month, year, etc.)
- **Label**: The label for the date field
- **Start Hidden Input**: A hidden input element containing the ISO 8601 formatted string of the
  start date
- **End Hidden Input**: A hidden input element containing the ISO 8601 formatted string of the end
  date

## Tutorial

Learn how to use the Date Range Field builder by starting simple and working your way up to more
complex examples and use cases. The goal is to teach you the key features and concepts of the
builder, so you won’t have to read through a bunch of API reference docs (Although, those are
available too!)

### Building a Date Range Field

Let's initialize our field using the `createDateRangeField` function, which returns an object
consisting of the stores & methods needed to construct the field.

To start off, we'll destructure the `field`, `startSegment`, `endSegment`, `label`, and
`segmentContents`.

```svelte showLineNumbers
<script lang="ts">
	import { createDateRangeField, melt } from '@melt-ui/svelte'

	const {
		elements: { field, startSegment, endSegment, label },
		states: { segmentContents }
	} = createDateRangeField()
</script>
```

The `field` is responsible for containing the date segments. The `label` for the date field is not
an actual `<label>` element, due to the way we interact with the field, but is still accessible to
screen readers in the same way.

```svelte showLineNumbers
<script lang="ts">
	import { createDateRangeField, melt } from '@melt-ui/svelte'

	const {
		elements: { field, startSegment, endSegment, label },
		states: { segmentContents }
	} = createDateRangeField()
</script>

<span use:melt={$label}>Trip Dates</span>
<div use:melt={$field}>
	<!-- ... -->
</div>
```

Unlike the other _elements_, `startSegment` & `endSegment` are functions which take a `SegmentPart`
as an argument, which is used to determine which segment this element represents.

While it's possible to use the start and end functions to render each segment individually like so:

```svelte showLineNumbers
<span use:melt={$label}>Trip Date</span>
<div use:melt={$field}>
	<div use:melt={$startSegment('day')}>
		<!-- ... -->
	</div>
	<div use:melt={$startSegment('month')}>
		<!-- ... -->
	</div>
	<div use:melt={$startSegment('year')}>
		<!-- ... -->
	</div>
	<div use:melt={$endSegment('day')}>
		<!-- ... -->
	</div>
	<!-- ...rest -->
</div>
```

It's not recommended, as the formatting doesn't adapt to the locale and type of date being
represented, which is one of the more powerful features this builder provides.

Instead, you can use the `segmentContents` state, which is an object containing `start` and `end`
properties, whose values are an array of objects necessary to form the date. Each object has a
<Code>part</Code> property, which is the `SegmentPart`, and a <Code>value</Code> property, which is
the locale-aware string representation of the segment.

```svelte showLineNumbers {12-16}
<script lang="ts">
	import { createDateRangeField, melt } from '@melt-ui/svelte'

	const {
		elements: { field, startSegment, endSegment, label },
		states: { segmentContents }
	} = createDateRangeField()
</script>

<span use:melt={$label}>Trip Dates</span>
<div use:melt={$field}>
	{#each $segmentContents.start as seg, i (i)}
		<div use:melt={$startSegment(seg.part)}>
			{seg.value}
		</div>
	{/each}
	<div aria-hidden="true">-</div>
	{#each $segmentContents.end as seg, i (i)}
		<div use:melt={$endSegment(seg.part)}>
			{seg.value}
		</div>
	{/each}
</div>
```

To actually use the value of the field, you can either use the `value` state directly, or if you're
using it within a form, the `startHiddenInput` & `endHiddenInput` elements, which are hidden input
elements containing an ISO 8601 formatted string of the value that input represents.

If you plan on using the hidden inputs, you'll need to pass the `startName` and `endName` props,
which will be used as the respective input's name attribute.

```svelte showLineNumbers {5-6,8-9,27-28}
<script lang="ts">
	import { createDateField, melt } from '@melt-ui/svelte'

	const {
		elements: { field, startSegment, endSegment, label, startHiddenInput, endHiddenInput },
		states: { segmentContents, value }
	} = createDateField({
		startName: 'tripStart',
		endName: 'tripEnd'
	})
</script>

<form method="POST">
	<span use:melt={$label}>Trip Dates</span>
	<div use:melt={$field}>
		{#each $segmentContents.start as seg, i (i)}
			<div use:melt={$startSegment(seg.part)}>
				{seg.value}
			</div>
		{/each}
		<div aria-hidden="true">-</div>
		{#each $segmentContents.end as seg, i (i)}
			<div use:melt={$endSegment(seg.part)}>
				{seg.value}
			</div>
		{/each}
		<input use:melt={$startHiddenInput} />
		<input use:melt={$endHiddenInput} />
	</div>
	<p>
		You selected:
		{#if $value}
			{$value}
		{/if}
	</p>
</form>
```

And that, along with some additional structure and styles, is all you need to get a fully functional
Date Range Field!

### The Power of Placeholder

In the previous example, we didn't pass any date-related props to the `createDateRangeField`
function, which means it defaulted to a `CalendarDate` object with the current date. What if we
wanted to start off with a different type of date, such as a `CalendarDateTime` or a
`ZonedDateTime`, but keep the initial value empty?

That's where the placeholder props come in, which consist of the uncontrolled `defaultPlaceholder`
prop, and the [controlled](/docs/controlled) `placeholder` prop.

In the absense of a `value` or `defaultValue` prop (which we'll cover soon), the placeholder holds
all the power in determining what type of date the field represents, and how it should be
rendered/formatted.

Let's convert our previous example into a Date & Time field, by passing a `CalendarDateTime` object
as the `defaultPlaceholder` prop.

```svelte showLineNumbers {3,9}
<script lang="ts">
	import { createDateRangeField, melt } from '@melt-ui/svelte'
	import { CalendarDateTime } from '@internationalized/date'

	const {
		elements: { field, startSegment, endSegment, label },
		states: { segmentContents }
	} = createDateRangeField({
		defaultPlaceholder: new CalendarDateTime(2023, 10, 11, 12, 30)
	})
</script>
```

<Preview code={snippets.defaultPh} variant="dark" size="sm">
	<svelte:component this={previews.defaultPh} />
</Preview>

As you can see above, by making that one change, the field now represents date and times, and the
segments have been updated to reflect that.

If your placeholder value is coming from a database or elsewhere, you can use one of the parser
functions provided by
[@internationalized/date](https://react-spectrum.adobe.com/internationalized/date/index.html) to
convert it into a `CalendarDateTime` object.

```ts
import { CalendarDateTime, parseDateTime } from '@internationalized/date'

// Constructor with a time
const date = new CalendarDateTime(2023, 10, 11, 12, 30)

// Constructor without a time (defaults to 00:00:00)
const date = new CalendarDateTime(2023, 10, 11)

// Parser function to convert an ISO 8601 formatted string
const date = parseDateTime('2023-10-11T12:30:00')
```

We can also just as easily convert the field into a Zoned Date & Time field, by passing a
`ZonedDateTime` object as the `defaultPlaceholder` prop.

```svelte showLineNumbers {3,9}
<script lang="ts">
	import { createDateRangeField, melt } from '@melt-ui/svelte'
	import { now, getLocalTimeZone } from '@internationalized/date'

	const {
		elements: { field, startSegment, endSegment, label },
		states: { segmentContents }
	} = createDateRangeField({
		defaultPlaceholder: now(getLocalTimeZone())
	})
</script>
```

We're using the `now` parser function to create a `ZonedDateTime` object with the current date and
time, and we're getting the user's local timezone using the `getLocalTimeZone` function.

<Preview code={snippets.nowLocalTz} variant="dark" size="sm">
	<svelte:component this={previews.nowLocalTz} />
</Preview>

Alternatively, we can hardcode the timezone to something like `America/Los_Angeles` by passing it as
the argument to the `now` function.

```svelte showLineNumbers {9}
<script lang="ts">
	import { createDateRangeField, melt } from '@melt-ui/svelte'
	import { now } from '@internationalized/date'

	const {
		elements: { field, startSegment, endSegment, label },
		states: { segmentContents }
	} = createDateRangeField({
		defaultPlaceholder: now('America/Los_Angeles')
	})
</script>
```

<Preview code={snippets.nowLA} variant="dark" size="sm">
	<svelte:component this={previews.nowLA} />
</Preview>

How you represent and store dates with timezones will depend entirely on your use case, but there
are a number of parser functions available to help you out, which you can read more about
[here](https://react-spectrum.adobe.com/internationalized/date/ZonedDateTime.html#introduction).

### Working with Values

Now that we've covered the basics of the Date Range Field, as well as the power of the placeholder,
let's look at how we can use the `defaultValue` to set the value of the field.

Remember, the placeholder props are only relevant in the absense of a `value` or `defaultValue`
prop, as they take precedence over the placeholder. When a `value` or `defaultValue` prop is passed,
the placeholder is set to the `end` value of those props.

We can demonstrate this by keeping the `defaultPlaceholder` prop as a `CalendarDate` object, and
passing a `DateRange` using `CalendarDateTime` objects as the `defaultValue` prop. It's not
recommended that you ever these to to different types, but it's useful to understand how the
placeholder & value props interact.

```svelte showLineNumbers {3,9-13}
<script lang="ts">
	import { createDateRangeField, melt } from '@melt-ui/svelte'
	import { CalendarDateTime, CalendarDate } from '@internationalized/date'

	const {
		elements: { field, startSegment, endSegment, label },
		states: { segmentContents }
	} = createDateRangeField({
		defaultPlaceholder: new CalendarDate(2023, 10, 11),
		defaultValue: {
			start: new CalendarDateTime(2023, 10, 11, 12, 30),
			end: new CalendarDateTime(2023, 10, 15, 12, 30)
		}
	})
</script>
```

<Preview code={snippets.defaultValue} variant="dark" size="sm">
	<svelte:component this={previews.defaultValue} />
</Preview>

As you can see, the field represents a `CalendarDateTime` object, and even if you clear the field,
it will retain that shape.

A really useful scenario for using the `defaultValue` prop and the `defaultPlaceholder` prop in
conjunction with one another is when a `defaultValue` may or may not be present, and you want to
ensure the field always represents a certain type of date.

For example, let's say this field is selecting a contact availability range in a user's profile,
which is optional, but you want to ensure that if they do enter a range, it's represented as a
`CalendarDateTime` object. The code to accomplishing that may look something like this:

```svelte showLineNumbers {3,11-14}
<script lang="ts">
	import { createDateRangeField, melt } from '@melt-ui/svelte'
	import { CalendarDateTime, CalendarDate, parseDateTime } from '@internationalized/date'

	export let data

	const {
		elements: { field, startSegment, endSegment, label },
		states: { segmentContents }
	} = createDateRangeField({
		defaultPlaceholder: new CalendarDate(2023, 10, 11),
		defaultValue: {
			start: data?.userAvailabilityStart ? parseDateTime(data?.userAvailabilityStart) : undefined,
			end: data?.userAvailabilityEnd ? parseDateTime(data?.userAvailabilityEnd) : undefined
		}
	})
</script>
```

If the user has those availability settings, we parse the ISO 8601 formatted string into a
`CalendarDateTime` object, and if not, we pass `undefined` for the start and end values, which will
cause the field to default to the `defaultPlaceholder` prop, which we've set to a `CalendarDate`
object.

The following example demonstrates how it would work in both scenarios (with and without
availability dates).

<Preview code={snippets.valueAndPh} variant="dark" size="sm">
	<svelte:component this={previews.valueAndPh} />
</Preview>

Situations like this make using the `defaultValue` and `defaultPlaceholder` props together a must.

### Validating Dates

This is where things start to get a lot more fun! This builder provides a few ways to validate
dates, which we'll cover in this section, starting with the `isDateUnavailable` prop.

The `isDateUnavailable` prop is a `Matcher` function, which takes a `DateValue` object as an
argument, and returns a boolean indicating whether or not that date is unavailable.

```ts
type Matcher = (date: DateValue) => boolean
```

If the date the user selects is unavailable, is marked as invalid, and you can do whatever you'd
like with that information.

Let's say that we don't want users to ever be able to select the 1st or the 15th of any month, as
those are the only two days we're not working on builder tutorials. We can setup a `Matcher`
function to accomplish just that.

```svelte showLineNumbers {2,4-6,12} /isInvalid/#hi /validation/#hi
<script lang="ts">
	import { createDateField, melt, type Matcher } from '@melt-ui/svelte'

	const isFirstOrFifteenth: Matcher = (date) => {
		return date.day === 1 || date.day === 15
	}

	const {
		elements: { field, startSegment, endSegment, label, validation },
		states: { value, segmentContents, isInvalid }
	} = createDateField({
		isDateUnavailable: isFirstOrFifteenth
	})
</script>
```

If you have a few different matchers you want to use, you can simply combine them like so:

```svelte showLineNumbers {8-10,12-14,20}
<script lang="ts">
	import { createDateField, melt, type Matcher } from '@melt-ui/svelte'

	const isFirstOrFifteenth: Matcher = (date) => {
		return date.day === 1 || date.day === 15
	}

	const isWeekend: Matcher = (date) => {
		return date.dayOfWeek === 0 || date.dayOfWeek === 6
	}

	const isDateUnavailable: Matcher = (date) => {
		return isFirstOrFifteenth(date) || isWeekend(date)
	}

	const {
		elements: { field, startSegment, endSegment, label, validation },
		states: { value, segmentContents, isInvalid }
	} = createDateField({
		isDateUnavailable
	})
</script>
```

Or if you want to get really fancy with it, you can create a helper function that takes an array of
matchers which you could use throughout your app.

```svelte showLineNumbers {12,14-18,24}
<script lang="ts">
	import { createDateField, melt, type Matcher } from '@melt-ui/svelte'

	const isFirstOrFifteenth: Matcher = (date) => {
		return date.day === 1 || date.day === 15
	}

	const isWeekend: Matcher = (date) => {
		return date.dayOfWeek === 0 || date.dayOfWeek === 6
	}

	const matchers = [isFirstOrFifteenth, isWeekend]

	const isDateUnavailable: (...matchers: Matcher[]) => Matcher = (...matchers) => {
		return (date) => {
			return matchers.some((matcher) => matcher(date))
		}
	}

	const {
		elements: { field, startSegment, endSegment, label, validation },
		states: { value, segmentContents, isInvalid }
	} = createDateField({
		isDateUnavailable
	})
</script>
```

When a field is marked as invalid, the `isInvalid` store will be set to `true`, and a `data-invalid`
attribute will be added to all the elements that make up the field, which you can use to style the
field however you'd like.

You'll want to use the `validation` element to display a message to the user indicating why the date
is invalid. It's automatically hidden when the field is valid, and is wired up via aria attributes
to give screen readers the information they need.

```svelte showLineNumbers {15}
<span use:melt={$label}>Availability</span>
<div use:melt={$field}>
	{#each $segmentContents.start as seg, i (i)}
		<div use:melt={$startSegment(seg.part)}>
			{seg.value}
		</div>
	{/each}
	<div aria-hidden="true">-</div>
	{#each $segmentContents.end as seg, i (i)}
		<div use:melt={$endSegment(seg.part)}>
			{seg.value}
		</div>
	{/each}
</div>
<small use:melt={$validation}> Date cannot be on the 1st or 15th of the month. </small>
```

Here's an example to get an idea of what you might do. Attempt to enter an unavailable date, and
you'll see the behavior in action.

<Preview code={snippets.unavailable} variant="dark" size="sm">
	<svelte:component this={previews.unavailable} />
</Preview>

The Date Field builder also accepts `minValue` and `maxValue` props to set the minimum and maximum
dates a user can select.

```svelte showLineNumbers {13-14}
<script lang="ts">
	import { createDateRangeField, melt } from '@melt-ui/svelte'
	import { CalendarDate } from '@internationalized/date'

	const {
		elements: { field, startSegment, endSegment, label, validation },
		states: { segmentContents }
	} = createDateRangeField({
		defaultValue: {
			start: new CalendarDate(2023, 10, 11),
			end: new CalendarDate(2023, 10, 13)
		},
		minValue: new CalendarDate(2023, 10, 11),
		maxValue: new CalendarDate(2024, 10, 11)
	})
</script>
```

In this example, we're limiting the selection dates to between October 11th, 2023 and October
11th, 2024.

<Preview code={snippets.minMax} variant="dark" size="sm">
	<svelte:component this={previews.minMax} />
</Preview>

If you increment the year of the end date to 2024, you'll see the validation message appear, as the
range exceeds the maximum date.

### Locale-aware Formatting

One of the coolest features of this builder is the ability to automatically format the segments and
placeholder based on the locale.

Of course it's up to you to decide how you get your user's locale, but once you have it, it's as
simple as passing it as the `locale` prop.

```svelte showLineNumbers {8}
<script lang="ts">
	import { createDateRangeField, melt } from '@melt-ui/svelte'

	const {
		elements: { field, startSegment, endSegment, label, validation },
		states: { segmentContents }
	} = createDateRangeField({
		locale: 'de'
	})
</script>
```

Here's an example showcasing a few different locales:

<Preview code={snippets.locales} variant="dark" size="default">
	<svelte:component this={previews.locales} />
</Preview>

Notice that they all have the same `defaultPlaceholder`, yet the segments are formatted differently
depending on the locale, and all it took was changing the `locale` prop. Pretty cool, right?

## API Reference

<APIReference {schemas} />
