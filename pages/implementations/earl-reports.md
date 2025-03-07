---
title: Reporting Format
---

## Contribute An Implementation

If you developed an accessibility tool or a testing methodology and would like to have your implementation included in the WAI website, there are two ways you can do so.

1. If you have a tool that can return a data format, you will need to run your tests against the [test cases](../testcases/) and [submit a report](../reporting/).
2. If you have a manual test methodology where you fill results into some report template or semi-automated tool, you [can use the ACT Implementor tool](https://act-implementor.netlify.app/#/) instead to produce implementation reports.

## Understanding the Reporting format

To display an accessibility test tool or methodology on the WAI website, use the [Evaluation And Reporting Language](https://www.w3.org/TR/EARL10-Schema/) expressed using [JSON-LD](https://www.w3.org/TR/json-ld/). Tool developers that have an EARL reporter should be able to provide their existing reports. If you don't already have an EARL reporter, we provided a basic data structure that you can use.

### Context And Graph

On the top level object, there should be a `@context` and `@graph`. The `@context` is a URL to a definition file that describes the data structure. The `@graph` property contains an array of `TestSubject` objects.

```json
{
  "@context": "https://act-rules.github.io/earl-context.json",
  "@graph": [ ... ]
}
```

### Test Subject

Create a single test subject for each [test case](../testcase/). The `TestSubject` has three required properties:

- `@type`: Must be `TestSubject`, to distinguish it from other data types that might exist in an EARL report.
- `source`: This must be the URL of the test case.
- `assertions`: This contains an array of `Assertion` objects

```json
{
  "@context": "https://act-rules.github.io/earl-context.json",
  "@graph": [{
    "@type": "TestSubject",
    "source": "https://act-rules.github.io/testcases/a1b64e/6c3ac31577c3cb2d968fc26c4075dd533b5513fc.html",
    "assertions": [ ... ]
  }]
}
```

**Advanced**: `assertions` is a `@reverse` of `earl:subject`, to group assertions by test subject. When the JSON-LD is framed, the structure is flattened back to `earl:subject`. Using `earl:subject` and a flat array of Assertions works as well.

### Assertion

Add an `Assertion` object to the `assertions` array for each outcome provided by any of the rules / test procedures in your implementation. Generally, test cases are atomic enough that every implemented rule would only have a single outcome, although in some cases (such as duplicate IDs) a rule may return more than one outcome. In such cases, there should be multiple Assertions, one for each outcome.

The following properties are required for each Assertion:

- `@type`: This must be `Assertion`, to distinguish it from other data types that might exist in an EARL report.
- `test`: See [Test Criterion](#test-criterion).
- `result.outcome`: One of the following values:

  - `earl:passed`: A node in the test case passed the rule
  - `earl:failed`: A node in the test case failed the rule
  - `earl:inapplicable`: The rule is inapplicable in the test case
  - `earl:cantTell`: The tool could not be sure whether or not a node passed of failed the rule
  - `earl:untested`: The rule was not executed on the test case

```json
{
	"@context": "https://act-rules.github.io/earl-context.json",
	"@graph": [
		{
			"@type": "TestSubject",
			"source": "https://act-rules.github.io/testcases/a1b64e/6c3ac31577c3cb2d968fc26c4075dd533b5513fc.html",
			"assertions": [
				{
					"@type": "Assertion",
					"result": { "outcome": "earl:passed" },
					"test": { ... },
				}
			]
		}
	]
}
```

### Test Criterion

For each result, we'll need to know which rule or procedure in the tool reported it, and which WCAG success criteria were failed. To do so, on the `test` property, add an object. This must include:

- `title`: A string, that uniquely identifies which rule or procedure in the tool found the result.
- `isPartOf`: An array of strings, listing the WCAG success criteria that fail when the rule or procedure fails. These must use the IDs of the WCAG success criteria, prefaced with `WCAG2:`. For example `WCAG2:name-role-value`.

```json
{
	"@context": "https://act-rules.github.io/earl-context.json",
	"@graph": [
		{
			"@type": "TestSubject",
			"source": "https://act-rules.github.io/testcases/a1b64e/6c3ac31577c3cb2d968fc26c4075dd533b5513fc.html",
			"assertions": [
				{
					"@type": "Assertion",
					"result": { "outcome": "earl:passed" },
					"test": {
						"title": "image-button-has-name",
						"isPartOf": ["WCAG2:non-text-content", "WCAG2:name-role-value"]
					}
				}
			]
		}
	]
}
```

**Note**: The IDs of WCAG success criteria can be found in [sc-urls.json](https://github.com/act-rules/act-tools/blob/main/src/data/sc-urls.json), on the `scId` property. These are the IDs introduced in WCAG 2.1, and continued in WCAG 2.2. Use these IDs, even when reporting for WCAG 2.0.

### isPartOf

Alternatively, it is possible to specify WCAG 2 success criteria by number instead of by ID. This can be done in the following format:

```json
{
	"@context": "https://act-rules.github.io/earl-context.json",
	"@graph": [
		{
			"@type": "TestSubject",
			"source": "https://act-rules.github.io/testcases/a1b64e/6c3ac31577c3cb2d968fc26c4075dd533b5513fc.html",
			"assertions": [
				{
					"@type": "Assertion",
					"result": { "outcome": "earl:passed" },
					"test": {
						"title": "image-button-has-name",
						"isPartOf": [{ "title": "WCAG 2: 1.1.1" }, { "title": "WCAG 2: 4.1.2" }]
					}
				}
			]
		}
	]
}
```

The format of the `title` property in the `isPartOf` array is fairly permissive. To be recognized as a success criterion, the string `WCAG 2` or `WCAG2`, and the success criterion number (separated with dots) must be included. For example `WCAG 2.1 criterion 1.1.1 Non-text content` would be recognized. To explicitly add the success criterion ID the `@id` property can be used.

**Advanced**: All required properties map to [EARL](http://www.w3.org/ns/earl#), except for `isPartOf`, `title`, and `source` which are properties of [Dublin Core](http://purl.org/dc/terms/). Property names can be anything, as long as they can be expanded to the correct URL.
