# tracking.spec.ts

## File Metadata
- **Path**: `packages/sdks-tests/src/tests/tracking.spec.ts`
- **Type**: Text file
- **Size**: 4,985 bytes (141 lines)
- **Extension**: `.ts`
- **Language**: TypeScript

## Original Source
```typescript
import { expect } from '@playwright/test';
import { EXCLUDE_GEN_1, EXCLUDE_RN, getBuilderSessionIdCookie, isRNSDK, test } from './helpers.js';

test.describe('Tracking', () => {
  test.describe('cookies', () => {
    test('do not appear if canTrack=false', async ({ page, context, packageName }) => {
      // TO-DO: figure out why Remix fails this test
      test.fail(packageName === 'gen1-remix');

      // by waiting for network requests, we guarantee that impression tracking POST was (NOT) made,
      // which guarantees that the cookie was set or not.
      await page.goto('/can-track-false', { waitUntil: 'networkidle' });

      const cookies = await context.cookies();
      const builderSessionCookie = cookies.find(cookie => cookie.name === 'builderSessionId');
      expect(builderSessionCookie).toBeUndefined();
    });
    test('appear by default', async ({ page, context }) => {
      test.fail(EXCLUDE_RN);
      const navigate = page.goto('/');
      const trackingRequestPromise = page.waitForRequest(
        req => req.url().includes('cdn.builder.io/api/v1/track') && req.method() === 'POST'
      );

      await navigate;
      // By waiting for the tracking POST request, we guarantee that the cookie is now set.
      await trackingRequestPromise;

      const builderSessionCookie = await getBuilderSessionIdCookie({ context });

      expect(builderSessionCookie).toBeDefined();
    });
  });
  test.describe('POST data', () => {
    test('POSTs correct impression data', async ({ page }) => {
      const navigate = page.goto('/');
      const trackingRequestPromise = page.waitForRequest(
        request =>
          request.url().includes('cdn.builder.io/api/v1/track') && request.method() === 'POST'
      );

      await navigate;
      const trackingRequest = await trackingRequestPromise;

      const data = trackingRequest.postDataJSON();

      const expected = {
        events: [
          {
            type: 'impression',
            data: {
              metadata: {},
              userAttributes: {
                device: 'desktop',
              },
            },
          },
        ],
      };

      if (isRNSDK) {
        expected.events[0].data.userAttributes.device = 'mobile';
      }

      const ID_REGEX = /^[a-f0-9]{32}$/;

      expect(data).toMatchObject(expected);
      expect(data.events[0].data.sessionId).toMatch(ID_REGEX);
      expect(data.events[0].data.visitorId).toMatch(ID_REGEX);
      expect(data.events[0].data.ownerId).toMatch(/abcd/);

      if (!isRNSDK) {
        expect(data.events[0].data.metadata.url).toMatch(/http:\/\/localhost:\d+\//);
        expect(data.events[0].data.userAttributes.urlPath).toBe('/');
        expect(data.events[0].data.userAttributes.host).toMatch(/localhost:[\d]+/);
      }
    });

    test('POSTs correct click data', async ({ page }) => {
      test.skip(EXCLUDE_GEN_1);
      await page.goto('/', { waitUntil: 'networkidle' });
      const trackingRequestPromise = page.waitForRequest(
        request =>
          request.url().includes('cdn.builder.io/api/v1/track') &&
          request.method() === 'POST' &&
          request.postDataJSON().events[0].type === 'click'
      );

      // click on an element
      await page.click('text=SDK Feature testing project');

      // get click tracking request
      const trackingRequest = await trackingRequestPromise;

      const data = trackingRequest.postDataJSON();

      const expected = {
        events: [
          {
            type: 'click',
            data: {
              metadata: {},
              userAttributes: {
                device: 'desktop',
              },
            },
          },
        ],
      };

      if (isRNSDK) {
        expected.events[0].data.userAttributes.device = 'mobile';
      }

      const ID_REGEX = /^[a-f0-9]{32}$/;

      expect(data).toMatchObject(expected);

      if (!isRNSDK) {
        // check that all the heatmap metadata is present

        expect(!isNaN(parseFloat(data.events[0].data.metadata.builderElementIndex))).toBeTruthy();
        expect(!isNaN(parseFloat(data.events[0].data.metadata.builderTargetOffset.x))).toBeTruthy();
        expect(!isNaN(parseFloat(data.events[0].data.metadata.builderTargetOffset.y))).toBeTruthy();
        expect(!isNaN(parseFloat(data.events[0].data.metadata.targetOffset.x))).toBeTruthy();
        expect(!isNaN(parseFloat(data.events[0].data.metadata.targetOffset.y))).toBeTruthy();
      }

      // baseline tests for impression tracking
      expect(data.events[0].data.sessionId).toMatch(ID_REGEX);
      expect(data.events[0].data.visitorId).toMatch(ID_REGEX);
      expect(data.events[0].data.ownerId).toMatch(/abcd/);

      if (!isRNSDK) {
        expect(data.events[0].data.metadata.url).toMatch(/http:\/\/localhost:\d+\//);
        expect(data.events[0].data.userAttributes.urlPath).toBe('/');
        expect(data.events[0].data.userAttributes.host).toMatch(/localhost:[\d]+/);
      }
    });
  });
});

```


## High-Level Overview
TO-DO: figure out why Remix fails this test
by waiting for network requests, we guarantee that impression tracking POST was (NOT) made,
which guarantees that the cookie was set or not.
By waiting for the tracking POST request, we guarantee that the cookie is now set.

## Detailed Walkthrough


### Imports (2)

- `./helpers.js`
- `@playwright/test`

## Usage Examples
*No usage examples found in source. Refer to tests or documentation.*

## Dependencies and Related Files
### Imports
- `./helpers.js` (local)
- `@playwright/test` (external)

## Performance and Security Notes
*No specific performance or security concerns detected*

## Testing
This is a test file.

---
*Generated by Repo Book Generator v1.0.0*
