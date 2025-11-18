# component-info.ts

## File Metadata
- **Path**: `packages/sdks/src/blocks/columns/component-info.ts`
- **Type**: Text file
- **Size**: 7,759 bytes (254 lines)
- **Extension**: `.ts`
- **Language**: TypeScript

## Original Source
```typescript
import type { ComponentInfo } from '../../types/components.js';

export const componentInfo: ComponentInfo = {
  // TODO: ways to statically preprocess JSON for references, functions, etc
  name: 'Columns',
  isRSC: true,

  inputs: [
    {
      name: 'columns',
      type: 'array',
      broadcast: true,
      subFields: [
        {
          name: 'blocks',
          type: 'array',
          hideFromUI: true,
          defaultValue: [
            {
              '@type': '@builder.io/sdk:Element',
              responsiveStyles: {
                large: {
                  display: 'flex',
                  flexDirection: 'column',
                  alignItems: 'stretch',
                  flexShrink: '0',
                  position: 'relative',
                  marginTop: '30px',
                  textAlign: 'center',
                  lineHeight: 'normal',
                  height: 'auto',
                  minHeight: '20px',
                  minWidth: '20px',
                  overflow: 'hidden',
                },
              },
              component: {
                name: 'Image',
                options: {
                  image:
                    'https://builder.io/api/v1/image/assets%2Fpwgjf0RoYWbdnJSbpBAjXNRMe9F2%2Ffb27a7c790324294af8be1c35fe30f4d',
                  backgroundPosition: 'center',
                  backgroundSize: 'cover',
                  aspectRatio: 0.7004048582995948,
                },
              },
            },
            {
              '@type': '@builder.io/sdk:Element',
              responsiveStyles: {
                large: {
                  display: 'flex',
                  flexDirection: 'column',
                  alignItems: 'stretch',
                  flexShrink: '0',
                  position: 'relative',
                  marginTop: '30px',
                  textAlign: 'center',
                  lineHeight: 'normal',
                  height: 'auto',
                },
              },
              component: {
                name: 'Text',
                options: {
                  text: '<p>Enter some text...</p>',
                },
              },
            },
          ],
        },
        {
          name: 'width',
          type: 'number',
          hideFromUI: true,
          helperText: 'Width %, e.g. set to 50 to fill half of the space',
        },
        {
          name: 'link',
          type: 'url',
          helperText:
            'Optionally set a url that clicking this column will link to',
        },
      ],
      defaultValue: [
        {
          blocks: [
            {
              '@type': '@builder.io/sdk:Element',
              responsiveStyles: {
                large: {
                  display: 'flex',
                  flexDirection: 'column',
                  alignItems: 'stretch',
                  flexShrink: '0',
                  position: 'relative',
                  marginTop: '30px',
                  textAlign: 'center',
                  lineHeight: 'normal',
                  height: 'auto',
                  minHeight: '20px',
                  minWidth: '20px',
                  overflow: 'hidden',
                },
              },
              component: {
                name: 'Image',
                options: {
                  image:
                    'https://builder.io/api/v1/image/assets%2Fpwgjf0RoYWbdnJSbpBAjXNRMe9F2%2Ffb27a7c790324294af8be1c35fe30f4d',
                  backgroundPosition: 'center',
                  backgroundSize: 'cover',
                  aspectRatio: 0.7004048582995948,
                },
              },
            },
            {
              '@type': '@builder.io/sdk:Element',
              responsiveStyles: {
                large: {
                  display: 'flex',
                  flexDirection: 'column',
                  alignItems: 'stretch',
                  flexShrink: '0',
                  position: 'relative',
                  marginTop: '30px',
                  textAlign: 'center',
                  lineHeight: 'normal',
                  height: 'auto',
                },
              },
              component: {
                name: 'Text',
                options: {
                  text: '<p>Enter some text...</p>',
                },
              },
            },
          ],
        },
        {
          blocks: [
            {
              '@type': '@builder.io/sdk:Element',
              responsiveStyles: {
                large: {
                  display: 'flex',
                  flexDirection: 'column',
                  alignItems: 'stretch',
                  flexShrink: '0',
                  position: 'relative',
                  marginTop: '30px',
                  textAlign: 'center',
                  lineHeight: 'normal',
                  height: 'auto',
                  minHeight: '20px',
                  minWidth: '20px',
                  overflow: 'hidden',
                },
              },
              component: {
                name: 'Image',
                options: {
                  image:
                    'https://builder.io/api/v1/image/assets%2Fpwgjf0RoYWbdnJSbpBAjXNRMe9F2%2Ffb27a7c790324294af8be1c35fe30f4d',
                  backgroundPosition: 'center',
                  backgroundSize: 'cover',
                  aspectRatio: 0.7004048582995948,
                },
              },
            },
            {
              '@type': '@builder.io/sdk:Element',
              responsiveStyles: {
                large: {
                  display: 'flex',
                  flexDirection: 'column',
                  alignItems: 'stretch',
                  flexShrink: '0',
                  position: 'relative',
                  marginTop: '30px',
                  textAlign: 'center',
                  lineHeight: 'normal',
                  height: 'auto',
                },
              },
              component: {
                name: 'Text',
                options: {
                  text: '<p>Enter some text...</p>',
                },
              },
            },
          ],
        },
      ],
      onChange: (options: Map<string, any>) => {
        function clearWidths() {
          columns.forEach((col) => {
            col.delete('width');
          });
        }

        const columns: Array<Map<string, any>> = options.get('columns');

        if (Array.isArray(columns)) {
          const containsColumnWithWidth = !!columns.find((col) =>
            col.get('width')
          );

          if (containsColumnWithWidth) {
            const containsColumnWithoutWidth = !!columns.find(
              (col) => !col.get('width')
            );
            if (containsColumnWithoutWidth) {
              clearWidths();
            } else {
              const sumWidths = columns.reduce((memo, col) => {
                return memo + col.get('width');
              }, 0);
              const widthsDontAddUp = sumWidths !== 100;
              if (widthsDontAddUp) {
                clearWidths();
              }
            }
          }
        }
      },
    },
    {
      name: 'space',
      type: 'number',
      defaultValue: 20,
      helperText: 'Size of gap between columns',
      advanced: true,
    },
    {
      name: 'stackColumnsAt',
      type: 'string',
      defaultValue: 'tablet',
      helperText: 'Convert horizontal columns to vertical at what device size',
      enum: ['tablet', 'mobile', 'never'],
      advanced: true,
    },
    {
      name: 'reverseColumnsWhenStacked',
      type: 'boolean',
      defaultValue: false,
      helperText:
        'When stacking columns for mobile devices, reverse the ordering',
      advanced: true,
    },
  ],
};

```


## High-Level Overview
TODO: ways to statically preprocess JSON for references, functions, etc

## Detailed Walkthrough

### Functions (1)

- `clearWidths()`

### Imports (1)

- `../../types/components.js`

### Exports (1)

- `componentInfo`

## Usage Examples
*No usage examples found in source. Refer to tests or documentation.*

## Dependencies and Related Files
### Imports
- `../../types/components.js` (local)

## Performance and Security Notes
*No specific performance or security concerns detected*

## Testing
*No test information available. Check for corresponding test files.*

---
*Generated by Repo Book Generator v1.0.0*
