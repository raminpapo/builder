# content-variants.lite.tsx

## File Metadata
- **Path**: `packages/sdks/src/components/content-variants/content-variants.lite.tsx`
- **Type**: Text file
- **Size**: 4,611 bytes (152 lines)
- **Extension**: `.tsx`
- **Language**: TypeScript (React)

## Original Source
```tsx
import {
  For,
  Show,
  onMount,
  useMetadata,
  useStore,
  useTarget,
} from '@builder.io/mitosis';
import { TARGET } from '../../constants/target.js';
import { handleABTestingSync } from '../../helpers/ab-tests.js';
import { getDefaultCanTrack } from '../../helpers/canTrack.js';
import ContentComponent from '../content/content.lite.jsx';
import InlinedScript from '../inlined-script.lite.jsx';
import InlinedStyles from '../inlined-styles.lite.jsx';
import type { ContentVariantsPrps } from './content-variants.types.js';
import {
  checkShouldRenderVariants,
  getScriptString,
  getUpdateCookieAndStylesScript,
  getVariants,
} from './helpers.js';

useMetadata({
  rsc: {
    componentType: 'server',
  },
  qwik: {
    setUseStoreFirst: true,
  },
});

type VariantsProviderProps = ContentVariantsPrps & {
  /**
   * For internal use only. Do not provide this prop.
   */
  __isNestedRender?: boolean;
};

export default function ContentVariants(props: VariantsProviderProps) {
  onMount(() => {
    /**
     * We unmount the non-winning variants post-hydration in Vue.
     */
    useTarget({
      vue: () => {
        state.shouldRenderVariants = false;
      },
      solid: () => {
        state.shouldRenderVariants = false;
      },
      svelte: () => {
        state.shouldRenderVariants = false;
      },
    });
  });

  const state = useStore({
    shouldRenderVariants: checkShouldRenderVariants({
      canTrack: getDefaultCanTrack(props.canTrack),
      content: props.content,
    }),
    get updateCookieAndStylesScriptStr() {
      return getUpdateCookieAndStylesScript(
        getVariants(props.content).map((value) => ({
          id: value.testVariationId!,
          testRatio: value.testRatio,
        })),
        props.content?.id || ''
      );
    },

    get hideVariantsStyleString() {
      return getVariants(props.content)
        .map((value) => `.variant-${value.testVariationId} { display: none; } `)
        .join('');
    },
    get defaultContent() {
      return state.shouldRenderVariants
        ? { ...props.content, testVariationId: props.content?.id }
        : handleABTestingSync({
            item: props.content,
            canTrack: getDefaultCanTrack(props.canTrack),
          });
    },
  });

  return (
    <>
      <Show when={!props.__isNestedRender && TARGET !== 'reactNative'}>
        <InlinedScript scriptStr={getScriptString()} />
      </Show>
      <Show when={state.shouldRenderVariants}>
        <InlinedStyles
          id={`variants-styles-${props.content?.id}`}
          styles={state.hideVariantsStyleString}
        />
        {/* Sets A/B test cookie for all `RenderContent` to read */}
        <InlinedScript scriptStr={state.updateCookieAndStylesScriptStr} />

        <For each={getVariants(props.content)}>
          {(variant) => (
            <ContentComponent
              key={variant.testVariationId}
              content={variant}
              showContent={false}
              model={props.model}
              data={props.data}
              context={props.context}
              apiKey={props.apiKey}
              apiVersion={props.apiVersion}
              customComponents={props.customComponents}
              linkComponent={props.linkComponent}
              canTrack={props.canTrack}
              locale={props.locale}
              enrich={props.enrich}
              isSsrAbTest={state.shouldRenderVariants}
              blocksWrapper={props.blocksWrapper}
              blocksWrapperProps={props.blocksWrapperProps}
              contentWrapper={props.contentWrapper}
              contentWrapperProps={props.contentWrapperProps}
              trustedHosts={props.trustedHosts}
            />
          )}
        </For>
      </Show>
      <ContentComponent
        {...useTarget({
          vue: { key: state.shouldRenderVariants.toString() },
          default: {},
        })}
        content={state.defaultContent}
        showContent
        model={props.model}
        data={props.data}
        context={props.context}
        apiKey={props.apiKey}
        apiVersion={props.apiVersion}
        customComponents={props.customComponents}
        linkComponent={props.linkComponent}
        canTrack={props.canTrack}
        locale={props.locale}
        enrich={props.enrich}
        isSsrAbTest={state.shouldRenderVariants}
        blocksWrapper={props.blocksWrapper}
        blocksWrapperProps={props.blocksWrapperProps}
        contentWrapper={props.contentWrapper}
        contentWrapperProps={props.contentWrapperProps}
        trustedHosts={props.trustedHosts}
      />
    </>
  );
}

```


## High-Level Overview
For internal use only. Do not provide this prop.

We unmount the non-winning variants post-hydration in Vue.


## Detailed Walkthrough

### Functions (1)

- `ContentVariants()`

### Imports (7)

- `../inlined-styles.lite.jsx`
- `../../helpers/canTrack.js`
- `./content-variants.types.js`
- `../../constants/target.js`
- `../inlined-script.lite.jsx`
- `../content/content.lite.jsx`
- `../../helpers/ab-tests.js`

### Exports (1)

- `ContentVariants`

## Usage Examples
*No usage examples found in source. Refer to tests or documentation.*

## Dependencies and Related Files
### Imports
- `../inlined-styles.lite.jsx` (local)
- `./helpers.js` (local)
- `../../helpers/canTrack.js` (local)
- `./content-variants.types.js` (local)
- `../../constants/target.js` (local)
- `@builder.io/mitosis` (external)
- `../inlined-script.lite.jsx` (local)
- `../content/content.lite.jsx` (local)
- `../../helpers/ab-tests.js` (local)

## Performance and Security Notes
*No specific performance or security concerns detected*

## Testing
*No test information available. Check for corresponding test files.*

---
*Generated by Repo Book Generator v1.0.0*
