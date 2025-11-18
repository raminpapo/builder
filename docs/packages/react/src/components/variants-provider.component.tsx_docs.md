# variants-provider.component.tsx

## File Metadata
- **Path**: `packages/react/src/components/variants-provider.component.tsx`
- **Type**: Text file
- **Size**: 5,136 bytes (166 lines)
- **Extension**: `.tsx`
- **Language**: TypeScript (React)

## Original Source
```tsx
'use client';
import * as React from 'react';
import { builder, Builder, BuilderContent, BuilderContentVariation } from '@builder.io/sdk';

function getData(content: BuilderContentVariation) {
  if (typeof content?.data === 'undefined') {
    return undefined;
  }

  const { blocks, blocksString } = content.data;
  const hasBlocks = Array.isArray(blocks) || typeof blocksString === 'string';
  const newData: any = {
    ...content.data,
    ...(hasBlocks && { blocks: blocks || JSON.parse(blocksString) }),
  };

  delete newData.blocksString;
  return newData;
}

const variantsScript = (variantsString: string, contentId: string) =>
  `
(function() {
  if (window.builderNoTrack || !navigator.cookieEnabled) {
    return;
  }

  var variants = ${variantsString};
  function removeVariants() {
    variants.forEach(function (template) {
      document.querySelector('template[data-template-variant-id="' + template.id + '"]').remove();
    });
    document.getElementById('variants-script-${contentId}').remove();
  }

  if (typeof document.createElement("template").content === 'undefined') {
    removeVariants();
    return ;
  }

  function setCookie(name,value,days) {
    var expires = "";
    if (days) {
        var date = new Date();
        date.setTime(date.getTime() + (days*24*60*60*1000));
        expires = "; expires=" + date.toUTCString();
    }
    document.cookie = name + "=" + (value || "")  + expires + "; path=/" + "; Secure; SameSite=None";
  }

  function getCookie(name) {
    var nameEQ = name + "=";
    var ca = document.cookie.split(';');
    for(var i=0;i < ca.length;i++) {
        var c = ca[i];
        while (c.charAt(0)==' ') c = c.substring(1,c.length);
        if (c.indexOf(nameEQ) == 0) return c.substring(nameEQ.length,c.length);
    }
    return null;
  }
  var cookieName = 'builder.tests.${contentId}';
  var variantInCookie = getCookie(cookieName);
  var availableIDs = variants.map(function(vr) { return vr.id }).concat('${contentId}');
  var variantId;
  if (availableIDs.indexOf(variantInCookie) > -1) {
    variantId = variantInCookie;
  }
  if (!variantId) {
    var n = 0;
    var random = Math.random();
    for (var i = 0; i < variants.length; i++) {
      var variant = variants[i];
      var testRatio = variant.testRatio;
      n += testRatio;
      if (random < n) {
        setCookie(cookieName, variant.id);
        variantId = variant.id;
        break;
      }
    }
    if (!variantId) {
      variantId = "${contentId}";
      setCookie(cookieName, "${contentId}");
    }
  }
  if (variantId && variantId !== "${contentId}") {
    var winningTemplate = document.querySelector('template[data-template-variant-id="' + variantId + '"]');
    if (winningTemplate) {
      var parentNode = winningTemplate.parentNode;
      var newParent = parentNode.cloneNode(false);
      newParent.appendChild(winningTemplate.content.firstChild);
      parentNode.parentNode.replaceChild(newParent, parentNode);
    }
  } else if (variants.length > 0) {
    removeVariants();
  }
})()`.replace(/\s+/g, ' ');

interface VariantsProviderProps {
  initialContent: BuilderContent;
  children: (variants: BuilderContent[], renderScript?: () => JSX.Element) => JSX.Element;
}

export const VariantsProvider = ({ initialContent, children }: VariantsProviderProps) => {
  if (Builder.isBrowser && !builder.canTrack) {
    return children([initialContent]);
  }

  const hasTests = Boolean(Object.keys(initialContent?.variations || {}).length);

  if (!hasTests) return children([initialContent]);

  const variants: BuilderContent[] = Object.keys(initialContent.variations!).map(id => ({
    id,
    ...initialContent.variations![id],
    data: getData(initialContent.variations![id]!),
  }));

  const allVariants = [...variants, initialContent];
  if (Builder.isServer) {
    const variantsJson = JSON.stringify(
      Object.keys(initialContent.variations || {}).map(item => ({
        id: item,
        testRatio: initialContent.variations![item]!.testRatio,
      }))
    );
    const renderScript = () => (
      <script
        id={`variants-script-${initialContent.id}`}
        dangerouslySetInnerHTML={{
          __html: variantsScript(variantsJson, initialContent.id!),
        }}
      ></script>
    );

    // render all variants on the server side
    return <React.Fragment>{children(allVariants, renderScript)}</React.Fragment>;
  }

  const cookieName = `builder.tests.${initialContent.id}`;

  let variantId = builder.getCookie(cookieName);

  if (!variantId && Builder.isBrowser) {
    let n = 0;
    const random = Math.random();
    for (let i = 0; i < variants.length; i++) {
      const variant = variants[i];
      const testRatio = variant.testRatio;
      n += testRatio!;
      if (random < n) {
        builder.setCookie(cookieName, variant.id);
        variantId = variant.id;
        break;
      }
    }
  }

  if (!variantId) {
    // render initial content when no winning variation or on the server
    variantId = initialContent.id;
    builder.setCookie(cookieName, variantId);
  }

  return children([allVariants.find(item => item.id === variantId)!]);
};

```


## High-Level Overview
JavaScript/TypeScript module. Contains 166 lines of code.

## Detailed Walkthrough

### Functions (7)

- `getData()`
- `removeVariants()`
- `setCookie()`
- `getCookie()`
- `variantsScript()`
- `VariantsProvider()`
- `renderScript()`

### Imports (2)

- `react`
- `@builder.io/sdk`

### Exports (1)

- `VariantsProvider`

## Usage Examples
*No usage examples found in source. Refer to tests or documentation.*

## Dependencies and Related Files
### Imports
- `react` (external)
- `@builder.io/sdk` (external)

## Performance and Security Notes
⚡ **Performance**: Uses array iteration - consider performance for large datasets

## Testing
Contains 1 test cases:
- ;

---
*Generated by Repo Book Generator v1.0.0*
