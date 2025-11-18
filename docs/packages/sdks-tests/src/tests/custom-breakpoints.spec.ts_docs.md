# custom-breakpoints.spec.ts

## File Metadata
- **Path**: `packages/sdks-tests/src/tests/custom-breakpoints.spec.ts`
- **Type**: Text file
- **Size**: 7,610 bytes (233 lines)
- **Extension**: `.ts`
- **Language**: TypeScript

## Original Source
```typescript
import { expect } from '@playwright/test';
import { expectStylesForElement, isRNSDK, test } from './helpers.js';

test.describe('Custom Breakpoints', () => {
  /* set breakpoint config in content -
breakpoints: {
  small: 500,
  medium: 800,
},
*/
  test.describe('when applied', () => {
    test('large desktop size', async ({ page }) => {
      await page.setViewportSize({ width: 801, height: 1000 });

      await page.goto('/custom-breakpoints');
      const breakpointsParam = page.locator(`text=BREAKPOINTS 500 - 800`);

      let expectedTextColor = 'rgb(0, 0, 0)'; // black text color
      if (isRNSDK) {
        expectedTextColor = 'rgb(65, 117, 5)'; // greenish text color
      }

      await expect(breakpointsParam).toHaveCSS('color', expectedTextColor);

      const column2 = page.locator(`text=Column 2`);

      let expectedColumnTextColor = 'rgb(0, 0, 0)'; // black text color
      if (isRNSDK) {
        expectedColumnTextColor = 'rgb(126, 211, 33)'; // greenish text color
      }

      await expect(column2).toHaveCSS('color', expectedColumnTextColor);

      // Skipping this image test for react-native.
      // Its difficult to locate the image in react-native as css selectors don't work as expected.
      if (!isRNSDK) {
        const image = page.locator(`.builder-block:has(img.builder-image)`);

        const expectedImageCss: Record<string, string> = {
          display: 'flex',
          width: '785px',
        };

        await expectStylesForElement({
          locator: image,
          expected: expectedImageCss,
        });
      }
    });

    test('medium tablet size', async ({ page }) => {
      await page.setViewportSize({ width: 501, height: 1000 });

      await page.goto('/custom-breakpoints');
      const breakpointsPara = page.locator(`text=BREAKPOINTS 500 - 800`);

      let expectedTextColor = 'rgb(208, 2, 27)'; // reddish text color
      if (isRNSDK) {
        expectedTextColor = 'rgb(65, 117, 5)'; // greenish text color
      }

      await expect(breakpointsPara).toHaveCSS('color', expectedTextColor);

      const column2 = page.locator(`text=Column 2`);

      let expectedColumnTextColor = 'rgb(223, 22, 22)'; // reddish text color
      if (isRNSDK) {
        expectedColumnTextColor = 'rgb(126, 211, 33)'; // greenish text color
      }

      await expect(column2).toHaveCSS('color', expectedColumnTextColor);

      // Skipping this image test for react-native.
      // Its difficult to locate the image in react-native as css selectors don't work as expected.
      if (!isRNSDK) {
        const image = page.locator(`.builder-block:has(img.builder-image)`);

        const expectedImageCss: Record<string, string> = {
          display: 'none',
        };

        await expectStylesForElement({
          locator: image,
          expected: expectedImageCss,
        });
      }
    });

    test('small mobile size', async ({ page }) => {
      await page.setViewportSize({ width: 500, height: 1000 });
      await page.goto('/custom-breakpoints');

      const breakpointsPara = page.locator(`text=BREAKPOINTS 500 - 800`);
      await expect(breakpointsPara).toHaveCSS('color', 'rgb(65, 117, 5)');

      const column2 = page.locator(`text=Column 2`);
      await expect(column2).toHaveCSS('color', 'rgb(126, 211, 33)');

      // Skipping this image test for react-native.
      // Its difficult to locate the image in react-native as css selectors don't work as expected.
      if (!isRNSDK) {
        const image = page.locator(`.builder-block:has(img.builder-image)`);

        const expectedImageCss: Record<string, string> = {
          display: 'flex',
          width: '121px',
          'max-width': '250px',
        };

        await expectStylesForElement({
          locator: image,
          expected: expectedImageCss,
        });
      }
    });
  });

  test.describe('when reset', () => {
    /*
    When no breakpoints are available, defaults are applied as
    breakpoints: {
      small: 640,
      medium: 991,
    }
  */
    test('large desktop size', async ({ page }) => {
      await page.setViewportSize({ width: 992, height: 1000 });
      await page.goto('/custom-breakpoints-reset');

      const breakpointsPara = page.locator(`text=BREAKPOINTS 500 - 800`);

      let expectedTextColor = 'rgb(0, 0, 0)'; // black text color
      if (isRNSDK) {
        expectedTextColor = 'rgb(65, 117, 5)'; // greenish text color
      }

      await expect(breakpointsPara).toHaveCSS('color', expectedTextColor);

      const column2 = page.locator(`text=Column 2`);

      let expectedColumnTextColor = 'rgb(0, 0, 0)'; // black text color
      if (isRNSDK) {
        expectedColumnTextColor = 'rgb(126, 211, 33)'; // greenish text color
      }

      await expect(column2).toHaveCSS('color', expectedColumnTextColor);

      // Skipping this image test for react-native.
      // Its difficult to locate the image in react-native as css selectors don't work as expected.
      if (!isRNSDK) {
        const image = page.locator(`.builder-block:has(img.builder-image)`);

        const expectedImageCss: Record<string, string> = {
          display: 'flex',
          width: '976px',
        };

        await expectStylesForElement({
          locator: image,
          expected: expectedImageCss,
        });
      }
    });

    test('medium tablet size', async ({ page }) => {
      await page.setViewportSize({ width: 641, height: 1000 });

      await page.goto('/custom-breakpoints-reset');
      const breakpointsPara = page.locator(`text=BREAKPOINTS 500 - 800`);

      let expectedTextColor = 'rgb(208, 2, 27)'; // reddish text color
      if (isRNSDK) {
        expectedTextColor = 'rgb(65, 117, 5)'; // greenish text color
      }

      await expect(breakpointsPara).toHaveCSS('color', expectedTextColor);

      const column2 = page.locator(`text=Column 2`);

      let expectedColumnTextColor = 'rgb(223, 22, 22)'; // reddish text color
      if (isRNSDK) {
        expectedColumnTextColor = 'rgb(126, 211, 33)'; // greenish text color
      }

      await expect(column2).toHaveCSS('color', expectedColumnTextColor);

      // Skipping this image test for react-native.
      // Its difficult to locate the image in react-native as css selectors don't work as expected.
      if (!isRNSDK) {
        const image = page.locator(`.builder-block:has(img.builder-image)`);

        const expectedImageCss: Record<string, string> = {
          display: 'none',
        };

        await expectStylesForElement({
          locator: image,
          expected: expectedImageCss,
        });
      }
    });

    test('small mobile size', async ({ page }) => {
      await page.setViewportSize({ width: 640, height: 1000 });
      await page.goto('/custom-breakpoints-reset');

      const breakpointsPara = page.locator(`text=BREAKPOINTS 500 - 800`);

      await expect(breakpointsPara).toHaveCSS('color', 'rgb(65, 117, 5)');

      const column2 = page.locator(`text=Column 2`);

      await expect(column2).toHaveCSS('color', 'rgb(126, 211, 33)');

      // Skipping this image test for react-native.
      // Its difficult to locate the image in react-native as css selectors don't work as expected.
      if (!isRNSDK) {
        const image = page.locator(`.builder-block:has(img.builder-image)`);

        const expectedImageCss: Record<string, string> = {
          display: 'flex',
          width: '156px',
          'max-width': '250px',
        };

        await expectStylesForElement({
          locator: image,
          expected: expectedImageCss,
        });
      }
    });
  });
});

```


## High-Level Overview

Skipping this image test for react-native.
Its difficult to locate the image in react-native as css selectors don't work as expected.

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
