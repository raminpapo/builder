# generate-content-url.test.ts

## File Metadata
- **Path**: `packages/sdks/src/functions/get-content/generate-content-url.test.ts`
- **Type**: Text file
- **Size**: 4,774 bytes (169 lines)
- **Extension**: `.ts`
- **Language**: TypeScript

## Original Source
```typescript
import { generateContentUrl } from './generate-content-url.js';
import type { GetContentOptions } from './types.js';

const testKey = 'YJIGb4i01jvw0SRdL5Bt';
const testModel = 'page';
const testId = 'c1b81bab59704599b997574eb0736def';

const options = {
  cachebust: 'true',
  noCache: 'true',
  'overrides.037948e52eaf4743afed464f02c70da4':
    '037948e52eaf4743afed464f02c70da4',
  'overrides.page': '037948e52eaf4743afed464f02c70da4',
  'overrides.page:/': '037948e52eaf4743afed464f02c70da4',
  preview: 'page',
};

describe('Generate Content URL', () => {
  test('generates the proper value for a simple query', () => {
    const output = generateContentUrl({
      apiKey: testKey,
      model: testModel,
      query: { id: testId },
    });
    expect(output).toMatchSnapshot();
  });

  test('Handles overrides correctly', () => {
    const output = generateContentUrl({
      apiKey: testKey,
      model: testModel,
      query: { id: testId },
      options,
    });
    expect(output).toMatchSnapshot();
  });

  test('generate content url with apiVersion as default', () => {
    const output = generateContentUrl({
      apiKey: testKey,
      model: testModel,
      query: { id: testId },
      options,
    });
    expect(output).toMatchSnapshot();
  });

  test('throw error when trying to generate content url with apiVersion as v1', () => {
    expect(() => {
      generateContentUrl({
        apiKey: testKey,
        model: testModel,
        query: { id: testId },
        options,
        apiVersion: 'v2' as GetContentOptions['apiVersion'],
      });
    }).toThrow(`Invalid apiVersion: expected 'v3', received 'v2'`);
  });

  test('generate content url with apiVersion as v3', () => {
    const output = generateContentUrl({
      apiKey: testKey,
      model: testModel,
      query: { id: testId },
      options,
      apiVersion: 'v3',
    });
    expect(output).toMatchSnapshot();
  });

  test('throw error when trying to generate content url with apiVersion as v1', () => {
    expect(() => {
      generateContentUrl({
        apiKey: testKey,
        model: testModel,
        query: { id: testId },
        options,
        apiVersion: 'v1' as GetContentOptions['apiVersion'],
      });
    }).toThrow(`Invalid apiVersion: expected 'v3', received 'v1'`);
  });

  test('throw error when trying to generate content url with an invalid apiVersion value', () => {
    expect(() => {
      generateContentUrl({
        apiKey: testKey,
        model: testModel,
        query: { id: testId },
        options,
        apiVersion: 'INVALID_API_VERSION' as GetContentOptions['apiVersion'],
      });
    }).toThrow(
      `Invalid apiVersion: expected 'v3', received 'INVALID_API_VERSION'`
    );
  });

  test('generate content url with enrich option true', () => {
    const output = generateContentUrl({
      apiKey: testKey,
      model: testModel,
      enrich: true,
    });
    expect(output).toMatchSnapshot();
  });

  test('generate content url with enrich option not present', () => {
    const output = generateContentUrl({
      apiKey: testKey,
      model: testModel,
    });
    expect(output).toMatchSnapshot();
  });

  test('generate content url with limit unset and check for noTraverse', () => {
    const output = generateContentUrl({
      apiKey: testKey,
      model: testModel,
    });
    expect(output).toMatchSnapshot();
  });

  test('generate content url with limit set to 2 and check for noTraverse', () => {
    const output = generateContentUrl({
      apiKey: testKey,
      model: testModel,
      limit: 2,
    });
    expect(output).toMatchSnapshot();
  });

  test('generate content url with limit set to 1 and check for noTraverse', () => {
    const output = generateContentUrl({
      apiKey: testKey,
      model: testModel,
      limit: 1,
    });
    expect(output).toMatchSnapshot();
  });

  test('generate content url with omit, fields, offset, includeUnpublished, cacheSeconds, staleCacheSeconds and sort combination', () => {
    const output = generateContentUrl({
      apiKey: testKey,
      model: testModel,
      omit: 'someId, some.nested.id',
      fields: 'id, nested.property',
      offset: 1,
      includeUnpublished: true,
      cacheSeconds: 5,
      staleCacheSeconds: 10,
      sort: {
        updatedDate: -1,
        createdDate: 1,
      },
    });
    expect(output).toMatchSnapshot();
  });

  test('generate content url when given invalid values of offset, includeUnpublished, cacheSeconds, staleCacheSeconds', () => {
    const output = generateContentUrl({
      apiKey: testKey,
      model: testModel,
      offset: -10,
      includeUnpublished: false,
      cacheSeconds: -5,
      staleCacheSeconds: -10,
    });
    expect(output).toMatchSnapshot();
  });
});

```


## High-Level Overview
JavaScript/TypeScript module. Contains 169 lines of code.

## Detailed Walkthrough


### Imports (2)

- `./generate-content-url.js`
- `./types.js`

## Usage Examples
*No usage examples found in source. Refer to tests or documentation.*

## Dependencies and Related Files
### Imports
- `./generate-content-url.js` (local)
- `./types.js` (local)

## Performance and Security Notes
*No specific performance or security concerns detected*

## Testing
This is a test file.

---
*Generated by Repo Book Generator v1.0.0*
